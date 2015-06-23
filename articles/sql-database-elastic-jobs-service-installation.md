<properties
	pageTitle="安装弹性数据库作业"
	description="演练如何安装弹性作业功能。"
	services="sql-database"
	documentationCenter=""
	manager="jhubbard"
	authors="sidneyh"
	editor=""/>

<tags ms.service="sql-database" ms.date="04/29/2015" wacn.date="06/23/2015"/>

# 安装弹性数据库作业服务

[弹性数据库池（预览版）](sql-database-elastic-pool-portal)提供可预测的模型用于部署大量数据库。创建弹性数据库池后，你可以使用**弹性数据库作业**针对弹性数据库池中的每个数据库执行管理任务。例如，可以部署新的架构，包括针对每个数据库设置 RLS 策略，以便只有具有适当凭据的人员才能查看敏感数据。下面介绍了如何安装**弹性数据库作业**。

**估计完成时间：**10 分钟。

## 先决条件
* Azure 订阅。若要获取试用版，请参阅[试用](/pricing/1rmb-trial/)。
* 一个弹性数据库池。请参阅[创建 Azure SQL Database 弹性池（预览版）](sql-database-elastic-pool-portal)。

## 安装服务组件
首先，请转到 [Azure 门户](https://manage.windowsazure.cn)。


1. 在弹性数据库池的仪表板视图中，单击“创建作业”。
2. 如果是首次创建作业，必须通过单击“预览版条款”安装**弹性数据库作业**。
3. 单击相应的复选框接受条款。
4. 在“安装服务”视图中，单击“作业凭据”。

	![安装服务][1]

5. 键入数据库管理员的用户名和密码。如果弹性数据库池中每个数据库已存在一个用于执行脚本的公用用户，请考虑使用此用户，这样就无需在每个数据库中添加新用户来执行脚本。安装过程中会创建新的 Azure SQL Database 服务器。在新服务器中，创建了一个名为控制数据库的新数据库，用于包含弹性数据库作业的元数据。此处创建的用户名和密码用于两个目的：(1) 登录到控制数据库；(2) 每当你运行作业来执行脚本时，会将此用户名和密码用作凭据来登录到弹性池中的每个数据库。

	![创建用户名和密码][2]
6. 单击“确定”按钮。几分钟后，将在新的[资源组](resource-group-portal)中为你创建组件。新资源组已固定到开始面板，如下所示。弹性数据库作业（云服务、SQL Database、 Service Bus 和存储空间）都在该组中创建。

	![开始面板中的资源组][3]


7. 如果你在安装弹性数据库作业时尝试创建或管理某个作业，则在提供**凭据**时，你将看到以下消息。

	![部署仍在进行][4]

8. 如果安装出于某种原因而失败，请删除资源组。请参阅[如何卸载弹性数据库作业组件](sql-database-elastic-jobs-uninstall)。


## 后续步骤

如果在安装弹性数据库作业时提供的、有执行脚本的相应权限的新凭据尚不存在于弹性数据库池中的每个数据库内，则必须在每个数据库中创建该凭据。请参阅[如何将用户添加到弹性数据库池](sql-database-elastic-jobs-add-logins-to-dbs)。
若要了解作业创建过程，请参阅[创建和管理弹性数据库作业](sql-database-elastic-jobs-create-and-manage)。

<!--Image references-->

[1]: ./media/sql-database-elastic-jobs-service-installation/screen-1.png
[2]: ./media/sql-database-elastic-jobs-service-installation/credentials.png
[3]: ./media/sql-database-elastic-jobs-service-installation/start-board.png
[4]: ./media/sql-database-elastic-jobs-service-installation/incomplete.png

<!---HONumber=61-->