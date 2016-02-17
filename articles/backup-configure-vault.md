<properties
	pageTitle="配置 Azure 备份服务以准备备份 Windows Server | Windows Azure"
	description="使用本教程可了解如何在 Microsoft 的 Azure 云产品/服务中使用备份服务来将 Windows Server 备份到云中。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="backup"
	ms.date="11/26/2015"
	wacn.date="12/17/2015"/>

# 进行备份 Windows 计算机所需的环境准备

本文将引导你完成预备步骤，以便能够在 Windows Server 上使用 Azure 备份。若要将 Windows Server 或 Windows 客户端备份到 Azure，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](/pricing/free-trial/)。
>[AZURE.NOTE]以前，若要注册备份服务器，你需要创建或获取一个 X.509 v3 证书。证书目前仍受支持，但是，为了简化将服务器注册到 Azure 保管库，你现在可以直接通过“快速启动”页生成保管库凭据。

## 开始之前
若要将 Windows Server 中的文件和数据备份到 Azure，你首先必须：

- **创建备份保管库** — 在 [Azure 备份管理门户](http://manage.windowsazure.cn)中创建一个保管库。
- **下载保管库凭据** — 从 Azure 备份保管库中的“仪表板”页下载保管库凭据，用于将 Windows 计算机注册到备份保管库。
- **安装 Azure 备份代理并注册服务器** — 在“仪表板”页中，单击用于下载 [Azure 备份代理](http://aka.ms/azurebackup_agent)的链接。安装该代理，然后使用保管库凭据将服务器注册到备份保管库。

[AZURE.INCLUDE [backup-create-vault-wgif](../includes/backup-create-vault-wgif.md)]

[AZURE.INCLUDE [backup-download-credentials](../includes/backup-download-credentials.md)]

[AZURE.INCLUDE [backup-install-agent](../includes/backup-install-agent.md)]

## 后续步骤
- [备份 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-backup-windows-server)
- [管理 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-manage-windows-server)
- [从 Azure 还原 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-restore-windows-server)
- [Azure 备份常见问题](/documentation/articles/backup-azure-backup-faq)
- [Azure 备份论坛](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=windowsazureonlinebackup)

<!---HONumber=Mooncake_0104_2016-->