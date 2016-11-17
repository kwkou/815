<properties 
	pageTitle="重新部署 Windows 虚拟机 |Azure" 
	description="说明如何通过重新部署 Windows 虚拟机来缓解 RDP 连接问题。" 
	services="virtual-machines-windows" 
	documentationCenter="virtual-machines" 
	authors="iainfoulds" 
	manager="timlt"
	tags="azure-resource-manager,top-support-issue" 
/>
	

<tags
	ms.service="virtual-machines-windows"
	ms.date="06/28/2016"
	wacn.date="08/15/2016"/>


# 将虚拟机重新部署到新的 Azure 节点

如果在对远程桌面 (RDP) 连接或应用程序对基于 Windows 的 Azure 虚拟机 (VM) 的访问进行故障排除时遇到困难，重新部署 VM 可能会有帮助。重新部署 VM 时，将 VM 移到 Azure 基础结构中的新节点，然后重新打开它，同时保留所有配置选项和关联的资源。本文介绍了如何使用 Azure PowerShell 或 Azure 门户预览重新部署 VM。

> [AZURE.NOTE] 重新部署 VM 后，临时磁盘都将丢失，并将更新与虚拟网络接口相关联的动态 IP 地址。

## 使用 Azure PowerShell

请确保已在计算机上安装最新的 Azure PowerShell 1.x。有关详细信息，请阅读[《How to install and configure Azure PowerShell》（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。

请使用以下 Azure PowerShell 命令来重新部署虚拟机：

	Set-AzureRmVM -Redeploy -ResourceGroupName $rgname -Name $vmname 


[AZURE.INCLUDE [virtual-machines-common-redeploy-to-new-node](../../includes/virtual-machines-common-redeploy-to-new-node.md)]


## 后续步骤
如果在连接 VM 时遇到问题，可以在 [troubleshooting RDP connections（RDP 连接故障排除）](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/) 或 [detailed RDP troubleshooting steps（详细的 RDP 故障排除步骤）](/documentation/articles/virtual-machines-windows-detailed-troubleshoot-rdp/) 中找到特定的帮助。如果无法访问在 VM 上运行的应用程序，还可以阅读 [application troubleshooting issues（应用程序故障排除问题）](/documentation/articles/virtual-machines-windows-troubleshoot-app-connection/)。

<!---HONumber=Mooncake_0808_2016-->