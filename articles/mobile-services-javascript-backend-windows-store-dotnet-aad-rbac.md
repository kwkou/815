<properties 
	pageTitle="使用 JavaScript 的移动服务和 Azure Active Directory 中基于角色的访问控制（Windows 应用商店）| Windows Azure" 
	description="了解如何使用 JavaScript 后端通过移动服务基于 Windows 应用商店应用程序中的 Azure Active Directory 角色控制访问。" 
	documentationCenter="windows" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="09/03/2015" 
	wacn.date="10/22/2015"/>

# 使用 .NET 的移动服务和 Azure Active Directory 中基于角色的访问控制

[AZURE.INCLUDE [mobile-services-selector-rbac](../includes/mobile-services-selector-rbac.md)]

##概述

基于角色的访问控制 (RBAC) 是对用户可充当的角色分配权限的做法，明确定义特定类别的用户可以或者不可以执行哪些操作。本教程将指导你了解如何将基本 RBAC 添加到 Azure 移动服务。

本教程将演示基于角色的访问控制，检查每个用户在 Azure Active Directory (AAD) 中定义的“销售”组的成员资格。访问检查将使用 Azure Active Directory 的 [Graph API]，通过移动服务后端中的 JavaScript 完成。只有属于销售角色的用户才可以查询数据。


>[AZURE.NOTE]本教程旨在扩充身份验证知识以加入授权实践。你应该先使用 Azure Active Directory 身份验证提供程序完成[向移动服务应用添加身份验证]教程。本教程将继续更新[向移动服务应用添加身份验证]教程中使用的 TodoItem 应用程序。

##先决条件

本教程需要的内容如下：

* 在 Windows 8.1 上运行的 Visual Studio 2013。
* 使用 Azure Active Directory 身份验证提供程序完成[向移动服务应用添加身份验证]教程。
* 完成[存储服务器脚本]教程，以熟悉如何使用 Git 存储库来存储服务器脚本。
 


##创建具有成员资格的销售组

[AZURE.INCLUDE [mobile-services-aad-rbac-create-sales-group](../includes/mobile-services-aad-rbac-create-sales-group.md)]


##为集成的应用程序生成密钥


在学习[向移动服务应用添加身份验证]教程的过程中，你在完成[注册以使用 Azure Active Directory 登录名]步骤时为集成的应用程序创建了注册。在本部分中，你将生成在使用该集成应用程序客户端 ID 读取目录信息时所用的密钥。

如果你已完成了[访问 Azure Active Directory Graph 信息]教程，则表示你已完成此步骤，因此可跳过本部分。

[AZURE.INCLUDE [mobile-services-generate-aad-app-registration-access-key](../includes/mobile-services-generate-aad-app-registration-access-key.md)]





##将共享的脚本添加到检查成员资格的移动服务

在本部分中，你将使用 Git 将名为 *rbac.js* 的共享脚本文件部署到你的移动服务。此共享脚本文件将包含使用 [Graph API] 检查用户的组成员身份的函数。

如果你不熟悉如何使用 Git 将脚本部署到移动服务，请在完成本部分之前，认真阅读[存储服务器脚本]教程。

1. 在移动服务的本地存储库的 *./service/shared/* 目录中，创建一个名为 *rbac.js* 的新脚本文件。
2. 将以下脚本添加到定义 `getAADToken` 函数的文件顶部。根据给定的 *tenant\_domain*、集成应用程序*客户端 ID* 和应用程序*密钥*，此函数提供了一个用于读取目录信息的 Graph 访问令牌。

    >[AZURE.NOTE]你应该缓存令牌，而不是每次执行访问权限检查时都创建一个新令牌。然后，在尝试使用 401 Authentication\_ExpiredToken 响应中的令牌结果时刷新缓存，如 [Graph API 错误参考]所述。为简单起见，以下代码中并未演示此操作，但这可以减轻 Active Directory 的额外网络流量。

        var appSettings = require('mobileservice-config').appSettings;
        var tenant_domain = appSettings.AAD_TENANT_DOMAIN;
        var clientID = appSettings.AAD_CLIENT_ID;
        var key = appSettings.AAD_CLIENT_KEY;
        exports.SalesGroup = appSettings.AAD_GROUP_ID;
         
        function getAADToken(callback) {
            var req = require("request");
            var options = {
                url: "https://login.chinacloudapi.cn/" + tenant_domain + "/oauth2/token?api-version=1.0",
                method: 'POST',
                form: {
                    grant_type: "client_credentials",
                    resource: "https://graph.chinacloudapi.cn",
                    client_id: clientID,
                    client_secret: key
                }
            };
            req(options, function (err, resp, body) {
                if (err || resp.statusCode !== 200) callback(err, null);
                else callback(null, JSON.parse(body).access_token);
                });
        }


3. 向 *rbac.js* 添加以下代码来定义 `getGroupId` 函数。此函数根据筛选器中所用的组名称，使用访问令牌来获取组 id。
 
    >[AZURE.NOTE]此代码将按名称查找 Active Directory 组。在许多情况下，将组 ID 存储为移动服务应用设置并只使用该组 ID 是较好的做法。这是因为组名称可能会更改，而 ID 会保持相同。但是，当组名称更改时，角色的作用域中通常至少有一项更改可能也需要移动服务代码的更新。

        function getGroupId(groupname, accessToken, callback) {
            var req = require("request");
            var options = {
                url: "https://graph.chinacloudapi.cn/" + tenant_domain + "/groups" + 
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


4. 将以下代码添加到 *rbac.js*，以定义调用 Graph REST API 的 [IsMemberOf] 终结点的 `isMemberOf` 函数。

    此函数是围绕 Graph REST API 的 [IsMemberOf] 终结点的瘦包装器。它使用 Graph 访问令牌，根据组 id 来检查用户的目录对象 id 是否在组中具有成员身份。

        function isMemberOf(access_token, userObjectId, groupObjectId, callback) {
            var req = require("request");
            var options = {
                url: "https://graph.chinacloudapi.cn/" + tenant_domain + "/isMemberOf" + "?api-version=2013-04-05",
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

    

5. 将以下导出的 `checkGroupMembership` 函数添加到 *rbac.js*。

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

6. 保存对 *rbac.js* 所做的更改。

##将基于角色的访问检查添加到数据库操作


当你完成[向移动服务应用添加身份验证]教程后，你应该已经将表操作设置为要求身份验证，如下所示。

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

##测试客户端的访问

[AZURE.INCLUDE [mobile-services-aad-rbac-test-app](../includes/mobile-services-aad-rbac-test-app.md)]







<!-- Images -->
[0]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/users.png
[1]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/group-membership.png
[2]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/sales-group.png
[3]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/table-perms.png
[4]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/insert-table-op-view.png
[5]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/sales-group-id.png
[6]: ./media/mobile-services-javascript-backend-windows-store-dotnet-aad-rbac/client-id-and-key.png

<!-- URLs. -->
[向移动服务应用添加身份验证]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/
[How to Register with the Azure Active Directory]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Azure 管理门户]: https://manage.windowsazure.cn/
[Directory Sync Scenarios]: http://msdn.microsoft.com/zh-cn/library/azure/jj573653.aspx
[存储服务器脚本]: /zh-cn/documentation/articles/mobile-services-store-scripts-source-control/
[注册以使用 Azure Active Directory 登录名]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Graph API]: http://msdn.microsoft.com/zh-cn/library/azure/hh974478.aspx
[Graph API 错误参考]: http://msdn.microsoft.com/zh-cn/library/azure/hh974480.aspx
[IsMemberOf]: http://msdn.microsoft.com/zh-cn/library/azure/dn151601.aspx
[访问 Azure Active Directory Graph 信息]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-aad-graph-info

<!---HONumber=74-->