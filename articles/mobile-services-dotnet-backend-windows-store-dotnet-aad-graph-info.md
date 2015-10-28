<properties 
	pageTitle="访问 Azure Active Directory Graph 信息（Windows 应用商店） | Windows Azure" 
	description="了解如何使用 Windows 应用商店应用程序中的 Graph API 访问 Azure Active Directory 信息。" 
	documentationCenter="windows" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/09/2015" 
	wacn.date="10/03/2015"/>

# 访问 Azure Active Directory Graph 信息



[AZURE.INCLUDE [mobile-services-selector-aad-graph](../includes/mobile-services-selector-aad-graph.md)]

##概述

与移动服务提供的其他身份提供程序一样，Azure Active Directory (AAD) 提供程序还支持丰富的Graph API，它可用于以编程方式访问该目录。在本教程中，你将要更新 ToDoList 应用程序，使经过身份验证的用户在该应用程序中以个性化的方式返回你使用 [Graph REST API] 从目录中检索到的其他用户信息。

有关 Azure AD Graph API 的详细信息，请参阅 [Azure Active Directory Graph 团队博客]。


>[AZURE.NOTE]本教程旨在扩充使用 Azure Active Directory 进行身份验证的知识。你应该已使用 Azure Active Directory 身份验证提供程序完成[向应用程序添加身份验证]教程。本教程将继续更新[向应用程序添加身份验证]教程中使用的 TodoItem 应用程序。




##先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [向应用程序添加身份验证]<br/>向 TodoList 示例应用程序添加登录要求。

+ [调用自定义 API]<br/>演示如何调用自定义 API。



## <a name="generate-key"></a>在 AAD 中为应用程序注册生成访问密钥


在学习[向应用程序添加身份验证]教程的过程中，你在完成[注册以使用 Azure Active Directory 登录名]步骤时为集成的应用程序创建了注册。在本部分中，你将生成在使用该集成应用程序客户端 ID 读取目录信息时所用的密钥。

[AZURE.INCLUDE [mobile-services-generate-aad-app-registration-access-key](../includes/mobile-services-generate-aad-app-registration-access-key.md)]


## <a name="create-api"></a>创建 GetUserInfo 自定义 API

在本部分中，你将创建 GetUserInfo 自定义 API ，该 API 将使用 Azure AD Graph API 从 AAD 中检索有关用户的其他信息。

如果你从未将自定义 API 与移动服务结合使用过，请在完成本部分之前，参考[自定义 API 教程]。

1. 在 Visual Studio 中，右键单击移动服务 .NET 后端项目，然后单击“管理 NuGet 包”。
2. 在“NuGet Package Manager”对话框中的搜索条件内，输入 **ADAL** 以查找并安装移动服务的 **Active Directory 身份验证库**。最近我们已使用 3.0.110281957-alpha（预发行版）ADAL 包对本教程进行测试。


3. 在 Visual Studio 中，右键单击移动服务项目的 **Controllers** 文件夹，然后单击“添加”以添加名为 `GetUserInfoController` 的新 **Microsoft Azure 移动服务自定义控制器**。客户端将调用此 API，以从 Active Directory 中获取用户信息。

4. 在新的 GetUserInfoController.cs 文件中，添加以下 `using` 语句。

        using Microsoft.WindowsAzure.Mobile.Service.Security;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using System.Globalization;
		using System.Threading.Tasks;
		using Newtonsoft.Json;
		using System.IO;

5. 在新的 GetUserInfoController.cs 文件中，添加以下 `UserInfo` 类以保存我们要从 AAD 收集的信息。

	    public class UserInfo
        {
	        public String displayName { get; set; }
	        public String streetAddress { get; set; }
	        public String city { get; set; }
	        public String state { get; set; }
	        public String postalCode { get; set; }
	        public String mail { get; set; }
	        public String[] otherMails { get; set; }
	
	        public override string ToString()
	        {
	            return "displayName : " + displayName + "\n" +
	                   "streetAddress : " + streetAddress + "\n" +
	                   "city : " + city + "\n" +
	                   "state : " + state + "\n" +
	                   "postalCode : " + postalCode + "\n" +
	                   "mail : " + mail + "\n" +
	                   "otherMails : " + string.Join(", ",otherMails);
	        }
	    }

6. 在 GetUserInfoController.cs 中，将以下成员变量添加到 `GetUserInfoController` 类。

        private const string AadInstance = "https://login.windows.net/{0}";
        private const string GraphResourceId = "https://graph.windows.net/";
        private const string APIVersion = "?api-version=2013-04-05";

        private string tenantdomain;
        private string clientid;
        private string clientkey;
        private string token = null;


7. 在 GetUserInfoController.cs 中，将以下 `GetAADToken` 方法添加到类。

        private async Task<string> GetAADToken()
        {
            // Try to get the AAD app settings from the mobile service.  
            if (!(Services.Settings.TryGetValue("AAD_CLIENT_ID", out clientid) &
                  Services.Settings.TryGetValue("AAD_CLIENT_KEY", out clientkey) &
                  Services.Settings.TryGetValue("AAD_TENANT_DOMAIN", out tenantdomain)))
            {
                Services.Log.Error("GetAADToken() : Could not retrieve mobile service app settings.");
                return null;
            }

            ClientCredential clientCred = new ClientCredential(clientid, clientkey);
            string authority = String.Format(CultureInfo.InvariantCulture, AadInstance, tenantdomain);
            AuthenticationContext authContext = new AuthenticationContext(authority);
            AuthenticationResult result = await authContext.AcquireTokenAsync(GraphResourceId, clientCred);

            if (result != null)            
                token = result.AccessToken;
            else
                Services.Log.Error("GetAADToken() : Failed to return a token.");

            return token;
        }

    此方法使用你在 [Azure 管理门户]中的移动服务上配置的应用程序设置来获取用于访问 Active Directory 的令牌。

8. 在 GetUserInfoController.cs 中，将以下 `GetAADUser` 方法添加到类。

        private async Task<UserInfo> GetAADUser()
        {
            ServiceUser serviceUser = (ServiceUser)this.User;

            // Need a user
            if (serviceUser == null || serviceUser.Level != AuthorizationLevel.User)
            {
                Services.Log.Error("GetAADUser() : No ServiceUser or wrong Authorizationlevel");
                return null;
            }

            // Get the user's AAD object id
            var idents = serviceUser.GetIdentitiesAsync().Result;
            var clientAadCredentials = idents.OfType<AzureActiveDirectoryCredentials>().FirstOrDefault();
            if (clientAadCredentials == null)
            {
                Services.Log.Error("GetAADUser() : Could not get AAD credientials for the logged in user.");
                return null;
            }

            if (token == null)
                await GetAADToken();

            if (token == null)
            {
                Services.Log.Error("GetAADUser() : No token.");
                return null;
            }

            // User the AAD Graph REST API to get the user's information
            string url = GraphResourceId + tenantdomain + "/users/" + clientAadCredentials.ObjectId + APIVersion;
            Services.Log.Info("GetAADUser() : Request URL : " + url);
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
            request.Method = "GET";
            request.ContentType = "application/json";
            request.Headers.Add("Authorization", token);
            UserInfo userinfo = null;
            try
            {
                WebResponse response = await request.GetResponseAsync();
                StreamReader sr = new StreamReader(response.GetResponseStream());
                string userjson = sr.ReadToEnd();
                userinfo = JsonConvert.DeserializeObject<UserInfo>(userjson);
                Services.Log.Info("GetAADUser user : " + userinfo.ToString());
            }
            catch(Exception e)
            {
                Services.Log.Error("GetAADUser exception : " + e.Message);
            }

            return userinfo;
        }

    此方法将获取已授权用户的 Active Directory 对象 ID，然后使用 Graph REST API 从 Active Diretory 中获取用户的信息。


9. 在 GetUserInfoController.cs 中，将 `Get` 方法替换为以下方法。此方法将使用 Graph REST API 从 Azure Active Directory 返回用户实体，并要求已授权用户调用该 API。

        // GET api/GetUserInfo
        [AuthorizeLevel(AuthorizationLevel.User)]
        public async Task<UserInfo> Get()
        {
            Services.Log.Info("Entered GetUserInfo custom controller!");
            return await GetAADUser();
        }

10. 保存更改并生成服务，以验证是否没有语法错误。
11. 将移动服务项目发布到 Azure 帐户。 


## <a name="update-app"></a>将应用程序更新为使用 GetUserInfo

在本部分中，你将更新你在[向应用程序添加身份验证]教程中实现的 `AuthenticateAsync` 方法，以调用自定义 API 并从 AAD 返回有关用户的其他信息。

[AZURE.INCLUDE [mobile-services-aad-graph-info-update-app](../includes/mobile-services-aad-graph-info-update-app.md)]
  


## <a name="test-app"></a>测试应用程序

[AZURE.INCLUDE [mobile-services-aad-graph-info-test-app](../includes/mobile-services-aad-graph-info-test-app.md)]





##<a name="next-steps"></a>后续步骤

在下一教程[在移动服务中配合 AAD 进行基于角色的访问控制]中，你将结合使用基于角色的访问控制与 Azure Active Directory (AAD) 来检查组成员资格，然后允许访问。



<!-- Anchors. -->
[Generate an access key for the App registration in AAD]: #generate-key
[Create a GetUserInfo custom API]: #create-api
[Update the app to use the custom API]: #update-app
[Test the app]: #test-app
[Next Steps]: #next-steps

<!-- Images -->


<!-- URLs. -->
[向应用程序添加身份验证]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-users
[How to Register with the Azure Active Directory]: /documentation/articles/mobile-services-how-to-register-active-directory-authentication
[Azure 管理门户]: https://manage.windowsazure.cn/
[Graph REST API]: http://msdn.microsoft.com/zh-cn/library/azure/hh974478.aspx
[自定义 API 教程]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-call-custom-api
[调用自定义 API]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-call-custom-api
[Store Server Scripts]: /documentation/articles/mobile-services-store-scripts-source-control
[注册以使用 Azure Active Directory 登录名]: /documentation/articles/mobile-services-how-to-register-active-directory-authentication
[Azure Active Directory Graph 团队博客]: http://go.microsoft.com/fwlink/?LinkId=510536
[Get User]: http://msdn.microsoft.com/zh-cn/library/azure/dn151678.aspx
[在移动服务中配合 AAD 进行基于角色的访问控制]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-aad-rbac
<!---HONumber=71-->