<properties
	pageTitle="使用 .NET 的移动服务和 Azure Active Directory 中基于角色的访问控制（Windows 应用商店）| Azure"
	description="了解如何使用 .NET 后端通过移动服务基于 Windows 应用商店应用程序中的 Azure Active Directory 角色控制访问。"
	documentationCenter="windows"
	authors="wesmc7777"
	manager="dwrede"
	editor=""
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="12/07/2015"
	wacn.date="01/29/2016"/>

# 使用 JavaScript 的移动服务和 Azure Active Directory 中基于角色的访问控制

[AZURE.INCLUDE [mobile-services-selector-rbac](../includes/mobile-services-selector-rbac.md)]

##概述

基于角色的访问控制 (RBAC) 是对用户可充当的角色分配权限的做法。它会明确定义特定类别的用户可以或者不可以执行哪些操作。本教程将指导你了解如何将基本 RBAC 添加到 Azure 移动服务。

本教程将演示基于角色的访问控制，检查每个用户在 Azure Active Directory (AAD) 中定义的“销售”组的成员资格。访问检查将在 .NET 移动服务后端中使用 Azure Active Directory 的 [Graph REST API] 来完成。只有属于“销售”组的用户才能查询数据。


>[AZURE.NOTE]本教程旨在扩充身份验证知识以加入授权实践。你应该先使用 Azure Active Directory 身份验证提供程序完成[向应用程序添加身份验证]教程。本教程将继续更新[向应用程序添加身份验证]教程中使用的 TodoItem 应用程序。

##先决条件

本教程需要的内容如下：

* 在 Windows 8.1 上运行的 Visual Studio 2013。
* 使用 Azure Active Directory 身份验证提供程序完成[向应用程序添加身份验证]教程。
 



##为集成的应用程序生成密钥


在学习[向应用程序添加身份验证]教程的过程中，你在完成[注册以使用 Azure Active Directory 登录名]步骤时为集成的应用程序创建了注册。在本部分中，你将生成在使用该集成应用程序客户端 ID 读取目录信息时所用的密钥。

[AZURE.INCLUDE [mobile-services-generate-aad-app-registration-access-key](../includes/mobile-services-generate-aad-app-registration-access-key.md)]



##创建具有成员资格的销售组

[AZURE.INCLUDE [mobile-services-aad-rbac-create-sales-group](../includes/mobile-services-aad-rbac-create-sales-group.md)]



##在移动服务上创建自定义授权属性 

在本部分中，你将创建一个可用来针对移动服务操作执行访问权限检查的新自定义授权属性。该属性将基于传递给它的角色名称查找 Active Directory 组。然后，将基于该组的成员资格执行访问权限检查。

1. 在 Visual Studio 中，右键单击移动服务 .NET 后端项目，然后单击“管理 NuGet 包”。

2. 在“NuGet Package Manager”对话框中的搜索条件内，输入 **ADAL** 以查找并安装移动服务的 **Active Directory 身份验证库**。最近我们已使用 3.3.205061641-alpha（预发行版）ADAL 包对本教程进行测试。

3. 在 Visual Studio 中，右键单击你的移动服务项目，然后依次单击“添加”和“新建文件夹”。将新文件夹命名为 **Utilities**。

4. 在 Visual Studio 中，右键单击新的 **Utilities** 文件夹，并添加名为 **AuthorizeAadRole.cs** 的新类文件。

    ![][0]

5. 在 AuthorizeAadRole.cs 文件的顶部添加以下 `using` 语句。

        using System.Net;
        using System.Net.Http;
        using System.Web.Http;
        using System.Web.Http.Controllers;
        using System.Web.Http.Filters;
		using Newtonsoft.Json;
        using Microsoft.WindowsAzure.Mobile.Service.Security;
        using Microsoft.WindowsAzure.Mobile.Service;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using System.Globalization;
		using System.IO;

6. 在 AuthorizeAadRole.cs 中，将以下枚举类型添加到 Utilities 命名空间。在此示例中，我们只需处理 **Sales** 角色。其他各项只是你可能要使用的组的示例。

        public enum AadRoles
        {
            Sales,
            Management,
            Development
        }

7. 在 AuthorizeAadRole.cs 中，将以下 `AuthorizeAadRole` 类定义添加到 Utilities 命名空间。

        [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = false, Inherited = true)]
        public class AuthorizeAadRole : AuthorizationFilterAttribute
        {
            private bool isInitialized;
            private bool isHosted;
	        private ApiServices services = null;
	
	        // Constants used with ADAL and the Graph REST API for AAD
	        private const string AadInstance = "https://login.chinacloudapi.cn/{0}";
	        private const string GraphResourceId = "https://graph.chinacloudapi.cn/";
	        private const string APIVersion = "?api-version=2013-04-05";
	
	        // App settings pulled from the Mobile Service
	        private string tenantdomain;
	        private string clientid;
	        private string clientkey;
	        private Dictionary<int, string> groupIds = new Dictionary<int, string>();
	
	        private string token = null;

            public AuthorizeAadRole(AadRoles role)
            {
                this.Role = role;
            }

	        // private class used to serialize the Graph REST API web response
	        private class MembershipResponse
	        {
	            public bool value;
	        }

            public AadRoles Role { get; private set; }

            // Generate a local dictionary for the role group ids configured as 
            // Mobile Service app settings
            private void InitGroupIds()
            {
            }

            // Use ADAL and the authentication app settings from the Mobile Service to 
            // get an AAD access token
            private string GetAADToken()
            {
            }

            // Given an AAD user id, check membership against the group associated with the role.
            private bool CheckMembership(string memberId)
            {
            }

            // Called when the user is attempting authorization
            public override void OnAuthorization(HttpActionContext actionContext)
            {
            }
        }



8. 在 AuthorizeAadRole.cs 中，按如下所示更新 `AuthorizeAadRole` 类中的 `InitGroupIds` 方法。此方法会创建一个字典，以将组 ID 映射到每个角色。

        private void InitGroupIds()
        {
            string groupId;
            
            if (services == null)
                return;

            if (!groupIds.ContainsKey((int)AadRoles.Sales))
            {
                if (services.Settings.TryGetValue("AAD_SALES_GROUP_ID", out groupId))
                {
                    groupIds.Add((int)AadRoles.Sales, groupId);
                }
                else
                    services.Log.Error("AAD_SALES_GROUP_ID app setting not found.");
            }
        }


9. 在 AuthorizeAadRole.cs 中，更新 `AuthorizeAadRole` 类中的 `GetAADToken` 方法。此方法使用存储在移动服务中的应用程序设置来获取从 ADAL 访问 AAD 的令牌。

    >[AZURE.NOTE]默认情况下，ADAL for .NET 包含内存中令牌缓存，以帮助减轻 Active Directory 的额外网络流量。但是，你可以编写自己的缓存实现，或完全禁用缓存。有关详细信息，请参阅 [ADAL for .NET]。

        // Use ADAL and the authentication app settings from the Mobile Service to get an AAD access token
        private async Task<string> GetAADToken()
        {
            // Try to get the required AAD authentication app settings from the mobile service.  
            if (!(services.Settings.TryGetValue("AAD_CLIENT_ID", out clientid) &
                  services.Settings.TryGetValue("AAD_CLIENT_KEY", out clientkey) &
                  services.Settings.TryGetValue("AAD_TENANT_DOMAIN", out tenantdomain)))
            {
                services.Log.Error("GetAADToken() : Could not retrieve mobile service app settings.");
                return null;
            }

            ClientCredential clientCred = new ClientCredential(clientid, clientkey);
            string authority = String.Format(CultureInfo.InvariantCulture, AadInstance, tenantdomain);
            AuthenticationContext authContext = new AuthenticationContext(authority);
            AuthenticationResult result = await authContext.AcquireTokenAsync(GraphResourceId, clientCred);

            if (result != null)
                token = result.AccessToken;
            else
                services.Log.Error("GetAADToken() : Failed to return a token.");

            return token;
        }

10. 在 AuthorizeAadRole.cs 中，更新 `AuthorizeAadRole` 类中的 `CheckMembership` 方法。此方法接收用户的对象 ID。然后，它使用 AAD Graph Rest API 来检查该对象 ID，以查看组的成员 ID 是否与 `AuthorizeAadRole` 类中选择的角色相关联。

        private bool CheckMembership(string memberId)
        {
            bool membership = false;
            string url = GraphResourceId + tenantdomain + "/isMemberOf" + APIVersion;
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);

            // Use the Graph REST API to check group membership in the AAD
            try
            {
                request.Method = "POST";
                request.ContentType = "application/json";
                request.Headers.Add("Authorization", token);
                using (var sw = new StreamWriter(request.GetRequestStream()))
                {
                    // Request body must have the group id and a member id to check for membership
                    string body = String.Format(""groupId":"{0}","memberId":"{1}"",
                        groupIds[(int)Role], memberId);
                    sw.Write("{" + body + "}");
                }

                WebResponse response = request.GetResponse();
                StreamReader sr = new StreamReader(response.GetResponseStream());
                string json = sr.ReadToEnd();
                MembershipResponse membershipResponse = JsonConvert.DeserializeObject<MembershipResponse>(json);
                membership = membershipResponse.value;
            }
            catch (Exception e)
            {
                services.Log.Error("OnAuthorization() exception : " + e.Message);
            }

            return membership;
        }


11. 在 AuthorizeAadRole.cs 中，使用以下代码更新 `AuthorizeAadRole` 类中的 `OnAuthorization` 方法。此代码要求调用移动服务的用户已在 AAD 上完成身份验证。然后，此代码将获取用户的 AAD 对象 ID，检查与该角色对应的 Active Directory 组的成员资格。

    >[AZURE.NOTE]你可以按名称查找 Active Directory 组。但是，在许多情况下，将组 ID 存储为移动服务应用程序设置是较好的做法。这是因为组名称很可能会更改，而 ID 会保持相同。

        public override void OnAuthorization(HttpActionContext actionContext)
        {
            if (actionContext == null)
            {
                throw new ArgumentNullException("actionContext");
            }

            services = new ApiServices(actionContext.ControllerContext.Configuration);

            // Check whether we are running in a mode where local host access is allowed 
			// through without authentication.
            if (!this.isInitialized)
            {
                HttpConfiguration config = actionContext.ControllerContext.Configuration;
                this.isHosted = config.GetIsHosted();
                this.isInitialized = true;
            }

            // No security when hosted locally
            if (!this.isHosted && actionContext.RequestContext.IsLocal)
            {
                services.Log.Warn("AuthorizeAadRole: Local Hosting.");
                return;
            }

            ApiController controller = actionContext.ControllerContext.Controller as ApiController;
            if (controller == null)
            {
                services.Log.Error("AuthorizeAadRole: No ApiController.");
            }

            bool isAuthorized = false;
            try
            {
                // Initialize a mapping for the group id to our enumerated type
                InitGroupIds();

                // Retrieve a AAD token from ADAL
                GetAADToken();
                if (token == null)
            {
                    services.Log.Error("AuthorizeAadRole: Failed to get an AAD access token.");
            }
                else
                {
            // Check group membership to see if the user is part of the group that corresponds to the role
                    if (!string.IsNullOrEmpty(groupIds[(int)Role]))
            {
                ServiceUser serviceUser = controller.User as ServiceUser;
                if (serviceUser != null && serviceUser.Level == AuthorizationLevel.User)
                {
                    var idents = serviceUser.GetIdentitiesAsync().Result;
                            AzureActiveDirectoryCredentials clientAadCredentials =
                                idents.OfType<AzureActiveDirectoryCredentials>().FirstOrDefault();
                    if (clientAadCredentials != null)
                    {
                                isAuthorized = CheckMembership(clientAadCredentials.ObjectId);
                        }
                    }
                }
            }
            }
            catch (Exception e)
            {
                services.Log.Error(e.Message);
            }
            finally
            {
                if (isAuthorized == false)
            {
                    services.Log.Error("Denying access");

                actionContext.Response = actionContext.Request
                    .CreateErrorResponse(HttpStatusCode.Forbidden, 
						"User is not logged in or not a member of the required group");
            }
        }
        }

12. 保存对 AuthorizeAadRole.cs 所做的更改。

##将基于角色的访问检查添加到数据库操作

1. 在 Visual Studio 中，展开移动服务项目中的 **Controllers** 文件夹。打开 TodoItemController.cs。

2. 在 TodoItemController.cs 中，为包含自定义授权属性的 utilities 命名空间添加 `using` 语句。

        using todolistService.Utilities;

3. 在 TodoItemController.cs 中，可以根据检查访问权限的方式，将属性添加到控制器类或单个方法。如果你要让所有控制器操作根据相同的角色来检查访问权限，只需将属性添加到类。请按如下所示将属性添加到类，以测试本教程。

        [AuthorizeAadRole(AadGroups.Sales)]
        public class TodoItemController : TableController<TodoItem>

    如果你只想要检查插入、更新和删除操作的访问权限，则应使用以下方式仅设置这些方法的属性。

        // PATCH tables/TodoItem
        [AuthorizeAadRole(AadGroups.Sales)]
        public Task<TodoItem> PatchTodoItem(string id, Delta<TodoItem> patch)
        {
            return UpdateAsync(id, patch);
        }

        // POST tables/TodoItem
        [AuthorizeAadRole(AadGroups.Sales)]
        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
        {
            TodoItem current = await InsertAsync(item);
            return CreatedAtRoute("Tables", new { id = current.Id }, current);
        }

        // DELETE tables/TodoItem
        [AuthorizeAadRole(AadGroups.Sales)]
        public Task DeleteTodoItem(string id)
        {
            return DeleteAsync(id);
        }


4. 保存 TodoItemController.cs 并生成移动服务，以验证是否没有语法错误。
5. 将移动服务发布到 Azure 帐户。


##测试客户端的访问

[AZURE.INCLUDE [mobile-services-aad-rbac-test-app](../includes/mobile-services-aad-rbac-test-app.md)]







<!-- Images -->
[0]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-aad-rbac/add-authorize-aad-role-class.png

<!-- URLs. -->
[向应用程序添加身份验证]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/
[How to Register with the Azure Active Directory]: /documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Azure Management Portal]: https://manage.windowsazure.cn/
[Directory Sync Scenarios]: http://msdn.microsoft.com/zh-cn/library/azure/jj573653.aspx
[Store Server Scripts]: /documentation/articles/mobile-services-store-scripts-source-control/
[注册以使用 Azure Active Directory 登录名]: /documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Graph REST API]: http://msdn.microsoft.com/zh-cn/library/azure/hh974478.aspx
[IsMemberOf]: http://msdn.microsoft.com/zh-cn/library/azure/dn151601.aspx
[ADAL for .NET]: https://msdn.microsoft.com/zh-cn/library/azure/jj573266.aspx

<!---HONumber=Mooncake_0118_2016-->