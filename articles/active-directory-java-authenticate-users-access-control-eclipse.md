<properties linkid="develop-java-how-to-guides-access-control" urlDisplayName="Access Control" pageTitle="How to use Access Control (Java) - Azure feature guide" metaKeywords="" description="Learn how to develop and use Access Control with Java in Azure." metaCanonical="" services="active-directory" documentationCenter="Java" title="How to Authenticate Web Users with Azure Access Control Service Using Eclipse" videoId="" scriptId="" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" />

# 如何使用 Eclipse 在 Azure Access Control 服务上对 Web 用户进行身份验证

本指南将说明如何在 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）中使用 Azure Access Control 服务 (ACS)。有关 ACS 的详细信息，请参阅[后续步骤][后续步骤]部分。

> [WACOM.NOTE]
> Azure 访问服务控制筛选器（由 Microsoft Open Technologies 提供）是一个社区技术预览版。作为预发行软件，Microsoft Open Technologies, Inc. 和 Microsoft 不会为它提供正式支持。

## 目录

-   [什么是 ACS？][什么是 ACS？]
-   [概念][概念]
-   [先决条件][先决条件]
-   [创建 ACS 命名空间][创建 ACS 命名空间]
-   [添加标识提供程序][添加标识提供程序]
-   [添加信赖方应用程序][添加信赖方应用程序]
-   [创建规则][创建规则]
-   [将证书上载到 ACS 命名空间][将证书上载到 ACS 命名空间]
-   [查看“应用程序集成”页][查看“应用程序集成”页]
-   [创建 Java Web 应用程序][创建 Java Web 应用程序]
-   [将 ACS 筛选器库添加到你的应用程序][将 ACS 筛选器库添加到你的应用程序]
-   [部署到计算模拟器][部署到计算模拟器]
-   [部署到 Azure][部署到 Azure]
-   [后续步骤][后续步骤]

## <a name="what-is"></a>什么是 ACS？

大多数开发人员都不是标识专家，他们通常都不想花时间开发针对其应用程序和服务的身份验证和授权机制。ACS 是一项 Azure 服务，可以轻松地对需要访问你的 Web 应用程序和服务的用户进行身份验证，而不必将复杂的身份验证逻辑注入代码中。

ACS 具有以下可用功能：

-   与 Windows Identity Foundation (WIF) 集成。
-   支持常用 Web 标识提供程序 (IP)（包括 Windows Live ID、Google、Yahoo! 和 Facebook）。
-   支持 Active Directory 联合身份验证服务 (AD FS) 2.0。
-   一项基于开放数据协议 (OData) 的管理服务，该服务提供对 ACS 设置的编程访问。
-   一个允许对 ACS 设置进行管理访问的管理门户。

有关 ACS 的详细信息，请参阅 [Access Control 服务 2.0][Access Control 服务 2.0]。

## <a name="concepts"></a>概念

ACS 在基于声明的标识的主体的基础上构建，它是一种创建针对本地运行或在云中运行的应用程序的身份验证机制的一致性方法。通常，应用程序和服务可使用基于声明的标识来获取所需的有关其组织内、其他组织内以及 Internet 上的用户的标识信息。

若要完成本指南中的任务，你应了解以下概念：

**客户端** - 在本操作方法指南的上下文中，这是一个尝试获取对你的 Web 应用程序的访问权的浏览器。

**信赖方 (RP) 应用程序** - RP 应用程序是一个将身份验证外包给外部权威机构的网站或服务。用标识行话来说，RP 信任该权威机构。本指南说明如何将你的应用程序配置为信任 ACS。

**令牌** - 令牌是一个安全数据集合，它通常会在对用户进行身份验证成功后颁发。它包含一组*声明*（经过身份验证的用户的属性）。声明可表示用户名、用户所属角色的标识符、用户年龄等。令牌通常已进行数字签名，这意味着始终能够找到令牌的颁发者，并且其内容无法篡改。用户可通过提供由 RP 应用程序信任的颁发机构颁发的有效令牌来获取对 RP 应用程序的访问权限。

**标识提供程序 (IP)** - IP 是指验证用户标识和颁发安全令牌的权威机构。实际上，令牌颁发工作是通过一项称作“安全令牌服务 (STS)”的特殊服务完成的。IP 的典型示例包括 Windows Live ID、Facebook、业务用户存储库（如 Active Directory）等。在将 ACS 配置为信任某个 IP 后，系统将接受并验证由该 IP 颁发的令牌。ACS 可一次性信任多个 IP，这意味着，如果你的应用程序信任 ACS，则你可立即将你的应用程序提供给 ACS 代表你信任的全部 IP 中的所有经过身份验证的用户。

**联合提供程序 (FP)** - IP 了解用户的直接信息，使用用户凭据对用户进行身份验证并发布其了解的用户相关信息。联合提供程序 (FP) 是一种不同类型的权威机构：它不会直接对用户进行身份验证，而是充当一个 RP 与一个或多个 IP 之间的中介和代理身份验证。IP 和 FP 都颁发安全令牌，因此它们都将使用安全令牌服务 (STS)。ACS 是一个 FP。

**ACS 规则引擎** - 将以简单声明转换规则的形式整理用于将来自受信任 IP 的传入令牌转换为将由 RP 使用的令牌的逻辑。ACS 包含一个规则引擎，该引擎负责应用你为 RP 指定的任何转换逻辑。

**ACS 命名空间** - 命名空间是 ACS 中的一个顶级分区，用于整理你的设置。命名空间包含一系列你信任的 IP、要为之服务的 RP 应用程序、希望规则引擎在处理传入令牌时使用的规则等。命名空间公开了各种终结点，可供应用程序和开发人员用来实现 ACS 的功能。

下图演示 ACS 身份验证如何使用 Web 应用程序：

![ACS 流程图][ACS 流程图]

1.  客户端（在此示例中为浏览器）从 RP 请求页面。
2.  由于尚未对请求进行身份验证，因此 RP 会将用户重定向到它信任的权威机构（即 ACS）。ACS 让用户选择已为此 RP 指定的 IP。用户选择适当的 IP。
3.  客户端浏览到该 IP 的身份验证页，并提示用户登录。
4.  在对客户端进行身份验证（例如，输入标识凭据）后，IP 将颁发一个安全令牌。
5.  颁发安全令牌后，IP 会将客户端重定向到 ACS，并且客户端会向 ACS 发送 IP 所颁发的安全令牌。
6.  ACS 验证 IP 所颁发的安全令牌，将此令牌中的标识声明输入 ACS 规则引擎，计算输出标识声明，并颁发包含这些输出声明的新安全令牌。
7.  ACS 将客户端重定向到 RP。客户端会向 RP 发送 ACS 所颁发的新安全令牌。RP 将验证 ACS 所颁发的安全令牌上的签名，验证此令牌中的声明，并返回最初请求的页面。

## <a name="pre"></a> 先决条件

若要完成本指南中的任务，你将需要：

-   Java 开发人员工具包 (JDK) 1.6 版或更高版本。
-   Eclipse IDE for Java EE Developers, Indigo 或更高版本。可从 <http://www.eclipse.org/downloads/> 下载。
-   分发基于 Java 的 Web 服务器或应用程序服务器，例如 Apache Tomcat、GlassFish、JBoss 应用程序服务器或 Jetty。
-   一个 Azure 订阅，可从 <http://www.windowsazure.cn/zh-cn/pricing/overview/> 获取。
-   Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）2014 年 4 月版。有关详细信息，请参阅[安装 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）][安装 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）]。
-   要用于应用程序的 X.509 证书。你将需要此证书的公用证书 (.cer) 格式版和个人信息交换 (.PFX) 格式版。（本教程后面将介绍用于创建此证书的选项）。
-   熟悉[在 Eclipse 中创建 Azure 的 Hello World 应用程序][在 Eclipse 中创建 Azure 的 Hello World 应用程序]中介绍的 Azure 计算模拟器和部署技术。

## <a name="create-namespace"></a>创建 ACS 命名空间

若要开始在 Azure 中使用 Access Control 服务 (ACS)，必须创建一个 ACS 命名空间。该命名空间提供了一个唯一范围，用于从应用程序中对 ACS 资源进行寻址。

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  单击 **Active Directory**。
3.  若要创建新的 Access Control 命名空间，请依次单击“新建”、“应用程序服务”、**“Access Control”** 和“快速创建”。
4.  输入该命名空间的名称。Azure 将验证该名称是否是唯一的。
5.  选择在其中使用该命名空间的区域。为了获得最佳性能，请使用要在其中部署应用程序的区域。
6.  如果你具有多个订阅，请选择要用于 ACS 命名空间的订阅。
7.  单击“创建”。

Azure 将创建并激活该命名空间。请等到新命名空间的状态变为“活动”，然后再继续操作。

## <a name="add-IP"></a>添加标识提供程序

在此任务中，你将添加与 RP 应用程序一起使用的 IP 来进行身份验证。出于演示目的，此任务将说明如何将 Windows Live 作为 IP 添加，但你可使用 ACS 管理门户中列出的任何 IP。

1.  在 [Azure 管理门户][Azure 管理门户]中，单击 **“Active Directory”**，选择一个 Access Control 命名空间，然后单击“管理”。这将打开 ACS 管理门户。
2.  在 ACS 管理门户的左侧导航窗格中，单击“标识提供程序”。
3.  Windows Live ID 在默认情况下将启用且无法删除。在本教程中，仅使用 Windows Live ID。不过，你可通过单击“添加”按钮在此屏幕中添加其他 IP。

Windows Live ID 现已作为你的 ACS 命名空间的 IP 启用。接下来，请将 Java Web 应用程序（稍后将创建）指定为 RP。

## <a name="add-RP"></a>添加信赖方应用程序

在此任务中，你将 ACS 配置为将 Java Web 应用程序识别为有效的 RP 应用程序。

1.  在 ACS 管理门户上，单击“信赖方应用程序”。
2.  在“信赖方应用程序”页上，单击“添加”。
3.  在“添加信赖方应用程序”页上，执行下列操作：

    1.  在“名称”中，键入 RP 的名称。在本教程中，请键入 **Azure Web
        App**。
    2.  在“模式”中，选择“手动输入设置”。
    3.  在“领域”中，键入 ACS 所颁发的安全令牌将应用于的 URI。对于此任务，请键入 **http://localhost:8080/**。
        ![要在计算模拟器中使用的信赖方领域][要在计算模拟器中使用的信赖方领域]
    4.  在“返回 URL”中，键入 ACS 将安全令牌返回到的 URL。对于此任务，请键入 **http://localhost:8080/MyACSHelloWorld/index.jsp**

        ![要在计算模拟器中使用的信赖方返回 URL][要在计算模拟器中使用的信赖方返回 URL]
    5.  接受其余字段中的默认值。

4.  单击“保存”。

现在你已成功完成对 Java Web 应用程序的配置，该应用程序在 Azure 计算模拟器（位于http://localhost:8080/）中运行时将作为 ACS 命名空间中的 RP。接下来，创建 ACS 用于处理 RP 的声明的规则。

## <a name="create-rules"></a>创建规则

在此任务中，你将定义用于决定将声明从 IP 传递到 RP 的方式的规则。在本指南中，我们只将 ACS 配置为直接复制输出令牌中的输入声明类型和值，而不筛选或修改它们。

1.  在 ACS 管理门户主页上，单击“规则组”。
2.  在“规则组”页上，单击“Azure Web App 的默认规则组”。
3.  在“编辑规则组”页上，单击“生成”。
4.  在“生成规则: Azure Web App 的默认规则组”页上，确保已选中 Windows Live ID，然后单击“生成”。
5.  在“编辑规则组”页上，单击“保存”。

## <a name="upload-certificate"></a>将证书上载到 ACS 命名空间

在此任务中，上载将用于对 ACS 命名空间所创建的令牌请求进行签名的 .PFX 证书。

1.  在 ACS 管理门户主页上，单击“证书和密钥”。
2.  在“证书和密钥”页上，单击“令牌签名”上方的“添加”。
3.  在“添加令牌签名证书或密钥”页上：

    1.  在“用于”部分中，单击“信赖方应用程序”并选择**“Azure Web App”**（你之前已将其设置为信赖方应用程序的名称）。
    2.  在“类型”部分中，选择“X.509 证书”。
    3.  在“证书”部分中，单击“浏览”按钮并导航到要使用的 X.509 证书文件。这将为 .PFX 文件。选择此文件，单击“打开”，然后在“密码”文本框中输入证书密码。请注意，出于测试目的，你可以使用自签名证书。若要创建自签名证书，请使用“ACS 筛选器库”对话框（后面将介绍）中的“新建”按钮，或使用 Azure Starter Kit for Java 的[项目网站][项目网站]中的 **encutil.exe** 实用工具（由 Microsoft Open Technologies 提供）。
    4.  确保选中“设为主”。你的“添加令牌签名证书或密钥”页应与下图中所示类似。

        ![添加令牌签名证书][添加令牌签名证书]
    5.  单击“保存”保存设置并关闭“添加令牌签名证书或密钥”页。

接下来，查看“应用程序集成”页中的信息并复制将 Java Web 应用程序配置为使用 ACS 所需的 URI。

## <a name="review-app-int"></a>查看“应用程序集成”页

你可在 ACS 管理门户的“应用程序集成”页上找到将 Java Web 应用程序（RP 应用程序）配置为使用的 ACS 所需的所有信息和代码。在配置 Java Web 应用程序以进行联合身份验证时需要此信息。

1.  在 ACS 管理门户上，单击“应用程序集成”。
2.  在“应用程序集成”页上，单击“登录页”。
3.  在“登录页集成”页上，单击**“Azure Web App”**。

在“登录页集成: Azure Web App”页上，“选项 1: 链接到托管 ACS 的登录页”中列出的 URL 将用于 Java Web 应用程序。在将 Azure Access Control 服务筛选器库添加到 Java 应用程序时需要此值。

## <a name="create-java-app"></a>创建 Java Web 应用程序

1.  在 Eclipse 中的菜单上，依次单击“文件”、“新建”和“动态 Web 项目”。（如果你在单击“文件”和“新建”后未看到“动态 Web 项目”作为可用项目列出，请执行下列操作：依次单击“文件”、“新建”和“项目”，展开“Web”，再单击“动态 Web 项目”，然后单击“下一步”。）在本教程中，将项目命名为 **MyACSHelloWorld**。（请务必使用此名称，因为本教程中的后续步骤会将你的 WAR 文件命名为 MyACSHelloWorld）。你的屏幕应与下图中所示类似：

    ![为 ACS 示例创建 Hello World 项目][为 ACS 示例创建 Hello World 项目]

    单击“完成”。

2.  在 Eclipse 的项目资源管理器视图中，展开 **MyACSHelloWorld**。右键单击**“WebContent”**，单击“新建”，然后单击“JSP 文件”。
3.  在“新建 JSP 文件”对话框中，将文件命名为 **index.jsp**。将父文件夹保留为 MyACSHelloWorld/WebContent，如下所示：

    ![为 ACS 示例添加 JSP 文件][为 ACS 示例添加 JSP 文件]

    单击**“下一步”**。

4.  在“选择 JSP 模板”对话框中，选择“新建 JSP 文件(html)”，然后单击“完成”。
5.  在 Eclipse 中打开 index.jsp 文件后，添加文本以便在现有 `<body>` 元素中显示 **Hello ACS World!**。更新后的`<body>` 内容应与下图中所示类似：

        <body>
          <b><% out.println("Hello ACS World!"); %></b>
        </body>

    保存 index.jsp。

## <a name="add_acs_filter_library"></a>将 ACS 筛选器库添加到你的应用程序

1.  在 Eclipse 的项目资源管理器中，右键单击**“MyACSHelloWorld”**，单击“生成路径”，然后单击“配置生成路径”。
2.  在“Java 生成路径”对话框中，单击“库”选项卡。
3.  单击“添加库”。
4.  单击“Azure Access Control 服务筛选器(由 MS Open Tech 提供)”，然后单击“下一步”。将显示“Azure Access Control 服务筛选器”对话框。（“位置”字段可能包含其他路径（取决于 Eclipse 的安装位置），并且版本号可能不同（取决于软件更新）。）

    ![添加 ACS 筛选器库][添加 ACS 筛选器库]

5.  使用已打开管理门户的“登录页集成”页的浏览器，复制“选项 1: 链接到托管 ACS 的登录页”字段中列出的 URL，并将其粘贴到 Eclipse 对话框的“ACS 身份验证终结点”字段。
6.  使用已打开管理门户的“编辑信赖方应用程序”页的浏览器，复制“领域”字段中列出的 URL，并将其复制到 Eclipse 对话框的“信赖方领域”字段。
7.  在 Eclipse 对话框的“安全性”部分，如果需要使用现有证书，请单击“浏览”，导航到要使用的证书，选中它，然后单击“打开”。或者，如果需要创建新证书，请单击“新建”以显示“新建证书”对话框，然后为新证书指定密码、.cer 文件的名称和 .pfx 文件的名称。
8.  选中“在 WAR 文件中嵌入证书”。以此方式嵌入证书会将证书包含在部署中，而不需要你将证书作为组件手动添加。（相反，如果你必须从 WAR 文件外部存储证书，则可将证书作为角色组件添加并取消选中“在 WAR 文件中嵌入证书”。）
9.  [可选] 让“需要 HTTPS 连接”保持选中状态。如果你设置此选项，则将需要使用 HTTPS 协议访问你的应用程序。如果你不需要 HTTPS 连接，请取消选中此选项。
10. 对于到计算模拟器的部署，你的“Azure ACS 筛选器”设置与下图中所示类似。

    ![用于部署到计算模拟器的 Azure ACS 筛选器设置][用于部署到计算模拟器的 Azure ACS 筛选器设置]

11. 单击“完成”。
12. 当出现一个说明将创建 web.xml 文件的对话框时，请单击“是”。
13. 单击“确定”关闭“Java 生成路径”对话框。

## <a name="deploy_compute_emulator"></a>部署到计算模拟器

1.  在 Eclipse 的项目资源管理器中，右键单击**“MyACSHelloWorld”**，单击**“Azure”**，然后单击“适用于 Azure 的包”。
2.  对于“项目名称”，请键入 **MyAzureACSProject**，然后单击“下一步”。
3.  选择一个 JDK 和一个应用程序服务器。（[在 Eclipse 中创建 Azure 的 Hello World 应用程序][在 Eclipse 中创建 Azure 的 Hello World 应用程序]教程中将详细介绍这些步骤）。
4.  单击“完成”。
5.  单击“在 Azure 模拟器中运行”按钮。
6.  当你的 Java Web 应用程序在计算模拟器中启动后，请关闭浏览器的所有实例（以便任何当前浏览器会话不会妨碍你的 ACS 登录测试）。
7.  通过在浏览器中打开 <http://localhost:8080/MyACSHelloWorld/>（如果选中“需要 HTTPS 连接”，则打开 <https://localhost:8080/MyACSHelloWorld/>）来运行你的应用程序。系统将提示你输入 Windows Live ID 登录名，随后你将被定向到为你的信赖方应用程序指定的返回 URL。
8.  查看完你的应用程序后，请单击“重置 Azure 模拟器”按钮。

## <a name="deploy_azure"></a>部署到 Azure

若要部署到 Azure，你将需要更改 ACS 命名空间的信赖方领域和返回 URL。

1.  在 Azure 管理门户的“编辑信赖方应用程序”页中，将“领域”修改为你已部署站点的 URL。将“示例”替换为你为部署指定的 DNS 名称。

    ![要在生产中使用的信赖方领域][要在生产中使用的信赖方领域]

2.  将“返回 URL”修改为你应用程序的 URL。将“示例”替换为你为部署指定的 DNS 名称。

    ![要在生产中使用的信赖方返回 URL][要在生产中使用的信赖方返回 URL]

3.  单击“保存”保存已更新的信赖方领域和返回 URL 更改。
4.  使“登录页集成”页在浏览器中保持打开状态，你很快将需要从中进行复制。
5.  在 Eclipse 的项目资源管理器中，右键单击**“MyACSHelloWorld”**，单击“生成路径”，然后单击“配置生成路径”。
6.  依次单击“库”选项卡、“Azure Access Control 服务筛选器”和“编辑”。
7.  使用已打开管理门户的“登录页集成”页的浏览器，复制“选项 1: 链接到托管 ACS 的登录页”字段中列出的 URL，并将其粘贴到 Eclipse 对话框的“ACS 身份验证终结点”字段。
8.  使用已打开管理门户的“编辑信赖方应用程序”页的浏览器，复制“领域”字段中列出的 URL，并将其复制到 Eclipse 对话框的“信赖方领域”字段。
9.  在 Eclipse 对话框的“安全性”部分，如果需要使用现有证书，请单击“浏览”，导航到要使用的证书，选中它，然后单击“打开”。或者，如果需要创建新证书，请单击“新建”以显示“新建证书”对话框，然后为新证书指定密码、.cer 文件的名称和 .pfx 文件的名称。
10. 让“在 WAR 文件中嵌入证书”保持选中状态，假使你要在 WAR 文件中嵌入证书。
11. [可选] 让“需要 HTTPS 连接”保持选中状态。如果你设置此选项，则将需要使用 HTTPS 协议访问你的应用程序。如果你不需要 HTTPS 连接，请取消选中此选项。
12. 对于到 Azure 的部署，你的 Azure ACS 筛选器设置将与下图中所示类似。

    ![用于生产部署的 Azure ACS 筛选器设置][用于生产部署的 Azure ACS 筛选器设置]

13. 单击“完成”关闭“编辑库”对话框。
14. 单击“确定”关闭“MyACSHelloWorld 的属性”对话框。
15. 在 Eclipse 中，单击“发布到 Azure 云”按钮。像在[在 Eclipse 中创建 Azure 的 Hello World 应用程序][在 Eclipse 中创建 Azure 的 Hello World 应用程序]主题的“将应用程序部署到 Azure”部分中一样对提示进行响应。

部署 Web 应用程序后，关闭任何打开的浏览器会话，运行你的 Web 应用程序，系统将提示你使用 Windows Live ID 凭据登录，然后将这些凭据发送到信赖方应用程序的返回 URL。

使用完你的 ACS Hello World 应用程序后，请务必删除部署（可在[在 Eclipse 中创建 Azure 的 Hello World 应用程序][在 Eclipse 中创建 Azure 的 Hello World 应用程序]主题中了解如何删除部署）。

## <a name="next_steps"></a>后续步骤

有关 ACS 返回给你的应用程序的安全声明标记语言 (SAML) 的检查，请参阅[如何查看 Azure Access Control 服务返回的 SAML][如何查看 Azure Access Control 服务返回的 SAML]。若要进一步了解 ACS 的功能并尝试将其用于更复杂的方案，请参阅 [Access Control 服务 2.0][Access Control 服务 2.0]。

另外，此示例使用了“在 WAR 文件中嵌入证书”选项。此选项简化了证书的部署。相反，如果你要将签名证书与 WAR 文件分隔开，可使用以下方法：

1.  在“Azure Access Control 服务筛选器”对话框的“安全性”部分，键入 **${env.JAVA\_HOME}/mycert.cer** 并取消选中“在 WAR 文件中嵌入证书”。（如果你的证书文件名不同，请调整 mycert.cer。）单击“完成”关闭对话框。
2.  在部署中将证书作为组件复制：在 Eclipse 的项目资源管理器中，展开**“MyAzureACSProject”**，右键单击“WorkerRole1”，单击“属性”，展开“Azure 角色”，然后单击“组件”。
3.  单击“添加”。
4.  在“添加组件”对话框中：

    1.  在“导入”部分中：

        1.  使用“文件”按钮导航到要使用的证书。
        2.  对于“方法”，请选择“复制”。

    2.  对于“名称”，单击文本框并接受默认名称。
    3.  在“部署”部分中：

        1.  对于“方法”，请选择“复制”。
        2.  对于“目标目录”，请键入 **%JAVA\_HOME%**。

    4.  你的“添加组件”对话框将与下图中所示类似。

        ![添加证书组件][添加证书组件]

    5.  单击“确定”。

此时，你的证书将包含在部署中。请注意，无论你是在 WAR 文件中嵌入证书还是将其作为组件添加到部署中，都需要将证书上载到命名空间，如[将证书上载到 ACS 命名空间][将证书上载到 ACS 命名空间]部分所述。

<!-- Eclipse-specific -->

  [后续步骤]: #next_steps
  [什么是 ACS？]: #what-is
  [概念]: #concepts
  [先决条件]: #pre
  [创建 ACS 命名空间]: #create-namespace
  [添加标识提供程序]: #add-IP
  [添加信赖方应用程序]: #add-RP
  [创建规则]: #create-rules
  [将证书上载到 ACS 命名空间]: #upload-certificate
  [查看“应用程序集成”页]: #review-app-int
  [创建 Java Web 应用程序]: #create-java-app
  [将 ACS 筛选器库添加到你的应用程序]: #add_acs_filter_library
  [部署到计算模拟器]: #deploy_compute_emulator
  [部署到 Azure]: #deploy_azure
  [Access Control 服务 2.0]: http://msdn.microsoft.com/zh-cn/library/hh147631.aspx
  [ACS 流程图]: ./media/active-directory-java-authenticate-users-access-control-eclipse/ACSFlow.png
  [安装 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）]: http://msdn.microsoft.com/zh-cn/library/azure/hh690946.aspx
  [在 Eclipse 中创建 Azure 的 Hello World 应用程序]: http://msdn.microsoft.com/zh-cn/library/azure/hh690944.aspx
  [Azure 管理门户]: https://manage.windowsazure.cn
  [要在计算模拟器中使用的信赖方领域]: ./media/active-directory-java-authenticate-users-access-control-eclipse/RelyingPartyRealmEmulator.png
  [要在计算模拟器中使用的信赖方返回 URL]: ./media/active-directory-java-authenticate-users-access-control-eclipse/RelyingPartyReturnURLEmulator.png
  [项目网站]: http://wastarterkit4java.codeplex.com/releases/view/61026
  [添加令牌签名证书]: ./media/active-directory-java-authenticate-users-access-control-eclipse/AddTokenSigningCertificate.png
  [为 ACS 示例创建 Hello World 项目]: ./media/active-directory-java-authenticate-users-access-control-eclipse/CreateACSHelloWorld.png
  [为 ACS 示例添加 JSP 文件]: ./media/active-directory-java-authenticate-users-access-control-eclipse/AddJSPFileACS.png
  [添加 ACS 筛选器库]: ./media/active-directory-java-authenticate-users-access-control-eclipse/AddACSFilterLibrary.png
  [用于部署到计算模拟器的 Azure ACS 筛选器设置]: ./media/active-directory-java-authenticate-users-access-control-eclipse/AddACSFilterLibraryEmulator.png
  [要在生产中使用的信赖方领域]: ./media/active-directory-java-authenticate-users-access-control-eclipse/RelyingPartyRealmProduction.png
  [要在生产中使用的信赖方返回 URL]: ./media/active-directory-java-authenticate-users-access-control-eclipse/RelyingPartyReturnURLProduction.png
  [用于生产部署的 Azure ACS 筛选器设置]: ./media/active-directory-java-authenticate-users-access-control-eclipse/AddACSFilterLibraryProduction.png
  [如何查看 Azure Access Control 服务返回的 SAML]: /en-us/develop/java/how-to-guides/view-saml-returned-by-acs/
  [添加证书组件]: ./media/active-directory-java-authenticate-users-access-control-eclipse/AddCertificateComponent.png
