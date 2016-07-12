<properties
	pageTitle="删除服务器并禁用保护 | Azure" 
	description="本文介绍如何从 Site Recovery 保管库中注销服务器，以及如何禁用虚拟机和物理服务器的保护。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="rayne-wiselman" 
	manager="jwhit" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.date="02/22/2016" 
	wacn.date="04/05/2016"/>

# 删除服务器并禁用保护

Azure Site Recovery 服务有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以协调虚拟机和物理服务器的复制、故障转移和恢复。虚拟机可复制到 Azure 中，也可复制到本地数据中心中。如需快速概览，请阅读[什么是 Azure Site Recovery？](/documentation/articles/site-recovery-overview/)

## 概述

本文说明如何从 Site Recovery 保管库中取消注册服务器，以及如何禁用 Site Recovery 保护的虚拟机保护。

请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。

## 取消注册 VMM 服务器

可以通过在 Azure Site Recovery 门户中的“服务器”选项卡上删除服务器，从保管库中取消注册 VMM 服务器。请注意：

-  **已连接 VMM 服务器**：建议在 VMM 服务器连接到 Azure 之后取消注册该服务器。这样可确保正确清理本地 VMM 服务器上的设置以及与之关联的 VMM 服务器（包含的云映射到要删除的服务器上的云的 VMM 服务器）。建议只在连接出现永久性问题时才删除未连接的服务器。
- **未连接 VMM 服务器**：如果 VMM 服务器未连接，当您删除该服务器时，需要手动运行一个脚本才能执行清理操作。该脚本位于 [Microsoft 库](https://gallery.technet.microsoft.com/scriptcenter/Cleanup-Script-for-Windows-95101439)中。请记下服务器的 VMM ID，以完成手动清理过程。
- **群集中的 VMM 服务器**：如果您需要取消注册在群集中部署的 VMM 服务器，请执行以下操作：

	- 如果服务器已连接，请删除“服务器”选项卡上已连接的 VMM 服务器。若要卸载服务器上的提供者，请登录到每个群集节点，然后从控制面板卸载该提供者。针对群集中的所有被动节点执行上一部分中所述的清理脚本，以删除注册条目。
	- 如果服务器未连接，你需要针对所有群集节点运行清理脚本。

### 取消注册未连接的 VMM 服务器

在你想要删除的 VMM 服务器上：

1. 从 Azure 门户取消注册 VMM 服务器。
2. 在 VMM 服务器上，下载清理脚本。
3. 使用“以管理员身份运行”选项打开 PowerShell，以更改默认 (LocalMachine) 范围的执行策略。
4. 根据脚本中的说明操作。 

在包含的云与你要删除的服务器上的云配对的 VMM 服务器上：

1. 运行清理脚本，然后执行步骤 2 至 4。
2. 指定已取消注册的 VMM 服务器的 VMM ID。 
3. 此脚本将删除 VMM 服务器的注册信息以及云配对信息。


## 取消注册 Hyper-V 站点中的 Hyper-V 服务器

部署 Azure Site Recovery 以保护 Hyper-V 站点中 Hyper-V 服务器（不包含任何 VMM 服务器）上的虚拟机时，可以从保管库取消注册 Hyper-V 服务器，如下所述：

1. 对 Hyper-V 服务器上的虚拟机禁用保护。
2. 在 Azure Site Recovery 门户中的“服务器”选项卡上选择服务器，然后选择“删除”。执行此操作时，服务器不需要连接到 Azure。
3. 运行以下脚本以清理服务器上的设置，并从保管库中取消注册该服务器。 

	    pushd .
	    try
	    {
		     $windowsIdentity=[System.Security.Principal.WindowsIdentity]::GetCurrent()
		     $principal=new-object System.Security.Principal.WindowsPrincipal($windowsIdentity)
		     $administrators=[System.Security.Principal.WindowsBuiltInRole]::Administrator
		     $isAdmin=$principal.IsInRole($administrators)
		     if (!$isAdmin)
		     {
		        "Please run the script as an administrator in elevated mode."
		        $choice = Read-Host
		        return;       
		     }
	
		    $error.Clear()    
			"This script will remove the old Azure Site Recovery Provider related properties. Do you want to continue (Y/N) ?"
			$choice =  Read-Host
		
			if (!($choice -eq 'Y' -or $choice -eq 'y'))
			{
			"Stopping cleanup."
			return;
			}
		
			$serviceName = "dra"
			$service = Get-Service -Name $serviceName
			if ($service.Status -eq "Running")
		    {
				"Stopping the Azure Site Recovery service..."
				net stop $serviceName
			}
		
		    $asrHivePath = "HKLM:\SOFTWARE\Microsoft\Azure Site Recovery"
			$registrationPath = $asrHivePath + '\Registration'
			$proxySettingsPath = $asrHivePath + '\ProxySettings'
		    $draIdvalue = 'DraID'
	    
		    if (Test-Path $asrHivePath)
		    {
		        if (Test-Path $registrationPath)
		        {
		            "Removing registration related registry keys."	
			        Remove-Item -Recurse -Path $registrationPath
		        }
	
		        if (Test-Path $proxySettingsPath)
	        {
			        "Removing proxy settings"
			        Remove-Item -Recurse -Path $proxySettingsPath
		        }
	
		        $regNode = Get-ItemProperty -Path $asrHivePath
		        if($regNode.DraID -ne $null)
		        {            
		            "Removing DraId"
		            Remove-ItemProperty -Path $asrHivePath -Name $draIdValue
		        }
			    "Registry keys removed."
		    }
	
		    # First retrive all the certificates to be deleted
			$ASRcerts = Get-ChildItem -Path cert:\localmachine\my | where-object {$_.friendlyname.startswith('ASR_SRSAUTH_CERT_KEY_CONTAINER') -or $_.friendlyname.startswith('ASR_HYPER_V_HOST_CERT_KEY_CONTAINER')}
			# Open a cert store object
			$store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My","LocalMachine")
			$store.Open('ReadWrite')
			# Delete the certs
			"Removing all related certificates"
		    foreach ($cert in $ASRcerts)
		    {
			    $store.Remove($cert)
		    }
	    }catch
	    {	
		    [system.exception]
		    Write-Host "Error occured" -ForegroundColor "Red"
		    $error[0] 
		    Write-Host "FAILED" -ForegroundColor "Red"
	    }
	    popd


## 停止保护 Hyper-V 虚拟机

如果您想要停止保护 HYPER-V 虚拟机，将需要删除对它的保护。根据删除保护的方式，您可能需要在虚拟机上手动清除保护设置。

### 删除保护

1. 在云属性的“虚拟机”选项卡中选择虚拟机，然后选择“删除”。
2. “确认删除虚拟机”页上有几个选项：

	- 禁用保护 — 如果您启用并保存此选项，虚拟机将不再受 Site Recovery 的保护。系统将自动清理虚拟机的保护设置。
	- 从保管库删除 — 如果您选择此选项，将只从 Site Recovery 保管库中删除虚拟机。不会影响虚拟机的本地保护设置。您将需要手动清除设置以删除保护设置并从 Azure 订阅中删除虚拟机，而且要删除保护设置，您需要使用下面的说明手动清除它们。

如果你选择删除虚拟机及其硬盘，将从目标位置删除它们。

### 手动清理保护设置（在 VMM 站点之间）

如果选择了“停止管理虚拟机”，请手动清理设置：

1. 在主服务器上，从 VMM 控制台运行此脚本，以清理主虚拟机的设置。在 VMM 控制台中，单击“PowerShell”按钮打开 VMM PowerShell 控制台。将 SQLVM1 替换为你的虚拟机名称。

	     $vm = get-scvirtualmachine -Name "SQLVM1"
	     Set-SCVirtualMachine -VM $vm -ClearDRProtection

2. 在辅助 VMM 服务器上，运行此脚本以清理辅助虚拟机的设置：

	    $vm = get-scvirtualmachine -Name "SQLVM1"
	    Remove-SCVirtualMachine -VM $vm -Force

3. 在辅助 VMM 服务器上运行脚本之后，刷新 Hyper-V 主机服务器上的虚拟机，以便在 VMM 控制台中重新检测辅助虚拟机。
4. 上述步骤只会清除 VMM 服务器的复制设置。如果你想要删除虚拟机的虚拟机复制，则需要在主虚拟机与辅助虚拟机上执行以下步骤。运行以下脚本来删除复制，并将 SQLVM1 替换为你的虚拟机名称。

	    Remove-VMReplication –VMName “SQLVM1”


### 手动清理保护设置（在本地 VMM 站点与 Azure 之间）

1. 在源 VMM 服务器上，运行此脚本以清理主虚拟机的设置：

	    $vm = get-scvirtualmachine -Name "SQLVM1"
	    Set-SCVirtualMachine -VM $vm -ClearDRProtection

2. 上述步骤只会清除 VMM 服务器的复制设置。在从 VMM 服务器删除复制之后，请务必使用此脚本删除 Hyper-V 主机服务器上运行的虚拟机的复制。将 SQLVM1 替换为你的虚拟机名称，将 host01.contoso.com 替换为 Hyper-V 主机服务器的名称。

	    $vmName = "SQLVM1"
	    $hostName  = "host01.contoso.com"
	    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'" -computername $hostName
	    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  -computername $hostName
	    $replicationService.RemoveReplicationRelationship($vm.__PATH)

### 手动清理保护设置（在 Hyper-V 站点与 Azure 之间）

1. 在源 Hyper-V 主机服务器上，使用此脚本删除虚拟机的复制。将 SQLVM1 替换为你的虚拟机名称。

	    $vmName = "SQLVM1"
	    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'"
	    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"
	    $replicationService.RemoveReplicationRelationship($vm.__PATH)

## 停止保护 VMware 虚拟机或物理服务器

如果您想要停止保护 VMware 虚拟机或物理服务器，将需要删除对它的保护。根据删除保护的方式，您可能需要在虚拟机上手动清除保护设置。

### 删除保护

1. 在云属性的“虚拟机”选项卡中选择虚拟机，然后选择“删除”。
2. 在“删除虚拟机”中选择下列选项之一：

	- **禁用保护(用于恢复深化和重设卷大小)** — 如果您已经执行了以下操作，则将只能查看和启用此选项：
		- **调整了虚拟机卷的大小** — 当您调整卷大小时，虚拟机将进入临界状态。如果发生这种情况请选择此选项。它将禁用保护，同时保留 Azure 中的恢复点。在重新启用对虚拟机的保护时，调整过大小的卷的数据将被传输到 Azure。
		- **运行故障转移** — 在您已通过运行从本地 VMware 虚拟机或物理服务器故障转移到 Azure 以测试环境后，选择此选项以开始再次保护您的本地虚拟机。此选项会禁用每个虚拟机，然后您将需要重新启用对它们的保护。请注意：
			- 禁用虚拟机的此项设置不会影响 Azure 中的副本虚拟机。
			- 您不能从虚拟机卸载移动服务。
	
	- **禁用保护** — 如果您启用并保存此选项，虚拟机将不再受 Site Recovery 的保护。系统将自动清理虚拟机的保护设置。
	- **从保管库删除** — 如果您选择此选项，将只从 Site Recovery 保管库中删除虚拟机。不会影响虚拟机的本地保护设置。若要删除计算机上的设置并从 Azure 订阅中删除虚拟机，则需要通过卸载移动服务来清理设置。
	
		![删除选项](./media/site-recovery-manage-registration-and-protection/remove-vm.png)

<!---HONumber=Mooncake_0328_2016-->