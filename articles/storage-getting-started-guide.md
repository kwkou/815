<properties 
	pageTitle="在 5 分钟内开始使用 Azure Blob、表和队列" 
	description="了解如何使用 Azure 快速入门项目和 Visual Studio 快速掌握 Windows Azure Blob、表和队列。" 
	services="storage" 
	documentationCenter=".net" 
	authors="Selcin" 
	manager="adinah" 
	editor=""/>
<tags ms.service="storage"
    ms.date="02/18/2015"
    wacn.date="04/15/2015"
    />










# 在 5 分钟内开始使用 Azure Blob、表和队列 

本教程演示如何通过在 Visual Studio 中开发一个简单的 Azure 应用程序，对 **Azure 存储 Blob**、**表**和**队列**进行快速编程。 

本教程包括有关快速掌握 Azure 存储空间的两个主要方案：

- 在 Azure 存储模拟器上运行你的第一个 Azure 存储空间应用程序
- 在 Azure 存储服务上运行你的第一个 Azure 存储空间应用程序

如果在深入分析代码之前你想要了解 Azure 存储空间，请参阅[后续步骤][]。

## 在 Azure 存储模拟器上运行你的第一个 Azure 存储空间应用程序

本部分演示如何通过开发一个访问 [Azure 存储模拟器](https://msdn.microsoft.com/zh-cn/library/azure/hh403989.aspx)的示例应用程序，对 **Azure 存储 Blob**、**表** 和 **队列**进行编程。Windows Azure 存储模拟器提供了一个本地环境，用于根据开发目的模拟 Azure Blob、队列和表服务。使用存储模拟器，你可以在本地针对存储服务测试你的应用程序，且不会产生任何费用。

若要完成本部分，请确保先执行以下必要任务：

1. 若要编译和生成应用程序，需要在计算机上安装 [Visual Studio](https://www.visualstudio.com/zh-cn/visual-studio-homepage-vs.aspx) 。如果尚未安装 Visual Studio，可以在安装 [Azure SDK 2.5 for Visual Studio 2013](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids)  或更高版本时安装 Visual Studio Express for Web。 
2. 请确保计算机上已安装了 [Azure SDK 2.5 for Visual Studio 2013](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids)  或更高版本，因为它包含 Azure 快速入门示例项目和[用于 .NET 的 Azure 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx)。  
3. 检查计算机上是否安装了 [.NET Framework 4.5](http://www.microsoft.com/zh-CN/download/details.aspx?id=30653) ，因为 Azure 快速入门示例项目需要用到它。如果你不确定计算机上安装了哪个版本的 .NET Framework，请参阅[如何：确定安装的 .NET Framework 版本](https://msdn.microsoft.com/zh-cn/vstudio/hh925568.aspx)。或者，按"开始"按钮或 Windows 键，并键入"控制面板"。然后，单击"程序" >"程序和功能"。查看所有已安装程序的列表中是否包含 .NET Framework 4.5。

现在，让我们在 Visual Studio 中使用 Azure 快速入门示例项目之一创建一个简单的 Azure 存储空间应用程序。本教程重点介绍 **Azure Blob 存储**、**Azure 表存储**和 **Azure 存储队列**示例项目。对于每个示例项目，都可以遵循以下说明，但如果在步骤 3.a 中选择了不同的模板时则例外：

1. 按"开始"按钮或 Windows 键，然后键入"Visual Studio 2013"或"VS Express 2013 for Web"。单击要启动的程序。
2. 在"文件"菜单中，单击"新建项目"。
3. 在"新建项目"对话框中，单击"已安装" >"模板" >"Visual C#" >"云" >"快速启动" >"数据服务"。
	- 3.a.  选择以下模板之一："Azure Blob 存储"、"Azure 表存储"或"Azure 存储队列"。 
	- 3.b. 确保选择 **.NET Framework 4.5** 作为目标框架。	
	- 3.c. 根据所选的模板来命名应用程序，例如 **DataBlobStorage**、**DataTableStorage** 或 **DataStorageQueue**。单击"确定"。这应会创建一个新的 Visual Studio 解决方案。请参阅以下示例屏幕截图：
	
	![Azure QuickStarts][Image1]

我们建议你查看源代码，以了解如何在运行该应用程序之前对 Azure 存储空间进行编程。若要查看代码，请在 Visual Studio 中的"查看"菜单上选择"解决方案资源管理器"。然后双击 Program.cs 文件。 

现在，请使用安装 Azure SDK 过程中安装的 [Azure 存储模拟器](https://msdn.microsoft.com/zh-cn/library/azure/hh403989.aspx)运行示例应用程序：

1.	启动 Azure 存储模拟器：按"开始"按钮或 Windows 键，然后通过键入 Azure 存储模拟器来搜索该程序。从应用程序列表中选择它，以将它启动。
2.	在 Visual Studio 中的"生成"菜单上，单击"生成解决方案"。 
3.	在"调试"菜单中，按 **F11** 逐步运行该解决方案，或按 **F5** 运行该解决方案。

## 在 Azure 存储服务上运行你的第一个 Azure 存储空间应用程序
本部分演示如何通过开发一个访问 [Azure 存储服务](/documentation/services/storage)的示例应用程序，对 **Azure 存储 Blob**、**表**和**队列**进行编程。.

若要完成本部分，请确保先执行以下必要任务：

1. 若要编译和生成应用程序，需要在计算机上安装 [Visual Studio](https://www.visualstudio.com/zh-cn/visual-studio-homepage-vs.aspx) 。如果尚未安装 Visual Studio，可以在安装 [Azure SDK 2.5 for Visual Studio 2013](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids)  或更高版本时安装 Visual Studio Express for Web。 
2. 请确保计算机上已安装了 [Azure SDK 2.5 for Visual Studio 2013](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids)  或更高版本，因为它包含 Azure 快速入门示例项目和[用于 .NET 的 Azure 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx)。  
3. 检查计算机上是否安装了 [.NET Framework 4.5](http://www.microsoft.com/zh-CN/download/details.aspx?id=30653) ，因为 Azure 快速入门示例项目需要用到它。如果你不确定计算机上安装了哪个版本的 .NET Framework，请参阅[如何：确定安装的 .NET Framework 版本](https://msdn.microsoft.com/zh-cn/vstudio/hh925568.aspx)。或者，按"开始"按钮或 Windows 键，并键入"控制面板"。然后，单击"程序">"程序和功能"。查看所有已安装程序的列表中是否包含 .NET Framework 4.5。
4.	获取 Azure 订阅（如果尚未获取），同时请创建一个**标准存储**帐户：
	- 若要获取 Azure 订阅，请参阅[免费试用](/pricing/1rmb-trial)，[购买选项](/pricing/overview)。
	- 若要在 Azure 中创建**标准存储**帐户，请参阅[如何创建、管理或删除存储帐户](/documentation/articles/storage-create-storage-account)。**注意：**Azure 中有两种类型的存储帐户：标准存储帐户和高级存储帐户。标准存储帐户提供对 Azure Blob、表和队列存储的访问。高级存储帐户当前只可用于在 Azure 虚拟机使用的磁盘上存储数据。有关详细信息，请参阅[高级存储：Azure 虚拟机工作负载的高性能存储](/documentation/articles/storage-premium-storage-preview-portal)。

现在，让我们在 Visual Studio 中使用 Azure 快速入门示例项目之一创建一个简单的 Azure 存储空间应用程序。本教程重点介绍 **Azure Blob 存储**、**Azure 表存储**和 **Azure 存储队列**示例项目。对于每个示例项目，都可以遵循以下说明，但如果在步骤 3.a 中选择了不同的模板时则例外：

1. 按"开始"按钮或 Windows 键，然后键入"Visual Studio 2013"或"VS Express 2013 for Web"。单击要启动的程序。
2. 在"文件"菜单中，单击"新建项目"。
3. 在"新建项目"对话框中，单击"已安装" > "模板" >"Visual C#" > **"云"** >"快速启动" > "数据服务"。
	- 3.a. 选择以下模板之一："Azure Blob 存储"、"Azure 表存储"或"Azure 存储队列"。 
	- 3.b. 确保选择 **.NET Framework 4.5** 作为目标框架。
	- 3.c. 根据所选的模板来命名应用程序，例如 **DataBlobStorage**、**DataTableStorage** 或 **DataStorageQueue**。单击"确定"。这应会创建一个新的 Visual Studio 解决方案。 

我们建议你查看源代码，以了解如何在运行该应用程序之前对 Azure 存储空间进行编程。若要查看代码，请在 Visual Studio 中的"查看"菜单上选择"解决方案资源管理器"。然后双击 Program.cs 文件。 

现在，请运行示例应用程序：

1.	在 Visual Studio 中的"查看"菜单上，选择"解决方案资源管理器"。双击 App.config 文件并注释掉 Azure SDK 存储模拟器的连接字符串： 

	`<!--<add key="StorageConnectionString" value = "UseDevelopmentStorage=true;"/>-->`

2.	取消注释 Azure 存储服务的连接字符串，并在 App.config 文件中提供存储帐户名称和访问密钥：
	`<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey]"` 

	若要查找存储帐户名称和访问密钥，请参阅[什么是存储帐户](/documentation/articles/storage-whatis-account)。 

3.	在 App.config 文件中提供存储帐户名称和访问密钥后,在"文件"菜单中，单击"全部保存"以保存所有项目文件。 
4.	在"生成"菜单中，单击"生成解决方案"。 
5.	在"调试"菜单中，按 **F11** 逐步运行该解决方案，或按 **F5** 运行该解决方案。


## 后续步骤
在本教程中，你已了解如何针对 Azure Blob 存储、Azure 表存储和 Azure 存储队列进行编程。 

如果你想要了解有关这些服务的详细信息，请访问以下链接：

* [Windows Azure 存储空间简介](/documentation/articles/storage-introduction)
* [如何通过 .NET 使用 Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs)
* [如何通过 .NET 使用表存储](/documentation/articles/storage-dotnet-how-to-use-tables)
* [如何通过 .NET 使用队列存储](/documentation/articles/storage-dotnet-how-to-use-queues)
* [Azure 存储文档](/documentation/services/storage)
* [Azure 存储空间 MSDN 参考](http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx)
* [Azure 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx)
* [Azure 存储 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx)

[Image1]: ./media/storage-getting-started-guide/QuickStart.png


<!--HONumber=50-->