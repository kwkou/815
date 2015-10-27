<properties 
	pageTitle="使用 JavaScript 后端通过移动服务访问 Azure Active Directory Graph 信息（Windows 应用商店）| Windows Azure" 
	description="了解如何使用 Graph API 在 JavaScript 后端移动服务中访问 Azure Active Directory 信息。" 
	documentationCenter="windows" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags
	ms.service="mobile-services"
	ms.date="09/03/2015" 
	wacn.date="10/22/2015"/>

# 使用 JavaScript 后端通过移动服务访问 Azure Active Directory Graph 信息


[AZURE.INCLUDE [mobile-services-selector-aad-graph](../includes/mobile-services-selector-aad-graph.md)]
##概述

与移动服务提供的其他身份提供程序一样，Azure Active Directory (AAD) 提供程序还支持丰富的Graph API，它可用于以编程方式访问该目录。在本教程中，你将要更新 ToDoList 应用程序，使经过身份验证的用户在该应用程序中以个性化的方式返回你使用 [Graph REST API] 从目录中检索到的其他用户信息。

有关 Azure AD Graph API 的详细信息，请参阅 [Azure Active Directory Graph 团队博客]。

>[AZURE.NOTE]本教程旨在扩充使用 Azure Active Directory 进行身份验证的知识。你应该已使用 Azure Active Directory 身份验证提供程序完成[向应用程序添加身份验证]教程。本教程将继续更新[向应用程序添加身份验证]教程中使用的 TodoItem 应用程序。



##先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [向应用程序添加身份验证]<br/>向 TodoList 示例应用程序添加登录要求。

+ [调用自定义 API]<br/>演示如何调用自定义 API。



##在 AAD 中为应用程序注册生成访问密钥


在学习[向应用程序添加身份验证]教程的过程中，你在完成[注册以使用 Azure Active Directory 登录名]步骤时为集成的应用程序创建了注册。在本部分中，你将生成在使用该集成应用程序客户端 ID 读取目录信息时所用的密钥。

[AZURE.INCLUDE [mobile-services-generate-aad-app-registration-access-key](../includes/mobile-services-generate-aad-app-registration-access-key.md)]



##创建 GetUserInfo 自定义 API

在本部分中，你将创建 GetUserInfo 自定义 API ，该 API 将使用 [Graph REST API] 从 AAD 中检索有关用户的其他信息。

如果你从未将自定义 API 与移动服务结合使用过，则应考虑在完成本部分之前查看[自定义 API 教程]。

1. 在 [Azure 管理门户]中，为你的移动服务创建新的 GetUserInfo 自定义 API 并将 get 方法的权限设为**仅限经过身份验证的用户**。

    ![][0]

2. 打开新 GetUserInfo API 的脚本编辑器，将以下变量添加到脚本顶部。

        var appSettings = require('mobileservice-config').appSettings;
        var tenant_domain = appSettings.AAD_TENANT_DOMAIN;
        var clientID = appSettings.AAD_CLIENT_ID;
        var key = appSettings.AAD_CLIENT_KEY;



3. 为 `getAADToken` 函数添加以下定义。

        function getAADToken(callback) {
            var req = require("request");
            var options = {
                url: "https://login.windows.net/" + tenant_domain + "/oauth2/token?api-version=1.0",
                method: 'POST',
                form: {
                    grant_type: "client_credentials",
                    resource: "https://graph.windows.net",
                    client_id: clientID,
                    client_secret: key
                }
            };
            req(options, function (err, resp, body) {
                if (err || resp.statusCode !== 200) callback(err, null);
                else callback(null, JSON.parse(body).access_token);
            });
        }

    根据给定的 *tenant\_domain*、集成应用程序*客户端 ID* 和应用程序*密钥*，此函数提供了一个用于读取目录信息的 Graph 访问令牌。

4. 添加以下 `getUser` 函数，以便使用 Graph API 返回用户的信息。

        function getUser(access_token, objectId, callback) {
            var req = require("request");
            var options = {
                url: "https://graph.windows.net/" + tenant_domain + "/users/" + objectId + "?api-version=1.0",
                method: 'GET',
                headers: {
                    "Authorization": "Bearer " + access_token
                }
            };
            req(options, function (err, resp, body) {
                if (err || resp.statusCode !== 200) callback(err, null);
                else callback(null, JSON.parse(body));
            });
        };

    此函数是围绕 Graph REST API 的[获取用户]终结点的瘦包装器。它用 Graph 访问令牌来获取使用用户对象 ID 的用户信息。

5. 使用其他函数按如下所示更新导出的 `get` 方法，以返回用户信息。

        exports.get = function (request, response) {
            if (request.user.level == 'anonymous') {
                response.send(statusCodes.UNAUTHORIZED, null);
                return;
            }
            var errorHandler = function (err) {
                console.log(err);
                response.send(statusCodes.INTERNAL_SERVER_ERROR, err);
            };
            request.user.getIdentities({
                success: function (identities) {
                    var objectId = identities.aad.oid;
                    getAADToken(function (err, access_token) {
                        if (err) errorHandler(err);
                        else getUser(access_token, objectId, function (err, user_info) {
                            if (err) errorHandler(err);
                            else {
                                console.log('GetUserInfo: ' + JSON.stringify(user_info, null, 4));
                                response.send(statusCodes.OK, user_info);
                              }
                            });
                    });
                },
                error: errorHandler
            });
        };


##将应用程序更新为使用 GetUserInfo


在本部分中，你将更新在 [身份验证入门] 教程中实现的 `AuthenticateAsync` 方法，以调用自定义 API 并从 AAD 返回有关用户的其他信息。

[AZURE.INCLUDE [mobile-services-aad-graph-info-update-app](../includes/mobile-services-aad-graph-info-update-app.md)]


 


##测试应用程序

[AZURE.INCLUDE [mobile-services-aad-graph-info-test-app](../includes/mobile-services-aad-graph-info-test-app.md)]




##后续步骤

在下一教程[在移动服务中配合 AAD 进行基于角色的访问控制]中，你将结合使用基于角色的访问控制与 Azure Active Directory (AAD) 来检查组成员资格，然后允许访问。



<!-- Images -->
[0]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-graph-info/create-getuserinfo.png


<!-- URLs. -->
[向应用程序添加身份验证]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/
[How to Register with the Azure Active Directory]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Azure 管理门户]: https://manage.windowsazure.cn/
[自定义 API 教程]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-call-custom-api/
[调用自定义 API]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-call-custom-api/
[Store Server Scripts]: /zh-cn/documentation/articles/mobile-services-store-scripts-source-control/
[注册以使用 Azure Active Directory 登录名]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Graph API]: http://msdn.microsoft.com/zh-cn/library/azure/hh974478.aspx
[Graph REST API]: http://msdn.microsoft.com/zh-cn/library/azure/hh974478.aspx
[获取用户]: http://msdn.microsoft.com/zh-cn/library/azure/dn151678.aspx
[在移动服务中配合 AAD 进行基于角色的访问控制]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/
[Azure Active Directory Graph 团队博客]: http://go.microsoft.com/fwlink/?LinkId=510536

<!---HONumber=74-->