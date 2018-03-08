We need to record the execution result of each test. A **schedule** may contain many tests, and the tests will be executed one by one.
One test may be pass/fail, or get some error during execution.
One test may depend on another one, so it will be blocked if its depending test is failed.
So we get the following status of an execution:
- Pass: test passed. The system works as expected.
- Fail: test failed. There may be some bug.
- Block: depending test is fail/block. We don't need to run this test.
- Error: unexpected exception occurs during the execution.

We need to know **when** the result is submitted.
## Information we needed for an execution result record
- ID: primary key
- Name: name of the test
- Result: execution result. 'a'=Initial, 'b'=Pass, 'c'='Fail, 'd'=Block, 'e'=Error, ....
- Schedule_id: which schedule this execution belongs to, should be existing.
- Submitted_time: when this record is submitted. Generally it means when the execution of this test is finished.
- Started_time: Start time of the execution. Generally it means when the test is started to run.

## Table schema for execution results
```mysql
CREATE TABLE `executions` (
  `execution_id` int(11) NOT NULL AUTO_INCREMENT,
  `script_name` varchar(100) NOT NULL,
  `execution_result` char(1) NOT NULL DEFAULT 'a',
  `schedule_id` int(11) NOT NULL,
  `started_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `submitted_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
   CONSTRAINT `executions_ibfk_1` FOREIGN KEY (`schedule_id`) 
   REFERENCES `schedule` (`schedule_id`) ON DELETE CASCADE,
  PRIMARY KEY (`execution_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

## Stored procedure to submit execution results into database
```mysql
DELIMITER $$

CREATE PROCEDURE `addexecutionresult`(
    `c_token` VARCHAR(36),
    `ex_script` VARCHAR(100),
    `ex_result` CHAR(1)
    )
BEGIN 

DECLARE notfound INT DEFAULT 0; 
DECLARE s_id INT(11) DEFAULT 0; 
DECLARE v_flag INT DEFAULT 0; 

DECLARE cur1 CURSOR FOR 
SELECT MAX(schedule_id) FROM schedule
    WHERE client_token = c_token AND schedule_status = 'a'; 

DECLARE CONTINUE HANDLER FOR NOT FOUND SET notfound = 1; 
  
OPEN cur1; 
FETCH cur1 INTO s_id; 
IF s_id is null THEN
    SET v_flag = -1; 
ELSE
    SET v_flag = 0;
    INSERT INTO executions(
        script_name,
        execution_result,
        schedule_id
      )
    VALUES(ex_script, ex_result, s_id); 
END IF;
CLOSE cur1;
SELECT v_flag;
END$$

DELIMITER ;
```

Then we can submit execution results from Python server
```python
@bottle.route('/xxx/result/add', method= ['GET', 'POST'])
def execution_result_add():
    '''
    Insert one execution result for a schedule.
    1. The method should be post.
    2. Check the token, which identifies different clients.
    3. process the script name to replace (`, BLANK) with t
    4. validation the result
    5. submit the result
    '''    
    #method should not be 'get'    
    if bottle.request.method == 'GET':
        return ''
    
    token = get_client_token()

    if verify_client_token(token) is True:
        #examine the parameters
        ex_script = bottle.request.POST.get('script')
        ex_result = bottle.request.POST.get('result')

        #validate the script name
        if '`' in ex_script or ' ' in ex_script or len(ex_script) >= 99:
            return ""

        #validate the result
        if (ex_result.isalpha() is False) or (len(ex_result) != 1):
            return ''
        
        #submit the result to database
        cnx = mysql.connector.connect(**config)
        cursor = cnx.cursor()
        
        args = (token, ex_script, ex_result)
        cursor.callproc("addexecutionresult", args)
        
        cnx.commit()
        cursor.close()
        cnx.close()
        
        #return the name of the schedule
        return 'success'
    else:
        #may be an attack, ignore
        return ''
```


## References
- Using stored procedure with Python/MySQL: https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-callproc.html
