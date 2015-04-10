<properties urlDisplayName="Azure Active Directory 基于角色的访问控制" pageTitle="移动服务和 Azure Active Directory（Windows 应用商店）中的基于角色的访问控制 |移动开发人员中心" metaKeywords="" description="了解如何根据您的 Windows 应用商店应用程序中的 Azure Active Directory 角色控制访问。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Role Based Access Control in Mobile Services and Azure Active Directory" authors="wesmc" manager="dwrede" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="dotnet" ms.topic="article" ms.date="09/29/2014" ms.author="wesmc" />

# 移动服务和 Azure Active Directory 中的基于角色的访问控制

[WACOM.INCLUDE [mobile-services-selector-rbac](../includes/mobile-services-selector-rbac.md)]


基于角色的访问控制 (RBAC) 是一种为用户能够拥有的角色分配权限的实践，能够很好地定义哪些特定类型的用户能做什么和不能做什么的界限。本教程将指导您如何向 Azure 移动服务中添加基本的 RBAC。

本教程将演示基于角色的访问控制，检查每个用户在 Azure Active Directory (AAD) 中定义的销售组的成员身份。访问检查将使用 Azure Active Directory 的 [Graph API]，通过移动服务后端中的 JavaScript 完成。只有属于销售角色的用户才可以查询数据。


>[AZURE.NOTE] 本教程旨在扩展您的身份验证知识，帮助您掌握授权实践。应先使用 Azure Active Directory 身份验证提供程序来完成[身份验证入门]教程。本教程将继续更新[身份验证入门]教程中使用的 TodoItem 应用程序。

本教程将指导您完成以下操作：

1. [创建销售组及成员]
2. [为集成的应用程序生成密钥]
3. [添加一个共享的脚本来检查成员资格] 
4. [向数据库操作中添加基于角色的访问检查]
5. [测试客户端访问]

本教程需要的内容如下：

* 运行在 Windows 8.1 上的 Visual Studio 2013。
* 使用 Azure Active Directory 身份验证提供程序完成[身份验证入门]教程。
* 完成[存储服务器脚本]教程，熟悉如何使用 Git 存储库来存储服务器脚本。
 


## <a name="create-group"></a>创建销售组及成员

[WACOM.INCLUDE [mobile-services-aad-rbac-create-sales-group](../includes/mobile-services-aad-rbac-create-sales-group.md)]


## <a name="generate-key"></a>为集成的应用程序生成密钥


在[身份验证入门]教程中，在完成"使用 Azure Active Directory Login]步骤"[的注册后，您为集成应用程序创建了一个注册。在本部分中，您将生成一个密钥，用于读取该集成应用程序的客户端 ID 的目录信息。 

[WACOM.INCLUDE [mobile-services-generate-aad-app-registration-access-key](../includes/mobile-services-generate-aad-app-registration-access-key.md)]





## <a name="add-shared-script"></a>将共享的脚本添加到检查成员资格的移动服务

在本部分，您将使用 Git 将名为 *rbac.js*的共享脚本文件部署到您的移动服务。这一共享的脚本文件将包含使用[Graph API]检查用户的组成员身份的函数。 

如果您不熟悉如何使用 Git 将脚本部署到移动服务，请在完成本部分之前，认真阅读[存储服务器脚本]教程。

1. 在移动服务的本地存储库的*./service/shared/*目录中，创建一个名为 *rbac.js* 的新脚本文件。
2. 将以下脚本添加到定义 `getAADToken`函数的文件顶部。鉴于 *tenant_domain*、集成应用程序 *client id*和应用程序 *key*，此函数提供了一个用于读取目录信息的 Graph 访问令牌。

    >[AZURE.NOTE] 您应缓存令牌，而不是在每次访问检查时创建一个新令牌。然后，在尝试使用 401 Authentication_ExpiredToken 响应中的令牌结果时刷新缓存，如[Graph API 错误参考]所述。为了简单起见，下面的代码中并没有体现这一点，但它将缓解 Active Directory 的额外网络流量。 

        var appSettings = require('mobileservice-config').appSettings;
        var tenant_domain = appSettings.AAD_TENANT_DOMAIN;
        var clientID = appSettings.AAD_CLIENT_ID;
        var key = appSettings.AAD_CLIENT_KEY;
        exports.SalesGroup = appSettings.AAD_GROUP_ID;
         
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


3. 添加以下代码到 *rbac.js*来定义 `getGroupId`函数。此函数根据筛选器中所用的组名称，使用访问令牌来获取组 id。
 
    >[AZURE.NOTE] 此代码按名称查找 Active Directory 组。在许多情况下，将组 id 存储为移动服务应用程序设置并只使用该组 id 可能更好。这是因为组名称可能会改变，但 id 保持不变。然而，如果组名称更改，通常至少角色范围也会发生变化，这可能还需要更新移动服务代码。   

        function getGroupId(groupname, accessToken, callback) {
            var req = require("request");
            var options = {
                url: "https://graph.windows.net/" + tenant_domain + "/groups" + 
                      "?$filter=displayName%20eq%20'" + groupname + "'" + 
	              "&api-version=2013-04-05" ,
                method: 'GET',
                headers: {
                    "Authorization": "Bearer " + accessToken,
                    "Content-Type": "application/json",
                }
            };
            req(options, function (err, resp, body) {
                if (err || resp.statusCode !== 200) callback(err, null);
                else callback(null, JSON.parse(body).value[0].objectId);
            });
        }


4. 添加以下代码到 *rbac.js*，以定义 `isMemberOf`调用 Graph REST api 的 [IsMemberOf] 端点的函数。

    此函数是 Graph REST API [IsMemberOf] 端点周围的瘦包装器。它使用 Graph 访问令牌，根据组 id 来检查用户的目录对象 id 是否在组中具有成员身份。

        function isMemberOf(access_token, userObjectId, groupObjectId, callback) {
            var req = require("request");
            var options = {
                url: "https://graph.windows.net/" + tenant_domain + "/isMemberOf" + "?api-version=2013-04-05",
                method: 'POST',
                headers: {
                    "Authorization": "Bearer " + access_token,
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({
                "groupId": groupObjectId,
                "memberId": userObjectId
                })
            };
            req(options, function (err, resp, body) {
                if (err || resp.statusCode !== 200) callback(err, null);
                else callback(null, JSON.parse(body).value);
            });
        };

    

7. 添加以下导出的 `checkGroupMembership`函数至 *rbac.js* 。  

    此函数将遮蔽其他脚本函数的使用，并从共享的脚本导出，以供执行实际访问检查的其他脚本调用。鉴于该移动服务用户对象和组 id，该脚本将针对用户的身份检索 Azure Active Directory 对象 id，并验证组的成员身份。

        exports.checkGroupMembership = function (user, groupName, callback) {
            user.getIdentities({
                success: function (identities) {
                    var objectId = identities.aad.oid;
                    getAADToken(function (err, access_token) {
                        if (err) callback(err);
		        else getGroupId(groupName, access_token, function (err, groupId) { 
                            if (err) callback(err);
                            else isMemberOf(access_token, objectId, groupId, function (err, isInGroup) {
       	                        if (err) errorHandler(err);
               	                else callback(null, isInGroup);
                            });
                        });
                    });
                },
                error: callback
            });
        }

8. 保存您对 *rbac.js*的更改。

## <a name="add-access-checking"></a>向数据库操作中添加基于角色的访问检查


当您完成[身份验证入门]教程后，你应该已经将表操作设置为要求身份验证，如下所示。

![][3]

对于需要身份验证的每个数据库操作，我们可以添加使用用户对象进行访问检查的脚本。

以下步骤演示如何使用移动服务中每个表操作的脚本，部署基于角色的访问控制。使用共享的 *rbac.js*脚本的脚本将针对每个表操作进行执行。

1. 在移动服务的本地 Git 存储库的*./service/table/*目录中，添加一个名为 *todoitem.insert.js*的新脚本文件。将以下脚本粘贴到该文件中。

        function insert(item, user, request) {
        
            var RBAC = require('../shared/rbac.js');
        
            RBAC.checkGroupMembership(user, "sales", function(err, isInGroup) {
                if (err) request.respond(err);
                else if (!isInGroup) request.respond(statusCodes.UNAUTHORIZED, null);
                else {
                    request.execute();
                }
            });
        }

2. 在移动服务的本地 Git 存储库的*./service/table/*目录中，添加一个名为 *todoitem.read.js*的新脚本文件。将以下脚本粘贴到该文件中。

        function read(query, user, request) {
        
            var RBAC = require('../shared/rbac.js');
        
            RBAC.checkGroupMembership(user, "sales", function(err, isInGroup) {
                if (err) request.respond(err);
                else if (!isInGroup) request.respond(statusCodes.UNAUTHORIZED, null);
                else {
                    request.execute();
                }
            });
        }

3. 在移动服务的本地 Git 存储库的*./service/table/*目录中，添加一个名为 *todoitem.update.js*的新脚本文件。将以下脚本粘贴到该文件中。

        function update(item, user, request) {
        
            var RBAC = require('../shared/rbac.js');
        
            RBAC.checkGroupMembership(user, "sales", function(err, isInGroup) {
                if (err) request.respond(err);
                else if (!isInGroup) request.respond(statusCodes.UNAUTHORIZED, null);
                else {
                    request.execute();
                }
            });
        }

4. 在移动服务的本地 Git 存储库的*./service/table/*目录中，添加一个名为 *todoitem.delete.js*的新脚本文件。将以下脚本粘贴到该文件中。

        function del(id, user, request) {
        
            var RBAC = require('../shared/rbac.js');
        
            RBAC.checkGroupMembership(user, "sales", function(err, isInGroup) {
                if (err) request.respond(err);
                else if (!isInGroup) request.respond(statusCodes.UNAUTHORIZED, null);
                else {
                    request.execute();
                }
            });
        }

5. 在 Git 存储库的命令行界面中，通过执行以下命令，为您添加的脚本文件添加跟踪。

        git add .

6. 在 Git 存储库的命令行界面中，通过执行以下命令提交更新。

        git commit -m "Added role based access control to table operations"
  
7. 在 Git 存储库的命令行界面中，通过执行以下命令将更新部署到移动服务的本地 Git 存储库。此命令假定您在 *master*分支中执行的更新。

        git push origin master

8. 在[Azure 管理门户]中，您应该能够导航到移动服务的表操作并在适当的位置查看更新，如下所示。 

    ![][4]

## <a name="test-client"></a>测试客户端的访问

[WACOM.INCLUDE [mobile-services-aad-rbac-test-app](../includes/mobile-services-aad-rbac-test-app.md)]





<!-- Anchors. -->
[创建销售组及成员]: #create-group
[为集成的应用程序生成密钥]: #generate-key
[添加一个共享的脚本来检查成员资格]: #add-shared-script
[向数据库操作中添加基于角色的访问检查]: #add-access-checking
[测试客户端访问]: #test-client


<!-- Images -->
[0]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/users.png
[1]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/group-membership.png
[2]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/sales-group.png
[3]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/table-perms.png
[4]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/insert-table-op-view.png
[5]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/sales-group-id.png
[6]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/client-id-and-key.png

<!-- URLs. -->
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/
[如何向 Azure Active Directory 注册]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Azure 管理门户]: https://manage.windowsazure.cn/
[目录同步方案]: http://msdn.microsoft.com/library/azure/jj573653.aspx
[存储服务器脚本]: /zh-cn/documentation/articles/mobile-services-store-scripts-source-control/
[注册以使用 Azure Active Directory Login]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Graph API]: http://msdn.microsoft.com/library/azure/hh974478.aspx
[Graph API 错误参考]: http://msdn.microsoft.com/library/azure/hh974480.aspx
[IsMemberOf]: http://msdn.microsoft.com/library/azure/dn151601.aspx
