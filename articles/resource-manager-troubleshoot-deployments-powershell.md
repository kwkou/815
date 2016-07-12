<properties
   pageTitle="使用 PowerShell 对部署进行故障排除 | Azure"
   description="介绍如何使用 Azure PowerShell 来检测和解决资源管理器部署的问题。"
   services="azure-resource-manager,virtual-machines"
   documentationCenter=""
   tags="top-support-issue"
   authors="tfitzmac"
   manager="timlt"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="03/21/2016"
   wacn.date="05/05/2016"/>

# 使用 Azure PowerShell 对资源组部署进行故障排除

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)
- [REST API](/documentation/articles/resource-manager-troubleshoot-deployments-rest/)

如果在将资源部署到 Azure 时发生错误，你需要进行故障排除。Azure PowerShell 提供 cmdlet 来让你轻松找到错误并确定可能的解决方法。

可以通过查看审核日志或部署操作来对部署进行故障排除。本主题将演示这两种方法。

如果部署之前先验证模板和基础结构，则可以避免一些错误。有关详细信息，请参阅 [使用 Azure Resource Manager 模板部署资源组](/documentation/articles/resource-group-template-deploy/)。

## 使用审核日志进行故障排除

[AZURE.INCLUDE [resource-manager-audit-limitations](../includes/resource-manager-audit-limitations.md)]

若要查看部署相关的错误，请使用以下步骤：

1. 若要检索日志条目，请运行 **Get-AzureRmLog** 命令。可以使用 **ResourceGroup** 和 **Status** 参数，以便只返回单个资源组失败的事件。如果未指定开始和结束时间，将返回最后一个小时的条目。
例如，若要检索过去一小时失败的操作，请运行：

        Get-AzureRmLog -ResourceGroup ExampleGroup -Status Failed

    可以指定特定的时间跨度。在下一个示例中，我们将查找过去 14 天失败的操作。

        Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14) -Status Failed
      
    或者，可以针对失败的操作设置确切的开始和结束时间：

        Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime 2015-08-28T06:00 -EndTime 2015-09-10T06:00 -Status Failed

2. 如果此命令返回了过多的条目和属性，你可以通过检索 **Properties** 属性来专注于审核。我们还将包含 **DetailedOutput** 参数，以查看错误消息。

        (Get-AzureRmLog -Status Failed -ResourceGroup ExampleGroup -DetailedOutput).Properties
        
    这会使用以下格式返回日志条目的属性：
        
        Content
        -------
        {}
        {[statusCode, Conflict], [statusMessage, {"Code":"Conflict","Message":"Website with given name mysite already exists...
        {[statusCode, Conflict], [serviceRequestId, ], [statusMessage, {"Code":"Conflict","Message":"Website with given name...

3. 可以通过查看一个特定条目的状态消息来进一步细化结果。

        (Get-AzureRmLog -Status Failed -ResourceGroup ExampleGroup -DetailedOutput).Properties[1].Content["statusMessage"] | ConvertFrom-Json
        
    这会使用以下格式返回状态消息：
        
        Code       : Conflict
        Message    : Website with given name mysite already exists.
        Target     :
        Details    : {@{Message=Website with given name mysite already exists.}, @{Code=Conflict}, @{ErrorEntity=}}
        Innererror :


## 使用部署操作进行故障排除

1. 若要获取部署的总体状态，请使用 **Get-AzureRmResourceGroupDeployment** 命令。你可以筛选结果，以便只获取失败的部署。

        Get-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup | Where-Object ProvisioningState -eq Failed
        
    这会使用以下格式返回失败的部署：
        
        DeploymentName    : ExampleDeployment
        ResourceGroupName : ExampleGroup
        ProvisioningState : Failed
        Timestamp         : 8/27/2015 8:03:34 PM
        Mode              : Incremental
        TemplateLink      :
        Parameters        :
        Name             Type                       Value
        ===============  =========================  ==========
        siteName         String                     ExampleSite
        hostingPlanName  String                     ExamplePlan
        siteLocation     String                     China East
        sku              String                     Free
        workerSize       String                     0
        
        Outputs           :

2. 每个部署通常由多个操作组成，而每个操作表示部署过程中的一个步骤。为了查明部署何处出现问题，通常需要查看部署操作的详细信息。可以用 **Get-AzureRmResourceGroupDeploymentOperation** 查看操作的状态。

        Get-AzureRmResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName ExampleDeployment | Format-List
        
    这会使用以下格式返回操作：
        
        Id          : /subscriptions/{guid}/resourceGroups/ExampleGroup/providers/Microsoft.Resources/deployments/ExampleDeployment/operations/8518B32868A437C8
        OperationId : 8518B32868A437C8
        Properties  : @{ProvisioningOperation=Create; ProvisioningState=Failed; Timestamp=2016-03-16T20:05:37.2638161Z;
                      Duration=PT2.8834832S; TrackingId=192fbfbf-a2e2-40d6-b31d-890861f78ed3; StatusCode=Conflict;
                      StatusMessage=; TargetResource=}

3. 若要获取有关操作的更多详细信息，请检索 **Properties** 对象。

        (Get-AzureRmResourceGroupDeploymentOperation -DeploymentName ExampleDeployment -ResourceGroupName ExampleGroup).Properties
        
    这会使用以下格式返回操作属性：
        
        ProvisioningOperation : Create
        ProvisioningState     : Failed
        Timestamp             : 2016-03-16T20:05:37.2638161Z
        Duration              : PT2.8834832S
        TrackingId            : 192fbfbf-a2e2-40d6-b31d-890861f78ed3
        StatusCode            : Conflict
        StatusMessage         : @{Code=Conflict; Message=Website with given name mysite already exists.; Target=;
        Details=System.Object[]; Innererror=}
        TargetResource        : @{Id=/subscriptions/{guid}/resourceGroups/ExampleGroup/providers/
        Microsoft.Web/Sites/mysite; ResourceType=Microsoft.Web/Sites; ResourceName=mysite}

4. 若要专注于失败操作的状态消息，请使用以下命令：

        ((Get-AzureRmResourceGroupDeploymentOperation -DeploymentName ExampleDeployment -ResourceGroupName ExampleGroup).Properties | Where-Object ProvisioningState -eq Failed).StatusMessage
        
    这会使用以下格式返回状态消息：
        
        Code       : Conflict
        Message    : Website with given name mysite already exists.
        Target     :
        Details    : {@{Message=Website with given name mysite already exists.}, @{Code=Conflict}, @{ErrorEntity=}}
        Innererror :

## 后续步骤

- 若要了解如何使用审核日志来监视其他类型的操作，请参阅 [使用资源管理器执行审核操作](/documentation/articles/resource-group-audit/)。
- 若要在执行部署之前验证部署，请参阅 [使用资源管理器模板部署资源组](/documentation/articles/resource-group-template-deploy/)。


<!---HONumber=Mooncake_0425_2016-->