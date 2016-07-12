<properties
   pageTitle="注册你的应用程序并获取客户端 ID 和密钥以便从代码连接到 SQL 数据库 |Azure"
   description="获取客户端 ID 和密钥以便从代码访问 SQL 数据库。"
   services="sql-database"
   documentationCenter=""
   authors="stevestein"
   manager="jhubbard"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.date="03/15/2016"
   wacn.date="04/06/2016"/>

# 获取客户端 ID 和密钥以便从代码连接到 SQL 数据库

要在代码中创建并管理 SQL 数据库，必须在 Azure Active Directory (AAD) 域中注册你的应用程序，此域和创建了 Azure 资源的订阅相关联。注册应用程序时，Azure 将生成一个客户端 ID 和密钥，在代码中需要使用此 ID 和密钥对你的应用程序进行身份验证。有关详细信息，请参阅 [Azure Active Directory](/documentation/services/active-directory)。

## 注册本机客户端应用程序并获取客户端 ID

要创建新应用程序并注册，请执行以下操作：

1. 登录[经典管理门户](https://manage.windowsazure.cn)（目前，注册应用程序需要在经典管理门户中完成）。
1. 在菜单中找到“Active Directory”，然后选择它。

    ![AAD][1]

2. 选择用于对应用程序进行身份验证的目录，然后单击该目录的**名称**。

    ![目录][4]

3. 在目录页上，单击“应用程序”。

    ![应用程序][5]

4. 单击“添加”以创建新的应用程序。

    ![添加应用程序][6]

5. 提供应用的“名称”，然后选择“本机客户端应用程序”。

    ![添加应用程序][7]

6. 提供“重定向 URI”。它不需要是实际的终结点，只要是有效的 URI 即可。

    ![添加应用程序][8]

7. 完成创建此应用程序，单击“配置”，然后复制“客户端 ID”（这是在代码中需要使用的值）。

    ![获取客户端 ID][9]


1. 在页面上向下滚动并单击“添加应用程序”。
1. 选择“Microsoft 应用”。
1. 选择“Azure 服务管理 API”，然后完成向导。
2. 在“其他应用程序的权限”部分找到“Azure 服务管理 API”，然后单击“委派权限”。
3. 选择“访问 Azure 服务管理...”。

    ![权限][2]

2. 单击页面底部的“保存”。



## 注册 web 应用程序（或 Web API）并获取客户端 ID 和密钥

若要创建新应用程序并将其注册到正确的 Active Directory 中，请执行以下操作：

1. 登录[经典管理门户](https://manage.windowsazure.cn)。
1. 在菜单中找到“Active Directory”，然后选择它。

    ![AAD][1]

2. 选择用于对应用程序进行身份验证的目录，然后单击该目录的**名称**。

    ![目录][4]

3. 在目录页上，单击“应用程序”。

    ![应用程序][5]

4. 单击“添加”以创建新的应用程序。

    ![添加应用程序][6]

5. 提供此应用程序的“名称”，然后选择“Web 应用程序和/或 Web API”。

    ![添加应用程序][10]

6. 提供“登录 URL”和“应用程序 ID URI”。它不需要是实际的终结点，只要是有效的 URI 即可。

    ![添加应用程序][11]

7. 完成创建此应用程序，然后单击“配置”。

    ![配置][12]

8. 滚动到“密钥”部分，然后在“选择持续时间”列表中选择“1 年”。保存后将显示此密钥值，因此我们稍后将返回并复制此密钥。

    ![设置密钥持续时间][13]



1. 在页面上向下滚动并单击“添加应用程序”。
1. 选择“Microsoft 应用”。
1. 找到并选择“Azure 服务管理 API”，然后完成向导。
2. 在“其他应用程序的权限”部分找到“Azure 服务管理 API”，然后单击“委派权限”。
3. 选择“访问 Azure 服务管理...”。

    ![权限][2]

2. 单击页面底部的“保存”。
3. 保存完成后找到并保存此客户端 ID 和密钥：

    ![Web 应用程序机密][14]



## 获取域名

身份验证代码中有时需要用到域名。轻松标识正确域名的一种方式是：

1. 转到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 将鼠标悬停在右上角的名称上，并记下弹出窗口中显示的域。

   




## 示例控制台应用程序


要获取所需的管理库，请在 Visual Studio 中使用“程序包管理器控制台”[](http://docs.nuget.org/Consume/Package-Manager-Console)安装以下程序包（“工具”>“NuGet 程序包管理器”>“程序包管理器控制台”）：


    PM> Install-Package Microsoft.Azure.Common.Authentication –Pre


创建名为“SqlDbAuthSample”的控制台应用程序，并将“Program.cs”的内容替换为以下内容：

    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using System;
    
    namespace SqlDbAuthSample
    {
    class Program
    {
        static void Main(string[] args)
        {
            token = GetAccessTokenforNativeClient();
            // token = GetAccessTokenUsingUserCredentials(new UserCredential("<email address>"));
            // token = GetAccessTokenForWebApp();

            Console.WriteLine("Signed in as: " + token.UserInfo.DisplayableId);

            Console.WriteLine("Press Enter to continue...");
            Console.ReadLine();
        }


        // authentication variables (native)
        static string nclientId = "<your client id>";
        static string nredirectUri = "<your redirect URI>";
        static string ndomainName = "<i.e. microsoft.partner.onmschina.cn>";
        static AuthenticationResult token;

        private static AuthenticationResult GetAccessTokenforNativeClient()
        {
            AuthenticationContext authContext = new AuthenticationContext
                ("https://login.chinacloudapi.cn/" + ndomainName /* Tenant ID or AAD domain */);

            token = authContext.AcquireToken
                ("https://management.azure.com/"/* the Azure Resource Management endpoint */,
                    nclientId,
            new Uri(nredirectUri),
            PromptBehavior.Auto /* with Auto user will not be prompted if an unexpired token is cached */);

            return token;
        }


        // authentication variables (web)
        static string wclientId = "<your client id>";
        static string wkey = "<your key>";
        static string wdomainName = "<i.e. microsoft.partner.onmschina.cn>";

        private static AuthenticationResult GetAccessTokenForWebApp()
        {
            AuthenticationContext authContext = new AuthenticationContext
                ("https://login.chinacloudapi.cn/" /* AAD URI */
                + wdomainName /* Tenant ID or AAD domain */);

            ClientCredential cc = new ClientCredential(wclientId, wkey);

            AuthenticationResult token = authContext.AcquireToken(
                "https://management.azure.com/"/* the Azure Resource Management endpoint */,
                cc);

            return token;
        }


        private static AuthenticationResult GetAccessTokenUsingUserCredentials(UserCredential userCredential)
        {
            AuthenticationContext authContext = new AuthenticationContext
                ("https://login.chinacloudapi.cn/" /* AAD URI */
                + ndomainName /* Tenant ID or AAD domain */);

            AuthenticationResult token = authContext.AcquireToken(
                "https://management.azure.com/"/* the Azure Resource Management endpoint */,
                nclientId /* application client ID from AAD*/,
                new Uri(nredirectUri) /* redirect URI */,
                PromptBehavior.Auto,
                new UserIdentifier(userCredential.UserName, UserIdentifierType.RequiredDisplayableId));

            return token;
        }
    }
    }


有关与 Azure AD 身份验证相关的特定代码示例，请参阅 MSDN 上的 [SQL Server 安全博客](http://blogs.msdn.com/b/sqlsecurity)。

## 另请参阅

- [使用 C# 创建 SQL 数据库](/documentation/articles/sql-database-get-started-csharp/)
- [通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)



<!--Image references-->
[1]: ./media/sql-database-client-id-keys/aad.png
[2]: ./media/sql-database-client-id-keys/permissions.png
[3]: ./media/sql-database-client-id-keys/getdomain.png
[4]: ./media/sql-database-client-id-keys/aad2.png
[5]: ./media/sql-database-client-id-keys/aad-applications.png
[6]: ./media/sql-database-client-id-keys/add.png
[7]: ./media/sql-database-client-id-keys/add-application.png
[8]: ./media/sql-database-client-id-keys/add-application2.png
[9]: ./media/sql-database-client-id-keys/clientid.png
[10]: ./media/sql-database-client-id-keys/add-application-web.png
[11]: ./media/sql-database-client-id-keys/add-application-app-properties.png
[12]: ./media/sql-database-client-id-keys/configure.png
[13]: ./media/sql-database-client-id-keys/key-duration.png
[14]: ./media/sql-database-client-id-keys/web-secrets.png

<!---HONumber=Mooncake_0328_2016-->
