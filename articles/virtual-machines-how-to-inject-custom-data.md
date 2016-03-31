<properties
	pageTitle="将自定义数据注入到虚拟机中 | Azure"
	description="本主题介绍如何在创建实例时将自定义数据注入到 Azure 虚拟机中，以及如何在 Windows 或 Linux 上找到自定义数据。"
	services="virtual-machines"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management" />

<tags
	ms.service="virtual-machines"
	ms.date="12/08/2015"
	wacn.date="01/29/2016"/>


#将自定义数据注入到 Azure 虚拟机中

无论操作系统是 Windows 还是 Linux 分发，在设置 Azure 虚拟机时，将脚本或其他数据注入到 Azure 虚拟机都是非常常见的方案。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]


本主题介绍如何执行以下操作：

- 在预配 Azure 虚拟机时，将数据注入到 Azure 虚拟机中。

- 针对 Windows 和 Linux 检索它。

- 使用某些系统上提供的特殊工具来检测和自动处理自定义数据。

> [AZURE.NOTE] 本文介绍了如何才能使用通过 Azure 服务管理 API 创建的 VM 注入自定义数据。若要了解如何使用 Azure 资源管理 API，请参阅[示例模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-customdata)。

## 将自定义数据注入到 Azure 虚拟机中

当前仅在 [Azure 命令行界面](https://github.com/Azure/azure-xplat-cli)上支持此功能。尽管你可以使用 `azure vm create` 命令的任何选项，但以下内容演示的是一个非常基本的方法。

	
	    PASSWORD='AcceptablePassword -- more than 8 chars, a cap, a num, a special'
	    VMNAME=mycustomdataubuntu
	    USERNAME=username
	    VMIMAGE= An image chosen from among those listed by azure vm image list
	    azure vm create $VMNAME $VMIMAGE $USERNAME $PASSWORD --location "China North" --json -d ./custom-data.txt -e 22
	


## 在虚拟机中使用自定义数据

+ 如果你的 Azure 虚拟机是基于 Windows 的虚拟机，则自定义数据文件将保存到 `%SYSTEMDRIVE%\AzureData\CustomData.bin`。虽然它已进行 Base64 编码，以便从本地计算机传输到新虚拟机，但它将自动解码并可以立即打开或使用。

   >[AZURE.NOTE] 如果该文件存在，它将被覆盖。目录的安全性已设为 **System:Full Control** 和 **Administrators:Full Control**。

+ 如果你的 Azure 虚拟机是基于 Linux 的虚拟机，则自定义数据文件将位于以下两个位置。数据将会是 base64 编码的，因此需要首先对数据进行解码。

    + 在 `/var/lib/waagent/ovf-env.xml` 上
    + 在 `/var/lib/waagent/CustomData` 上



## Azure 中的 Cloud-init

如果你的 Azure 虚拟机来自 Ubuntu 或 CoreOS 映像，则可以使用 CustomData 将 cloud-config 发送到 cloud-init。或者，如果你的自定义数据文件是一个脚本，则 cloud-init 只需执行它。

### Ubuntu 云映像

在大多数 Azure Linux 映像中，你将编辑“/etc/waagent.conf”来配置临时资源磁盘和交换文件。有关详细信息，请参阅 [Azure Linux 代理用户指南](/documentation/articles/virtual-machines-linux-agent-user-guide)。

但是，在 Ubuntu 云映像上，你必须使用 cloud-init 来配置资源磁盘（又称“临时”磁盘）和交换分区。有关更多详细信息，请参阅 Ubuntu wiki 上的以下页面：[AzureSwapPartitions](https://wiki.ubuntu.com/AzureSwapPartitions)。



<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤：使用 cloud-init

有关更多详细信息，请参阅[适用于 Ubuntu 的 cloud-init 文档](https://help.ubuntu.com/community/CloudInit)。

<!--Link references-->
[添加 Azure 角色服务管理 REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/jj157186.aspx)

[Azure 命令行界面](https://github.com/Azure/azure-xplat-cli)

<!---HONumber=Mooncake_0118_2016-->