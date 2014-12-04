<properties linkid="develop-java-how-to-guides-web-sso" urlDisplayName="Web SSO" pageTitle="Azure Active Directory 包含的单一登录功能 (Java)" metaKeywords="Azure Java web app, Azure single sign-on, Azure Java Active Directory" description="了解如何使用 Azure Active Directory 包含的单一登录创建 Java Web 应用程序。" metaCanonical="" services="active-directory" documentationCenter="Java" title="通过 Java 和 Azure Active Directory 实现的 Web 单一登录" authors="" solutions="" manager="" editor="" />

# 通过 Java 和 Azure Active Directory 实现的 Web 单一登录

## <a name="introduction"></a>介绍

本教程将向 Java 开发人员介绍如何利用 Azure Active Directory 为 Office 365 客户的用户实现单一登录。你将了解如何执行以下操作：

-   在客户的租户中设置 Web 应用程序
-   使用 WS 联合身份验证保护应用程序

### 先决条件

本教程使用特定的应用程序服务器，但如果您是熟练的 Java 开发人员，则也可以将下面所述的过程应用于其他环境。学习本教程需要满足以下开发环境先决条件：

-   [针对 Azure Active Directory 的 Java 示例代码][针对 Azure Active Directory 的 Java 示例代码]
-   [Java 运行时环境 1.6][Java 运行时环境 1.6]
-   [JBoss Application Server 7.1.1 最终版][JBoss Application Server 7.1.1 最终版]
-   [JBoss Developer Studio IDE][JBoss Developer Studio IDE]
-   启用了 SSL 的 Internet Information Services (IIS) 7.5
-   Windows PowerShell
-   [Office 365 PowerShell Commandlets][Office 365 PowerShell Commandlets]

### 目录

-   [介绍][介绍]
-   [步骤 1：创建 Java 应用程序][步骤 1：创建 Java 应用程序]
-   [步骤 2：在公司的目录租户中设置应用程序][步骤 2：在公司的目录租户中设置应用程序]
-   [步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序][步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序]
-   [摘要][摘要]

## <a name="createapp"></a>步骤 1：创建 Java 应用程序

此步骤介绍如何创建将表示受保护资源的简单 Java 应用程序。对此资源的访问权限将通过公司的 STS 管理的联合身份验证（将在本教程后面部分介绍）授予。

1.  打开 JBoss Developer Studio 的新实例。
2.  在**文件**菜单中，单击**新建**，然后单击**项目...**。
3.  在**新建项目**对话框中，展开 **Maven** 文件夹，单击 **Maven 项目**，然后单击**下一步**。
4.  在**新建 Maven 项目”**对话框中，选中**创建简单项目(跳过 archetype 选择)**，然后单击**下一步**。
5.  在**新建 Maven 项目**对话框的下一页上，在**组 ID** 和 **项目 ID** 文本框中键入 *sample*。然后，从**打包**下拉菜单中选择 **war**，然后单击**完成**。
6.  将创建新的 Maven 项目。在左侧的**项目资源管理器**菜单中，展开 **sample** 项目，右键单击 **pom.xml** 文件，单击**打开方式**，然后单击**文本编辑器**。
7.  在 **pom.xml** 文件中，在 *\<project\>* 节中添加以下 XML：

        <repositories>
            <repository>
                <id>jboss-public-repository-group</id>
                <name>JBoss Public Maven Repository Group</name>
                <url>https://repository.jboss.org/nexus/content/groups/public-jboss/</url>
                <layout>default</layout>
                <releases>
                    <enabled>true</enabled>
                    <updatePolicy>never</updatePolicy>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>never</updatePolicy>
                </snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
                <id>jboss-public-repository-group</id>
                <name>JBoss Public Maven Repository Group</name>
                <url>https://repository.jboss.org/nexus/content/groups/public-jboss/</url>
                <layout>default</layout>
                <releases>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
            </pluginRepository>
        </pluginRepositories>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>2.0.2</version>
                    <configuration>
                        <source>1.6</source>
                        <target>1.6</target>
                    </configuration>
                </plugin>
            </plugins>
        </build> 

    输入此 XML 后，保存 **pom.xml** 文件。

8.  右键单击 **sample** 项目并单击 **Maven**，然后单击**更新 Maven 项目**。在“更新 Maven 项目”对话框中，单击“确定”。此步骤将使用 **pom.xml** 更改来更新项目。

9.  右键单击 **sample** 项目，单击**“新建”**，然后单击**“JSP 文件”**。

10. 在**“新建 JSP 文件”**对话框上，将新文件的路径更改为 *sample/src/main/webapp*。然后，将文件命名为 **index.jsp** 并单击**“完成”**。

11. 将自动打开新的 **index.jsp** 文件。将自动生成的代码替换为以下形式，然后保存文件：

        <%@ page language="java" contentType="text/html; charset=ISO-8859-1"  pageEncoding="ISO-8859-1"%>
        <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
        <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
            <title>Index</title>
        </head>
        <body>
            <h3>Index Page</h3>
        </body>
        </html> 

12. 右键单击 **sample** 项目并单击**“运行身份”**，然后单击**“在服务器上运行”**。

13. 在**“在服务器上运行”**对话框上，确保选择了**“JBoss Enterprise Application Platform 6.x”**，然后单击**“已完成”**。

## <a name="provisionapp"></a>步骤 2：在公司的目录租户中设置应用程序

此步骤介绍 Azure Active Directory 客户的管理员如何在其租户中设置 Java 应用程序以及如何配置单一登录。完成此步骤后，公司的员工可以使用其 Office 365 帐户对 Web 应用程序进行身份验证。

设置过程的第一步是为应用程序创建新的服务主体。Azure Active Directory 将使用服务主体来向目录注册和验证应用程序。

1.  如果尚未下载和安装 Office 365 PowerShell Commandlets，则执行此操作。
2.  在“开始”菜单中，运行“用于 Windows PowerShell 的 Microsoft Online Services 模块”控制台。此控制台提供了用于配置有关 Office 365 租户的属性（如创建和修改服务主体）的命令行环境。
3.  若要导入必需的 **MSOnlineExtended** 模块，键入以下命令并按 Enter：

        Import-Module MSOnlineExtended -Force

4.  若要连接到 Office 365 目录，你需要提供公司的管理员凭据。请键入以下命令并按 Enter，然后在提示符处输入凭据：

        Connect-MsolService

5.  现在，你将为应用程序创建新的服务主体。请键入以下命令并按 Enter：

        New-MsolServicePrincipal -ServicePrincipalNames @("javaSample/localhost") -DisplayName "Federation Sample Web Site" -Type Symmetric -Usage Verify -StartDate "12/01/2012" -EndDate "12/01/2013" 

    此步骤将输出类似于下面的信息：

        The following symmetric key was created as one was not supplied qY+Drf20Zz+A4t2w e3PebCopoCugO76My+JMVsqNBFc=
        DisplayName           : Federation Sample Java Web Site
        ServicePrincipalNames : {javaSample/localhost}
        ObjectId              : 59cab09a-3f5d-4e86-999c-2e69f682d90d
        AppPrincipalId        : 7829c758-2bef-43df-a685-717089474505
        TrustedForDelegation  : False
        AccountEnabled        : True
        KeyType               : Symmetric
        KeyId                 : f1735cbe-aa46-421b-8a1c-03b8f9bb3565
        StartDate             : 12/01/2012 08:00:00 a.m.
        EndDate               : 12/01/2013 08:00:00 a.m.
        Usage                 : Verify 

    > [WACOM.NOTE]
    > 你应该保存此输出，尤其是生成的对称密钥。此密钥仅在服务主体创建期间对你显示，你以后将无法检索它。使用 Graph API 在目录中读取和写入信息还需要其他值。

6.  最后一个步骤是为应用程序设置答复 URL。在进行身份验证尝试之后，将在答复 URL 中发送响应。请键入以下命令并按 Enter：

        $replyUrl = New-MsolServicePrincipalAddresses -Address "https://localhost:8443/sample" 

        Set-MsolServicePrincipal -AppPrincipalId "7829c758-2bef-43df-a685-717089474505" -Addresses $replyUrl 

现在已经在目录中设置 Web 应用程序，公司员工可以使用它来进行 Web 单一登录。

## <a name="protectapp"></a>步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序

此步骤演示如何使用 Windows Identity Foundation (WIF) 和作为必备组件中的示例代码包的一部分下载的 **waad-federation** 库来添加对联合登录的支持。你还将添加登录页并配置应用程序和目录租户之间的信任关系。

1.  在 JBoss Developer Studio 中，单击**文件**，然后单击**导入**。

2.  在**“导入”**对话框中，展开 **Maven** 文件夹，单击**“现有 Maven 项目”**，然后单击**“下一步”**。

3.  在**“导入 Maven 项目”**对话框中，将**“根目录”**路径设置为您将示例代码中的 **waad-federation** 库下载到的位置。然后，选中 **waad-federation** 项目中的 **pom.xml** 文件旁边的复选框并单击**“完成”**。

4.  展开 **sample** 项目，右键单击 **pom.xml** 文件，单击**“打开方式”**，然后单击**“文本编辑器”**。

5.  在 **pom.xml** 文件中，在 *\<project\>* 节中添加以下 XML，然后保存文件：

        <dependencies>
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>servlet-api</artifactId>
                <version>3.0-alpha-1</version>
            </dependency>
            <dependency>
                <groupId>com.microsoft.samples.waad.federation</groupId>
                <artifactId>waad-federation</artifactId>
                <version>0.0.1-SNAPSHOT</version>
            </dependency>
        </dependencies> 

6.  右键单击 **sample** 项目，单击 **Maven**，然后单击**“更新项目”**。在“更新 Maven 项目”对话框中，单击“确定”。此步骤将使用 **pom.xml** 更改来更新项目。

7.  右键单击 **sample** 项目，单击**“新建”**，然后单击**“筛选器”**。

8.  在**“创建筛选器”**对话框中，为**“类名称”**条目键入 *FederationFilter*，然后单击**“完成”**。

9.  将打开自动生成的 **FederationFilter.java** 文件。将其中的代码替换为以下代码并保存该文件：

        import java.io.IOException;
        import javax.servlet.Filter;
        import javax.servlet.FilterChain;
        import javax.servlet.FilterConfig;
        import javax.servlet.ServletException;
        import javax.servlet.ServletRequest;
        import javax.servlet.ServletResponse;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.util.regex.*;
        import com.microsoft.samples.federation.FederatedLoginManager; import com.microsoft.samples.federation.URLUTF8Encoder;

        public class FederationFilter implements Filter {
            private String loginPage;
            private String allowedRegex;
            public void init(FilterConfig config) throws ServletException {
                this.loginPage = config.getInitParameter("login-page-url");
                this.allowedRegex = config.getInitParameter("allowed-regex");
            }
            public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
                    if (request instanceof HttpServletRequest) { 
                        HttpServletRequest httpRequest = (HttpServletRequest) request;
                        if (!httpRequest.getRequestURL().toString().contains(this.loginPage)) {
                            FederatedLoginManager loginManager = FederatedLoginManager.fromRequest(httpRequest);
                            boolean allowedUrl = Pattern.compile(this.allowedRegex).matcher(httpRequest.getRequestURL().toString()).find();
                            if (!allowedUrl && !loginManager.isAuthenticated()) {
                                HttpServletResponse httpResponse = (HttpServletResponse) response;
                                String encodedReturnUrl = URLUTF8Encoder.encode(httpRequest.getRequestURL().toString());
                                httpResponse.setHeader("Location", this.loginPage + "?returnUrl=" + encodedReturnUrl);
                                httpResponse.setStatus(302);
                                return;
                            }
                        }
                    }
                chain.doFilter(request, response);
            }
            public void destroy() {
            }
        } 

10. 在**“项目资源管理器”**中，依次展开 **src**、**main** 和 **webapp** 文件夹。右键单击 **web.xml** 文件，单击**“打开方式”**，然后单击**“文本编辑器”**。

11. 在 **web.xml** 文件中，您将添加一个筛选器来处理受保护和未受保护的页面，它会将未经过身份验证的用户重定向到登录页（作为 **login-page-url** 筛选器参数）。但是，如果 URL 与 **allowed-regex** 筛选器参数中指定的正则表达式匹配，则不会被筛选掉。在 *\<web-app\>* 节中添加以下 XML，然后保存 **web.xml** 文件。

        <filter>
            <filter-name>FederationFilter</filter-name>
            <filter-class>FederationFilter</filter-class>
            <init-param>
                <param-name>login-page-url</param-name>
                <param-value>/sample/login.jsp</param-value>
            </init-param>
            <init-param>
                <param-name>allowed-regex</param-name>
                <param-value>(\/sample\/login.jsp|\/sample\/wsfed-saml|\/sample\/oauth)</param-value>
            </init-param>
        </filter>
        <filter-mapping>
            <filter-name>FederationFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping> 

12. 若要创建登录页，请右键单击 **sample** 项目，单击**“新建”**，然后单击**“JSP 文件”**。

13. 在**“新建 JSP 文件”**对话框上，将新文件的路径更改为 *sample/src/main/webapp*。然后，将文件命名为 **login.jsp** 并单击**“完成”**。

14. 将自动打开新的 **login.jsp** 文件。将自动生成的代码替换为以下形式，然后保存文件：

        <%@ page language="java" contentType="text/html; charset=ISO-8859-1"  pageEncoding="ISO-8859-1"%>
        <%@ page import="com.microsoft.samples.federation.*"%>
        <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
        <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1"> 
            <title>Login Page</title>
        </head>
        <body>
            <h3>Login Page</h3>
            <a href="<%=FederatedLoginManager.getFederatedLoginUrl(request.getParameter("returnUrl"))%>"><%=FederatedConfiguration.getInstance().getStsFriendlyName()%></a>
        </body>
        </html> 

15. 在**“项目资源管理器”**中，展开 **sample** 项目的 **/src/main** 文件夹。右键单击 **resources** 文件夹，单击**“新建”**，然后单击**“其他”**。

16. 在**“新建”**对话框中，展开 **JBoss Tools Web** 文件夹，单击**“属性文件”**，然后单击**“下一步”**。

17. 在**“新建文件属性”**对话框中，将文件命名为 **federation**，然后单击**“完成”**。

18. 在**“项目资源管理器”**中，展开 **sample** 项目的 **/src/main/resources** 文件夹。右键单击 **federation.properties** 文件，单击**“打开方式”**，然后单击**“文本编辑器”**。

19. 在 **federation.properties** 文件中，包含以下配置项，然后保存文件：

        federation.trustedissuers.issuer=https://accounts.accesscontrol.windows.net/v2/wsfederation
        federation.trustedissuers.thumbprint=qY+Drf20Zz+A4t2we3PebCopoCugO76My+JMVsqNBFc=
        federation.trustedissuers.friendlyname=Fabrikam
        federation.audienceuris=spn:7829c758-2bef-43df-a685-717089474505
        federation.realm=spn:7829c758-2bef-43df-a685-717089474505
        federation.reply=https://localhost:8443/sample/wsfed-saml 

    > [WACOM.NOTE]
    > **audienceuris** 和 **realm** 值前面都必须为 "spn:"。

20. 现在您需要创建新的 Servlet。右键单击 **sample** 项目，依次单击**“新建”**、**“其他”**和 **Servlet**。

21. 在**“创建 Servlet”**对话框上，提供 *FederationServlet* 的**类名称**，然后单击**“完成”**。

22. 将自动打开 **FederationServlet.java** 文件。将其内容替换为以下代码，然后保存文件：

        import java.io.IOException;
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import com.microsoft.samples.federation.FederatedAuthenticationListener;
        import com.microsoft.samples.federation.FederatedLoginManager;
        import com.microsoft.samples.federation.FederatedPrincipal;
        import com.microsoft.samples.federation.FederationException;

        public class FederationServlet extends HttpServlet {
            private static final long serialVersionUID = 1L;

            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                String token = request.getParameter("wresult").toString();

                if (token == null) {
                    response.sendError(400, "You were supposed to send a wresult parameter with a token");
                }
                FederatedLoginManager loginManager = FederatedLoginManager.fromRequest(request, new SampleAuthenticationListener());

                try {
                    loginManager.authenticate(token, response);
                } catch (FederationException e) {
                    response.sendError(500, "Oops! and error occurred.");
                }
            }

            private class SampleAuthenticationListener implements FederatedAuthenticationListener {
                @Override
                public void OnAuthenticationSucceed(FederatedPrincipal principal) {
                    // ***
                    // do whatever you want with the principal object that contains the token's claims
                    // ***
                }
            }
        } 

23. 在**“项目资源管理器”**中，展开 **src/main/webapp/WEB-INF** 文件夹。右键单击 **web.xml** 文件，单击**“打开方式”**，然后单击**“文本编辑器”**。

24. 在 **web.xml** 文件中，将 *\<url-pattern\>* 节中的 **/FederationServlet** 设置替换为 **/ws-saml**。例如：

        <servlet>
            <description></description>
            <display-name>FederationServlet</display-name>
            <servlet-name>FederationServlet</servlet-name>
            <servlet-class>FederationServlet</servlet-class>
        </servlet>
        <servlet-mapping>
            <servlet-name>FederationServlet</servlet-name>
            <url-pattern>/wsfed-saml</url-pattern>
        </servlet-mapping> 

25. 在**“项目资源管理器”**中，展开 **src/main/webapp** 文件夹。右键单击 **index.jsp** 文件，单击**“打开方式”**，然后单击**“文本编辑器”**。

26. 在 **index.jsp** 文件中，将现有代码替换为以下代码，然后保存文件：

        <%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
        <%@ page import="com.microsoft.samples.federation.*"%>
        <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
        <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
            <title>Index Page</title>
        </head>
        <body>
            <h3>Welcome <strong><%=FederatedLoginManager.fromRequest(request).getPrincipal().getName()%></strong>!</h3>
            <h2>Claim list:</h2>
            <ul> 
                <% for (Claim claim : FederatedLoginManager.fromRequest(request).getClaims()) { %>
                <li><%= claim.toString()%></li>
                <%  } %>
            </ul>
        </body>
        </html> 

27. 在**“项目资源管理器”**中，展开 **src/main/webapp/WEB-INF** 文件夹。右键单击 **web.xml** 文件，单击**“打开方式”**，然后单击**“文本编辑器”**。

28. 我们现在将为应用程序启用 SSL。在 **web.xml** 文件中，在 *\<web-app\>* 节中插入以下 *\<security-constraint\>* 部分，然后保存文件：

        <security-constraint>
            <web-resource-collection>
                <web-resource-name>SSL Forwarding</web-resource-name>
                <url-pattern>/*</url-pattern>
                <http-method>POST</http-method>
                <http-method>GET</http-method>
            </web-resource-collection>
            <user-data-constraint>
                <transport-guarantee>CONFIDENTIAL</transport-guarantee>
            </user-data-constraint>
        </security-constraint> 

    > [WACOM.NOTE]
    > 开始进行之前，请确保已将 JBoss Server 配置为支持 SSL。

29. 现在我们已做好端到端运行应用程序的准备。右键单击 **sample** 项目，单击“运行身份”，然后单击“在服务器上运行”。接受之前指定的值，然后单击**“完成”**。

30. JBoss 浏览器将打开登录页。如果单击“出色的计算机”链接，您将重定向到“Office 365 标识提供程序”页，您可在其中使用目录租户凭据进行登录。例如，*john.doe@fabrikam.onmicrosoft.com*。

31. 登录后，您将作为经过身份验证的用户再次重定向到受保护页面 (**sample/index.jsp**)。

## <a name="summary"></a>摘要

本教程说明了如何创建和配置使用 Azure Active Directory 的单一登录功能的单租户 Java 应用程序。

  [针对 Azure Active Directory 的 Java 示例代码]: https://github.com/WindowsAzure/azure-sdk-for-java-samples/tree/master/WAAD.WebSSO.JAVA
  [Java 运行时环境 1.6]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
  [JBoss Application Server 7.1.1 最终版]: http://www.jboss.org/jbossas/downloads/
  [JBoss Developer Studio IDE]: https://devstudio.jboss.com/earlyaccess/
  [Office 365 PowerShell Commandlets]: http://technet.microsoft.com/zh-cn/library/jj151814.aspx
  [介绍]: #introduction
  [步骤 1：创建 Java 应用程序]: #createapp
  [步骤 2：在公司的目录租户中设置应用程序]: #provisionapp
  [步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序]: #protectapp
  [摘要]: #summary
