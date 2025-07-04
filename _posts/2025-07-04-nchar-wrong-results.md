---
title: "Wrong Results with nchar & flash cache "
date: 2025-05-12
layout: post
tags: sqlcl oracle
---

Recently a security maintenance was applied to our Exadata Cloud Infrastructure system. As part of this, storage cells were upgraded to **OSS_25.1.5.0.0_LINUX.X64_250530**.

We started to notice some problems on our JD Edwards EnterpriseOne system (which uses NCHAR datatypes extensively).

Let me share the testcase and results:

```sql
DROP TABLE t;

CREATE TABLE t(a NCHAR(2), padding VARCHAR2(1000));

INSERT INTO t   
SELECT CASE 
   WHEN MOD(level, 1000) = 0 THEN 'A'
   WHEN MOD(level, 1000) = 1 THEN 'B'
   ELSE 'X'
 END,
 RPAD('X', 1000, 'X')
FROM dual CONNECT BY level <= 100000;

SELECT count(*) FROM t WHERE a IN ('A','B');  
SELECT count(*) FROM t WHERE a IN ('A','B');
```


```sql
SQL> DROP TABLE t;
Table T dropped.

SQL> CREATE TABLE t(a NCHAR(2), padding VARCHAR2(1000));
Table T created.

SQL> INSERT INTO t   
  2  SELECT CASE 
  3      WHEN MOD(level, 1000) = 0 THEN 'A'
  4      WHEN MOD(level, 1000) = 1 THEN 'B'
  5      ELSE 'X'
  6    END,
  7  RPAD('X', 1000, 'X')
  8* FROM dual CONNECT BY level <= 100000;
100,000 rows inserted.

SQL> SELECT count(*) FROM t WHERE a IN ('A','B');  
   COUNT(*) 
___________ 
        200 

SQL> SELECT count(*) FROM t WHERE a IN ('A','B'); 
   COUNT(*) 
___________ 
          1
```

**Warning:** If you have any JD Edwards system running on Exadata (or use NCHAR datatypes for other reasons), please be cautious before applying this security patch (or allowing Oracle Cloud to apply it automatically).

**Workaround** We've found that disabling flash cache on affected tables resolves the issue using the following command, until we can get the patch rolled back:
`sql> ALTER TABLE t STORAGE(cell_flash_cache none);`