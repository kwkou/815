<properties
	pageTitle="标准异地复制或活动异地复制的安全性配置"
	description="本主题介绍管理 SQL 数据库的标准或活动异地复制方案时的安全注意事项。"
	services="sql-database"
	documentationCenter="na"
	authors="carlrabeler"
	manager="jhubbard"
	editor="monicar" />


<tags
	ms.service="sql-database"
	ms.date="02/01/2016"
	wacn.date="06/14/2016" />

# 标准异地复制或活动异地复制的安全性配置

## 概述
本主题介绍配置和控制[标准与活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)时所要满足的身份验证要求，以及设置用户对辅助数据库的访问权限所要执行的步骤。有关使用异地复制的详细信息，请参阅[在中断后恢复 Azure SQL 数据库](/documentation/articles/sql-database-disaster-recovery/)。

## 使用包含的用户
借助 [V12 版的 Azure SQL 数据库](/documentation/articles/sql-database-v12-whats-new/)，SQL 数据库现在可以支持包含的用户。不同于必须映射到 master 数据库中登录名的传统用户，包含的用户完全由数据库自身管理。这带来了两个好处。在异地复制方案中，用户无需进行任何附加配置就能继续连接到辅助数据库，因为数据库会管理用户。从登录的立场来看，此配置还有潜在的缩放性和性能优势。有关详细信息，请参阅[包含的数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)。

对于包含的用户，如果有多个数据库使用相同的登录名，则你必须为每个数据库单独管理该用户（例如密码更改），而不是在服务器级管理登录名。

>[AZURE.NOTE] 如果你想要单独更改主数据库和辅助数据库的读取访问权限，必须使用传统的登录名和用户。包含的用户无法独立于主数据库在辅助数据库上管理。

## 使用传统的登录名和用户
如果你要使用传统的登录名和用户（而不是包含的用户），必须采取额外的步骤以确保相同的登录名存在于辅助数据库服务器上。以下部分概述了相关的步骤和其他注意事项。

### 设置用户对联机辅助数据库的访问权限
要使辅助数据库能够用作只读数据库（联机辅助数据库）或者故障转移应用场合中的有效数据库副本，必须为辅助数据库指定相应的安全配置。

只有服务器管理员才能成功完成本主题后面所述的所有步骤。本主题的后面将会说明执行每个步骤所需的特定权限。

随时可以执行对活动异地复制联机辅助数据库进行用户访问的准备工作。此准备工作包括下面所述的三个步骤：

1. 确定有权访问主数据库的登录名。
2. 在源服务器上找到这些登录名的 SID。
3. 在目标服务器上生成与源服务器中的 SID 匹配的登录名。

#### 1\.确定有权访问主数据库的登录名：
该过程的第一个步骤就是确定必须在目标服务器上复制哪些登录名。可以使用一对 SELECT 语句来完成此操作，其中一个语句用于源服务器上的逻辑 master 数据库，另一个语句用于主数据库本身。

只有服务器管理员或 **LoginManager** 服务器角色的成员，才能使用以下 SELECT 语句确定源服务器上的登录名。

	SELECT [name], [sid] 
	FROM [sys].[sql_logins] 
	WHERE [type_desc] = 'SQL_Login'

只有 db\_owner 数据库角色的成员、dbo 用户或服务器管理员，才能确定主数据库中的所有数据库用户主体。

	SELECT [name], [sid]
	FROM [sys].[database_principals]
	WHERE [type_desc] = 'SQL_USER'

#### 2\.查找步骤 1 中确定的登录名的 SID：
通过将前一部分中所述的查询的输出进行比较以及对 SID 进行匹配，你可以将服务器登录名映射到数据库用户。包含数据库用户以及匹配的 SID 的登录名有权以该数据库用户主体的身份访问该数据库。

可以使用以下查询来查看所有用户主体及其在数据库中的 SID。只有 db\_owner 数据库角色的成员或服务器管理员才能运行此查询。

	SELECT [name], [sid]
	FROM [sys].[database_principals]
	WHERE [type_desc] = 'SQL_USER'

>[AZURE.NOTE] **INFORMATION\_SCHEMA** 和 **sys** 用户具有 *NULL* SID，**guest** SID 为 **0x00**。如果数据库创建者是服务器管理员而不是 **DbManager** 的成员，则 **dbo** SID 可能以 *0x01060000000001648000000000048454* 开头。

#### 3\.在目标服务器上生成登录名：
最后一个步骤是转到一个或多个目标服务器，并使用相应的 SID 生成登录名。基本语法如下。

	CREATE LOGIN [<login name>]
	WITH PASSWORD = <login password>,
	SID = <desired login SID>

>[AZURE.NOTE] 如果你要授予用户对辅助数据库而不是主数据库的访问权限，你可以使用以下语法更改主服务器上的用户登录名来实现此目的。
>
>ALTER LOGIN <login name> DISABLE
>
>DISABLE 不会更改密码，因此你始终可以根据需要启用该登录名。

## 设置终止连续复制关系后的用户访问权限
发生故障转移时，必须停止主数据库与活动辅助数据库之间的连续复制关系。有关此过程的信息，请参阅[在中断后恢复 Azure SQL 数据库](/documentation/articles/sql-database-disaster-recovery/)。

在使用标准异地复制时，用户无法访问脱机辅助数据库，因此，在终止连续复制关系后，必须更改用户帐户。

如果未在目标服务器上复制登录 SID，则终止后只允许服务器管理员访问辅助数据库。如果启动复制的用户为 DbManager，则除非已从源服务器复制其登录 SID，否则他们将无权访问辅助数据库。在复制过程的整个生命周期内，情况都是如此。

终止复制时，作为终止过程的一部分，[dbo] 用户主体将更改为与启动复制的用户的登录 SID 相匹配，并且其访问权限将会还原。这种情况对其他数据库用户不适用。

用于启动终止操作的用户帐户和关联的登录名应该在目标服务器与数据库中存在，以确保在完成终止后，该用户帐户能够访问辅助数据库。

有关故障转移后需要执行的步骤的详细信息，请参阅[确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize/)。

## 后续步骤
有关 SQL 数据库的异地复制和其他业务连续性功能的详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

<!---HONumber=Mooncake_0606_2016-->