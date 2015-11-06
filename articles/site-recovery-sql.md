<properties 
	pageTitle="使用 SQL Server 和 Azure Site Recovery 实现灾难恢复" 
	description="Azure Site Recovery 可以协调 SQL Server 到辅助本地站点或 Azure 的复制、故障转移和恢复。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="rayne-wiselman" 
	manager="jwhit" 
	editor="tysonn"/>

<tags 
	ms.service="site-recovery"  
	ms.date="06/03/2015" 
	wacn.date="10/03/2015"/>


# 使用 SQL Server 和 Azure 站点恢复实现灾难恢复 

站点恢复是有助于实现业务连续性和灾难恢复 (BCDR) 策略的 Azure 服务，因为它可以协调复制、故障转移和恢复虚拟机和物理服务器。站点恢复支持多种复制机制，能够以一致的方式保护、复制计算机以及将其故障转移到 Azure 或辅助数据中心。在 [Azure 站点恢复概述](/documentation/articles/site-recovery-overview)中获取所有部署方案的概述。

 本文介绍如何结合使用 SQL Server BCDR 技术和站点恢复来保护应用程序的 SQL Server 后端。您应该先充分了解 SQL Server BCDR 功能（故障转移群集、AlwaysOn 可用性组、数据库镜像和日志传送）与站点恢复，然后再部署本文中所述的方案。



## 概述

许多工作负载都使用 SQL Server 作为基础。SharePoint、Dynamics 和 SAP 等应用程序使用 SQL Server 实现数据服务。SQL Server 可用性组等 SQL Server 高可用性和灾难恢复功能可以保护和恢复 SQL Server 数据库。

站点恢复可以保护作为 Hyper-V 虚拟机、VMware 虚拟机或物理服务器运行的 SQL Server。

<table border="1" cellspacing="4" cellpadding="4">
    <tbody>
    <tr align="left" valign="top">
		<td colspan = "2"><b>Hyper-V</b></td>
		<td colspan = "2"><b>VMware</b></td>
		<td colspan = "2"><b>物理服务器</b></td>
    </tr>
    <tr align="left" valign="top">
		<td>本地到本地</td>
		<td>本地到 Azure</td>
		<td>本地到本地</td>
		<td>本地到 Azure</td>
		<td>本地到本地</td>
		<td>本地到 Azure</td>
    </tr>
	<tr align="left" valign="top">
		<td>是</td>
		<td>是</td>
		<td>是</td>
		<td>即将支持</td>
		<td>是</td>
		<td>即将支持</td>
    </tr>
    </tbody>
    </table>

## 支持和集成

站点恢复可与表中汇总的本机 SQL Server BCDR 技术集成，以提供灾难恢复解决方案。

<table border="1" cellspacing="4" cellpadding="4">
    <tbody>
    <tr align="left" valign="top">
		<td><b>功能</b></td>
		<td><b>详细信息</b></td>
		<td><b>SQL Server 版本</b></td>
    </tr>
    </tr><tr align="left" valign="top">
		<td><b>AlwaysOn 可用性组</b></td>
		<td><p>SQL Server 的多个独立实例，每个实例在包含多个节点的故障转移群集中运行。</p> <p>数据库可以分组到可在 SQL Server 实例上复制（镜像）的故障转移组，因此不需要任何共享存储。</p> <p>在主站点与一个或多个辅助站点之间提供灾难恢复。使用同步复制与自动故障转移在可用性组中配置 SQL Server 数据库时，可以在不共享任何内容的群集中设置两个节点。</p></td>
		<td>SQL Server 2014/2012 Enterprise 版本</td>
    </tr>
	<tr align="left" valign="top">
		<td><b>故障转移群集 (AlwaysOn FCI)</b></td>
		<td><p>SQL Server 利用 Windows 故障转移群集实现本地 SQL Server 工作负载的高可用性。</p><p>使用共享磁盘运行 SQL Server 实例的节点是在故障转移群集中配置的。如果实例关闭，群集将故障转移到另一个节点。</p> <p>群集无法防止共享存储的故障或中断。共享磁盘可以使用 iSCSI、光纤通道或共享 VHDX 来实现。</p></td>
		<td><p>SQL Server Enterprise 版本</p> <p>SQL Server Standard 版本（仅限两个节点）</p></td>
	<tr align="left" valign="top">
		<td><b>高安全性模式下的数据库镜像</b></td>
		<td>在单个辅助副本中保护单个数据库。提供高安全性（同步）和高性能（异步）复制模式。不需要故障转移群集。</td>
		<td><p>SQL Server 2008 R2</p><p>SQL Server Enterprise 的所有版本</p></td>
    </tr>
    </tr>
	<tr align="left" valign="top">
		<td><b>独立 SQL Server</b></td>
		<td>SQL Server 和数据库托管在单个服务器（物理或虚拟）上。如果是虚拟服务器，则主机群集用于高可用性。没有来宾级别的高可用性。</td>
		<td>Enterprise 或 Standard 版本</td>
 
    </tbody>
    </table>

下表汇总了有关将 SQL Server BCDR 技术集成到站点恢复部署的建议。

<table border="1" cellspacing="4" cellpadding="4">
    <tbody>
    <tr align="left" valign="top">
		<td><b>版本</b></td>
		<td><b>细分版本</b></td>
		<td><b>部署</b></td>
		<td><b>本地到本地</b></td>
		<td><b>本地到 Azure</b>&lt;/td
    </tr>
    <tr align="left" valign="top">
		<td rowspan = "3">SQL Server 2014 或 2012</td>
		<td rowspan = "2">Enterprise</td>
		<td>故障转移群集实例</td>
		<td>AlwaysOn 可用性组</td>
		<td>AlwaysOn 可用性组</td>
    </tr>
	<tr align="left" valign="top">
		<td>实现高可用性的 AlwaysOn 可用性组</td>
		<td>AlwaysOn 可用性组</td>
		<td>AlwaysOn 可用性组</td>
    </tr>
	<tr align="left" valign="top">
		<td>标准</td>
		<td>故障转移群集实例</td>
		<td>使用本地镜像进行站点恢复复制</td>
		<td>使用本地镜像进行站点恢复复制</td>
    </tr>
	<tr align="left" valign="top">
		<td>Enterprise 或 Standard</td>
		<td>独立</td>
		<td>使用本地镜像进行站点恢复复制</td>
		<td>使用本地镜像进行站点恢复复制</td>
    </tr>
	<tr align="left" valign="top">
		<td>SQL Server 2008 R2</td><td>Enterprise 或 Standard</td>
		<td>独立</td>
		<td>使用本地镜像进行站点恢复复制</td>
		<td>使用本地镜像进行站点恢复复制</td>
    </tr>
    </tbody>
    </table>


## 部署先决条件

以下是开始之前需要满足的条件：


- 运行受支持 SQL Server 版本的本地 SQL Server 部署。通常还需要为 SQL Server 安装 Active Directory。
- 要部署的方案所要满足的先决条件。可在每篇部署文章中找到先决条件。此处列出了这些先决条件，[站点恢复 概述](/documentation/articles/site-recovery-overview)中也提供了相应的链接。
- 如果您要在 Azure 中设置恢复，则需要在 SQL Server 虚拟机上运行 [Azure 虚拟机准备情况评估](http://www.microsoft.com/download/details.aspx?id=40898)工具，以确保虚拟机与 Azure 和站点恢复兼容。

## 设置保护

需要执行几个步骤：

- 设置 Active Directory
- 为 SQL Server 群集或独立服务器设置保护

### 设置 Active Directory

需要在辅助恢复站点上安装 Active Directory 才能使 SQL Server 正常运行。可以使用以下几个选项：

- **小型企业** — 如果您有少量的应用程序和适用于本地站点的单个域控制器，而且您想要故障转移整个站点，则我们建议您使用站点恢复复制将域控制器复制到辅助数据中心或 Azure。

- **中大型企业** — 如果您有大量的应用程序、您运行的是 Active Directory 林，而且您想要按应用程序或工作负载进行故障转移，则我们建议您在辅助数据中心或 Azure 配置附加的域控制器。请注意，如果您要使用 AlwaysOn 可用性组恢复到远程站点，建议您在辅助站点或 Azure 上配置另一个域控制器，供已恢复 SQL Server 实例使用。

本文档中的说明假设辅助位置提供了域控制器。

### 为独立 SQL Server 设置保护


在此配置中，建议您使用站点恢复复制保护 SQL Server 计算机。确切步骤取决于 SQL Server 是设置为虚拟机还是物理服务器，以及您想要复制到 Azure 还是辅助本地站点。在 [站点恢复概述](/documentation/articles/site-recovery-overview)中获取有关所有部署方案的说明。


### 为 SQL Server 群集设置保护（2012 或 2014 Enterprise）

如果 SQL Server 使用可用性组实现高可用性或使用故障转移群集实例，我们建议您也在恢复站点上使用可用性组。请注意，本指南适用于不使用分布式事务的应用程序。

##### 本地到本地

1. 在可用性组中[配置数据库](https://msdn.microsoft.com/zh-cn/library/hh213078.aspx)。
2. 在辅助站点上创建新的虚拟网络。
3. 在新的虚拟网络与主站点之间配置站点到站点 VPN。
4. 在恢复站点上创建虚拟机，并在其上安装 SQL Server。
5. 将现有的 AlwaysOn 可用性组扩展为新的 SQL Server 虚拟机。将此 SQL Server 实例配置为异步副本。
6. 创建可用性组侦听器，或更新现有的侦听器，以包含异步副本虚拟机。
7. 确保应用程序场是使用侦听器设置的。如果它是使用数据库服务器名称设置的，请将其更新为使用侦听器，以便不需要在故障转移后重新配置该场。

#### 本地到 Azure

当您复制到 Azure 时，配置多个可用性组非常具有挑战性，因为每个可用性组都需要一个专用的侦听器，而且配置每个侦听器需要独立的云服务。建议您配置一个包含所有数据库的可用性组。

1. 创建 Azure 虚拟网络。
2. 在本地站点与此网络之间设置站点到站点 VPN。
3. 在网络中创建一个新的 SQL Server Azure 虚拟机，并将其配置为异步可用性组副本。如果您在故障转移到 Azure 后需要 SQL Server 层的高可用性，请在 Azure 中配置两个异步副本。
4. 在虚拟网络中设置域控制器的副本。
5. 确保已在虚拟机上启用虚拟机扩展。只有这样，才能在恢复计划中推送 SQL Server 特定的脚本。
6. 使用 Azure 的内部负载平衡器配置可用性组的 SQL Server 侦听器。
7. 将应用程序层配置为使用侦听器访问数据库层。对于使用分布式事务的应用程序，建议您使用站点恢复进行 SAN 复制或 VMWare 站点到站点复制。

### 为 SQL Server 群集设置保护（Standard 或 2008 R2）

对于运行 SQL Server Standard 版本或 SQL Server 2008 R2 的群集，建议您使用站点恢复复制来保护 SQL Server。

#### 本地到本地

- 对于 Hyper-V 环境，如果应用程序使用分布式事务，建议您部署[包含 SAN 复制的站点恢复](/documentation/articles/site-recovery-vmm-san)。

- 对于 VMware 环境，建议您部署 [VMware 到 VMware](/documentation/articles/site-recovery-vmware-to-vmware) 保护。

#### 本地到 Azure

复制到 Azure 时，站点恢复不支持来宾群集。SQL Server 也不会为 Standard 版本提供低成本灾难恢复解决方案。建议您在独立 SQL Server 中保护本地 SQL Server，并在 Azure 中恢复它。


1. 在本地站点中配置其他独立 SQL Server 实例。
2. 将此实例配置为需要保护的数据库的镜像。在高安全模式下配置镜像。
3.	根据环境（[Hyper-V](/documentation/articles/site-recovery-hyper-v-site-to-azure) 或 [VMware](/documentation/articles/site-recovery-vmware-to-azure)）在本地站点上配置 站点恢复。
4.	使用站点恢复复制将新的 SQL Server 实例复制到 Azure。该实例是高安全镜像副本，因此会将它与主群集同步，但会使用站点恢复复制将它复制到 Azure。

下图演示了此设置。

![标准群集](./media/site-recovery-sql/BCDRStandaloneClusterLocal.png)



##创建恢复计划

恢复计划会将应该一起故障转移的计算机分组在一起。在开始之前，请先详细了解[恢复计划](/documentation/articles/site-recovery-create-recovery-plans)和[故障转移](/documentation/articles/site-recovery-failover)。


### 创建 SQL Server 群集（SQL Server 2012/2014 Enterprise）的恢复计划

#### 配置 SQL Server 脚本以故障转移到 Azure

在此方案中，我们将为恢复计划使用自定义脚本和 Azure 自动化，以配置 SQL Server 可用性组的脚本化故障转移。

1.	为脚本创建本地文件，以故障转移可用性组。此示例脚本将在 Azure 副本上指定可用性组的路径，并将其故障转移到该副本实例。此脚本将通过使用自定义脚本扩展传递，以便在 SQL Server 副本虚拟机上运行。



    	Param(
    	[string]$SQLAvailabilityGroupPath
    	)
    	import-module sqlps
    	Switch-SqlAvailabilityGroup -Path $SQLAvailabilityGroupPath -AllowDataLoss -force

2.	将脚本上载到 Azure 存储帐户中的 Blob。使用以下示例：

    	$context = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName "Account" -StorageAccountKey "Key"
    	Set-AzureStorageBlobContent -Blob "AGFailover.ps1" -Container "script-container" -File "ScriptLocalFilePath" -context $context

3.	创建 Azure 自动化 Runbook，以便在 Azure 中调用 SQL Server 副本虚拟机上的脚本。使用此示例脚本来实现此目的。[详细了解](/documentation/articles/site-recovery-runbook-automation)如何在恢复计划中使用自动化 Runbook。

    	workflow SQLAvailabilityGroupFailover
    	{
    		param (
        		[Object]$RecoveryPlanContext
    		)

    		$Cred = Get-AutomationPSCredential -name 'AzureCredential'
	
    		#Connect to Azure
    		$AzureAccount = Add-AzureAccount -Environment AzureChinaCloud -Credential $Cred
    		$AzureSubscriptionName = Get-AutomationVariable –Name ‘AzureSubscriptionName’
    		Select-AzureSubscription -SubscriptionName $AzureSubscriptionName
    
    		InLineScript
    		{
			#Update the script with name of your storage account, key and blob name
			$context = New-AzureStorageContext -StorageAccountName "Account" -StorageAccountKey "Key";
			$sasuri = New-AzureStorageBlobSASToken -Container "script-container"- Blob "AGFailover.ps1" -Permission r -FullUri -Context $context;
     
			Write-output "failovertype " + $Using:RecoveryPlanContext.FailoverType;
               
			if ($Using:RecoveryPlanContext.FailoverType -eq "Test")
					{
           		#Skipping TFO in this version.
           		#We will update the script in a follow-up post with TFO support
           		Write-output "tfo: Skipping SQL Failover";
					}
			else
					{
           		Write-output "pfo/ufo";
           		#Get the SQL Azure Replica VM.
           		#Update the script to use the name of your VM and Cloud Service
           		$VM = Get-AzureVM -Name "SQLAzureVM" -ServiceName "SQLAzureReplica";     
       
           		Write-Output "Installing custom script extension"
           		#Install the Custom Script Extension on teh SQL Replica VM
           		Set-AzureVMExtension -ExtensionName CustomScriptExtension -VM $VM -Publisher Microsoft.Compute -Version 1.3| Update-AzureVM; 
                    
           		Write-output "Starting AG Failover";
           		#Execute the SQL Failover script
           		#Pass the SQL AG path as the argument.
       
           		$AGArgs="-SQLAvailabilityGroupPath sqlserver:\sql\sqlazureVM\default\availabilitygroups\testag";
       
           		Set-AzureVMCustomScriptExtension -VM $VM -FileUri $sasuri -Run "AGFailover.ps1" -Argument $AGArgs | Update-AzureVM;
       
           		Write-output "Completed AG Failover";

					}
        
    		}
    	}

4.	当您创建应用程序的恢复计划时，请添加可调用自动化 Runbook 的 "pre-Group 1 boot" 脚本化步骤以故障转移可用性组。

#### 配置 SQL Server 脚本以故障转移到辅助站点

1. 将此示例脚本添加到主站点和辅助站点上的 VMM 库。

    	Param(
    	[string]$SQLAvailabilityGroupPath
    	)
    	import-module sqlps
    	Switch-SqlAvailabilityGroup -Path $SQLAvailabilityGroupPath -AllowDataLoss -force

2. 当您创建应用程序的恢复计划时，请添加可调用脚本的 "pre-Group 1 boot" 脚本化步骤以故障转移可用性组。

### 创建 SQL Server 群集 (Standard) 的恢复计划

#### 配置 SQL Server 脚本以故障转移到 Azure

1.	创建脚本的本地文件，以故障转移 SQL Server 数据库镜像。使用此示例脚本。

    	Param(
    	[string]$database
    	)
    	Import-module sqlps
    	Invoke-sqlcmd –query “ALTER DATABASE $database SET PARTNER FORCE_SERVICE_ALLOW_DATA_LOSS”

2.	将脚本上载到 Azure 存储帐户中的 Blob。使用此示例脚本。

    	$context = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName "Account" -StorageAccountKey "Key"
    	Set-AzureStorageBlobContent -Blob "AGFailover.ps1" -Container "script-container" -File "ScriptLocalFilePath" -context $context

3.	创建 Azure 自动化 Runbook，以便在 Azure 中调用 SQL Server 副本虚拟机上的脚本。使用此示例脚本来实现此目的。[详细了解](/documentation/articles/site-recovery-runbook-automation)如何在恢复计划中使用自动化 Runbook。在执行此操作之前，请确保虚拟机代理在故障转移的 SQL Server 虚拟机上运行。

    	workflow SQLAvailabilityGroupFailover
		{
    		param (
        		[Object]$RecoveryPlanContext
    		)

    	$Cred = Get-AutomationPSCredential -name 'AzureCredential'
	
    	#Connect to Azure
    	$AzureAccount = Add-AzureAccount -Environment AzureChinaCloud -Credential $Cred
    	$AzureSubscriptionName = Get-AutomationVariable –Name ‘AzureSubscriptionName’
    	Select-AzureSubscription -SubscriptionName $AzureSubscriptionName
    
    	InLineScript
    	{
		#Update the script with name of your storage account, key and blob name
		$context = New-AzureStorageContext -StorageAccountName "Account" -StorageAccountKey "Key";
		$sasuri = New-AzureStorageBlobSASToken -Container "script-container" -Blob "AGFailover.ps1" -Permission r -FullUri -Context $context;
     
		Write-output "failovertype " + $Using:RecoveryPlanContext.FailoverType;
               
		if ($Using:RecoveryPlanContext.FailoverType -eq "Test")
				{
           		#Skipping TFO in this version.
           		#We will update the script in a follow-up post with TFO support
           		Write-output "tfo: Skipping SQL Failover";
				}
		else
					{
           		Write-output "pfo/ufo";
           		#Get the SQL Azure Replica VM.
           		#Update the script to use the name of your VM and Cloud Service
           		$VM = Get-AzureVM -Name "SQLAzureVM" -ServiceName "SQLAzureReplica";     
       
           		Write-Output "Installing custom script extension"
           		#Install the Custom Script Extension on teh SQL Replica VM
           		Set-AzureVMExtension -ExtensionName CustomScriptExtension -VM $VM -Publisher Microsoft.Compute -Version 1.3| Update-AzureVM; 
                    
           		Write-output "Starting AG Failover";
           		#Execute the SQL Failover script
           		#Pass the SQL AG path as the argument.
       
           		$AGArgs="-SQLAvailabilityGroupPath sqlserver:\sql\sqlazureVM\default\availabilitygroups\testag";
       
           		Set-AzureVMCustomScriptExtension -VM $VM -FileUri $sasuri -Run "AGFailover.ps1" -Argument $AGArgs | Update-AzureVM;
       
           		Write-output "Completed AG Failover";

					}
        
    		}
		}



4. 在恢复计划中添加以下步骤，以故障转移 SQL Server 层：

	- 对于计划的故障转移，请添加主端脚本，以便在“组关闭”之后关闭主群集。
	- 将 SQL Server 数据库镜像虚拟机添加到恢复计划，最好是添加到第一个引导组。
5.	添加故障转移后脚本，以便使用上述自动化脚本故障转移此虚拟机内部的镜像副本。请注意，由于数据库实例名称将会更改，因此必须重新配置应用程序层才能使用新的数据库。


#### 配置 SQL Server 脚本以故障转移到辅助站点

1.	将此示例脚本添加到主站点和辅助站点上的 VMM 库。

    	Param(
    	[string]$database
    	)
    	Import-module sqlps
    	Invoke-sqlcmd –query “ALTER DATABASE $database SET PARTNER FORCE_SERVICE_ALLOW_DATA_LOSS”

2.	将 SQL Server 数据库镜像虚拟机添加到恢复计划，最好是添加到第一个引导组。
3.	添加故障转移后脚本，以便使用上述 VMM 脚本故障转移此虚拟机内部的镜像副本。请注意，由于数据库实例名称将会更改，因此必须重新配置应用程序层才能使用新的数据库。





## 测试故障转移注意事项

如果您使用的是 AlwaysOn 可用性组，将无法执行 SQL Server 层的测试故障转移。请考虑使用以下选项作为替代方法：

- 选项 1

	1. 执行应用程序层和前端层的测试故障转移。
	2. 更新应用程序层，以便在只读模式下访问副本，并执行应用程序的只读测试。

- 选项 2
	1.	创建 SQL Server 虚拟机实例的副本（使用 VMM 克隆进行站点到站点备份或 Azure 备份），并在测试网络中启动该副本
	2.	使用恢复计划执行测试故障转移。

## 故障回复注意事项

对于 SQL 标准群集，非计划故障转移后的故障回复需要从镜像实例 SQL Server 备份并还原到原始群集，然后重新建立镜像。





 

<!---HONumber=71-->