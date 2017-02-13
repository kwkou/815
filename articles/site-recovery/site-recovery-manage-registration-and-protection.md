<properties
    pageTitle="删除服务器并禁用保护 | Azure"
    description="本文介绍如何从 Site Recovery 保管库中注销服务器，以及如何禁用虚拟机和物理服务器的保护。"
    services="site-recovery"
    documentationcenter=""
    author="rayne-wiselman"
    manager="cfreeman"
    editor="" />
<tags
    ms.assetid="ef1f31d5-285b-4a0f-89b5-0123cd422d80"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="12/28/2016"
    wacn.date="02/10/2017"
    ms.author="raynew" />  


# 删除服务器并禁用保护

Azure Site Recovery 服务有助于实现业务连续性和灾难恢复 (BCDR) 策略。该服务可以协调虚拟机和物理服务器的复制、故障转移与恢复。虚拟机可复制到 Azure 中，也可复制到本地辅助数据中心中。如需快速概览，请阅读[什么是 Azure Site Recovery？](/documentation/articles/site-recovery-overview/)

本文介绍如何从 Azure 门户的恢复服务保管库中取消注册服务器，以及如何禁用受 Site Recovery 保护的计算机的保护。


## 取消注册已连接的配置服务器

如果将 Windows/Linux 物理服务器复制到 Azure，可按如下所述从保管库中取消注册已连接的配置服务器：

1. 禁用计算机保护。在“受保护的项”>“复制的项”中，右键单击计算机 >“删除”。
2. 取消任何策略的关联。在“Site Recovery 基础结构”>“对于 VMWare 和物理机”>“复制策略”中，右键单击关联的策略。右键单击配置服务器 >“取消关联”。
3. 删除任何其他本地进程或主目标服务器。在“Site Recovery 基础结构”>“对于 VMWare 和物理机”>“配置服务器”中，右键单击服务器 >“删除”。
4. 删除配置服务器。
5. 手动卸载在主目标服务器（单独的服务器或在配置服务器上运行的服务器）上运行的移动服务。
6. 卸载任何其他的进程服务器。
7. 卸载配置服务器。
8. 在配置服务器上，卸载由 Site Recovery 安装的 MySQL 实例。
9. 在配置服务器的注册表中删除项 ``HKEY_LOCAL_MACHINE\Software\Microsoft\Azure Site Recovery``。

## 取消注册未连接的配置服务器

如果将 Windows/Linux 物理服务器复制到 Azure，可按如下所述从保管库中取消注册未连接的配置服务器：

1. 禁用计算机保护。在“受保护的项”>“复制的项”中，右键单击计算机 >“删除”。选择“停止管理计算机”。
2. 删除任何其他本地进程或主目标服务器。在“Site Recovery 基础结构”>“对于 VMWare 和物理机”>“配置服务器”中，右键单击服务器 >“删除”。
3. 删除配置服务器。
4. 手动卸载在主目标服务器（单独的服务器或在配置服务器上运行的服务器）上运行的移动服务。
5. 卸载任何其他的进程服务器。
6. 卸载配置服务器。
7. 在配置服务器上，卸载由 Site Recovery 安装的 MySQL 实例。
8. 在配置服务器的注册表中删除项 ``HKEY_LOCAL_MACHINE\Software\Microsoft\Azure Site Recovery``。

## 取消注册连接的 VMM 服务器

根据最佳实践要求，我们建议在 VMM 服务器连接到 Azure 之后取消注册该服务器。这样可确保正确清理 VMM 服务器（以及其他具有配对云的 VMM 服务器）上的设置。只有在连接出现永久性问题时，才应删除未连接的服务器。如果未连接 VMM 服务器，需手动运行一个脚本来清理设置。

1. 停止将云中的 VM 复制到要删除的 VMM 服务器上。
2. 删除由需要删除的 VMM 服务器上的云使用的任何网络映射。在“Site Recovery 基础结构”>“对于 System Center VMM”>“网络映射”中，右键单击网络映射 >“删除”。
3. 取消复制策略与要删除的 VMM 服务器上的云的关联。在“Site Recovery 基础结构”>“对于 System Center VMM”>“复制策略”中，右键单击关联的策略。右键单击云 >“取消关联”。
4. 删除 VMM 服务器或主动 VMM 节点。在“Site Recovery 基础结构”>“对于 System Center VMM”>“VMM 服务器”中，右键单击服务器 >“删除”。
5. 手动卸载 VMM 服务器上的提供程序。如果有一个群集，请从所有节点删除。
6. 若要复制到 Azure，请从已删除云的 Hyper-V 主机中手动删除 Microsoft 恢复服务代理。



### 取消注册未连接的 VMM 服务器

1. 停止将云中的 VM 复制到要删除的 VMM 服务器上。
2. 删除由需要删除的 VMM 服务器上的云使用的任何网络映射。在“Site Recovery 基础结构”>“对于 System Center VMM”>“网络映射”中，右键单击网络映射 >“删除”。
3. 记下 VMM 服务器的 ID。
4. 取消复制策略与要删除的 VMM 服务器上的云的关联。在“Site Recovery 基础结构”>“对于 System Center VMM”>“复制策略”中，右键单击关联的策略。右键单击云 >“取消关联”。
5. 删除 VMM 服务器或主动节点。在“Site Recovery 基础结构”>“对于 System Center VMM”>“VMM 服务器”中，右键单击服务器 >“删除”。
6. 在 VMM 服务器上下载并运行[清理脚本](http://aka.ms/asr-cleanup-script-vmm)。使用“以管理员身份运行”选项打开 PowerShell，更改默认 (LocalMachine) 范围的执行策略。在脚本中，指定要删除的 VMM 服务器的 ID。脚本会从服务器中删除注册和云配对信息。
5. 在任何其他包含云的 VMM 服务器上运行清理脚本，这些云已与需要删除的 VMM 服务器上的云配对。
6. 在任何其他被动 VMM 群集节点（已安装提供程序）上运行清理脚本。
7. 手动卸载 VMM 服务器上的提供程序。如果有一个群集，请从所有节点删除。
8. 若要复制到 Azure，可从已删除云的 Hyper-V 主机中删除 Microsoft 恢复服务代理。

## 取消注册 Hyper-V 站点中的 Hyper-V 主机

未由 VMM 托管的 Hyper-V 主机将收集到 Hyper-V 站点中。在 Hyper-V 站点中删除主机，如下所示：

1. 禁用位于主机上的 Hyper-V VM 的复制。
2. 取消关联 Hyper-V 站点的策略。在“Site Recovery 基础结构”>“对于 Hyper-V 站点”>“复制策略”中，右键单击关联的策略。右键单击站点 >“取消关联”。
3. 删除 Hyper-V 主机。在“Site Recovery 基础结构”>“对于 System Center VMM”>“Hyper-V 主机”中，右键单击服务器 >“删除”。
4. 从 Hyper-V 站点中删除所有主机后，将该站点删除。在“Site Recovery 基础结构”>“对于 System Center VMM”>“Hyper-V 站点”中，右键单击站点 >“删除”。
5. 在每个已删除的 Hyper-V 主机上运行以下脚本。该脚本清理服务器上的设置，并从保管库中取消注册该服务器。

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



## 禁用对物理服务器的保护

1. 在“受保护的项”>“复制的项”中，右键单击计算机 >“删除”。
2. 在“删除计算机”中，选择以下选项之一：
    - **禁用对计算机的保护(推荐)**。使用此选项可停止复制计算机。将自动清理 Site Recovery 设置。此选项仅在以下情况下显示：
        - **已调整 VM 卷的大小** - 调整卷大小时，虚拟机将进入临界状态。选择此选项将禁用保护，同时保留 Azure 中的恢复点。在重新启用对虚拟机的保护时，调整过大小的卷的数据将被传输到 Azure。
        - **最近已运行故障转移** - 运行故障转移来测试环境以后，选择此选项即可再次开始保护本地计算机。此选项禁用每个虚拟机，然后用户需重新启用对虚拟机的保护。通过此选项禁用虚拟机不会影响 Azure 中的副本虚拟机。请勿从计算机中卸载移动服务。
    - **停止管理计算机**。如果选择此选项，将仅从保管库删除计算机。不会影响虚拟机的本地保护设置。若要删除计算机上的设置并从 Azure 订阅中删除计算机，需要通过卸载移动服务来清理设置。

## 在 VMM 云中禁用对 Hyper-V VM 的保护

1. 在“受保护的项”>“复制的项”中，右键单击计算机 >“删除”。
2. 在“删除计算机”中，选择以下选项之一：

    - **禁用对计算机的保护(推荐)**。使用此选项可停止复制计算机。将自动清理 Site Recovery 设置。
    - **停止管理计算机**。如果选择此选项，将仅从保管库删除计算机。不会影响虚拟机的本地保护设置。若要删除计算机上的设置并从 Azure 订阅中删除计算机，需根据以下说明手动清理设置。请注意，若选择删除虚拟机及其硬盘，将从目标位置删除它们。

### 清理保护设置 - 复制到辅助 VMM 站点

若已选择“停止管理计算机”且要复制到辅助站点，则请在主服务器上运行此脚本，以便清理主虚拟机的设置。在 VMM 控制台中，单击“PowerShell”按钮打开 VMM PowerShell 控制台。将 SQLVM1 替换为相应虚拟机名称。

	     $vm = get-scvirtualmachine -Name "SQLVM1"
	     Set-SCVirtualMachine -VM $vm -ClearDRProtection

2. 在辅助 VMM 服务器上，运行此脚本以清理辅助虚拟机的设置：

	    $vm = get-scvirtualmachine -Name "SQLVM1"
	    Remove-SCVirtualMachine -VM $vm -Force

3. 在辅助 VMM 服务器上刷新 Hyper-V 主机服务器上的虚拟机，以便在 VMM 控制台中重新检测辅助 VM。
4. 上述步骤清理 VMM 服务器上的复制设置。若要停止虚拟机的复制，请在主 VM 和辅助 VM 上运行以下脚本。将 SQLVM1 替换为你的虚拟机名称。

	    Remove-VMReplication –VMName “SQLVM1”

### 清理保护设置 - 复制到 Azure

1. 若已选择“停止管理计算机”且要复制到 Azure，请通过 VMM 控制台使用 PowerShell 在源 VMM 服务器上运行此脚本。

	    $vm = get-scvirtualmachine -Name "SQLVM1"
	    Set-SCVirtualMachine -VM $vm -ClearDRProtection

2. 上述步骤清理 VMM 服务器上的复制设置。若要停止运行在 Hyper-V 主机服务器上的虚拟机的复制，请运行以下脚本。将 SQLVM1 替换为你的虚拟机的名称，将 host01.contoso.com 替换为 Hyper-V 主机服务器的名称。

	    $vmName = "SQLVM1"
	    $hostName  = "host01.contoso.com"
	    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'" -computername $hostName
	    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  -computername $hostName
	    $replicationService.RemoveReplicationRelationship($vm.__PATH)


## 在 Hyper-V 站点中禁用对 Hyper-V VM 的保护

若要在没有 VMM 服务器的情况下将 Hyper-V VM 复制到 Azure，请执行此过程。

1. 在“受保护的项”>“复制的项”中，右键单击计算机 >“删除”。
2. 在“删除计算机”中，可以选择以下选项：

   - **禁用对计算机的保护(推荐)**。使用此选项可停止复制计算机。将自动清理 Site Recovery 设置。
   - **停止管理计算机**。如果选择此选项，将仅从保管库删除计算机。不会影响虚拟机的本地保护设置。若要删除计算机上的设置并从 Azure 订阅中删除虚拟机，需手动清理设置。如果你选择删除虚拟机及其硬盘，将从目标位置删除它们。
3. 若已选择“停止管理计算机”，请在源 Hyper-V 主机服务器上运行此脚本，删除虚拟机的复制。将 SQLVM1 替换为你的虚拟机名称。

	    $vmName = "SQLVM1"
	    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'"
	    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"
	    $replicationService.RemoveReplicationRelationship($vm.__PATH)

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description: wording update; add "取消注册未连接的配置服务器" section-->