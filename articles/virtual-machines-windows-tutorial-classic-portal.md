<properties
	pageTitle="在管理门户中创建运行 Windows 的 VM | Azure"
	description="在 Azure 管理门户中创建 Windows 虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="01/06/2016"
	wacn.date="02/26/2016"/>

# 在 Azure 管理门户中创建运行 Windows 的虚拟机

> [AZURE.SELECTOR]
- [Azure 管理门户](/documentation/articles/virtual-machines-windows-tutorial-classic-portal)
- [PowerShell: 经典部署](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)

<!-- HHTML comment in to break between the selector and the note in the include below-->

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]

本教程演示在 Azure 管理门户中创建运行 Windows 的 Azure 虚拟机 (VM) 是多么容易。我们将使用 Windows Server 映像作为示例，但这只是 Azure 提供的众多映像的其中一个。

你也可以使用[自己的映像](/documentation/articles/virtual-machines-create-upload-vhd-windows-server)创建 VM。若要了解此方法和其他方法，请参阅[创建 Windows 虚拟机的不同方式](/documentation/articles/virtual-machines-windows-choices-create-vm)。

[AZURE.INCLUDE [free-trial-note](../includes/free-trial-note.md)]

## <a id="createvirtualmachine"> </a>如何创建虚拟机

本部分演示如何使用 Azure 管理门户中的**“从库中”**选项创建虚拟机。此选项提供的配置选项多于**“快速创建”**选项。例如，如果你想要将虚拟机加入到虚拟网络，则需要使用**“从库中”**选项。

[AZURE.INCLUDE [virtual-machines-create-WindowsVM](../includes/virtual-machines-create-windowsvm.md)]

## 后续步骤

- 登录到虚拟机。有关说明，请参阅[登录到运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-log-on-windows-server)。

- 附加磁盘以存储数据。你可以附加空磁盘和包含数据的磁盘。有关说明，请参阅[将数据磁盘附加到使用经典部署模型创建的 Windows 虚拟机](/documentation/articles/storage-windows-attach-disk)。

<!---HONumber=Mooncake_0215_2016-->