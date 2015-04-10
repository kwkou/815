<properties urlDisplayName="访问 Azure Active Directory Graph 信息" pageTitle="访问 Azure Active Directory Graph 信息 （Windows 应用商店） |移动开发人员中心" metaKeywords="" description="了解如何使用 Windows 应用商店应用程序中的 Graph API 访问 Azure Active Directory 信息。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Accessing Azure Active Directory Graph Information" authors="wesmc" manager="dwrede" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="dotnet" ms.topic="article" ms.date="09/29/2014" ms.author="wesmc" />

# 访问 Azure Active Directory Graph 信息

[WACOM.INCLUDE [mobile-services-selector-aad-graph](../includes/mobile-services-selector-aad-graph.md)]


与移动服务提供的其他身份提供程序一样，Azure Active Directory (AAD) 提供程序还支持丰富的[Graph API]，它可用于以编程方式访问该目录。在本教程中，您将更新 ToDoList 应用程序，根据您使用[Graph API]从目录中检索到的其他用户信息对已经过身份验证的用户的应用程序体验进行个性化定制。

>[AZURE.NOTE] 本教程旨在扩展您的 Azure Active Directory 身份验证知识。应先使用 Azure Active Directory 身份验证提供程序来完成[身份验证入门]教程。本教程将继续更新[身份验证入门]教程中使用的 TodoItem 应用程序。



本教程将指导您完成以下操作：


1. [在 AAD 中生成应用程序注册的访问键] 
2. [创建 GetUserInfo 自定义 API] 
3. [更新应用程序以使用自定义 API] 
4. [测试应用]

##先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [身份验证入门]<br/>向 TodoList 示例应用程序添加登录要求。

+ [自定义 API 教程]<br/>演示如何调用自定义 API。 



## <a name="generate-key"></a>在 AAD 中生成应用程序注册的访问键


在[身份验证入门]教程中，[注册以使用 Azure Active Directory Login]步骤"的注册后，您为集成应用程序创建了一个注册。在本部分中，您将生成一个密钥，用于读取该集成应用程序的客户端 ID 的目录信息。 

[WACOM.INCLUDE [mobile-services-generate-aad-app-registration-access-key](../includes/mobile-services-generate-aad-app-registration-access-key.md)]



## <a name="create-api"></a>创建 GetUserInfo 自定义 API

在本部分中，您将创建 GetUserInfo 自定义 API ，该 API 将使用 [Graph REST API]从 AAD 中检索有关用户的其他信息。

如果您从未使用过移动服务的自定义 API，则应考虑在完成本部分之前查看[自定义 API 教程]。

1. 在[Azure 管理门户]中，为您的移动服务创建新的 GetUserInfo 自定义 API 并将 get 方法的权限设为**仅限经过身份验证的用户**。

    ![][0]

2. 打开新 GetUserInfo API 的脚本编辑器，将以下变量添加到脚本顶部。

        var appSettings = require('mobileservice-config').appSettings;
        var tenant_domain = appSettings.AAD_TENANT_DOMAIN;
        var clientID = appSettings.AAD_CLIENT_ID;
        var key = appSettings.AAD_CLIENT_KEY;



3. 为函数添加 `getAADToken`以下定义。

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

    鉴于 *tenant_domain*、集成应用程序 *client id*和应用程序 *key*，此函数提供了一个用于读取目录信息的 Graph 访问令牌。

4. 请添加使用 Graph API 来返回用户信息的下列 `getUser`函数。

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

    此函数是 Graph REST API [获取用户] 端点周围的瘦包装器。它用 Graph 访问令牌来获取使用用户对象 ID 的用户信息。

5. 按照如下所示更新导出 `get`方法，以返回使用其他函数的用户信息。

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


## <a name="update-app"></a>更新应用程序，以使用 GetUserInfo


在本部分中，您将更新 `AuthenticateAsync`在[身份验证入门]教程中使用的方法，以调用自定义 API 并从 AAD 返回有关用户的其他信息。 

[WACOM.INCLUDE [mobile-services-aad-graph-info-update-app](../includes/mobile-services-aad-graph-info-update-app.md)]


 


## <a name="test-app"></a>测试应用

[WACOM.INCLUDE [mobile-services-aad-graph-info-test-app](../includes/mobile-services-aad-graph-info-test-app.md)]




##<a name="next-steps"></a>后续步骤

在接下来的教程"[移动服务中 AAD 提供的基于角色的访问控制]"中，您将使用 Azure Active Directory (AAD) 基于角色的访问控制，在允许访问之前检查组成员身份。 


<!-- Anchors. -->
[在 AAD 中生成应用程序注册的访问键]: #generate-key
[创建 GetUserInfo 自定义 API]: #create-api
[更新应用程序以使用自定义 API]: #update-app
[测试应用]: #test-app
[后续步骤]:#next-steps

<!-- Images -->
[0]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-graph-info/create-getuserinfo.png


<!-- URLs. -->
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/
[如何向 Azure Active Directory 注册]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Azure 管理门户]: https://manage.windowsazure.cn/
[自定义 API 教程]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-call-custom-api/
[存储服务器脚本]: /zh-cn/documentation/articles/mobile-services-store-scripts-source-control/
[注册以使用 Azure Active Directory Login]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Graph API]: http://msdn.microsoft.com/library/azure/hh974478.aspx
[Graph REST API]: http://msdn.microsoft.com/library/azure/hh974478.aspx
[获取用户]: http://msdn.microsoft.com/library/azure/dn151678.aspx
[移动服务中 AAD 提供的基于角色的访问控制]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/
