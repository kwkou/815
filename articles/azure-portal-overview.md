<properties
	pageTitle=" Azure 门户概述"
	description="了解如何使用 Azure 门户。"
	services=""
	documentationCenter=""
	authors="davidwrede"
	manager="dwrede"
	editor="jimbe"/>

<tags
	ms.service="na"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="12/16/2015"
	wacn.date="05/13/2016"
	ms.author="dwrede"/>

#  Azure 门户概述

 Azure 门户是一个中心位置，你可以在其中预配和管理 Azure 资源。本教程将帮助你熟悉此门户，并且向你介绍如何使用以下一些关键功能：
- **综合应用商店**，你可以在其中浏览数千个来自 Microsoft 和其他供应商的应用，你可以购买这些应用和/或对其进行设置。
- **统一且可升级的浏览体验**，使你轻松查找你关注的资源并执行各种管理操作。
- **一致的管理页面**（或边栏选项卡），以一致的方式显示设置、操作、计费信息、运行状况监视和使用情况数据等等，使你能够管理 Azure 的各种服务。
- **个性化体验**，你可以创建自定义的开始屏幕，其中显示当你登录时想要看见的信息。您还可以自定义包含磁贴的任意管理边栏选项卡。

 ![熟悉 Azure 门户 UI][UIOrientation]

## 准备工作

您必须拥有有效的 Azure 订阅，才能完整浏览本教程。如果你没有，请立即[注册1元试用版](/pricing/1rmb-trial/)。拥有订阅后，你便可以访问 [Azure 门户](https://portal.azure.cn)。

## 如何创建资源

Azure 应用商店提供数千商品，您可以在一个位置集中创建商品。假设您要新建 Windows Server 2012 VM。“+新建”中心是您从应用商店进入特色类别的精选组的入口点。每个类别都有少量的特色项目，以及指向完整应用商店（显示所有类别和搜索）的链接。若要新建 Windows Server 2012 VM，请执行以下操作：

1.	Windows Server 2012 是一种特色类别，因此您可以从计算类别中选择它。  
2.	在表单上填写一些基本输入信息。
3.	单击“创建”。此时，系统会立即开始预配你的 VM。

当您的资源已完成创建时，通知中心会向您发出提醒，并且管理边栏选项卡也会打开（您稍后始终可以浏览资源）。

![门户类别][PortalCategories]


## 如何查找资源

您始终可以将经常访问的资源固定到启动板，但您也可能需要浏览不经常访问的内容。您可以通过浏览中心（如下所示）访问所有资源。您可以按订阅进行筛选、选择列/调整列大小，并能通过单击各个项导航到管理边栏选项卡。

![浏览中心][BrowseHub]

## 如何管理和委托资源访问权限

在此边栏选项卡中，您可以使用远程桌面连接到虚拟机、监视关键性能指标、使用基于角色的访问 (RBAC) 控制对此 VM 的访问、配置 VM，并能执行其他重要的管理任务。根据角色委托访问权限对大规模管理至关重要。若要委托资源访问权限，请执行以下操作：

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

如果您遇到问题，请随时与我们联系。门户具有帮助和支持页面，可以为您指引正确的方向。根据[支持计划](/support/plans/)，你还可以直接在门户中创建支持请求。创建支持请求之后，您可以在门户内管理请求的生命周期。您可以导航到“浏览”->“帮助 + 支持”，访问帮助和支持页面。

![帮助和支持][HelpSupport]

## 摘要

让我们回顾一下你在本教程中学习的内容：
- 你学习了如何注册、获取订阅，并浏览到门户
- 你通过门户 UI 进行导航，学习了如何创建和浏览资源
- 你学习了如何创建资源和浏览资源
- 你学习了结构或管理边栏选项卡，以及如何统一管理不同类型的资源
- 你学习了如何自定义门户，以便将你关心的信息置于前面或中心位置
- 你学习了如何使用基于角色的访问 (RBAC) 控制对资源的访问
- 你学习了如何获得帮助和支持


[UIOrientation]: ./media/azure-portal-how-to-use/azure_portal_1.png
[PortalCategories]: ./media/azure-portal-how-to-use/azure_portal_2.png
[BrowseHub]: ./media/azure-portal-how-to-use/azure_portal_3.png
[ManageResource]: ./media/azure-portal-how-to-use/azure_portal_4.png
[CustomizeBlades]: ./media/azure-portal-how-to-use/azure_portal_5.png
[HelpSupport]: ./media/azure-portal-how-to-use/azure_portal_6.png

<!---HONumber=Mooncake_0418_2016-->