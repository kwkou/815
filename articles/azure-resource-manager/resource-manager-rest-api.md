<properties
    pageTitle="Resource Manager REST API | Azure"
    description="概述 Resource Manager REST API 身份验证和用例"
    services="azure-resource-manager"
    documentationcenter="na"
    author="navalev"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="e8d7a1d2-1e82-4212-8288-8697341408c5"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/13/2017"
    wacn.date="02/10/2017"
    ms.author="navale;tomfitz;" />  


# Resource Manager REST API
> [AZURE.SELECTOR]
- [Azure PowerShell](/documentation/articles/powershell-azure-resource-manager/)
- [Azure CLI](/documentation/articles/xplat-cli-azure-resource-manager/)
- [门户](/documentation/articles/resource-group-portal/)
- [REST API](/documentation/articles/resource-manager-rest-api/)

在每次调用 Azure Resource Manager、每次部署模板以及每次配置存储帐户时，都会对 Azure Resource Manager 的 RESTful API 进行一次或多次调用。本主题专门介绍这些 API 以及如何在完全不使用任何 SDK 的情况下调用它们。如果想要完全控制对 Azure 发出的请求，或者偏好语言的 SDK 不可用或不支持所需的操作，这种方法可能十分有用。

本文不逐一介绍 Azure 中公开的每个 API，而是使用某些操作举例说明如何连接到这些 API。了解基础知识后，可继续阅读 [Azure Resource Manager REST API Reference](https://docs.microsoft.com/rest/api/resources/)（Azure Resource Manager REST API 参考），查找有关如何使用其余 API 的详细信息。

## 身份验证
Resource Manager 的身份验证由 Azure Active Directory (Azure AD) 处理。若要连接到任何 API，首先需要使用 Azure AD 进行身份验证，接收可传递给每个请求的身份验证令牌。由于我们讨论的是如何直接对 REST API 进行单纯调用，因此假设你不想要根据用户名和密码提示进行身份验证。另外，假设你不使用双重身份验证机制。因此，我们将创建用于登录的所谓 Azure AD 应用程序和服务主体。但请记住，Azure AD 支持多个身份验证过程，而这些过程全都可用于检索后续 API 请求所需的身份验证令牌。有关分步说明，请参阅[创建 Azure AD 应用程序和服务主体](/documentation/articles/resource-group-create-service-principal-portal/)。

### <a name="generating-an-access-token"></a> 生成访问令牌
通过调用位于 login.chinacloudapi.cn 的 Azure AD 来对 Azure AD 进行身份验证。若要进行身份验证，需要提供以下信息：

* Azure AD 租户 ID（用于登录的 Azure AD 名称，通常与你的公司同名，但不一定总是如此）
* 应用程序 ID（在创建 Azure AD 应用程序期间获取）
* 密码（在创建 Azure AD 应用程序时选择）

在以下 HTTP 请求中，请确保将“Azure AD Tenant ID”、“Application ID”和“Password”替换为正确的值。

**常规 HTTP 请求：**

    POST /<Azure AD Tenant ID>/oauth2/token?api-version=1.0 HTTP/1.1 HTTP/1.1
    Host: login.chinacloudapi.cn
    Cache-Control: no-cache
    Content-Type: application/x-www-form-urlencoded

    grant_type=client_credentials&resource=https%3A%2F%2Fmanagement.core.chinacloudapi.cn%2F&client_id=<Application ID>&client_secret=<Password>

将（如果身份验证成功）生成类似于下面的响应：

    {
      "token_type": "Bearer",
      "expires_in": "3600",
      "expires_on": "1448199959",
      "not_before": "1448196059",
      "resource": "https://management.core.chinacloudapi.cn/",
      "access_token": "eyJ0eXAiOiJKV1QiLCJhb...86U3JI_0InPUk_lZqWvKiEWsayA"
    }

（为方便阅读，上述响应中的 access\_token 已缩短）

**使用 Bash 生成访问令牌：**

    curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&resource=https://management.core.chinacloudapi.cn&client_id=<application id>&client_secret=<password you selected for authentication>" https://login.chinacloudapi.cn/<Azure AD Tenant ID>/oauth2/token?api-version=1.0

**使用 PowerShell 生成访问令牌：**

    Invoke-RestMethod -Uri https://login.chinacloudapi.cn/<Azure AD Tenant ID>/oauth2/token?api-version=1.0 -Method Post
     -Body @{"grant_type" = "client_credentials"; "resource" = "https://management.core.chinacloudapi.cn/"; "client_id" = "<application id>"; "client_secret" = "<password you selected for authentication>" }

响应包含访问令牌和令牌有效期限的相关信息，以及可以将该令牌用于哪项资源的相关信息。在前面的 HTTP 调用中收到的访问令牌必须针对所有请求传递到 Resource Manager API。请将它作为名为“Authorization”、值为“Bearer YOUR\_ACCESS\_TOKEN”的标头值传递。请注意“Bearer”与访问令牌之间有空格。

从上面的 HTTP 结果可以看到，令牌在特定时间段内保持有效状态，你应在这段期间缓存并重复使用同一令牌。即使你可以在每次调用 API 时对 Azure AD 进行身份验证，但这样做的效率很低。

## 调用 Resource Manager REST API
本主题只使用几个 API 来说明 REST 操作的基本用法。有关所有操作的信息，请参阅 [Azure Resource Manager REST API](https://docs.microsoft.com/rest/api/resources/)。

### 列出所有订阅
可以执行的最简单操作之一是列出可以访问的可用订阅。在以下请求中，可以看到如何以标头的形式传入访问令牌：

（将 YOUR\_ACCESS\_TOKEN 替换为实际访问令牌。）

    GET /subscriptions?api-version=2015-01-01 HTTP/1.1
    Host: management.chinacloudapi.cn
    Authorization: Bearer YOUR_ACCESS_TOKEN
    Content-Type: application/json

...因此，会收到此服务主体可以访问的订阅列表

（为方便阅读，订阅 ID 已缩短）

    {
      "value": [
        {
          "id": "/subscriptions/3a8555...555995",
          "subscriptionId": "3a8555...555995",
          "displayName": "Azure Subscription",
          "state": "Enabled",
          "subscriptionPolicies": {
            "locationPlacementId": "Internal_2014-09-01",
            "quotaId": "Internal_2014-09-01"
          }
        }
      ]
    }

### 列出特定订阅中的所有资源组
适用于 Resource Manager API 的所有资源嵌套在资源组中。可以使用以下 HTTP GET 请求，在 Resource Manager 中查询订阅中现有的资源组。请注意这一次如何将订阅 ID 作为 URL 的一部分传入。

（将 YOUR\_ACCESS\_TOKEN 和 SUBSCRIPTION\_ID 替换为实际的访问令牌和订阅 ID）

    GET /subscriptions/SUBSCRIPTION_ID/resourcegroups?api-version=2015-01-01 HTTP/1.1
    Host: management.chinacloudapi.cn
    Authorization: Bearer YOUR_ACCESS_TOKEN
    Content-Type: application/json

收到的响应取决于是否已定义任何资源组以及定义了多少个（如果已定义）。

（为方便阅读，订阅 ID 已缩短）

    {
        "value": [
            {
                "id": "/subscriptions/3a8555...555995/resourceGroups/myfirstresourcegroup",
                "name": "myfirstresourcegroup",
                "location": "chinaeast",
                "properties": {
                    "provisioningState": "Succeeded"
                }
            },
            {
                "id": "/subscriptions/3a8555...555995/resourceGroups/mysecondresourcegroup",
                "name": "mysecondresourcegroup",
                "location": "chinanorth",
                "tags": {
                    "tagname1": "My first tag"
                },
                "properties": {
                    "provisioningState": "Succeeded"
                }
            }
        ]
    }

### 创建资源组
到目前为止，我们只是通过 Resource Manager API 查询了信息。接下来可以创建一些资源。让我们从最简单的资源组开始。以下 HTTP 请求在所选的区域/位置创建一个资源组并在其中添加一个标记。

（将 YOUR\_ACCESS\_TOKEN、SUBSCRIPTION\_ID、RESOURCE\_GROUP\_NAME 替换为实际的访问令牌、订阅 ID 和要创建的资源组名称）

    PUT /subscriptions/SUBSCRIPTION_ID/resourcegroups/RESOURCE_GROUP_NAME?api-version=2015-01-01 HTTP/1.1
    Host: management.chinacloudapi.cn
    Authorization: Bearer YOUR_ACCESS_TOKEN
    Content-Type: application/json

    {
      "location": "chinanorth",
      "tags": {
        "tagname1": "test-tag"
      }
    }

如果成功，将返回类似于下面的响应：

    {
      "id": "/subscriptions/3a8555...555995/resourceGroups/RESOURCE_GROUP_NAME",
      "name": "RESOURCE_GROUP_NAME",
      "location": "chinanorth",
      "tags": {
        "tagname1": "test-tag"
      },
      "properties": {
        "provisioningState": "Succeeded"
      }
    }

现已在 Azure 中成功创建一个资源组。祝贺你！

### 使用 Resource Manager 模板将资源部署到资源组
在 Resource Manager 中，可以使用模板部署资源。模板定义多个资源及其依赖项。本部分假设读者熟悉 Resource Manager 模板，演示如何进行 API 调用以开始部署。有关构建模板的详细信息，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。

模板的部署与调用其他 API 的方式并没有太大差别。一个重要方面就是模板部署需要相当长的时间。API 调用只会返回，开发人员需负责查询部署状态，了解部署何时完成。有关详细信息，请参阅[跟踪异步 Azure 操作](/documentation/articles/resource-manager-async-operations/)。

本示例使用 [GitHub](https://github.com/Azure/azure-quickstart-templates) 上提供的一个公开模板。使用的模板将 Linux VM 部署到中国北部区域。尽管本示例使用了公共存储库（如 GitHub）中提供的模板，但你也可以传递整个模板作为请求的一部分。请注意，在请求中提供的值是部署的模板中使用的值。

（将 SUBSCRIPTION\_ID、RESOURCE\_GROUP\_NAME、DEPLOYMENT\_NAME、YOUR\_ACCESS\_TOKEN、GLOBALY\_UNIQUE\_STORAGE\_ACCOUNT\_NAME、ADMIN\_USER\_NAME、ADMIN\_PASSWORD 和 DNS\_NAME\_FOR\_PUBLIC\_IP 替换为请求适用的值）

    PUT /subscriptions/SUBSCRIPTION_ID/resourcegroups/RESOURCE_GROUP_NAME/providers/microsoft.resources/deployments/DEPLOYMENT_NAME?api-version=2015-01-01 HTTP/1.1
    Host: management.chinacloudapi.cn
    Authorization: Bearer YOUR_ACCESS_TOKEN
    Content-Type: application/json

    {
      "properties": {
        "templateLink": {
          "uri": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-simple-linux-vm/azuredeploy.json",
          "contentVersion": "1.0.0.0",
        },
        "mode": "Incremental",
        "parameters": {
            "newStorageAccountName": {
              "value": "GLOBALY_UNIQUE_STORAGE_ACCOUNT_NAME"
            },
            "adminUsername": {
              "value": "ADMIN_USER_NAME"
            },
            "adminPassword": {
              "value": "ADMIN_PASSWORD"
            },
            "dnsNameForPublicIP": {
              "value": "DNS_NAME_FOR_PUBLIC_IP"
            },
            "ubuntuOSVersion": {
              "value": "15.04"
            }
        }
      }
    }

为方便阅读本文档，此处省略了此请求的较长 JSON 响应。响应中包含创建的样板化部署的相关信息。

## 后续步骤

- 若要了解如何处理异步 REST 操作，请参阅[跟踪异步 Azure 操作](/documentation/articles/resource-manager-async-operations/)。

<!---HONumber=Mooncake_0206_2017-->
<!-- Update_Description: meta data;wording update;add link reference -->