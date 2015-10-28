<properties 
	pageTitle="代表用户访问 SharePoint | 移动开发人员中心" 
	description="了解如何代表用户调用 SharePoint" 
	documentationCenter="" 
	authors="mattchenderson" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/08/2015" 
	wacn.date="10/03/2015"/>

# 代表用户访问 SharePoint

本主题说明如何代表当前登录的用户访问 SharePoint API。如果你更愿意观看视频，右侧的视频片段提供了与本教程相同的步骤。在视频中，Mat Velloso 将引导你更新 Windows 应用商店应用程序，以便与 SharePoint Online 交互。


在本教程中，你将要更新“使用 Active Directory 身份验证库单一登录对应用程序进行身份验证”教程中的应用程序，以便在添加 TodoItem 时在 SharePoint Online 中创建一个 Word 文档。

本教程将引导你完成这些基本步骤，以启用对 SharePoint 的委托访问：

1. [注册应用程序以便对 SharePoint 进行委托访问]
2. [将 SharePoint 信息添加到移动服务]
3. [获取访问令牌并调用 SharePoint API]
4. [创建并上载 Word 文档]
5. [测试应用程序]

本教程需要的内容如下：

* 在 Windows 8.1 上运行的 Visual Studio 2013。
* 有效的 [SharePoint Online] 订阅
* 已完成[使用 Active Directory 身份验证库单一登录对应用程序进行身份验证]教程。应该使用 SharePoint 订阅提供的租户。

## <a name="configure-permissions"></a>配置应用程序以便对 SharePoint 进行委托访问
默认情况下，你从 AAD 收到的令牌具有有限的权限。若要访问第三方资源或 SaaS 应用程序（例如 SharePoint Online），你必须明确允许此类访问。

1. 在 [Azure 管理门户]的“Active Directory”部分中，选择你的租户。导航至你为移动服务创建的 Web 应用程序。

    ![][0]

2. 在“配置”选项卡中，向下滚动到“对其他应用程序的权限”部分。选择“Office 365 SharePoint Online”，然后授与“编辑或删除用户的文件”委托权限。然后单击“保存”。

    ![][1]

现在，你已配置由 AAD 颁发 SharePoint 访问令牌给移动服务。

## <a name="store-credentials"></a>将 SharePoint 信息添加到移动服务

若要调用 SharePoint，你必须指定移动服务需要联系的终结点。还需要能够证明移动服务的标识。请使用客户端 ID 和客户端机密对来完成此操作。你已在 AAD 登录设置期间获取并存储了移动服务的客户端 ID。由于这是一些机密凭据，因此不应以明文形式存储在代码中。而是要将这些值设为移动服务的应用程序设置。

1. 返回租户的“AAD 应用程序”选项卡，然后为移动服务选择 Web 应用程序。

2. 在“配置”下，向下滚动到“密钥”。你将通过生成新密钥来获取客户端机密。请注意，在创建密钥并退出页面后，你将无法再从门户访问此密钥。必须在创建后，将此值复制并存储在安全的位置。选择密钥的持续时间，然后单击“保存”，并复制生成的值。

    ![][2]

3. 在管理门户的“移动服务”部分中，导航至“配置”选项卡，然后向下滚动至“应用程序设置”。你可以在此处提供密钥-值对，以帮助参考所需的凭据。

    ![][3]

4. 将 SP\_Authority 设置为 AAD 租户的机构终结点。此项应与客户端应用程序所用的机构值相同。其格式为 https://login.windows.net/contoso.onmicrosoft.com

5. 将 SP\_ClientSecret 设置为你先前获取的客户端机密值。

6. 将 SP\_SharePointURL 设置为 SharePoint 站点的 URL。其格式应为 https://contoso-my.sharepoint.com

你可以使用 ApiServices.Settings 在代码中再次获取这些值。

## <a name="obtain-token"></a>获取访问令牌并调用 SharePoint API

若要访问 SharePoint，你需要一个以 SharePoint 为目标受众的特殊访问令牌。若要获取此令牌，你需要使用移动服务的标识以及为用户颁发的令牌，来回调 AAD。

1. 在 Visual Studio 中，打开移动服务后端项目。

[AZURE.INCLUDE [mobile-services-dotnet-adal-install-nuget](../includes/mobile-services-dotnet-adal-install-nuget.md)]

2. 在该移动服务后端项目中，创建名为 SharePointUploadContext 的新类。在该类中添加以下项：

        private String accessToken;
        private String mySiteApiPath;
        private String clientId;
        private String clientSecret;
        private String sharepointURL;
        private String authority;
        private const string getFilesPath = "/getfolderbyserverrelativeurl('Documents')/Files";

        public static async Task<SharePointUploadContext> createContext(ServiceUser user, ServiceSettingsDictionary settings)
        {
            //Get the current access token
            AzureActiveDirectoryCredentials creds = (await user.GetIdentitiesAsync()).OfType<AzureActiveDirectoryCredentials>().FirstOrDefault();
            string userToken = creds.AccessToken;
            return new SharePointUploadContext(userToken, settings);
        }

        private SharePointUploadContext(string userToken, ServiceSettingsDictionary settings)
        {
            //Call ADAL and request a token to SharePoint with the access token
            AuthenticationContext ac = new AuthenticationContext(authority);
            AuthenticationResult ar = ac.AcquireToken(sharepointURL, new ClientCredential(clientId, clientSecret), new UserAssertion(userToken));
            accessToken = ar.AccessToken;
            string upn = ar.UserInfo.UserId;
            mySiteApiPath = "/personal/" + upn.Replace('@','_').Replace('.','_') + "/_api/web"; 
            clientId = settings.AzureActiveDirectoryClientId;
            clientSecret = settings["SP_ClientSecret"];
            sharepointURL = settings["SP_SharePointURL"];
            authority = settings["SP_Authority"];
        }

3. 现在，请创建用于将文件添加到用户文档库的方法：

        public async Task<bool> UploadDocument(string docName, byte[] document)
        {
            string uploadUri = sharepointURL + mySiteApiPath + getFilesPath + string.Format(@"/Add(url='{0}.docx', overwrite=true)", docName);
            using (HttpClient client = new HttpClient())
            {
                Func<HttpRequestMessage> requestCreator = () =>
                {
                    HttpRequestMessage UploadRequest = new HttpRequestMessage(HttpMethod.Post, uploadUri);
                    UploadRequest.Content = new System.Net.Http.ByteArrayContent(document);
                    UploadRequest.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
                    UploadRequest.Content.Headers.ContentType.Parameters.Add(new NameValueHeaderValue("odata", "verbose"));
                    return UploadRequest;
                };
                using (HttpRequestMessage uploadRequest = requestCreator.Invoke())
                {
                    uploadRequest.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
                    HttpResponseMessage uploadResponse = await client.SendAsync(uploadRequest);
                }
            }
            return true;
        }

## <a name="create-document"></a>创建并上载 Word 文档

若要创建 Word 文档，你将要使用 OpenXML NuGet 包。请打开 NuGet Manager 并搜索 DocumentFormat.OpenXml，然后安装此包。

1. 将以下代码添加到 TodoItemController。这将会基于 TodoItem 创建一个 Word 文档。该文档的文本将是项的名称。

        private static byte[] CreateWordDocument(TodoItem todoItem)
        {
            byte[] document;
            using (MemoryStream generatedDocument = new MemoryStream())
            {
                using (WordprocessingDocument package = WordprocessingDocument.Create(generatedDocument, WordprocessingDocumentType.Document))
                {
                    MainDocumentPart mainPart = package.MainDocumentPart;
                    if (mainPart == null)
                    {
                        mainPart = package.AddMainDocumentPart();
                        new Document(new Body()).Save(mainPart);
                    }
                    Body body = mainPart.Document.Body;
                    Paragraph p =
                        new Paragraph(
                            new Run(
                                new Text(todoItem.Text)));
                    body.Append(p);
                    mainPart.Document.Save();
                }
                document = generatedDocument.ToArray();
            }
            return document;
        }

2. 将 PostTodoItem 替换为以下内容：

        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
        {
            TodoItem current = await InsertAsync(item);
            
            SharePointUploadContext context = await SharePointUploadContext.createContext((ServiceUser)this.User, Services.Settings);
            byte[] document = CreateWordDocument(item);
            bool uploadResult = await context.UploadDocument(item.Id, document);
            
            return CreatedAtRoute("Tables", new { id = current.Id }, current);
        }

## <a name="test-application"></a>测试应用程序

1. 将更改发布到后端，然后运行客户端应用程序。根据提示登录，然后插入新的 TodoItem。

2. 导航至 SharePoint 站点，并以同一用户的身份登录。

3. 选择“OneDrive”选项卡。在“文档文件夹”下，你应会看到具有 GUID 标题的 Word 文档。当你打开此文档时，应会看到 TodoItem 的文本。

    ![][4]


<!-- Images. -->

[0]: ./media/mobile-services-dotnet-backend-calling-sharepoint-on-behalf-of-user/aad-web-application.png
[1]: ./media/mobile-services-dotnet-backend-calling-sharepoint-on-behalf-of-user/aad-sharepoint-permissions.png
[2]: ./media/mobile-services-dotnet-backend-calling-sharepoint-on-behalf-of-user/aad-manage-secret-key.png
[3]: ./media/mobile-services-dotnet-backend-calling-sharepoint-on-behalf-of-user/mobile-services-app-settings-sharepoint.png
[4]: ./media/mobile-services-dotnet-backend-calling-sharepoint-on-behalf-of-user/sharepoint-document-created.png

<!-- Anchors. -->

[注册应用程序以便对 SharePoint 进行委托访问]: #configure-permissionss
[将 SharePoint 信息添加到移动服务]: #store-credentials
[获取访问令牌并调用 SharePoint API]: #obtain-token
[创建并上载 Word 文档]: #create-document
[测试应用程序]: #test-application

<!-- URLs. -->
[Azure 管理门户]: https://manage.windowsazure.cn/
[SharePoint Online]: http://office.microsoft.com/zh-cn/sharepoint/
[使用 Active Directory 身份验证库单一登录对应用程序进行身份验证]: /documentation/articles/mobile-services-windows-store-dotnet-adal-sso-authentication
<!---HONumber=71-->