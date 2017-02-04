<properties 
   pageTitle="在 Visual Studio 中配置和使用存储模拟器 | Azure"
   description="在 Visual Studio 中配置和使用存储模拟器"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />

<tags
    ms.assetid="c8e7996f-6027-4762-806e-614b93131867"
    ms.service="storage"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/18/2016"
    wacn.date="02/04/2017"
    ms.author="tarcher" />

# 在 Visual Studio 中配置和使用存储模拟器


## 概述
Azure SDK 开发环境包含存储模拟器，这是一个用于模拟本地开发计算机上的 Azure 中可用的 Blob、队列和表存储服务的实用工具。如果要生成采用 Azure 存储空间服务的云服务，或者编写调用存储服务的任何外部应用程序，你可以利用存储模拟器，在本地测试代码。Azure Tools for Microsoft Visual Studio 已将存储模拟器管理集成到 Visual Studio 中。Azure Tools 会在首次使用时初始化存储模拟器数据库，在你从 Visual Studio 中运行或调试代码时启动存储模拟器服务，并通过 Azure 存储资源管理器提供对存储模拟器数据的只读访问。

有关存储模拟器的详细信息，包括系统要求和自定义配置说明，请参阅 [Use the Azure Storage Emulator for Development and Testing（使用 Azure 存储模拟器进行开发和测试）](/documentation/articles/storage-use-emulator/)。

>[AZURE.NOTE] 存储模拟器的模拟与 Azure 存储空间服务之间在功能上存在一些差异。有关具体差异的信息，请参阅 Azure SDK 文档中的 [Differences Between the Storage Emulator and Azure Storage Services（存储模拟器与 Azure 存储空间服务之间的差异）](/documentation/articles/storage-use-emulator/)。

## 为存储模拟器配置连接字符串

若要从角色内的代码访问存储模拟器，需要配置指向存储模拟器并且以后可以更改为指向 Azure 存储帐户的连接字符串。连接字符串是一个配置设置，你的角色可在运行时读取该设置以连接到存储帐户。有关创建连接字符串的详细信息，请参阅 [Configuring the Azure Application（配置 Azure 应用程序）](https://msdn.microsoft.com/zh-cn/library/azure/2da5d6ce-f74d-45a9-bf6b-b3a60c5ef74e#BK_SettingsPage)。

>[AZURE.NOTE] 可通过使用 **DevelopmentStorageAccount** 属性将引用返回到代码的存储模拟器帐户中。如果要通过代码访问存储模拟器，此方法能正常工作；但是如果你计划将应用程序发布到 Azure，则需创建连接字符串来访问 Azure 存储帐户并修改代码（将其发布前）以便使用该连接字符串。如果要在存储模拟器帐户与 Azure 存储帐户之间进行频繁切换，使用连接字符串可以简化此过程。

## 初始化和运行存储模拟器

可指定在 Visual Studio 中运行或调试的时间，Visual Studio 将自动启动存储模拟器。在解决方案资源管理器中打开“Azure”项目的快捷菜单，然后选择“属性”。在“开发”选项卡上的“启动 Azure 存储模拟器”列表中，选择“True”（如果尚未设置为该值）。

当你第一次从 Visual Studio 中运行或调试服务时，存储模拟器将启动一个初始化过程。此过程保留用于存储模拟器的本地端口，并创建存储模拟器数据库。完成后，除非删除存储模拟器数据库，否则将不需要再次运行此过程。

>[AZURE.NOTE] 从 2012 年 6 月版的 Azure Tools 开始，存储模拟器默认在 SQL Express LocalDB 中运行。在 Azure Tools 此前的版本中，存储模拟器针对 SQL Express 2005 或 2008 的默认实例运行。你必须先安装该实例，然后才能安装 Azure SDK。你也可以针对 SQL Express 的命名实例或 Microsoft SQL Server 的命名实例或默认实例运行存储模拟器。如果你需要配置存储模拟器来运行默认实例以外的一个实例，请参阅 [Use the Azure Storage Emulator for Development and Testing（使用 Azure 存储模拟器进行开发和测试）](/documentation/articles/storage-use-emulator/)。

存储模拟器提供一个用户界面，用于查看本地存储服务的状态以及启动、停止和重置这些服务。启动存储模拟器服务之后，可以通过在 Windows 任务栏中右键单击“Microsoft Azure 模拟器”的通知区域图标，来显示用户界面或者启动或停止服务。

## 在服务器资源管理器中查看存储模拟器数据

服务器资源管理器中的“Azure 存储空间”节点可让你查看存储帐户（包括存储模拟器）中的数据，以及更改 Blob 和表数据的设置。有关详细信息，请参阅 [Browsing and Managing Storage Resources with Server Explorer（使用服务器资源管理器浏览和管理存储资源）](https://msdn.microsoft.com/zh-cn/library/azure/ff683677.aspx)。

<!---HONumber=Mooncake_0912_2016-->