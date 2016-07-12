<properties
	pageTitle="使用 PowerShell 和 Azure 资源管理器从一个 VMM 站点到另一个 VMM 站点保护虚拟机"
	description="使用 PowerShell 和 Azure 资源管理器在两个本地 VMM 站点和 Azure 之间实现自动保护。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="site-recovery"
	ms.date="08/26/2015"
	wacn.date="12/15/2015"/>
	

#  使用 PowerShell 和 Azure 资源管理器实现 VMM 站点到 VMM 站点的保护


## 概述

Azure Site Recovery 可在许多部署方案中安排虚拟机的复制、故障转移和恢复，为业务连续性和灾难恢复 (BCDR) 策略发挥作用。有关部署方案的完整列表，请参阅 [Azure Site Recovery 概述](/documentation/articles/site-recovery-overview/)。

Azure PowerShell 是一个模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。它可与两种类型的模块配合工作：Azure 配置文件模块或 Azure 资源管理器 (ARM) 模块。

本文介绍如何使用 Windows PowerShell® 及 ARM 来部署 Azure Site Recovery 以配置和安排两个 VMM 站点之间的虚拟机保护。位于主站点中的 VMM 私有云中的 Hyper-V 宿主服务器上运行的虚拟机会使用 Hyper-V 副本进行复制并向辅助 VMM 站点进行故障转移。

你无需是一名 PowerShell 专家就可以使用本文章，但是本文假定你理解诸如模块、cmdlet 和会话等基本概念。有关 Windows PowerShell 的更多信息，请参阅 [Windows PowerShell 入门](http://technet.microsoft.com/library/hh857337.aspx)。
- 了解有关[将 Azure PowerShell 和 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)的更多信息。

本文包括了方案的先决条件并展示了如何设置 Azure PowerShell 以与 Site Recovery 配合使用、创建 Site Recovery 保管库、在 VMM 服务器上安装 Azure Site Recovery 提供程序及在保管库中注册服务器、为 VMM 云配置将应用于所有受保护虚拟机的保护设置，然后为这些虚拟机启用保护。最后将测试故障转移以确保一切都正常工作。


## 开始之前

确保已满足以下先决条件：

- 需要一个 [Azure](/) 帐户。你可以从[免费试用版](/pricing/free-trial/)开始。此外，你可以阅读 [Azure 站点恢复管理器定价](/pricing/details/site-recovery/)。
- 你需要在运行于 System Center 2012 R2 的主站点和辅助站点中有一个 VMM 服务器。
- 每个 VMM 服务器应至少有一个包含以下各项的云：
	- 一个或多个 VMM 主机组。
	- 每个主机组中有一个或多个 Hyper-V 主机服务器或群集。
	- 源 Hyper-V 服务器上有一个或多个虚拟机。
	- 了解有关设置 VMM 云的更多信息：

		- [System Center 2012 R2 VMM 中私有云的新增功能](https://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/MDC-B357#fbid=dfgvHAmYryA)及 [VMM 2012 和云](http://www.server-log.com/blog/2011/8/26/vmm-2012-and-the-clouds.html)。
		- [配置 VMM 云结构](/documentation/articles/site-recovery-best-practices/)
		- [在 VMM 中创建私有云](https://technet.microsoft.com/zh-cn/library/jj860425.aspx)和[演练：使用 System Center 2012 SP1 VMM 创建私有云](http://blogs.technet.com/b/keithmayer/archive/2013/04/18/walkthrough-creating-private-clouds-with-system-center-2012-sp1-virtual-machine-manager-build-your-private-cloud-in-a-month.aspx)
- 至少运行 Windows Server 2012 的一个或多个 Hyper-V 服务器，具有 Hyper-V 角色并且安装了最新更新。服务器或群集必须包含在一个 VMM 云中。
- 如果你在群集中运行 Hyper-V，请注意，如果你具有基于静态 IP 地址的群集，则不会自动创建群集代理。你需要手动配置群集代理。有关说明，请参阅[配置 Hyper-V 副本代理](http://social.technet.microsoft.com/wiki/contents/articles/18792.configure-replica-broker-role-cluster-to-cluster-replication.aspx)。

	- 你将需要 Azure PowerShell。确保你正在运行 Azure PowerShell 0.9.6 版或更高版本。阅读[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。 
	- 安装 Azure PowerShell 会安装一个自定义控制台。你可以从这个控制台或从任何其他主机程序，例如 Windows PowerShell ISE 运行 PowerShell 命令。

## 步骤 1：设置 PowerShell


1. 打开 PowerShell 控制台，然后运行以下命令以切换到 ARM 模块：

    `Switch-AzureMode AzureResourceManager`

3. 现在，运行以下命令以将你的 Azure 帐户添加到 PowerShell 会话。该 cmdlet 提供你输入你的帐户的登录凭据。

    `Add-AzureAccount -Environment AzureChinaCloud`

	请注意，如果你是代表租户的 CSP 合作伙伴，则在添加 Azure 帐户时，你需要将客户指定为一名租户：

    `Add-AzureAccount -Environment AzureChinaCloud -Tenant "customer"`

5. 一个帐户可以有多个订阅，因此你需要将你要使用的订阅与帐户关联在一起。

    `Select-AzureSubscription -SubscriptionName $SubscriptionName`

6. 如果你在订阅中首次使用 Site Recovery cmdlet，则你需要为 Site Recovery 注册 Azure 提供程序：

    `Register-AzureProvider -ProviderNamespace Microsoft.SiteRecovery`

## 步骤 2：设置 Site Recovery 保管库

1. 如果你目前还没有 Site Recovery 保管库，则需要通过运行 [New-AzureSiteRecoveryVault](https://msdn.microsoft.com/zh-cn/library/azure/dn954225.aspx) cmdlet 来创建一个保管库：

    `New-AzureSiteRecoveryVault -Location $VaultGeo -Name $VaultName;
    $vault = Get-AzureSiteRecoveryVault -Name $VaultName;`

## 步骤 3：生成保管库注册密钥

1. 运行 [Get-AzureSiteRecoveryVaultSettingsFile](https://msdn.microsoft.com/zh-cn/library/azure/dn850404.aspx) cmdlet 以获得一个保管库注册密钥。你需要用此密钥在保管库中注册 VMM 服务器：

    `$VaultSettingsFile = Get-AzureSiteRecoveryVaultSettingsFile -Location $VaultGeo -Name $VaultName -Path $OutputPathForSettingsFile;`

## 步骤 4：安装 Azure Site Recovery 提供者

1. 从 Site Recovery 保管库的快速启动页下载提供程序安装文件。
2. 使用以下命令在每个 VMM 服务器上创建一个文件夹：

    `pushd C:\ASR`

3. 运行以下命令以从下载的提供程序中抽取文件：

    `AzureSiteRecoveryProvider.exe /x:. /q`

4. 通过运行以下命令来安装提供程序：

    `.\SetupDr.exe /i` `$installationRegPath = "hklm:\software\Microsoft\Microsoft System Center Virtual Machine Manager Server\DRAdapter"
	do
	{
                $isNotInstalled = $true;
                if(Test-Path $installationRegPath)
                {
                                $isNotInstalled = $false;
                }
	}While($isNotInstalled)`

5. 等待安装完成，然后在保管库中注册服务器：

    `$BinPath = $env:SystemDrive+"\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin"
	pushd $BinPath
	$encryptionFilePath = "C:\temp"
	.\DRConfigurator.exe /r /Credentials $VaultSettingFilePath /vmmfriendlyname $env:COMPUTERNAME /dataencryptionenabled $encryptionFilePath /startvmmservice`

## 步骤 5：设置保管库上下文

1. 运行 Get-AzureSiteRecoveryVault cmdlet 以确保所有命令在特定保管库下运行：

    `$Vault = Get-AzureSiteRecoveryVault -ResouceGroupName $ResourceGroupName | where { $_.Name -eq $VaultName}`

2. 下载保管库设置：

    `$VaultSettingsFile = Get-AzureSiteRecoveryVaultSettingsFile -Vault $Vault -Path $OutputPathForSettingsFile`

3. 若要确保 cmdlet 在保管库中运行，请运行：

    `Import-AzureSiteRecoveryVaultSettingsFile -Path $VaultSetingsFile.FilePath`

## 步骤 3：配置云保护设置

在保管库中注册 VMM 服务器以后，你可以配置云保护设置，这些设置将要应用到位于已注册 VMM 服务器中云内的 Hyper-V 主机上的所有虚拟机。

1. 为主云和辅助云在保管库中分别创建一个容器：

    `$PrimaryContainer = Get-AzureSiteRecoveryProtectionContainer -FriendlyName  $PrimaryCloudName` `$$RecoveryContainer = Get-AzureSiteRecoveryProtectionContainer -FriendlyName  $RecoveryCloudName`

2. 配置要应用到云的保护设置：

    `New-AzureSiteRecoveryProtectionProfile -Name $ProtectionProfileName -ReplicationProvider HyperVReplica -ReplicationMethod Online -ReplicationFrequencyInSeconds 30 -RecoveryPoints 1 -ApplicationConsistentSnapshotFrequencyInHours 0 -ReplicationPort 8083 -Authentication Kerberos -AllowReplicaDeletion`

3. 启动一个作业以将云容器与云保护设置关联在一起：

    `Start-AzureSiteRecoveryProtectionProfileAssociationJob -ProtectionProfile $ProtectionProfile -PrimaryProtectionContainer $PrimaryContainer -RecoveryProtectionContainer $RecoveryContainer`


## 步骤 4：启用虚拟机保护

为 VMM 云中的虚拟机启用保护：

1. 获取你要保护的虚拟机：

    `$VM = Get-AzureSiteRecoveryProtectionEntity -ProtectionContainer $PrimaryContainer -FriendlyName $VMName `

2. 为虚拟机启用保护：

    `Set-AzureSiteRecoveryProtectionEntity -ProtectionEntity $VM -Protection Enable`


## 步骤 5：运行测试故障转移

1.	选择要进行故障转移的虚拟机：

    `$VM = Get-AzureSiteRecoveryProtectionEntity -ProtectionContainer $PrimaryContainer -FriendlyName  $VMName`

2. 运行一个测试故障转移作业：

    `$ currentJob = Start-AzureSiteRecoveryTestFailoverJob -ProtectionEntity $VM -Direction PrimaryToRecovery`

3. 检查进行了故障转移的虚拟机出现在辅助站点中并且完成故障转移：

    `Resume-AzureSiteRecoveryJob -Id $currentJob.Name`


## 后续步骤

如有关于此方案的问题和评论，请访问 [Site Recovery 论坛](https://social.msdn.microsoft.com/forums/zh-cn/home?forum=hypervrecovmgr/)

<!---HONumber=74-->