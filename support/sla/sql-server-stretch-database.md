<properties
	pageTitle="适用于 SQL Server 伸展数据库的 SLA"
    description="我们保证，至少 99.9% 的时间内客户在其 SQL Server 伸展数据库和我们的 Internet 网关之间存在连接。"
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>

<tags ms.service="legal" ms.date="07/2016" wacn.date="07/2016" wacn.lang="cn"/>

> [AZURE.LANGUAGE]
- [中文](/support/legal/sql-server-stretch-database/)
- [English](/support/legal/sql-server-stretch-database-en/)

# 适用于 SQL Server 伸展数据库的 SLA

我们保证，至少 99.9% 的时间内客户在其 SQL Server 伸展数据库和我们的 Internet 网关之间存在连接。

## SLA 详细信息

其他定义

**“数据库”**是指 SQL Server 伸展数据库的一个实例。

SQL Server 伸展数据库服务的每月正常运行时间计算和服务级别

**“最大可用分钟数”**是指在给定 Azure 订阅的某个计费月份内给定数据库在 Azure 中部署的总分钟数。

**“停机时间”**是指客户在给定 Azure 订阅中部署的所有数据库中数据库不可用的累计总分钟数。如果在某一分钟内客户持续与给定数据库建立连接的所有尝试均失败，则这一分钟对于该数据库而言视为不可用。

SQL Server 伸展数据库服务的**“每月正常运行时间百分比”**计算方法为：在给定 Azure 订阅的某个计费月份内，最大可用分钟数减去停机时间再除以最大可用分钟数。“每月正常运行时间百分比”的计算公式如下：

	每月正常运行时间百分比 = (最大可用分钟数 - 停机时间) / 最大可用分钟数

客户在使用 Azure SQL 伸展数据库服务时适用以下服务级别和服务费抵扣：

每月正常运行时间百分比	|服务费抵扣
---|---
< 99.9%	|10%
< 99%	|25%
