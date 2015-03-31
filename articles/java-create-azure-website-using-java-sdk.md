<properties 
	pageTitle="在 Azure 上创建网站 - Azure SDK for Java" 
	description="了解如何以编程方式在 Azure 上创建一个网站。" 
	services="" 
	documentationCenter="Java" 
	authors="donntrenton" 
	manager="wpickett" 
	editor="jimbe"/>

# 使用 Azure SDK for Java 在 Azure 上创建一个网站

本演练演示如何创建一个 Azure SDK for Java 应用程序以用于在 Azure 上创建网站，然后将一个应用程序部署到该网站。它由两个部分组成：

- 第 1 部分演示如何生成一个用于在 Azure 上创建网站的 Java 应用程序。
- 第 2 部分演示如何创建一个简单的 JSP"Hello World"应用程序并将代码部署到新建的网站，然后使用 FTP 客户端将文件传输到该网站。


# 先决条件

## 在 Azure 中创建并配置云资源

在开始此过程之前，你需要拥有有效的 Azure 订阅，并在 Azure 上设置默认的 Active Directory (AD)。


## 在 Azure 中创建 Active Directory (AD)

如果你的 Azure 订阅中还没有 Active Directory (AD)，请使用你的 Microsoft 帐户登录 Azure 管理门户 (AMP)。如果你有多个订阅，请单击"订阅"并选择要用于此项目的订阅的默认目录。然后单击"应用"切换到该订阅视图。

1. 从左侧菜单中选择"Active Directory"。**单击"新建 > 目录 > 自定义创建"**。

2. 在"添加目录"中，选择"创建新目录"。

3. 在"名称"中输入目录名称。

4. 在"域"中输入域名。这是默认情况下你的目录附带的基本域名，它采用 `<域名>.onmicrosoft.com` 格式。可以根据目录名称或你拥有的其他域名将它命名。以后，可以添加你的组织已在使用的其他域名。有关 AD 的详细信息，请参阅[什么是 Azure AD 目录？](http://technet.microsoft.com/library/jj573650.aspx)。


## 创建 Azure 的管理证书

Azure SDK for Java 使用管理证书在 Azure 订阅中进行身份验证。对于使用服务管理 API 代表订阅所有者管理订阅资源的客户端应用程序，你可以使用这些 X.509 v3 证书来对其进行身份验证。

此过程中的网站创建代码使用自签名证书在 Azure 上进行身份验证。对于此过程，你需要事先创建一个证书并将其上载到 Azure 管理门户 (AMP)。这包括以下步骤：

- 生成表示客户端证书的 PFX 文件，并将其保存在本地。
- 从 PFX 文件生成管理证书（CER 文件）。
- 将 CER 文件上载到你的 Azure 订阅。
- 将 PFX 文件转换为 JKS，因为 Java 以这种格式来使用证书进行身份验证。
- 编写引用本地 JKS 文件的应用程序身份验证代码。

完成此过程后，CER 证书将驻留在 Azure 订阅中，JKS 证书将驻留在本地驱动器中。有关管理证书的详细信息，请参阅[创建并上载 Azure 的管理证书](http://msdn.microsoft.com/library/azure/gg551722.aspx)。


### 创建证书

若要创建自己的自签名证书，请在操作系统上打开一个命令控制台并运行以下命令。

> **注意：**运行此命令的计算机必须已安装 JDK。 此外，keytool 的路径取决于 JDK 的安装位置。有关详细信息，请参阅 Java 联机文档中的[密钥和证书管理工具 (keytool)](http://docs.oracle.com/javase/6/docs/technotes/tools/windows/keytool.html) 。

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
- `<keystore-id>` 是密钥库条目标识符（例如  `AzureRemoteAccess`）。
- `<cert-store-dir>` 是要存储证书的目录的路径（例如 `C:/Certificates`）。
- `<cert-file-name>` 是证书文件的名称（例如 `AzureWebDemoCert`）。
- `<password>` 是选择用于保护证书的密码；它的长度必须至少为 6 个字符。可以不输入密码，但不建议这样做。
- `<dname>` 是要与别名关联的 X.500 可分辨名称，它用作自签名证书中的颁发者和使用者字段。

有关详细信息，请参阅[创建并上载 Azure 的管理证书](http://msdn.microsoft.com/library/azure/gg551722.aspx)。


### 上载证书

若要将自签名证书上载到 Azure，请转到 Azure 管理门户中的"设置"页，然后单击"管理证书"选项卡。单击页面底部的"上载"，然后导航到你创建的 CER 文件所在位置。


### 将 PFX 文件转换为 JKS

在 Windows 命令提示符下（以管理员身份运行），键入 cd 转到包含证书的目录，然后并运行以下命令，其中，`<java-install-dir>`是计算机安装 Java 的目录：

    <java-install-dir>/bin/keytool.exe -importkeystore
     -srckeystore <cert-store-dir>/<cert-file-name>.pfx
     -destkeystore <cert-store-dir>/<cert-file-name>.jks
     -srcstoretype pkcs12 -deststoretype JKS

1. 出现提示时，输入目标密钥库密码；这将是 JKS 文件的密码。

2. 出现提示时，输入源密钥库密码；这是你为 PFX 文件指定的密码。

两个密码不一定要相同。可以不输入密码，但不建议这样做。


# 生成网站创建应用程序

## 创建 Eclipse 工作区和 Maven 项目

在本部分中，你将要给名为 AzureWebDemo 的网站创建应用程序创建一个工作区和一个 Maven 项目。

1. 创建新的 Maven 项目。单击"文件 > 新建 > Maven 项目"。在"新建 Maven 项目"中，选择"创建一个简单项目"和"使用默认的工作区位置"。

2. 在"新建 Maven 项目"第二页上，指定以下信息：

    - 组 ID：`com.<用户名>.azure.webdemo`
    - 项目 ID：AzureWebDemo
    - 版本：0.0.1-SNAPSHOT
    - 打包：jar
    - 名称：AzureWebDemo

    单击"完成"。

3. 在项目资源管理器中打开新项目的 pom.xml 文件。选择"依赖关系"选项卡。由于这是一个新项目，因此尚未列出任何包。

4. 打开"Maven 存储库"视图。**单击"窗口 > 显示视图 > 其他 > Maven > Maven 存储库"，然后单击"确定"**。"Maven 存储库"视图将出现在 IDE 的底部。

5. 打开"全局存储库"，右键单击中央存储库，然后选择"重新生成索引"。

    ![][1]
    
    此步骤可能需要几分钟时间，具体取决于你的连接速度。重新生成索引后，**中央** Maven 存储库中应会显示 Microsoft Azure 包。

6. 在"依赖关系"中，单击"添加"。在"输入组 ID..."中，输入 `azure-management`。选择基础管理和网站管理的包：

        com.microsoft.azure  azure-management
        com.microsoft.azure  azure-management-websites

单击"确定"。Azure 包随即会出现在"依赖关系"列表中。


## 编写 Java 代码以通过调用 Azure SDK 来创建网站

接下来，请编写调用 Azure SDK for Java 中的 API 的代码，以创建 Azure 网站。

1. 创建一个 Java 类以用于包含主入口点代码。在项目资源管理器中右键单击项目节点，然后选择"新建 > 类"。

2. 在"新建 Java 类"中，将该类命名为 `Program`，并选中"public static void main"复选框。所选内容应如下所示：

    ![][2]

3. 单击"完成"。Program.java 文件将出现在项目资源管理器中。


## 调用 Azure API 以创建网站


### 添加所需的导入

在 Program.java 中添加以下导入；这些导入可让你访问使用 Azure API 的管理库中的类：

    import java.io.IOException;
    import java.net.URI;
    import java.net.URISyntaxException;
    import java.util.ArrayList;
    import javax.xml.parsers.ParserConfigurationException;
    import com.microsoft.windowsazure.exception.ServiceException;
    import org.xml.sax.SAXException;
    
    // Imports for service management configuration
    import com.microsoft.windowsazure.Configuration;
    import com.microsoft.windowsazure.management.configuration.ManagementConfiguration;
    
    // Service management imports for website creation
    import com.microsoft.windowsazure.management.websites.*;
    import com.microsoft.windowsazure.management.websites.models.*;
    
    // Imports for authentication
    import com.microsoft.windowsazure.core.utils.KeyStoreType;


### 提供主入口点代码

提供调用 Azure 服务管理 API 的主入口点代码以创建 Azure 网站。代码将输出指示成功或失败的 HTTP 响应状态；如果成功，则输出创建的网站的名称。

    public class Program {
    
      // Parameter definitions used for authentication
      static String uri = "https://management.core.chinacloudapi.cn/";
      static String subscriptionId = "<subscription-id>";
      static String keyStoreLocation = "<certificate-store-path>";
      static String keyStorePassword = "<certificate-password>";
    
      // Define website parameter values
      private static String websiteName = "DemoWebsite";
      private static String webSpaceName = WebSpaceNames.WESTUSWEBSPACE;
      private static String hostName = ".chinacloudsites.cn";

其中：

- `<subscription-id>` 是你要在其中创建资源的 Azure 订阅 ID。
- `<certificate-store-path>` 是本地证书存储目录中的 JKS 文件的路径和文件名。例如， `C:/Certificates/CertificateName.jks`（对于 Linux）和 `C:\Certificates\CertificateName.jks`（对于 Windows）。
- `<certificate-password>` 是你在创建 JKS 证书时指定的密码。
- `websiteName` 可以是所需的任何名称；此过程将其命名为 `DemoWebsite`。域名以网站名称开头，以主机名结尾，因此，在本例中，完整的域为 `demowebsite.chinacloudsites.cn`。
- 应该按上面所示指定 `webSpaceName` 和 `hostName`。


### 定义 Web 创建方法

接下来，定义用于创建网站的方法。此方法  `createWebSite` 指定网站的参数和 Web 空间。它还创建并配置 [WebSiteManagementClient](http://dl.windowsazure.com/javadoc/com/microsoft/windowsazure/management/websites/WebSiteManagementClient.html) 对象定义的网站管理 客户端。网站管理客户端对于在 Azure 中创建网站至关重要。它提供 RESTful web 服务，使应用程序能够通过调用服务管理 API 来管理 Azure 网站（创建、更新、删除，等等）。

    private static void createWebSite() throws Exception {
      ArrayList<String> hostNamesValue = new ArrayList<String>();
      hostNamesValue.add(websiteName + hostName);

      // Set webspace details
      WebSiteCreateParameters.WebSpaceDetails webSpaceDetails = new WebSiteCreateParameters.WebSpaceDetails(); 
      webSpaceDetails.setGeoRegion(GeoRegionNames.WESTUS); 
      webSpaceDetails.setPlan(WebSpacePlanNames.VIRTUALDEDICATEDPLAN); 
      webSpaceDetails.setName(webSpaceName); 

      // Set website parameters
      WebSiteCreateParameters createParameters = new WebSiteCreateParameters(); 
      createParameters.setName(websiteName);  
      createParameters.setWebSpaceName(webSpaceName); 
      createParameters.setWebSpace(webSpaceDetails); 
      createParameters.setSiteMode(WebSiteMode.Basic); 
      createParameters.setComputeMode(WebSiteComputeMode.Shared); 
      createParameters.setHostNames(hostNamesValue); 

      // Configuration for the service management client
      Configuration config = ManagementConfiguration.configure(
        new URI(uri),
        subscriptionId,
        keyStoreLocation, // Path to the JKS file
        keyStorePassword, // Password for the JKS file
        KeyStoreType.jks  // Flag that you are using a JKS keystore
        );

      // Create the website management client to call Azure APIs;
      // pass it the service management configuration object
      WebSiteManagementClient webSiteManagementClient = WebSiteManagementService.create(config);

      // Create the website
      WebSiteCreateResponse webSiteCreateResponse = webSiteManagementClient.getWebSitesOperations().create(webSpaceName, createParameters);
      
      // Output the HTTP status code of the response; 200 indicates 
      // that the request succeeded; 4xx indicates failure
      System.out.println("----------");
      System.out.println("Website created - HTTP response " + webSiteCreateResponse.getStatusCode() + "\n");

      // Retrieve and output the name of the website this app created;
      String websitename = webSiteCreateResponse.getWebSite().getName();
      System.out.println("----------\n");
      System.out.println("Name of website created: " + websitename + "\n");
      System.out.println("----------\n");
    }

最后，从 `main` 调用 `createWebSite`：

    public static void main(String[] args)
      throws IOException, URISyntaxException, ServiceException, ParserConfigurationException, SAXException, Exception {

      // Create website
      createWebSite();

      }
    }


### 运行应用程序并验证网站创建结果

若要验证应用程序是否能够运行，请单击"运行 > 运行"。在应用程序完成运行后，你应该会在 Eclipse 控制台中看到以下输出：

    ----------
    Website created - HTTP response 200
    
    ----------
    
    Name of website created: WebDemoWebsite
    
    ----------

登录 AMP 并单击"网站"。在数分钟内，新网站应会出现在"网站"列表中。


# 将应用程序部署到网站

在运行 AzureWebDemo 并创建新网站后，请登录 AMP，单击"网站"，然后在"网站"列表中选择"WebDemoWebsite"。在网站的仪表板页上，单击"浏览"（或单击网站 URL `webdemowebsite.chinacloudsites.cn`）导航到该网站。你将会看到一个空白的占位符页，因为尚未将任何内容发布到网站。

接下来，你要创建一个"Hello World"应用程序并将其部署到网站。


## 创建 JSP Hello World 应用程序

### 创建应用程序

为了演示如何将应用程序部署到 Web，以下过程说明了如何创建一个简单的"Hello World"应用程序并将其传送到应用程序创建的网站。

1. 单击"文件 > 新建 > 动态 Web 项目"。将它命名为 `JSPHello`。不需要在此对话框中更改其他任何设置。单击"完成"。

    ![][3]

2. 在项目资源管理器中，展开JSPHello 项目，右键单击"WebContent"，然后单击"新建 > JSP 文件"。在"新建 JSP 文件"对话框中，将新文件命名为 `index.jsp`。单击"下一步"。

3. 在"选择 JSP 模板"对话框中，选择"新建 JSP 文件(html)"，然后单击"完成"。

4. 在 index.jsp 的 `<head>` 和 `<body>` 标记部分中添加以下代码：

        <head>
          ...
          java.util.Date date = new java.util.Date();
        </head>
    
        <body>
          Hello, the time is <%= date %> 
        </body>


### 在 localhost 中运行 Hello World 应用程序

在运行此应用程序之前，你需要配置几个属性。

1. 右键单击"JSPHello"项目并选择"属性"。

2. 在"属性"对话框中：选择"Java 生成路径"，选择"顺序和导出"选项卡，选中"JRE 系统库"，然后单击"上移"将其移至列表顶部。

    ![][4]

3. 同样在"属性"对话框中：选择"目标运行时"，然后单击"新建"。

4. 在"新建服务器运行时环境"对话框中选择一个服务器（如 Apache Tomcat v7.0），然后单击"下一步"。在"Tomcat 服务器"对话框中，将"名称"设置为"Apache Tomcat v7.0"，并将"Tomcat 安装目录"设置为要在其中安装所要使用的 Tomcat 服务器的目录``。

    ![][5]

    单击"完成"。

5. 随后你将返回"属性"对话框的"目标运行时"页。选择"Apache Tomcat v7.0"，然后单击"确定"。

    ![][6]

6. 在 Eclipse 的"运行"菜单中，单击"运行"。在"运行方式"对话框中，选择"在服务器上运行"。在"在服务器上运行"对话框中，选择"Tomcat v7.0 服务器"：

    ![][7]

    单击"完成"。

7. 当应用程序运行时，你应该会在 Eclipse 中的 localhost 窗口中看到显示的"JSPHello"页 (`http://localhost:8080/JSPHello/`)，其中显示了以下消息：

    `Hello World, the time is Tue Oct 28 16:56:05 PDT 2014`


### 将应用程序导出为 WAR

将 web 项目文件导出为 web 存档 (WAR) 文件，以便可以将它部署到网站。以下 web 项目文件驻留在 WebContent 文件夹中：

    META-INF
    WEB-INF
    index.jsp

1. 右键单击 WebContent 文件夹并选择"导出"。

2. 在"导出选择"对话框中，单击"Web > WAR 文件"，然后单击"下一步"。

3. 在"WAR 导出"对话框中，选择当前项目中的 src 目录，并在末尾添加 WAR 文件名。例如：

    `<project-path>/JSPHello/src/JSPHello.war`

有关部署 WAR 文件的详细信息，请参阅[将应用程序添加到 Azure 上的 Java 网站](/documentation/articles/web-sites-java-add-app/)。


## 使用 FTP 部署 Hello World 应用程序

选择第三方 FTP 客户端来发布应用程序。此过程将介绍两个选项：Azure 中内置的 Kudu 控制台；FileZilla，这是一个带有便捷式图形 UI 的常用工具。

> **注意：**Azure Plugin for Eclipse with Java 2.4 支持部署到存储帐户和云服务，但当前不支持部署到网站。你可以根据[在 Eclipse 中创建 Azure 的 Hello World 应用程序](http://msdn.microsoft.com/library/azure/hh690944.aspx)中所述，使用 Azure 部署项目来部署到存储帐户和云服务。但是，该插件提供的"部署到 Azure"工具当前不支持网站。使用其他方法（例如 FTP 或 GitHub）将文件传输到网站。

> **注意：**不建议从 Windows 命令提示符使用 FTP（Windows 随附的 FTP.EXE 命令行实用工具）。使用活动 FTP 的 FTP 客户端（例如 FTP.EXE）往往无法穿过防火墙工作。活动 FTP 会指定一个基于 LAN 的内部地址，而 FTP 服务器有可能无法连接到该地址。

有关使用 FTP 部署到 Azure 网站的详细信息，请参阅以下主题：

- [通过 Azure 管理门户管理网站：FTP 凭据](/documentation/articles/web-sites-manage/#ftp-credentials)
- [如何使用 FTP 实用工具部署](/documentation/articles/web-sites-deploy/#ftp)


### 设置部署凭据

确保已运行 **AzureWebDemo** 应用程序来创建网站。这是要将文件传输到的网站。

1. 登录 AMP 并单击"网站"。确保 **WebDemoWebsite** 已显示在网站列表中，并确保它正在运行。单击"WebDemoWebsite"以打开其"仪表板"页。

2. 在"仪表板"页上的"速览"下，单击"设置部署凭据"（如果你已部署凭据，则此选项将显示为"重置部署凭据"）。

    部署凭据与某个 Microsoft 帐户关联。需要指定可用于使用 Git 和 FTP 进行部署的用户名和密码。可以使用这些凭据部署到与你的 Microsoft 帐户关联的所有 Azure 订阅中的任何网站。在对话框中提供 Git 和 FTP 部署凭据，并记下用户名和密码以供将来使用。


### 获取 FTP 连接信息

若要使用 FTP 将应用程序文件部署到新建的网站，你需要获取连接信息。可通过两种方法获取连接信息。一种方法是访问网站的"仪表板"页；另一种方法是下载网站的发布配置文件。发布配置文件是一个 XML 文件，它提供 Azure 网站的 FTP 主机名和登录凭据等信息。你可以使用此用户名和密码部署到与 Azure 帐户关联的所有订阅中的任何网站，而不仅仅是此网站。

从网站的"仪表板"页获取 FTP 连接信息：

1. 在"速览"下，查找并复制"FTP 主机名"。这是类似于 `ftp://cnws-prod-sha-001.ftp.chinacloudsites.chinacloudapi.cn` 的 URI。

2. 在"速览"下，查找并复制"部署/FTP 用户"。此值的格式为 *WebsiteName\DeploymentUsername*；例如 `WebDemoWebsite\deployer77`。

从网站的发布配置文件获取 FTP 连接信息：

1. 在网站的"仪表板"中的"速览"下，单击"下载发布配置文件"。这会将一个 .publishsettings 文件下载到本地驱动器。

2. 在 XML 编辑器或文本编辑器中打开 .publishsettings 文件并找到包含 `publishMethod ="FTP"` 的 `<publishProfile>` 元素。该元素应类似于：

        <publishProfile
            profileName="WebDemoWebsite - FTP"
            publishMethod="FTP"
            publishUrl="ftp://cnws-prod-sha-001.ftp.chinacloudsites.chinacloudapi.cn/site/wwwroot"
            ftpPassiveMode="True"
            userName="WebDemoWebsite\$WebDemoWebsite"
            userPWD="<deployment-password>"
            ...
        </publishProfile>

3. 请注意，网站的 `publishProfile` 设置将按如下所示映射到 FileZilla 站点管理器设置：

- `publishUrl` 与你在"主机"中设置的"FTP 主机名"值相同。
- `publishMethod ="FTP"` 表示你已将"协议"设置为"FTP - 文件传输协议"，已将"加密"设置为"使用普通 FTP"。
- `userName` 和 `userPWD` 是你在重置部署凭据时指定的实际用户名和密码值。 `userName` 与"部署/FTP 用户"相同。它们将映射到 FileZilla 中的"用户"和"密码"。
- `ftpPassiveMode ="True"` 表示 FTP 站点使用被动 FTP 传输；在"传输设置"选项卡上选择"被动"。


### 配置网站以托管 Java 应用程序

在将应用程序发布到网站之前，你需要更改几项配置设置，使该网站可以托管 Java 应用程序。

1. 在 AMP 中，转到网站的"仪表板"页，然后单击"配置"。在"配置"页上指定以下设置。

2. 在"Java 版本"中，默认值为"关闭"；选择你的应用程序所针对的 Java 版本，例如 1.7.0_51。完成此操作后，还请确保"Web 容器"已设置为 Tomcat 服务器的版本。

3. 在"默认文档"中，添加 index.jsp 并将其移到列表的顶部。（Azure 网站的默认文件为 hostingstart.html。）

4. 单击"保存"。


### 使用 Kudu 将应用程序发布到网站

发布应用程序的一种方法是使用 Azure 中内置的 Kudu 调试控制台。使用 Kudu 的好处之一在于，它的行为已知是稳定的，并且与 Azure 网站和 Tomcat 服务器相一致。可以通过浏览到以下格式的 URL 来以网站资源的形式访问该控制台：

`https://<WebsiteName>.scm.chinacloudsites.cn/DebugConsole`

1. 对于此过程，Kudu 控制台位于以下 URL 中；请浏览到此位置：

    `https://webdemowebsite.scm.chinacloudsites.cn/DebugConsole`

2. 从顶部菜单中，选择"调试控制台 > CMD"。

3. 在控制台命令行中，导航到 `/site/wwwroot`（或单击"站点" ``，然后在页面顶部的目录视图中单击"wwwroot"``）：

    `cd /site/wwwroot`

4. 指定"Java 版本"后，Tomcat 服务器应会创建一个 webapps 目录。在控制台命令行中，导航到 webapps 目录：

    `mkdir webapps`

    `cd webapps`

5. 将 JSPHello.war 从 `<project-path>/JSPHello/src/` 拖放到 Kudu 目录视图中的 `/site/wwwroot/webapps` 下。请不将它拖放到"拖到此处以上载和压缩"区域，因为 Tomcat 会将其解压缩。

  ![][8]

JSPHello.war 自身首先会显示在目录区域中：

  ![][9]

不久之后（可能小于 5 分钟），Tomcat 服务器会将 WAR 文件解压缩到解包的 JSPHello 目录中。单击根目录以查看 index.jsp 是否已解压缩并复制到此处。如果是，请导航回到 webapps 目录，查看是否已创建解包的 JSPHello 目录。如果你看不到这些项，请等待片刻后重复操作。

  ![][10]


### 使用 FileZilla 将应用程序发布到网站（可选）

可用于发布应用程序的另一个工具是 FileZilla，这是一个带有便捷式图形 UI 的常用第三方 FTP 客户端。你可以从 [http://filezilla-project.org/](http://filezilla-project.org/) 下载并安装 FileZilla， 如果尚未安装的话。有关使用该客户端的详细信息，请参阅 [FileZilla 文档](https://wiki.filezilla-project.org/Documentation) 以及此博客文章 [FTP 客户端 - 第 4 部分：FileZilla](http://blogs.msdn.com/b/robert_mcmurray/archive/2008/12/17/ftp-clients-part-4-filezilla.aspx)。

1. 在 FileZilla 中，单击"文件 > 站点管理器"。
2. 在"站点管理器"对话框中，单击"新建站点"。随后，一个新的空白 FTP 站点将出现在"选择条目"中，其中会提示你提供一个名称。对于此过程，请将它命名为 `AzureWebDemo FTP`。

    在"常规"选项卡上指定以下设置：
    - **主机：**输入你在 AMP 中从网站仪表板复制的"FTP 主机名"。
    - **端口:**（将其留空，因为这是被动传输，并且服务器会确定要使用的端口。）
    - **协议：**FTP 文件传输协议
    - **加密：**使用普通 FTP
    - **登录类型：**正常
    - **用户：**输入你在 AMP 中从网站仪表板复制的"部署/FTP 用户"。这是完整的 FTP 用户名，其格式为 WebsiteName\Username。
    - **密码：**输入你在设置部署凭据时指定的密码。

    在"传输设置"选项卡上，选择"被动"。

3. 单击"连接"。如果连接成功，FileZilla 控制台将显示"状态: ``已连接"消息，并发出 `LIST` 命令以列出目录内容。
4. 在"本地"站点面板中，选择 JSPHello.war 文件所在的源目录；路径如下所示：

    `<project-path>/JSPHello/src/`

5. 在"远程"站点面板中，选择目标文件夹。WAR 文件将会部署到网站根目录下的 webapps 目录中。

6. 导航到 `/site/wwwroot`，右键单击 `wwwroot`，然后选择"创建目录"。将该目录命名为 `webapps`，然后进入该目录。

7. 将 JSPHello.war 传输到 `/site/wwwroot/webapps`。在"本地"文件列表中选择 JSPHello.war，右键单击它，然后选择"上载"。随后它应该会出现在 `/site/wwwroot/webapps` 中。

8. 将 JSPHello.war 复制到 webapps 目录后，Tomcat 服务器将自动解包（解压缩）该 WAR 文件中的文件。尽管 Tomcat 服务器马上就会解包，但文件可能需要在很长时间（可能是几小时）之后才会出现在 FTP 客户端中。


### 在网站上运行 Hello World 应用程序

1. 在上载 WAR 文件并确认 Tomcat 服务器已创建解包的 `JSPHello` 目录后，请浏览到 `http://webdemowebsite.chinacloudsites.cn/JSPHello` 以运行该应用程序。

    > **注意：**如果从 AMP 单击"浏览"，你可能会看到默认网页，其中指出"已成功创建这个基于 Java 的 Web 应用程序"。可能需要刷新网页才能查看应用程序输出而不是默认网页。

2. 当应用程序运行时，你应会看到具有以下输出的网页：

    `Hello World, the time is Tue Nov 04 19:31:44 GMT 2014`


### 清理 Azure 资源

此过程将在 Azure 上创建一个网站。只要该网站存在，你就要支付网站资源的费用。除非你打算继续使用该网站以进行测试或开发，否则应考虑停止或删除它。已停止的网站仍会产生较小的费用，但你随时可以重新启动它。删除某个网站会清除已上载到该网站的所有数据。



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
