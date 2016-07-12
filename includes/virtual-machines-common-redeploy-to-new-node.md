<!-- Ibiza portal: tested -->

如果你不知道如何通过远程桌面 (RDP) 来连接到基于 Windows 的 Azure 虚拟机，或者不知道如何通过 SSH 来连接到基于 Linux 的 Azure 虚拟机，则可参考本文来自行解决问题，不需要反复地求助于支持和调整虚拟机大小。当你通过 Azure PowerShell 来调用重新部署操作时，Azure 会重新部署你的虚拟机。

请注意，完成此操作后，你会丢失临时磁盘数据，而系统则会更新与虚拟机关联的动态 IP 地址。


## 使用 Azure PowerShell

请确保已在计算机上安装最新的 Azure PowerShell 1.x。有关详细信息，请阅读[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

请使用以下 Azure PowerShell 命令来重新部署虚拟机：

	Set-AzureRmVM -Redeploy -ResourceGroupName $rgname -Name $vmname 


在运行该命令的同时，请在 [Azure 门户预览](https://portal.azure.cn)中查看你的虚拟机。请注意，VM 的“状态”更改如下：

1. 初始“状态”为“正在运行”

	![重新部署初始状态](./media/virtual-machines-common-redeploy-to-new-node/statusrunning1.png)

2. “状态”更改为“正在更新”

	![重新部署状态正在更新](./media/virtual-machines-common-redeploy-to-new-node/statusupdating.png)

3. “状态”更改为“正在启动”

	![重新部署状态正在启动](./media/virtual-machines-common-redeploy-to-new-node/statusstarting.png)

4. “状态”更改为“正在运行”

	![重新部署最终状态](./media/virtual-machines-common-redeploy-to-new-node/statusrunning2.png)

如果“状态”变回“正在运行”，则表明 VM 已成功完成重新部署。

<!---HONumber=Mooncake_0418_2016-->