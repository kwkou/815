<properties
    pageTitle="通过 Power BI Embedded 进行身份验证和授权"
    description="通过 Power BI Embedded 进行身份验证和授权"
    services="power-bi-embedded"
    documentationcenter=""
    author="guyinacube"
    manager="erikre"
    editor=""
    tags=""
    translationtype="Human Translation" />
    
<tags
    ms.assetid="1c1369ea-7dfd-4b6e-978b-8f78908fd6f6"
    ms.service="power-bi-embedded"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="powerbi"
    ms.date="03/11/2017"
    wacn.date="04/24/2017"
    ms.author="asaxton"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="ad674abf097861c81d04ee56f19021cb6e788911"
    ms.lasthandoff="04/14/2017" />

# <a name="authenticating-and-authorizing-with-power-bi-embedded"></a>通过 Power BI Embedded 进行身份验证和授权

Power BI Embedded 服务使用**密钥**和**应用令牌**进行身份验证和授权，而不是使用显式的最终用户身份验证。 在此模型中，应用程序管理最终用户的身份验证和授权。 如有必要，应用将创建并发送应用令牌，以指示服务来呈现所请求的报表。 此设计不要求应用使用 Azure Active Directory 进行用户身份验证和授权，但仍然可以这样做。

## <a name="two-ways-to-authenticate"></a>进行身份验证的两种方式

**密钥** - 对于所有 Power BI Embedded REST API 调用，可以使用密钥。 在 **Azure 门户预览**中，可以通过依次单击“所有设置”和“访问密钥”来找到密钥。 请始终像对待密码一样对待密钥。 这些密钥有权在特定的工作区集合上执行任何 REST API 调用。

若要在 REST 调用中使用密钥，请添加以下授权标头：            

    Authorization: AppKey {your key}

**应用令牌** - 应用令牌用于所有嵌入请求。 它们被设计为运行客户端，因此被限制到单个报表，并且最好设置一个过期时间。

应用令牌是由某个密钥签名的 JWT（JSON Web 令牌）。

应用令牌可以包含下列声明：

| 声明 | 说明 |
| --- | --- |
| **ver** |应用令牌的版本。 当前版本为 0.2.0。 |
| **aud** |令牌的目标接收方。 对于 Power BI Embedded，请使用：“https://analysis.chinacloudapi.cn/powerbi/api”。 |
| **iss** |一个字符串，指示颁发了令牌的应用程序。 |
| **type** |要创建的应用令牌的类型。 当前唯一支持的类型是 **embed**。 |
| **wcn** |要为其颁发令牌的工作区集合名称。 |
| **wid** |要为其颁发令牌的工作区 ID。 |
| **rid** |要为其颁发令牌的报表 ID。 |
| **username**（可选） |与 RLS 一起使用，这是一个字符串，可以在应用 RLS 规则时帮助标识用户。 |
| **roles**（可选） |一个字符串，包含当应用行级别安全性规则时可选择的角色。 如果传递多个角色，则应当以字符串数组形式传递它们。 |
| **scp**（可选） |一个字符串，包含权限范围。 如果传递多个角色，则应当以字符串数组形式传递它们。 |
| **exp**（可选） |指示令牌将过期的时间。 它们应当作为 Unix 时间戳传入。 |
| **nbf**（可选） |指示令牌开始生效的时间。 它们应当作为 Unix 时间戳传入。 |

示例应用令牌将如下所示：

    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ2ZXIiOiIwLjIuMCIsInR5cGUiOiJlbWJlZCIsIndjbiI6Ikd1eUluQUN1YmUiLCJ3aWQiOiJkNGZlMWViMS0yNzEwLTRhNDctODQ3Yy0xNzZhOTU0NWRhZDgiLCJyaWQiOiIyNWMwZDQwYi1kZTY1LTQxZDItOTMyYy0wZjE2ODc2ZTNiOWQiLCJzY3AiOiJSZXBvcnQuUmVhZCIsImlzcyI6IlBvd2VyQklTREsiLCJhdWQiOiJodHRwczovL2FuYWx5c2lzLndpbmRvd3MubmV0L3Bvd2VyYmkvYXBpIiwiZXhwIjoxNDg4NTAyNDM2LCJuYmYiOjE0ODg0OTg4MzZ9.v1znUaXMrD1AdMz6YjywhJQGY7MWjdCR3SmUSwWwIiI

当解码后，它将如下所示：

    Header

    {
        typ: "JWT",
        alg: "HS256:
    }

    Body

    {
      "ver": "0.2.0",
      "wcn": "SupportDemo",
      "wid": "ca675b19-6c3c-4003-8808-1c7ddc6bd809",
      "rid": "96241f0f-abae-4ea9-a065-93b428eddb17",
      "iss": "PowerBISDK",
      "aud": "https://analysis.chinacloudapi.cn/powerbi/api",
      "exp": 1360047056,
      "nbf": 1360043456
    }


SDK 中提供了可以更轻松地创建应用令牌的方法。 例如，对于 .NET，可以查看 [Microsoft.PowerBI.Security.PowerBIToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken) 类和 [CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_) 方法。

对于 .NET SDK，可以参考 [Scopes](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.scopes)。

## <a name="scopes"></a>作用域

使用 Embed 令牌时，对于你允许对其进行访问的资源，你可能希望限制其使用。 为此，你可以生成一个包含带作用域的权限的令牌。

下面是 Power BI Embedded 的可用作用域。

|作用域|说明|
|---|---|
|Dataset.Read|提供对指定数据集进行读取的权限。|
|Dataset.Write|提供向指定数据集进行写入的权限。|
|Dataset.ReadWrite|提供对指定数据集进行读取以及向其进行写入的权限。|
|Report.Read|提供对指定报表进行查看的权限。|
|Report.ReadWrite|提供对指定报表进行查看和编辑的权限。|
|Workspace.Report.Create|提供在指定工作区中创建新报表的权限。|
|Workspace.Report.Copy|提供对指定工作区内的现有报表进行克隆的权限。|

你可以通过在作用域之间使用空格来提供多个作用域，如下所示。

    string scopes = "Dataset.Read Workspace.Report.Create";

**必需的声明 - 作用域**

scp: {scopesClaim} scopesClaim 可以是一个字符串或字符串数组，指明允许对工作区资源（报表、数据集，等等）具有的权限。

已定义了作用域的已解码令牌将类似于以下内容。

    Header

    {
        typ: "JWT",
        alg: "HS256:
    }

    Body

    {
      "ver": "0.2.0",
      "wcn": "SupportDemo",
      "wid": "ca675b19-6c3c-4003-8808-1c7ddc6bd809",
      "rid": "96241f0f-abae-4ea9-a065-93b428eddb17",
      "scp": "Report.Read",
      "iss": "PowerBISDK",
      "aud": "https://analysis.chinacloudapi.cn/powerbi/api",
      "exp": 1360047056,
      "nbf": 1360043456
    }


### <a name="operations-and-scopes"></a>操作和作用域

|操作|目标资源|令牌权限|
|---|---|---|
|基于数据集创建（在内存中）新报表。|数据集|Dataset.Read|
|基于数据集创建（在内存中）新报表并保存该报表。|数据集|* Dataset.Read<br>* Workspace.Report.Create|
|查看和浏览/编辑（在内存中）现有报表。 Report.Read implies Dataset.Read. 这不会允许保存编辑内容的权限。|报表|Report.Read|
|编辑和保存现有报表。|报表|Report.ReadWrite|
|保存报表的副本（另存为）。|报表|* Report.Read<br>* Workspace.Report.Copy|

## <a name="heres-how-the-flow-works"></a>下面是流的工作原理
1. 将 API 密钥复制到应用程序中。 可以在 **Azure 门户预览**中获取密钥。
   
    ![](./media/powerbi-embedded-get-started-sample/azure-portal.png)
2. 令牌将发布声明，并且有过期时间。
   
    ![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-2.png)
3. 令牌使用 API 访问密钥获得签名。
   
    ![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-3.png)
4. 用户请求查看报表。
   
    ![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-4.png)
5. 使用 API 访问密钥验证令牌。
   
	![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-5.png)
6. Power BI Embedded 向用户发送报表。
   
	![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-6.png)

**Power BI Embedded** 向用户发送报表后，用户可以在自定义应用中查看报表。 例如，如果导入了 [Analyzing Sales Data PBIX sample](http://download.microsoft.com/download/1/4/E/14EDED28-6C58-4055-A65C-23B4DA81C4DE/Analyzing_Sales_Data.pbix)（分析销售数据 PBIX 示例），该示例 Web 应用将如下所示：

![](./media/powerbi-embedded-get-started-sample/sample-web-app.png)

## <a name="see-also"></a>另请参阅

[CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_)  
[Power BI Embedded 示例入门](/documentation/articles/power-bi-embedded-get-started-sample/)  
[常见 Power BI Embedded 方案](/documentation/articles/power-bi-embedded-scenarios/)  
[Power BI Embedded 入门](/documentation/articles/power-bi-embedded-get-started/)  
[PowerBI-CSharp Git 存储库](https://github.com/Microsoft/PowerBI-CSharp)  
有更多问题？ [试用 Power BI 社区](http://community.powerbi.com/)

<!--Update_Description: wording update-->