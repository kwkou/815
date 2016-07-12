<properties
   pageTitle="SQL 数据仓库 Transact-SQL 参考 | Azure"
   description="SQL 数据仓库使用的 Transact-SQL 主题参考内容的链接。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/08/2016"
   wacn.date="04/11/2016"/>

#Transact-SQL 主题

## 数据定义语言 (DDL) 语句

- [ALTER DATABASE](https://msdn.microsoft.com/zh-cn/library/mt204042.aspx)
- [ALTER INDEX](https://msdn.microsoft.com/zh-cn/library/ms188388.aspx)
- [ALTER PROCEDURE](https://msdn.microsoft.com/zh-cn/library/ms189762.aspx)
- [ALTER SCHEMA](https://msdn.microsoft.com/zh-cn/library/ms173423.aspx)
- [ALTER TABLE](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)
- [CREATE COLUMNSTORE INDEX](https://msdn.microsoft.com/zh-cn/library/gg492153.aspx)
- [CREATE DATABASE](https://msdn.microsoft.com/zh-cn/library/mt204021.aspx)
- [CREATE DATABASE SCOPED CREDENTIAL](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)
- [CREATE EXTERNAL DATA SOURCE](https://msdn.microsoft.com/zh-cn/library/dn935022.aspx)
- [CREATE EXTERNAL FILE FORMAT](https://msdn.microsoft.com/zh-cn/library/dn935026.aspx)
- [CREATE EXTERNAL TABLE](https://msdn.microsoft.com/zh-cn/library/dn935021.aspx)
- [CREATE FUNCTION](https://msdn.microsoft.com/zh-cn/library/mt203952.aspx)
- [CREATE INDEX](https://msdn.microsoft.com/zh-cn/library/ms188783.aspx)
- [CREATE PROCEDURE](https://msdn.microsoft.com/zh-cn/library/ms187926.aspx)
- [CREATE SCHEMA](https://msdn.microsoft.com/zh-cn/library/ms189462.aspx)
- [CREATE STATISTICS](https://msdn.microsoft.com/zh-cn/library/ms188038.aspx)
- [CREATE TABLE](https://msdn.microsoft.com/zh-cn/library/mt203953.aspx)
- [CREATE TABLE AS SELECT](https://msdn.microsoft.com/zh-cn/library/mt204041.aspx)
- [CREATE VIEW](https://msdn.microsoft.com/zh-cn/library/ms187956.aspx)
- [DROP EXTERNAL DATA SOURCE](https://msdn.microsoft.com/zh-cn/library/mt146367.aspx)
- [DROP EXTERNAL FILE FORMAT](https://msdn.microsoft.com/zh-cn/library/mt146379.aspx)
- [DROP EXTERNAL TABLE](https://msdn.microsoft.com/zh-cn/library/mt130698.aspx)
- [DROP INDEX](https://msdn.microsoft.com/zh-cn/library/ms176118.aspx)
- [DROP PROCEDURE](https://msdn.microsoft.com/zh-cn/library/ms174969.aspx)
- [DROP STATISTICS](https://msdn.microsoft.com/zh-cn/library/ms175075.aspx)
- [DROP TABLE](https://msdn.microsoft.com/zh-cn/library/ms173790.aspx)
- [DROP SCHEMA](https://msdn.microsoft.com/zh-cn/library/ms186751.aspx)
- [DROP VIEW](https://msdn.microsoft.com/zh-cn/library/ms173492.aspx)
- [RENAME](https://msdn.microsoft.com/zh-cn/library/mt631611.aspx)
- [TRUNCATE TABLE](https://msdn.microsoft.com/zh-cn/library/ms177570.aspx)
- [UPDATE STATISTICS](https://msdn.microsoft.com/zh-cn/library/ms187348.aspx)

## 数据操作语言 (DML) 语句

- [删除](https://msdn.microsoft.com/zh-cn/library/ms189835.aspx)
- [INSERT](https://msdn.microsoft.com/zh-cn/library/ms174335.aspx)
- [UPDATE](https://msdn.microsoft.com/zh-cn/library/ms177523.aspx)

## 数据库控制台命令

- [DBCC DROPCLEANBUFFERS](https://msdn.microsoft.com/zh-cn/library/ms187762.aspx)
- [DBCC FREEPROCCACHE](https://msdn.microsoft.com/zh-cn/library/mt204018.aspx)
- [DBCC SHRINKLOG](https://msdn.microsoft.com/zh-cn/library/mt204020.aspx)
- [DBCC PDW\_SHOWEXECUTIONPLAN](https://msdn.microsoft.com/zh-cn/library/mt204017.aspx)
- [DBCC PDW\_SHOWPARTITIONSTATS](https://msdn.microsoft.com/zh-cn/library/mt204013.aspx)
- [DBCC PDW\_SHOWSPACEUSED](https://msdn.microsoft.com/zh-cn/library/mt204028.aspx)
- [DBCC SHOW\_STATISTICS](https://msdn.microsoft.com/zh-cn/library/mt204043.aspx)

## 查询语句

- [SELECT](https://msdn.microsoft.com/zh-cn/library/ms189499.aspx)
- [WITH common\_table\_expression](https://msdn.microsoft.com/zh-cn/library/ms175972.aspx)
- [EXCEPT 和 INTERSECT](https://msdn.microsoft.com/zh-cn/library/ms188055.aspx)
- [EXPLAIN](https://msdn.microsoft.com/zh-cn/library/mt631615.aspx)
- [从](https://msdn.microsoft.com/zh-cn/library/ms177634.aspx)
- [使用 PIVOT 和 UNPIVOT](https://msdn.microsoft.com/zh-cn/library/ms177410.aspx)
- [GROUP BY](https://msdn.microsoft.com/zh-cn/library/ms177673.aspx)
- [HAVING](https://msdn.microsoft.com/zh-cn/library/ms180199.aspx)
- [ORDER BY](https://msdn.microsoft.com/zh-cn/library/ms188385.aspx)
- [OPTION](https://msdn.microsoft.com/zh-cn/library/ms190322.aspx)
- [UNION](https://msdn.microsoft.com/zh-cn/library/ms180026.aspx)
- [WHERE](https://msdn.microsoft.com/zh-cn/library/ms188047.aspx)
- [TOP](https://msdn.microsoft.com/zh-cn/library/ms189463.aspx)
- [别名](https://msdn.microsoft.com/zh-cn/library/mt631614.aspx)
- [搜索条件](https://msdn.microsoft.com/zh-cn/library/ms173545.aspx)
- [子查询](https://msdn.microsoft.com/zh-cn/library/mt631613.aspx)

## 安全语句

- 权限：[GRANT](https://msdn.microsoft.com/zh-cn/library/ms187965.aspx)、[DENY](https://msdn.microsoft.com/zh-cn/library/ms188338.aspx)、[REVOKE](https://msdn.microsoft.com/zh-cn/library/ms187728.aspx)
- [ALTER AUTHORIZATION](https://msdn.microsoft.com/zh-cn/library/ms187359.aspx)
- [ALTER CERTIFICATE](https://msdn.microsoft.com/zh-cn/library/ms189511.aspx)
- [ALTER DATABASE ENCRYPTION KEY](https://msdn.microsoft.com/zh-cn/library/bb630389.aspx)
- [ALTER LOGIN](https://msdn.microsoft.com/zh-cn/library/ms189828.aspx)
- [ALTER MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms186937.aspx)
- [ALTER ROLE](https://msdn.microsoft.com/zh-cn/library/ms189775.aspx)
- [ALTER USER](https://msdn.microsoft.com/zh-cn/library/ms176060.aspx)
- [BACKUP CERTIFICATE](https://msdn.microsoft.com/zh-cn/library/ms178578.aspx)
- [CLOSE MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms188387.aspx)
- [CREATE CERTIFICATE](https://msdn.microsoft.com/zh-cn/library/ms187798.aspx)
- [CREATE DATABASE ENCRYPTION KEY](https://msdn.microsoft.com/zh-cn/library/bb677241.aspx)
- [CREATE LOGIN](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx)
- [CREATE MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx)
- [CREATE ROLE](https://msdn.microsoft.com/zh-cn/library/ms187936.aspx)
- [CREATE USER](https://msdn.microsoft.com/zh-cn/library/ms173463.aspx)
- [DROP CERTIFICATE](https://msdn.microsoft.com/zh-cn/library/ms179906.aspx)
- [DROP DATABASE ENCRYPTION KEY](https://msdn.microsoft.com/zh-cn/library/bb630256.aspx)
- [DROP LOGIN](https://msdn.microsoft.com/zh-cn/library/ms188012.aspx)
- [DROP MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms180071.aspx)
- [DROP ROLE](https://msdn.microsoft.com/zh-cn/library/ms174988.aspx)
- [DROP USER](https://msdn.microsoft.com/zh-cn/library/ms189438.aspx)
- [OPEN MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms174433.aspx)


## 后续步骤
有关更多 TSQL 示例，请参阅 [SQL 数据仓库开发概述][]。

<!--Image references-->

<!--Article references-->
[SQL 数据仓库开发概述]: /documentation/articles/sql-data-warehouse-overview-reference/

<!--MSDN references-->


<!--Other Web references-->

<!---HONumber=Mooncake_0307_2016-->