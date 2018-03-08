We need to record the execution result of each test. A **schedule** may contain many tests, and the tests will be executed one by one.
One test may be pass/fail, or get some error during execution.
One test may depend on another one, so it will be blocked if its depending test is failed.
So we get the following status of an execution:
- Pass: test passed. The system works as expected.
- Fail: test failed. There may be some bug.
- Block: depending test is fail/block. We don't need to run this test.
- Error: unexpected exception occurs during the execution.

We need to know **when** the result is submitted.
Below is the information we needed for an execution result record:
- ID: primary key
- Name: name of the test
- Result: execution result. 'a'=Initial, 'b'=Pass, 'c'='Fail, 'd'=Block, 'e'=Error, ....
- Schedule_id: which schedule this execution belongs to, should be existing.
- Submitted_time: when this record is submitted. Generally it means when the execution of this test is finished.
- Started_time: Start time of the execution. Generally it means when the test is started to run.

From the above schema, we can draw the below table definition.
```mysql
CREATE TABLE `executions` (
  `execution_id` int(11) NOT NULL AUTO_INCREMENT,
  `script_name` varchar(100) NOT NULL,
  `execution_result` char(1) NOT NULL DEFAULT 'a',
  `schedule_id` int(11) NOT NULL,
  `started_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `submitted_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
   CONSTRAINT `executions_ibfk_1` FOREIGN KEY (`schedule_id`) REFERENCES `schedule` (`schedule_id`) ON DELETE CASCADE,
  PRIMARY KEY (`execution_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

