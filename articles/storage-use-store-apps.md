<properties 
	pageTitle="在 Windows 应用商店应用中使用 Azure 存储 | Azure" 
	description="了解如何创建使用 Azure Blob、队列、表或文件存储的 Windows 应用商店应用。"
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor="cgronlun"/>

<tags 
	ms.service="storage" 
	ms.date="02/24/2016" 
	wacn.date="04/18/2016"/>





# 如何在 Windows 应用商店应用程序中使用 Azure 存储空间

## 概述

本指南演示了如何开始开发使用 Azure 存储空间的 Windows 应用商店应用程序。

## 下载所需工具

- [Visual Studio](https://www.visualstudio.com/zh-cn/visual-studio-homepage-vs.aspx) 便于生成、调试、本地化、包装和部署 Windows 应用商店应用程序。需要 Visual Studio 2012 或更高版本。
- [Azure 存储空间客户端库](https://www.nuget.org/packages/WindowsAzure.Storage)提供了用于使用 Azure 存储空间的 Windows 运行时类库。
- [适用于 Windows 应用商店应用的 WCF 数据服务工具](http://www.microsoft.com/download/details.aspx?id=30714)利用 Visual Studio 中对 Windows 应用商店应用程序的客户端 OData 支持，扩展了“添加服务引用”体验。

## 开发应用

### 做好准备

在 Visual Studio 2012 或更高版本中创建新的 Windows 应用商店应用程序项目：

![store-apps-storage-vs-project][store-apps-storage-vs-project]

接下来，通过以下方法添加对 Azure 存储客户端库的引用：右键单击“引用”，单击“添加引用”，然后浏览到你已下载的针对 Windows 运行时的存储客户端库：

![store-apps-storage-choose-library][store-apps-storage-choose-library]

### 将库用于 Blob 和队列服务

此时，你的应用已做好调用 Azure Blob 和队列服务的准备。添加以下 using 语句，以便可以直接引用 Azure 存储类型：

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;

接下来，向你的页面添加一个按钮。将以下代码添加到其 **Click** 事件，并使用 [async 关键字](http://msdn.microsoft.com/zh-cn/library/vstudio/hh156513.aspx)修改你的事件处理程序方法：

    var credentials = new StorageCredentials(accountName, accountKey);
    var account = new CloudStorageAccount(credentials, true);
    var blobClient = account.CreateCloudBlobClient();
    var container = blobClient.GetContainerReference("container1");
    await container.CreateIfNotExistsAsync();

此代码假定你有两个字符串变量：accountName和accountKey。它们分别表示你的存储帐户名称以及与该帐户关联的帐户密钥。

构建并运行应用程序。单击该按钮将检查名为container1的容器在你的帐户中是否存在，如果不存在则创建它。

### 将库用于表服务

用于与 Azure 表服务进行通信的类型取决于针对 Windows 应用商店应用库的 WCF 数据服务。接下来，通过使用程序包管理器控制台添加对所需 WCF 库的引用：

![store-apps-storage-package-manager][store-apps-storage-package-manager]

使用以下命令可以将程序包管理器指向你的计算机上的位置：

    Install-Package Microsoft.Data.OData.WindowsStore -Source "C:\Program Files (x86)\Microsoft WCF Data Services\5.0\bin\NuGet"

该命令将自动添加对你的项目的所有必需的引用。如果你不想使用程序包管理器控制台，可以将你的本地计算机上的 WCF Data Services NuGet 文件夹添加到程序包源的列表，然后通过在[使用对话框管理 NuGet 程序包](http://docs.nuget.org/docs/start-here/Managing-NuGet-Packages-Using-The-Dialog)中介绍的用户界面添加引用。

在你引用 WCF Data Services NuGet 程序包后，更改你的按钮的 Click 事件中的代码：

    var credentials = new StorageCredentials(accountName, accountKey);
    var account = new CloudStorageAccount(credentials, true);
    var tableClient = account.CreateCloudTableClient();
    var table = tableClient.GetTableReference("table1");
    await table.CreateIfNotExistsAsync();

该代码将检查名为table1的表在你的帐户中是否存在，如果不存在则创建它。

你还可以添加对 Microsoft.WindowsAzure.Storage.Table.dll 的引用，该 dll 位于你下载的同一个程序包中。该库包含其他一些功能，例如基于反射的序列化和一般查询。请注意，该库不支持 JavaScript。



[store-apps-storage-vs-project]: ./media/storage-use-store-apps/store-apps-storage-vs-project.png
[store-apps-storage-choose-library]: ./media/storage-use-store-apps/store-apps-storage-choose-library.png
[store-apps-storage-package-manager]: ./media/storage-use-store-apps/store-apps-storage-package-manager.png
 

<!---HONumber=Mooncake_0411_2016-->