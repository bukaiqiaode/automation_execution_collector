# automation_execution_collector
Some notes of how I tried to automate the sanity execution for daily build.


Each day, we execute automated scripts for daily build. We want to keep the execution result of each script for further analysis/presentation. We need some method to identify the date of the execution result. And the solution is to group the execution of scripts into something like 'schedule', where:

- A message is sent to server to create a new 'schedule', each time starting execution for a daily build.
- After execution of each script, the result is reported to the server. The server will associate the result with the 'schedule'.
- After all executions are finished, a message is sent to server to 'close' the 'schedule'.
- When querying, we can get the information like 'script xxx is pass/fail/error/block in schedule yyy'.

To handle the exception conditions of the workflow:

- Client need to append its token, when communicating with server, to avoid illigal attacks.
- A 'schedule' should have flag to identify status like 'created/running/closed/...'.
- Client need to specify the 'schedule' when requesting to append an execution result.
- Client need to 'closed' the schedule, when execution is finished.
- Appending to a 'closed schedule' will be refused.
- Administrator can help to update results for 'closed schedule'.

To help us to know the start/end time of each schedule, timestamps are used. One timestamp is created, when creating a new schedule. One timestamp is used, when closing a schedule. Then we get the data-format of a schedule:

- ID: primary key, automatic increasing
- Name: name of the schedule, GUID
- Status: flag the status of the schedule
- Created time: timestamp
- last_modify_time: timestamp

Statement to create table for schedule under MySQL:
```sql
CREATE TABLE `schedule` (
  `schedule_id` int(11) NOT NULL AUTO_INCREMENT,
  `schedule_name` varchar(36) NOT NULL,
  `schedule_status` char(1) DEFAULT 'a',
  `created_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `last_modify_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`schedule_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```
