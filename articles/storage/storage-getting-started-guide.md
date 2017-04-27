<properties
    pageTitle="五分钟内开始使用 Azure 存储 | Azure"
    description="使用 Azure 存储快速启动、Visual Studio 和 Azure 存储模拟器快速掌握 Azure Blob、表和队列。 在五分钟内运行你的第一个 Azure 存储空间应用程序。"
    services="storage"
    documentationcenter=".net"
    author="mmacy"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="582f76f8-c814-4a69-8a5c-1fd0e0d5d8f2"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="get-started-article"
    ms.date="02/23/2017"
    wacn.date="04/24/2017"
    ms.author="marsma"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="0a9207a7264d2ead5017f6de6de3e74b7a5ed41f"
    ms.lasthandoff="04/14/2017" />

# <a name="get-started-with-azure-storage-in-five-minutes"></a>五分钟内开始使用 Azure 存储空间
## <a name="overview"></a>概述
通过 Azure 存储空间进行开发很容易入门。 本教程向你说明如何构建一个 Azure 存储空间应用程序并快速运行。 你将使用用于 .NET 的 Azure SDK 随附的快速启动模板。 这些快速启动模板包含可以立刻运行的代码，用于演示部分通过 Azure 存储空间进行的基本编程方案。

在深入分析代码之前，若要详细了解 Azure 存储，请参阅[后续步骤](#next-steps)。

## <a name="prerequisites"></a>先决条件
在开始之前，你需要符合以下先决条件：

1. 若要编译和生成应用程序，需在计算机上安装 [Visual Studio 2015](https://www.visualstudio.com/) 或更高版本。
2. 安装最新版的[用于 .NET 的 Azure SDK](/downloads/)。 SDK 包括 Azure 快速入门示例项目、Azure 存储模拟器和 [用于 .NET 的 Azure 存储空间客户端库](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)。
3. 确保在你的计算机上安装了 [.NET Framework 4.5](http://www.microsoft.com/download/details.aspx?id=30653) ，因为我们将在本教程中使用的 Azure 快速入门示例项目需要它。

    如果你不确定计算机上安装了哪个版本的 .NET Framework，请参阅 [如何：确定安装的 .NET Framework 版本](https://msdn.microsoft.com/vstudio/hh925568.aspx)。 或者，按“开始”按钮或 Windows 键，并键入“控制面板”。 然后，单击“程序”  >  “程序和功能”，然后在已安装程序中确定是否列出 .NET Framework 4.5。
4. 你将需要 Azure 订阅和 Azure 存储帐户。

    - 若要获取 Azure 订阅，请参阅 [1 元试用](/pricing/1rmb-trial/)、[购买选项](/pricing/overview/)。
    - 若要在 Azure 中创建存储帐户，请参阅 [如何创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)。

## <a name="run-your-first-azure-storage-application-against-azure-storage-in-the-cloud"></a>使用 Azure 存储模拟器在云中运行你的第一个 Azure 存储空间应用程序
拥有帐户之后，即可以在 Visual Studio 中使用 Azure 快速启动示例项目之一创建一个简单的 Azure 存储空间应用程序。 本教程重点介绍 Azure 存储空间的示例项目：**Azure 存储： Blobs**、**Azure 存储：文件**、**Azure 存储：队列**以及 **Azure 存储：表**：

1. 启动 Visual Studio。

2. 在“文件”菜单上，单击“新建项目”

3. 在“新建项目”对话框中，单击“已安装” > “模板” > “Visual C#” > “云” > “快速启动” > “数据服务”。
   
   a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。 选择以下模板之一：**Azure 存储： Blobs****Azure 存储：文件****Azure 存储：队列**或 **Azure 存储：表**
   
   b.保留“数据库类型”设置，即设置为“共享”。 确保选择 **.NET Framework 4.5** 作为目标框架。
   
   c. 为你的项目指定一个名称并创建新的 Visual Studio 解决方案，如下所示：

   ![Azure 快速启动][Image1]

你可能想要在运行应用程序之前检查源代码。 若要查看代码，请在 Visual Studio 中的“查看”菜单上选择“解决方案资源管理器”。 然后双击 Program.cs 文件。

接下来，运行示例应用程序：

1. 在 Visual Studio 中的“查看”菜单上，选择“解决方案资源管理器”。 打开 **App.config** 文件并注释掉 Azure 存储模拟器的连接字符串：

    `<!--<add key="StorageConnectionString" value = "UseDevelopmentStorage=true;"/>-->`

2. 在 App.config 文件中取消注释 Azure 存储服务的连接字符串，并用帐户凭据替换 `[AccountName]` 和 `[AccountKey]`：

    `<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey];EndpointSuffix=core.chinacloudapi.cn"`

    若要检索存储帐户访问密钥，请参阅 [管理存储访问密钥](/documentation/articles/storage-create-storage-account/#manage-your-storage-access-keys)。

3. 在 App.config 文件中提供存储帐户名称和访问密钥后,在“文件” 菜单中，单击“全部保存”以保存所有项目文件。

4. 在“生成”菜单中，单击“生成解决方案”。

5. 在“调试”菜单中，按 **F11** 逐步运行该解决方案，或按 **F5** 运行该解决方案。

## <a name="run-your-first-azure-storage-application-locally-against-the-azure-storage-emulator"></a>使用 Azure 存储模拟器在本地运行你的第一个 Azure 存储空间应用程序
[Azure 存储模拟器](/documentation/articles/storage-use-emulator/) 提供了一个针对开发目的模拟 Azure Blob、队列和表服务的本地环境。 你可以使用存储模拟器在本地测试你的应用程序，而无需创建 Azure 订阅或存储帐户，并且不会产生任何费用。

若要尝试，让我们在 Visual Studio 中使用 Azure 快速启动示例项目之一创建一个简单的 Azure 存储空间应用程序。 本教程重点介绍 **Azure Blob 存储**、**Azure 表存储**和 **Azure 队列存储**示例项目：

1. 启动 Visual Studio。
2. 在“文件”菜单上，单击“新建项目”
3. 在“新建项目”对话框中，单击“已安装” > “模板” > “Visual C#” > “云” > “快速启动” > “数据服务”。
    
    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。 选择以下模板之一：**Azure 存储： Blobs****Azure 存储：文件****Azure 存储：队列**或 **Azure 存储：表**
    
    b.保留“数据库类型”设置，即设置为“共享”。 确保选择 **.NET Framework 4.5** 作为目标框架。
    
    c. 为你的项目指定一个名称并创建新的 Visual Studio 解决方案，如下所示：

    ![Azure 快速启动][Image1]

4. 在 Visual Studio 中的“查看”菜单上，选择“解决方案资源管理器”。 打开 App.config 文件并注释掉 Azure 存储帐户的连接字符串（如果已添加）。 然后取消注释 Azure 存储模拟器的连接字符串：

    `<add key="StorageConnectionString" value = "UseDevelopmentStorage=true;"/>`

你可能想要在运行应用程序之前检查源代码。 若要查看代码，请在 Visual Studio 中的“查看”菜单上选择“解决方案资源管理器”。 然后双击 Program.cs 文件。

接下来，在 Azure 存储模拟器中运行示例应用程序:

1. 按“开始”按钮或 Windows 键，然后搜索 Azure 存储模拟器并启动该应用程序。 当模拟器启动时，你将在“Windows 任务视图”区域看到图标和通知。
2. 在 Visual Studio 中，单击“生成”菜单上的“生成解决方案”。
3. 在“调试”菜单上，按 **F11** 逐步运行该解决方案，或按 **F5** 从头到尾运行该解决方案。

## <a name="next-steps"></a>后续步骤
若要了解有关 Azure 存储空间的详细信息，请参阅以下资源：

* [Azure 存储空间简介](/documentation/articles/storage-introduction/)
* [通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)
* [通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)
* [通过 .NET 开始使用 Azure 队列存储](/documentation/articles/storage-dotnet-how-to-use-queues/)
* [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)
* [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)
* [Azure 存档文档](/documentation/services/storage/)
* [适用于 .NET 的 Azure 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)
* [Azure 存储 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx)

[Image1]: ./media/storage-getting-started-guide/QuickStart.png
<!--Update_Description: Vistual Studio configure steps update to latest; add anchors to H2 titles-->
