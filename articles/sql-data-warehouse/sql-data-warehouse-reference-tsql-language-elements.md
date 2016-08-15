<properties
   pageTitle="SQL 数据仓库 Transact-SQL 语言元素 | Azure"
   description="用于 SQL 数据仓库的 Transact-SQL 语言元素的参考内容链接列表。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/02/2016"
   wacn.date="06/20/2016"/>

# 语言元素

## 核心元素

- [语法约定](https://msdn.microsoft.com/zh-cn/library/ms177563.aspx)
- [对象命名规则](https://msdn.microsoft.com/zh-cn/library/ms175874.aspx)
- [保留的关键字](https://msdn.microsoft.com/zh-cn/library/ms189822.aspx)
- [排序规则](https://msdn.microsoft.com/zh-cn/library/ff848763.aspx)
- [注释](https://msdn.microsoft.com/zh-cn/library/ms181627.aspx)
- [常量](https://msdn.microsoft.com/zh-cn/library/ms179899.aspx)
- [数据类型](https://msdn.microsoft.com/zh-cn/library/ms187752.aspx)
- [EXECUTE](https://msdn.microsoft.com/zh-cn/library/ms188332.aspx)
- [表达式](https://msdn.microsoft.com/zh-cn/library/ms190286.aspx)
- [KILL](https://msdn.microsoft.com/zh-cn/library/ms173730.aspx)
- [IDENTITY 属性解决方法](https://msdn.microsoft.com/zh-cn/library/ms186775.aspx)
- [PRINT](https://msdn.microsoft.com/zh-cn/library/ms176047.aspx)
- [USE](https://msdn.microsoft.com/zh-cn/library/ms188366.aspx)

## 批、流控制和变量

- [BEGIN...END](https://msdn.microsoft.com/zh-cn/library/ms190487.aspx)
- [BREAK](https://msdn.microsoft.com/zh-cn/library/ms181271.aspx)
- [DECLARE @local\_variable](https://msdn.microsoft.com/zh-cn/library/ms188927.aspx)
- [IF...ELSE](https://msdn.microsoft.com/zh-cn/library/ms182717.aspx)
- [RAISERROR](https://msdn.microsoft.com/zh-cn/library/ms178592.aspx)
- [SET@local\_variable](https://msdn.microsoft.com/zh-cn/library/ms189484.aspx)
- [THROW](https://msdn.microsoft.com/zh-cn/library/ee677615.aspx)
- [TRY...CATCH](https://msdn.microsoft.com/zh-cn/library/ms175976.aspx)
- [WHILE](https://msdn.microsoft.com/zh-cn/library/ms178642.aspx)

## 运算符
- [\+（加）](https://msdn.microsoft.com/zh-cn/library/ms178565.aspx)
- [\+（字符串串联）](https://msdn.microsoft.com/zh-cn/library/ms177561.aspx)
- [-（负）](https://msdn.microsoft.com/zh-cn/library/ms189480.aspx)
- [-（减）](https://msdn.microsoft.com/zh-cn/library/ms189518.aspx)
- [*（乘）](https://msdn.microsoft.com/zh-cn/library/ms176019.aspx)
- [/（除）](https://msdn.microsoft.com/zh-cn/library/ms175009.aspx)
- [取模](https://msdn.microsoft.com/zh-cn/library/ms190279.aspx)

## 要匹配的通配符
- [=（等于）](https://msdn.microsoft.com/zh-cn/library/ms175118.aspx)
- [>（大于）](https://msdn.microsoft.com/zh-cn/library/ms178590.aspx)
- [<（小于）](https://msdn.microsoft.com/zh-cn/library/ms179873.aspx)
- [>=（大于或等于）](https://msdn.microsoft.com/zh-cn/library/ms181567.aspx)
- [<=（小于或等于）](https://msdn.microsoft.com/zh-cn/library/ms174978.aspx)
- [<>（不等于）](https://msdn.microsoft.com/zh-cn/library/ms176020.aspx)
- [!=（不等于）](https://msdn.microsoft.com/zh-cn/library/ms190296.aspx)
- [AND](https://msdn.microsoft.com/zh-cn/library/ms188372.aspx)
- [BETWEEN](https://msdn.microsoft.com/zh-cn/library/ms187922.aspx)
- [EXISTS](https://msdn.microsoft.com/zh-cn/library/ms188336.aspx)
- [IN](https://msdn.microsoft.com/zh-cn/library/ms177682.aspx)
- [IS [NOT] NULL](https://msdn.microsoft.com/zh-cn/library/ms188795.aspx)
- [LIKE](https://msdn.microsoft.com/zh-cn/library/ms179859.aspx)
- [NOT](https://msdn.microsoft.com/zh-cn/library/ms189455.aspx)
- [或者](https://msdn.microsoft.com/zh-cn/library/ms188361.aspx)

### 位运算符

- [&（位与）](https://msdn.microsoft.com/zh-cn/library/ms174965.aspx)
- [|（位或）](https://msdn.microsoft.com/zh-cn/library/ms186714.aspx)
- [^（位异或）](https://msdn.microsoft.com/zh-cn/library/ms190277.aspx)
- [~（位非）](https://msdn.microsoft.com/zh-cn/library/ms173468.aspx)
- [^=（位异或等于）](https://msdn.microsoft.com/zh-cn/library/cc627413.aspx)
- [|=（位或等于）](https://msdn.microsoft.com/zh-cn/library/cc627409.aspx)
- [&=（位与等于）](https://msdn.microsoft.com/zh-cn/library/cc627427.aspx)

## 函数

- [@@DATEFIRST](https://msdn.microsoft.com/zh-cn/library/ms187766.aspx)
- [@@ERROR](https://msdn.microsoft.com/zh-cn/library/ms188790.aspx)
- [@@LANGUAGE](https://msdn.microsoft.com/zh-cn/library/ms177557.aspx)
- [@@SPID](https://msdn.microsoft.com/zh-cn/library/ms189535.aspx)
- [@@TRANCOUNT](https://msdn.microsoft.com/zh-cn/library/ms187967.aspx)
- [@@VERSION](https://msdn.microsoft.com/zh-cn/library/ms177512.aspx)
- [ABS](https://msdn.microsoft.com/zh-cn/library/ms189800.aspx)
- [ACOS](https://msdn.microsoft.com/zh-cn/library/ms178627.aspx)
- [ASCII](https://msdn.microsoft.com/zh-cn/library/ms177545.aspx)
- [ASIN](https://msdn.microsoft.com/zh-cn/library/ms181581.aspx)
- [ATAN](https://msdn.microsoft.com/zh-cn/library/ms181746.aspx)
- [ATN2](https://msdn.microsoft.com/zh-cn/library/ms173854.aspx)
- [CASE](https://msdn.microsoft.com/zh-cn/library/ms181765.aspx)
- [CAST 和 CONVERT](https://msdn.microsoft.com/zh-cn/library/ms187928.aspx)
- [CEILING](https://msdn.microsoft.com/zh-cn/library/ms189818.aspx)
- [CHAR](https://msdn.microsoft.com/zh-cn/library/ms187323.aspx)
- [CHARINDEX](https://msdn.microsoft.com/zh-cn/library/ms186323.aspx)
- [COALESCE](https://msdn.microsoft.com/zh-cn/library/ms190349.aspx)
- [COL\_NAME](https://msdn.microsoft.com/zh-cn/library/ms174974.aspx)
- [COLLATIONPROPERTY](https://msdn.microsoft.com/zh-cn/library/ms190305.aspx)
- [CONCAT](https://msdn.microsoft.com/zh-cn/library/hh231515.aspx)
- [COS](https://msdn.microsoft.com/zh-cn/library/ms188919.aspx)
- [COT](https://msdn.microsoft.com/zh-cn/library/ms188921.aspx)
- [COUNT](https://msdn.microsoft.com/zh-cn/library/ms175997.aspx)
- [COUNT\_BIG](https://msdn.microsoft.com/zh-cn/library/ms190317.aspx)
- [CURRENT\_TIMESTAMP](https://msdn.microsoft.com/zh-cn/library/ms188751.aspx)
- [CURRENT\_USER](https://msdn.microsoft.com/zh-cn/library/ms176050.aspx)
- [DATABASEPROPERTYEX](https://msdn.microsoft.com/zh-cn/library/ms186823.aspx)
- [DATALENGTH](https://msdn.microsoft.com/zh-cn/library/ms173486.aspx)
- [DATEADD](https://msdn.microsoft.com/zh-cn/library/ms186819.aspx)
- [DATEDIFF](https://msdn.microsoft.com/zh-cn/library/ms189794.aspx)
- [DATEFROMPARTS](https://msdn.microsoft.com/zh-cn/library/hh213228.aspx)
- [DATENAME](https://msdn.microsoft.com/zh-cn/library/ms174395.aspx)
- [DATEPART](https://msdn.microsoft.com/zh-cn/library/ms174420.aspx)
- [DATETIME2FROMPARTS](https://msdn.microsoft.com/zh-cn/library/hh213312.aspx)
- [DATETIMEFROMPARTS](https://msdn.microsoft.com/zh-cn/library/hh213233.aspx)
- [DATETIMEOFFSETFROMPARTS](https://msdn.microsoft.com/zh-cn/library/hh231077.aspx)
- [DAY](https://msdn.microsoft.com/zh-cn/library/ms176052.aspx)
- [DB\_ID](https://msdn.microsoft.com/zh-cn/library/ms186274.aspx)
- [DB\_NAME](https://msdn.microsoft.com/zh-cn/library/ms189753.aspx)
- [DEGREES](https://msdn.microsoft.com/zh-cn/library/ms178566.aspx)
- [DENSE\_RANK](https://msdn.microsoft.com/zh-cn/library/ms173825.aspx)
- [DIFFERENCE](https://msdn.microsoft.com/zh-cn/library/ms188753.aspx)
- [EOMONTH](https://msdn.microsoft.com/zh-cn/library/hh213020.aspx)
- [ERROR\_MESSAGE](https://msdn.microsoft.com/zh-cn/library/ms190358.aspx)
- [ERROR\_NUMBER](https://msdn.microsoft.com/zh-cn/library/ms175069.aspx)
- [ERROR\_PROCEDURE](https://msdn.microsoft.com/zh-cn/library/ms188398.aspx)
- [ERROR\_SEVERITY](https://msdn.microsoft.com/zh-cn/library/ms178567.aspx)
- [ERROR\_STATE](https://msdn.microsoft.com/zh-cn/library/ms180031.aspx)
- [EXP](https://msdn.microsoft.com/zh-cn/library/ms179857.aspx)
- [FLOOR](https://msdn.microsoft.com/zh-cn/library/ms178531.aspx)
- [GETDATE](https://msdn.microsoft.com/zh-cn/library/ms188383.aspx)
- [GETUTCDATE](https://msdn.microsoft.com/zh-cn/library/ms178635.aspx)
- [HAS\_DBACCESS](https://msdn.microsoft.com/zh-cn/library/ms187718.aspx)
- [HASHBYTES](https://msdn.microsoft.com/zh-cn/library/ms174415.aspx)
- [INDEXPROPERTY](https://msdn.microsoft.com/zh-cn/library/ms187729.aspx)
- [ISDATE](https://msdn.microsoft.com/zh-cn/library/ms187347.aspx)
- [ISNULL](https://msdn.microsoft.com/zh-cn/library/ms184325.aspx)
- [ISNUMERIC](https://msdn.microsoft.com/zh-cn/library/ms186272.aspx)
- [LAG](https://msdn.microsoft.com/zh-cn/library/hh231256.aspx)
- [LEAD](https://msdn.microsoft.com/zh-cn/library/hh213125.aspx)
- [LEFT](https://msdn.microsoft.com/zh-cn/library/ms177601.aspx)
- [LEN](https://msdn.microsoft.com/zh-cn/library/ms190329.aspx)
- [LOG](https://msdn.microsoft.com/zh-cn/library/ms190319.aspx)
- [LOG10](https://msdn.microsoft.com/zh-cn/library/ms175121.aspx)
- [LOWER](https://msdn.microsoft.com/zh-cn/library/ms174400.aspx)
- [LTRIM](https://msdn.microsoft.com/zh-cn/library/ms177827.aspx)
- [MAX](https://msdn.microsoft.com/zh-cn/library/ms187751.aspx)
- [MIN](https://msdn.microsoft.com/zh-cn/library/ms179916.aspx)
- [MONTH](https://msdn.microsoft.com/zh-cn/library/ms187813.aspx)
- [NCHAR](https://msdn.microsoft.com/zh-cn/library/ms182673.aspx)
- [NTILE](https://msdn.microsoft.com/zh-cn/library/ms175126.aspx)
- [NULLIF](https://msdn.microsoft.com/zh-cn/library/ms177562.aspx)
- [OBJECT\_ID](https://msdn.microsoft.com/zh-cn/library/ms190328.aspx)
- [OBJECT\_NAME](https://msdn.microsoft.com/zh-cn/library/ms186301.aspx)
- [OBJECTPROPERTY](https://msdn.microsoft.com/zh-cn/library/ms176105.aspx)
- [OIBJECTPROPERTYEX](https://msdn.microsoft.com/zh-cn/library/ms188390.aspx)
- [ODBCS 标量函数](https://msdn.microsoft.com/zh-cn/library/bb630290.aspx)
- [OVER 子句](https://msdn.microsoft.com/zh-cn/library/ms189461.aspx)
- [PARSENAME](https://msdn.microsoft.com/zh-cn/library/ms188006.aspx)
- [PATINDEX](https://msdn.microsoft.com/zh-cn/library/ms188395.aspx)
- [PERCENTILE\_CONT](https://msdn.microsoft.com/zh-cn/library/hh231473.aspx)
- [PERCENTILE\_DISC](https://msdn.microsoft.com/zh-cn/library/hh231327.aspx)
- [PI](https://msdn.microsoft.com/zh-cn/library/ms189512.aspx)
- [POWER](https://msdn.microsoft.com/zh-cn/library/ms174276.aspx)
- [QUOTENAME](https://msdn.microsoft.com/zh-cn/library/ms176114.aspx)
- [RADIANS](https://msdn.microsoft.com/zh-cn/library/ms189742.aspx)
- [RANK](https://msdn.microsoft.com/zh-cn/library/ms176102.aspx)
- [REPLACE](https://msdn.microsoft.com/zh-cn/library/ms186862.aspx)
- [REPLICATE](https://msdn.microsoft.com/zh-cn/library/ms174383.aspx)
- [REVERSE](https://msdn.microsoft.com/zh-cn/library/ms180040.aspx)
- [RIGHT](https://msdn.microsoft.com/zh-cn/library/ms177532.aspx)
- [ROUND](https://msdn.microsoft.com/zh-cn/library/ms175003.aspx)
- [ROW\_NUMBER](https://msdn.microsoft.com/zh-cn/library/ms186734.aspx)
- [RTRIM](https://msdn.microsoft.com/zh-cn/library/ms178660.aspx)
- [SCHEMA\_ID](https://msdn.microsoft.com/zh-cn/library/ms188797.aspx)
- [SCHEMA\_NAME](https://msdn.microsoft.com/zh-cn/library/ms175068.aspx)
- [SERVERPROPERTY](https://msdn.microsoft.com/zh-cn/library/ms174396.aspx)
- [SESSION\_USER](https://msdn.microsoft.com/zh-cn/library/ms177587.aspx)
- [SIGN](https://msdn.microsoft.com/zh-cn/library/ms188420.aspx)
- [SIN](https://msdn.microsoft.com/zh-cn/library/ms188377.aspx)
- [SMALLDATETIMEFROMPARTS](https://msdn.microsoft.com/zh-cn/library/hh213396.aspx)
- [SOUNDEX](https://msdn.microsoft.com/zh-cn/library/ms187384.aspx)
- [SPACE](https://msdn.microsoft.com/zh-cn/library/ms187950.aspx)
- [SQL\_VARIANT\_PROPERTY](https://msdn.microsoft.com/zh-cn/library/ms178550.aspx)
- [SQRT](https://msdn.microsoft.com/zh-cn/library/ms176108.aspx)
- [SQUARE](https://msdn.microsoft.com/zh-cn/library/ms173569.aspx)
- [STATS\_DATE](https://msdn.microsoft.com/zh-cn/library/ms190330.aspx)
- [STDEV](https://msdn.microsoft.com/zh-cn/library/ms190474.aspx)
- [STDEVP](https://msdn.microsoft.com/zh-cn/library/ms176080.aspx)
- [STR](https://msdn.microsoft.com/zh-cn/library/ms189527.aspx)
- [STUFF](https://msdn.microsoft.com/zh-cn/library/ms188043.aspx)
- [SUBSTRING](https://msdn.microsoft.com/zh-cn/library/ms187748.aspx)
- [SUM](https://msdn.microsoft.com/zh-cn/library/ms187810.aspx)
- [SUSER\_SNAME](https://msdn.microsoft.com/zh-cn/library/ms174427.aspx)
- [SWITCHOFFSET](https://msdn.microsoft.com/zh-cn/library/bb677244.aspx)
- [SYSDATETIME](https://msdn.microsoft.com/zh-cn/library/bb630353.aspx)
- [SYSDATETIMEOFFSET](https://msdn.microsoft.com/zh-cn/library/bb677334.aspx)
- [SYSTEM\_USER](https://msdn.microsoft.com/zh-cn/library/ms179930.aspx)
- [SYSUTCDATETIME](https://msdn.microsoft.com/zh-cn/library/bb630387.aspx)
- [TAN](https://msdn.microsoft.com/zh-cn/library/ms190338.aspx)
- [TERTIARY\_WEIGHTS](https://msdn.microsoft.com/zh-cn/library/ms186881.aspx)
- [TIMEFROMPARTS](https://msdn.microsoft.com/zh-cn/library/hh213398.aspx)
- [TODATETIMEOFFSET](https://msdn.microsoft.com/zh-cn/library/bb630335.aspx)
- [TYPE\_ID](https://msdn.microsoft.com/zh-cn/library/ms181628.aspx)
- [TYPE\_NAME](https://msdn.microsoft.com/zh-cn/library/ms189750.aspx)
- [TYPEPROPERTY](https://msdn.microsoft.com/zh-cn/library/ms188419.aspx)
- [UNICODE](https://msdn.microsoft.com/zh-cn/library/ms180059.aspx)
- [UPPER](https://msdn.microsoft.com/zh-cn/library/ms180055.aspx)
- [USER](https://msdn.microsoft.com/zh-cn/library/ms186738.aspx)
- [USER\_NAME](https://msdn.microsoft.com/zh-cn/library/ms188014.aspx)
- [VAR](https://msdn.microsoft.com/zh-cn/library/ms186290.aspx)
- [VARP](https://msdn.microsoft.com/zh-cn/library/ms188735.aspx)
- [YEAR](https://msdn.microsoft.com/zh-cn/library/ms186313.aspx)
- [XACT\_STATE](https://msdn.microsoft.com/zh-cn/library/ms189797.aspx)

## 事务

- [事务](https://msdn.microsoft.com/zh-cn/library/mt204031.aspx)

## 诊断会话

- [CREATE DIAGNOSTICS SESSION](https://msdn.microsoft.com/zh-cn/library/mt204029.aspx)

## 过程

- [sp\_addrolemember](https://msdn.microsoft.com/zh-cn/library/ms187750.aspx)
- [sp\_columns](https://msdn.microsoft.com/zh-cn/library/ms176077.aspx)
- [sp\_configure](https://msdn.microsoft.com/zh-cn/library/ms188787.aspx)
- [sp\_datatype\_info\_90](https://msdn.microsoft.com/zh-cn/library/mt204014.aspx)
- [sp\_droprolemember](https://msdn.microsoft.com/zh-cn/library/ms188369.aspx)
- [sp\_execute](https://msdn.microsoft.com/zh-cn/library/ff848746.aspx)
- [sp\_executesql](https://msdn.microsoft.com/zh-cn/library/ms188001.aspx)
- [sp\_fkeys](https://msdn.microsoft.com/zh-cn/library/ms175090.aspx)
- [sp\_pdw\_add\_network\_credentials](https://msdn.microsoft.com/zh-cn/library/mt204011.aspx)
- [sp\_pdw\_database\_encryption](https://msdn.microsoft.com/zh-cn/library/mt219360.aspx)
- [sp\_pdw\_database\_encryption\_regenerate\_system\_keys](https://msdn.microsoft.com/zh-cn/library/mt204033.aspx)
- [sp\_pdw\_log\_user\_data\_masking](https://msdn.microsoft.com/zh-cn/library/mt204023.aspx)
- [sp\_pdw\_remove\_network\_credentials](https://msdn.microsoft.com/zh-cn/library/mt204038.aspx)
- [sp\_pkeys](https://msdn.microsoft.com/zh-cn/library/ms189813.aspx)
- [sp\_prepare](https://msdn.microsoft.com/zh-cn/library/ff848808.aspx)
- [sp\_special\_columns\_100](https://msdn.microsoft.com/zh-cn/library/mt204025.aspx)
- [sp\_sproc\_columns](https://msdn.microsoft.com/zh-cn/library/ms182705.aspx)
- [sp\_statistics](https://msdn.microsoft.com/zh-cn/library/ms173842.aspx)
- [sp\_tables](https://msdn.microsoft.com/zh-cn/library/ms186250.aspx)
- [sp\_unprepare](https://msdn.microsoft.com/zh-cn/library/ff848735.aspx)



## SET 语句

- [SET ANSI\_DEFAULTS](https://msdn.microsoft.com/zh-cn/library/ms188340.aspx)
- [SET ANSI\_NULL\_DFLT\_OFF](https://msdn.microsoft.com/zh-cn/library/ms187356.aspx)
- [SET ANSI\_NULL\_DFLT\_ON](https://msdn.microsoft.com/zh-cn/library/ms187375.aspx)
- [SET ANSI\_NULLS](https://msdn.microsoft.com/zh-cn/library/ms188048.aspx)
- [SET ANSI\_PADDING](https://msdn.microsoft.com/zh-cn/library/ms187403.aspx)
- [SET ANSI\_WARNINGS](https://msdn.microsoft.com/zh-cn/library/ms190368.aspx)
- [SET ARITHABORT](https://msdn.microsoft.com/zh-cn/library/ms190306.aspx)
- [SET ARITHIGNORE](https://msdn.microsoft.com/zh-cn/library/ms184341.aspx)
- [SET CONCAT\_NULL\_YIELDS\_NULL](https://msdn.microsoft.com/zh-cn/library/ms176056.aspx)
- [SET DATEFIRST](https://msdn.microsoft.com/zh-cn/library/ms181598.aspx)
- [SET DATEFORMAT](https://msdn.microsoft.com/zh-cn/library/ms189491.aspx)
- [SET FMTONLY](https://msdn.microsoft.com/zh-cn/library/ms173839.aspx)
- [SET IMPLICIT\_TRANSACITONS](https://msdn.microsoft.com/zh-cn/library/ms187807.aspx)
- [SET LOCK\_TIMEOUT](https://msdn.microsoft.com/zh-cn/library/ms189470.aspx)
- [SET NUMBERIC\_ROUNDABORT](https://msdn.microsoft.com/zh-cn/library/ms188791.aspx)
- [SET QUOTED\_IDENTIFIER](https://msdn.microsoft.com/zh-cn/library/ms174393.aspx)
- [SET ROWCOUNT](https://msdn.microsoft.com/zh-cn/library/ms188774.aspx)
- [SET TEXTSIZE](https://msdn.microsoft.com/zh-cn/library/ms186238.aspx)
- [SET TRANSACTION ISOLATION LEVEL](https://msdn.microsoft.com/zh-cn/library/ms173763.aspx)
- [SET XACT\_ABORT](https://msdn.microsoft.com/zh-cn/library/ms188792.aspx)


## 后续步骤
有关更多参考信息，请参阅 [SQL 数据仓库参考概述][]。

<!--Image references-->

<!--Article references-->
[SQL Data Warehouse development overview]: /documentation/articles/sql-data-warehouse-overview-reference/

<!--MSDN references-->

<!---HONumber=Mooncake_0307_2016-->