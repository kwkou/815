<properties
	pageTitle="准备环境以备份 Resource Manager 部署的虚拟机 | Azure"
	description="确保对环境进行准备，以便在 Azure 中备份虚拟机"
	services="backup"
	documentationCenter=""
	authors="markgalioto"
	manager="cfreeman"
	editor=""
	keywords="备份; 正在备份;"/>

<tags
	ms.service="backup"
	ms.date="06/03/2016"
	wacn.date="09/06/2016"/>


# 准备好环境以备份 ARM 虚拟机

> [AZURE.SELECTOR]
- [Resource Manager 模型](/documentation/articles/backup-azure-arm-vms-prepare/)
- [经典模型](/documentation/articles/backup-azure-vms-prepare/)

本文提供有关准备环境以备份 Resource Manager 部署的虚拟机 (VM) 的步骤。过程中显示的步骤将使用 Azure PowerShell。

Azure 备份服务提供两种类型的保管库（备份保管库和恢复服务保管库）来保护 VM。备份保管库保护使用经典部署模型部署的 VM。恢复服务保管库保护**经典部署和 Resource Manager 部署的 VM**。必须使用恢复服务保管库来保护 Resource Manager 部署的 VM。

>[AZURE.NOTE] Azure 有两种用于创建和使用资源的部署模型：[Resource Manager 和经典部署模型](/documentation/articles/resource-manager-deployment-model/)。有关使用经典部署模型 VM 的详细信息，请参阅[准备好环境以备份 Azure 虚拟机](/documentation/articles/backup-azure-vms-prepare/)。

请确保符合以下先决条件，这样才能保护或备份 Resource Manager 部署的虚拟机 (VM)：

- 在与 VM 相同的位置创建恢复服务保管库（或标识现有的恢复服务保管库）。
- 选择方案、定义备份策略并定义要保护的项。
- 检查虚拟机上的 VM 代理安装。
- 检查网络连接

如果你确定你的环境满足这些条件，请转到[备份 VM 的文章](/documentation/articles/backup-azure-vms/)。如果需要设置或检查上述任何先决条件，本文将引导你逐步完成先决条件的准备步骤。


## 备份和还原 VM 时的限制

准备环境之前，请先了解限制。

- 不支持备份超过 16 个数据磁盘的虚拟机。
- 不支持备份使用保留 IP 地址且未定义终结点的虚拟机。
- 不支持在恢复过程中替换现有虚拟机。如果在 VM 存在时尝试还原 VM，还原操作将会失败。
- 不支持跨区域备份和恢复。
- 可以在 Azure 的所有公共区域中备份虚拟机（请参阅支持区域的[清单](https://azure.microsoft.com/regions/#services)）。在创建保管库期间，如果你要寻找的区域目前不受支持，则不会在下拉列表中显示它。
- 只可以备份选定操作系统版本的虚拟机：
  - **Linux**：请参阅 [Azure 认可的分发版列表](/documentation/articles/virtual-machines-linux-endorsed-distros/)。只要虚拟机上装有 VM 代理，其他自带的 Linux 分发版应该也能正常运行。
  - **Windows Server**：不支持低于 Windows Server 2008 R2 的版本。
- 仅支持通过 PowerShell 还原属于多 DC 配置的域控制器 (DC) VM。阅读有关[还原多 DC 域控制器](/documentation/articles/backup-azure-restore-vms/#restoring-domain-controller-vms)的详细信息。
- 仅支持通过 PowerShell 还原采用以下特殊网络配置的虚拟机。还原操作完成后，在 UI 中使用还原工作流创建的 VM 将不采用这些网络配置。若要了解详细信息，请参阅[还原采用特殊网络配置的 VM](/documentation/articles/backup-azure-restore-vms/#restoring-vms-with-special-netwrok-configurations)。
  - 采用负载平衡器配置的虚拟机（内部和外部）
  - 使用多个保留 IP 地址的虚拟机
  - 使用多个网络适配器的虚拟机

## 为 VM 创建恢复服务保管库

1. 首先使用以下命令行登录你的 Azure 订阅。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

1. 如果你是首次使用 Azure 备份，则必须使用 **[Register-AzureRMResourceProvider](https://msdn.microsoft.com/zh-cn/library/mt603685.aspx)** cmdlet 注册用于订阅的 Azure 恢复服务提供程序。

        Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"

2. 恢复服务保管库是一种 Resource Manager 资源，因此需要将它放在资源组中。你可以使用现有的资源组，也可以使用 **[New-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/mt603739.aspx)** cmdlet 创建新的资源组。创建新的资源组时，请指定资源组的名称和位置。

        New-AzureRmResourceGroup –Name "test-rg" –Location "China North"

3. 使用 **[New-AzureRmRecoveryServicesVault](https://msdn.microsoft.com/zh-cn/library/mt643910.aspx)** cmdlet 创建新的保管库。确保为保管库指定的位置与用于资源组的位置是相同的。

        New-AzureRmRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "West US"

4. 指定要使用的存储冗余类型；你可以使用[本地冗余存储 (LRS)](/documentation/articles/storage-redundancy/#locally-redundant-storage) 或[异地冗余存储 (GRS)](/documentation/articles/storage-redundancy/#geo-redundant-storage)。以下示例显示，testVault 的 -BackupStorageRedundancy 选项设置为 GeoRedundant。

        $vault1 = Get-AzureRmRecoveryServicesVault –Name "testVault"
        Set-AzureRmRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant

## 选择备份目标、设置策略并定义要保护的项
现在，你已经创建了恢复服务保管库，因此可以使用它来保护虚拟机。但是，在应用保护之前，必须设置保管库上下文，并且需验证保护策略。保管库上下文定义了保管库中受保护的数据类型。保护策略是指对备份作业的运行时间以及每个备份快照的保留时长进行计划。

在 VM 上启用保护之前，必须设置保管库上下文。该上下文将应用到所有后续 cmdlet。

	PS C:\> Get-AzureRmRecoveryServicesVault -Name testvault | Set-AzureRmRecoveryServicesVaultContext

### 创建保护策略

当你创建新保管库时，它附带了一个默认策略。此策略会在每天的指定时间触发备份作业。根据默认策略，备份快照将保留 30 天。可以使用默认策略快速保护你的 VM，以后再使用不同的详细信息编辑该策略。

若要查看保管库中的可用策略列表，请使用 **[Get-AzureRmRecoveryServicesBackupProtectionPolicy](https://msdn.microsoft.com/zh-cn/library/mt723300.aspx)**：

	PS C:\> Get-AzureRmRecoveryServicesBackupProtectionPolicy -WorkloadType AzureVM
	Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
	----                 ------------       -------------------- ----------                ----------
	DefaultPolicy        AzureVM            AzureVM              4/14/2016 5:00:00 PM

> [AZURE.NOTE] PowerShell 中 BackupTime 字段的时区是 UTC。

一个备份保护策略至少与一个保留策略相关联。保留策略定义在 Azure 备份中保留恢复点的时限。使用 **Get-AzureRmRecoveryServicesBackupRetentionPolicyObject** 可以查看默认保留策略。同理，可以使用 **Get-AzureRmRecoveryServicesBackupSchedulePolicyObject** 获取默认计划策略。计划和保留策略对象将用作 **New-AzureRmRecoveryServicesBackupProtectionPolicy** cmdlet 的输入。

备份保护策略定义对某个项目进行备份的时间和频率。New-AzureRmRecoveryServicesBackupProtectionPolicy cmdlet 创建用于保存备份策略信息的 PowerShell 对象。该备份策略用作 Enable-AzureRmRecoveryServicesBackupProtection cmdlet 的输入。

	PS C:\> $schPol = Get-AzureRmRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
	PS C:\>  $retPol = Get-AzureRmRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
	PS C:\>  New-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -WorkloadType AzureVM -RetentionPolicy $retPol -SchedulePolicy $schPol
	Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
	----                 ------------       -------------------- ----------                ----------
	NewPolicy           AzureVM            AzureVM              4/24/2016 1:30:00 AM

### 启用保护

启用保护涉及两个对象 - 项和策略。需要提供这两个对象才能为保管库启用保护。将策略与保管库关联之后，将在策略计划中定义的时间触发备份工作流。

在非加密型 ARM VM 上启用保护

	PS C:\> $pol=Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
	PS C:\> Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"

若要在加密型 VM 上启用保护[使用 BEK 和 KEK 加密]，需提供相应权限，允许 Azure 备份服务读取密钥保管库中的密钥和机密。

	PS C:\> Set-AzureRmKeyVaultAccessPolicy -VaultName 'KeyVaultName' -ResourceGroupName 'RGNameOfKeyVault' -PermissionsToKeys backup,get,list -PermissionsToSecrets get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
	PS C:\> $pol=Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
	PS C:\> Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"

对于基于 ASM 的 VM

	PS C:\>  $pol=Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
	PS C:\>  Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V1VM" -ServiceName "ServiceName1"

### 修改保护策略

若要修改策略，请修改 BackupSchedulePolicyObject 或 BackupRetentionPolicy 对象，并使用 Set-AzureRmRecoveryServicesBackupProtectionPolicy 修改策略

以下示例将保留计数更改为 365。

	PS C:\> $retPol = Get-AzureRmRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
	PS C:\> $retPol.DailySchedule.DurationCountInDays = 365
	PS C:\> $pol= Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name NewPolicy
	PS C:\> Set-AzureRmRecoveryServicesBackupProtectionPolicy -Policy $pol  -RetentionPolicy  $RetPol

## 在虚拟机中安装 VM 代理

Azure VM 代理必须安装在 Azure 虚拟机上，备份扩展才能运行。如果 VM 创建自 Azure 资源库，则 VM 代理已存在于虚拟机上。此处提供的信息适用于不是使用从 Azure 映像库创建的 VM 的情况（例如，从本地数据中心迁移的 VM）。在这种情况下，需要安装 VM 代理才能保护虚拟机。

了解 [VM 代理](https://go.microsoft.com/fwLink/?LinkID=390493&clcid=0x409)以及[如何安装 VM 代理](/documentation/articles/virtual-machines-windows-classic-manage-extensions/)。

如果在备份 Azure VM 时遇到问题，请先检查是否已在虚拟机上正确安装 Azure VM 代理（请参阅下表）。如果你创建了自定义 VM，[请先确保已选中“安装 VM 代理”复选框](/documentation/articles/virtual-machines-windows-classic-agents-and-extensions/)，然后再预配虚拟机。

下表提供了适用于 Windows 和 Linux VM 的 VM 代理的其他信息。

| **操作** | **Windows** | **Linux** |
| --- | --- | --- |
| 安装 VM 代理 | <li>下载并安装[代理 MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。你需要有管理员权限才能完成安装。<li>[更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，指明已安装代理。 | <li>从 GitHub 安装最新的 [Linux 代理](https://github.com/Azure/WALinuxAgent)。你需要有管理员权限才能完成安装。<li>[更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，指明已安装代理。 |
| 更新 VM 代理 | 更新 VM 代理与重新安装 [VM 代理二进制文件](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)一样简单。<br>确保在更新 VM 代理时，没有任何正在运行的备份操作。 | 按照[更新 Linux VM 代理](/documentation/articles/virtual-machines-linux-update-agent/)上的说明进行操作。<br>确保在更新 VM 代理时，没有任何正在运行的备份操作。 |
| 验证 VM 代理安装 | <li>导航到 Azure VM 中的 C:\\WindowsAzure\\Packages 文件夹。<li>你应会发现 WaAppAgent.exe 文件已存在。<li> 右键单击该文件，转到“属性”，然后选择“详细信息”选项卡。“产品版本”字段应为 2.6.1198.718 或更高。 | 不适用 |


### 备份扩展

在虚拟机上安装 VM 代理后，Azure 备份服务会将备份扩展安装到 VM 代理上。Azure 备份服务会无缝地升级和修补备份扩展。

无论 VM 是否在运行，备份服务都安装备份扩展。VM 运行时，很有可能会获得应用程序一致的恢复点。但是，即使 VM 已关闭并且无法安装扩展，Azure 备份服务也会继续备份 VM。这被称为脱机 VM。在这种情况下，恢复点将是崩溃一致性恢复点。


## 网络连接

为了管理 VM 快照，备份扩展需要连接 Azure 公共 IP 地址。如果未建立适当的 Internet 连接，虚拟机的 HTTP 请求将会超时，并且备份操作将会失败。如果你的部署中配置了访问限制（如通过网络安全组 (NSG)），请选择其中一个选项来提供备份流量的明确路径：

- [Whitelist the Azure datacenter IP ranges（将 Azure 数据中心 IP 范围加入白名单）](http://www.microsoft.com/zh-cn/download/details.aspx?id=41653)- 请参阅相关文章以获取有关如何将 IP 地址加入白名单的说明。
- 部署 HTTP 代理服务器来路由流量。

在确定使用哪个选项时，要取舍的不外乎是易管理性、控制粒度和成本等要素。

|选项|优点|缺点|
|------|----------|-------------|
|将 IP 范围加入白名单| 无额外成本<br><br>若要在 NSG 中打开访问权限，请使用 <i>Set-AzureNetworkSecurityRule</i> cmdlet | 管理起来很复杂，因为受影响的 IP 范围会不断变化。<br><br>允许访问整个 Azure，而不只是存储空间。|
|HTTP 代理| 通过允许的存储 URL 在代理中进行精细控制；<br>对 VM 进行单点 Internet 访问；<br>不受 Azure IP 地址变化的影响| 通过代理软件运行 VM 带来的额外成本。|

### 将 Azure 数据中心 IP 范围加入白名单

若要将 Azure 数据中心 IP 范围加入白名单，请参阅 [Azure website（Azure 网站）](http://www.microsoft.com/zh-cn/download/details.aspx?id=41653)以获取有关 IP 范围的详细信息和说明。

### 使用 HTTP 代理进行 VM 备份
备份 VM 时，VM 上的备份扩展会使用 HTTPS API 将快照管理命令发送到 Azure 存储空间。将通过 HTTP 代理路由备份扩展流量，因为它是为了访问公共 Internet 而配置的唯一组件。

>[AZURE.NOTE] 至于应该使用何种代理软件，我们不提供任何建议。请确保你选取的代理可以进行下述配置步骤。

以下示例图像显示了使用 HTTP 代理所要执行的三个配置步骤：

- 应用程序 VM 通过代理 VM 路由所有发往公共 Internet 的 HTTP 流量。
- 代理 VM 允许来自虚拟网络中 VM 的传入流量。
- 名为 NSF-lockdown 的网络安全组 (NSG) 需要一个安全规则来允许代理 VM 的出站 Internet 流量。

![NSG 与 HTTP 代理部署图](./media/backup-azure-vms-prepare/nsg-with-http-proxy.png)

若要使用 HTTP 代理来与公共 Internet 通信，请遵循以下步骤：

#### 步骤 1。配置传出网络连接

###### 对于 Windows 计算机
将设置本地系统帐户的代理服务器配置。

1. 下载 [PsExec](https://technet.microsoft.com/zh-cn/sysinternals/bb897553)
2. 在权限提升的提示符下运行以下命令：

        psexec -i -s "c:\Program Files\Internet Explorer\iexplore.exe"

     该命令将打开 Internet Explorer 窗口。
3. 转到“工具”->“Internet 选项”->“连接”->“LAN 设置”。
4. 验证系统帐户的代理设置。设置代理 IP 和端口。
5. 关闭 Internet Explorer。

这样就会设置计算机范围的代理配置，可以用于任何传出的 HTTP/HTTPS 流量。
   
如果已在当前用户帐户（非本地系统帐户）中设置代理服务器，请使用以下脚本将设置应用到 SYSTEMACCOUNT：

    $obj = Get-ItemProperty -Path Registry::”HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections"
    Set-ItemProperty -Path Registry::”HKEY_USERS\S-1-5-18\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections" -Name DefaultConnectionSettings -Value $obj.DefaultConnectionSettings
    Set-ItemProperty -Path Registry::”HKEY_USERS\S-1-5-18\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections" -Name SavedLegacySettings -Value $obj.SavedLegacySettings
    $obj = Get-ItemProperty -Path Registry::”HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
    Set-ItemProperty -Path Registry::”HKEY_USERS\S-1-5-18\Software\Microsoft\Windows\CurrentVersion\Internet Settings" -Name ProxyEnable -Value $obj.ProxyEnable
    Set-ItemProperty -Path Registry::”HKEY_USERS\S-1-5-18\Software\Microsoft\Windows\CurrentVersion\Internet Settings" -Name Proxyserver -Value $obj.Proxyserver

>[AZURE.NOTE] 如果在代理服务器日志中发现“(407)需要代理身份验证”，请检查身份验证设置是否正确。

###### 对于 Linux 计算机 

将以下代码行添加到 ```/etc/environment``` 文件：

    http_proxy=http://<proxy IP>:<proxy port>

将以下代码行添加到 ```/etc/waagent.conf``` 文件：

    HttpProxy.Host=<proxy IP>
    HttpProxy.Port=<proxy port>

#### 步骤 2.在代理服务器上允许传入连接：

1. 在代理服务器上打开 Windows 防火墙。访问防火墙的最简单方法搜索“具有高级安全性的 Windows 防火墙”。

    ![打开防火墙](./media/backup-azure-vms-prepare/firewall-01.png)

2. 在“Windows 防火墙”对话框中，右键单击“入站规则”，然后单击“新建规则...”。

    ![创建新规则](./media/backup-azure-vms-prepare/firewall-02.png)

3. 在“新建入站规则向导”中针对“规则类型”选择“自定义”选项，然后单击“下一步”。
4. 若要在页面上选择“程序”，可先选择“所有程序”，然后单击“下一步”。

5. 在“协议和端口”页面上输入以下信息，然后单击“下一步”：

    ![创建新规则](./media/backup-azure-vms-prepare/firewall-03.png)

    - 对于“协议类型”，请选择“TCP”
    - 对于“本地端口”，请选择“特定端口”，然后在下面的字段中指定已配置的 ```<Proxy Port>```。
    - 对于“远程端口”，请选择“所有端口”

    至于该向导的其余部分，可一路单击到最后，然后为此规则指定一个名称。

#### 步骤 3.向 NSG 添加例外规则：

在 Azure PowerShell 命令提示符下输入以下命令：

以下命令将在 NSG 中添加一个例外。此例外允许从 10.0.0.5 上的任何端口流向端口 80 (HTTP) 或 443 (HTTPS) 上的任何 Internet 地址的 TCP 流量。如果你需要访问公共 Internet 中的特定端口，请确保也将该端口添加到 ```-DestinationPortRange```。

    Get-AzureNetworkSecurityGroup -Name "NSG-lockdown" |
    Set-AzureNetworkSecurityRule -Name "allow-proxy " -Action Allow -Protocol TCP -Type Outbound -Priority 200 -SourceAddressPrefix "10.0.0.5/32" -SourcePortRange "*" -DestinationAddressPrefix Internet -DestinationPortRange "80-443"


这些步骤使用本示例中的特定名称和值。在输入或者将详细信息剪切并粘贴到代码中时，请使用部署的名称和值。


确定已建立网络连接后，可以开始备份 VM 了。请参阅[备份 Resource Manager 部署的 VM](/documentation/articles/backup-azure-arm-vms/)。

## 有疑问？
如果你有疑问，或者希望包含某种功能，请[给我们反馈](http://aka.ms/azurebackup_feedback)。

## 后续步骤
现在你已准备好环境来备份 VM，下一个逻辑步骤是创建备份。规划文章提供了有关备份 VM 的更详细信息。

- [备份虚拟机](/documentation/articles/backup-azure-vms/)
- [计划 VM 备份基础结构](/documentation/articles/backup-azure-vms-introduction/)
- [管理虚拟机备份](/documentation/articles/backup-azure-manage-vms/)

<!---HONumber=Mooncake_0711_2016-->