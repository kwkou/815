<properties
   pageTitle="监视 Resource Manager 部署型虚拟机备份 | Azure"
   description="监视 Resource Manager 部署型虚拟机备份生成的事件和警报"
   services="backup"
   documentationCenter="dev-center-name"
   authors="markgalioto"
   manager="cfreeman"
   editor=""/>

<tags
	ms.service="backup"
	ms.date="08/11/2016"
	wacn.date="11/11/2016"/>  


# 监视 Azure 虚拟机备份的警报

目前，在 Azure 中国，暂时还不能通过 Azure 门户预览来监控 Azure 虚拟机备份。不过你可以通过 PowerShell 自定义警报。

## 使用 PowerShell 自定义警报
你可以获取门户中作业的自定义警报通知。若要获取这些作业，请针对操作日志事件定义基于 PowerShell 的警报规则。使用 *PowerShell 1.3.0 或更高版本*。

若要定义自定义通知以便在备份失败时发出警报，可使用类似于以下脚本的命令：

```
PS C:\> $actionEmail = New-AzureRmAlertRuleEmail -CustomEmail contoso@microsoft.com
PS C:\> Add-AzureRmLogAlertRule -Name backupFailedAlert -Location "China East" -ResourceGroup RecoveryServices-DP2RCXUGWS3MLJF4LKPI3A3OMJ2DI4SRJK6HIJH22HFIHZVVELRQ-East-US -OperationName Microsoft.Backup/RecoveryServicesVault/Backup -Status Failed -TargetResourceId /subscriptions/86eeac34-eth9a-4de3-84db-7a27d121967e/resourceGroups/RecoveryServices-DP2RCXUGWS3MLJF4LKPI3A3OMJ2DI4SRJK6HIJH22HFIHZVVELRQ-East-US/providers/microsoft.backupbvtd2/RecoveryServicesVault/trinadhVault -Actions $actionEmail
```

**ResourceId**：可以从审核日志获取 ResourceId。ResourceId 是操作日志的“资源”列中提供的 URL。

**OperationName**：OperationName 采用“Microsoft.RecoveryServices/recoveryServicesVault/*EventName*”格式，其中的 *EventName* 可以是：<br/>
- Register <br/>
- Unregister <br/>
- ConfigureProtection <br/>
- Backup <br/>
- Restore <br/>
- StopProtection <br/>
- DeleteBackupData <br/>
- CreateProtectionPolicy <br/>
- DeleteProtectionPolicy <br/>
- UpdateProtectionPolicy <br/>

**Status**：支持的值为 Started、Succeeded 或 Failed。

**ResourceGroup**：这是资源所属的资源组。可以将“资源组”列添加到生成的日志。资源组是一种可用的事件信息类型。

**Name**：警报规则的名称。

**CustomEmail**：指定要向其发送警报通知的自定义电子邮件地址。

**SendToServiceOwners**：此选项会将警报通知发送给订阅的所有管理员和共同管理员。可以在 **New-AzureRmAlertRuleEmail** cmdlet 中使用

### 对警报的限制
基于事件的警报存在以下限制：

1. 警报在恢复服务保管库的所有虚拟机上触发。无法针对恢复服务保管库中的一部分虚拟机自定义警报。
2. 此功能以预览版提供。[了解详细信息](/documentation/articles/insights-powershell-samples/#create-alert-rules)
3. 警报从“alerts-noreply@mail.windowsazure.cn”发送。目前你无法修改电子邮件发件人。


## 后续步骤

通过事件日志，可以针对备份操作进行很好的事后总结和审核。将记录以下操作：

- 注册
- 注销
- 配置保护
- 备份（二者均可以按需备份的形式进行计划）
- 还原
- 停止保护
- 删除备份数据
- 添加策略
- 删除策略
- 更新策略
- 取消作业

有关 Azure 服务中事件、操作和审核日志的详细说明，请参阅 [View events and audit logs](/documentation/articles/insights-debugging-with-events/)（查看事件和审核日志）一文。

有关如何从恢复点重新创建虚拟机的信息，请查看 [Restore Azure VMs](/documentation/articles/backup-azure-restore-vms/)（还原 Azure VM）。如果你需要有关如何保护虚拟机的信息，请参阅 [First look: Back up VMs to a Recovery Services vault](/documentation/articles/backup-azure-vms-first-look/)（初步了解：将 VM 备份到恢复服务保管库）。

<!---HONumber=Mooncake_0822_2016-->