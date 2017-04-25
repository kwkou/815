<properties
    pageTitle="嵌入 Azure Power BI Embedded 中的报表 | Microsoft 文档"
    description="了解如何将 Power BI Embedded 中的报表嵌入应用程序。"
    services="power-bi-embedded"
    documentationcenter=""
    author="guyinacube"
    manager="erikre"
    editor=""
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="power-bi-embedded"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="powerbi"
    ms.date="03/11/2017"
    wacn.date="04/24/2017"
    ms.author="asaxton"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="eb937b587cbca86289f82debe8950b59a2e29539"
    ms.lasthandoff="04/14/2017" />

# <a name="embed-a-report-in-power-bi-embedded"></a>嵌入 Power BI Embedded 中的报表

了解如何将 Power BI Embedded 中的报表嵌入应用程序。

我们将会探讨如何真正在应用程序中嵌入报表。 本文假设工作区集合中的工作区内已有一个报表。 如果尚未完成该步骤，请参阅 [Power BI Embedded 入门](/documentation/articles/power-bi-embedded-get-started/)。

可以配合 JavaScript 使用 .NET (C#) 或 Node.js SDK，在 Power BI Embedded 中轻松构建应用程序。 

## <a name="using-the-access-keys-to-use-rest-apis"></a>借助访问密钥使用 REST API

若要调用 REST API，可以传递访问密钥（可从给定工作区集合的 Azure 门户预览中获取）。 有关详细信息，请参阅 [Power BI Embedded 入门](/documentation/articles/power-bi-embedded-get-started/)。

## <a name="get-a-report-id"></a>获取报表 ID

每个访问令牌基于某个报表。 需要获取想要嵌入的报表的给定报表 ID。 可以通过调用“获取报表”REST API 来实现此目的。[](https://msdn.microsoft.com/zh-cn/library/azure/mt711510.aspx) 此操作将返回报表 ID 和嵌入 URL。 可以使用 Power BI .NET SDK 或通过直接调用 REST API 来实现此目的。

### <a name="using-the-power-bi-net-sdk"></a>使用 Power BI .NET SDK

使用 .NET SDK 时，需要创建基于从 Azure 门户预览中获取的访问密钥的令牌凭据。 这就需要安装 [Power BI API NuGet 包](https://www.nuget.org/profiles/powerbi)。

**安装 NuGet 包**

    Install-Package Microsoft.PowerBI.Api

**C# 代码**

    using Microsoft.PowerBI.Api.V1;
    using Microsoft.Rest;

    var credentials = new TokenCredentials("{access key}", "AppKey");
    var client = new PowerBIClient(credentials);
    client.BaseUri = new Uri(https://api.powerbi.cn);

    var reports = (IList<Report>)client.Reports.GetReports(workspaceCollectionName, workspaceId).Value;

    // Select the report that you want to work with from the collection of reports.

### <a name="calling-the-rest-api-directly"></a>直接调用 REST API

    System.Net.WebRequest request = System.Net.WebRequest.Create("https://api.powerbi.cn/v1.0/collections/{collectionName}/workspaces/{workspaceId}/Reports") as System.Net.HttpWebRequest;

    request.Method = "GET";
    request.ContentLength = 0;
    request.Headers.Add("Authorization", String.Format("AppKey {0}", accessToken.Value));

    using (var response = request.GetResponse() as System.Net.HttpWebResponse)
    {
        using (var reader = new System.IO.StreamReader(response.GetResponseStream()))
        {
            // Work with JSON response to get the report you want to work with.
        }

    }

## <a name="create-an-access-token"></a>创建访问令牌

Power BI Embedded 使用嵌入令牌，即经过 HMAC 签名的 JSON Web 令牌。 令牌已使用 Azure Power BI Embedded 工作区集合中的访问密钥签名。 默认情况下，嵌入令牌用于提供对要嵌入应用程序的报表的只读访问权限。 嵌入令牌是针对特定报表颁发的，应该与嵌入 URL 相关联。

应在服务器上创建访问令牌，因为要使用访问密钥对令牌进行签名/加密。 有关如何创建访问令牌的信息，请参阅[通过 Power BI Embedded 进行身份验证和授权](/documentation/articles/power-bi-embedded-app-token-flow/)。 此外，还可以查看 [CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_) 方法。 以下示例演示了如何使用用于 Power BI 的 .NET SDK。

将要使用前面检索的报表 ID。 创建嵌入令牌后，将使用访问密钥来生成可在 JavaScript 界面中使用的令牌。 *PowerBIToken 类*要求安装 [Power BI Core NuGut 包](https://www.nuget.org/packages/Microsoft.PowerBI.Core/)。

**安装 NuGet 包**

    Install-Package Microsoft.PowerBI.Core

**C# 代码**

    using Microsoft.PowerBI.Security;

    // rlsUsername, roles and scopes are optional.
    embedToken = PowerBIToken.CreateReportEmbedToken(workspaceCollectionName, workspaceId, reportId, rlsUsername, roles, scopes);

    var token = embedToken.Generate("{access key}");

### <a name="adding-permission-scopes-to-embed-tokens"></a>将权限范围添加到嵌入令牌

使用嵌入令牌时，可能需要限制你授权访问的资源的使用。 为此，可以生成一个权限范围受到约束的令牌。 有关详细信息，请参阅[范围](/documentation/articles/power-bi-embedded-app-token-flow/#scopes/)

## <a name="embed-using-javascript"></a>使用 JavaScript 嵌入

获取访问令牌和报表 ID 后，我们使用 JavaScript 来嵌入报表。 这就需要安装 [Power BI JavaScript NuGet 包](https://www.nuget.org/packages/Microsoft.PowerBI.JavaScript/)。 https://embedded.powerbi.com/appTokenReportEmbed 即为 embedUrl。

> [AZURE.NOTE]
> 可以使用 [JavaScript 报表嵌入示例](https://microsoft.github.io/PowerBI-JavaScript/demo/)测试功能。 我们还提供了适用于不同操作的代码示例。

**安装 NuGet 包**

    Install-Package Microsoft.PowerBI.JavaScript

**JavaScript 代码**

    <script src="/scripts/powerbi.js"></script>
    <div id="reportContainer"></div>

    var embedConfiguration = {
        type: 'report',
        accessToken: 'eyJ0eXAiO...Qron7qYpY9MI',
        id: '5dac7a4a-4452-46b3-99f6-a25915e0fe55',
        embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed'
    };

    var $reportContainer = $('#reportContainer');
    var report = powerbi.embed($reportContainer.get(0), embedConfiguration);

### <a name="set-the-size-of-embedded-elements"></a>设置嵌入元素的大小

报表将根据其容器大小自动嵌入。 若要重写默认的嵌入大小，只需添加 CSS 类属性，或者宽度和高度的内联样式。

## <a name="see-also"></a>另请参阅

[示例入门](/documentation/articles/power-bi-embedded-get-started-sample/)  
[在 Power BI Embedded 中进行身份验证和授权](/documentation/articles/power-bi-embedded-app-token-flow/)  
[CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_)  
[JavaScript 嵌入示例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
[Power BI JavaScript 包](https://www.nuget.org/packages/Microsoft.PowerBI.JavaScript/)  
[Power BI API NuGet 包](https://www.nuget.org/profiles/powerbi)
[Power BI Core NuGut 包](https://www.nuget.org/packages/Microsoft.PowerBI.Core/)  
[PowerBI-CSharp Git 存储库](https://github.com/Microsoft/PowerBI-CSharp)  
[PowerBI-Node Git 存储库](https://github.com/Microsoft/PowerBI-Node)  
有更多问题？ [试用 Power BI 社区](http://community.powerbi.com/)

