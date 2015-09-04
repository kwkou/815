<properties 
	pageTitle="具有 Python Tools 2.1 for Visual Studio 的 Azure 上的 Bottle 和 Azure 表存储" 
	description="了解如何使用 Python Tools for Visual Studio 来创建 Bottle 应用程序，该应用程序在 Azure 表存储中存储数据并且可以部署到 Azure App Service Web Apps。" 
	services="app-service\web" 
	documentationCenter="python" 
	authors="huguesv" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.date="04/16/2015" 
	wacn.date="08/29/2015"/>




# 具有 Python Tools 2.1 for Visual Studio 的 Azure 上的 Bottle 和 Azure 表存储 

在本教程中，我们将使用 [Python Tools for Visual Studio] 通过其中一个 PTVS 样本模板创建一个简单的轮询 Web 应用。还以[视频](https://www.youtube.com/watch?v=GJXDGaEPy94)形式提供了本教程。

轮询 Web 应用定义其存储库的抽象，因此您可以轻松地在不同类型存储库（内存中、Azure 表存储、MongoDB）之间进行切换。

我们将学习如何创建 Azure 存储帐户、如何配置应用程序以使用 Azure 表存储以及如何将 Web 应用发布到 [Azure App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714)。

请参阅 [Python 开发人员中心]以获取更多文章，这些文章涵盖了通过 PTVS 使用 Bottle、Flask 和 Django web 框架开发 Azure App Service Web Apps，使用 MongoDB、Azure 表存储、MySQL 和 SQL 数据库服务来开发 Azure App Service Web Apps。虽然本文将着重介绍 App Service，但是步骤相似于开发 [Azure 云服务]。

+ [先决条件](#prerequisites)
+ [创建项目](#create-the-project)
+ [创建 Azure 存储帐户](#create-an-azure-storage-account)
+ [配置项目](#configure-the-project)
+ [了解 Azure 表存储](#explore-the-azure-table-storage)
+ [发布到 Azure 网站](#publish-to-an-azure-website)
+ [配置 Azure 网站](#configure-the-azure-website)
+ [后续步骤](#next-steps)

##<a name="prerequisites"></a>先决条件

 - Visual Studio 2012 或 2013
 - [Python Tools 2.1 for Visual Studio]
 - [Python Tools 2.1 for Visual Studio 样本 VSIX]
 - [Azure SDK Tools for VS 2013] 或 [Azure SDK Tools for VS 2012]
 - [Python 2.7 32 位]或 [Python 3.4 32 位]

[AZURE.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

>[AZURE.NOTE]如果您想要在注册 Azure 帐户之前开始使用 Azure App Service，请转到[试用 App Service](http://go.microsoft.com/fwlink/?LinkId=523751)，您可以在 App Service 中立即创建一个生存期较短的入门 Web 应用。你不需要使用信用卡，也不需要做出承诺。

## 创建项目

在此部分中，我们将使用样本模板创建 Visual Studio 项目。我们将创建虚拟环境并安装所需软件包。然后，我们将使用默认内存中存储库在本地运行应用程序。

1.  在 Visual Studio 中，选择**文件**、**新建项目**。

1.  在 **Python**、**Samples** 下提供来自 PTVS 样本 VSIX 的项目模板。选择**轮询 Bottle Web 项目**，然后单击“确定”以创建项目。

  	![新建项目对话框](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleNewProject.png)

1.  系统将提示您安装外部软件包。选择**安装到虚拟环境**。

  	![外部软件包对话框](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleExternalPackages.png)

1.  选择 **Python 2.7** 或 **Python 3.4** 作为基础解释程序。

  	![添加虚拟环境对话框](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonAddVirtualEnv.png)

1.  按 `F5` 以确认该应用程序有效。默认情况下，该应用程序使用内存中存储库，这并不需要任何配置。停止 web 服务器时，所有数据都会丢失。

1.  单击**创建样本轮询**，然后单击轮询并进行投票。

  	![Web 浏览器](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleInMemoryBrowser.png)

## 创建 Azure 存储帐户

要使用存储操作，你需要一个 Azure 存储帐户。可通过以下步骤创建存储帐户。

1.  登录到 [Azure 门户]。

2. 单击门户左下角的**新建**图标，然后单击**数据 + 存储** > **存储**。提供存储帐户一个唯一名称并为其创建一个新[资源组](/documentation/articles/resource-group-overview)。

1.  依次单击**数据服务**、**存储**和**快速创建**。

  	![快速创建](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonAzureStorageCreate.png)

	创建存储帐户后，**通知**按钮将呈绿色闪烁**成功**，且存储帐户的边栏选项卡处于打开状态以显示属于您创建的新资源组。

  	<!-- ![Quick Create](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonAzureStorageCreate.png) -->

5. 单击存储帐户边栏选项卡处的**设置**部分。记录帐户名称和主密钥。

	我们将需要此信息来配置要下一节中的项目。

## 配置项目

在此部分中，我们将配置应用程序以使用我们刚刚创建的存储帐户。然后我们将在本地运行应用程序。

1.  在 Visual Studio 中，右键单击 Solution Explorer 中的项目节点，然后选择**属性**。单击**调试**选项卡。

  	![项目调试设置](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleAzureTableStorageProjectDebugSettings.png)

1.  在**调试服务器命令**、**环境**中设置应用程序所需的环境变量的值。

        REPOSITORY_NAME=azuretablestorage
        STORAGE_NAME=<storage account name>
        STORAGE_KEY=<primary access key>

    您**启动调试**时，这会设置环境变量。如果您**启动但不调试**希望设置变量，还在**运行服务器命令**下设置相同值。

    或者，可以使用 Windows 控制面板来定义环境变量。如果您想要避免将凭证存储在源代码中/项目文件中，这是更好的选择。请注意，您将需要重新启动 Visual Studio 以使新环境值可用于应用程序。

1.  实现 Azure 表存储库中的代码在 **models/azuretablestorage.py** 中。请参阅[文档]以了解如何从 Python 使用表服务的更多信息。

1.  使用 `F5` 来运行应用程序。使用**创建样本轮询**来创建的轮询以及通过投票提交的数据将在 Azure 表存储中序列化。

1.  浏览到**有关**页以验证该应用程序是否正在使用 **Azure 表存储**存储库。

  	![Web 浏览器](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleAzureTableStorageAbout.png)

## 了解 Azure 表存储

很容易地使用 Visual Studio 中的 Server Explorer 查看和编辑存储表。本部分中，我们将使用 Server Explorer 查看轮询应用程序表的内容。

> [AZURE.NOTE]这要求安装 Microsoft Azure 工具，这些可作为 [Azure SDK for .NET] 的一部分而提供。

1.  打开**服务器资源管理器**。展开 **Azure**、**存储**、您的存储帐户，然后展开**表**。

  	<!-- ![Server Explorer](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonServerExplorer.png) -->

1.  双击**轮询**或**选择**表以在文档窗口中查看内容，以及添加/删除/编辑实体。

  	<!-- ![Table Query Results](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonServerExplorerTable.png) -->

## 将 Web 应用发布到 Azure App Service

Azure.NET SDK 轻松实现将 Web 应用部署到 Azure App Service。

1.  在**解决方案资源管理器**中，右键单击项目节点并选择**发布**。

  	<!-- ![Publish Web Dialog](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonPublishWebSiteDialog.png) -->

1.  单击 **Windows Azure Web Apps**。

1.  单击**新建**以创建一个 Web 应用。

1.  填写以下字段并单击**创建**。
	-	**Web 应用名称**
	-	**App Service 计划**
	-	**资源组**
	-	**区域**
	-	将保留**数据库服务器**设置为**没有数据库**

  	<!-- ![Create Web App on Microsoft Azure Dialog](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonCreateWebSite.png) -->

1.  接受其他所有默认值，然后单击**发布**。

1.  Web 浏览器会自动打开到已发布 Web 应用。如果浏览到“有关”页，您将看到它使用**内存中**存储库，而不是 **Azure 表存储**存储库。

    这是因为未在 Azure App Service 的 Web Apps 实例上设置环境变量，因此它将使用 **settings.py** 中指定的默认值。

## 配置 Web Apps 实例

在此部分中，我们将配置 Web Apps 实例的环境变量。

1.  在 [Azure 门户]，通过单击**浏览** > **Web Apps** > 您的 Web 应用名称来打开 Web 应用的边栏选项卡。

1.  在 Web 应用边栏选项卡中，单击**所有设置**，然后单击**应用程序设置**。

  	<!-- ![Top Menu](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonWebSiteTopMenu.png) -->

1.  向下滚动到**应用设置**部分并设置 **REPOSITORY_NAME**、**STORAGE_NAME** 和 **STORAGE_KEY** 的值（如**配置项目**部分所述）。

  	<!-- ![App Settings](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonWebSiteConfigureSettingsTableStorage.png) -->

1. 单击**存保**，然后单击**重新启动**，最后单击**浏览**。

  	<!-- ![Bottom Menu](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonWebSiteConfigureBottomMenu.png) -->

1.  您应该看到 Web 应用按预期方式使用 **Azure 表存储**存储库工作。

    祝贺你！

  	![Web 浏览器](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleAzureBrowser.png)

## 后续步骤

请按照下面的链接以了解有关 Python Tools for Visual Studio、 Bottle 和 Azure 表存储的更多信息。

- [Python Tools for Visual Studio 文档]
  - [Web 项目]
  - [云服务项目]
  - [在 Microsoft Azure 上的远程调试]
- [Bottle 文档]
- [Azure 存储空间]
- [Azure SDK for Python]
- [如何从 Python 使用表存储服务]

## 发生的更改
* 有关从网站更改为 App Service 的指南，请参阅：[Azure App Service 及其对现有 Azure 服务的影响](http://go.microsoft.com/fwlink/?LinkId=529714)
* 有关从旧门户更改为新门户的指南，请参阅：[有关在门户中导航的参考](http://go.microsoft.com/fwlink/?LinkId=529715)


<!--Link references-->
[Python 开发人员中心]: /develop/python/
[Azure 云服务]: /documentation/articles/cloud-services-python-ptvs
[文档]: /documentation/articles/storage-python-how-to-use-table-storage
[如何从 Python 使用表存储服务]: /documentation/articles/storage-python-how-to-use-table-storage

<!--External Link references-->
[Azure 门户]: https://manage.windowsazure.cn
[Azure SDK for .NET]: http://azure.microsoft.com/downloads/
[Python Tools for Visual Studio]: http://aka.ms/ptvs
[Python Tools 2.1 for Visual Studio]: http://go.microsoft.com/fwlink/?LinkId=517189
[Python Tools 2.1 for Visual Studio 样本 VSIX]: http://go.microsoft.com/fwlink/?LinkId=517189
[Azure SDK Tools for VS 2013]: http://go.microsoft.com/fwlink/?LinkId=323510
[Azure SDK Tools for VS 2012]: http://go.microsoft.com/fwlink/?LinkId=323511
[Python 2.7 32 位]: http://go.microsoft.com/fwlink/?LinkId=517190
[Python 3.4 32 位]: http://go.microsoft.com/fwlink/?LinkId=517191
[Python Tools for Visual Studio 文档]: http://pytools.codeplex.com/documentation
[Bottle 文档]: http://bottlepy.org/docs/dev/index.html
[在 Microsoft Azure 上的远程调试]: http://pytools.codeplex.com/wikipage?title=Features%20Azure%20Remote%20Debugging
[Web 项目]: http://pytools.codeplex.com/wikipage?title=Features%20Web%20Project
[云服务项目]: http://pytools.codeplex.com/wikipage?title=Features%20Cloud%20Project
[Azure 存储空间]: /documentation/services/storage
[Azure SDK for Python]: https://github.com/Azure/azure-sdk-for-python
 

<!---HONumber=67-->