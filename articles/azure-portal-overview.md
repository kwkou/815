<properties
	pageTitle="Microsoft Azure 预览门户概述"
	description="了解如何使用 Microsoft Azure 预览门户。"
	services=""
	documentationCenter=""
	authors="davidwrede"
	manager="dwrede"
	editor="jimbe"/>

<tags
	ms.service="na"
	ms.date="04/28/2015"
	wa.date="10/3/2015"/>

# Microsoft Azure 预览门户概述

Microsoft Azure 预览门户是一个中心位置，您可以在其中预配和管理 Azure 资源。借助本教程，您将熟悉预览门户以及如何使用它的一些关键功能：- **综合应用商店**：您可以浏览 Microsoft 和其他供应商提供的数千可供购买和/或预配的产品。- **统一且可缩放的浏览体验**：您可以轻松查找所需的资源，并执行各种管理操作。- **一致的管理页面**（或边栏选项卡）：您可以通过一致的方式公开设置、操作、计费信息、运行状况监视和使用量数据等，从而管理 Azure 的各种服务。- **个性体验**：您可以创建自定义的开始屏幕，显示您在登录时随时想要查看的信息。您还可以自定义包含磁贴的任意管理边栏选项卡。

 ![熟悉 Azure 门户 UI][UIOrientation]

## 准备工作

您必须拥有有效的 Azure 订阅，才能完整浏览本教程。如果您没有，请立即[注册免费试用版](http://www.windowsazure.cn/pricing/1rmb-trial/)。拥有订阅后，您便可以访问预览门户 [https://portal.azure.com]。

## 如何创建资源

Azure 应用商店提供数千商品，您可以在一个位置集中创建商品。假设您要新建 Windows Server 2012 VM。“+新”中心是您从应用商店进入特色类别的精选组的入口点。每个类别都有少量的特色项目，以及指向完整应用商店（显示所有类别和搜索）的链接。若要新建 Windows Server 2012 VM，请执行以下操作：

1.	Windows Server 2012 是一种特色类别，因此您可以从计算类别中选择它。  
2.	在表单上填写一些基本输入信息。
3.	单击“创建”。此时，系统会立即开始预配您的 VM。

当您的资源已完成创建时，通知中心会向您发出提醒，并且管理边栏选项卡也会打开（您稍后始终可以浏览资源）。

![门户类别][PortalCategories]


## 如何查找资源

您始终可以将经常访问的资源固定到启动板，但您也可能需要浏览不经常访问的内容。您可以通过浏览中心（如下所示）访问所有资源。您可以按订阅进行筛选、选择列/调整列大小，并能通过单击各个项导航到管理边栏选项卡。

![浏览中心][BrowseHub]

## 如何管理和委托资源访问权限

在此边栏选项卡中，您可以使用远程桌面连接到虚拟机、监视关键性能指标、使用基于角色的访问 (RBAC) 控制对此 VM 的访问、配置 VM，并能执行其他重要的管理任务。根据角色委托访问权限对大规模管理至关重要。有关详情，请单击[此处](/documentation/articles/role-based-access-control-configure)。若要委托资源访问权限，请执行以下操作：

1.	浏览您的资源。
2.	单击“必备”部分中的“所有设置”。
3.	单击设置列表中的“用户”。
4.	单击命令栏中的“添加”。
5.	选择用户和角色。

![管理资源][ManageResource]

## 如何自定义资源边栏选项卡

Azure 会为您的资源预配置边栏选项卡，但这些边栏选项卡上的磁贴由您控制。您可以轻松地转到自定义模式来添加、删除、调整或重新排列磁贴。若要自定义边栏选项卡，请执行以下操作：

1.	浏览您的资源。
2.	单击要自定义的边栏选项卡顶部的“...”。
3.	单击“添加部件”。
4.	开始拖放部件。  

![自定义边栏选项卡][CustomizeBlades]

## 如何获取帮助

如果您遇到问题，请随时与我们联系。预览门户具有帮助和支持页面，可以为您指引正确的方向。根据[支持计划](http://www.windowsazure.cn/support/plans/)，您还可以直接在门户中创建支持请求。创建支持请求之后，您可以在门户内管理请求的生命周期。您可以导航到“浏览”->“帮助 + 支持”，访问帮助和支持页面。

![帮助和支持][HelpSupport]

## 摘要

让我们回顾一下本教程的内容：- 您学习了如何注册和获取订阅，以及如何浏览门户 - 您熟悉了门户 UI，并学习了如何创建和浏览资源 - 您学习了如何创建和浏览资源 - 您了解了结构或管理边栏选项卡，并学习了如何一致地管理不同类型的资源 - 您学习了如何自定义门户，主要显示您关注的信息 - 您学习了如何使用基于角色的访问 (RBAC) 控制资源访问权限 - 您学习了如何获取帮助和支持

Microsoft Azure 预览门户大大简化了在云中构建和管理应用程序的工作。请参阅[管理博客](http://www.windowsazure.cn/blog/topics/management/)，了解最新动态，因为我们一直坚持[聆听反馈](http://feedback.azure.com/forums/223579-azure-preview-portal)并做出改进。有关所有 Azure 更新，您还可以访问 [ScottGu 的博客](http://weblogs.asp.net/scottgu)。

[UIOrientation]: ./media/azure-portal-how-to-use/azure_portal_1.png
[PortalCategories]: ./media/azure-portal-how-to-use/azure_portal_2.png
[BrowseHub]: ./media/azure-portal-how-to-use/azure_portal_3.png
[ManageResource]: ./media/azure-portal-how-to-use/azure_portal_4.png
[CustomizeBlades]: ./media/azure-portal-how-to-use/azure_portal_5.png
[HelpSupport]: ./media/azure-portal-how-to-use/azure_portal_6.png

<!---HONumber=71-->