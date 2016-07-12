<properties
	pageTitle="从 Azure SQL 数据库备份中还原单个表 | Azure"
	description="了解如何从 Azure SQL 数据库备份中还原单个表。"
	services="sql-database"
	documentationCenter=""
	authors="dalechen"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="03/11/2016"
	wacn.date="04/06/2016"/>


# 如何从 Azure SQL 数据库备份中还原单个表

你可能会遇到这种情况，不小心修改了 SQL 数据库中的某些数据，而现在你希望恢复单个受影响的表。本文介绍如何基于所选的性能层，从 Azure SQL 数据库自动执行的一个备份中还原数据库中的单个表。

## 准备步骤：重命名表，并还原数据库的一个副本
1. 确定 Azure SQL 数据库中你要替换为还原的副本的表。使用 Microsoft SQL Management Studio 重命名此表。例如，将此表重命名为 &lt;table name&gt;\_old。

	**注意** 为了避免受到阻止，请确保你要重命名的表没有任何正在运行的活动。如果你遇到问题，请确保在维护时段内执行此过程。

2. 将数据库的一个备份还原到你想要恢复到的一个时间点。要实现此操作，请参阅[从用户错误中恢复 Azure SQL 数据库](/documentation/articles/sql-database-user-error-recovery/)中的步骤。

	**注意**：
	- 还原的数据库的名称的格式为 DBName+TimeStamp；例如，**Adventureworks2012\_2016-01-01T22-12Z**。此步骤不会覆盖服务器上现有的数据库名称。这是一项安全措施，目的是要让用户在删除其当前数据库之前确认还原的数据库，然后重命名此还原的数据库供生产之用。
	- 服务根据不同的性能层使用不同的备份保留期指标来自动备份从基本到高级的所有性能层：

| 数据库还原 | 基本层 | 标准层 | 高级层 |
| :-- | :-- | :-- | :-- |
| 时间点还原 | 7 天内的任何还原点|14 天内的任何还原点| 35 天内的任何还原点|

## 使用 SQL 数据库迁移工具从还原的数据库中复制表
1. 下载并安装 [SQL 数据库迁移向导](https://sqlazuremw.codeplex.com)。

2. 打开 SQL 数据库迁移向导，在“选择处理”页上选择“分析/迁移数据库”，然后单击“下一步”。
![SQL 数据库迁移向导 - 选择处理](./media/sql-database-cloud-migrate-restore-single-table-azure-backup/1.png)
3. 在“连接到服务器”对话框中应用以下设置：
 - **服务器名称**：你的 SQL Azure 实例
 - **身份验证**：**SQL Server 身份验证**。输入你的登录凭据。
 - **数据库**：**Master 数据库（列出所有数据库）**。
 - **注意** 默认情况下此向导将保存你的登录信息。如果你不想保存，请选择“忘记登录信息”。
![SQL 数据库迁移向导 - 选择源 - 步骤 1](./media/sql-database-cloud-migrate-restore-single-table-azure-backup/2.png)
4. 在“选择源”对话框中，选择“准备步骤”部分中的还原的数据库名称作为你的源，然后单击“下一步”。

	![SQL 数据库迁移向导 - 选择源 - 步骤 2](./media/sql-database-cloud-migrate-restore-single-table-azure-backup/3.png)

5. 在“选择对象”对话框中，选择“选择特定的数据库对象”选项，然后选择你想要迁移到目标服务器的表。
![SQL 数据库迁移向导 - 选择对象](./media/sql-database-cloud-migrate-restore-single-table-azure-backup/4.png)

6. 在“脚本向导摘要”页面中，当询问你是否准备好生成 SQL 脚本时，单击“是”。你还可以选择保存此 TSQL 脚本供以后使用。
![SQL 数据库迁移向导 - 脚本向导摘要](./media/sql-database-cloud-migrate-restore-single-table-azure-backup/5.png)

7. 在“结果摘要”页面上，单击“下一步”。

	![SQL 数据库迁移向导 - 结果摘要](./media/sql-database-cloud-migrate-restore-single-table-azure-backup/6.png)

8. 在“设置目标服务器连接”页面上，单击“连接到服务器”，然后输入详细信息，如下所示：
	- **服务器名称**：目标服务器实例
	- **身份验证**：**SQL Server 身份验证**。输入你的登录凭据。
	- **数据库**：**Master 数据库（列出所有数据库）**。此选项可列出目标服务器上的所有数据库。

	![SQL 数据库迁移向导 - 设置目标服务器连接](./media/sql-database-cloud-migrate-restore-single-table-azure-backup/7.png)

9. 单击“连接”，选择你要将此表移动到的目标数据库，然后单击“下一步”。将完成运行之前生成的脚本，你会看到复制到目标数据库的最近移动的表。

## 验证步骤
1. 查询并测试最近复制的表，以确保数据完好无损。确认后，你可以删除“准备步骤”部分中的重命名表（例如 &lt;table name&gt;\_old）。

<!---HONumber=Mooncake_0328_2016-->
