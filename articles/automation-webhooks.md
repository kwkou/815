<properties pageTitle="Azure 自动化 Webhook" description="一个可供客户端通过 HTTP 调用在 Azure 自动化中启动 Runbook 的 Webhook。本文介绍了如何创建 Webhook，以及如何通过调用 Webhook 来启动 Runbook。" services="automation" documentationCenter="" authors="bwren" manager="stevenka" editor="tysonn"/>

<tags ms.service="automation" ms.date="05/13/2015" wacn.date="06/16/2015"/> 

# Azure 自动化 Webhook

*Webhook* 可以用来在 Azure 自动化中通过单个 HTTP 请求来启动特定的 Runbook。这样，外部服务（例如 GitHub 或自定义应用程序）就可以在不通过 Azure 自动化 API 实施完整解决方案的情况下启动 Runbook。

![Webhook](./media/automation-webhooks/webhooks-overview.png)

你可以将 Webhook 与[在 Azure 自动化中启动 Runbook](automation-starting-a-runbook) 中其他启动 Runbook 的方法进行比较。

## Webhook 详细信息

下表介绍了你必须为 Webhook 配置的属性。

| 属性 | 说明 |  
|:---|:---|  
| Name | 你可以提供要用于 Webhook 的任何名称，因为该名称不会公开给客户端。它只供你用来标识 Azure 自动化中的 Runbook。</br> 最好是为 Webhook 提供一个与要使用该名称的客户端相关的名称。 |  
|URL |Webhook 的 URL 是客户端通过 HTTP POST 来调用的唯一地址，用于启动链接到 Webhook 的 Runbook。它是在你创建 Webhook 时自动生成的。不能指定自定义 URL。</br> </br> URL 包含一个允许第三方系统调用 Runbook 的安全令牌，不需要进一步进行身份验证。因此，应将其视为密码。出于安全原因，你只能在创建 Webhook 时通过 Azure 预览门户查看该 URL。你应该将保存在安全位置的 URL 记下来，供将来使用。 |  
|到期日期 | 与证书一样，每个 Webhook 都有一个过期日期，到了过期日期 Webhook 将不再可用。此到期日期在 Webhook 创建后将无法更改，而在到达到期日期以后，Webhook 也将无法再次启用。在这种情况下，你必须创建另一个 Webhook 来替换当前的 Webhook，并对客户端进行更新，使之能够使用新的 Webhook。 |  
| Enabled | Runbook 在创建时默认启用。如果将 Runbook 设置为“Disabled”，将没有客户端能够使用它。你可以在创建 Webhook 时设置 **Enabled** 属性，也可以在创建后随时设置它。 |


### Parameters
Webhook 可以定义 Runbook 参数的值，当该 Webhook 启动 Runbook 时会用到这些值。Webhook 必须包含 Runbook 的任何必需参数的值，可以包含可选参数的值。链接到单个 Runbook 的多个 Webhook 可以使用不同的参数值。

>[AZURE.NOTE]创建 Webhook 后，不能更改由 Webhook 目前设置的参数值。你必须创建另一个使用其他参数值的 Webhook。

客户端在使用 Webhook 启动 Runbook 时，不能重写在 Webhook 中定义的参数值。为了从客户端接收数据，Runbook 可能会接受名为 **$WebhookData** 且类型为[对象]的单个参数，该参数会包含客户端包括在 POST 请求中的数据。

![Webhookdata](./media/automation-webhooks/webhookdata.png)

**$WebhookData** 对象将具有以下属性：

| 属性 | 说明 |  
|:-------- |:------------|  
| WebhookName | Webhook 的名称。 |  
| RequestHeader | 传入 POST 请求的标头。 |  
| RequestBody | 传入 POST 请求的正文。 |

不需配置 Webhook 即可支持 **$WebhookData** 参数，也不需要 Runbook 来接受它。如果 Runbook 没有定义该参数，则会忽略从客户端发送的请求的任何详细信息。

如果你在创建 Webhook 时为 $WebhookData 指定了值，则会在 Webhook 使用客户端 POST 请求中的数据启动 Runbook 时重写该值，即使该客户端的请求正文中不包含任何数据。如果你使用 Webhook 以外的方法启动包含 $WebhookData 的 Runbook，则可为能够被 Runbook 识别的 $WebhookData 提供值。该值应该是一个其属性与 $Webhookdata 相同的对象，这样 Runbook 就可以正常地使用它。

>[AZURE.NOTE]所有输入参数的值都会通过 Runbook 作业进行记录。这意味着，客户端提供的任何输入都将通过 $WebhookData 记录下来，并可供有权访问自动化作业的任何人使用。因此，在 Webhook 调用中包括敏感信息时，应该特别小心。

## “安全”

Webhook 的安全性取决于其 URL 的私密性，可以通过 URL 中包含的安全令牌来调用它。如果请求是向正确的 URL 发出的，Azure 自动化不对请求进行任何身份验证。因此，不应将 Webhook 用于执行敏感性很高的功能却不使用替代方法来验证请求的 Runbook。

你可以将逻辑包括在 Runbook 中，以便确定它是否是通过 Webhook 调用的，只需检查 $WebhookData 参数的 **WebhookName** 属性即可。该 Runbook 还可以执行进一步的验证，只需在 **RequestHeader** 或 **RequestBody** 属性中查找特定信息即可。

另一种策略是让 Runbook 在收到 Webhook 请求时对外部条件执行某些验证。例如，当有新的内容提交到 GitHub 存储库时，可通过 GitHub 调用 Runbook。Runbook 在继续之前，可能会连接到 GitHub 来验证是否确实有新的提交内容。

## 创建 Webhook

在 Azure 预览门户中使用以下过程来创建新的链接到 Runbook 的 Webhook。

1. 在 Azure 预览门户的**“Runbook”边栏选项卡**中，单击需要通过 Webhook 来启动以查看其详细信息边栏选项卡的 Runbook。
 
2. 单击边栏选项卡顶部的 **Webhook** 以打开“添加 Webhook”边栏选项卡。
 
	![Webhook 按钮](./media/automation-webhooks/webhooks-button.png)
	
3. 单击“创建新的 Webhook”以打开“创建 Webhook 边栏选项卡”。

4. 指定 Webhook 的“名称”、“到期日期”，以及是否应启用它。若需这些属性的详细信息，请参阅 [Webhook 详细信息](#details-of-a-webhook)。

5. 单击复制图标，然后按 Ctrl+C 以复制 Webhook 的 URL。然后，将其记录在某个安全的位置。**一旦创建 Webhook，你就不能再次检索该 URL。**
 
	![Webhook URL](./media/automation-webhooks/copy-webhook-url.png)

6. 单击“创建”以创建 Webhook。

7. 单击“参数”为 Runbook 参数提供值。

8. 配置完 Webhook 后，单击“确定”。


## 使用 Webhook

若要使用 Webhook，客户端应用程序必须发出带 Webhook URL 的 HTTP POST。Webhook 的语法将按以下格式。

	http://<Webhook Server>/token?=<Token Value>


客户端在响应 POST 请求时会收到以下代码之一。

| 代码 | 文本 | 说明 |
|:------------- |:------------- |:----- |
| 202 | 已接受 | 已接受该请求，并已成功启动 Runbook。 |
| 400 | 错误的请求 | 出于以下原因之一，未接受该请求。<ul> <li>Webhook 已过期。</li> <li>Webhook 处于禁用状态。</li> <li>URL 中的令牌无效。</li></ul> |
| 500 | 内部服务器错误 | URL 有效，但出现了错误。请重新提交请求。 |

客户端无法确定 Runbook 作业何时完成，也无法从 Webhook 获取其完成状态，因为 Webhook 不返回 Runbook 作业的标识符。你只能验证该请求是否已成功提交。

### 示例

下面的示例通过 Windows PowerShell 使用 Webhook 来启动 Runbook。此示例包括标头和正文中可供 Runbook 使用的数据。请注意，任何可以发出 HTTP 请求的语言都可以使用 Webhook。

	$uri = "https://oaaswebhookcurrent.chinacloudapp.cn/webhooks?token=8ud0dSrSo%2fvHWpYbklW%3c8s0GrOKJZ9Nr7zqcS%2bIQr4c%3d"
	$headers = @{"header1"="headerval1";"header2"="headerval2"}
	$body = "some request body"

	Invoke-RestMethod -Method Post -Uri $uri -Headers $headers -Body $body

以下示例 Runbook 接受了以前的请求，然后从 Webhook 中检索数据。

	workflow Sample-Webhook
	{
		param (	
				[object]$WebhookData
		)
	
		$WebhookName 	= 	$WebhookData.WebhookName
		$WebhookHeaders = 	$WebhookData.RequestHeader
		$WebhookBody 	= 	$WebhookData.RequestBody
	} 

	

## 相关文章

- [启动 Runbook](automation-starting-a-runbook)
- [查看 Runbook 作业的状态](automation-viewing-the-status-of-a-runbook-job)

<!---HONumber=60-->