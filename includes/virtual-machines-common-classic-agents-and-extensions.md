VM 扩展可帮助你：

* 修改安全性和识别功能，例如重置帐户值和使用反恶意软件
* 启动、停止或配置监视和诊断
* 重置或安装连接功能，如 RDP 和 SSH
* 诊断、监视和管理 VM

还有许多其他功能。 新的 VM 扩展功能定期发布。 本文介绍适用于 Windows 和 Linux 的 Azure VM 代理，以及这些代理如何支持 VM 扩展功能。 有关按功能类别列出的 VM 扩展的列表，请参阅 [Azure VM 扩展和功能](/documentation/articles/virtual-machines-windows-extensions-features/)。

## <a name="azure-vm-agents-for-windows-and-linux"></a>适用于 Windows 和 Linux 的 Azure VM 代理
Azure 虚拟机代理（VM 代理）是一个安全的轻型进程，用于在 Azure 虚拟机的实例上安装、配置和删除 VM 扩展。 VM 代理充当 Azure VM 的安全本地控制服务。 该代理加载的扩展提供特定功能，以在使用实例时提高工作效率。

存在两种 Azure VM 代理，一种用于 Windows VM，另一种用于 Linux VM。

如果需要一个虚拟机实例来使用一个或多个 VM 扩展，该实例必须有安装的 VM 代理。 通过 Azure 门户创建的虚拟机映像和**应用商店**提供的映像会自动在创建过程中安装 VM 代理。 如果虚拟机实例缺少 VM 代理，则可在创建虚拟机实例以后安装该 VM 代理。 也可安装随后上载的自定义 VM 映像中的代理。

> [AZURE.IMPORTANT]
> 这些 VM 代理是非常轻量级的，可启用虚拟机实例的安全管理的服务。 可能也存在你不想要 VM 代理的情况。 如果是这样，请务必使用 Azure CLI 或 PowerShell 创建未安装 VM 代理的 VM。 尽管可以物理删除 VM 代理，但实例上的 VM 扩展的行为是不确定的。 因此，不支持删除已安装的 VM 代理。
>

在下列情况下会启用 VM 代理：

* 用户使用 Azure 门户，然后从**应用商店**选择一个映像，通过这种方式创建 VM 的实例。
* 通过 [New-AzureVM](https://msdn.microsoft.com/zh-cn/library/azure/dn495254.aspx) 或 [New-AzureQuickVM](https://msdn.microsoft.com/zh-cn/library/azure/dn495183.aspx) cmdlet 创建 VM 实例时。 可以通过在 [Add-AzureProvisioningConfig](https://msdn.microsoft.com/zh-cn/library/azure/dn495299.aspx) cmdlet 中添加 **-DisableGuestAgent** 参数来创建没有 VM 代理的 VM。

* 用户手动下载 VM 代理并将其安装在现有的 VM 实例上，然后将 **ProvisionGuestAgent** 值设置为 **true**。 可以对 Windows 和 Linux 代理使用此方法，只需使用 PowerShell 命令或 REST 调用即可。 （如果在手动安装 VM 代理后未设置 **ProvisionGuestAgent** 值，则将无法正常检测到添加的 VM 代理。）以下代码示例演示如何使用 PowerShell 执行此操作，其中 `$svc` 和 `$name` 参数已确定：

        $vm = Get-AzureVM -ServiceName $svc -Name $name
        $vm.VM.ProvisionGuestAgent = $TRUE
        Update-AzureVM -Name $name -VM $vm.VM -ServiceName $svc

* 用户创建一个 VM 映像，其中包括已安装的 VM 代理。 如果存在包含 VM 代理的映像，则可将该映像上载到 Azure。 对于 Windows VM，下载 [Windows VM 代理 .msi 文件](http://go.microsoft.com/fwlink/?LinkID=394789)并安装 VM 代理。 对于 Linux VM，请从位于 <https://github.com/Azure/WALinuxAgent> 的 GitHub 存储库安装 VM 代理。 有关如何在 Linux 上安装 VM 代理的详细信息，请参阅 [Azure Linux VM 代理用户指南](/documentation/articles/virtual-machines-linux-agent-user-guide/)。

> [AZURE.NOTE]
> 在 PaaS 中，VM 代理名为 **WindowsAzureGuestAgent**，并且始终可在 Web 角色和辅助角色 VM 上找到。 （有关详细信息，请参阅 [Azure 角色体系结构](http://blogs.msdn.com/b/kwill/archive/2011/05/05/windows-azure-role-architecture.aspx)。）角色 VM 的 VM 代理现在可以按向永久性虚拟机添加扩展的相同方式向云服务 VM 添加扩展。 角色 VM 和永久性 VM 上的 VM 扩展的最大差异表现在添加 VM 扩展的时候。 使用角色 VM 时，扩展先添加到云服务，然后添加到该云服务中的部署。
>
> 使用 [Get-AzureServiceAvailableExtension](https://msdn.microsoft.com/zh-cn/library/azure/dn722498.aspx) cmdlet 可列出所有可用的角色 VM 扩展。
>
>

## <a name="find-add-update-and-remove-vm-extensions"></a>查找、添加、更新和删除 VM 扩展
有关这些任务的详细信息，请参阅[添加、查找、更新和删除 Azure VM 扩展](/documentation/articles/virtual-machines-windows-classic-manage-extensions/)。