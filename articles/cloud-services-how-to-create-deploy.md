<properties
	pageTitle="如何创建和部署云服务 | Azure"
	description="了解如何在 Azure 中使用“快速创建”方法创建和部署云服务。"
	services="cloud-services"
	documentationCenter=""
	authors="Thraka"
	manager="timlt"
	editor=""/>

<tags
	ms.service="cloud-services"
	ms.date="04/15/2016"
	wacn.date="05/31/2016"/>




# 如何创建和部署云服务

> [AZURE.SELECTOR]
- [Azure 管理门户](/documentation/articles/cloud-services-how-to-create-deploy/)

Azure 管理门户为你提供两种创建和部署云服务的方法：“快速创建”和“自定义创建”。

本主题介绍如何使用“快速创建”方法创建新的云服务，然后使用“上载”在 Azure 中上载和部署云服务包。使用此方法时，Azure 管理门户在你进行操作时将提供方便的链接供你完成所有要求。如果你在创建云服务后已准备好对其进行部署，则可以使用“自定义创建”同时执行这两项操作。

> [AZURE.NOTE] 如果你计划从 Visual Studio Team Services (VSTS) 发布云服务，请使用“快速创建”，然后从“快速启动”或仪表板设置 VSTS 发布。有关详细信息，请参阅[使用 Visual Studio Team Services 向 Azure 持续传送项目][TFSTutorialForCloudService]，或查看“快速启动”页的帮助。

## 概念
要将应用程序部署为 Azure 中的云服务，需要以下三个组件：

- **服务定义**：  
  云服务定义文件 (.csdef) 定义服务模型，包括角色数量。

- **服务配置**：  
  云服务配置文件 (.cscfg) 为云服务和各个角色提供配置设置，包括角色实例的数量。

- **服务包**：  
  服务包 (.cspkg) 包含应用程序代码和配置以及服务定义文件。
  
你可以通过[此处](/documentation/articles/cloud-services-model-and-package/)了解有关这些内容以及如何创建包的详细信息。

## 准备应用程序
在可以部署云服务之前，必须根据你的应用程序代码创建云服务包 (.cspkg)，并创建云服务配置文件 (.cscfg)。Azure SDK 提供了用于准备这些必需的部署文件的工具。你可以从 [Azure 下载](/downloads)页安装 SDK，并使用你选择用于开发应用程序代码的语言。

在你导出服务包之前，三种云服务功能需要特殊的配置：

- 如果你要部署使用安全套接字层 (SSL) 进行数据加密的云服务，请[为应用程序配置 SSL](/documentation/articles/cloud-services-configure-ssl-certificate/#step-2-modify-the-service-definition-and-configuration-files)。

- 如果要配置与角色实例的远程桌面连接，请[为这些角色配置远程桌面](/documentation/articles/cloud-services-role-enable-remote-desktop/)。

- 如果要为云服务配置详细监视，请为云服务启用 Azure 诊断。“最少监视”（默认监视级别）使用从角色实例（虚拟机）的主机操作系统中收集到的性能计数器。“详细监视”根据角色实例中的性能数据收集其他度量信息，以便对处理应用程序期间出现的问题进行进一步分析。若要了解如何启用 Azure 诊断，请参阅[在 Azure 中启用诊断](/documentation/articles/cloud-services-dotnet-diagnostics/)。

要使用 Web 角色或辅助角色创建云服务，你必须[创建服务包](/documentation/articles/cloud-services-model-and-package/#servicepackagecspkg)。

## 开始之前

- 如果你尚未安装 Azure SDK，请单击“安装 Azure SDK”以打开[ Azure 下载页](/downloads)，然后下载你选择用于开发代码的相应语言的 SDK。（你也可以稍后执行此操作。）

- 如果任何角色实例需要证书，请创建这些证书。云服务需要带有私钥的 .pfx 文件。你可以在创建和部署云服务时[将这些证书上载到 Azure](/documentation/articles/cloud-services-configure-ssl-certificate/#step-3-upload-a-certificate)。

- 如果你计划将云服务部署到关联组，请创建关联组。可以使用关联组将你的云服务和其他 Azure 服务部署到某个区域中的同一位置。你可以在 Azure 管理门户的“网络”区域中的“地缘组”页上创建地缘组。


## 如何：使用“快速创建”创建云服务

1. 在 [Azure 管理门户](http://manage.windowsazure.cn)中，单击“新建”>“计算”>“云服务”>“快速创建”。

	![云服务\_快速创建](./media/cloud-services-how-to-create-deploy/CloudServices_QuickCreate.png)

2. 在“URL”中，输入要在公用 URL 中使用的子域名称，以便在生产部署中访问你的云服务。生产部署的 URL 格式为：http://*myURL*.chinacloudapp.cn。

3. 在“区域或地缘组”中，选择要在其中部署云服务的地理区域或地缘组。如果要将云服务和其他 Azure 服务部署到某个区域中的同一位置，请选择一个地缘组。

4. 单击“创建云服务”。

	![云服务\_区域](./media/cloud-services-how-to-create-deploy/CloudServices_Regionlist.png)

	可以在窗口底部的消息区域中监视过程的状态。

	“云服务”区域随即打开，并显示新的云服务。当状态更改为“已创建”时，云服务创建即成功完成。

	![云服务\_云服务页](./media/cloud-services-how-to-create-deploy/CloudServices_CloudServicesPage.png)


## 如何：为云服务上载证书

1. 在 [Azure 管理门户](http://manage.windowsazure.cn)中单击“云服务”，单击云服务的名称，然后单击“证书”。

	![云服务\_快速创建](./media/cloud-services-how-to-create-deploy/CloudServices_EmptyDashboard.png)


2. 单击“上载证书”或“上载”。

3. 在“文件”中，使用“浏览”选择证书（.pfx 文件）。

4. 在“密码”中，输入证书的私钥。

5. 单击“确定”（复选标记）。

	![云服务\_添加证书](./media/cloud-services-how-to-create-deploy/CloudServices_AddaCertificate.png)

	可以在消息区域中查看上载进度，如下所示。上载完成时，证书即添加到表中。在消息区域中，单击“确定”关闭消息。

	![云服务\_证书进度](./media/cloud-services-how-to-create-deploy/CloudServices_CertificateProgress.png)

## 如何：部署云服务

1. 在 [Azure 管理门户](http://manage.windowsazure.cn)中单击“云服务”，单击云服务的名称，然后单击“仪表板”。

2. 单击“上载新的生产部署”或“上载”。

3. 在“部署标签”中，输入新部署的名称 - 例如 MyCloudServicev4。

3. 在“包”中，使用“浏览”选择要使用的服务包文件 (.cspkg)。

4. 在“配置”中，使用“浏览”选择要使用的服务配置文件 (.cscfg)。

5. 如果云服务将包括只具有一个实例的任何角色，请选中“即使一个或多个角色包含单个实例也进行部署”复选框以使部署继续进行。

    如果每个角色至少具有两个实例，那么 Azure 在维护和服务更新期间只能保证他人可访问云服务的概率是 99.95%。如果需要，你可以在部署云服务后，在“缩放”页上添加其他角色实例。有关详细信息，请参阅[服务级别协议](/support/legal/sla)。

6. 单击“确定”（复选标记）以开始云服务部署。

	![云服务\_上载包](./media/cloud-services-how-to-create-deploy/CloudServices_UploadaPackage.png)

	可以在消息区域中监视部署状态。单击“确定”隐藏消息。

	![云服务\_上载过程](./media/cloud-services-how-to-create-deploy/CloudServices_UploadProgress.png)

## 验证确认你的部署已成功完成

1. 单击“仪表板”。

	该状态应该显示该服务**正在运行**。

2. 在“速览”下，单击网站 URL 以在 Web 浏览器中打开你的云服务。

    ![云服务\_速览](./media/cloud-services-how-to-create-deploy/CloudServices_QuickGlance.png)

 
## 后续步骤

* [云服务的常规配置](/documentation/articles/cloud-services-how-to-configure/)。
* [配置自定义域名](/documentation/articles/cloud-services-custom-domain-name/)。
* [管理云服务](/documentation/articles/cloud-services-how-to-manage/)。
* 配置 [SSL 证书](/documentation/articles/cloud-services-configure-ssl-certificate/)。

<!---HONumber=Mooncake_0523_2016-->
