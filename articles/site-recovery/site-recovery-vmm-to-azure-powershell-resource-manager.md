<properties
    pageTitle="使用 Azure Site Recovery 和 PowerShell（资源管理器）在 VMM 云中复制 Hyper-V 虚拟机 | Azure"
    description="使用 Azure Site Recovery 和 PowerShell 在 VMM 云中复制 Hyper-V 虚拟机"
    services="site-recovery"
    documentationcenter=""
    author="Rajani-Janaki-Ram"
    manager="rochakm"
    editor="raynew" />
<tags
    ms.assetid="6ac509ad-5024-43d8-b621-d8fec019b9a9"
    ms.service="site-recovery"
    ms.workload="backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/02/2017"
    wacn.date="03/10/2017"
    ms.author="rajanaki" />  


# 使用 PowerShell 和 Azure Resource Manager 将 VMM 云中的 Hyper-V 虚拟机复制到 Azure

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/site-recovery-vmm-to-azure/)
- [PowerShell - Resource Manager](/documentation/articles/site-recovery-vmm-to-azure-powershell-resource-manager/)
- [经典门户](/documentation/articles/site-recovery-vmm-to-azure-classic/)
- [PowerShell - 经典](/documentation/articles/site-recovery-deploy-with-powershell/)



## 概述

Azure Site Recovery 可在许多部署方案中安排虚拟机的复制、故障转移和恢复，为业务连续性和灾难恢复 (BCDR) 策略发挥作用。有关部署方案的完整列表，请参阅 [Azure Site Recovery 概述](/documentation/articles/site-recovery-overview/)。

本文说明当你设置 Azure Site Recovery 以便将 System Center VMM 云中的 Hyper-V 虚拟机复制到 Azure 存储空间时，如何使用 PowerShell 来自动完成所要执行的常见任务。

本文包含方案的先决条件，并演示

- 如何设置恢复服务保管库
- 在源 VMM 服务器上安装 Azure Site Recovery 提供程序
- 在保管库中注册服务器、添加 Azure 存储帐户
- 在 Hyper-V 主机服务器上安装 Azure 恢复服务代理
- 为 VMM 云配置将应用于所有受保护虚拟机的保护设置
- 为这些虚拟机启用保护。
- 测试故障转移以确保一切都正常工作。

如果在设置本方案时遇到问题，请将你的问题发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。

> [AZURE.NOTE] Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用资源管理器部署模型。

## 开始之前

确保已满足以下先决条件：

### Azure 先决条件

- 需要一个 [Azure](https://azure.cn/) 帐户。如果没有，请使用 [1rmb 试用版](/pricing/1rmb-trial/)。此外，可以了解 [Azure Site Recovery Manager 定价](/pricing/details/site-recovery/)。
- 若要复制到 CSP 订阅方案，需要一个 CSP 订阅。若要详细了解 CSP 计划，请参阅[如何注册 CSP 计划](https://msdn.microsoft.com/zh-cn/library/partnercenter/mt156995.aspx)。
- 你将需要一个 Azure v2 存储 (ARM) 帐户来存储复制到 Azure 的数据。需要为帐户启用地域复制。该帐户应位于 Azure Site Recovery 服务所在的同一区域，并与同一订阅或 CSP 订阅相关联。若要详细了解如何设置 Azure 存储，请参阅 [Azure 存储简介](/documentation/articles/storage-introduction/)。
- 需确保要保护的虚拟机符合 [Azure 虚拟机先决条件](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。

> [AZURE.NOTE] 目前只能通过 Powershell 执行 VM 级别的操作。很快将提供对恢复计划级别操作的支持。现在，你只能在“受保护的 VM”粒度执行故障转移，而不能在恢复计划级别执行。

### VMM 先决条件
- 你需要具有运行 System Center 2012 R2 的 VMM 服务器。
- 包含你要保护的虚拟机的任何 VMM 服务器必须运行有 Azure Site Recovery 提供程序。这是在部署 Azure Site Recovery 期间安装的。
- 在 VMM 服务器上需要至少有一个你要保护的云。云应当包含：
	- 一个或多个 VMM 主机组。
	- 每个主机组中有一个或多个 Hyper-V 主机服务器或群集。
	- 源 Hyper-V 服务器上有一个或多个虚拟机。
- 了解有关设置 VMM 云的更多信息：
	- 在 [System Center 2012 R2 VMM 中私有云的新增功能](http://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/MDC-B357)及 [VMM 2012 和云](http://www.server-log.com/blog/2011/8/26/vmm-2012-and-the-clouds.html)中阅读有关私有 VMM 云的详细信息。
	- 了解有关[配置 VMM 云结构](/documentation/articles/site-recovery-best-practices/)的更多信息
	- 在你的云结构元素就位后，通过[在 VMM 中创建私有云](https://technet.microsoft.com/zh-cn/library/jj860425.aspx)和[Walkthrough: Creating private clouds with System Center 2012 SP1 VMM（演练：使用 System Center 2012 SP1 VMM 创建私有云）](https://blogs.technet.microsoft.com/b/keithmayer/archive/2013/04/18/walkthrough-creating-private-clouds-with-system-center-2012-sp1-virtual-machine-manager-build-your-private-cloud-in-a-month.aspx)了解如何创建私有云。

### Hyper-V 先决条件

- Hyper-V 主机服务器必须至少运行具有 Hyper-V 角色的 **Windows Server 2012** 或 **Microsoft Hyper-V Server 2012** 并且安装了最新的更新。
- 如果在群集中运行 Hyper-V，请注意，如果具有基于静态 IP 地址的群集，则不会自动创建群集代理。你需要手动配置群集代理。对于
- 有关说明，请参阅 [How to Configure Hyper-V Replica Broker（如何配置 Hyper-V 副本代理）](http://blogs.technet.com/b/haroldwong/archive/2013/03/27/server-virtualization-series-hyper-v-replica-broker-explained-part-15-of-20-by-yung-chou.aspx)。
- 你要为其管理保护的任何 Hyper-V 主机服务器或群集必须包括在 VMM 云中。

### 网络映射先决条件
当在 Azure 中保护虚拟机时，网络映射会在源 VMM 服务器上的虚拟机网络与目标 Azure 网络之间进行映射以实现以下功能：

- 在同一网络上进行故障转移的所有计算机都可以彼此连接到对方，不管它们位于哪个恢复计划中。
- 如果在目标 Azure 网络上设置了网络网关，则虚拟机可以连接到其他本地虚拟机。
- 如果没有配置网络映射，则只有在同一恢复计划中进行故障转移的虚拟机能够在故障转移到 Azure 后彼此连接到对方。

如果希望部署网络映射，需要满足下列条件：

- 源 VMM 服务器上要保护的虚拟机应当连接到某个 VM 网络。该网络应当该链接到与该云相关联的逻辑网络。
- 在故障转移后复制的虚拟机可以连接到的 Azure 网络。你将在故障转移时选择此网络。此网络应与 Azure Site Recovery 订阅位于同一区域中。

若要详细了解网络映射，请参阅：

- [如何在 VMM 中配置逻辑网络](https://technet.microsoft.com/zh-cn/library/jj721568.aspx)
- [如何在 VMM 中配置 VM 网络和网关](https://technet.microsoft.com/zh-cn/library/jj721575.aspx)
- [如何在 Azure 中配置和监视虚拟网络](home/features/virtual-network)


###PowerShell 必决条件
确保已将 Azure PowerShell 准备就绪。如果你已使用 PowerShell，则升级到 0.8.10 或更高版本。有关设置 PowerShell 的信息，请参阅 [Guide to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（Azure PowerShell 安装和配置指南）。安装并配置 PowerShell 后，可在[此处](https://msdn.microsoft.com/zh-cn/library/dn850420.aspx)查看该服务的所有可用 cmdlet。

若要了解可帮助你使用 cmdlet 的提示（如在 Azure PowerShell 中通常如何处理参数值、输入和输出），请参阅 [Azure Cmdlet 入门](https://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx)。

## 步骤 1：设置订阅 

1. 从 Azure powershell 登录到你的 Azure 帐户：使用以下 cmdlet
 
		$UserName = "<user@live.com>"
		$Password = "<password>"
		$SecurePassword = ConvertTo-SecureString -AsPlainText $Password -Force
		$Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $UserName, $SecurePassword
		Login-AzureRmAccount -EnvironmentName ChinaAzureCloud #-Credential $Cred 
	

2. 获取你的订阅的列表。其中还会列出每个订阅的 subscriptionID。记下你希望在其中创建恢复服务保管库的订阅的订阅 ID

		Get-AzureRmSubscription 

3. 通过提及订阅 ID 设置要在其中创建恢复服务保管库的订阅

		Set-AzureRmContext –SubscriptionID <subscriptionId>

## 步骤 2：创建恢复服务保管库
1. 如果没有 Azure Resource Manager 资源组，请创建一个资源组
   
        New-AzureRmResourceGroup -Name #ResourceGroupName -Location #location
2. 创建新的恢复服务保管库，并将所创建的 ASR 保管库对象保存在变量（后面将用到）中。你还可以使用 Get-AzureRMRecoveryServicesVault cmdlet 检索 ASR 保管库对象后期创建：-

		$vault = New-AzureRmRecoveryServicesVault -Name #vaultname -ResouceGroupName #ResourceGroupName -Location #location 

## 步骤 3：设置恢复服务保管库上下文

通过运行以下命令设置保管库上下文。
   
       Set-AzureRmSiteRecoveryVaultSettings -ARSVault $vault

## 步骤 4：安装 Azure Site Recovery 提供者

1.	在 VMM 计算机上，通过运行以下命令创建一个目录：
	
		New-Item c:\ASR -type directory
		
2.	通过运行以下命令，使用下载的提供程序提取文件
	
		pushd C:\ASR\
		.\AzureSiteRecoveryProvider.exe /x:. /q

	
3.	使用以下命令安装提供程序：
	
		.\SetupDr.exe /i
		$installationRegPath = "hklm:\software\Microsoft\Microsoft System Center Virtual Machine Manager Server\DRAdapter"
		do
		{
		                $isNotInstalled = $true;
		                if(Test-Path $installationRegPath)
		                {
		                                $isNotInstalled = $false;
		                }
		}While($isNotInstalled)

    等待安装完成。
	
4.	使用以下命令在保管库中注册服务器：
	
		$BinPath = $env:SystemDrive+"\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin"
		pushd $BinPath
		$encryptionFilePath = "C:\temp".\DRConfigurator.exe /r /Credentials $VaultSettingFilePath /vmmfriendlyname $env:COMPUTERNAME /dataencryptionenabled $encryptionFilePath /startvmmservice

	
## 步骤 5：创建 Azure 存储帐户

1. 如果你没有 Azure 存储帐户，请运行以下命令，在与保管库相同的地区创建一个启用异地复制的帐户：

		$StorageAccountName = "teststorageacc1"	#StorageAccountname
		$StorageAccountGeo  = "China East" 	
		$ResourceGroupName =  “myRG” 			#ResourceGroupName 
		$RecoveryStorageAccount = New-AzureRmStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -Type “StandardGRS” -Location $StorageAccountGeo

请注意，存储帐户必须位于 Azure Site Recovery 服务所在的同一区域，并与同一订阅相关联。

## 步骤 6：安装 Azure 恢复服务代理

1. 从 [http:/aka.ms/latestmarsagent](http:/aka.ms/latestmarsagent "http:/aka.ms/latestmarsagent") 下载 Azure 恢复服务代理，并将其安装在 VMM 云中要保护的每个 Hyper-V 主机服务器上。

2. 在所有 VMM 主机上运行以下命令：

	marsagentinstaller.exe /q /nu

## 步骤 7：配置云保护设置

1.	通过运行以下命令在 Azure 中创建复制策略：

	
		$ReplicationFrequencyInSeconds = "300";    	#options are 30,300,900
		$PolicyName = “replicapolicy”
		$Recoverypoints = 6            		#specify the number of recovery points 

		$policryresult = New-AzureRmSiteRecoveryPolicy -Name $policyname -ReplicationProvider HyperVReplicaAzure -ReplicationFrequencyInSeconds $replicationfrequencyinseconds -RecoveryPoints $recoverypoints -ApplicationConsistentSnapshotFrequencyInHours 1 -RecoveryAzureStorageAccountId "/subscriptions/q1345667/resourceGroups/test/providers/Microsoft.Storage/storageAccounts/teststorageacc1"

2.	通过运行以下命令获取保护容器：
	
		$PrimaryCloud = "testcloud"
		$protectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $PrimaryCloud;  
  
3.	通过使用已创建的作业并提及友好策略名称，来获取策略详细信息并将其存储在变量中：

		$policy = Get-AzureRmSiteRecoveryPolicy -FriendlyName $policyname

4.	开始将保护容器与复制策略相关联：

		$associationJob  = Start-AzureRmSiteRecoveryPolicyAssociationJob -Policy     $Policy -PrimaryProtectionContainer $protectionContainer  

5.	在作业完成后，运行以下命令：
   
		$job = Get-AzureRmSiteRecoveryJob -Job $associationJob
   		if($job -eq $null -or $job.StateDescription -ne "Completed")
   		 {
        $isJobLeftForProcessing = $true;
    	}

6.	在作业完成处理后，运行以下命令：

		if($isJobLeftForProcessing)
    	{
    	Start-Sleep -Seconds 60
    	}
        }While($isJobLeftForProcessing)

若要检查作业是否完成，请遵循[监视活动](#monitor)中的步骤。

## 步骤 8：配置网络映射

在开始网络映射之前，请验证源 VMM 服务器上的虚拟机是否已连接到一个 VM 网络。此外，请创建一个或多个 Azure 虚拟机。

在 [Create a virtual network with a site-to-site VPN connection using Azure Resource Manager and PowerShell](/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/)（使用 Azure Resource Manager 和 PowerShell 创建具有站点到站点 VPN 连接的虚拟网络）中，了解更多有关如何使用 Azure Resource Manager 和 PowerShell 创建虚拟网络的信息。

请注意，可以将多个虚拟机网络映射到单个 Azure 网络。如果目标网络具有多个子网，并且其中一个子网与源虚拟机所在的子网同名，则在故障转移后副本虚拟机将连接到该目标子网。如果没有具有匹配名称的目标子网，则虚拟机将连接到网络中的第一个子网。

1. 第一条命令将获取当前 Azure Site Recovery 保管库的服务器。该命令将 Azure Site Recovery 服务器存储在 $Servers 数组变量中。

		$Servers = Get-AzureRmSiteRecoveryServer

2. 第二条命令将获取 $Servers 数组中第一个服务器的站点恢复网络。该命令在 $Networks 变量中存储网络。


    	$Networks = Get-AzureRmSiteRecoveryNetwork -Server $Servers[0]

3. 第三条命令获取 Azure 虚拟网络，然后将该值存储在 $AzureVmNetworks 变量中。
   
		$AzureVmNetworks =  Get-AzureRmVirtualNetwork

4. 最后一个 cmdlet 将在主网络与 Azure 虚拟机网络之间创建映射。该 cmdlet 将主网络指定为 $Networks 的第一个元素。该 cmdlet 将虚拟机网络指定为 $AzureVmNetworks 的第一个元素。

		New-AzureRmSiteRecoveryNetworkMapping -PrimaryNetwork $Networks[0] -AzureVMNetworkId $AzureVmNetworks[0]


## 步骤 9：为虚拟机启用保护

在正确配置服务器、云和网络后，可以在云中为虚拟机启用保护。

 注意以下事项：

 - 虚拟机必须满足 Azure 要求。可以在规划指南中的[先决条件和支持](/documentation/articles/site-recovery-best-practices/)中查看这些要求。

 - 若要启用保护，必须为虚拟机设置操作系统和操作系统磁盘属性。当你使用虚拟机模板在 VMM 中创建虚拟机时，可以设置属性。也可以在虚拟机属性的“常规”和“硬件配置”选项卡中为现有虚拟机设置这些属性。如果未在 VMM 中设置这些属性，可以在 Azure Site Recovery 门户中配置它们。


  1. 若要启用保护，请运行以下命令以获取保护容器：
	
			$ProtectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $CloudName
	
  2. 通过运行以下命令获取保护实体 (VM)：
	
	 		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -friendlyName $VMName -ProtectionContainer $protectionContainer
		
  3. 通过运行以下命令为 VM 启用 DR：

			$jobResult = Set-AzureRmSiteRecoveryProtectionEntity -ProtectionEntity $protectionentity -Protection Enable –Force -Policy $policy -RecoveryAzureStorageAccountId  $storageID "/subscriptions/217653172865hcvkchgvd/resourceGroups/rajanirgps/providers/Microsoft.Storage/storageAccounts/teststorageacc1



## 测试你的部署

若要测试你的部署，可针对一个虚拟机运行测试故障转移，或者创建一个包括多个虚拟机的恢复计划并针对该计划运行测试故障转移。测试故障转移在隔离的网络中模拟你的故障转移和恢复机制。请注意：

- 如果想要在故障转移之后使用远程桌面连接到 Azure 中的虚拟机，请在虚拟机上启用远程桌面连接，然后运行测试故障转移。
- 故障转移后，你将要使用公共 IP 地址通过远程桌面连接到 Azure 中的虚拟机。如果要执行此操作，请确保没有任何域策略阻止你使用公共地址连接到虚拟机。

若要检查作业是否完成，请遵循[监视活动](#monitor)中的步骤。


### 运行测试故障转移

1.	通过运行以下命令来启动测试故障转移：
	
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $protectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryTestFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity -AzureVMNetworkId <string>  

### 运行计划的故障转移

1. 通过运行以下命令来启动计划的故障转移：
	
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $protectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity -AzureVMNetworkId <string>  

### 运行非计划的故障转移

1. 通过运行以下命令来启动非计划的故障转移：
		
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $protectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryUnPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity -AzureVMNetworkId <string>  

	
## <a name=monitor></a>监视活动

使用以下命令来监视活动。请注意，必须在执行不同的作业之前等待处理完成。

	Do
	{
        $job = Get-AzureSiteRecoveryJob -Id $associationJob.JobId;
        Write-Host "Job State:{0}, StateDescription:{1}" -f Job.State, $job.StateDescription;
        if($job -eq $null -or $job.StateDescription -ne "Completed")
        {
        	$isJobLeftForProcessing = $true;
        }

	if($isJobLeftForProcessing)
        {
        	Start-Sleep -Seconds 60
        }
	}While($isJobLeftForProcessing)



## 后续步骤

[详细了解](https://msdn.microsoft.com/zh-cn/library/azure/mt637930.aspx) Azure Site Recovery 和 Azure Resource Manager PowerShell cmdlet。

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add Microsoft Hyper-V Server 2012 as prerequisites option-->