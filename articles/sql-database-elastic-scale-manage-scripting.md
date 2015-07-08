<properties 
	pageTitle="编写弹性缩放脚本" 
	description="使用 PowerShell 和 Azure Automation 服务 Runbook 编写弹性缩放脚本" 
	services="sql-database" 
	documentationCenter="" 
	manager="stuartozer" 
	authors="Joseidz" 
	editor=""/>

<tags
   ms.service="sql-database"
   ms.date="02/03/2015"
   wacn.date="05/25/2015"/>


# 使用脚本管理弹性缩放


## Azure Automation 服务 

[Azure Automation](/documentation/services/automation) 为 Azure 平台提供一个功能强大、极有必要的 PowerShell 工作流执行服务。现在，你可以将常见 Azure 门户体验内难以执行的维护任务自动化。你只需创作一个 PowerShell 工作流（在 Azure Automation 中称为 **Runbook**），将它上载到云，然后计划执行 Runbook 的时间。本文档针对一些分片弹性示例提供 Azure Automation 的端到端设置。有关详细信息，请参阅[预览版通告](http://blogs.technet.com/b/in_the_cloud/archive/2014/04/15/announcing-the-microsoft-azure-automation-preview.aspx)。

在本示例中，Azure Automation 用架计划和工作负载执行框架。可将 Azure Automation 视为[云中的 SQL 代理](http://azure.microsoft.com/blog/2014/06/26/azure-automation-your-sql-agent-in-the-cloud)。 

除了本文档外，还可以参考以下附加资源：

* [Azure Automation 入门](/documentation/articles/automation-create-runbook-from-samples)
* [分步指南：Windows Azure Automation 预览版新增功能入门](http://blogs.technet.com/b/keithmayer/archive/2014/04/04/step-by-step-getting-started-with-windows-azure-automation.aspx)
* [Windows Azure Automation](http://blogs.technet.com/b/cbernier/archive/2014/04/08/microsoft-azure-automation.aspx)
* 在 [Automation 论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazurezhchs)咨询具体的 Azure Automation 问题。  


## 先决条件

[熟悉](/documentation/articles/automation-create-runbook-from-samples) Windows Azure Automation 预览版服务。


## 分片弹性 PowerShell 文件

以下 PowerShell 文件集包含使用 Azure Automation 完成横向和纵向缩放方案的基本命令。 

这些示例演示如何使用 PowerShell 示例模块来执行基本分片弹性任务。结合 Windows Azure Automation 服务和相应的 Azure Automation Runbook，你可以创建自动化和计划的作业，以设置新的分片和/或根据一组规则更改特定分片的性能级别。 

**SetupShardedEnvironment.ps1**：此 PowerShell Runbook 会执行分片环境的一次性设置，在环境中提供分片映射管理器和范围分片映射。 

**ProvisionByDate.ps1**：在未来某日的工作负载之前预先设置新的数据库。将会创建数据库且根据日期戳 (YYYYMMDD) 命名，并在分片映射管理器中注册为范围 [YYYYMMDD, YYYYMMDD + 1D)。 

**ProvisionBySize.ps1**：在当前数据库的容量不足时设置新的数据库。 

**ReduceServiceTier.ps1**：循环访问提供的分片映射中的分片，并确定每个分片是否应该降低性能级别。有两个条件决定了分片是否为候选项：1) 分片的当前服务层，2) 数据库的新旧程度。  

**ShardManagement.psm1**：提供一组与分片映射管理员交互的方法。 

**SqlDatabaseHelpers.psm1**：提供一组与 Azure SQL 数据库交互的方法。 

**ShardElasticity.psm1**：提供一组执行横向缩放和纵向缩放的方法。 

**ShardElasticityModule.psd1**：提供一组与弹性缩放和 Azure SQL DB 交互的方法。 

## 成本

请注意，执行 PowerShell 示例脚本将会导致创建数据库，因而对订阅所有者引发实际成本。基础 Azure SQL DB 的[费率](/home/features/sql-database/#price)与其他 Azure SQL DB 数据库相同。从 11 月 1 日开始的成本是： 

* SetupShardedEnvironment Runbook 会在基本数据库上创建分片映射管理器（$0.0069/小时），并在基本数据库上设置第一个分片（$0.0069/小时）。 

* ProvisionBySize 和 ProvisionByDate Runbook 都会设置标准 S0 数据库（$0.0208/小时）。为了抵销这些成本，如果搭配 ReduceServiceTier Runbook 执行，则新设置的数据库的服务层在经过一天之后，将会从标准 S0（$0.0208/小时）降低到基本版（$0.0069/小时） 

最后，在所提供示例的范围内，使用 [Azure Automation](/documentation/services/automation) 当前不会对订阅拥有者引发任何费用。有关详细信息，请参阅 [Azure Automation 定价页](/home/features/automation/#price)。 

## 加载 Runbook 

1. 下载 **ShardElasticity.zip** 文件，并解压缩其内容。
2. [使用 NuGet 添加对弹性缩放二进制文件的引用](/documentation/articles/sql-database-elastic-scale-add-references-visual-studio)
3. 查找弹性缩放客户端库 (**Microsoft.Azure.SqlDatabase.ElasticScale.Client.dll**)。
4. 将 DLL 放入 ShardElasticityModule 文件夹，并压缩该文件夹。 
3. 在 Azure Automation 帐户中，上载 ShardElasticityModule.zip 文件作为**资产**。 
4. 在 Azure Automation 中，创建名为  *ElasticScaleCredential* 的**资产凭据**，其中包含 Azure SQL 数据库 服务器的用户名和密码。 
5. 创建名为  *SqlServerName* 的**资产变量**，作为完整的 Azure SQL 数据库 服务器名称。 
5. 上载 **SetupShardedEnvironment.ps1**、**ProvisionBySize.ps1**、**ProvisionByDate.ps1** 和 **ProvisionByDate.ps1** 作为 Runbook。 
6. 以一次性操作形式测试 **SetupShardedEnvironment.ps1** Runbook，以设置分片环境。 
7. 发布剩余的一个或多个 Runbook，并将 Runbook 链接到计划。 
8. 通过"作业"选项卡观察 Runbook 的输出。 

如果"快速示例说明"未成功，请参阅下面的"详细示例说明"。  

## 使用 Runbook

1. 创作并打包 PowerShell 模块 
2. 创建 Windows Azure Automation 帐户 
3. 将 PowerShell 模块上载到 Azure Automation 作为资产 
4. 创建 Azure Automation 凭据和变量资产 
5. 将 PowerShell Runbook 上载到 Azure Automation 
6. 设置分片环境 
7. 测试 Automation Runbook 
8. 发布 Runbook 
9. 计划 Runbook 


## 创作并打包 PowerShell 模块 

第一个步骤是创建 PowerShell 模块，以引用弹性缩放程序集并打包此模块，以便能够将它上载到 Azure Automation 服务作为资产。 

1. 下载"ShardElasticity.zip"文件。
2. 解压缩所有的内容。

    ![Extract all][1]
3. 获取弹性缩放客户端 DLL（即 Microsoft.Azure.SqlDatabase.ElasticScale.Client.dll），并将以下文件复制到你在步骤 1 中下载的本地"ShardElasticityModule"文件夹可以通过两种方法实现此目的：1) 通过弹性缩放 NuGet 包[链接]下载 DLL，或者 2) 从弹性缩放初学者工具包项目（必须已生成）[链接]，转到 \bin\Debug\ 以获取该 DLL。

    ![Obtain Dll][2]

4. 压缩 ShardElasticityModule 文件夹。 

    注意：Azure Automation 要求满足多个命名约定：以模块名称 ShardElasticityModule.psm1 为例，zip 文件名称必须完全匹配 (ShardElasticityModule.zip)。zip 文件包含 ShardElasticityModule 文件夹（名称符合模块的名称），而此文件夹又包含 psm1 文件。如果未遵循此结构，Azure Automation 将无法解包模块。

5.    在验证压缩文件夹的内容和结构符合要求之后，请继续下一步。它看起来应该如下：

    ![dll][3]


## 将 PowerShell 模块上载到 Azure Automation 作为资产 

将上述 PowerShell 模块上载到你的 Azure Automation 帐户。例如，模块会包含一组可从 Runbook 引用的分片弹性函数和弹性缩放 DLL。 

1. 按一下屏幕顶部功能区中的"资产"。
2. 按一下页面底部的"导入模块"。 
3. 按一下"浏览文件..."，找到上方的 **ShardElasticityModule.zip** 文件。 
4. 选择正确的文件后，单击框右下角的复选标记将它导入。Azure Automation 服务将导入该模块。 
5. 继续下一步。成功时的情况如下图所示。如果未成功导入模块，请确保 zip 文件符合上述约定。

    ![Assets][10] 
      
## 创建 Azure Automation 凭据和变量资产 

Azure Automation 可以分别创建凭据和变量资产，供许多 Runbook 引用，你不需要将凭据和常用变量硬编码到 Runbook 中。例如，更改密码，但只在一个地方发生。 

1. 选择你刚刚创建的新 Azure Automation 帐户。
2. 在 **ShardElasticityExamples** 帐户下，单击功能区中的"资产"。
3. 单击屏幕底部的"添加设置"。  
4. 单击"添加凭据"。 

    ![Add credential][9]
4. 选择"Windows PowerShell 凭据"作为"凭据类型"，输入 **ElasticScaleCredential** 作为名称。说明是可选的。  
5. 单击框右下角的箭头。 

    注意：若要直接使用 Runbook 而不修改，请使用说明中提供的变量名称，一字不差。Runbook 会引用变量名称。 
5. 针对你要执行分片弹性示例的 Azure SQL DB 服务器，插入用户名和密码（两次）。 
 
6. 若要创建变量资产，请单击"添加设置"，然后选择"添加变量"。 

    ![Add variable][7]
7. 针对此教程，请创建完全限定 Azure SQL DB 服务器名称（将放置分片映射管理器和分片数据库）的变量。选择"字符串"作为"变量类型"，并输入 **SqlServerName**。单击箭头以继续。 
8. 输入完全限定的 Azure SQL DB 服务器名称作为"值"，然后单击复选标记。 
9. 现在，你已创建分片弹性 Runbook 中要使用的凭据和变量资产。继续下一步。成功时的情况如下所示： 
    

## 将 PowerShell Runbook 上载到 Azure Automation 

上载提供的四个示例 PowerShell Runbook。 

1. 若要上载新的 Azure Automation Runbook，请单击功能区中的"RUNBOOK"。 
2. 在屏幕的底部单击"导入"。 
3. 导航到文件所在的文件夹，选择 **SetupShardedEnvironment.ps1**，然后单击复选标记。 
4. 对于剩余的三个 PowerShell Runbook（**ProvisionByDate.ps1**、**ProvisionBySize.ps1** 和 **ReduceServiceTier.ps1**），请重复步骤 2 和 3。 
5. 继续下一步。 

## 设置分片环境 

下一步是执行 **SetupShardedEnvironment** Runbook，它会设置分片环境，在环境中提供分片映射管理器 DB、范围分片映射和当天的分片。   

1. 选择 **SetupShardedEnvironment** Runbook，单击其名称即可。 
2. 单击功能区上的"创作"。 
3. 在此屏幕中，你将看到构成 Runbook 的代码（还可以根据需要编辑代码）。若要设置分片环境，请单击屏幕底部的"测试"按钮。确认你要保存并测试 Runbook。 
4. 你可以在输出窗格中查看作业的状态。完成时，你指定的 Azure SQL DB 服务器中会填入分片映射管理器数据库和分片数据库。**SetupShardedEnvironment** Runbook 只用来设置分片的环境，而不是根据重复计划来运行。继续下一步。  

## 测试 Automation Runbook 

在发布和计划每个 Runbook 之前，测试是否可以成功执行该 Runbook。 

1. 单击页面顶部功能区的"RUNBOOK"。 
2. 单击 **ProvisionByDate** Runbook。
3. 单击页面顶部功能区的"创作"。 
4. 单击"保存"，然后单击"测试"。
5. 对 **ReduceServiceTier** 重复上述步骤。 

    请注意，由于 **ProvisionBySize** 和 **ProvisionByDate** 都会设置新的分片（使用不同的算法），因此当前不需要运行 **ProvisionByDate**。 

## 发布 Runbook 
下一步是发布 Runbook，使它可以计划为定期执行。 

1. 单击页面底部的"发布"。 
2. 单击"已发布"。 
3. 继续下一步。
 
## 计划 Runbook 

最后一个步骤是创建计划并链接到以上发布的 Runbook。 

1. 单击页面顶部的"计划"。 
2. 单击"链接到新计划"。
3. 适当地为计划命名，然后单击右箭头按钮。
4. 配置计划。
5. 完成后，单击框底部的复选标记。
6. 根据前面建立的计划执行作业后，请单击页面顶部功能区中的"作业"选项，然后选择计划的作业。 

## 结束语 

现在，你已成功创作并打包 PowerShell 模块、将 PowerShell 模块上载到 Azure Automation 作为资产，并已创建、测试、发布和计划 Runbook。若要监视 Runbook 的运行状况和状态，请使用仪表板和作业选项卡。 

提供的示例只是大致说明了组合弹性缩放、Azure SQL DB 和 Azure Automation 之后可发挥的用途。请进行试验，并以这些示例为基础，让你的 Azure SQL DB 弹性缩放应用程序能够横向和/或纵向缩放。 

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-scripting/zip.png
[2]: ./media/sql-database-elastic-scale-scripting/dll.png
[3]: ./media/sql-database-elastic-scale-scripting/lookslikethis.png
[4]: ./media/sql-database-elastic-scale-scripting/automation.png
[5]: ./media/sql-database-elastic-scale-scripting/prompt.png
[6]: ./media/sql-database-elastic-scale-scripting/success.png
[7]: ./media/sql-database-elastic-scale-scripting/add-variable.png
[8]: ./media/sql-database-elastic-scale-scripting/sign-up.png
[9]: ./media/sql-database-elastic-scale-scripting/add-credential.png
[10]: ./media/sql-database-elastic-scale-scripting/assets.png

<!--HONumber=55-->