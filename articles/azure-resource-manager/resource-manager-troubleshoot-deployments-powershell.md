<properties
    pageTitle="使用 PowerShell 查看部署操作 | Azure"
    description="介绍如何使用 Azure PowerShell 来检测 Resource Manager 部署的问题。"
    services="azure-resource-manager,virtual-machines"
    documentationcenter=""
    tags="top-support-issue"
    author="tfitzmac"
    manager="timlt"
    editor="" />  

<tags
    ms.assetid="51cb9716-ab95-4024-afc4-ae9cf5b2f4b7"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-multiple"
    ms.workload="infrastructure"
    ms.date="06/14/2016"
    wacn.date="12/26/2016"
    ms.author="tomfitz" />  


# 使用 Azure PowerShell 查看部署操作
>[AZURE.SELECTOR]
[Portal](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)
[PowerShell](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)
[Azure CLI](/documentation/articles/resource-manager-troubleshoot-deployments-cli/)
[REST API](/documentation/articles/resource-manager-troubleshoot-deployments-rest/)

可以通过 Azure PowerShell 查看部署操作。当你在部署过程中收到错误时，可能最想要查看操作，因此本文将重点介绍如何查看已失败的操作。PowerShell 提供 cmdlet 来让你轻松找到错误并确定可能的解决方法。

[AZURE.INCLUDE [resource-manager-troubleshoot-introduction](../../includes/resource-manager-troubleshoot-introduction.md)]

如果部署之前先验证模板和基础结构，则可以避免一些错误。你还可以在部署过程中记录其他请求和响应信息，以方便以后进行故障排除。若要了解有关验证和记录请求和响应信息的内容，请参阅 [Deploy a resource group with Azure Resource Manager template](/documentation/articles/resource-group-template-deploy/)（使用 Azure Resource Manager 模板部署资源组）。

## 使用部署操作进行故障排除
1. 若要获取部署的总体状态，请使用 **Get-AzureRmResourceGroupDeployment** 命令。你可以筛选结果，以便只获取失败的部署。
   
        Get-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup | Where-Object ProvisioningState -eq Failed
   
    这会使用以下格式返回失败的部署：
   
        DeploymentName          : Microsoft.Template
        ResourceGroupName       : ExampleGroup
        ProvisioningState       : Failed
        Timestamp               : 6/14/2016 9:55:21 PM
        Mode                    : Incremental
        TemplateLink            :
        Parameters              :
                    Name                Type                 Value
                    ===============     ===================  ==========
                    storageAccountName  String               tfstorage9855
                    adminUsername       String               tfadmin
                    adminPassword       SecureString
                    dnsNameforLBIP      String               dns
                    vmNamePrefix        String               myVM
                    imagePublisher      String               MicrosoftWindowsServer
                    imageOffer          String               WindowsServer
                    imageSKU            String               2012-R2-Datacenter
                    lbName              String               myLB
                    nicNamePrefix       String               nic
                    publicIPAddressName String               myPublicIP
                    vnetName            String               myVNET
                    vmSize              String               Standard_D1
   
        Outputs                 :
        DeploymentDebugLogLevel :
2. 每个部署通常由多个操作组成，而每个操作表示部署过程中的一个步骤。为了查明部署何处出现问题，通常需要查看部署操作的详细信息。可以用 **Get-AzureRmResourceGroupDeploymentOperation** 查看操作的状态。
   
        Get-AzureRmResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName Microsoft.Template
   
    它将返回多个操作，其中每个操作采用以下格式：
   
        Id             : /subscriptions/{guid}/resourceGroups/ExampleGroup/providers/Microsoft.Resources/deployments/Microsoft.Template/operations/A3EB2DA598E0A780
        OperationId    : A3EB2DA598E0A780
        Properties     : @{provisioningOperation=Create; provisioningState=Succeeded; timestamp=2016-06-14T21:55:15.0156208Z;
                         duration=PT23.0227078S; trackingId=11d376e8-5d6d-4da8-847e-6f23c6443fbf;
                         serviceRequestId=0196828d-8559-4bf6-b6b8-8b9057cb0e23; statusCode=OK; targetResource=}
        PropertiesText : {duration:PT23.0227078S, provisioningOperation:Create, provisioningState:Succeeded,
                         serviceRequestId:0196828d-8559-4bf6-b6b8-8b9057cb0e23...}
3. 若要获取有关失败操作的更多详细信息，请检索状态为“失败”的操作的属性。
   
        (Get-AzureRmResourceGroupDeploymentOperation -DeploymentName Microsoft.Template -ResourceGroupName ExampleGroup).Properties | Where-Object ProvisioningState -eq Failed
   
    它将返回所有失败的操作，其中每个操作采用以下格式：
   
        provisioningOperation : Create
        provisioningState     : Failed
        timestamp             : 2016-06-14T21:54:55.1468068Z
        duration              : PT3.1449887S
        trackingId            : f4ed72f8-4203-43dc-958a-15d041e8c233
        serviceRequestId      : a426f689-5d5a-448d-a2f0-9784d14c900a
        statusCode            : BadRequest
        statusMessage         : @{error=}
        targetResource        : @{id=/subscriptions/{guid}/resourceGroups/ExampleGroup/providers/
                                Microsoft.Network/publicIPAddresses/myPublicIP;
                                resourceType=Microsoft.Network/publicIPAddresses; resourceName=myPublicIP}
   
    请记下操作的跟踪 ID。你将在下一步中使用它来重点关注特定操作。
4. 若要获取特定失败操作的状态消息，请使用以下命令：
   
        ((Get-AzureRmResourceGroupDeploymentOperation -DeploymentName Microsoft.Template -ResourceGroupName ExampleGroup).Properties | Where-Object trackingId -eq f4ed72f8-4203-43dc-958a-15d041e8c233).StatusMessage.error
   
    将返回：
   
        code           message                                                                        details
        ----           -------                                                                        -------
        DnsRecordInUse DNS record dns.chinanorth.chinacloudapp.cn is already used by another public IP. {}

## 使用审核日志进行故障排除
[AZURE.INCLUDE [resource-manager-audit-limitations](../../includes/resource-manager-audit-limitations.md)]

若要查看部署相关的错误，请使用以下步骤：

1. 若要检索日志条目，请运行 **Get-AzureRmLog** 命令。可以使用 **ResourceGroup** 和 **Status** 参数，以便只返回单个资源组失败的事件。如果未指定开始和结束时间，将返回最后一个小时的条目。例如，若要检索过去一小时失败的操作，请运行：
   
        Get-AzureRmLog -ResourceGroup ExampleGroup -Status Failed
   
    可以指定特定的时间跨度。在下一个示例中，我们将查找过去一天失败的操作。
   
        Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-1) -Status Failed
   
    或者，可以针对失败的操作设置确切的开始和结束时间：
   
        Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime 2015-08-28T06:00 -EndTime 2015-09-10T06:00 -Status Failed
2. 如果此命令返回了过多的条目和属性，则可通过检索 **Properties** 属性来专注于审核操作。我们还将包含 **DetailedOutput** 参数，以查看错误消息。
   
        (Get-AzureRmLog -Status Failed -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-1) -DetailedOutput).Properties
   
    这会使用以下格式返回日志条目的属性：
   
        Content
        -------
        {} 
        {[statusCode, BadRequest], [statusMessage, {"error":{"code":"DnsRecordInUse","message":"DNS record dns.chinanorth.clouda...
        {[statusCode, BadRequest], [serviceRequestId, a426f689-5d5a-448d-a2f0-9784d14c900a], [statusMessage, {"error":{"code...
3. 根据这些结果，让我们重点关注第二个元素。你可以通过查看该条目的状态消息来进一步细化结果。
   
        ((Get-AzureRmLog -Status Failed -ResourceGroup ExampleGroup -DetailedOutput -StartTime (Get-Date).AddDays(-1)).Properties[1].Content["statusMessage"] | ConvertFrom-Json).error
   
    将返回：
   
        code           message                                                                        details
        ----           -------                                                                        -------
        DnsRecordInUse DNS record dns.chinanorth.chinacloudapp.cn is already used by another public IP. {}

## 后续步骤
* 有关解决特定部署错误的帮助，请参阅 [Resolve common errors when deploying resources to Azure with Azure Resource Manager](/documentation/articles/resource-manager-common-deployment-errors/)（解决使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误）。
* 若要了解如何使用审核日志来监视其他类型的操作，请参阅 [Audit operations with Resource Manager](/documentation/articles/resource-group-audit/)（使用 Resource Manager 执行审核操作）。
* 若要在执行部署之前验证部署，请参阅 [Deploy a resource group with Azure Resource Manager template](/documentation/articles/resource-group-template-deploy/)（使用 Azure Resource Manager 模板部署资源组）。

<!---HONumber=Mooncake_1219_2016-->