<properties 
   pageTitle="Azure SDK for .NET 2.9 发行说明" 
   description="Azure SDK for .NET 2.9 发行说明" 
   services="app-service\web" 
   documentationCenter=".net" 
   authors="Juliako" 
   manager="erikre" 
   editor=""/>

<tags
	ms.service="app-service"
	ms.date="04/25/2016"
	wacn.date="06/29/2016"/>


# Azure SDK for .NET 2.9 发行说明

##概述

本文档包含 Azure SDK for .NET 2.9 版本的发行说明。

有关此版本中的更新的详细信息，请参阅 [Azure SDK 2.9 通告文章](https://azure.microsoft.com/blog/announcing-visual-studio-azure-tools-and-sdk-2-9/)。

## Azure SDK 2.9 for Visual Studio 2015 Update 2 和 Visual Studio“15”预览版
 
此更新包含下列 bug 修复：

- 与 REST API 客户端生成有关的问题，在此问题中，字符串“未知类型”会作为代码生成文件夹的名称和/或放入所生成代码的命名空间的名称出现。
- 与计划 Web 作业有关的问题，在此问题中，验证信息无法传递给计划程序预配进程。

此更新包含下列新功能：

- 在 Azure 预配对话框的“服务”选项卡中支持辅助 Azure。 

##Azure Data Lake Tools for Visual Studio 2015 Update 2
 
此更新包含下列工具：

- **Azure Data Lake Tools** for Visual Studio 现在已合并到 Azure SDK for .NET 版本中。当你安装 Azure SDK 时，便会自动安装此工具。 

	此工具会经常更新，请转到[此处](http://aka.ms/datalaketool)获取更新。

- **服务器资源管理器**现在支持查看所有内容并创建一些 U-SQL 元数据实体。

##Azure 资源管理器 

此版本添加了对 ARM 模板的[密钥保管库](/documentation/articles/resource-manager-keyvault-parameter/)支持。

##另请参阅

[Azure SDK 2.9 通知文章](https://azure.microsoft.com/blog/announcing-visual-studio-azure-tools-and-sdk-2-9/)

<!---HONumber=Mooncake_0509_2016-->