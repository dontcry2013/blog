---
title: MySQL find_in_set
date: 2020-02-17 21:21:33
categories: [Database]
tags: [MySQL]
---
# MySQL FIND_IN_SET Function

MySQL `FIND_IN_SET()` function to return the position of a string in a comma-separated list of strings.

> FIND_IN_SET(needle,haystack);

``` sql
SELECT FIND_IN_SET('y','x,y,z'); -- 2
SELECT FIND_IN_SET('a',''); -- 0
```
<!--more-->


``` sql
CREATE TABLE IF NOT EXISTS divisions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(25) NOT NULL,
    belts VARCHAR(200) NOT NULL
);

INSERT INTO divisions(name,belts)
VALUES ('O-1','white,yellow,orange'),
    ('O-2','purple,green,blue'),
    ('O-3','brown,red,black'),
    ('O-4','white,yellow,orange'),
    ('O-5','purple,green,blue'),
    ('O-6','brown,red'),
    ('O-7','black'),
    ('O-8','white,yellow,orange'),
    ('O-9','purple,green,blue'),
    ('O-10','brown,red');

SELECT name, belts FROM divisions WHERE FIND_IN_SET('red', belts);
```

`SELECT` will return

name|belts
---|---
O-3|brown,red,black
O-6|brown,red
O-10|brown,red

## NOT operator
`SELECT name, belts FROM divisions WHERE NOT FIND_IN_SET('red', belts);`

NOT operator with the FIND_IN_SET() function to find the divisions that do not belong the red belt:


# My Example

Tabel `teaching_arrangements` has a column called `teaching_branch_session_time_id`
which formed with `teaching_branch_session_times`'s primary key, and joined by `','`.

id | cohort_id | teaching_branch_session_time_id
---|---|---
59706|377|1490,1491
59720|378|1490,1491
59734|376|1490,1491
59748|369|1494,1495
59764|368|1494,1495
59780|367|1494,1495

In order to get the start time and end time of each session, `find_in_set` is required to introduce in.

Another thing is we should not `join` tables, use the so called `implicit join`


``` sql
SELECT 
  _origin.id, _origin.cohort_id, _origin.teaching_branch_session_time_id, min(start_time), max(end_time) 
FROM ( 
  SELECT 
    ta.id, cohort_id, ta.teaching_branch_session_time_id, tbst.id AS session_id, tbst.start_time, tbst.end_time 
  FROM 
    teaching_arrangements ta, teaching_branch_session_times tbst 
  WHERE 
    ta.date = '2020-02-18' AND find_in_set(tbst.id, ta.teaching_branch_session_time_id) 
) _origin GROUP BY id
```

id|cohort_id|teaching_branch_session_time_id|min(start_time)|max(end_time)
---|---|---|---|---
59706|377|1490,1491|08:00:00|09:45:00
59720|378|1490,1491|08:00:00|09:45:00
59734|376|1490,1491|08:00:00|09:45:00
59748|369|1494,1495|14:00:00|15:45:00
59764|368|1494,1495|14:00:00|15:45:00
59780|367|1494,1495|14:00:00|15:45:00

# Question

Why the first used more time to get results?

``` sql
SELECT 
  _arrangements.*, tbst.id AS session_id, tbst.start_time, tbst.end_time 
FROM 
  (
    SELECT 
      ta.id, cohort_id, ta.teaching_branch_session_time_id 
    FROM 
      teaching_arrangements ta 
    WHERE 
      ta.date = '2020-02-18') AS _arrangements
  ), teaching_branch_session_times tbst
```

``` sql
SELECT 
    ta.id, cohort_id, ta.teaching_branch_session_time_id, tbst.id AS session_id, tbst.start_time, tbst.end_time 
  FROM 
    teaching_arrangements ta, teaching_branch_session_times tbst 
  WHERE 
    ta.date = '2020-02-18'
```
