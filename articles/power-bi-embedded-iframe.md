<properties
   pageTitle="Microsoft Power BI Embedded 预览版 - 嵌入带有 IFrame 的 Power BI 报表"
   description="Microsoft Power BI Embedded 预览版 - 将报表集成到应用程序中的基本代码，如何使用 Power BI Embedded 应用程序令牌进行身份验证，如何获取报表"
   services="power-bi-embedded"
   documentationCenter=""
   authors="dvana"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="power-bi-embedded"
   ms.date="05/16/2016"
   wacn.date="05/30/2016"/>

# 嵌入带有 IFrame 的 Power BI 报表
本文演示使用 **Power BI Embedded** REST API、应用程序令牌、IFrame 以及一些 JavaScript 将报表集成或嵌入到应用程序中的基本代码。

在 [Microsoft Power BI Embedded 预览版入门](/documentation/articles/power-bi-embedded-get-started) 中，了解如何配置**工作区集合**，为报表内容保留一个或多个**工作区**。然后，在 [Microsoft Power BI Embedded 示例入门](/documentation/articles/power-bi-embedded-get-started-sample)中，了解如何将报表导入**工作区**。

本文演示将报表嵌入应用程序中的步骤。若要按照本文进行操作，请在 GitHub 中下载[集成带有 IFrame 的报表](https://github.com/Azure-Samples/power-bi-embedded-iframe)示例。此示例是一个简单的 ASP.NET Web 表单应用程序，用于说明集成报表所需的基本 C# 和 JavaScript 代码。有关使用模型-视图-控制器 (MVC) 设计模式来集成报表的更高级示例，请参阅 GitHub 上的 [Web 应用示例仪表板](http://go.microsoft.com/fwlink/?LinkId=761493)。

接下来我们将介绍如何将 **Power BI Embedded** 报表集成到应用程序中。

以下是集成报表的步骤。

- 步骤 1：[在工作区中获取一个报表](#GetReport)。在此步骤中，请使用应用程序令牌流来获取调用 [Get Reports（获取报表）](https://msdn.microsoft.com/zh-cn/library/mt711510.aspx)REST 操作的访问令牌。从“获取报表”列表中获取报表后，即可将该报表嵌入带有**IFrame** 元素的应用程序中。
- 步骤 2：[将报表嵌入应用程序中](#EmbedReport)。在此步骤中，请使用报表的嵌入令牌、一些 JavaScript 以及 IFrame 将报表集成或嵌入到 Web 应用中。

如果希望运行示例以查看如何集成报表，请在 GitHub 中下载[集成带有 IFrame 的报表](https://github.com/Azure-Samples/power-bi-embedded-iframe)示例，并配置三个 Web.Config 设置：

- **AccessKey**：**AccessKey** 用于生成 JSON Web 令牌 (JWT)，以获取报表和嵌入报表。若要了解如何获取 **AccessKey**，请参阅 [Microsoft Power BI Embedded 预览版入门](/documentation/articles/power-bi-embedded-get-started)。
- **WorkspaceName**：若要了解如何获取 **WorkspaceName**，请参阅 [Microsoft Power BI Embedded 预览版入门](/documentation/articles/power-bi-embedded-get-started)。
- **WorkspaceId**：若要了解如何获取 **WorkspaceId**，请参阅 [Microsoft Power BI Embedded 预览版入门](/documentation/articles/power-bi-embedded-get-started)。

后续部分演示集成报表所需的代码。

<a name="GetReport"/></a>
## 在工作区中获取一个报表

若要将报表集成到应用程序中，需要报表 **ID** 和 **embedUrl**。若要获取报表 **ID** 和 **embedUrl**，请调用 [Get Reports（获取报表）](https://msdn.microsoft.com/zh-cn/library/mt711510.aspx)REST 操作，并从 JSON 列表中选择一个报表。在[将报表嵌入应用程序中](#EmbedReport)，请使用报表 **ID** 和 **embedUrl** 将报表嵌入应用程序中。

### 获取报表的 JSON 响应
```
{
  "@odata.context":"https://api.powerbi.com/beta/collections/{WorkspaceName}/workspaces/{WorkspaceId}/$metadata#reports","value":[
    {
      "id":"804d3664-…-e71882055dba","name":"Import report sample","webUrl":"https://embedded.powerbi.com/reports/804d3664-...-e71882055dba","embedUrl":"https://embedded.powerbi.com/appTokenReportEmbed?reportId=804d3664-...-e71882055dba"
    },{
      "id":"1d7cff58-…-380610e263a9","name":"Sample Report 2","webUrl":"https://embedded.powerbi.com/reports/1d7cff58-...-380610e263a9","embedUrl":"https://embedded.powerbi.com/appTokenReportEmbed?reportId=1d7cff58-...-380610e263a9"
    }
  ]
}

```

若要调用 [Get Reports（获取报表）](https://msdn.microsoft.com/zh-cn/library/mt711510.aspx) REST 操作，请使用应用程序令牌。若要了解有关应用程序令牌流的详细信息，请参阅[关于 Power BI Embedded 中的应用程序令牌流](/documentation/articles/power-bi-embedded-app-token-flow)。下面的代码说明如何获取报表的 JSON 列表。若要嵌入报表，请参阅[将报表嵌入应用程序中](#EmbedReport)。

```
protected void getReportsButton_Click(object sender, EventArgs e)
{
    //Get an app token to generate a JSON Web Token (JWT). An app token flow is a claims-based design pattern.
    //To learn how you can code an app token flow to generate a JWT, see the PowerBIToken class.
    var appToken = PowerBIToken.CreateDevToken(workspaceName, workspaceId);

    //After you get a PowerBIToken which has Claims including your WorkspaceName and WorkspaceID,
    //you generate JSON Web Token (JWT) . The Generate() method uses classes from System.IdentityModel.Tokens: SigningCredentials,
    //JwtSecurityToken, and JwtSecurityTokenHandler.
    string jwt = appToken.Generate(accessKey);

    //Set app token textbox to JWT string to show that the JWT was generated
    appTokenTextbox.Text = jwt;

    //Construct reports uri resource string
    var uri = String.Format("https://api.powerbi.com/beta/collections/{0}/workspaces/{1}/reports", workspaceName, workspaceId);

    //Configure reports request
    System.Net.WebRequest request = System.Net.WebRequest.Create(uri) as System.Net.HttpWebRequest;
    request.Method = "GET";
    request.ContentLength = 0;

    //Set the WebRequest header to AppToken, and jwt
    //Note the use of AppToken instead of Bearer
    request.Headers.Add("Authorization", String.Format("AppToken {0}", jwt));

    //Get reports response from request.GetResponse()
    using (var response = request.GetResponse() as System.Net.HttpWebResponse)
    {
        //Get reader from response stream
        using (var reader = new System.IO.StreamReader(response.GetResponseStream()))
        {
            //Deserialize JSON string into PBIReports
            PBIReports PBIReports = JsonConvert.DeserializeObject<PBIReports>(reader.ReadToEnd());

            //Clear Textbox
            tb_reportsResult.Text = string.Empty;

            //Get each report
            foreach (PBIReport rpt in PBIReports.value)
            {
                tb_reportsResult.Text += String.Format("{0}/t{1}/t{2}/n", rpt.id, rpt.name, rpt.embedUrl);
            }
        }
    }
}
```

<a name="EmbedReport"/></a>
## 将报表嵌入应用程序中

在将报表嵌入应用程序之前，需要提供报表的嵌入令牌。此令牌类似于用于调用 **Power BI Embedded** REST 操作的应用程序令牌，但它针对报表资源而非 REST 资源生成。以下是获取报表的应用程序令牌的代码。若要使用报表的应用程序令牌，请参阅[将报表嵌入应用程序中](#EmbedReportJS)。

<a name="EmbedReportToken"/></a>
### 获取报表的应用程序令牌

```
protected void getReportAppTokenButton_Click(object sender, EventArgs e)
{
    //Get an embed token for a report.
    var token = PowerBIToken.CreateReportEmbedToken(workspaceName, workspaceId, getReportAppTokenTextBox.Text);

    //After you get a PowerBIToken which has Claims including your WorkspaceName and WorkspaceID,
    //you generate JSON Web Token (JWT) . The Generate() method uses classes from System.IdentityModel.Tokens: SigningCredentials,
    //JwtSecurityToken, and JwtSecurityTokenHandler.
    string jwt = token.Generate(accessKey);

    //Set textbox to JWT string. The JavaScript embed code uses the embed jwt for a report.
    reportAppTokenTextbox.Text = jwt;
}
```

<a name="EmbedReportJS"/></a>
### 将报表嵌入应用程序中

若要将 **Power BI** 报表嵌入应用程序中，请使用 IFrame 和一些 JavaScript 代码。以下是用于嵌入报表的示例 IFrame 和 JavaScript 代码。若要查看用于嵌入报表的所有示例代码，请参阅 GitHub 上的[集成带有 IFrame 的报表](https://github.com/Azure-Samples/power-bi-embedded-iframe)。

![Iframe](./media/power-bi-embedded-integrate-report/Iframe.png)


```
window.onload = function () {
    // Client side click to embed a selected report.
    var el = document.getElementById("bEmbedReportAction");
    if (el.addEventListener) {
        el.addEventListener("click", updateEmbedReport, false);
    } else {
        el.attachEvent('onclick', updateEmbedReport);
    }

};

// Update embed report
function updateEmbedReport() {
    // Check if the embed url was selected
    var embedUrl = document.getElementById('tb_EmbedURL').value;

    // To load a report do the following:
    // 1: Set the url
    // 2: Add a onload handler to submit the auth token
    var iframe = document.getElementById('iFrameEmbedReport');
    iframe.src = embedUrl;
    iframe.onload = postActionLoadReport;
}

// Post the auth token to the iFrame.
function postActionLoadReport() {

    // Get the app token.
    accessToken = document.getElementById('MainContent_reportAppTokenTextbox').value;

    // Construct the push message structure
    var m = { action: "loadReport", accessToken: accessToken };
    message = JSON.stringify(m);

    // Push the message.
    iframe = document.getElementById('iFrameEmbedReport');
    iframe.contentWindow.postMessage(message, "*");
}
```
将报表嵌入应用程序中后，可以对报表进行筛选。下一部分演示如何使用 URL 语法筛选报表。

## 筛选报表

可以使用 URL 语法筛选嵌入的报表。要进行筛选，可以将查询字符串参数添加到 iFrame src url，并指定筛选器。可以**按值筛选**和**隐藏筛选窗格**。


**按值筛选**

若要按值进行筛选，请按如下所示使用 **$filter** 查询语法和 **eq** 运算符：

```
https://app.powerbi.com/reportEmbed
?reportId=d2a0ea38-0694-...-ee9655d54a4a&
$filter={tableName/fieldName}%20eq%20'{fieldValue}'
```

例如，可以筛选“连锁商店”为“Lindseys”的位置。url 的筛选部分如下所示：

```
$filter=Store/Chain%20eq%20'Lindseys'
```

> [AZURE.NOTE] {tableName/fieldName} 不能包含空格或特殊字符。{fieldValue} 接受单个分类值。

**隐藏筛选窗格**

若要隐藏**筛选窗格**，请按如下所示将 **filterPaneEnabled** 添加到报表查询字符串中：

```
&filterPaneEnabled=false
```

## 结束语

在本文中，你已经了解了将 **Power BI** 报表集成到应用程序中的代码。若要快速开始将报表集成到应用程序中，请在 GitHub 上下载以下示例：

- [集成带有 IFrame 的报表示例](https://github.com/Azure-Samples/power-bi-embedded-iframe)
- [Web 应用示例仪表板](http://go.microsoft.com/fwlink/?LinkId=761493)

## 另请参阅
- [Microsoft Power BI Embedded 预览版入门](/documentation/articles/power-bi-embedded-get-started)
- [示例入门](/documentation/articles/power-bi-embedded-get-started-sample)
- [System.IdentityModel.Tokens.SigningCredentials](https://msdn.microsoft.com/zh-cn/library/system.identitymodel.tokens.signingcredentials.aspx)
- [System.IdentityModel.Tokens.JwtSecurityToken](https://msdn.microsoft.com/zh-cn/library/system.identitymodel.tokens.jwtsecuritytoken.aspx)
- [System.IdentityModel.Tokens.JwtSecurityTokenHandler](https://msdn.microsoft.com/zh-cn/library/system.identitymodel.tokens.signingcredentials.aspx)
- [Get Reports（获取报表）](https://msdn.microsoft.com/zh-cn/library/mt711510.aspx)

<!---HONumber=Mooncake_0523_2016-->