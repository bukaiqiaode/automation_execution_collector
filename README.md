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
- Appending to a 'closed schedule' will be ignored.
- Administrator can help to update results for 'closed schedule'.
- A new shedule will be created, ignoring where there is still non-closed schedule for the client, when creating request received.
- Administrator can use seperate scripts to check unexpected condition of the system, and figure out the root-cause.


## Python code to verify time-format using regular expression
```python
  import re
  
  pStr = "xxxx"
  #24-hour format
  result = re.match( r'[012][0-9]:[012345][0-9]', pStr, re.I)
  
  #12-hour format
  pattern_1 = r'[1][012]:[012345][0-9] [AP]M'
  pattern_2 = r'[1-9]:[012345][0-9] [AP]M'
  result_1 = re.match(pattern_1, str_val, re.I)
  result_2 = re.match(pattern_2, str_val, re.I)
  if result_1 or result_2:
    pass
  
```
