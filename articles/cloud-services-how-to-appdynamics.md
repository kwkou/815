<properties linkid="manage-services-how-to-use-appdynamics" urlDisplayName="Monitor with AppDynamics" pageTitle="How to use AppDynamics with Azure" metaKeywords="" description="Learn how to use AppDynamics for Azure." metaCanonical="" services="cloud-services" documentationCenter="" title="How To Use AppDynamics for Azure" authors="ryanwi" solutions="" manager="" editor="" />

# 如何对 Azure 使用 AppDynamics

本主题介绍了如何开始对 Azure 使用 AppDynamics。

## 目录

-   [什么是 AppDynamics？][什么是 AppDynamics？]
-   [先决条件][先决条件]
-   [注册 AppDynamics 帐户][注册 AppDynamics 帐户]
-   [从 AppDynamics 下载 .NET 代理][从 AppDynamics 下载 .NET 代理]
-   [将 .NET 代理添加到 Azure 角色并修改启动设置][将 .NET 代理添加到 Azure 角色并修改启动设置]
-   [将 AppDynamics 检测的应用程序发布到 Azure][将 AppDynamics 检测的应用程序发布到 Azure]
-   [监视应用程序][监视应用程序]

## <span id="what"></span></a>什么是 AppDynamics？

AppDynamics 是一种应用程序性能监视解决方案，可帮助你：

-   确定生产环境中的问题，如用户请求缓慢和停止以及出现错误

-   故障排除和隔离此类问题的根源

AppDynamics 包含两个组件：

-   应用程序服务器代理：应用程序服务器 .NET 代理可收集你服务器中的数据。你对要监视的每个角色实例运行一个单独的代理。可以从 AppDynamics 门户下载该代理。

-   AppDynamics 控制器：该代理会将其信息发送到 Azure 上的 AppDynamics 控制器托管的服务。通过使用基于 Web 浏览器的控制台，可以登录控制器以监视、分析应用程序并对其进行故障排除。

    ![AppDynamics 图][AppDynamics 图]

## <span id="prereq"></span></a> 先决条件

-   Visual Studio 2010 或更高版本
-   要监视的 Visual Studio 解决方案
-   Azure SDK
-   Azure 帐户

## <span id="register"></span></a>注册 AppDynamics 帐户

注册适用于 Azure 的 AppDynamics 帐户：

1.  在 Azure Marketplace ([][]<https://datamarket.azure.com/browse/Applications></a>) 上，单击 AppDynamics 旁边的“免费试用”或“注册”。

    如果你选择“注册”，则将收到免费的全功能版 AppDynamics Pro for Azure，它将在 30 天后降级为功能有限的免费版 AppDynamics Lite for Azure。对于此选项，你无需提供信用卡。你随时可以升级到 AppDynamics Pro for Azure。

    如果你选择“免费试用”，则会收到免费的全功能版 AppDynamics Pro for Azure。对于此选项，你需要提供信用卡。30 天后，除非你取消订购，否则将从你的信用卡帐户中扣除费用以便你继续使用 AppDynamics Pro for Azure。

    你需要为要监视的每个角色实例获取一个代理许可。例如，运行 2 个 Web 角色实例和 2 个辅助角色实例的网站需要 4 个代理许可。

2.  在注册页面上，提供你的用户信息、密码、电子邮件地址、公司名称以及要监视的应用程序的名称，因为你将使用 Azure 发布它。

3.  单击“立即注册”。

    将向你在注册页面上提供的地址发送一封电子邮件，其中包括你的 AppDynamics 凭据和分配给你的帐户的 AppDynamics 控制器 URL（主机和端口）。请保存此信息。

    如果你已拥有其他产品的 AppDynamics 凭据，则可使用这些凭据进行登录。

    还将为你提供一个 AppDynamics 帐户主页。

    你将登录到 AppDynamics 帐户主页。

    你的 AppDynamics 帐户主页包括：

    -   控制器 URL：从其登录到你在 AppDynamics 控制器托管的服务中拥有的帐户

    -   AppDynamics 凭据：帐户名和访问密钥

    -   AppDynamics 下载网站的链接：从其下载 AppDynamics .NET 代理

    -   Pro 试用期的剩余天数

    -   指向 AppDynamics 现有视频和文档的链接

    你可以通过在 Web 浏览器中输入 AppDynamics 帐户主页的 URL 并使用你的 AppDynamics 凭据进行登录来随时访问该主页。

## <span id="download"></span></a>从 AppDynamics 下载 .NET 代理

1.  导航到 AppDynamics 下载网站。URL 位于你的欢迎电子邮件中和你的 AppDynamics 帐户主页上。

2.  使用你的 AppDynamics 帐户名和访问密钥进行登录。

3.  下载名为 AppDynamicsdotNetAgentSetup64.msi 的文件。请不要运行该文件。

## <span id="addagent"></span></a>将 .NET 代理添加到 Azure 角色并修改启动设置

此步骤将检测 Visual Studio 解决方案中的角色以通过 AppDynamics 进行监视。对 Azure 使用 AppDynamics 时，不需要执行任何传统的 Windows 向导式安装过程。

1.  在 Visual Studio 中创建新的 Azure 项目，或打开现有的 Azure 项目。

2.  如果你创建了一个新项目，请将 Web 角色和/或辅助角色项目添加到解决方案中。

3.  对于要监视的每个 Web 角色和辅助角色项目，请添加下载的 .NET 代理 .msi 文件。

    请注意，当每个角色项目拥有一个附加的 .NET 代理 .msi 时，项目中每个角色实例将需要一个单独的 .NET 代理许可。

4.  对于要监视的每个 Web 角色和辅助角色项目，请添加名为 startup.cmd 的文本文件并将以下行粘贴到该文件中：

        if defined COR_PROFILER GOTO END 
        SETLOCAL EnableExtensions 
        REM Run the agent installer 
        AppDynamicsdotNetAgentSetup64.msi AD_Agent_Environment=Azure AD_Agent_ControllerHost=%1 AD_Agent_ControllerPort=%2 AD_Agent_AccountName=%3 AD_Agent_AccessKey=%4 AD_Agent_ControllerApplication=%5 /quiet /log d:\adInstall.log  
        SHUTDOWN /r /c "Rebooting the instance after the installation of AppDynamics Monitoring Agent" /t 0 
        GOTO END   
        :END

5.  对于要监视的每个 Web 角色和辅助角色，为 AppDynamics 代理 .msi 文件设置“复制到输出目录”属性，并为 startup.cmd 文件设置“始终复制”。

    ![始终复制][始终复制]

6.  在 Azure 项目的 ServiceDefinition.csdef 文件中，添加一个“Startup Task”元素，该元素将为每个 WorkerRole 和 WebRole 元素调用带参数的 startup.cmd。

    添加以下行：

        <Startup>
        <Task commandLine="startup.cmd [your_controller_host] [your_controller_port] [your_account_name] [your_access_key] [your_application_name]" executionContext="elevated" taskType="simple"/>
        </Startup>

    其中：

    -   *your controller host* 和 *your controller port* 是分配给你的帐户的控制器主机和端口，*your account name* 和 *your access key* 是 AppDynamics 分配给你的凭据。此信息位于你注册 AppDynamics 时收到的电子邮件中和你的 AppDynamic 主页上。请参阅[注册 AppDynamics 帐户][注册 AppDynamics 帐户]。

    -   *your application name* 是你为应用程序选择的名称。此名称用于在 AppDynamics 控制器界面中标识应用程序。

    你的 ServiceDefinition.csdef 文件将与下面类似：

    ![服务定义][服务定义]

## <a name="publish"></a>将 AppDynamics 检测的应用程序发布到 Azure

对于每个 AppDynamics 检测的角色项目：

1.  在 Visual Studio 中，选择角色项目。

2.  选择“发布到 Azure”。

## <a name="monitor"></a>监视应用程序

1.  通过欢迎电子邮件中和 AppDynamics 帐户主页上提供的 URL 登录到 AppDynamics 控制器。

2.  向你的应用程序发送一些请求，这会产生一些要监视的流量且需等待几分钟。

3.  在 AppDynamics 控制器中，选择你的应用程序。

4.  监视应用程序。

## <a name="learn"></a>了解详细信息

请参见你的 AppDynamics 帐户主页，获取指向文档和视频的链接。

本文档的最新更新包含在 wiki 版本中，网址为：[][1]<http://docs.appdynamics.com/display/ADAZ/How+To+Use+AppDynamics+for+Windows+Azure></a>。

  [什么是 AppDynamics？]: #what
  [先决条件]: #prereq
  [注册 AppDynamics 帐户]: #register
  [从 AppDynamics 下载 .NET 代理]: #download
  [将 .NET 代理添加到 Azure 角色并修改启动设置]: #addagent
  [将 AppDynamics 检测的应用程序发布到 Azure]: #publish
  [监视应用程序]: #monitor
  [AppDynamics 图]: ./media/cloud-services-how-to-appdynamics/addiagram.png
  []: https://datamarket.azure.com/browse/Applications
  [始终复制]: ./media/cloud-services-how-to-appdynamics/adcopyalways.png
  [服务定义]: ./media/cloud-services-how-to-appdynamics/adscreen.png
  [1]: http://docs.appdynamics.com/display/ADAZ/How+To+Use+AppDynamics+for+Windows+Azure
