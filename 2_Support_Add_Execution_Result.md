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
- Schedule_id: which schedule this execution belongs to
- Updated_time: when this record is updated

