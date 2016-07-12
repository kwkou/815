<properties 
	pageTitle="如何设置计算机以使用 .NET 进行媒体服务开发" 
	description="了解使用适用于 .NET 的媒体服务 SDK 进行媒体服务开发所要满足的先决条件。此外，了解如何创建 Visual Studio 应用程序。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"  
	wacn.date="06/20/2016"/>

#使用 .NET 进行媒体服务开发

[AZURE.INCLUDE [media-services-selector-setup](../includes/media-services-selector-setup.md)]

本主题介绍如何开始使用 .NET 开发媒体服务应用程序。

**Azure 媒体服务 .NET SDK** 库允许你使用 .NET 为媒体服务编程。为了进一步方便使用 .NET 进行开发，我们提供了 **Azure 媒体服务 .NET SDK 扩展**库。此库包含一组扩展方法和帮助器函数，可简化你的 .NET 代码。这两个库都通过 **NuGet** 和 **GitHub** 提供。


##先决条件

-   在新的或现有的 Azure 订阅中拥有一个媒体服务帐户。请参阅主题[如何创建媒体服务帐户](/documentation/articles/media-services-create-account/)。
-   操作系统：Windows 10、Windows 7、Windows 2008 R2 或 Windows 8。
-   .NET Framework 4.5。
-    Visual Studio 2015、Visual Studio 2013、Visual Studio 2012 或 Visual Studio 2010 SP1（专业版、高级版、旗舰版或速成版）。


##创建和配置 Visual Studio 项目

本部分演示如何在 Visual Studio 中创建项目，以及如何将该项目设置为进行媒体服务开发。在本示例中，该项目为 C# Windows 控制台应用程序，但此处所示的设置步骤同样适用于针对媒体服务应用程序（例如，Windows 窗体应用程序或 ASP.NET Web 应用）创建的其他类型的项目。

本部分说明如何使用 **NuGet** 添加媒体服务 .NET SDK 和其他依赖库。

此外，你还可以从 GitHub（[github.com/Azure/azure-sdk-for-media-services](https://github.com/Azure/azure-sdk-for-media-services) 和 [github.com/Azure/azure-sdk-for-media-services-extensions](https://github.com/Azure/azure-sdk-for-media-services-extensions)）获取最新的媒体服务 .NET SDK 资料，构建解决方案，并添加对客户端项目的引用。请注意，会自动下载并提取所有必需的依赖项。

1. 在 Visual Studio 2013、Visual Studio 2012 或 Visual Studio 2010 SP1 中创建一个新的 C# 控制台应用程序。输入“名称”、“位置”和“解决方案名称”，然后单击“确定”。

2. 生成解决方案。

2. 使用 “NuGet” 安装和添加 “Azure 媒体服务 .NET SDK 扩展”。安装此包也会安装“媒体服务 .NET SDK” 并添加所有其他必需的依赖项。
1. 确保已安装最新版本的 NuGet。有关详细信息和安装说明，请参阅 [NuGet](http://nuget.codeplex.com/)。

2. 在“解决方案资源管理器”中，右键单击项目名称，然后选择“管理 NuGet 包”。

此时将显示“管理 NuGet 包”对话框。

	3. 在“联机”库中，搜索 Azure MediaServices Extensions，选择“Azure Media Services .NET SDK Extensions”，然后单击“安装”按钮。
 
		此时将修改项目并添加对 Media Services .NET SDK Extensions、Media Services .NET SDK 和其他依赖程序集的引用。

4. 若要升级更干净的开发环境，请考虑启用 NuGet 包还原。有关详细信息，请参阅 [NuGet 包还原](http://docs.nuget.org/consume/package-restore)。

3. 添加对 **System.Configuration** 程序集的引用。此程序集包含用于访问配置文件（例如，App.config）的 System.Configuration.**ConfigurationManager** 类。

若要使用“管理引用”对话框添加引用，请执行以下操作：

1. 在解决方案资源管理器中，右键单击项目名称。然后，选择“添加”和“引用”。

此时将显示“管理引用”对话框。

2. 在 .NET Framework 程序集下，找到并选择 System.Configuration 程序集。
3. 按“确定”。


4. 打开 App.config 文件（如果该文件未按默认添加到项目中，请添加）并在该文件中添加 *appSettings* 节。如以下示例中所示设置 Azure 媒体服务帐户名和帐户密钥的值。

若要获取“帐户名”和“帐户密钥”信息，请打开“Azure 管理门户”，选择你的媒体服务帐户，然后单击“管理密钥”按钮。


<configuration> ... <appSettings> <add key="MediaServicesAccountName" value="Media-Services-Account-Name" /> <add key="MediaServicesAccountKey" value="Media-Services-Account-Key" /> </appSettings>
	  
	</configuration>

5. 使用以下代码覆盖位于 Program.cs 文件开头的现有 using 语句。

		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Text;
		using System.Threading.Tasks;
		using System.Configuration;
		using System.Threading;
		using System.IO;
		using Microsoft.WindowsAzure.MediaServices.Client;

现在，你可以开始开发媒体服务应用程序了。




<!---HONumber=Mooncake_0613_2016-->