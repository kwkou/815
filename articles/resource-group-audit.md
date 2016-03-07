<properties 
	pageTitle="使用资源管理器执行审核操作 | Azure" 
	description="使用资源管理器中的审核日志查看用户操作和错误。显示 PowerShell、Azure CLI 和 REST。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="10/27/2015" 
	wacn.date="12/31/2015"/>

# 使用资源管理器执行审核操作

当你在部署期间或在解决方案生存期内遇到问题时，需要了解发生了什么问题。资源管理器提供了两种方式来查明发生了什么情况及其原因。你可以使用部署命令来检索有关特定部署和操作的信息。或者，你可以使用审核日志来检索有关部署以及在解决方案生存期采取的其他操作的信息。本主题着重于审核日志。

审核日志包含对资源执行的所有操作。因此，如果组织中的用户修改了资源，你将可以识别相应的操作、时间和用户。

在使用审核日志时，需要记住两个重要限制：

1. 审核日志仅保留 90 天。
2. 只能查询 15 天或更少天数内的日志。

可以通过 Azure PowerShell、Azure CLI、REST API 或 Azure 门户检索审核日志中的信息。

## PowerShell

[AZURE.INCLUDE [powershell-preview-inline-include](../includes/powershell-preview-inline-include.md)]

若要检索日志条目，请运行 **Get-AzureRmLog** 命令（如果 PowerShell 版本低于 1.0 预览版，则运行 **Get-AzureResourceGroupLog**）。你可以提供附加参数来筛选条目列表。

以下示例演示了如何使用审核日志来调查在解决方案生存期内执行的操作。你可以查看此操作的发生时间以及请求者。开始日期和结束日期以日期格式指定。

    PS C:\> Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime 2015-08-28T06:00 -EndTime 2015-09-10T06:00

或者，可以使用 date 函数来指定日期范围，例如过去 15 天。

    PS C:\> Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-15)

根据指定的开始时间，前面的命令可能会返回对该资源组执行的一长串操作。你可以提供搜索条件，以筛选所要查找的结果。例如，如果你想要调查 Web 应用的停止方式，可以运行以下命令并查看 someone@example.com 所执行的停止操作。

    PS C:\> Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-15) | Where-Object OperationName -eq Microsoft.Web/sites/stop/action

    Authorization     :
                        Scope     : /subscriptions/xxxxx/resourcegroups/ExampleGroup/providers/Microsoft.Web/sites/ExampleSite
                        Action    : Microsoft.Web/sites/stop/action
                        Role      : Subscription Admin
                        Condition :
    Caller            : someone@example.com
    CorrelationId     : 84beae59-92aa-4662-a6fc-b6fecc0ff8da
    EventSource       : Administrative
    EventTimestamp    : 8/28/2015 4:08:18 PM
    OperationName     : Microsoft.Web/sites/stop/action
    ResourceGroupName : ExampleGroup
    ResourceId        : /subscriptions/xxxxx/resourcegroups/ExampleGroup/providers/Microsoft.Web/sites/ExampleSite
    Status            : Succeeded
    SubscriptionId    : xxxxx
    SubStatus         : OK

在下一个示例中，我们将只查找在指定的开始时间后执行失败的操作。我们还将包含 **DetailedOutput** 参数，以查看错误消息。

    PS C:\> Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-15) -Status Failed –DetailedOutput
    
如果此命令返回了过多的条目和属性，你可以通过检索 **properties** 属性来专注于审核。

    PS C:\> (Get-AzureRmLog -Status Failed -ResourceGroup ExampleGroup -DetailedOutput).Properties

    Content
    -------
    {}
    {[statusCode, Conflict], [statusMessage, {"Code":"Conflict","Message":"Website with given name mysite already exists...
    {[statusCode, Conflict], [serviceRequestId, ], [statusMessage, {"Code":"Conflict","Message":"Website with given name...

此外，你可以通过查看状态消息进一步细化结果。

    PS C:\> (Get-AzureRmLog -Status Failed -ResourceGroup ExampleGroup -DetailedOutput).Properties[1].Content["statusMessage"] | ConvertFrom-Json

    Code       : Conflict
    Message    : Website with given name mysite already exists.
    Target     :
    Details    : {@{Message=Website with given name mysite already exists.}, @{Code=Conflict}, @{ErrorEntity=}}
    Innererror :


## Azure CLI

若要检索日志条目，请运行 **azure group log show** 命令。

    azure group log show ExampleGroup

可以使用 JSON 实用程序（如 [jq](http://stedolan.github.io/jq/download/)）来筛选结果。以下示例演示了如何查找涉及更新 Web 配置文件的操作。

    azure group log show ExampleGroup --json | jq ".[] | select(.operationName.localizedValue == "Update web sites config")"

可以添加 **–-last-deployment** 参数，将返回的条目限制为自上次部署以来执行的操作。

    azure group log show ExampleGroup --last-deployment

如果自上次部署以来执行的操作列表太长，你可以筛选结果以便只显示失败的操作。

    azure group log show tfCopyGroup --last-deployment --json | jq ".[] | select(.status.value == "Failed")"

                                   /Microsoft.Web/Sites/ExampleSite
    data:    SubscriptionId:       <guid>
    data:    EventTimestamp (UTC): Thu Aug 27 2015 13:03:27 GMT-0700 (Pacific Daylight Time)
    data:    OperationName:        Update website
    data:    OperationId:          cb772193-b52c-4134-9013-673afe6a5604
    data:    Status:               Failed
    data:    SubStatus:            Conflict (HTTP Status Code: 409)
    data:    Caller:               someone@example.com
    data:    CorrelationId:        a8c7a2b4-5678-4b1b-a50a-c17230427b1e
    data:    Description:
    data:    HttpRequest:          clientRequestId: <guid>
                                   clientIpAddress: 000.000.000.000
                                   method:          PUT

    data:    Level:                Error
    data:    ResourceGroup:        ExampleGroup
    data:    ResourceProvider:     Azure Web Sites
    data:    EventSource:          Administrative
    data:    Properties:           statusCode:       Conflict
                                   serviceRequestId:
                                   statusMessage:    {"Code":"Conflict","Message":"Website with given name
                                   ExampleSite already exists.","Target":null,"
                                   Details
                                   ":[{"Message":"Website with given name ExampleSite already exists."},
                                   {"Code":"Conflict"},
                                   {"ErrorEntity":{"Code":"Conflict","Message":"Website with given
                                   name ExampleSite already exists.","ExtendedCode
                                   ":"
                                   54001","MessageTemplate":"Website with given name {0} already exists.",
                                   "Parameters":["ExampleSite"],"
                                   InnerErrors":null}}],"Innererror":null}



## REST API

可用于处理审核日志的 REST 操作属于 [Insights REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931943.aspx)。若要检索审核日志事件，请参阅[列出订阅中的管理事件](https://msdn.microsoft.com/zh-cn/library/azure/dn931934.aspx)。

## 后续步骤

- 若要了解如何设置安全策略，请参阅[管理对资源的访问权限](/documentation/articles/resource-group-rbac)。
- 若要了解如何向服务主体授予访问权限，请参阅[使用 Azure 资源管理器对服务主体进行身份验证](/documentation/articles/resource-group-authenticate-service-principal)。
- 若要了解如何操作所有用户的资源，请参阅[使用 Azure 资源管理器锁定资源](/documentation/articles/resource-group-lock-resources)。

<!---HONumber=Mooncake_1221_2015-->