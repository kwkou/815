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
	ms.date="08/21/2015"
	wacn.date="11/02/2015"/>

# 配置 Microsoft Azure 备份以准备备份 Windows Server

本文将指导你完成启用 Azure 备份功能的整个过程。若要将 Windows Server 或 Windows 客户端备份到 Azure，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。
>[AZURE.NOTE]以前，若要注册备份服务器，你需要创建或获取一个 X.509 v3 证书。证书目前仍受支持，但是，为了简化将服务器注册到 Azure 保管库，你现在可以直接通过“快速启动”页生成保管库凭据。

## 开始之前
若要将 Windows Server 中的文件和数据备份到 Azure，你首先必须：

- **创建备份保管库** — 在 Azure 备份控制台中创建一个保管库。
- **下载保管库凭据** — 在 Azure 备份中，将你创建的管理证书上载到保管库。
- **安装 Azure 备份代理并注册服务器** — 通过 Azure 备份安装代理，并在备份保管库中注册服务器。

[AZURE.INCLUDE [backup-create-vault](../includes/backup-create-vault.md)]

[AZURE.INCLUDE [backup-download-credentials](../includes/backup-download-credentials.md)]

[AZURE.INCLUDE [backup-install-agent](../includes/backup-install-agent.md)]

## 后续步骤
- [备份 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-backup-windows-server)
- [管理 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-manage-windows-server)
- [从 Azure 还原 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-restore-windows-server)
- [Azure 备份常见问题](/documentation/articles/backup-azure-backup-faq)
- [Azure 备份论坛](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=windowsazureonlinebackup)

<!---HONumber=76-->