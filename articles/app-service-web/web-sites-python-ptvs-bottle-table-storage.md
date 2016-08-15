<properties 
	pageTitle="具有 Python Tools 2.1 for Visual Studio 的 Azure 上的 Bottle 和 Azure 表存储" 
	description="了解如何使用 Python Tools for Visual Studio 来创建 Bottle 应用程序，该应用程序在 Azure 表存储中存储数据并且可以部署到 Azure Web 应用。" 
	services="app-service\web" 
	documentationCenter="python" 
	authors="huguesv" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web"
	ms.date="02/20/2016"
	wacn.date="05/24/2016"/>




# 具有 Python Tools 2.2 for Visual Studio 的 Azure 上的 Bottle 和 Azure 表存储 

在本教程中，我们将使用 [Python Tools for Visual Studio] 通过一个 PTVS 样本模板创建简单的轮询 Web 应用。

轮询 Web 应用定义其存储库的抽象，因此您可以轻松地在不同类型存储库（内存中、Azure 表存储、MongoDB）之间进行切换。

我们将了解如何创建 Azure 存储帐户、如何将 Web 应用配置为使用 Azure 表存储，以及如何将 Web 应用发布到 [Azure Web 应用](/documentation/services/web-sites/)中。

请参阅 [Python 开发人员中心]以获取更多文章，这些文章介绍了如何通过 PTVS（使用 Bottle、Flask 和 Django Web 框架）、MongoDB、Azure 表存储、MySQL 和 SQL 数据库服务来开发 Azure Web 应用。虽然本文将着重介绍 Azure Web 应用，但 [Azure 云服务]的开发步骤也是类似的。

##<a name="prerequisites"></a>先决条件

 - Visual Studio 2013 或 2015
 - [Python Tools 2.2 for Visual Studio]
 - [Python Tools 2.2 for Visual Studio 示例 VSIX]
 - [Azure SDK Tools for VS 2013] 或 [Azure SDK Tools for VS 2015]
 - [Python 2.7（32 位）]或 [Python 3.4（32 位）]

[AZURE.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 创建项目

在此部分中，我们将使用样本模板创建 Visual Studio 项目。我们将创建虚拟环境并安装所需软件包。然后，我们将使用默认内存中存储库在本地运行应用程序。

1.  在 Visual Studio 中，依次选择“文件”和“新建项目”。

1.  在 **Python**、**Samples** 下提供来自 PTVS 样本 VSIX 的项目模板。选择**轮询 Bottle Web 项目**，然后单击“确定”以创建项目。

  	![新建项目对话框](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleNewProject.png)

1.  系统将提示您安装外部软件包。选择**安装到虚拟环境**。

  	![外部包对话框](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleExternalPackages.png)

1.  选择“Python 2.7”或“Python 3.4”作为基础解释器。

  	![添加虚拟环境对话框](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonAddVirtualEnv.png)

1.  按 `F5` 确认应用程序能否正常运行。默认情况下，该应用程序使用内存中存储库，这并不需要任何配置。停止 web 服务器时，所有数据都会丢失。

1.  单击“创建样本轮询”，然后单击一个轮询进行投票。

  	![Web 浏览器](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleInMemoryBrowser.png)

## 创建 Azure 存储帐户

要使用存储操作，你需要一个 Azure 存储帐户。可通过以下步骤创建存储帐户。

1.  登录到 [Azure 经典管理门户]。

2. 单击经典管理门户左下角的**新建**图标，然后单击**数据 + 存储** > **存储**。提供存储帐户一个唯一名称。

1.  依次单击**数据服务**、**存储**和**快速创建**。

  	![快速创建](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonAzureStorageCreate.png)

5. 单击存储帐户边栏选项卡处的**设置**部分。记录帐户名称和主密钥。

	我们将需要使用此信息在下一部分中配置您的项目。

## 配置项目

在此部分中，我们将配置应用程序以使用我们刚刚创建的存储帐户。然后我们将在本地运行应用程序。

1.  在 Visual Studio 中，右键单击 Solution Explorer 中的项目节点，然后选择**属性**。单击“调试”选项卡。

  	![项目调试设置](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleAzureTableStorageProjectDebugSettings.png)

1.  在**调试服务器命令**、**环境**中设置应用程序所需的环境变量的值。

        REPOSITORY_NAME=azuretablestorage
        STORAGE_NAME=<storage account name>
        STORAGE_KEY=<primary access key>

    当您**开始调试**时，这便会设置环境变量。如果您**启动但不调试**希望设置变量，还在**运行服务器命令**下设置相同值。

    或者，可以使用 Windows 控制面板来定义环境变量。如果您想要避免将凭证存储在源代码中/项目文件中，这是更好的选择。请注意，您将需要重新启动 Visual Studio 以使新环境值可用于应用程序。

1.  实施 Azure 表存储库的代码位于 **models/azuretablestorage.py** 中。请参阅[文档]，详细了解如何通过 Python 使用表服务。

1.  使用 `F5` 运行应用程序。使用“创建样本轮询”创建的轮询以及通过投票提交的数据会在 Azure 表存储中进行序列化。

1.  转到“关于”页面，验证应用程序是否在使用 **Azure 表存储库**。

  	![Web 浏览器](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleAzureTableStorageAbout.png)

## 了解 Azure 表存储

很容易地使用 Visual Studio 中的 Server Explorer 查看和编辑存储表。本部分中，我们将使用 Server Explorer 查看轮询应用程序表的内容。

> [AZURE.NOTE] 这要求安装 Azure 工具，作为 [Azure SDK for .NET] 的一部分。

1.  打开**服务器资源管理器**。展开 **Azure**、**存储**、您的存储帐户，然后展开**表**。

  	<!-- ![Server Explorer](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonServerExplorer.png) -->

1.  双击“轮询”或“选择”表，在文档窗口中查看表的内容，以及添加/删除/编辑实体。

  	<!-- ![Table Query Results](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonServerExplorerTable.png) -->

## 将 Web 应用发布到 Azure Web 应用

借助 Azure.NET SDK，你可以轻松地将 Web 应用部署到 Azure 中。

1.  在“解决方案资源管理器”中，右键单击项目节点，然后选择“发布”。

  	<!-- ![Publish Web Dialog](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonPublishWebSiteDialog.png) -->

1.  单击“导入”，选择已下载的“发布配置文件”。

	如果还没有创建 Web 应用，可以登录 [Azure 经典管理门户](https://manage.windowsazure.cn/)创建一个，然后再“仪表板”的“速览”下，下载“发布配置文件”。

1.  接受其他所有默认值，然后单击**发布**。

1.  此时，您的 Web 浏览器会自动打开已发布的 Web 应用。如果您转到“关于”页面，则会看到它使用的是**内存**存储库，而不是 **Azure 表存储库**。

    这是因为未在 Azure Web 应用实例上设置环境变量，因此它使用的是 **settings.py** 中指定的默认值。

## 配置 Web 应用实例

在此部分中，我们将配置 Web 应用实例的环境变量。

1.  在 [Azure 经典管理门户]，通过单击**浏览** > **Web Apps** > 您的 Web 应用名称来打开 Web 应用的边栏选项卡。

1.  在 Web 应用边栏选项卡中，单击**所有设置**，然后单击**应用程序设置**。

  	<!-- ![Top Menu](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonWebSiteTopMenu.png) -->

1.  向下滚动到**应用设置**部分并设置 **REPOSITORY_NAME**、**STORAGE_NAME** 和 **STORAGE_KEY** 的值（如**配置项目**部分所述）。

  	<!-- ![App Settings](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonWebSiteConfigureSettingsTableStorage.png) -->

1. 依次单击“保存”、“重启”和“浏览”。

  	<!-- ![Bottom Menu](./media/web-sites-python-ptvs-bottle-table-storage/PollsCommonWebSiteConfigureBottomMenu.png) -->

1.  您应该会看到 Web 应用使用 **Azure 表存储库**按预期方式运行。

    祝贺你！

  	![Web 浏览器](./media/web-sites-python-ptvs-bottle-table-storage/PollsBottleAzureBrowser.png)

## 后续步骤

请按照下面的链接以了解有关 Python Tools for Visual Studio、 Bottle 和 Azure 表存储的更多信息。

- [Python Tools for Visual Studio 文档]
  - [Web 项目]
  - [云服务项目]
  - [在 Azure 上进行远程调试]
- [Bottle 文档]
- [Azure 存储空间]
- [Azure SDK for Python]
- [如何从 Python 使用表存储服务]


<!--Link references-->
[Python 开发人员中心]: /develop/python/
[Azure 云服务]: /documentation/articles/cloud-services-python-ptvs/
[文档]: /documentation/articles/storage-python-how-to-use-table-storage/
[如何从 Python 使用表存储服务]: /documentation/articles/storage-python-how-to-use-table-storage/

<!--External Link references-->
[Azure 经典管理门户]: https://manage.windowsazure.cn
[Azure SDK for .NET]: /downloads/
[Python Tools for Visual Studio]: http://aka.ms/ptvs
[Python Tools 2.1 for Visual Studio]: http://go.microsoft.com/fwlink/?LinkId=517189
[Python Tools 2.1 for Visual Studio Samples VSIX]: http://go.microsoft.com/fwlink/?LinkId=517189
[Python Tools 2.2 for Visual Studio]: http://go.microsoft.com/fwlink/?LinkID=624025
[Python Tools 2.2 for Visual Studio 示例 VSIX]: http://go.microsoft.com/fwlink/?LinkID=624025
[Azure SDK Tools for VS 2015]: http://go.microsoft.com/fwlink/?LinkId=518003
[Azure SDK Tools for VS 2013]: http://go.microsoft.com/fwlink/?LinkId=323510
[Azure SDK Tools for VS 2012]: http://go.microsoft.com/fwlink/?LinkId=323511
[Python 2.7（32 位）]: http://go.microsoft.com/fwlink/?LinkId=517190
[Python 3.4（32 位）]: http://go.microsoft.com/fwlink/?LinkId=517191
[Python Tools for Visual Studio 文档]: http://pytools.codeplex.com/documentation
[Bottle 文档]: http://bottlepy.org/docs/dev/index.html
[在 Azure 上进行远程调试]: http://pytools.codeplex.com/wikipage?title=Features%20Azure%20Remote%20Debugging
[Web 项目]: http://pytools.codeplex.com/wikipage?title=Features%20Web%20Project
[云服务项目]: http://pytools.codeplex.com/wikipage?title=Features%20Cloud%20Project
[Azure 存储空间]: /documentation/services/storage
[Azure SDK for Python]: https://github.com/Azure/azure-sdk-for-python
 

<!---HONumber=76-->