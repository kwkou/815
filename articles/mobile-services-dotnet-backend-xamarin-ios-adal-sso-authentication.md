<properties urlDisplayName="Active Directory SSO Authentication with ADAL" pageTitle="使用 Active Directory 身份验证库单一登录对应用程序进行身份验证 (Xamarin.iOS) | 移动开发人员中心" metaKeywords="" description="了解如何在 Xamarin.iOS 应用程序中使用 ADAL 对单一登录的用户进行身份验证。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Authenticate your app with Active Directory Authentication Library Single Sign-On" authors="wesmc,mahender" manager="dwrede" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-xamarin-ios" ms.devlang="dotnet" ms.topic="article" ms.date="09/29/2014" ms.author="wesmc,mahender" />

# 使用 Active Directory 身份验证库单一登录对应用程序进行身份验证

[WACOM.INCLUDE [mobile-services-selector-adal-sso](../includes/mobile-services-selector-adal-sso.md)]

在本教程中，你将使用 Active Directory 身份验证库向快速入门项目添加身份验证功能。 

若要能够对用户进行身份验证，必须向 Azure Active Directory (AAD) 注册你的应用程序。此过程分为两个步骤。首先，你必须注册你的移动服务，并公开其上的权限。其次，你必须注册你的 Xamarin.iOS 应用程序，并授予它对这些权限的访问权限


>[AZURE.NOTE] 本教程旨在帮助你更好地了解如何使用移动服务对 Xamarin.iOS 应用程序进行单一登录 Azure Active Directory 身份验证。如果这是你第一次体验移动服务，请先完成[移动服务入门]教程。

本教程将指导你完成以下基本步骤：

1. [向 Azure Active Directory 注册你的移动服务]
2. [向 Azure Active Directory 注册你的应用程序]
3. [将移动服务配置为要求身份验证]
4. [向客户端应用程序添加身份验证代码]
5. [测试使用身份验证的客户端]

本教程需要以下项：

* XCode 4.5 和 iOS 6.0（或更高版本） 
* OS X 上安装了 [Xamarin 扩展]的 Visual Studio 或 [Xamarin Studio]
* 完成[移动服务入门]或[数据处理入门]教程。
* Microsoft Azure 移动服务 SDK
* [用于 iOS 的 Active Directory 身份验证库的 Xamarin 绑定]。

[WACOM.INCLUDE [mobile-services-dotnet-adal-register-service](../includes/mobile-services-dotnet-adal-register-service.md)]

[WACOM.INCLUDE [mobile-services-dotnet-adal-register-client](../includes/mobile-services-dotnet-adal-register-client.md)]

## <a name="require-authentication"></a>将移动服务配置为要求身份验证

[WACOM.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)]

## <a name="add-authentication-code"></a>向客户端应用程序添加身份验证代码

1. 将 Active Directory 身份验证库的 Xamarin 绑定添加到你的 Xamarin.iOS 项目。在 Visual Studio 2013 中，右键单击**"引用"**，然后选择**"添加引用"**。然后浏览到绑定库，并单击**"添加"**。请务必同时从 ADAL 源添加情节提要。

2. 将以下代码添加到 QSTodoService 类： 

        private MobileServiceUser user;
        public MobileServiceUser User { get { return user; } }

        public async Task Authenticate()
        {
            var aadAuthority = @"<INSERT-AUTHORITY-HERE>";
            var aadResource = @"<INSERT-RESOURCE-URI-HERE>";
            var aadClientId = @"<INSERT-CLIENT-ID-HERE>";
            var aadRedirect = @"<INSERT-REDIRECT-URI-HERE>";

            ADAuthenticationError error;
            var aadContext = ADAuthenticationContext.AuthenticationContextWithAuthority(aadAuthority, out error);
            if (error != null)
            {
                throw new InvalidOperationException("Error creating the AAD context: " + error.ErrorDetails);
            }

            var tcs = new TaskCompletionSource<string>();
            aadContext.AcquireTokenWithResource(aadResource, aadClientId, new NSUrl(aadRedirect), result =>
                {
                    if (result.Status == ADAuthenticationResultStatus.SUCCEEDED)
                    {
                        tcs.SetResult(result.AccessToken);
                    }
                    else
                    {
                        string errorDetails;
                        if (result.Status == ADAuthenticationResultStatus.USER_CANCELLED)
                        {
                            errorDetails = "Authentication cancelled by user";
                        }
                        else
                        {
                            errorDetails = result.Error.ErrorDetails;
                        }

                        tcs.SetException(new InvalidOperationException("Error during authentication: " + errorDetails));
                    }
                });
            string accessToken = await tcs.Task;

            var token = new JObject(new JProperty("access_token", accessToken));

            try
            {
                user = await this.client.LoginAsync(MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory, token);
            }
            catch (Exception ex)
            {
                Console.Error.WriteLine (@"ERROR - AUTHENTICATION FAILED {0}", ex.Message);
            }
        }

6. 在上面的  `AuthenticateAsync` 方法的代码中，将 **INSERT-AUTHORITY-HERE** 替换为在其中进行应用程序设置的租户的名称，格式应为 https://login.windows.net/tenant-name.onmicrosoft.com。可以在 [Azure 管理门户]中从 Azure Active Directory 的"域"选项卡复制出此值。

7. 在上面的  `AuthenticateAsync` 方法的代码中，将 **INSERT-RESOURCE-URI-HERE** 替换为你的移动服务的**"应用程序 ID URI"**。如果你按照[如何向 Azure Active Directory 注册]主题进行操作，你的应用程序 ID URI 应该类似于 https://todolist.azure-mobile.cn/login/aad。

8. 在上面的  `AuthenticateAsync` 方法的代码中，将 **INSERT-CLIENT-ID-HERE** 替换为你从本机客户端应用程序复制的客户端 ID。

9. 在上面的  `AuthenticateAsync` 方法的代码中，将 **INSERT-REDIRECT-URI-HERE** 替换为你的移动服务的 /login/done 终结点。这应类似于 https://todolist.azure-mobile.cn/login/done。


3. 在 QSTodoListViewController 中，通过在对 RefreshAsync() 的调用之前添加以下代码来修改 **ViewDidLoad**；

        if (QSTodoService.DefaultService.User == null)
        {
            await QSTodoService.DefaultService.Authenticate();
        }

## <a name="test-client"></a>测试使用身份验证的客户端

1. 在"运行"菜单中，单击"运行"以启动应用程序 
2. 你将收到要你登录 Azure Active Directory 的提示。  
3. 该应用程序将进行身份验证并返回 Todo 项目。

   ![](./media/mobile-services-dotnet-backend-xamarin-ios-adal-sso-authentication/mobile-services-app-run.png)

<!-- Anchors. -->
[向 Azure Active Directory 注册你的移动服务]: #register-mobile-service-aad
[向 Azure Active Directory 注册你的应用程序]: #register-app-aad
[将移动服务配置为要求身份验证]: #require-authentication
[向客户端应用程序添加身份验证代码]: #add-authentication-code
[测试使用身份验证的客户端]: #test-client

<!-- URLs. -->
[数据处理入门]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-data/
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-ios-get-started/
[如何向 Azure Active Directory 注册]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Azure 管理门户]: https://manage.windowsazure.cn/
[用于 iOS 的 Active Directory 身份验证库的 Xamarin 绑定]: https://github.com/AzureADSamples/NativeClient-Xamarin-iOS
[Xamarin 扩展]: http://xamarin.com/visual-studio
[Xamarin Studio]: http://xamarin.com/download
