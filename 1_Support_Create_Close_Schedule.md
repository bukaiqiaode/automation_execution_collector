To help us to know the start/end time of each schedule, timestamps are used. One timestamp is created, when creating a new schedule. One timestamp is used, when closing a schedule. Then we get the data-format of a schedule:

- ID: primary key, automatic increasing
- Name: name of the schedule, GUID
- Status: flag the status of the schedule
- Created time: timestamp
- last_modify_time: timestamp
- client_token: to identify the owner of this schedule

## Statement to create table for schedule under MySQL:
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

## Python bottle service to query the schedules from MySQL and return to client in Json:
```python
# -*- coding: UTF-8 -*-
import bottle
import json
import mysql.connector
import time
from mysql.connector import errorcode
from datetime import date, datetime, timedelta

class mschedule(object):
    def __init__(self, id, name, status, created_time, last_modified_time):
        self.id = id
        self.name = name
        self.status = status
        self.created_time = created_time
        self.last_modified_time = last_modified_time
    def toDict(self):
        '''
        Use a dictionary to hold the data and return
        The reason is that Json module only do convertion for dictionary
        '''
        info = {}
        info['id'] = self.id
        info['name'] = self.name
        info['status'] = self.status
        info['created_time'] = time.mktime(self.created_time.timetuple())
        info['last_modified_time'] = time.mktime(self.last_modified_time.timetuple())
        return info

@bottle.route('/allschedule', method = ['GET'])    
def all_schedules_json():    
    #connect to the server, doesn't specify the database to use
    cnx = mysql.connector.connect(**config)

    #get the cursor to use
    cursor = cnx.cursor()
    
    #query data from table
    query = (
        "select schedule_id, schedule_name, schedule_status, created_time, last_modify_time from schedule"
        )
    cursor.execute(query)

    ret_content = []
    for (schedule_id, schedule_name, schedule_status, created_time, last_modify_time) in cursor:
        tmp_schedule = mschedule(schedule_id, schedule_name, schedule_status, created_time, last_modify_time)
        ret_content.append(tmp_schedule.toDict())
            
    #close the cursor
    cursor.close()
    cnx.close()
    
    #return str(ret_content)
    return {"response":ret_content} 
```

## Python code for adding a schedule into the database
Technical point:
- Define the method supported for the request.
- The way to get parameter from GET/POST `bottle.request.GET.get('xxx')`
- The way to identify the mehod used by the request.
- The way to generate a UUID using python and change it to string.
- The way to insert a record into MySQL using Connector/Python

```python
@bottle.route('/xxx/schedule/add', method= ['GET', 'POST'])
def add_execution_schedule():
    '''
    Create a new schedule, to hold the execution results
    1. The method should be post.
    2. Check the token, which identifies different clients.
    3. Generate a UUID as the name of the schedule.
    4. Create a new schedule in the database.
    5. Return the name of the schedule
    '''    
    #method should not be 'get'    
    if bottle.request.method == 'GET':
        return ''
    
    token = get_client_token()

    if verify_client_token(token) is True:
        #create the name for the schedule
        temp_schedule_name = str(uuid.uuid1())
        
        #create a record for the schedule in the database
        cnx = mysql.connector.connect(**config)
        cursor = cnx.cursor()
        add_st = (
            "insert into schedule "
            "(schedule_name, client_token)"
            "values( %s, %s )"
            )
        the_name = temp_schedule_name
        the_token = token
        record_a = (the_name, the_token)
        cursor.execute(add_st, record_a)
        cnx.commit()
        cursor.close()
        cnx.close()
        
        #return the name of the schedule
        return the_name
    else:
        #may be an attack, ignore
        return ''
```

## Python code for closing a schedule in the database
Technical point:
- Definition of statement, when there is only one parameter for the SQL query.

```python
@bottle.route('/xxx/schedule/close', method= ['GET', 'POST'])
def execution_schedule_close():
    '''
    Close a new schedule for a client
    1. The method should be post.
    2. Check the token, which identifies different clients.
    3. Close the schedule of the client.
    
    It is better to use stored procedure to find the most recent and then close it.
    But to make it easier, here we just close all the schedules for the client.
    '''    
    #method should not be 'get'    
    if bottle.request.method == 'GET':
        return ''
    
    token = get_client_token()

    if verify_client_token(token) is True:
        cnx = mysql.connector.connect(**config)
        cursor = cnx.cursor()
        close_st = (
            "update schedule "
            "set schedule_status = 'b'"
            "where client_token = '%s'"
            )
        the_token = token
        cursor.execute(close_st%(the_token))
        cnx.commit()
        cursor.close()
        cnx.close()
        
        #return the name of the schedule
        return ''
    else:
        #may be an attack, ignore
        return ''
```

## Python code for requesting to add/close a schedule from client
Technical point:
- Use request to post data.

```python
import requests

def create_schedule():
    mtoken = 'xxx'
    
    r = requests.post('http://a.b.c.d:port/xxx/schedule/add',
                     data = {'xxx': mtoken}
                     )
    print r.content.strip()

def close_schedule():
    mtoken = 'xxx'
    
    r = requests.post('http://a.b.c.d:port/xxx/schedule/close',
                     data = {'xxx': mtoken}
                     )
    print r.content.strip()
```
