<properties linkid="develop-media-services-how-to-guides-set-up-computer" urlDisplayName="Set Up Computer for Media Services" pageTitle="How to Set Up Computer for Media Services - Azure" metaKeywords="" description="Learn about the prerequisites for Media Services using the Media Services SDK for .NET. Also learn how to create a Visual Studio app." metaCanonical="" services="media-services" documentationCenter="" title="Setting up your computer for Media Services development" authors="migree" solutions="" manager="" editor="" />

设置计算机以进行 Media Services 开发
====================================

本部分介绍使用 Media Services SDK for .NET 进行 Media Services 开发需要满足的一般性先决条件。另外，还向开发人员演示了如何创建一个用于 Media Services SDK 开发的 Visual Studio 应用程序。

### 先决条件

-   在新的或现有的 Azure 订阅中拥有一个 Media Services 帐户。请参阅主题[如何创建 Media Services 帐户](http://www.windowsazure.com/zh-cn/manage/services/media-services/how-to-create-a-media-services-account/)。
-   操作系统：Windows 7、Windows 2008 R2 或 Windows 8。
-   .NET Framework 4.5 或 .NET Framework 4。
-   Visual Studio 2012 或 Visual Studio 2010 SP1（专业版、高级专业版、旗舰版或学习版）。
-   使用 [windowsazure.mediaservices Nuget](http://nuget.org/packages/windowsazure.mediaservices) 程序包安装 Azure SDK for .NET。以下部分说明了如何使用 **Nuget** 安装该 Azure SDK。

针对 Media Services 设置 Azure 帐户
-----------------------------------

若要设置你的 Media Services 帐户，请使用 Azure 管理门户（建议）。请参阅主题 [如何创建 Media Services 帐户][]。在管理门户中创建帐户后，便可以设置你的计算机以进行 Media Services 开发。

### 在 Visual Studio 中创建应用程序

本部分演示如何在 Visual Studio 中创建项目，以及如何将该项目设置为进行 Media Services 开发。在本示例中，该项目为 C\# Windows 控制台应用程序，但此处所示的设置步骤同样适用于针对 Media Services 应用程序（例如，Windows 窗体应用程序或 ASP.NET Web 应用程序）创建的其他类型的项目。

1.  在 Visual Studio 2012 或 Visual Studio 2010 SP1 中创建一个新的 C\# **控制台应用程序**。输入“名称”、“位置”和“解决方案名称”，然后单击**“确定”**。
2.  添加对 **System.Configuration** 程序集的引用。若要添加对 **System.Configuration** 的引用，请在**解决方案资源管理器**中，右键单击**“引用”**节点，然后选择**“添加引用...”**。在“管理引用”对话框中，**选择** **“System.Configuration”**，然后单击**“确定”**。
3.  添加对 **Azure SDK for .NET**(Microsoft.WindowsAzure.StorageClient.dll)、**Azure Media Services SDK for .NET** (Microsoft.WindowsAzure.MediaServices.Client.dll) 和 **WCF Data Services 5.0 for OData V3** (Microsoft.Data.OData.dll) 库的引用，只需通过 [windowsazure.mediaservices Nuget](http://nuget.org/packages/windowsazure.mediaservices) 程序包进行操作即可。

<!-- -->

    To add references using Nuget, do the following.In Visual Studio **Main Menu**, select **TOOLS** -> **Library Package Manager** -> **Package Manager Console**.In the console window type **Install-Package windowsazure.mediaservices** and press **Enter**.

1.  使用以下代码覆盖位于 Program.cs 文件开头的现有 using 语句。

using System;

     using System.Linq;
        using System.Configuration;
        using System.IO;
        using System.Text;
        using System.Threading;
        using System.Threading.Tasks;
        using System.Collections.Generic;
        using Microsoft.WindowsAzure;
        using Microsoft.WindowsAzure.MediaServices.Client;

现在，你可以开始开发 Media Services 应用程序了。

连接到 Media Services
---------------------

在 Media Services 编程过程中，几乎所有操作都需要引用服务器上下文对象。服务器上下文允许你通过编程方式访问所有 Media Services 编程对象。

若要获取对服务器上下文的引用，可按照以下代码示例返回上下文类型的新实例。将 Media Services 帐户名称和帐户密钥（已在帐户设置过程中获得）传递到构造函数中。

    static CloudMediaContext GetContext()
    {
    // Gets the service context. 
    return new CloudMediaContext(_accountName, _accountKey);
    } 

可以定义一个类型为 **CloudMediaContext** 的模块级变量来存储对 **GetContext** 之类的方法返回的服务器上下文的引用，这通常很有用。本主题的代码示例的其余部分使用名为 **\_context** 的变量来引用服务器上下文。

后续步骤
--------

你已经完成计算机设置并创建了进行 Media Services 编程所需的 Visual Studio 解决方案，现在可转到 [如何创建加密的资产并上载到存储中][] 主题。[如何创建 Media Services 帐户]：http://go.microsoft.com/fwlink/?linkid=256662 [如何创建加密的资产并上载到存储中]：http://go.microsoft.com/fwlink/?LinkID=301733&clcid=0x409

