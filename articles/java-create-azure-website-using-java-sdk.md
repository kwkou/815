<properties 
	pageTitle="使用 Azure SDK for Java 在 Azure 中创建 Web 应用" 
	description="了解如何使用 Azure SDK for Java 以编程方式在 Azure Web 应用上创建 Web 应用。" 
	tags="azure-classic-portal"
	services="app-service\web" 
	documentationCenter="Java" 
	authors="donntrenton" 
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="multiple"
	ms.date="02/25/2016"
	wacn.date="04/26/2016"/>


# 使用 Azure SDK for Java 在 Azure 中创建 Web 应用

<!-- Azure Active Directory workflow is not yet available on the Azure Management Portal -->

## 概述

本演练演示如何创建一个 Azure SDK for Java 应用程序，以便在 [Azure Web 应用][]中创建一个 Web 应用，然后将一个应用程序部署到该应用。它由两个部分组成：

- 第 1 部分演示如何生成创建 Web 应用的 Java 应用程序。
- 第 2 部分演示如何创建简单的 JSP "Hello World" 应用程序，然后使用 FTP 客户端将代码部署到 Azure Web 应用。


## 先决条件

### 软件安装

本文中的 AzureWebDemo 应用程序代码是使用 Azure Java SDK 0.7.0 编写的，你可以使用 [Web 平台安装程序][] (WebPI) 进行安装。此外，请确保使用最新版本的 Azure Toolkit for Eclipse。安装 SDK 之后，通过在“Maven 存储库”中运行“更新索引”更新 Eclipse 项目中的依赖项，然后在“依赖项”窗口中重新添加每个包的最新版本。可以通过单击“帮助 > 安装详细信息”验证 Eclipse 中已安装软件的版本；至少应具有以下版本：

- Package for Azure Libraries for Java 0.7.0.20150309
- Eclipse IDE for Java EE Developers 4.4.2.20150219


### 在 Azure 中创建并配置云资源

在开始此过程之前，你需要拥有有效的 Azure 订阅，并在 Azure 上设置默认的 Active Directory (AD)。


### 在 Azure 中创建 Active Directory (AD)

如果你的 Azure 订阅中还没有 Active Directory (AD)，请使用你的 Microsoft 帐户登录 [Azure 经典管理门户][]。如果你有多个订阅，请单击“订阅”并选择要用于此项目的订阅的默认目录。然后单击“应用”切换到该订阅视图。

1. 从左侧菜单中选择“Active Directory”。单击“新建 > 目录 > 自定义创建”。

2. 在“添加目录”中，选择“创建新目录”。

3. 在“名称”中输入目录名称。

4. 在“域”中输入域名。这是默认情况下你的目录附带的基本域名，它采用 `<domain_name>.partner.onmschina.cn` 格式。可以根据目录名称或你拥有的其他域名将它命名。以后，可以添加你的组织已在使用的其他域名。

5. 在“国家或地区”中选择你的区域设置。

有关 AD 的详细信息，请参阅[什么是 Azure AD 目录][]？


### 创建 Azure 的管理证书

Azure SDK for Java 使用管理证书在 Azure 订阅中进行身份验证。对于使用服务管理 API 代表订阅所有者管理订阅资源的客户端应用程序，你可以使用这些 X.509 v3 证书来对其进行身份验证。

此过程中的代码使用自签名证书在 Azure 上进行身份验证。对于此过程，你需要事先创建一个证书并将其上载到 [Azure 经典管理门户][]。这包括以下步骤：

- 生成表示客户端证书的 PFX 文件，并将其保存在本地。
- 从 PFX 文件生成管理证书（CER 文件）。
- 将 CER 文件上载到你的 Azure 订阅。
- 将 PFX 文件转换为 JKS，因为 Java 以这种格式来使用证书进行身份验证。
- 编写引用本地 JKS 文件的应用程序身份验证代码。

完成此过程后，CER 证书将驻留在 Azure 订阅中，JKS 证书将驻留在本地驱动器中。有关管理证书的详细信息，请参阅[创建并上载 Azure 的管理证书][]。


#### 创建证书

若要创建自己的自签名证书，请在操作系统上打开一个命令控制台并运行以下命令。

> **注意：**运行此命令的计算机必须已安装 JDK。此外，keytool 的路径取决于安装 JDK 的位置。有关详细信息，请参阅 Java 联机文档中的[密钥和证书管理工具 (keytool)][]。

创建 .pfx 文件：

    <java-install-dir>/bin/keytool -genkey -alias <keystore-id>
     -keystore <cert-store-dir>/<cert-file-name>.pfx -storepass <password>
     -validity 3650 -keyalg RSA -keysize 2048 -storetype pkcs12
     -dname "CN=Self Signed Certificate 20141118170652"

创建 .cer 文件：

    <java-install-dir>/bin/keytool -export -alias <keystore-id>
     -storetype pkcs12 -keystore <cert-store-dir>/<cert-file-name>.pfx
     -storepass <password> -rfc -file <cert-store-dir>/<cert-file-name>.cer

其中：

- `<java-install-dir>` 是 Java 安装目录的路径。
- `<keystore-id>` 是密钥库条目标识符（例如 `AzureRemoteAccess`）。
- `<cert-store-dir>` 是要存储证书的目录的路径（例如 `C:/Certificates`）。
- `<cert-file-name>` 是证书文件的名称（例如 `AzureWebDemoCert`）。
- `<password>` 是选择用于保护证书的密码；它的长度必须至少为 6 个字符。可以不输入密码，但不建议这样做。
- `<dname>` 是要与别名关联的 X.500 可分辨名称，它用作自签名证书中的颁发者和使用者字段。

有关详细信息，请参阅[创建并上载 Azure 的管理证书][]。


#### 上载证书

若要将自签名证书上载到 Azure，请转到经典管理门户中的“设置”页，然后单击“管理证书”选项卡。单击页面底部的“上载”，然后导航到你创建的 CER 文件的所在位置。


#### 将 PFX 文件转换为 JKS

在 Windows 命令提示符下（以管理员身份运行），键入 cd 转到包含证书的目录，然后运行以下命令，其中，`<java-install-dir>` 是计算机安装 Java 的目录：

    <java-install-dir>/bin/keytool.exe -importkeystore
     -srckeystore <cert-store-dir>/<cert-file-name>.pfx
     -destkeystore <cert-store-dir>/<cert-file-name>.jks
     -srcstoretype pkcs12 -deststoretype JKS

1. 出现提示时，输入目标密钥库密码；这将是 JKS 文件的密码。

2. 出现提示时，输入源密钥库密码；这是你为 PFX 文件指定的密码。

两个密码不一定要相同。可以不输入密码，但不建议这样做。


## 构建 Web 应用创建应用程序

### 创建 Eclipse 工作区和 Maven 项目

在本部分中，你将要给名为 AzureWebDemo 的 Web 应用创建应用程序创建工作区和 Maven 项目。

1. 创建新的 Maven 项目。单击“文件 > 新建 > Maven 项目”。在“新建 Maven 项目”中，选择“创建简单项目”和“使用默认工作区位置”。

2. 在“新建 Maven 项目”的第二页上，指定以下信息：

    - 组 ID：`com.<username>.azure.webdemo`
    - 项目 ID：AzureWebDemo
    - 版本：0.0.1-SNAPSHOT
    - 打包：jar
    - 名称：AzureWebDemo

    单击“完成”。

3. 在项目资源管理器中打开新项目的 pom.xml 文件。选择“依赖项”选项卡。由于这是一个新项目，因此尚未列出任何包。

4. 打开“Maven 存储库”视图。单击“窗口 > 显示视图 > 其他 > Maven > Maven 存储库”，然后单击“确定”。“Maven 存储库”视图将出现在 IDE 的底部。

5. 打开“全局存储库”，右键单击“中央”存储库，然后选择“重新生成索引”。

    ![][1]
    
    此步骤可能需要几分钟时间，具体取决于你的连接速度。重新生成索引后，“中央”Maven 存储库中应会显示 Azure 包。

6. 在“依赖项”中，单击“添加”。在“输入组 ID...”中输入 `azure-management`。选择基础管理和 Azure Web 应用管理的包：

        com.microsoft.azure  azure-management
        com.microsoft.azure  azure-management-websites

    > **注意：**如果在新版本发布后更新依赖项，
    > 则需要重新添加此列表中的每个依赖项。
    > 单击“添加”后，选择每个依赖项，
    > 则会在“依赖项”列表中显示新的版本号。

单击**“确定”**。Azure 包随即会出现在“依赖项”列表中。


### 编写 Java 代码，以通过调用 Azure SDK 来创建 Web 应用

接下来，请编写调用 Azure SDK for Java 中的 API 的代码，以创建 Azure Web 应用。

1. 创建一个 Java 类以用于包含主入口点代码。在项目资源管理器中，右键单击项目节点，然后选择“新建 > 类”。

2. 在“新建 Java 类”中，将类命名为 `WebCreator`，并选中“public static void main”复选框。所选内容应如下所示：

    ![][2]

3. 单击“完成”。WebCreator.java 文件将在项目资源管理器中出现。


### 调用 Azure API 以创建 Azure Web 应用


#### 添加所需的导入

在 WebCreator.java 中添加以下导入；使用这些导入可以访问使用 Azure API 的管理库中的类：

    // General imports
    import java.net.URI;
    import java.util.ArrayList;
    
    // Imports for Exceptions
    import java.io.IOException;
    import java.net.URISyntaxException;
    import javax.xml.parsers.ParserConfigurationException;
    import com.microsoft.windowsazure.exception.ServiceException;
    import org.xml.sax.SAXException;
    
    // Imports for Azure Websites management configuration
    import com.microsoft.windowsazure.Configuration;
    import com.microsoft.windowsazure.management.configuration.ManagementConfiguration;
    
    // Service management imports for Azure Websites creation
    import com.microsoft.windowsazure.management.websites.*;
    import com.microsoft.windowsazure.management.websites.models.*;
    
    // Imports for authentication
    import com.microsoft.windowsazure.core.utils.KeyStoreType;


#### 定义主入口点类

因为 AzureWebDemo 应用程序的目的是创建 Azure Web 应用，所以请将此应用程序的主类命名为 `WebAppCreator`。此类提供调用 Azure 服务管理 API 的主入口点代码，以创建 Web 应用。

为 Web 应用和 Web 空间添加以下参数定义。你将需要提供你自己的 Azure 订阅 ID 和证书信息。

    public class WebAppCreator {
    
        // Parameter definitions used for authentication.
        private static String uri = "https://management.core.chinacloudapi.cn/";
        private static String subscriptionId = "<subscription-id>";
        private static String keyStoreLocation = "<certificate-store-path>";
        private static String keyStorePassword = "<certificate-password>";
    
        // Define Web app parameter values.
        private static String webAppName = "WebDemoWebApp";
        private static String domainName = ".chinacloudsites.cn";
        private static String webSpaceName = WebSpaceNames.WESTUSWEBSPACE;
        private static String appServicePlanName = "WebDemoAppServicePlan";

其中：

- `<subscription-id>` 是要用于创建资源的 Azure 订阅 ID。
- `<certificate-store-path>` 是你的本地证书存储区目录中的 JKS 文件的路径和文件名。例如，对于 Linux，则为 `C:/Certificates/CertificateName.jks`；对于 Windows，则为 `C:\Certificates\CertificateName.jks`。
- `<certificate-password>` 是你在创建 JKS 证书时指定的密码。
- `webAppName` 可以是你选择的任何名称；此过程使用名称 `WebDemoWebApp`。完整的域名称是 `webAppName`，并追加了 `domainName`，因此在这种情况下，完整的域是 `webdemowebapp.chinacloudsites.cn`。
- `domainName` 应按照以上所示进行指定。

> **注意：**每次运行此应用程序时，
> 你需要更改 `webAppName` 和 `appServicePlanName` 的值（或在 Azure 经典管理门户上删除 Web 应用），
> 然后再次运行应用程序。否则，
> 由于 Azure 上已存在相同的资源，所以执行会失败。


#### 定义 Web 创建方法

接下来，定义用于创建 Web 应用的方法。此方法 (`createWebApp`) 指定 Web 应用的参数和 Web 空间。它还会创建并配置 Azure Web 应用管理客户端，该客户端由 [WebSiteManagementClient][] 对象进行定义。管理客户端对于创建 Web 应用至关重要。它提供 RESTful web 服务，使应用程序能够通过调用服务管理 API 来管理 Web 应用（执行创建、更新和删除等操作）。

    private static void createWebApp() throws Exception {

        // Specify configuration settings for the Azure Websites management client.
        Configuration config = ManagementConfiguration.configure(
            new URI(uri),
            subscriptionId,
            keyStoreLocation,  // Path to the JKS file
            keyStorePassword,  // Password for the JKS file
            KeyStoreType.jks   // Flag that you are using a JKS keystore
        );

        // Create the Azure Websites management client to call Azure APIs
        // and pass it the Azure Websites management configuration object.
        WebSiteManagementClient webAppManagementClient = WebSiteManagementService.create(config);

        // Create an App Service plan for the Web app with the specified parameters.
        WebHostingPlanCreateParameters appServicePlanParams = new WebHostingPlanCreateParameters();
        appServicePlanParams.setName(appServicePlanName);
        appServicePlanParams.setSKU(SkuOptions.Free);
        webAppManagementClient.getWebHostingPlansOperations().create(webSpaceName, appServicePlanParams);

        // Set webspace parameters.
        WebSiteCreateParameters.WebSpaceDetails webSpaceDetails = new WebSiteCreateParameters.WebSpaceDetails();
        webSpaceDetails.setGeoRegion(GeoRegionNames.WESTUS);
        webSpaceDetails.setPlan(WebSpacePlanNames.VIRTUALDEDICATEDPLAN);
        webSpaceDetails.setName(webSpaceName);

        // Set Web app parameters.
        // Note that the server farm name takes the App Service plan name.
        WebSiteCreateParameters webAppCreateParameters = new WebSiteCreateParameters();
        webAppCreateParameters.setName(webAppName);
        webAppCreateParameters.setServerFarm(appServicePlanName);
        webAppCreateParameters.setWebSpace(webSpaceDetails);

        // Set usage metrics attributes.
        WebSiteGetUsageMetricsResponse.UsageMetric usageMetric = new WebSiteGetUsageMetricsResponse.UsageMetric();
        usageMetric.setSiteMode(WebSiteMode.Basic);
        usageMetric.setComputeMode(WebSiteComputeMode.Shared);

        // Define the Web app object.
        ArrayList<String> fullWebAppName = new ArrayList<String>();
        fullWebAppName.add(webAppName + domainName);
        WebSite webApp = new WebSite();
        webApp.setHostNames(fullWebAppName);

        // Create the Web app.
        WebSiteCreateResponse webAppCreateResponse = webAppManagementClient.getWebSitesOperations().create(webSpaceName, webAppCreateParameters);

        // Output the HTTP status code of the response; 200 indicates the request succeeded; 4xx indicates failure.
        System.out.println("----------");
        System.out.println(" Web app created - HTTP response " + webAppCreateResponse.getStatusCode() + "\n");

        // Output the name of the Web app that this application created.
        String shinyNewWebAppName = webAppCreateResponse.getWebSite().getName();
        System.out.println("----------\n");
        System.out.println("Name of Web app created: " + shinyNewWebAppName + "\n");
        System.out.println("----------\n");
    }

代码将输出指示成功或失败的 HTTP 响应状态；如果成功，则输出创建的 Web 应用的名称。


#### 定义 main() 方法

提供调用 createWebApp() 的 main() 方法代码，以创建 Web 应用。

最后，从 `main` 调用 `createWebApp`：

        public static void main(String[] args)
            throws IOException, URISyntaxException, ServiceException,
            ParserConfigurationException, SAXException, Exception {

            // Create Web app
            createWebApp();

        }  // end of main()

    }  // end of WebAppCreator class


#### 运行应用程序并验证 Web 应用创建

若要验证应用程序是否运行，请单击“运行 > 运行”。在应用程序完成运行后，你应该会在 Eclipse 控制台中看到以下输出：

    ----------
   Web app created - HTTP response 200
    
    ----------
    
    Name of Web app created: WebDemoWebApp
    
    ----------

登录到 Azure 经典管理门户并单击“Web Apps”。在数分钟内，新 Web 应用应会出现在“Web Apps”列表中。


## 将应用程序部署到 Web 应用

运行 AzureWebDemo 并创建新 Web 应用后，请登录经典管理门户，单击“Web Apps”，然后在“Web Apps”列表中选择“WebDemoWebApp”。在 Web 应用的仪表板页上，单击“浏览”（或单击 URL `webdemowebapp.chinacloudsites.cn`）导航到该 Web 应用。你将会看到一个空白的占位符页，因为尚未将任何内容发布到 Web 应用。

接下来，你要创建一个“Hello World”应用程序并将其部署到 Web 应用。


### 创建 JSP Hello World 应用程序

#### 创建应用程序

为了演示如何将应用程序部署到 Web，以下过程说明了如何创建简单的 "Hello World" Java 应用程序，并将其上载到应用程序创建的 Azure Web 应用。

1. 单击“文件 > 新建 > 动态 Web 项目”。将它命名为 `JSPHello`。不需要在此对话框中更改其他任何设置。单击“完成”。

    ![][3]

2. 在项目资源管理器中，展开“JSPHello”项目，右键单击“WebContent”，然后单击“新建 > JSP 文件”。在“新建 JSP 文件”对话框中，将新文件命名为 `index.jsp`。单击“下一步”。

3. 在“选择 JSP 模板”对话框中，选择“新建 JSP 文件 (html)”，然后单击“完成”。

4. 在 index.jsp 中，在 `<head>` 和 `<body>` 标记部分中添加以下代码：

        <head>
          ...
          java.util.Date date = new java.util.Date();
        </head>
    
        <body>
          Hello, the time is <%= date %> 
        </body>


#### 在 localhost 中运行 Hello World 应用程序

在运行此应用程序之前，你需要配置几个属性。

1. 右键单击“JSPHello”项目并选择“属性”。

2. 在“属性”对话框中：选择“Java 生成路径”，选择“顺序和导出”选项卡，选中“JRE 系统库”，然后单击“上移”将其移至列表顶部。

    ![][4]

3. 同样在“属性”对话框中：选择“目标运行时”，然后单击“新建”。

4. 在“新建服务器运行时环境”对话框中，选择一个服务器（如“Apache Tomcat v7.0”），然后单击“下一步”。在“Tomcat 服务器”对话框中，将“名称”设置为 `Apache Tomcat v7.0`，并将“Tomcat 安装目录”设置为在其中安装所要使用的 Tomcat 服务器版本的目录。

    ![][5]

    单击“完成”。

5. 随后你将返回“属性”对话框的“目标运行时”页。选择“Apache Tomcat v7.0”，然后单击“确定”。

    ![][6]

6. 在 Eclipse 的“运行”菜单中，单击“运行”。在“运行方式”对话框中，选择“在服务器上运行”。在“在服务器上运行”对话框中，选择“Tomcat v7.0 服务器”：

    ![][7]

    单击“完成”。

7. 当应用程序运行时，你应该会在 Eclipse (`http://localhost:8080/JSPHello/`) 的 localhost 窗口中看到显示的“JSPHello”页，其中显示了以下消息：

    `Hello World, the time is Tue Mar 24 23:21:10 GMT 2015`


#### 将应用程序导出为 WAR

将 Web 项目文件导出为 Web 存档 (WAR) 文件，以便可以将它部署到 Web 应用。以下 web 项目文件驻留在 WebContent 文件夹中：

    META-INF
    WEB-INF
    index.jsp

1. 右键单击 WebContent 文件夹并选择“导出”。

2. 在“导出选择”对话框中，单击“Web > WAR 文件”，然后单击“下一步”。

3. 在“WAR 导出”对话框中，选择当前项目中的 src 目录，并在末尾添加 WAR 文件名。例如：

    `<project-path>/JSPHello/src/JSPHello.war`

有关部署 WAR 文件的详细信息，请参阅[将 Java 应用程序添加到 Azure Web 应用](/documentation/articles/web-sites-java-add-app/)。


### 使用 FTP 部署 Hello World 应用程序

选择第三方 FTP 客户端来发布应用程序。此过程将介绍FileZilla，这是一个带有便捷式图形 UI 的常用工具。

> **注意：**Azure Toolkit for Eclipse 支持部署到存储帐户和云服务，但当前不支持部署到 Web 应用。你可以使用 Azure 部署项目部署到存储帐户和云服务，但不能部署到 Web 应用。使用其他方法（例如 FTP 或 GitHub）将文件传输到 Web 应用。

> **注意：**我们不建议通过 Windows 命令提示符（Windows 随附的命令行 FTP.EXE 实用工具）使用 FTP。使用活动 FTP 的 FTP 客户端（如 FTP.EXE）通常无法通过防火墙工作。活动 FTP 指定基于 LAN 的内部地址，FTP 服务器可能无法连接到该地址。

有关使用 FTP 部署到 Azure Web 应用的详细信息，请参阅以下主题：

- [使用 FTP 实用工具部署](/documentation/articles/web-sites-deploy/)


#### 设置部署凭据

确保已运行 **AzureWebDemo** 应用程序来创建 Web 应用。你会将文件转移到此位置。

1. 登录到经典管理门户并单击“Web Apps”。确保“WebDemoWebApp”已显示在 Web Apps 列表中，并确保它正在运行。单击“WebDemoWebApp”以打开其“仪表板”页。

2. 在“仪表板”页的“速览”下，单击“设置部署凭据”（如果你已有部署凭据，则此选项应为“重置部署凭据”）。

    部署凭据与某个 Microsoft 帐户关联。需要指定可用于使用 Git 和 FTP 进行部署的用户名和密码。可以使用这些凭据部署到与你的 Microsoft 帐户关联的所有 Azure 订阅中的任何 Web 应用。在对话框中提供 Git 和 FTP 部署凭据，并记下用户名和密码以供将来使用。


#### 获取 FTP 连接信息

若要使用 FTP 将应用程序文件部署到新建的 Web 应用，你需要获取连接信息。可通过两种方法获取连接信息。一种方法是访问 Web 应用的“仪表板”页；另一种方法是下载 Web 应用的发布配置文件。发布配置文件是一个 XML 文件，它提供 Azure 中 Web 应用的 FTP 主机名和登录凭据等信息。你可以使用此用户名和密码部署到与 Azure 帐户关联的所有订阅中的任何 Web 应用，而不仅仅是此 Web 应用。

从 Web 应用的“仪表板”页获取 FTP 连接信息：

1. 在“速览”下，查找并复制“FTP 主机名”。这是类似于 `ftp://cnws-prod-sha-001.ftp.chinacloudsites.chinacloudapi.cn` 的 URI。

2. 在“速览”下，查找并复制“部署/FTP 用户”。此值的形式为 *WebsiteName\\DeploymentUsername*；例如 `WebDemoWebsite\deployer77`。

从 Web 应用的发布配置文件获取 FTP 连接信息：

1. 在 Web 应用的“仪表板”中的“速览”下，单击“下载发布配置文件”。这会将一个 .publishsettings 文件下载到本地驱动器。

2. 在 XML 编辑器或文本编辑器中打开 .publishsettings 文件并找到包含 `publishMethod="FTP"` 的 `<publishProfile>` 元素。该元素应类似于：

        <publishProfile
            profileName="WebDemoWebApp - FTP"
            publishMethod="FTP"
            publishUrl="ftp://waws-prod-bay-NNN.ftp.azurewebsites.chinacloudapi.cn/site/wwwroot"
            ftpPassiveMode="True"
            userName="WebDemoWebApp\$WebDemoWebApp"
            userPWD="<deployment-password>"
            ...
        </publishProfile>

3. 请注意， Web 应用的 `publishProfile` 设置将按如下所示映射到 FileZilla 站点管理员设置：

- `publishUrl` 与你在“主机”中设置的“FTP 主机名”的值相同。
- `publishMethod="FTP"` 表示你已将“协议”设置为“FTP - 文件传输协议”，已将“加密”设置为“使用普通 FTP”。
- `userName` 和 `userPWD` 是你在重置部署凭据时指定的实际用户名和密码值的密钥。`userName` 与“部署/FTP 用户”相同。它们将映射到 FileZilla 中的“用户”和“密码”。
- `ftpPassiveMode="True"` 表示 FTP 站点使用被动 FTP 传输；在“传输设置”选项卡上选择“被动”。


#### 配置 Web 应用以托管 Java 应用程序

发布应用程序之前，你需要更改几项配置设置，使 Web 应用可以托管 Java 应用程序。

1. 在经典管理门户中，转到 Web 应用的“仪表板”页，然后单击“配置”。在“配置”页上指定以下设置。

2. 在“Java 版本”中，默认值为“关闭”；选择你的应用程序所针对的 Java 版本，例如 1.7.0\_51。完成此操作后，还请确保“Web 容器”已设置为 Tomcat 服务器的版本。

3. 在“默认文档”中，添加 index.jsp 并将其上移至列表的顶部。（ Web 应用的默认文件为 hostingstart.html。）

4. 单击“保存”。

#### 使用 FileZilla 发布应用程序（可选）

可用于发布应用程序的另一个工具是 FileZilla，这是一个带有便捷式图形 UI 的常用第三方 FTP 客户端。如果尚未安装，则可以从 [http://filezilla-project.org/](http://filezilla-project.org/) 中下载并安装 FileZilla。有关使用客户端的详细信息，请参阅 [FileZilla 文档](https://wiki.filezilla-project.org/Documentation) 和 [FTP 客户端 - 第 4 部分：FileZilla](http://blogs.msdn.com/b/robert_mcmurray/archive/2008/12/17/ftp-clients-part-4-filezilla.aspx) 上的此博客条目。

1. 在 FileZilla 中，单击“文件 > 站点管理员”。
2. 在“站点管理员”对话框中，单击“新建站点”。随后，“选择条目”中会出现新的空白 FTP 站点，提示你提供名称。对于此过程，请将它命名为 `AzureWebDemo-FTP`。

    在“常规”选项卡上，指定以下设置：
    -“主机”：输入从仪表板中复制的“FTP 主机名”
    -“端口”：（将其留空，因为这是被动传输并且服务器会确定使用的端口。）
    -“协议”：FTP 文件传输协议 
    -“加密”：使用普通的 FTP 
    -“登录类型”：普通 
    -“用户”：输入从仪表板中复制的部署/FTP 用户。这是完整的 FTP 用户名，其形式为 *webappname\\username*。
    -“密码”：输入在设置部署凭据时指定的密码。

    在“传输设置”选项卡上，选择“被动”。

3. 单击“连接”。如果成功，FileZilla 的控制台则会显示 `Status: Connected` 消息并发布 `LIST` 命令，以列出目录内容。

4. 在“本地”站点面板中，选择 JSPHello.war 文件所在的源目录；路径与以下路径类似：

    `<project-path>/JSPHello/src/`

5. 在“远程”站点面板中，选择目标文件夹。WAR 文件将会部署到 Web 应用根目录下的 `webapps` 目录中。导航到 `/site/wwwroot`，右键单击 `wwwroot`，然后选择“创建目录”。将目录命名为 `webapps`，然后进入该目录。

6. 将 JSPHello.war 传输到 `/site/wwwroot/webapps`。在“本地”文件列表中选择 JSPHello.war，右键单击它，然后选择“上载”。随后它应该会出现在 `/site/wwwroot/webapps` 中。

7. 将 JSPHello.war 复制到 webapps 目录后，Tomcat 服务器将自动解包（解压缩）该 WAR 文件中的文件。尽管 Tomcat 服务器马上就会解包，但文件可能需要在很长时间（可能是几小时）之后才会出现在 FTP 客户端中。


#### 在 Web 应用上运行 Hello World 应用程序

1. 上载 WAR 文件并确认 Tomcat 服务器已创建解包的 `JSPHello` 目录后，请浏览到 `http://webdemowebapp.chinacloudsites.cn/JSPHello` 以运行该应用程序。

    > **注意：**如果从经典管理门户单击“浏览”，则可能获得默认网页，
    网页显示“已成功创建此基于 Java 的 Web 应用。
    ” 你可能需要刷新网页才能查看应用程序输出，
    而不是默认网页。

2. 当应用程序运行时，你应会看到具有以下输出的网页：

    `Hello World, the time is Tue Mar 24 23:21:10 GMT 2015`


#### 清理 Azure 资源

此过程创建 Azure Web 应用。只要 Web 应用存在，你就要支付资源的费用。除非你打算继续使用该 Web 应用进行测试或开发，否则应考虑停止或删除它。已停止的 Web 应用仍会产生较小的费用，但你随时可以重新启动它。删除某个 Web 应用会清除已上载到该 Web 应用的所有数据。



  [1]: ./media/java-create-azure-website-using-java-sdk/eclipse-maven-repositories-rebuild-index.png
  [2]: ./media/java-create-azure-website-using-java-sdk/eclipse-new-java-class.png
  [3]: ./media/java-create-azure-website-using-java-sdk/eclipse-new-dynamic-web-project.png
  [4]: ./media/java-create-azure-website-using-java-sdk/eclipse-java-build-path.png
  [5]: ./media/java-create-azure-website-using-java-sdk/eclipse-targeted-runtimes-tomcat-server.png
  [6]: ./media/java-create-azure-website-using-java-sdk/eclipse-targeted-runtimes-properties-page.png
  [7]: ./media/java-create-azure-website-using-java-sdk/eclipse-run-on-server.png
  [8]: ./media/java-create-azure-website-using-java-sdk/kudu-console-drag-drop.png
  [9]: ./media/java-create-azure-website-using-java-sdk/kudu-console-jsphello-war-1.png
  [10]: ./media/java-create-azure-website-using-java-sdk/kudu-console-jsphello-war-2.png
 

[Azure Web 应用]: /documentation/services/web-sites/
[Web 平台安装程序]: http://go.microsoft.com/fwlink/?LinkID=252838
[Azure Toolkit for Eclipse]: https://msdn.microsoft.com/zh-cn/library/azure/hh690946.aspx
[Azure Management Portal]: https://manage.windowsazure.cn
[什么是 Azure AD 目录]: http://technet.microsoft.com/zh-cn/library/jj573650.aspx
[创建并上载 Azure 的管理证书]: /documentation/articles/cloud-services-certs-create/
[密钥和证书管理工具 (keytool)]: http://docs.oracle.com/javase/6/docs/technotes/tools/windows/keytool.html
[WebSiteManagementClient]: http://azure.github.io/azure-sdk-for-java/com/microsoft/azure/management/websites/WebSiteManagementClient.html
[WebSpaceNames]: http://azure.github.io/azure-sdk-for-java/com/microsoft/windowsazure/management/websites/models/WebSpaceNames.html
[Azure 经典管理门户]: https://manage.windowsazure.cn

<!---HONumber=79-->