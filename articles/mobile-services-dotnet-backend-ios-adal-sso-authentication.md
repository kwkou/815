<properties
	pageTitle="使用 Active Directory 身份验证库单一登录对应用进行身份验证 (iOS) | Azure"
	description="了解如何使用 iOS 应用程序中的 ADAL 对单一登录用户进行身份验证。"
	documentationCenter="ios"
	authors="mattchenderson"
	manager="dwrede"
	editor=""
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="12/15/2015"
	wacn.date="04/01/2016"/>

# 使用 Active Directory 身份验证库单一登录对应用程序进行身份验证

[AZURE.INCLUDE [mobile-services-selector-adal-sso](../includes/mobile-services-selector-adal-sso.md)]

##概述

在本教程中，你将使用 Active Directory 身份验证库向快速入门项目添加身份验证功能。

若要能够对用户进行身份验证，必须向 Azure Active Directory (AAD) 注册你的应用程序。此过程分为两个步骤。首先，你必须注册你的移动服务，并公开其上的权限。其次，你必须注册你的 iOS 应用程序，并授予它对这些权限的访问权限


>[AZURE.NOTE]本教程旨在帮助你更好地了解如何使用移动服务对 iOS 应用程序进行单一登录 Azure Active Directory 身份验证。如果这是你第一次体验移动服务，请先完成[移动服务入门]教程。


##先决条件


本教程需要的内容如下：

* XCode 4.5 和 iOS 6.0（或更高版本）
* 完成[移动服务入门]教程。
* 完成[注册应用以使用 Azure Active Directory 帐户登录]
* Azure 移动服务 SDK
* [适用于 iOS 的 Active Directory 身份验证库]

[AZURE.INCLUDE [mobile-services-dotnet-adal-register-client](../includes/mobile-services-dotnet-adal-register-client.md)]

##将移动服务配置为要求身份验证

[AZURE.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)]

##向客户端应用程序添加身份验证代码

1. 下载[适用于 iOS 的 Active Directory 身份验证库]并将其包含在项目中。请务必同时从 ADAL 源添加情 Storyboard。

2. 在 QSTodoListViewController 中包含具有以下项的 ADAL：

        #import "ADALiOS/ADAuthenticationContext.h"

2. 然后，添加以下方法：

        - (void) loginAndGetData
        {
            MSClient *client = self.todoService.client;
            if (client.currentUser != nil) {
                return;
            }

            NSString *authority = @"<INSERT-AUTHORITY-HERE>";
            NSString *resourceURI = @"<INSERT-RESOURCE-URI-HERE>";
            NSString *clientID = @"<INSERT-CLIENT-ID-HERE>";
            NSString *redirectURI = @"<INSERT-REDIRECT-URI-HERE>";

            ADAuthenticationError *error;
            ADAuthenticationContext *authContext = [ADAuthenticationContext authenticationContextWithAuthority:authority error:&error];
            NSURL *redirectUri = [[NSURL alloc]initWithString:redirectURI];

            [authContext acquireTokenWithResource:resourceURI clientId:clientID redirectUri:redirectUri completionBlock:^(ADAuthenticationResult *result) {
                if (result.tokenCacheStoreItem == nil)
                {
                    return;
                }
                else
                {
                    NSDictionary *payload = @{
                        @"access_token" : result.tokenCacheStoreItem.accessToken
                    };
                    [client loginWithProvider:@"windowsazureactivedirectory" token:payload completion:^(MSUser *user, NSError *error) {
                        [self refresh];
                    }];
                }
            }];
        }


4. 在上面的 `loginAndGetData` 方法的代码中，将 **INSERT-AUTHORITY-HERE** 替换为在其中进行应用程序设置的租户的名称，格式应为 https://login.chinacloudapi.cn/tenant-name.onmicrosoft.com。可以在 [Azure 管理门户]中从 Azure Active Directory 的“域”选项卡复制此值。

5. 在上面的 `loginAndGetData` 方法的代码中，将 **INSERT-RESOURCE-URI-HERE** 替换为你的移动服务的“应用程序 ID URI”。如果你按照[如何向 Azure Active Directory 注册]主题进行操作，你的应用程序 ID URI 应该类似于 https://todolist.azure-mobile.net/login/aad。

6. 在上面的 `loginAndGetData` 方法的代码中，将 **INSERT-CLIENT-ID-HERE** 替换为你从本机客户端应用程序复制的客户端 ID。

7. 在上面的 `loginAndGetData` 方法的代码中，将 **INSERT-REDIRECT-URI-HERE** 替换为移动服务的 /login/done 终结点。它应该类似于 https://todolist.azure-mobile.net/login/done。


8. 在 QSTodoListViewController 中修改 `ViewDidLoad`，方法是将 `[self refresh]` 替换为以下内容：

        [self loginAndGetData];

##测试使用身份验证的客户端

1. 在“产品”菜单中，单击“运行”以启动应用程序。
2. 您将收到登录 Azure Active Directory 的提示。  
3. 该应用程序将进行身份验证并返回 Todo 项目。

   ![](./media/mobile-services-dotnet-backend-ios-adal-sso-authentication/mobile-services-app-run.png)



<!-- URLs. -->
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started/
[注册应用以使用 Azure Active Directory 帐户登录]: /documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[如何向 Azure Active Directory 注册]: /documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Azure 经典门户]: https://manage.windowsazure.cn/
[适用于 iOS 的 Active Directory 身份验证库]: https://github.com/MSOpenTech/azure-activedirectory-library-for-ios

<!---HONumber=Mooncake_0118_2016-->