<properties
    pageTitle="示例入门"
    description="Power BI Embedded，使用 SDK 将交互式 Power BI 报表添加到商业智能应用程序"
    services="power-bi-embedded"
    documentationcenter=""
    author="guyinacube"
    manager="erikre"
    editor=""
    tags=""
    translationtype="Human Translation" />

<tags
    ms.assetid="d8a9ef78-ad4e-4bc7-9711-89172dc5c548"
    ms.service="power-bi-embedded"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="powerbi"
    ms.date="03/02/2017"
    wacn.date="04/24/2017"
    ms.author="asaxton"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="5b1c68750589590134d83022e731005e1140df8d"
    ms.lasthandoff="04/14/2017" />

# <a name="get-started-with-power-bi-embedded-sample"></a>Power BI Embedded 示例入门

通过 **Power BI Embedded**，可以将 Power BI 报表集成到 web 或移动应用程序。 本文介绍了 **Power BI Embedded** 入门示例。

在继续之前，可能需要保存以下资源。 在将 Power BI 报表集成到示例应用和自己的应用中时，这些资源都可以提供帮助。

- [Web 应用示例仪表板](https://github.com/kustbilla/Mooncake_PowerBI_Embedded)
- [Power BI Embedded API 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt711507.aspx)
- [Power BI Embedded .NET SDK ](http://go.microsoft.com/fwlink/?LinkId=746472)（通过 NuGet 提供）
- [JavaScript 报表嵌入示例](https://microsoft.github.io/PowerBI-JavaScript/demo)

> [AZURE.NOTE] 
> 需要先在 Azure 订阅中创建至少一个 **工作区集合** 才能配置和运行 Power BI Embedded 入门示例。 若要了解如何在 Azure 门户中创建**工作区集合**，请参阅 [Power BI Embedded 入门](/documentation/articles/power-bi-embedded-get-started/)。

## <a name="configure-the-sample-app"></a>配置示例应用

下面将引导完成 Visual Studio 开发环境的设置，以便访问运行示例应用时所需的组件。

1. 下载并解压缩 GitHub 上的 [Power BI Embedded - Integrate a report into a web app](http://go.microsoft.com/fwlink/?LinkId=761493) （Power BI Embedded - 将报表集成到 Web 应用中）示例。

	>[AZURE.IMPORTANT] 示例中所使用的一些URL只适用于 Global 环境，用于 Azure China 需要做相应的替换，
	> 如：\ProvisionSample\App.config 中的 https://api.powerbi.com 需要替换成 https://api.powerbi.cn,
	> https://management.azure.com 需要替换成 https://management.chinacloudapi.cn，
	> \ProvisionSample\Program.cs 中的 https://management.core.windows.net 需要替换成 https://management.core.chinacloudapi.cn，
	> \ProvisionSample\ProgramExtensions.cs 中的 https://login.windows.net 需要替换成 https://login.chinacloudapi.cn.
	> 具体详细的关于URL使用方面的区别请看[中国区 Azure 应用程序开发说明](/documentation/articles/developerdifferences)。
	> 

2. 在 Visual Studio 中打开“PowerBI embedded.sln”  。 可能需要在 NuGET 程序包管理器控制台中执行“Update-Package”  命令来更新此解决方案中使用的程序包。
3. 生成解决方案。
4. 运行“ProvisionSample”  控制台应用。 在示例控制台应用中，预配一个工作区并导入 PBIX 文件。
5. 要预配新的**工作区**，请选择选项 5“在现有工作区集合中预配新的工作区”。

    ![](./media/powerbi-embedded-get-started-sample/console-option-5.png)
6. 输入**工作区集合**名称和**访问密钥**。 可以通过 **Azure 门户**获取这些信息。 若要详细了解如何获取**访问密钥**，请参阅“Power BI Embedded 入门”中的[查看 Power BI API 访问密钥](/documentation/articles/power-bi-embedded-get-started/#view-power-bi-api-access-keys/)。

    ![](./media/powerbi-embedded-get-started-sample/azure-portal.png)
7. 复制并保存新创建的 **工作区 ID** 以便在本文后面部分使用。 创建**工作区 ID** 之后，可以在 **Azure 门户**中找到该数据。

    ![](./media/powerbi-embedded-get-started-sample/workspace-id.png)
8. 若要将 PBIX 文件导入到**工作区**，请选择选项 **6**, “将 PBIX 文件导入到现有工作区”。 如果没有现有的 PBIX 文件，则可以下载 [Retail Analysis Sample PBIX](http://go.microsoft.com/fwlink/?LinkID=780547)（零售分析示例 PBIX）。
9. 如果出现提示，请为**数据集**输入一个易记的名称。

应该会看到如下所示的响应：

    Checking import state... Publishing
    Checking import state... Succeeded

> [AZURE.NOTE]
> 如果 PBIX 文件包含任何直接查询连接，请选择选项 7 以更新连接字符串。

此时， **工作区**中已导入了一个 Power BI PBIX 报表。 接下来将演示如何运行 **Power BI Embedded** 入门示例 Web 应用。

## <a name="run-the-sample-web-app"></a>运行示例 Web 应用
Web 应用示例是一个示例应用程序，用于呈现**工作区**中导入的报表。 下面介绍了如何配置 Web 应用示例。

1. 在 **PowerBI Embedded** Visual Studio 解决方案中，右键单击“EmbedSample”Web 应用程序，然后选择“设为启动项目”。
2. 在 **web.config** 的 **EmbedSample** Web 应用程序中，编辑 **appSettings** 中的 **AccessKey** 和 **WorkspaceCollection** 名称，以及 **WorkspaceId**。

        <appSettings>
            <add key="powerbi:AccessKey" value="" />
            <add key="powerbi:ApiUrl" value="https://api.powerbi.cn" />
            <add key="powerbi:WorkspaceCollection" value="" />
            <add key="powerbi:WorkspaceId" value="" />
        </appSettings>

3. 运行 **EmbedSample** Web 应用程序。

运行 **EmbedSample** Web 应用程序后，左侧的导航面板应包含“报表”菜单。 若要查看导入的报表，请展开“报表”，然后单击任一报表。 如果已导入了 [Retail Analysis Sample PBIX](http://go.microsoft.com/fwlink/?LinkID=780547)（零售分析示例 PBIX），则示例 Web 应用将如下所示：

![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-sample-left-nav.png)

单击某个报表后， **EmbedSample** Web 应用程序应类似于以下内容：

![](./media/powerbi-embedded-get-started-sample/sample-web-app.png)

## <a name="explore-the-sample-code"></a>探索示例代码

**Power BI Embedded** 示例是一个 Web 应用示例，演示了如何将 **Power BI** 报表集成到应用中。 它采用模型-视图-控制器 (MVC) 设计模式来演示最佳做法。 本部分重点介绍可以在 **PowerBI Embedded** Web 应用解决方案中浏览的示例代码。 模型-视图-控制器 (MVC) 模式根据用户输入的三个单独的类（模型、视图和控件）对域、演示文稿和操作分开进行建模。 若要了解关于 MVC 的详细信息，请参阅 [Learn About ASP.NET](http://www.asp.net/mvc)（了解 ASP.NET）。

**Power BI Embedded** 示例代码分隔方式如下所示。 每个部分在 PowerBI embedded.sln 解决方案中都包括了文件名称，以便轻松查找示例中的代码。

> [AZURE.NOTE]
> 本节总结了演示如何编写代码的示例代码。 若要查看完整的示例，请加载 Visual Studio 中的 PowerBI embedded.sln 解决方案。

### <a name="model"></a>模型

该示例包含 **ReportsViewModel** 和 **ReportViewModel**。

**ReportsViewModel.cs**：表示多个 Power BI 报表。

    public class ReportsViewModel
    {
        public List<Report> Reports { get; set; }
    }

**ReportViewModel.cs**：表示一个 Power BI 报表。

    public classReportViewModel
    {
        public IReport Report { get; set; }

        public string AccessToken { get; set; }
    }

### <a name="connection-string"></a>连接字符串

连接字符串必须采用以下格式：

    Data Source=tcp:MyServer.database.chinacloudapi.cn,1433;Initial Catalog=MyDatabase

使用一般的服务器和数据库属性将会失败。 例如：Server=tcp:MyServer.database.chinacloudapi.cn,1433;Database=MyDatabase。

### <a name="view"></a>查看

**视图**用于管理多个 Power BI **报表**或一个 Power BI **报表**的显示情况。

**Reports.cshtml**：循环访问 **Model.Reports** 以创建 **ActionLink**。 **ActionLink** 包含以下内容：

| 部分 | 说明 |
| --- | --- |
| 标题 |报表的名称。 |
| QueryString |指向报表 ID 的链接。 |

    <div id="reports-nav" class="panel-collapse collapse">
        <div class="panel-body">
            <ul class="nav navbar-nav">
                @foreach (var report in Model.Reports)
                {
                    var reportClass = Request.QueryString["reportId"] == report.Id ? "active" : "";
                    <li class="@reportClass">
                        @Html.ActionLink(report.Name, "Report", new { reportId = report.Id })
                    </li>
                }
            </ul>
        </div>
    </div>

Report.cshtml：为 **PowerBIReportFor** 设置 **Model.AccessToken** 和 Lambda 表达式。

    @model ReportViewModel

    ...

    <div class="side-body padding-top">
        @Html.PowerBIAccessToken(Model.AccessToken)
        @Html.PowerBIReportFor(m => m.Report, new { style = "height:85vh" })
    </div>

### <a name="controller"></a>控制器

**DashboardController.cs**：创建用于传递**应用令牌** 的 PowerBIClient。 JSON Web 令牌 (JWT) 是基于**签名密钥**生成的，用于获取**凭据**。 **凭据**用于创建 **PowerBIClient** 实例。 创建 **PowerBIClient** 实例后，可以调用 GetReports() 和 GetReportsAsync()。

    CreatePowerBIClient()

        private IPowerBIClient CreatePowerBIClient()
        {
            var credentials = new TokenCredentials(accessKey, "AppKey");
            var client = new PowerBIClient(credentials)
            {
                BaseUri = new Uri(apiUrl)
            };

            return client;
        }

    ActionResult Reports()

        public ActionResult Reports()
        {
            using (var client = this.CreatePowerBIClient())
            {
                var reportsResponse = client.Reports.GetReports(this.workspaceCollection, this.workspaceId);

                var viewModel = new ReportsViewModel
                {
                    Reports = reportsResponse.Value.ToList()
                };

                return PartialView(viewModel);
            }
        }

    Task<ActionResult> Report(string reportId)

        public async Task<ActionResult> Report(string reportId)
        {
            using (var client = this.CreatePowerBIClient())
            {
                var reportsResponse = await client.Reports.GetReportsAsync(this.workspaceCollection, this.workspaceId);
                var report = reportsResponse.Value.FirstOrDefault(r => r.Id == reportId);
                var embedToken = PowerBIToken.CreateReportEmbedToken(this.workspaceCollection, this.workspaceId, report.Id);

                var viewModel = new ReportViewModel
                {
                    Report = report,
                    AccessToken = embedToken.Generate(this.accessKey)
                };

                return View(viewModel);
            }
        }

### <a name="integrate-a-report-into-your-app"></a>将报表集成到应用中

创建**报表**后，可以使用 **IFrame** 嵌入 Power BI **报表**。 以下是 **Power BI Embedded** 示例的 powerbi.js 中的代码片段。

![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-iframe-code.png)

## <a name="filter-reports-embedded-in-your-application"></a>筛选应用程序中嵌入的报表

可以使用 URL 语法筛选嵌入的报表。 要进行筛选，可以使用指定的筛选器将带运算符 **eq** 的 **$filter** 查询字符串参数添加到 iFrame src url。 以下为筛选查询语法：

    https://app.powerbi.com/reportEmbed
    ?reportId=d2a0ea38-...-9673-ee9655d54a4a&
    $filter={tableName/fieldName}%20eq%20'{fieldValue}'

> [AZURE.NOTE]
> {tableName/fieldName} 不能包含空格或特殊字符。 {fieldValue} 接受单个分类值。  

## <a name="see-also"></a>另请参阅

- [常见 Power BI Embedded 方案](/documentation/articles/power-bi-embedded-scenarios/)  
- [在 Power BI Embedded 中进行身份验证和授权](/documentation/articles/power-bi-embedded-app-token-flow/)  
- [嵌入报表](/documentation/articles/power-bi-embedded-embed-report/)  
- [从数据集创建新报表](/documentation/articles/power-bi-embedded-create-report-from-dataset/)  
- [Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/)  
- [JavaScript 嵌入示例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
- 有更多问题？ [试用 Power BI 社区](http://community.powerbi.com/)

<!--Update_Description: wording update-->