<properties
    pageTitle="使用 PowerShell 和 Azure Resource Manager 复制 Hyper-V VM | Azure"
    description="在 PowerShell 和 Azure Resource Manager 中使用 Azure Site Recovery 将 Hyper-V VM 自动复制到 Azure。"
    services="site-recovery"
    documentationcenter=""
    author="bsiva"
    manager="abhiag"
    editor="" />
<tags
    ms.assetid="05e0d76e-c3f5-4845-8052-094019b6d102"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="backup-recovery"
    ms.date="02/06/2017"
    wacn.date="03/10/2017"
    ms.author="bsiva" />  


# 使用 PowerShell 和 Azure Resource Manager 在本地 Hyper-V 虚拟机与 Azure 之间复制

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/site-recovery-hyper-v-site-to-azure/)
- [PowerShell - Resource Manager](/documentation/articles/site-recovery-deploy-with-powershell-resource-manager/)
- [经典门户](/documentation/articles/site-recovery-hyper-v-site-to-azure-classic/)



## 概述

Azure Site Recovery 可在许多部署方案中安排虚拟机的复制、故障转移和恢复，为业务连续性和灾难恢复策略发挥作用。有关部署方案的完整列表，请参阅 [Azure Site Recovery 概述](/documentation/articles/site-recovery-overview/)。

Azure PowerShell 是一个模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。它可与两种类型的模块配合工作：Azure 配置文件模块或 Azure Resource Manager 模块。

Site Recovery PowerShell cmdlet 在 Azure PowerShell for Azure Resource Manager 中提供，可帮助你保护和恢复你在 Azure 中的服务器。

本文介绍如何使用 Windows PowerShell 及 Azure Resource Manager 来部署 Site Recovery，以便配置和安排对 Azure 的服务器保护。本文所用示例演示了如何通过联合使用 Azure PowerShell 和 Azure Resource Manager 对 Hyper-V 主机上的虚拟机保护、故障转移和恢复到 Azure。

> [AZURE.NOTE] 当前你可以使用 Site Recovery PowerShell cmdlet 配置以下保护、故障转移和恢复方式：从一个虚拟机管理器站点到另一个站点、从虚拟机管理器站点到 Azure 和从 Hyper-V 站点到 Azure。

你无需是一名 PowerShell 专家就可以使用本文章，但是你需要理解诸如模块、cmdlet 和会话等基本概念。有关 Windows PowerShell 的详细信息，请参阅 [Windows PowerShell 入门](http://technet.microsoft.com/zh-cn/library/hh857337.aspx)。

另请阅读有关[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)的更多信息。

> [AZURE.NOTE] 参与云解决方案提供商 (CSP) 计划的 Microsoft 合作伙伴可以根据客户的各自的 CSP 订阅（租户订阅）情况对客户服务器的保护措施进行配置和管理。

## 开始之前

确保已满足以下先决条件：

- 一个 [Azure](https://azure.cn/) 帐户。你可以从[试用版](/pricing/1rmb-trial/)开始。此外，你可以阅读 [Azure Site Recovery Manager 定价](/pricing/details/site-recovery/)。
- Azure PowerShell 1.0。有关此版本及其安装方法的信息，请参阅 [Azure PowerShell 1.0](/documentation/articles/powershell-install-configure/)。
- [AzureRM.SiteRecovery](https://www.powershellgallery.com/packages/AzureRM.SiteRecovery/) 和 [AzureRM.RecoveryServices](https://www.powershellgallery.com/packages/AzureRM.RecoveryServices/) 模块。你可以从 [PowerShell 库](https://www.powershellgallery.com/)获取这些模块的最新版本

本文将举例说明如何使用 Azure Powershell 和 Azure Resource Manager 来配置和管理对服务器的保护。本文中使用的示例演示了如何将在 Hyper-V 主机上运行的虚拟机保护到 Azure。下面是特定于该示例的先决条件。如需不同站点恢复方案的各种详细要求，请参阅与该方案相关的文档。

- 需要一台运行 Windows Server 2012 R2 或 Microsoft Hyper-V Server 2012 R2 的 Hyper-V 主机，其中包含一个或多个虚拟机。
- Hyper-V 服务器应直接或通过代理连接到 Internet。
- 要保护的虚拟机应符合[虚拟机先决条件](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。

## 步骤 1：登录到你的 Azure 帐户


1. 打开 PowerShell 控制台，然后运行以下命令以登录到 Azure 帐户。该 cmdlet 会打开一个网页，提示你输入帐户凭据。

    	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

	此外，也可以通过 `-Credential` 参数将帐户凭据作为参数包含在 `Login-AzureRmAccount` cmdlet 中。

	如果你是代表租户的 CSP 合作伙伴，则需使用 tenantID 或租户主域名将客户指定为一名租户。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud -Tenant "fabrikam.com"

2. 一个帐户可以有多个订阅，因此请将需要使用的订阅与帐户关联在一起。

    	Select-AzureRmSubscription -SubscriptionName $SubscriptionName

3.  使用以下命令验证订阅是否已注册，以便将 Azure 提供程序用于恢复服务和 Site Recovery：

	- `Get-AzureRmResourceProvider -ProviderNamespace  Microsoft.RecoveryServices`  

	-  `Get-AzureRmResourceProvider -ProviderNamespace  Microsoft.SiteRecovery`  


	如果在上述两个命令的输出中已将 **RegistrationState** 设置为 **Registered**，则可继续执行步骤 2。否则，你将需要注册订阅中缺失的提供程序。

	若要为站点恢复和恢复服务注册 Azure 提供程序，请运行以下命令：

    	Register-AzureRmResourceProvider -ProviderNamespace Microsoft.SiteRecovery
    	Register-AzureRmResourceProvider -ProviderNamespace Microsoft.RecoveryServices

	使用 `Get-AzureRmResourceProvider -ProviderNamespace  Microsoft.RecoveryServices` 和 `Get-AzureRmResourceProvider -ProviderNamespace  Microsoft.SiteRecovery` 命令验证提供程序是否已成功注册。



## 步骤 2：设置恢复服务保管库

1. 创建一个可在其中创建该保管库的 Azure Resource Manager 资源组，或者使用现有资源组。可以使用以下命令创建新的资源组：

		New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Geo  

	其中，$ResourceGroupName 变量包含需要创建的资源组的名称，$Geo 变量包含可在其中创建资源组的 Azure 区域（例如：“China East”）。

	可以使用 `Get-AzureRmResourceGroup` cmdlet 获取订阅中资源组的列表。

2. 创建如下所示的新的 Azure 恢复服务保管库：

		$vault = New-AzureRmRecoveryServicesVault -Name <string> -ResourceGroupName <string> -Location <string>

	可以使用 `Get-AzureRmRecoveryServicesVault` cmdlet 检索现有保管库的列表。

> [AZURE.NOTE] 如果希望在通过经典门户或 Azure 服务管理 PowerShell 模块创建的 Site Recovery 保管库上执行操作，可以使用 `Get-AzureRmSiteRecoveryVault` cmdlet 检索此类保管库的列表。你需要对所有新操作都创建新的恢复服务保管库。你此前创建的 Site Recovery 保管库将继续受到支持，但不会有最新功能。

## 步骤 3：设置恢复服务保管库上下文

1.  通过运行以下命令设置保管库上下文：

		Set-AzureRmSiteRecoveryVaultSettings -ARSVault $vault

## 步骤 4：创建 Hyper-V 站点，然后为该站点生成新的保管库注册密钥。

1. 创建新的 Hyper-V 站点，如下所示：

		$sitename = "MySite"                #Specify site friendly name
		New-AzureRmSiteRecoverySite -Name $sitename

	此 cmdlet 会启动一个创建该站点所需的站点恢复作业，然后返回一个站点恢复作业对象。等待作业完成，然后验证作业已成功完成。

	你可以检索作业对象，以便使用 Get-AzureRmSiteRecoveryJob cmdlet 查看当前的作业状态。
2. 生成和下载站点的注册密钥，如下所示：

		$SiteIdentifier = Get-AzureRmSiteRecoverySite -Name $sitename | Select -ExpandProperty SiteIdentifier
		Get-AzureRmRecoveryServicesVaultSettingsFile -Vault $vault -SiteIdentifier $SiteIdentifier -SiteFriendlyName $sitename -Path $Path

	将已下载的密钥复制到 Hyper-V 主机。你需要通过该密钥将 Hyper-V 主机注册到站点。

## 步骤 5：在 Hyper-V 主机上安装 Azure Site Recovery 提供程序和 Azure 恢复服务代理

1. 从[此处](https://aka.ms/downloaddra)下载最新版本的提供程序的安装程序

2. 在 Hyper-V 主机上运行安装程序，在安装结束时继续执行注册步骤。

3. 在系统提示时提供下载的站点注册密钥，然后完成注册过程，将 Hyper-V 主机注册到站点。

4. 使用以下命令验证 Hyper-V 主机是否已注册到站点：

		$server =  Get-AzureRmSiteRecoveryServer -FriendlyName $server-friendlyname

## 步骤 6：创建复制策略，然后将其与保护容器相关联

1. 创建复制策略，如下所示：

		$ReplicationFrequencyInSeconds = "300";    	#options are 30,300,900
		$PolicyName = “replicapolicy”
		$Recoverypoints = 6            		#specify the number of recovery points
		$storageaccountID = Get-AzureRmStorageAccount -Name "mystorea" -ResourceGroupName "MyRG" | Select -ExpandProperty Id

		$PolicyResult = New-AzureRmSiteRecoveryPolicy -Name $PolicyName -ReplicationProvider “HyperVReplicaAzure” -ReplicationFrequencyInSeconds $ReplicationFrequencyInSeconds  -RecoveryPoints $Recoverypoints -ApplicationConsistentSnapshotFrequencyInHours 1 -RecoveryAzureStorageAccountId $storageaccountID

	检查返回的作业，确保复制策略创建成功。

	>[AZURE.IMPORTANT] 指定的存储帐户应与恢复服务保管库处于同一 Azure 区域，并且应已启用异地复制。
	>
	> - 如果指定的恢复存储帐户的类型为 Azure 存储空间（经典），则在对受保护计算机进行故障转移时，会将该计算机恢复为 Azure IaaS（经典）。
	> - 如果指定的恢复存储帐户的类型为 Azure 存储空间 (Azure Resource Manager)，则在对受保护计算机进行故障转移时，会将该计算机恢复为 Azure IaaS (Azure Resource Manager)。

2. 获取对应于该站点的保护容器，如下所示：

		$protectionContainer = Get-AzureRmSiteRecoveryProtectionContainer
3.	开始将保护容器与复制策略相关联，如下所示：

		$Policy = Get-AzureRmSiteRecoveryPolicy -FriendlyName $PolicyName
		$associationJob  = Start-AzureRmSiteRecoveryPolicyAssociationJob -Policy $Policy -PrimaryProtectionContainer $protectionContainer

	等待关联作业完成，确保它已成功完成。

##步骤 7：为虚拟机启用保护

1. 获取与你要保护的 VM 相对应的保护实体，如下所示：

		$VMFriendlyName = "Fabrikam-app"                    #Name of the VM
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

2. 开始保护虚拟机，如下所示：

		$Ostype = "Windows"                                 # "Windows" or "Linux"
		$DRjob = Set-AzureRmSiteRecoveryProtectionEntity -ProtectionEntity $protectionEntity -Policy $Policy -Protection Enable -RecoveryAzureStorageAccountId $storageaccountID  -OS $OStype -OSDiskName $protectionEntity.Disks[0].Name

	>[AZURE.IMPORTANT] 指定的存储帐户应与恢复服务保管库处于同一 Azure 区域，并且应已启用异地复制。
	>
	> - 如果指定的恢复存储帐户的类型为 Azure 存储空间（经典），则在对受保护计算机进行故障转移时，会将该计算机恢复为 Azure IaaS（经典）。
	> - 如果指定的恢复存储帐户的类型为 Azure 存储空间 (Azure Resource Manager)，则在对受保护计算机进行故障转移时，会将该计算机恢复为 Azure IaaS (Azure Resource Manager)。

	> 如果要保护的 VM 有多个附加磁盘，则需使用 *OSDiskName* 参数指定操作系统磁盘。

3. 等待虚拟机在完成初始复制后进入受保护状态。这可能需要一段时间，具体取决于诸如要复制的数据量和 Azure 的可用上游带宽等因素。在 VM 进入受保护的状态后，将更新作业的状态和 StateDescription，如下所示。

		PS C:\> $DRjob = Get-AzureRmSiteRecoveryJob -Job $DRjob

		PS C:\> $DRjob | Select-Object -ExpandProperty State
		Succeeded

		PS C:\> $DRjob | Select-Object -ExpandProperty StateDescription
		Completed

4. 更新各种恢复属性，例如 VM 角色大小和进行故障转移时需要将虚拟机的网络接口卡连接到的 Azure 网络。

		PS C:\> $nw1 = Get-AzureRmVirtualNetwork -Name "FailoverNw" -ResourceGroupName "MyRG"

		PS C:\> $VMFriendlyName = "Fabrikam-App"

		PS C:\> $VM = Get-AzureRmSiteRecoveryVM -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

		PS C:\> $UpdateJob = Set-AzureRmSiteRecoveryVM -VirtualMachine $VM -PrimaryNic $VM.NicDetailsList[0].NicId -RecoveryNetworkId $nw1.Id -RecoveryNicSubnetName $nw1.Subnets[0].Name

		PS C:\> $UpdateJob = Get-AzureRmSiteRecoveryJob -Job $UpdateJob

		PS C:\> $UpdateJob


		Name             : b8a647e0-2cb9-40d1-84c4-d0169919e2c5
		ID               : /Subscriptions/a731825f-4bf2-4f81-a611-c331b272206e/resourceGroups/MyRG/providers/Microsoft.RecoveryServices/vault
		                   s/MyVault/replicationJobs/b8a647e0-2cb9-40d1-84c4-d0169919e2c5
		Type             : Microsoft.RecoveryServices/vaults/replicationJobs
		JobType          : UpdateVmProperties
		DisplayName      : Update the virtual machine
		ClientRequestId  : 805a22a3-be86-441c-9da8-f32685673112-2015-12-10 17:55:51Z-P
		State            : Succeeded
		StateDescription : Completed
		StartTime        : 10-12-2015 17:55:53 +00:00
		EndTime          : 10-12-2015 17:55:54 +00:00
		TargetObjectId   : 289682c6-c5e6-42dc-a1d2-5f9621f78ae6
		TargetObjectType : ProtectionEntity
		TargetObjectName : Fabrikam-App
		AllowedActions   : {Restart}
		Tasks            : {UpdateVmPropertiesTask}
		Errors           : {}



## 步骤 8：运行测试故障转移

1. 运行一个测试故障转移作业，如下所示：

		$nw = Get-AzureRmVirtualNetwork -Name "TestFailoverNw" -ResourceGroupName "MyRG" #Specify Azure vnet name and resource group

		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -FriendlyName $VMFriendlyName -ProtectionContainer $protectionContainer

		$TFjob = Start-AzureRmSiteRecoveryTestFailoverJob -ProtectionEntity $protectionEntity -Direction PrimaryToRecovery -AzureVMNetworkId $nw.Id

2. 验证是否在 Azure 中创建了测试 VM。（在 Azure 中创建测试 VM 之后，将暂停测试故障转移作业。在恢复该作业后，将清除已创建的项目，此时作业才算完成，如下一步骤所示。）

3. 完成测试故障转移，如下所示：

    	$TFjob = Resume-AzureRmSiteRecoveryJob -Job $TFjob


##后续步骤

[详细了解](https://msdn.microsoft.com/zh-cn/library/azure/mt637930.aspx) Azure Site Recovery 和 Azure Resource Manager PowerShell cmdlet。

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add Microsoft Hyper-V Server 2012 R2 as prerequisites option-->