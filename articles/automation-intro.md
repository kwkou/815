<properties
	pageTitle="什么是 Azure 自动化？"
	description="了解 Azure 自动化提供的价值并获取常见问题的答案，以便您可以开始创建和使用 Runbook。"
	services="automation"
	documentationCenter=""
	authors="bwren"
	manager="stevenka"
	editor=""/>

<tags
	ms.service="automation"
	ms.date="09/17/2015"
	wacn.date="11/02/2015"/>

# 什么是 Azure Automation？

借助 Windows Azure 自动化，用户可以自动完成通常要在云环境和企业环境中执行的手动、长时间进行、易出错且重复性高的任务。本文提供了有关 Azure 自动化的一些常见问题的简要解答。您可以参考此库中的其他文章，以了解有关不同主题的更多详细信息。

## Azure 自动化提供什么价值？

Azure 自动化节省时间并提高您在云和企业环境中执行常规管理任务的可靠性。您可以按 Runbook 实现这些过程，Runbook 可以执行多个任务而无需人工干预，甚至可以安排它们定期自动执行。

## Runbook 是什么？

Runbook 是 Azure 自动化中执行某些自动化过程的一组任务。它可以是一个简单的过程（如启动虚拟机或创建日志项），也可是复杂 Runbook，组合其他较小的 Runbook 来执行跨多个资源甚至多个云的复杂过程。

例如，您可能具有一个用于设置新虚拟机的现有手动流程，包括创建虚拟机、将其连接到网络、为其分配一个 IP 地址，然后通知用户该虚拟机已就绪等多个步骤。您无需手动执行每个步骤，而是可以创建一个将所有这些任务作为单个过程执行的 Runbook。您将启动该 Runbook、提供所需的信息，如虚拟机名称、IP 地址和收件人电子邮件，然后无需干预，等待流程自动完成即可。


## Runbook 可以自动化哪些任务？

在 Azure 自动化中的 Runbook 均基于 PowerShell 工作流，因此它们可以执行 PowerShell 可以完成的任何工作。如果应用程序或服务具有一个 API，Runbook 可以使用它。如果您有一个用于 Runbook 的 PowerShell 模块，可以将该模块加载到 Azure 自动化中，并在 Runbook 中包括这些 cmdlet。Azure 自动化 Runbook 在 Azure 云中运行，因此可以访问云中的任何资源或可从云中访问的外部资源。


## 从哪里获得 Runbook?

[Runbook 库](http://msdn.microsoft.com/zh-cn/library/azure/dn781422.aspx)包含来自 Microsoft 和社区的 Runbook，既可以在您的环境中使用未经修改的 Runbook，也可以根据自己的目的对其进行自定义。您还可以将它们作为学习如何创建自己的 Runbook 的参考。您甚至可以将您认为对其他用户有用的自己的 Runbook 分享到库中。


## 如何创建我自己的 Runbook？

您可以从头开始[创建您自己的 Runbook](http://msdn.microsoft.com/zh-cn/library/azure/dn643637.aspx)，根据您自己的要求修改[Runbook 库](http://msdn.microsoft.com/zh-cn/library/azure/dn781422.aspx)中的 Runbook。如果您愿意直接处理 PowerShell 代码，可以在 Azure 门户中[使用文本编辑器编辑 Runbook](http://msdn.microsoft.com/zh-cn/library/azure/dn879137.aspx) 或脱机编辑。


## Azure 自动化如何与其他自动化工具关联？

[Service Management Automation (SMA)](http://technet.microsoft.com/zh-cn/library/dn469260.aspx) 用于自动处理私有云中的管理任务。它作为 [Windows Azure Pack](http://www.microsoft.com/zh-cn/server-cloud/products/windows-azure-pack/default.aspx) 的组件本地安装在您的数据中心中。SMA 和 Azure 自动化使用基于 Windows PowerShell 工作流的相同的 Runbook 格式。

[System Center 2012 Orchestrator](http://technet.microsoft.com/zh-cn/library/hh237242.aspx) 适用于本地资源的自动化。它使用与 Azure 自动化和 Service Management Automation 不同的 Runbook 格式，并且具有图形界面，可用于创建 Runbook 而无需编写任何脚本。它的 Runbook 由专门为 Orchestrator 编写的集成包中的活动构成。

## 从哪里可以获得详细信息？

我们提供了多种资源，以帮助您了解有关 Azure 自动化的信息以及学习如何创建您自己的 Runbook。

- “Azure 自动化库”就是您当前正在查看的资源。该库中的文章提供了有关配置和管理 Azure 自动化以及创作自己的 Runbook 的完整文档。
- [Azure PowerShell cmdlet](http://msdn.microsoft.com/zh-cn/library/jj156055.aspx) 提供了有关使用 Windows PowerShell 自动完成 Azure 操作的信息。Runbook 使用这些 cmdlet 来处理 Azure 资源。
- [Azure 自动化博客](http://azure.microsoft.com/blog/tag/azure-automation)提供了有关来自 Microsoft 的 Azure 自动化的最新信息。欢迎订阅 Azure 自动化博客，随时了解 Azure 自动化团队提供的最新信息。
- [自动化论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=azureautomation)允许您提出有关 Azure 自动化的问题，并将由 Microsoft 和自动化社区提供解答。

## 我是否可以提供反馈？

**欢迎提供反馈！** 如果您正在寻找 Azure 自动化 Runbook 解决方案或集成模块，请在脚本中心发布脚本请求。如果您有关于 Azure 自动化的反馈或功能请求，请将其发布在[用户之声](/product-feedback)。谢谢！

<!---HONumber=76-->