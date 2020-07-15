---
title: MySql DQL、DML、DDL、DCL概念
date: 2020-07-15 11:07:00
tags: 
    - MySql
description: 浏览文档的过程中经常能遇到DQL、DML、DDL、DCL这些使用的限制，本文对上述这几个概念及相关操作进行一个简单的介绍。
---
SQL(structure Query Language)语言共分为四大类：数据查询语言DQL，数据操纵语言DML，数据定义语言DDL，数据控制语言DCL。

1. 数据查询语言DQL(data query language)
    
    DQL即是由SELECT子句，FROM子句，WHERE子句组成的查询块，也就是我们平时用的查询语句：
    ```sql
    SELECT COLUMN_NAME,COLUMN_NAME
    FROM TABLE_NAME [WHERE Clause] [LIMIT N][OFFSET M]
    ```

2. 数据操纵语言DML(data manipulation language)
    DML是我们平时对数据更新的操作，主要有如下三种：
    1. 插入：INSERT
        ```sql
        INSERT INTO TABLE_NAME (field1,field2,...fieldN)
        VALUES (value1,value2,...valueN);
        ```
    2. 更新：UPDATE
        ```sql
        UPDATE TABLE_NAME
        SET field1=NEW-value1,field2=NEW-value2 [WHERE Clause]
        ```
    3. 删除：DELETE
        ```sql
        DELETE FROM table_name [WHERE Clause]
        ```
    
3. 数据定义语言DDL(data defination language)
    DDL用来创建数据库中的各种的表、视图、索引、同义词、聚簇等：
    ```sql
     CREATE TABLE/VIEW/INDEX/SYN/CLUSTER --(表,视图,索引,同义词,簇)
    ```
    TODO 同义词和簇的作用
    
    **[注]** DDL操作是隐性提交的，不能rollback。

4. 数据控制语言DCL(data control language)
DCL用来授予或回收访问数据库的某种特权，并控制数据库操纵事务发生的时间及效果，对数据库实行监视等。如：
    1. 授权：GRANT
    2. 禁止：DENY
    3. 回收：REVOKE
    4. 提交：COMMIT
    5. 回滚：ROLLBACK

5. 其他:
    - 事务控制语句TCL(transaction control language)：
        1. SAVEPOINT：保存点    
        2. ROLLBACK：回退到某点
        3. COMMIT：提交事务
    - 提交类型
        1. 显式提交
        ```sql
        COMMIT
        ```
        2. 自动提交
        ```sql
        SET AUTOCOMMIT ON
        ```
        插入、修改、删除语句执行后，系统将自动进行提交
        3. 隐式提交
        ALTER，AUDIT，COMMENT，CONNECT，CREATE，DISCONNECT，DROP，EXIT，GRANT，NOAUDIT，QUIT，REVOKE，RENAME