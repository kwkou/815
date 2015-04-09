<properties linkid="manage-services-how-to-create-and-deploy-a-cloud-service" urlDisplayName="How to create and deploy" pageTitle="如何创建和部署云服务 - Azure" metaKeywords="Azure creating cloud service, deleting cloud service" description="了解如何在 Azure 中使用"快速创建"方法创建和部署云服务。" metaCanonical="" services="cloud-services" documentationCenter="" title="How to Create and Deploy a Cloud Service" authors="ryanwi" solutions="" manager="" editor="" />






#如何创建和部署云服务

Azure 管理门户为你提供两种创建和部署云服务的方法：**"快速创建"和"自定义创建"******。 

本主题介绍如何使用"快速创建"方法创建新的云服务，然后使用****"上载"功能在 Azure 中上载和部署云服务包。使用此方法时，Azure 管理门户可提供方便的链接来根据你的需要完成所有要求。如果你在创建云服务时还准备部署该云服务，则可以使用****"自定义创建"同时执行这两项操作。 

**注意：**如果你计划从 Windows Team Foundation Services (TFS) 发布云服务，请使用"快速创建"，然后从****"快速启动"或仪表板设置 TFS 发布。有关详细信息，请参阅[使用 Visual Studio Online 向 Azure 持续交付][TFSTutorialForCloudService]，或参阅**快速启动**页中的帮助。

##目录##

* [概念](#concepts)
* [准备应用程序](#prepare)
* [开始之前](#begin)
* [如何：使用"快速创建"创建云服务](#quick)
* [如何：为云服务上载证书](#uploadcertificate)
* [如何：部署云服务](#deploy)


##<a id="concepts"></a>概念##
要将应用程序部署为 Azure 中的云服务，需要以下三个组件：

>- **服务定义文件：**云服务定义文件 (.csdef) 可定义服务模型，包括角色数量。

>- **服务配置文件：**云服务配置文件 (.cscfg) 提供云服务和各个角色的配置设置，包括角色实例的数量。

>- **服务包：**服务包 (.cspkg) 包含应用程序代码和服务定义文件。

##<a id="prepare"></a>准备应用程序##
在可以部署云服务之前，必须根据你的应用程序代码创建云服务包 (.cspkg)，并创建云服务配置文件 (.cscfg)。每个云服务包都包含应用程序文件和配置。服务配置文件提供了配置设置。

Azure SDK 提供了用于准备这些必需的部署文件的工具。你可以从 [Azure 下载](http://www.windowsazure.cn/downloads/)页安装你选择用于开发应用程序代码的相应语言的 SDK。

在你导出服务包之前，三种云服务功能需要特殊的配置：

- 如果你要部署使用安全套接字层 (SSL) 进行数据加密的云服务，请为你的应用程序配置 SSL。有关详细信息，请参阅[如何在 HTTPS 终结点上配置 SSL 证书](http://msdn.microsoft.com/zh-cn/library/windowsazure/ff795779.aspx)。

- 如果要配置与角色实例的远程桌面连接，请为这些角色配置远程桌面。有关为远程访问准备服务定义文件的详细信息，请参阅[在 Azure 中为角色设置远程桌面连接](http://msdn.microsoft.com/zh-cn/library/hh124107.aspx)。

- 如果要为云服务配置详细监视，请为云服务启用 Azure Diagnostics。"最少监视"（默认监视级别）使用从角色实例（虚拟机）的主机操作系统中收集到的性能计数器数据。"详细监视"根据角色实例中的性能数据收集其他度量信息，以便对处理应用程序期间出现的问题进行进一步分析。若要了解如何启用 Azure Diagnostics，请参阅[在 Azure 中启用 Diagnostics](/zh-cn/documentation/articles/cloud-services-dotnet-diagnostics/)。

- 若要创建包含 Web 角色或辅助角色部署的云服务，必须创建服务包。如需与包相关的文件的详细信息，请参阅[为 Azure 设置云服务](http://msdn.microsoft.com/zh-cn/library/hh124108.aspx)。若要创建包文件，请参阅[将 Microsoft Azure 应用程序打包](http://msdn.microsoft.com/zh-cn/library/hh403979.aspx)。如果使用 Visual Studio 开发应用程序，请参阅[使用 Azure Tools 发布云服务](http://msdn.microsoft.com/zh-cn/library/ff683672.aspx)。

##<a id="begin"></a>开始之前##

- 如果你尚未安装 Azure SDK，请单击****"安装 Azure SDK"以打开 [Azure 下载页](http://www.windowsazure.cn/downloads/)，然后下载你选择用于开发代码的相应语言的 SDK。（你也可以稍后执行此操作。）

- 如果任何角色实例需要证书，请创建这些证书。云服务需要带有私钥的 .pfx 文件。你可以在创建和部署云服务时将这些证书上载到 Azure。有关证书的信息，请参阅[管理证书](http://msdn.microsoft.com/zh-cn/library/gg981929.aspx)。

- 如果你计划将云服务部署到地缘组，请创建地缘组。可以使用地缘组将你的云服务和其他 Azure 服务部署到某个区域中的同一位置。你可以在管理门户的"网络"区域中的"地缘组"页上创建地缘组********。有关详细信息，请参阅[在管理门户中创建地缘组](http://msdn.microsoft.com/zh-cn/library/jj156209.aspx)。


##<a id="quick"></a>如何：使用"快速创建"创建云服务##

1. 在[管理门户](http://manage.windowsazure.cn/)中，单击"新建"****>"计算"****>"云服务"****>"快速创建"****。

	![CloudServices_QuickCreate](./media/cloud-services-how-to-create-deploy/CloudServices_QuickCreate.png)

2. 在"URL"****中，输入要在公用 URL 中使用的子域名称，以便在生产部署中访问你的云服务。生产部署的 URL 格式为：: http://*myURL*.chinacloudapp.cn.

3. 在"区域或地缘组"****中，选择要在其中部署云服务的地理区域或地缘组。如果要将云服务和其他 Azure 服务部署到某个区域中的同一位置，请选择一个地缘组。

4. 单击"创建云服务"****。

	![CloudServices_Region](./media/cloud-services-how-to-create-deploy/CloudServices_Regionlist.png)

	可以在窗口底部的消息区域中监视过程的状态。

	"云服务"****区域随即打开，并显示新的云服务。当状态更改为"已创建"时，云服务创建即成功完成。

	![CloudServices_CloudServicesPage](./media/cloud-services-how-to-create-deploy/CloudServices_CloudServicesPage.png)


##<a id="uploadcertificate"></a>如何：为云服务上载证书##

1. 在[管理门户](http://manage.windowsazure.cn/)中，依次单击"云服务"、云服务的名称、"证书"********。

	![CloudServices_QuickCreate](./media/cloud-services-how-to-create-deploy/CloudServices_EmptyDashboard.png)


2. 单击"上载证书"或"上载"********。

3. 在"文件"中，使用"浏览"选择证书（.pfx 文件）********。

4. 在****"密码"中，输入证书的私钥。

5. 单击"确定"（复选标记）****。

	![CloudServices_AddaCertificate](./media/cloud-services-how-to-create-deploy/CloudServices_AddaCertificate.png)

	可以在消息区域中查看上载进度，如下所示。上载完成时，证书即添加到表中。在消息区域中，单击"确定"关闭消息。

	![CloudServices_CertificateProgress](./media/cloud-services-how-to-create-deploy/CloudServices_CertificateProgress.png)

##<a id="deploy"></a>如何：部署云服务##

1. 在[管理门户](http://manage.windowsazure.cn/)中，依次单击"云服务"、云服务的名称、"仪表板"********。

	仪表板将在生产环境中打开，此时，你可以选择"过渡"以在过渡环境中部署应用程序。有关详细信息，请参阅[在 Azure 中管理部署](http://msdn.microsoft.com/zh-cn/library/gg433027.aspx)。

	 
2. 单击"上载新的生产部署"或"上载"********。

3. 在"部署标签"****中，输入新部署的名称 - 例如 MyCloudServicev4。

3. 在****"包"中，使用"浏览"****选择要使用的服务包文件 (.cspkg)。

4. 在****"配置"中，使用"浏览"****选择要使用的服务配置文件 (.cscfg)。

5. 如果云服务将包括只具有一个实例的任何角色，请选中"即使一个或多个角色包含单个实例也进行部署"****复选框以使部署继续进行。

 如果每个角色至少具有两个实例，那么 Azure 在维护和服务更新期间只能保证他人可访问云服务的概率是 99.95%。如果需要，你可以在部署云服务后，在"缩放"****页上添加其他角色实例。有关详细信息，请参阅[服务级别协议](/support/legal/sla/)。

6. 单击"确定"（复选标记）以开始云服务部署****。

	![CloudServices_UploadaPackage](./media/cloud-services-how-to-create-deploy/CloudServices_UploadaPackage.png)
 

	可以在消息区域中监视部署状态。单击"确定"以隐藏消息。

	![CloudServices_UploadProgress](./media/cloud-services-how-to-create-deploy/CloudServices_UploadProgress.png)

###验证部署是否已成功完成##

1. 单击"仪表板"****。

	状态应显示服务"正在运行"****。

2. 在"速览"****下，单击网站 URL 以在 Web 浏览器中打开你的云服务。

[TFSTutorialForCloudService]: /zh-cn/documentation/articles/cloud-services-continuous-delivery-use-vso/

	![CloudServices_QuickGlance](./media/cloud-services-how-to-create-deploy/CloudServices_QuickGlance.png)

<!--HONumber=39-->
