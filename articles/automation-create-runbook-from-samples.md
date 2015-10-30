<properties 
	pageTitle="Azure 自动化入门" 
	description="了解如何在 Azure 中导入和运行自动化作业。" 
	services="automation" 
	documentationCenter="" 
	authors="bwren" 
	manager="stevenka" 
	editor=""/>

<tags 
	ms.service="automation" 
	ms.date="09/08/2015"
	wacn.date="10/17/2015"/>


# Azure 自动化入门

## 什么是 Azure 自动化？

通过 Windows Azure 自动化，用户可以自动完成通常要在云环境中执行的手动、长时间进行、易出错且重复性高的任务。你可以使用 Runbook 来创建、监视、管理和部署 Azure 环境中的资源。所谓的 Runbook，基本上就是指 Windows PowerShell 工作流。在本文中，你将学习运行一个简单示例 Runbook 的教程。然后，你将查找用于浏览更高级服务功能的资源。

## 教程
本教程将逐步引导你完成创建自动化帐户、将一个示例“Hello World”Runbook 导入 Azure 自动化、执行该 Runbook 并查看其输出的过程。

若要完成本教程，你需要一个 Azure 订阅。如果你没有订阅，可以[注册免费试用版](/pricing/1rmb-trial/)。

[AZURE.INCLUDE [automation-note-authentication](../includes/automation-note-authentication.md)]

## <a name="automationaccount"></a>创建自动化帐户

自动化帐户是自动化资源的容器：它提供了一种方式让你划分环境，或者进一步组织你的工作流。有关详细信息，请参阅自动化库中的[自动化帐户](https://msdn.microsoft.com/zh-cn/library/dn794195.aspx)。如果你已创建自动化帐户，可以跳过此步骤。

1.	登录到 [Azure 管理门户](http://manage.windowsazure.cn)。

2.	在管理门户中，单击“创建自动化帐户”。

	![创建帐户](./media/automation-create-runbook-from-samples/automation_01_CreateAccount.png)

3.	在“添加新的自动化帐户”页上，输入一个名称并选择该帐户的区域。该区域指定要将帐户中的自动化资源存储在哪个位置。这不会影响帐户的功能，但是，如果帐户所在区域靠近其他 Azure 资源的存储位置，则 Runbook 可以更快地执行。准备就绪后，单击复选标记。

	![添加新帐户](./media/automation-create-runbook-from-samples/automation_02_addnewautoacct.png)

## <a name="importrunbook"></a>从 Runbook 库导入 Runbook

[Runbook 库](https://msdn.microsoft.com/zh-cn/library/azure/dn781422.aspx)包括你可以直接导入 Azure 自动化帐户的示例 Runbook，从而可让你利用其他 Azure 自动化和 PowerShell 用户的工作。在此步骤中，你将使用该库导入“Hello World”示例 Runbook。

4.	在“自动化”页上，单击你刚刚创建的新帐户。

	![新帐户](./media/automation-create-runbook-from-samples/automation_03_NewAutoAcct.png)

5.	单击“RUNBOOK”。

	![Runbook 选项卡](./media/automation-create-runbook-from-samples/automation_04_RunbooksTab.png)

6.	单击“新建”>“Runbook”>“从库中”。

	![Runbook 库](./media/automation-create-runbook-from-samples/automation_05_ImportGallery.png)

7.  选择“教程”类别，然后选择“Hello World for Azure 自动化”。单击右箭头按钮。

	![导入 Runbook](./media/automation-create-runbook-from-samples/automation_06_ImportRunbook.png)

8.  查看 Runbook 的内容，然后单击右箭头按钮。

	![Runbook 定义](./media/automation-create-runbook-from-samples/automation_07_RunbookDefinition.png)

8.	查看 Runbook 的详细信息，然后单击复选标记按钮。

	![Runbook 详细信息](./media/automation-create-runbook-from-samples/automation_08_RunbookDetails.png)

## <a name="publishrunbook"></a>发布 Runbook 

Runbook 最初是以草稿模式导入的。这意味着，在授权它可作为新版本运行之前，你可以继续对它进行编辑。由于此示例 Runbook 不需要额外的配置，你可以按原样立即将其发布。有关详细信息，请参阅[发布 Runbook](https://msdn.microsoft.com/zh-cn/library/dn879137.aspx)。

9.	Runbook 导入完成后，单击“Write-HelloWorld”。

	![导入的 Runbook](./media/automation-create-runbook-from-samples/automation_07_ImportedRunbook.png)

9.	单击“创作”，然后单击“草稿”。

	可以在草稿模式下修改 Runbook 的内容。对于此 Runbook，你不需要做出任何修改。

	![创作草稿](./media/automation-create-runbook-from-samples/automation_08_AuthorDraft.png)

10.	单击“发布”以提升 Runbook，使其随时可在生产环境中使用。

	![发布](./media/automation-create-runbook-from-samples/automation_085_Publish.png)

11.	当系统提示你确认时，请单击“是”。

	![有关保存并发布 Runbook 的提示](./media/automation-create-runbook-from-samples/automation_09_SavePubPrompt.png)

## <a name="startrunbook"></a>启动 Runbook

在导入并发布 Runbook 后，你可以运行它，然后检查输出。有关详细信息，请参阅[启动 Runbook](https://msdn.microsoft.com/zh-cn/library/dn643628.aspx) 和 [Runbook 输出和消息](https://msdn.microsoft.com/zh-cn/library/dn879148.aspx)。

12.	打开 **Write-HelloWorld** Runbook 后，单击“启动”。

	![已发布](./media/automation-create-runbook-from-samples/automation_10_PublishStart.png)

13.	在“指定 Runbook 参数值”页上，键入要用作 Write-HelloWorld.ps1 脚本的输入参数的名称，然后单击复选标记。

	![Runbook 参数](./media/automation-create-runbook-from-samples/automation_11_RunbookParams.png)

14.	单击“作业”以查看你刚刚启动的 Runbook 作业的状态，然后单击“作业启动”列中的时间戳以查看作业摘要。

	![Runbook 状态](./media/automation-create-runbook-from-samples/automation_12_RunbookStatus.png)

15.	在“摘要”页上，你可以查看作业的摘要、输入参数和输出。

	![Runbook 摘要](./media/automation-create-runbook-from-samples/automation_13_RunbookSummary_callouts.png)

祝贺你！ 你已完成本教程。

## <a name="nextsteps"></a>后续步骤 
1. 本教程中的简单 Runbook *不会管理 Azure 服务*。大多数 Runbook 使用 [Azure cmdlet](http://msdn.microsoft.com/zh-cn/library/jj156055.aspx) 进行管理，这需要对你的 Azure 订阅进行身份验证。请遵循[配置 Azure 以通过 Runbook 进行管理](https://msdn.microsoft.com/zh-cn/library/dn865019.aspx)中的说明配置你的 Azure 订阅，以使用这些 cmdlet。  
2. 有关 Azure 自动化功能的详细信息，请参阅下面列出的[资源](#resources)。
3. 订阅 [Azure 自动化博客](http://azure.microsoft.com/blog/tag/azure-automation)，随时了解 Azure 自动化团队提供的最新信息。

## <a name="resources"></a>资源

我们提供了其他多种资源，以帮助你了解有关 Azure 自动化的信息以及如何创建你自己的 Runbook。

- [Azure 自动化库](https://msdn.microsoft.com/zh-cn/library/azure/dn643629.aspx)提供了有关配置和管理 Azure 自动化以及创作自己的 Runbook 的完整文档。 
- [Azure PowerShell cmdlet](http://msdn.microsoft.com/zh-cn/library/jj156055.aspx) 提供了有关使用 Windows PowerShell 自动完成 Azure 操作的信息。Runbook 使用这些 cmdlet 来处理 Azure 资源。
- [Azure 自动化博客](http://azure.microsoft.com/blog/tag/azure-automation)提供了有关来自 Microsoft 的 Azure 自动化的最新信息。

<!--
- [Automation Forum](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=azureautomation) allows you to post questions about Azure Automation to be addressed by Microsoft and the Automation community.
-->


## 示例和实用 Runbook

Microsoft 和 Azure 自动化社区提供了示例 Runbook，可帮助你开始创建自己的解决方案和实用 Runbook，可用作更大自动化任务的构建基块。可以从[脚本中心](http://go.microsoft.com/fwlink/p/?LinkId=393029)下载这些示例 Runbook，或者使用 [Runbook 库](https://msdn.microsoft.com/zh-cn/library/azure/dn781422.aspx)直接将其导入 Azure 自动化。
  

<!--
## Feedback

<strong>Give us feedback!</strong>  If you are looking for an Azure Automation runbook solution or an integration module, post a Script Request on Script Center. If you have feedback or feature requests for Azure Automation, post them on [User Voice](http://feedback.windowsazure.cn/forums/34192--general-feedback). Thanks!
-->

<!---HONumber=74-->