<properties linkid="manage-services-how-to-create-and-deploy-a-cloud-service" urlDisplayName="How to create and deploy" pageTitle="How to create and deploy a cloud service - Azure" metaKeywords="Azure creating cloud service, deleting cloud service" description="Learn how to create and deploy a cloud service using the Quick Create method in Azure." metaCanonical="" services="cloud-services" documentationCenter="" title="How to Create and Deploy a Cloud Service" authors="ryanwi" solutions="" manager="" editor="" />

# 如何创建和部署云服务

[WACOM.INCLUDE [免责声明][免责声明]]

Azure 管理门户为你提供两种创建和部署云服务的方法：“快速创建”和“自定义创建”。

本主题介绍如何使用“快速创建”方法创建新的云服务，然后使用“上载”功能在 Azure 中上载和部署云服务包。使用此方法时，Azure 管理门户可提供方便的链接来根据你的需要完成所有要求。如果你在创建云服务时还准备部署该云服务，则可以使用“自定义创建”同时执行这两项操作。

**注意**：如果你计划从 Windows Team Foundation Services (TFS) 发布云服务，请使用“快速创建”，然后从“快速启动”或仪表板设置 TFS 发布。有关详细信息，请参阅[使用 Team Foundation Service 预览版向 Azure 持续传送项目][使用 Team Foundation Service 预览版向 Azure 持续传送项目]，或查看“快速启动”页中的帮助。

## 目录

-   [概念][概念]
-   [准备应用程序][准备应用程序]
-   [开始之前][开始之前]
-   [如何：使用“快速创建”创建云服务][如何：使用“快速创建”创建云服务]
-   [如何：为云服务上载证书][如何：为云服务上载证书]
-   [如何：部署云服务][如何：部署云服务]

## <span id="concepts"></span></a>概念

要将应用程序部署为 Azure 中的云服务，需要以下三个组件：

> -   **服务定义文件**：云服务定义文件 (.csdef) 可定义服务模型，包括角色数量。

> -   **服务配置文件**：云服务配置文件 (.cscfg) 提供云服务和各个角色的配置设置，包括角色实例的数量。

> -   **服务包**：服务包 (.cspkg) 包含应用程序代码和服务定义文件。

## <span id="prepare"></span></a>准备应用程序

在可以部署云服务之前，必须根据你的应用程序代码创建云服务包 (.cspkg)，并创建云服务配置文件 (.cscfg)。每个云服务包都包含应用程序文件和配置。服务配置文件提供了配置设置。

Azure SDK 提供了用于准备这些必需的部署文件的工具。你可以从 [Azure 下载][Azure 下载]页安装你选择用于开发应用程序代码的相应语言的 SDK。

如果你是初次接触云服务，可从 [Azure 代码示例][Azure 代码示例]下载云服务包 (.cspkg) 和服务配置文件 (.cscfg) 示例。

在你导出服务包之前，三种云服务功能需要特殊的配置：

-   如果你要部署使用安全套接字层 (SSL) 进行数据加密的云服务，请为你的应用程序配置 SSL。有关详细信息，请参阅[如何在 HTTPS 终结点上配置 SSL 证书][如何在 HTTPS 终结点上配置 SSL 证书]。

-   如果要配置与角色实例的远程桌面连接，请为这些角色配置远程桌面。有关为远程访问准备服务定义文件的详细信息，请参阅[为角色设置远程桌面连接概述][为角色设置远程桌面连接概述]。

-   如果要为云服务配置详细监视，请为云服务启用 Azure 诊断。“最少监视”（默认监视级别）使用从角色实例（虚拟机）的主机操作系统中收集到的性能计数器数据。“详细监视”根据角色实例中的性能数据收集其他度量信息，以便对处理应用程序期间出现的问题进行进一步分析。若要了解如何启用 Azure 诊断，请参阅[在 Azure 中启用诊断][在 Azure 中启用诊断]。

## <span id="begin"></span></a>开始之前

-   如果你尚未安装 Azure SDK，请单击“安装 Azure SDK”以打开 [Azure 下载页][Azure 下载]，然后下载你选择用于开发代码的相应语言的 SDK。（你也可以稍后执行此操作。）

-   如果任何角色实例需要证书，请创建这些证书。云服务需要带有私钥的 .pfx 文件。你可以在创建和部署云服务时将这些证书上载到 Azure。有关创建证书的信息，请参阅[如何在 HTTPS 终结点上配置 SSL 证书][如何在 HTTPS 终结点上配置 SSL 证书]。

-   如果你计划将云服务部署到地缘组，请创建地缘组。可以使用地缘组将你的云服务和其他 Azure 服务部署到某个区域中的同一位置。你可以在管理门户的“网络”区域中的“地缘组”页上创建地缘组。有关详细信息，请参阅“地缘组”页上的帮助。

## <span id="quick"></span></a>如何：使用“快速创建”创建云服务

1.  在[管理门户][管理门户]中，依次单击“新建”、“云服务”和“快速创建”。

    ![云服务\_快速创建][云服务\_快速创建]

2.  在“URL”中，输入要在公用 URL 中使用的子域名称，以便在生产部署中访问你的云服务。生产部署的 URL 格式为：http://*myURL*.cloudapp.net。

3.  在“区域/地缘组”中，选择要在其中部署云服务的地理区域或地缘组。如果要将云服务和其他 Azure 服务部署到某个区域中的同一位置，请选择一个地缘组。

    > [WACOM.NOTE]
    > 若要创建地缘组，请打开管理门户的“网络”区域，单击“地缘组”，然后单击“创建新的地缘组”或“创建”。可以使用你在前面的 Azure 管理门户中创建的地缘组，也可以使用 Azure 服务管理 API 创建和管理地缘组。有关详细信息，请参阅[针对地缘组的操作][针对地缘组的操作]。

4.  单击“创建云服务”。

    可以在窗口底部的消息区域中监视过程的状态。

    “云服务”区域随即打开，并显示新的云服务。当状态更改为“已创建”时，云服务创建即成功完成。

    ![云服务\_云服务页][云服务\_云服务页]

    如果云服务中的任何角色需要安全套接字层 (SSL) 数据加密证书，并且该证书尚未上载到 Azure，则你必须先上载该证书才能部署云服务。上载证书后，在角色实例中运行的任何 Windows 应用程序都可以访问该证书。

## <span id="uploadcertificate"></span></a>如何：为云服务上载证书

1.  在[管理门户][管理门户]中单击“云服务”。然后单击云服务的名称以打开仪表板。

    ![云服务\_空仪表板][云服务\_空仪表板]

2.  单击“证书”以打开“证书”页，如下所示。

    ![云服务\_证书页][云服务\_证书页]

3.  单击“添加新证书”或“上载”。
    “添加证书”随即打开。

    ![云服务\_添加证书][云服务\_添加证书]

4.  在“证书文件”中，使用“浏览”选择要使用的证书（.pfx 文件）。

5.  在“密码”中，输入证书的私钥。

6.  单击“确定”（复选标记）。

    可以在消息区域中查看上载进度，如下所示。上载完成时，证书即添加到表中。在消息区域中，单击向下箭头以关闭消息，或单击 X 删除消息。

    ![云服务\_证书进度][云服务\_证书进度]

    可以从仪表板或从“快速启动”部署你的云服务。

## <span id="deploy"></span></a>如何：部署云服务

1.  在[管理门户][管理门户]中单击“云服务”。然后单击云服务的名称以打开仪表板。

2.  单击“快速启动”（“仪表板”左侧的图标）打开“快速启动”页，如下所示。（也可以使用仪表板上的“上载”部署你的云服务。）

    ![云服务\_快速启动页][云服务\_快速启动页]

3.  如果你尚未安装 Azure SDK，请单击“安装 Azure SDK”以打开 [Azure 下载页][Azure 下载]，然后下载你选择用于开发代码的相应语言的 SDK。

    在下载页上，你还可以安装客户端库和源代码，以使用 Node.js、Java、PHP 和其他语言开发可部署为可缩放的 Azure 云服务的 Web 应用程序。

    > [WACOM.NOTE]
    > 对于以前创建的云服务（以前称为*托管服务*），你将需要确保虚拟机（角色实例）上的来宾操作系统与所安装的 Azure SDK 版本兼容。有关详细信息，请参阅 [Azure SDK 发行说明][Azure SDK 发行说明]。

4.  单击“新建生产部署”或“新建过渡部署”。

    如果你希望在将云服务部署到生产环境之前先在 Azure 中测试该云服务，则可以将其部署到过渡环境。在过渡环境中，使用云服务的全局唯一标识符 (GUID) 在 URL 中标识该云服务 (*GUID*.cloudapp.net)。在生产环境中，将使用你所分配的更加友好的 DNS 前缀（例如，*myservice*.cloudapp.net）。在准备好将处于过渡环境的云服务升级到生产环境时，可使用“交换”将客户端请求重定向到该部署。

    当你选择部署环境时，会打开“上载包”。

    ![云服务\_上载包][云服务\_上载包]

5.  在“部署名称”中，输入新部署的名称 - 例如 MyCloudServicev1。

6.  在“包”中，使用“浏览”选择要使用的服务包文件 (.cspkg)。

7.  在“配置”中，使用“浏览”选择要使用的服务配置文件 (.cscfg)。

8.  如果云服务将包括只具有一个实例的任何角色，请选中“即使一个或多个角色包含单个实例也进行部署”复选框以使部署继续进行。

 如果每个角色至少具有两个实例，那么 Azure 在维护和服务更新期间只能保证他人可访问云服务的概率是 99.95%。如果需要，你可以在部署云服务后，在“缩放”页上添加其他角色实例。有关详细信息，请参阅[服务级别协议][服务级别协议]。

9.  单击“确定”（复选标记）以开始云服务部署。

    可以在消息区域中监视部署状态。单击向下箭头可隐藏消息。

    ![云服务\_上载过程][云服务\_上载过程]

### 验证你的部署是否已成功完成

1.  单击**“仪表板”**。

2.  在“速览”下，单击网站 URL 以在 Web 浏览器中打开你的云服务。

    ![CloudServices_QuickGlance](./media/cloud-services-how-to-create-deploy/CloudServices_QuickGlance.png)

  [免责声明]: ../includes/disclaimer.md
  [使用 Team Foundation Service 预览版向 Azure 持续传送项目]: http://go.microsoft.com/fwlink/?LinkID=251796&clcid=0x409
  [概念]: #concepts
  [准备应用程序]: #prepare
  [开始之前]: #begin
  [如何：使用“快速创建”创建云服务]: #quick
  [如何：为云服务上载证书]: #uploadcertificate
  [如何：部署云服务]: #deploy
  [Azure 下载]: http://www.windowsazure.com/zh-cn/develop/downloads/
  [Azure 代码示例]: http://code.msdn.microsoft.com/windowsazure/
  [如何在 HTTPS 终结点上配置 SSL 证书]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ff795779.aspx
  [为角色设置远程桌面连接概述]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433010.aspx
  [在 Azure 中启用诊断]: http://www.windowsazure.com/zh-cn/develop/net/common-tasks/diagnostics/
  [管理门户]: http://manage.windowsazure.cn/
  [云服务\_快速创建]: ./media/cloud-services-how-to-create-deploy/CloudServices_QuickCreate.png
  [针对地缘组的操作]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460798.aspx
  [云服务\_云服务页]: ./media/cloud-services-how-to-create-deploy/CloudServices_CloudServicesPage.png
  [云服务\_空仪表板]: ./media/cloud-services-how-to-create-deploy/CloudServices_EmptyDashboard.png
  [云服务\_证书页]: ./media/cloud-services-how-to-create-deploy/CloudServices_CertificatesPage.png
  [云服务\_添加证书]: ./media/cloud-services-how-to-create-deploy/CloudServices_AddaCertificate.png
  [云服务\_证书进度]: ./media/cloud-services-how-to-create-deploy/CloudServices_CertificateProgress.png
  [云服务\_快速启动页]: ./media/cloud-services-how-to-create-deploy/CloudServices_QuickStartPage.png
  [Azure SDK 发行说明]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh552718.aspx
  [云服务\_上载包]: ./media/cloud-services-how-to-create-deploy/CloudServices_UploadaPackage.png
  [服务级别协议]: http://www.windowsazure.com/zh-cn/support/legal/sla/
  [云服务\_上载过程]: ./media/cloud-services-how-to-create-deploy/CloudServices_UploadProgress.png
