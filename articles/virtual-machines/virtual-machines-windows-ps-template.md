<properties
  pageTitle="使用 Resource Manager 模板创建 VM | Microsoft Docs"
  description="将 Resource Manager 模板与 PowerShell 配合使用，以轻松创建新的 Windows 虚拟机。"
  services="virtual-machines-windows" 
  documentationcenter=""
  author="davidmu1"
  manager="timlt"
  editor=""
  tags="azure-resource-manager"/>

<tags
  ms.assetid="19129d61-8c04-4aa9-a01f-361a09466805"
  ms.service="virtual-machines-windows"
  ms.workload="na"
  ms.tgt_pltfrm="vm-windows"
  ms.devlang="na"
  ms.topic="article"
  ms.date="01/06/2017"
  wacn.date="02/24/2017"
  ms.author="davidmu"/>

# 使用 Resource Manager 模板创建 Windows 虚拟机

本文演示了如何使用 PowerShell 部署 Azure Resource Manager 模板。[此模板](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json)将在包含单个子网的新虚拟网络上部署运行 Windows Server 的单个虚拟机。

有关虚拟机资源的详细说明，请参阅 [Azure Resource Manager 模板中的虚拟机](/documentation/articles/virtual-machines-windows-template-description/)。若要详细了解模板中的所有资源，请参阅 [Azure Resource Manager 模板演练](/documentation/articles/resource-manager-template-walkthrough/)。

执行本文中的步骤大约需要 5 分钟时间。

## 步骤 1：安装 Azure PowerShell
有关安装最新版本的 Azure PowerShell、选择订阅和登录帐户的信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

## 步骤 2：创建资源组
必须在[资源组](/documentation/articles/resource-group-overview/)中部署所有资源。

1. 获取可以创建资源的可用位置列表。

        Get-AzureRmLocation | sort DisplayName | Select DisplayName

2. 在所选位置中创建资源组。本示例演示了在**中国北部**位置创建一个名为 **myResourceGroup** 的资源组：

        New-AzureRmResourceGroup -Name "myResourceGroup" -Location "China North"

    用户应看到与此示例类似的内容：

        ResourceGroupName : myResourceGroup
        Location          : chinaeast
        ProvisioningState : Succeeded
        Tags              :
        ResourceId        : /subscriptions/{subscription-id}/resourceGroups/myResourceGroup

## 步骤 3：创建资源
部署模板并在出现提示时提供参数值。此示例在所创建的资源组中部署了 101-vm-simple-windows 模板：

    New-AzureRmResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json" 

需要提供 VM 上的管理员帐户名称、帐户密码和 DNS 前缀。

用户应看到与此示例类似的内容：

    DeploymentName    : azuredeploy
    ResourceGroupName : myResourceGroup
    ProvisioningState : Succeeded
    Timestamp         : 12/29/2016 8:11:37 PM
    Mode              : Incremental
    TemplateLink      :
                        Uri            : https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/
                        101-vm-simple-windows/azuredeploy.json
                        ContentVersion : 1.0.0.0
    Parameters        :
                        Name             Type                       Value
                        ===============  =========================  ==========
                        adminUsername    String                     myAdminUser
                        adminPassword    SecureString
                        dnsLabelPrefix   String                     myDomain
                        windowsOSVersion String                     2016-Datacenter
    Outputs           :
                        Name             Type                       Value
                        ===============  =========================  ===========
                        hostname         String                     myDomain.chinaeast.chinacloudapp.cn
    DeploymentDebugLogLevel :

> [AZURE.NOTE]
还可通过本地文件部署模板和参数。有关详细信息，请参阅[对 Azure 存储使用 Azure PowerShell](/documentation/articles/storage-powershell-guide-full/)。

## 后续步骤
* 如果部署出现问题，请参阅[排查使用 Azure Resource Manager 时的 Azure 常见部署错误](/documentation/articles/resource-manager-common-deployment-errors/)
* 若要了解如何管理刚创建的虚拟机，请参阅[使用 Azure Resource Manager 和 PowerShell 管理虚拟机](/documentation/articles/virtual-machines-windows-ps-manage/)。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->