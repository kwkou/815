<properties
    pageTitle="使用 Azure 备份还原已加密 VM 的密钥保管库密钥和机密 | Azure"
    description="了解如何使用 PowerShell 在 Azure 备份中还原密钥保管库密钥和机密"
    services="backup"
    documentationcenter=""
    author="JPallavi"
    manager="vijayts"
    editor="" />  

<tags
    ms.assetid="45214083-d5fc-4eb3-a367-0239dc59e0f6"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/18/2016"
    wacn.date="12/21/2016"
    ms.author="JPallavi" />  


# 使用 Azure 备份还原已加密 VM 的密钥保管库密钥和机密
本文介绍在密钥和机密不存在于密钥保管库中的情况下，如何使用 Azure VM 备份对加密的 Azure VM 进行还原。若需为还原的 VM 保留单独的密钥（密钥加密密钥）和机密（BitLocker 加密密钥）副本，则也可使用这些步骤。

## 先决条件
1. **备份加密的 VM** - 已使 Azure 备份备份加密的 Azure VM。有关如何备份已加密 Azure VM 的详细信息，请参阅[使用 PowerShell 管理 Azure VM 的备份和还原](/documentation/articles/backup-azure-vms-automation/)一文。
2. **配置 Azure 密钥保管库** - 确保需将密钥和机密还原到其中的密钥保管库已存在。有关密钥保管库管理的详细信息，请参阅 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)一文。

## 设置恢复服务保管库
使用以下步骤登录到 PowerShell 并设置恢复服务保管库上下文

### 登录到 Azure PowerShell
使用以下 cmdlet 登录到 Azure 帐户


	PS C:\> Login-AzureRmAccount -EnvironmentName AzureChinaCloud


### 设置恢复服务保管库上下文
登录后，使用以下 cmdlet 获取可用订阅的列表


	PS C:\> Get-AzureRmSubscription


选择具有可用资源的订阅


	PS C:\> Set-AzureRmContext -SubscriptionId "<subscription-id>"


使用已在其中为已加密 VM 启用备份的恢复服务保管库设置保管库上下文


	PS C:\> Get-AzureRmRecoveryServicesVault -ResourceGroupName "<rg-name>" -Name "<rs-vault-name>" | Set-AzureRmRecoveryServicesVaultContext


### 获取恢复点
在保管库中选择代表已加密 Azure 虚拟机的容器


	PS C:\> $namedContainer = Get-AzureRmRecoveryServicesBackupContainer -ContainerType "AzureVM" -Status "Registered" -Name "<vm-name>"


使用此容器获取相应虚拟机的备份项


	PS C:\> $backupitem = Get-AzureRmRecoveryServicesBackupItem -Container $namedContainer -WorkloadType "AzureVM"


获取变量 rp 中所选备份项的恢复点数组


	PS C:\> $startDate = (Get-Date).AddDays(-7)
	PS C:\> $endDate = Get-Date
	PS C:\> $rp = Get-AzureRmRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()

## 还原加密的虚拟机
使用以下步骤还原加密的 VM 及其密钥和机密。

### 还原密钥
上面的数组 $rp 按时间进行反向排序，以最新的恢复点作为索引 0。例如：$rp[0] 选择最新的恢复点。


	PS C:\> $rp1 = Get-AzureRmRecoveryServicesBackupRecoveryPoint -RecoveryPointId $rp[0].RecoveryPointId -Item $backupItem -KeyFileDownloadLocation "C:\Users\downloads"


> [AZURE.NOTE]
此 cmdlet 成功运行以后，会在运行它的计算机的指定文件夹中生成一个 Blob 文件。此 Blob 文件表示加密形式的密钥加密密钥。
> 
> 

使用以下 cmdlet 将密钥还原到密钥保管库。


	PS C:\> Restore-AzureKeyVaultKey -VaultName "contosokeyvault" -InputFile "C:\Users\downloads\key.blob"


### 还原机密
从上面获得的恢复点还原机密数据


	PS C:\> $rp1.KeyAndSecretDetails.SecretUrl

	https://contosokeyvault.vault.chinacloudapi.cn/secrets/B3284AAA-DAAA-4AAA-B393-60CAA848AAAA/20aaae9eaa99996d89d99a29990d999a


> [AZURE.NOTE]
vault.chinacloudapi.cn 之前的文本表示原始密钥保管库名称。secrets/ 之后的文本表示机密名称。
> 
> 

若需使用同一机密名称，请从上面运行的 cmdlet 的输出中获取机密名称和值。否则，应对下面的 $secretname 进行更新，以便使用新的机密名称。


	PS C:\> $secretname = "B3284AAA-DAAA-4AAA-B393-60CAA848AAAA"
	PS C:\> $secretdata = $rp1.KeyAndSecretDetails.SecretData
	PS C:\> $Secret = ConvertTo-SecureString -String $secretdata -AsPlainText -Force


如果还需还原 VM，则请为机密设置标记。对于标记 DiskEncryptionKeyFileName，值应包含计划使用的机密的名称。


	PS C:\> $Tags = @{'DiskEncryptionKeyEncryptionAlgorithm' = 'RSA-OAEP';'DiskEncryptionKeyFileName' = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA.BEK';'DiskEncryptionKeyEncryptionKeyURL' = 'https://contosokeyvault.vault.chinacloudapi.cn:443/keys/KeyName/84daaac999949999030bf99aaa5a9f9';'MachineName' = 'vm-name'}


> [AZURE.NOTE]
DiskEncryptionKeyFileName 的值与上面获得的机密名称相同。还原密钥并使用 [Get-AzureKeyVaultKey](https://msdn.microsoft.com/zh-cn/library/dn868053.aspx) cmdlet 后，即可从密钥保管库获得 DiskEncryptionKeyEncryptionKeyURL 的值
> 
> 

将机密重新设置到密钥保管库中


	PS C:\> Set-AzureKeyVaultSecret -VaultName "contosokeyvault" -Name $secretname -SecretValue $secret -Tags $Tags -SecretValue $Secret -ContentType  "Wrapped BEK"


### 恢复虚拟机
如果已使用 Azure VM 备份备份已加密 VM，则可通过上面的 PowerShell cmdlet 将密钥和机密重新还原到密钥保管库中。还原密钥和机密以后，请参阅[使用 PowerShell 管理 Azure VM 的备份和还原](/documentation/articles/backup-azure-vms-automation/)一文，了解如何还原已加密 VM。

<!---HONumber=Mooncake_1212_2016-->
