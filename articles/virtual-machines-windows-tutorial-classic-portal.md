<properties
	pageTitle="创建在 Azure 中运行 Windows 的虚拟机"
	description="在 Azure 门户中创建 Windows 虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="08/11/2015"
	wacn.date="09/18/2015"/>

# 在 Azure 门户中创建运行 Windows 的虚拟机

> [AZURE.SELECTOR]
- [Azure portal](/documentation/articles/virtual-machines-windows-tutorial-classic-portal)
- [PowerShell: Resource Manager deployment](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)
- [PowerShell: Classic deployment](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)


本教程演示在 Azure 门户中创建 Azure 虚拟机 (VM) 是多么容易。我们将使用 Windows Server 映像作为示例，但这只是 Azure 提供的众多映像的其中一个。请注意，映像的选择取决于订阅。例如，桌面映像可能适用于 MSDN 订户。

你也可以使用[自己的映像](/documentation/articles/virtual-machines-create-upload-vhd-windows-server)创建 VM。若要了解此方法和其他方法，请参阅[创建 Windows 虚拟机的不同方式](/documentation/articles/virtual-machines-windows-choices-create-vm)。


## <a id="createvirtualmachine"> </a>如何创建虚拟机

本部分演示如何使用 Azure 门户中的“从库中”选项创建虚拟机。此选项提供的配置选项多于**“快速创建”**选项。例如，如果你想要将虚拟机加入到虚拟网络，则需要使用**“从库中”**选项。

> [AZURE.NOTE]你还可以尝试使用功能更丰富且可自定义的 [Azure 预览门户](https://manage.windowsazure.cn)来创建虚拟机、自动部署多 VM 应用程序模板、使用增强的 VM 监视和诊断功能，以及执行更多操作。这两个门户提供的 VM 配置选项基本上重叠，但并不完全相同。

[AZURE.INCLUDE [virtual-machines-create-WindowsVM](../includes/virtual-machines-create-windowsvm.md)]

## 后续步骤

- 登录到虚拟机。有关说明，请参阅[如何登录到运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-log-on-windows-server)。

- 附加磁盘以存储数据。你可以附加空磁盘和包含数据的磁盘。有关说明，请参阅[附加数据磁盘教程](/documentation/articles/storage-windows-attach-disk)。

## 其他资源

若要了解有关可为虚拟机配置哪些内容以及何时可以进行配置的详细信息，请参阅[关于 Azure VM 配置设置](http://msdn.microsoft.com/zh-cn/library/azure/dn763935.aspx)。

<!---HONumber=70-->