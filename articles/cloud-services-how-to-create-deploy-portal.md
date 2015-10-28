<properties
	pageTitle="如何创建和部署云服务 | Windows Azure"
	description="了解如何在 Azure 中使用“快速创建”方法创建和部署云服务。"
	services="cloud-services"
	documentationCenter=""
	authors="Thraka"
	manager="timlt"
	editor=""/>

<tags
	ms.service="cloud-services"
	ms.date="06/30/2015"
	wacn.date="10/17/2015"/>




# 如何创建和部署云服务

> [AZURE.SELECTOR]
- [Azure Portal](/documentation/articles/cloud-services-how-to-create-deploy)
- [Azure Portal](/documentation/articles/cloud-services-how-to-create-deploy-portal)

Azure 门户为你提供两种创建和部署云服务的方法：“快速创建”和“自定义创建”。

本文介绍如何使用“快速创建”方法创建新的云服务，然后使用“上载”在 Azure 中上载和部署云服务包。使用此方法时，Azure 门户在你进行操作时将提供方便的链接供你完成所有要求。如果你在创建云服务时还准备部署该云服务，则可以使用“自定义创建”同时执行这两项操作。

> [AZURE.NOTE]如果你计划从 Visual Studio Online (VSO) 发布云服务，请使用“快速创建”，然后从“Azure 快速启动”或仪表板设置 VSO 发布。有关详细信息，请参阅[使用 Visual Studio Online 向 Azure 持续交付][TFSTutorialForCloudService]，或查看“快速启动”页的帮助。

## 概念
要将应用程序部署为 Azure 中的云服务，需要以下三个组件：

- **服务定义**：云服务定义文件 (.csdef) 定义服务模型，包括角色数量。

- **服务配置**：云服务配置文件 (.cscfg) 为云服务和各个角色提供配置设置，包括角色实例的数量。

- **服务包**：服务包 (.cspkg) 包含应用程序代码和配置以及服务定义文件。

您可以通过[此处](/documentation/articles/cloud-services-model-and-package)了解有关这些内容以及如何创建包的详细信息。

## 准备应用程序
在可以部署云服务之前，必须根据你的应用程序代码创建云服务包 (.cspkg)，并创建云服务配置文件 (.cscfg)。Azure SDK 提供了用于准备这些必需的部署文件的工具。你可以从 [Azure 下载](http://azure.microsoft.com/downloads/)页安装 SDK，并使用你选择用于开发应用程序代码的语言。

在你导出服务包之前，三种云服务功能需要特殊的配置：

- 如果你要部署使用安全套接字层 (SSL) 进行数据加密的云服务，请为你的应用程序配置 SSL。有关详细信息，请参阅[如何在 HTTPS 终结点上配置 SSL 证书](https://msdn.microsoft.com/zh-cn/library/azure/ff795779.aspx)。

- 如果要配置与角色实例的远程桌面连接，请为这些角色配置远程桌面。有关为远程访问准备服务定义文件的详细信息，请参阅[在 Azure 中为角色设置远程桌面连接](https://msdn.microsoft.com/zh-cn/library/hh124107.aspx)。

- 如果要为云服务配置详细监视，请为云服务启用 Azure 诊断。*“最少监视”*（默认监视级别）使用从角色实例（虚拟机）的主机操作系统中收集到的性能计数器。“详细监视”根据角色实例中的性能数据收集其他度量信息，以便对处理应用程序期间出现的问题进行进一步分析。若要了解如何启用 Azure 诊断，请参阅[在 Azure 中启用诊断](/documentation/articles/cloud-services-dotnet-diagnostics)。

- 要使用 Web 角色或辅助角色创建云服务，你必须创建服务包。有关与包相关的文件的详细信息，请参阅[设置 Azure 的云服务](http://msdn.microsoft.com/zh-cn/library/hh124108.aspx)。若要创建包文件，请参阅[打包 Azure 应用程序](http://msdn.microsoft.com/zh-cn/library/hh403979.aspx)。如果使用的 Visual Studio 开发应用程序，请参阅[使用 Azure 工具发布云服务](http://msdn.microsoft.com/zh-cn/library/ff683672.aspx)。

## 开始之前

- 如果你尚未安装 Azure SDK，请单击“安装 Azure SDK”以打开[ Azure 下载页](/downloads/)，然后下载你选择用于开发代码的相应语言的 SDK。（你也可以稍后执行此操作。）

- 如果任何角色实例需要证书，请创建这些证书。云服务需要带有私钥的 .pfx 文件。你可以在创建和部署云服务时将这些证书上载到 Azure。有关证书的详细信息，请参阅[管理证书](http://msdn.microsoft.com/zh-cn/library/gg981929.aspx)。

- 如果你计划将云服务部署到关联组，请创建关联组。可以使用关联组将你的云服务和其他 Azure 服务部署到某个区域中的同一位置。你可以在**“关联组”**页上的管理门户的**“网络”**区域中创建地缘组。获取详细信息。


## 步骤 3：创建云服务并上载部署包

1. 登录到 [Azure 门户][]。 
2. 单击“新建”>“计算”，然后向下滚动到“云服务”并单击它。

    ![发布云服务](media/cloud-services-how-to-create-deploy-portal/create-cloud-service.png)

3. 在新的“云服务”边栏选项卡中，输入 **DNS 名称**的值。
4. 创建一个新**资源组**或选择一个现有的资源组。
5. 选择“位置”。
6. 选择“包”，然后在“上载包”边栏选项卡上，填写必填字段。  

     如果你的任何角色包含单个实例，请确保选中“即使一个或多个角色包含单个实例也进行部署”。

7. 请确保选中“开始部署”。
8. 单击**“确定”**。

    ![发布云服务](./media/cloud-services-how-to-create-deploy-portal/select-package.png)

## 上载证书

如果你的部署包已[配置为使用证书](/documentation/articles/cloud-services-configure-ssl-certificate-portal#modify)，你现在就可以上载证书。

1. 选择“证书”，并在“添加证书”边栏选项卡中，选择 SSL 证书 .pfx 文件，然后提供证书的**密码**，
2. 单击“附加证书”，然后单击“添加证书”边栏选项卡上的“确定”。
3. 单击“云服务”边栏选项卡上的“创建”。当部署达到**“就绪”**状态时，你可以继续执行后续步骤。

    ![发布云服务](./media/cloud-services-how-to-create-deploy-portal/attach-cert.png)


## 验证确认你的部署已成功完成

1. 单击云服务实例

	该状态应该显示该服务**正在运行**。

2. 在**必备**下，单击**站点 URL** 以在 Web 浏览器中打开你的云服务。

    ![云服务\_速览](./media/cloud-services-how-to-create-deploy-portal/running.png)


[TFSTutorialForCloudService]: http://go.microsoft.com/fwlink/?LinkID=251796&clcid=0x409
 

<!---HONumber=74-->