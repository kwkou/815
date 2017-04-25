<properties
    pageTitle="在 Azure Power BI Embedded 中在报表的查看和编辑模式之间切换 | Microsoft 文档"
    description="了解在 Azure Power BI Embedded 中如何在报表的查看和编辑模式之间切换。"
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
    ms.openlocfilehash="376e780c521871f423245b7e661ca4d0f7ab602c"
    ms.lasthandoff="04/14/2017" />

# <a name="toggle-between-view-and-edit-mode-for-reports-in-power-bi-embedded"></a>在 Azure Power BI Embedded 中在报表的查看和编辑模式之间切换

了解在 Azure Power BI Embedded 中如何在报表的查看和编辑模式之间切换。

## <a name="creating-an-access-token"></a>创建访问令牌

你将需要创建一个访问令牌，以授予你查看和编辑报表的能力。 若要编辑并保存报表，你需要具有 **Report.ReadWrite** 令牌权限。 有关详细信息，请参阅[在 Power BI Embedded 中进行身份验证和授权](/documentation/articles/power-bi-embedded-app-token-flow/)。

> [AZURE.NOTE]
> 这将允许你编辑现有报表并保存更改。 如果还需要支持**另存为**的功能，则需要提供额外的权限。 有关详细信息，请参阅[作用域](/documentation/articles/power-bi-embedded-app-token-flow/#scopes/)。

    using Microsoft.PowerBI.Security;

    // rlsUsername and roles are optional
    string scopes = "Report.ReadWrite";
    PowerBIToken embedToken = PowerBIToken.CreateReportEmbedTokenForCreation(workspaceCollectionName, workspaceId, datasetId, null, null, scopes);

    var token = embedToken.Generate("{access key}");

## <a name="embed-configuration"></a>嵌入配置

为了在编辑模式下看到“保存”按钮，需要提供权限和 viewMode。 有关详细信息，请参阅[嵌入配置详细信息](https://github.com/Microsoft/PowerBI-JavaScript/wiki/Embed-Configuration-Details)。

例如，在 JavaScript 中：

       <div id="reportContainer"></div>

        // Get models. Models, it contains enums that can be used.
        var models = window['powerbi-client'].models;

        // Embed configuration used to describe the what and how to embed.
        // This object is used when calling powerbi.embed.
        // This also includes settings and options such as filters.
        // You can find more information at https://github.com/Microsoft/PowerBI-JavaScript/wiki/Embed-Configuration-Details.
        var config= {
            type: 'report',
            accessToken: 'eyJ0eXAiO...Qron7qYpY9MI',
            embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed',
            id:  '5dac7a4a-4452-46b3-99f6-a25915e0fe55',
            permissions: models.Permissions.ReadWrite /*both save & save as buttons will be visible*/,
            viewMode: models.ViewMode.View,
            settings: {
                filterPaneEnabled: true,
                navContentPaneEnabled: true
            }
        };

        // Get a reference to the embedded report HTML element
        var reportContainer = $('#reportContainer')[0];

        // Embed the report and display it within the div container.
        var report = powerbi.embed(reportContainer, config);

这表示将根据设置为 **models.ViewMode.View** 的 **viewMode** 在查看模式下嵌入报表。

## <a name="view-mode"></a>查看模式

如果处于编辑模式下，可以使用以下 JavaScript 切换到查看模式。

    // Get a reference to the embedded report HTML element
    var reportContainer = $('#reportContainer')[0];

    // Get a reference to the embedded report.
    report = powerbi.get(reportContainer);

    // Switch to view mode.
    report.switchMode("view");


## <a name="edit-mode"></a>编辑模式

如果处于查看模式下，可以使用以下 JavaScript 切换到编辑模式。

    // Get a reference to the embedded report HTML element
    var reportContainer = $('#reportContainer')[0];

    // Get a reference to the embedded report.
    report = powerbi.get(reportContainer);

    // Switch to edit mode.
    report.switchMode("edit");


## <a name="see-also"></a>另请参阅

[示例入门](/documentation/articles/power-bi-embedded-get-started-sample/)  
[嵌入报表](/documentation/articles/power-bi-embedded-embed-report/)  
[在 Power BI Embedded 中进行身份验证和授权](/documentation/articles/power-bi-embedded-app-token-flow/)  
[CreateReportEmbedToken](https://docs.microsoft.com/dotnet/api/microsoft.powerbi.security.powerbitoken?redirectedfrom=MSDN#methods_)  
[JavaScript 嵌入示例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
[PowerBI-CSharp Git 存储库](https://github.com/Microsoft/PowerBI-CSharp)  
[PowerBI-Node Git 存储库](https://github.com/Microsoft/PowerBI-Node)  
有更多问题？ [试用 Power BI 社区](http://community.powerbi.com/)

