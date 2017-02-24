<properties
    pageTitle="使用 PowerShell (Resource Manager) 将 VMM 云中的 Hyper-V 虚拟机复制到辅助 VMM 站点 | Azure"
    description="介绍如何部署 Azure Site Recovery，以便使用 PowerShell (Resource Manager) 来协调 VMM 云中 Hyper-V VM 到辅助 VMM 站点的复制、故障转移和恢复"
    services="site-recovery"
    documentationcenter=""
    author="sujaytalasila"
    manager="rochakm"
    editor="raynew" />  

<tags
    ms.assetid="9d38e9c3-217c-4e44-830c-575e9a4141f2"
    ms.service="site-recovery"
    ms.workload="backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/01/2016"
    wacn.date="02/24/2017"
    ms.author="sutalasi" />  


# 使用 PowerShell (Resource Manager) 将 VMM 云中的 Hyper-V 虚拟机复制到辅助 VMM 站点

> [AZURE.SELECTOR]
- [Azure 管理门户](/documentation/articles/site-recovery-vmm-to-vmm/)
- [经典门户](/documentation/articles/site-recovery-vmm-to-vmm-classic/)
- [PowerShell - Resource Manager](/documentation/articles/site-recovery-vmm-to-vmm-powershell-resource-manager/)

欢迎使用 Azure Site Recovery！ 如果你要将 System Center Virtual Machine Manager (VMM) 云中管理的本地 Hyper-V 虚拟机复制到辅助站点，请参考本文。

本文说明当你设置 Azure Site Recovery 以便将 System Center VMM 云中的 Hyper-V 虚拟机复制到辅助站点中的 System Center VMM 云时，如何使用 PowerShell 来自动完成所要执行的常见任务。

本文包含方案的先决条件，并演示

- 如何设置恢复服务保管库
- 在源 VMM 服务器和目标 VMM 服务器上安装 Azure Site Recovery 提供程序
- 在保管库中注册 VMM 服务器
- 为 VMM 云配置复制策略。策略中的所有复制设置将应用到所有受保护的虚拟机
- 为虚拟机启用保护。
- 单独测试或作为恢复计划的一部分测试 VM 的故障转移情况，确保一切正常。
- 单独执行或作为恢复计划的一部分执行 VM 的计划内或计划外故障转移，确保一切正常。

如果在设置本方案时遇到问题，请将你的问题发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。

> [AZURE.NOTE] Azure 提供两个不同的[部署模型](/documentation/articles/resource-manager-deployment-model/)来创建和处理资源：Azure Resource Manager 和经典。Azure 还有两个门户 — 支持经典部署模型的 Azure 经典门户，以及支持两种部署模型的 Azure 门户。本文介绍资源管理器部署模型。



## 本地先决条件
下面是在主要和辅助本地站点中部署此方案所要满足的条件：

**先决条件** | **详细信息** 
--- | ---
**VMM** | 我们建议在主站点和辅助站点中各部署一个 VMM 服务器。<br/><br/> 也可以[在单个 VMM 服务器上的云之间复制](/documentation/articles/site-recovery-single-vmm/)。为此，至少需要在 VMM 服务器上配置两个云。<br/><br/> VMM 服务器应当至少运行具有最新更新的 System Center 2012 SP1。<br/><br/> 每个 VMM 服务器上必须配置了一个或多个云，所有云中必须设置了 Hyper-V 容量配置文件。<br/><br/>云必须包含一个或多个 VMM 主机组。<br/><br/>在 [Configuring the VMM cloud fabric](/documentation/articles/site-recovery-prereq/)（配置 VMM 云结构）和 [Walkthrough: Creating private clouds with System Center 2012 SP1 VMM](http://blogs.technet.com/b/keithmayer/archive/2013/04/18/walkthrough-creating-private-clouds-with-system-center-2012-sp1-virtual-machine-manager-build-your-private-cloud-in-a-month.aspx)（演练：使用 System Center 2012 SP1 VMM 创建私有云）中详细了解如何设置 VMM 云。<br/><br/> VMM 服务器应具有 Internet 访问权限。 
**Hyper-V** | Hyper-V 服务器必须至少运行具有 Hyper-V 角色且安装了最新更新的 Windows Server 2012。<br/><br/> Hyper-V 服务器应包含一个或多个 VM。<br/><br/> Hyper-V 主机服务器应位于主要和辅助 VMM 云中的主机组内。<br/><br/> 如果正在 Windows Server 2012 R2 上的群集中运行 Hyper-V，则应安装[更新 2961977](https://support.microsoft.com/zh-cn/kb/2961977)<br/><br/>如果正在 Windows Server 2012 上的群集中运行 Hyper-V，请注意，如果使用基于静态 IP 地址的群集，则不会自动创建该群集代理。你需要手动配置群集代理。[了解详细信息](http://social.technet.microsoft.com/wiki/contents/articles/18792.configure-replica-broker-role-cluster-to-cluster-replication.aspx)。
**提供商** | 在站点恢复部署期间，需要在 VMM 服务器上安装 Azure Site Recovery 提供程序。提供程序通过 HTTPS 443 与站点恢复通信，以协调复制。数据复制是通过 LAN 或 VPN 连接在主要和辅助 Hyper-V 服务器之间发生的。<br/><br/> 在 VMM 服务器上运行的提供程序需有权访问以下 URL：*.hypervrecoverymanager.windowsazure.cn; *.accesscontrol.chinacloudapi.cn; *.backup.windowsazure.cn; *.blob.core.chinacloudapi.cn; *.store.core.chinacloudapi.cn.<br/><br/> 此外，还要允许从 VMM 服务器到 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)的防火墙通信，并允许 HTTPS (443) 协议。

### 网络映射先决条件
网络映射在主要和辅助 VMM 服务器上的 VMM VM 网络之间进行映射：

- 故障转移后，以最佳方式在辅助 Hyper-V 主机上放置副本 VM。
- 将副本 VM 连接到适当的 VM 网络。
- 如果不配置网络映射，则故障转移之后，副本 VM 将不会连接到任何网络。
- 如果想要在站点恢复部署期间设置网络映射，则需要执行以下操作：

	- 确保源 Hyper-V 主机服务器上的 VM 已连接到 VMM VM 网络。该网络应当该链接到与该云相关联的逻辑网络。
	- 验证用于恢复的辅助云中是否配置了相应的 VM 网络。该 VM 网络应该链接到与辅助云关联的逻辑网络。


在以下文章中详细了解如何配置 VMM 网络

- [如何在 VMM 中配置逻辑网络](https://technet.microsoft.com/zh-cn/library/jj721568.aspx)
- [如何在 VMM 中配置 VM 网络和网关](https://technet.microsoft.com/zh-cn/library/jj721575.aspx)

[详细了解](/documentation/articles/site-recovery-network-mapping/)网络映射的工作原理。

###PowerShell 必决条件
确保已将 Azure PowerShell 准备就绪。如果你已使用 PowerShell，则升级到 0.8.10 或更高版本。有关设置 PowerShell 的信息，请参阅 [Guide to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（Azure PowerShell 安装和配置指南）。安装并配置 PowerShell 后，可在[此处](https://msdn.microsoft.com/zh-cn/library/dn850420.aspx)查看该服务的所有可用 cmdlet。

若要了解可帮助你使用 cmdlet 的提示（如在 Azure PowerShell 中通常如何处理参数值、输入和输出），请参阅 [Azure Cmdlet 入门](https://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx)。

## 步骤 1：设置订阅 

1. 从 Azure powershell 登录到你的 Azure 帐户：使用以下 cmdlet
 
		$UserName = "<user@live.com>"
		$Password = "<password>"
		$SecurePassword = ConvertTo-SecureString -AsPlainText $Password -Force
		$Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $UserName, $SecurePassword
		Login-AzureRmAccount -EnvironmentName AzureChinaCloud #-Credential $Cred 
	

2. 获取你的订阅的列表。其中还会列出每个订阅的 subscriptionID。记下你希望在其中创建恢复服务保管库的订阅的订阅 ID

		Get-AzureRmSubscription 

3. 通过提及订阅 ID 设置要在其中创建恢复服务保管库的订阅

		Set-AzureRmContext –SubscriptionID <subscriptionId>


## 步骤 2：创建恢复服务保管库 

1. 如果还没有 Azure Resource Manager 资源组，则创建一个

		New-AzureRmResourceGroup -Name #ResourceGroupName -Location #location

2. 创建新的恢复服务保管库，并将所创建的 ASR 保管库对象保存在变量（后面将用到）中。你还可以使用 Get-AzureRMRecoveryServicesVault cmdlet 检索 ASR 保管库对象后期创建：-

		$vault = New-AzureRmRecoveryServicesVault -Name #vaultname -ResouceGroupName #ResourceGroupName -Location #location 

## 步骤 3：设置恢复服务保管库上下文

1.  如果已创建保管库，则运行以下命令来获取该保管库。

		$vault = Get-AzureRmRecoveryServicesVault -Name #vaultname

2.  通过运行以下命令设置保管库上下文。

		Set-AzureRmSiteRecoveryVaultSettings -ARSVault $vault



## 步骤 4：安装 Azure Site Recovery 提供者

1.	在 VMM 计算机上，通过运行以下命令创建一个目录：
	
		New-Item c:\ASR -type directory
		
2.	通过运行以下命令，使用下载的提供者提取文件
	
		pushd C:\ASR\
		.\AzureSiteRecoveryProvider.exe /x:. /q

	
3.	使用以下命令安装提供者：
	
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


## 步骤 5：创建和关联复制策略

1.	通过运行以下命令创建 Hyper-V 2012 R2 复制策略：

	
		$ReplicationFrequencyInSeconds = "300";    	#options are 30,300,900
		$PolicyName = “replicapolicy”
		$RepProvider = HyperVReplica2012R2
		$Recoverypoints = 24            		#specify the number of hours to retain recovery pints
		$AppConsistentSnapshotFrequency = 4 #specify the frequency (in hours) at which app consistent snapshots are taken
		$AuthMode = "Kerberos"  #options are "Kerberos" or "Certificate"
		$AuthPort = "8083"  #specify the port number that will be used for replication traffic on Hyper-V hosts
		$InitialRepMethod = "Online" #options are "Online" or "Offline"

		$policyresult = New-AzureRmSiteRecoveryPolicy -Name $policyname -ReplicationProvider $RepProvider -ReplicationFrequencyInSeconds $Replicationfrequencyinseconds -RecoveryPoints $recoverypoints -ApplicationConsistentSnapshotFrequencyInHours $AppConsistentSnapshotFrequency -Authentication $AuthMode -ReplicationPort $AuthPort -ReplicationMethod $InitialRepMethod 

	> [AZURE.NOTE] VMM 云包含的 Hyper-V 主机可能运行不同版本的 Windows Server（如 Hyper-V 先决条件中所示），但复制策略是特定于 OS 版本的。如果不同的主机运行在不同的操作系统版本上，则请为每类 OS 版本创建不同的复制策略。例如，如果你有 5 个主机运行在 Windows Servers 2012 上，3 个主机运行在 Windows Server 2012 R2 上，则请创建两种复制策略 – 一种策略用于一种类型的操作系统版本。

2.	通过运行以下命令获取主保护容器（主 VMM 云）和恢复保护容器（恢复 VMM 云）：
	
		$PrimaryCloud = "testprimarycloud"
		$primaryprotectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $PrimaryCloud;  

		$RecoveryCloud = "testrecoverycloud"
		$recoveryprotectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $RecoveryCloud;  
  
3.	使用该策略的友好名称检索在步骤 1 中创建的策略

		$policy = Get-AzureRmSiteRecoveryPolicy -FriendlyName $policyname

4.	开始将保护容器（VMM 云）与复制策略相关联：

		$associationJob  = Start-AzureRmSiteRecoveryPolicyAssociationJob -Policy     $Policy -PrimaryProtectionContainer $primaryprotectionContainer -RecoveryProtectionContainer $recoveryprotectionContainer

5.	等待策略关联作业完成。你可以使用以下 PowerShell 代码段来查看作业是否已完成。
   
		$job = Get-AzureRmSiteRecoveryJob -Job $associationJob
   		if($job -eq $null -or $job.StateDescription -ne "Completed")
   		 {
        	$isJobLeftForProcessing = $true;
    	}

 	在作业完成处理后，运行以下命令：

		if($isJobLeftForProcessing)
    	{
    	Start-Sleep -Seconds 60
    	}
        }While($isJobLeftForProcessing)


若要检查作业是否完成，请遵循[监视活动](#monitor)中的步骤。

## 步骤 5：配置网络映射

1. 第一条命令将获取当前 Azure Site Recovery 保管库的服务器。该命令将 Azure Site Recovery 服务器存储在 $Servers 数组变量中。

		$Servers = Get-AzureRmSiteRecoveryServer

2. 以下命令获取源 VMM 服务器和目标 VMM 服务器的站点恢复网络。

    	$PrimaryNetworks = Get-AzureRmSiteRecoveryNetwork -Server $Servers[0]

		$RecoveryNetworks = Get-AzureRmSiteRecoveryNetwork -Server $Servers[1]

	
	> [AZURE.NOTE] 在服务器数组中，源 VMM 服务器可以是一个，也可以是第二个。查看 VMM 服务器的名称，获取相应的网络


4. 最后一个 cmdlet 将在主网络与恢复网络之间创建映射。该 cmdlet 将主要网络指定为 $PrimaryNetworks 的第一个元素，将恢复网络指定为 $RecoveryNetworks 的第一个元素。

		New-AzureRmSiteRecoveryNetworkMapping -PrimaryNetwork $PrimaryNetworks[0] -RecoveryNetwork $RecoveryNetworks[0]

## 步骤 6：配置存储映射
1. 以下命令在 $storageclassifications 变量中获取存储分类的列表。
   
        $storageclassifications = Get-AzureRmSiteRecoveryStorageClassification
2. 以下命令在 $SourceClassificaion 变量中获取源分类，在 $TargetClassification 变量中获取目标分类。
   
        $SourceClassificaion = $storageclassifications[0]
   
        $TargetClassification = $storageclassifications[1]

    > [AZURE.NOTE] 源和目标分类可以是数组中的任何元素。请参考以下命令的输出，了解 $storageclassifications 数组中源和目标分类的索引。

    > Get-AzureRmSiteRecoveryStorageClassification | Select-Object -Property FriendlyName, Id | Format-Table


1. 以下 cmdlet 在源分类与目标分类之间创建映射。
   
        New-AzureRmSiteRecoveryStorageClassificationMapping -PrimaryStorageClassification $SourceClassificaion -RecoveryStorageClassification $TargetClassification

## 步骤 7：为虚拟机启用保护
在正确配置服务器、云和网络后，可以在云中为虚拟机启用保护。


  1. 若要启用保护，请运行以下命令以获取保护容器：
	
			$PrimaryProtectionContainer = Get-AzureRmSiteRecoveryProtectionContainer -friendlyName $PrimaryCloudName
	
  2. 通过运行以下命令获取保护实体 (VM)：
	
	 		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -friendlyName $VMName -ProtectionContainer $PrimaryProtectionContainer
		
  3. 通过运行以下命令为 VM 启用复制：

			$jobResult = Set-AzureRmSiteRecoveryProtectionEntity -ProtectionEntity $protectionentity -Protection Enable -Policy $policy


## 测试你的部署

若要测试你的部署，可针对一台虚拟机运行测试故障转移，或者创建一个包括多个虚拟机的恢复计划并针对该计划运行测试故障转移。测试故障转移在隔离的网络中模拟你的故障转移和恢复机制。

> [AZURE.NOTE] 你可以在 Azure 门户中为应用程序创建恢复计划。

若要检查作业是否完成，请遵循[监视活动](#monitor)中的步骤。


### 运行测试故障转移

1.	运行以下 cmdlet 所获取的 VM 网络是需要将 VM 通过测试性故障转移方式转移到其中的网络。

		$Servers = Get-AzureRmSiteRecoveryServer
		$RecoveryNetworks = Get-AzureRmSiteRecoveryNetwork -Server $Servers[1]

2.	执行以下操作，对 VM 进行测试性故障转移：
	
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -FriendlyName $VMName -ProtectionContainer $PrimaryprotectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryTestFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity -VMNetwork $RecoveryNetworks[1] 

2.	执行以下操作，对恢复计划进行测试性故障转移：
	
		$recoveryplanname = "test-recovery-plan"

		$recoveryplan = Get-AzureRmSiteRecoveryRecoveryPlan -FriendlyName $recoveryplanname

		$jobIDResult =  Start-AzureRmSiteRecoveryTestFailoverJob -Direction PrimaryToRecovery -Recoveryplan $recoveryplan -VMNetwork $RecoveryNetworks[1] 

### 运行计划的故障转移

1. 执行以下操作，对 VM 进行计划内故障转移：
	
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity

2. 执行以下操作，对恢复计划进行计划内故障转移：
	
		$recoveryplanname = "test-recovery-plan"

		$recoveryplan = Get-AzureRmSiteRecoveryRecoveryPlan -FriendlyName $recoveryplanname

		$jobIDResult =  Start-AzureRmSiteRecoveryPlannedFailoverJob -Direction PrimaryToRecovery -Recoveryplan $recoveryplan

### 运行非计划的故障转移

1. 执行以下操作，对 VM 进行计划外故障转移：
		
		$protectionEntity = Get-AzureRmSiteRecoveryProtectionEntity -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

		$jobIDResult =  Start-AzureRmSiteRecoveryUnPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity 

2\. 执行以下操作，对恢复计划进行计划外故障转移：
		
		$recoveryplanname = "test-recovery-plan"

		$recoveryplan = Get-AzureRmSiteRecoveryRecoveryPlan -FriendlyName $recoveryplanname

		$jobIDResult =  Start-AzureRmSiteRecoveryUnPlannedFailoverJob -Direction PrimaryToRecovery -ProtectionEntity $protectionEntity 
	
## <a name="monitor"></a>监视活动

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

<!---HONumber=Mooncake_1205_2016-->