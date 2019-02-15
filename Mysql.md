## Mysql

#### 事务的隔离级别

mysql默认的事务处理级别是'REPEATABLE-READ',也就是可重复读

1.查看当前会话隔离级别

select @@tx_isolation;

2.查看系统当前隔离级别

select @@global.tx_isolation;

3.设置当前会话隔离级别

set session transaction isolatin level repeatable read;

4.设置系统当前隔离级别

set global transaction isolation level repeatable read;

### 常见sql

查看mysql版本   

` SELECT VERSION();`

查看表的描述

` desc table_name;`

#### 存储过程相关 

-- 查看所有的存储过程
show PROCEDURE status; 

-- 查看特定数据库存储过程
show PROCEDURE status where db='test'; 

-- 用指定的登录名查看该用户创建的存储过程
show PROCEDURE status where definer='root@localhost';  -- @localhost为用户登录位置(本地登录)

-- 查看指定时间段创建存储过程
show PROCEDURE status where created between '2017-02-17 00:00:00' 
and '2017-02-17 23:59:59';

用系统表mysql.proc来查看：
-- 查看所有的存储过程信息
select * from mysql.proc;

-- 查看特定数据库里的存储过程
select * from mysql.proc where db='test'; 

-- 查看某个用户定义的存储过程
select * from mysql.proc where definer='root@localhost'; 

-- 查看某时间段创建的存储过程
select * from mysql.proc where created between '2017-02-17 00:00:00' 
and '2017-02-17 23:59:59';

#### 触发器相关

--查看当前库中所有的触发器:
SELECT * FROM Sysobjects WHERE xtype = 'TR'

--查看当前库中所有的触发器和与之相对应的表：
SELECT tb2.name AS tableName,tb1.name AS triggerName FROM Sysobjects tb1 JOIN Sysobjects tb2 ON tb1.parent_obj=tb2.id WHERE tb1.type='TR'

--显示触发器的定义:
EXEC sp_helptext '触发器名'

--查看触发器的有关信息:
EXEC sp_help '触发器名'

--查看表中的触发器类型:
EXEC sp_helptrigger '表名'