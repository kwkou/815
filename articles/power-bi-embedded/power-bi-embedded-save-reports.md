<properties
    pageTitle="在 Azure Power BI Embedded 中保存报表 | Microsoft 文档"
    description="了解如何在 Power BI Embedded 中保存报表。 需要有适当的权限才能成功执行此操作。"
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
    wacn.date="04/28/2017"
    ms.author="asaxton"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="f462b6ad638a13c0620351f7584a665d9a476f3b"
    ms.lasthandoff="04/14/2017" />

# <a name="save-reports-in-power-bi-embedded"></a>在 Power BI Embedded 中保存报表

了解如何在 Power BI Embedded 中保存报表。 需要有适当的权限才能成功执行此操作。

在 Power BI Embedded 中，可以编辑和保存现有的报表。 还可以创建一个新报表，然后将它另存为新报表来创建另一个报表。

若要保存报表，首先需要为特定的报表创建具有适当范围的令牌：

- 若要启用“保存”，需要 Report.ReadWrite 范围
- 若要启用“另存为”，需要 Report.Read 和 Workspace.Report.Copy 范围
- 若要启用“保存”和“另存为”，需要 Report.ReadWrite 和 Workspace.Report.Copy 范围

若要在文件菜单中相应地启用“保存”/“另存为”按钮，需要在嵌入报表时，在 Embed 配置中提供适当的权限：

- models.Permissions.ReadWrite
- models.Permissions.Copy
- models.Permissions.All

> [AZURE.NOTE]
> 访问令牌也需要适当的范围。 有关详细信息，请参阅[范围](/documentation/articles/power-bi-embedded-app-token-flow/#scopes/)。

## <a name="embed-report-in-edit-mode"></a>嵌入处于编辑模式的报表

假设你要在应用中嵌入一个处于编辑模式的报表，为此，只需在 Embed 配置中传递适当的属性，然后调用 powerbi.embed()。 需要提供权限和 viewMode 才能在编辑模式下看到“保存”和“另存为”按钮。 有关详细信息，请参阅 [Embed 配置详细信息](https://github.com/Microsoft/PowerBI-JavaScript/wiki/Embed-Configuration-Details)。

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
            embedUrl: 'https://embedded.powerbi.cn/appTokenReportEmbed',
            id:  '5dac7a4a-4452-46b3-99f6-a25915e0fe55',
            permissions: models.Permissions.All /*both save & save as buttons will be visible*/,
            viewMode: models.ViewMode.Edit,
            settings: {
                filterPaneEnabled: true,
                navContentPaneEnabled: true
            }
        };

        // Get a reference to the embedded report HTML element
        var reportContainer = $('#reportContainer')[0];

        // Embed the report and display it within the div container.
        var report = powerbi.embed(reportContainer, config);

现在，将会在应用中嵌入一个处于编辑模式的报表。

## <a name="save-report"></a>保存报表

使用适当的令牌和权限嵌入处于编辑模式的报表后，可以通过文件菜单或 JavaScript 保存该报表：

     // Get a reference to the embedded report.
        report = powerbi.get(reportContainer);

     // Save report
        report.save();

## <a name="save-as"></a>另存为

    // Get a reference to the embedded report.
        report = powerbi.get(reportContainer);
    
        var saveAsParameters = {
            name: "newReport"
        };

        // SaveAs report
        report.saveAs(saveAsParameters);

> [AZURE.IMPORTANT]
> 只有在调用“另存为”之后，才能创建新报表。 保存后，画布仍显示处于编辑模式的旧报表，而不是新报表。 需要嵌入已创建的新报表。 这就需要获取新的访问令牌，因为令牌是根据每个报表创建的。

然后，需要在调用“另存为”后加载新报表。 此操作类似于嵌入任何报表。

    <div id="reportContainer"></div>
  
    var embedConfiguration = {
            accessToken: 'eyJ0eXAiO...Qron7qYpY9MJ',
            embedUrl: 'https://embedded.powerbi.cn/appTokenReportEmbed',
            reportId: '5dac7a4a-4452-46b3-99f6-a25915e0fe54',
        };
    
        // Grab the reference to the div HTML element that will host the report
        var reportContainer = $('#reportContainer')[0];

        // Embed report
        var report = powerbi.embed(reportContainer, embedConfiguration);

## <a name="see-also"></a>另请参阅

- [示例入门](/documentation/articles/power-bi-embedded-get-started-sample/)  
- [嵌入报表](/documentation/articles/power-bi-embedded-embed-report/)  
- [从数据集创建新报表](/documentation/articles/power-bi-embedded-create-report-from-dataset/)  
- [在 Power BI Embedded 中进行身份验证和授权](/documentation/articles/power-bi-embedded-app-token-flow/)  
- [Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/)  
- [JavaScript 嵌入示例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
- 有更多问题？ [试用 Power BI 社区](http://community.powerbi.com/)

