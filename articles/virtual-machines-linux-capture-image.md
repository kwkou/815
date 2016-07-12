<!-- ARM: tested -->

<properties
	pageTitle="捕获 Linux VM 以用作模板 | Azure"
	description="了解如何捕获使用 Azure 资源管理器部署模型创建的、基于 Linux 的 Azure 虚拟机 (VM) 的映像。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/15/2016"
	wacn.date="06/29/2016"/>


# 如何捕获 Linux 虚拟机以用作资源管理器模板

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代[经典部署模型](/documentation/articles/virtual-machines-linux-classic-capture-image/)。


使用 Azure 命令行界面 (CLI) 来捕获运行 Linux 的 Azure 虚拟机，以便你可以将其用作 Azure 资源管理器模板来创建其他虚拟机。此模板包括操作系统磁盘和附加到虚拟机的数据磁盘，但不包括创建 Azure 资源管理器 VM 所需的虚拟网络资源，因此大多数情况下，你需要在创建另一个使用此模板的虚拟机之前，先单独设定。

## 开始之前

这些步骤假定你已使用 Azure 资源管理器部署模型创建了 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘和完成其他自定义事项（如安装应用程序）。你可以通过几种方式实现，如 Azure CLI。如果尚未执行此操作，请参阅以下在 Azure 资源管理器模式下使用 Azure CLI 的说明:

- [使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)

例如，可以在中国北部地区创建一个名为 *MyResourceGroup* 的资源组。然后，使用类似于下方指令的 **azure vm quick-create** 命令在资源组中部署 Ubuntu 14.04 LTS VM。

 	azure vm quick-create -g MyResourceGroup -n <your-virtual-machine-name> "chinanorth" -y Linux -Q canonical:ubuntuserver:14.04.2-LTS:latest -u <your-user-name> -p <your-password>

VM 预配完成并运行后，你可能想要连接和安装数据磁盘。请参阅[此处](/documentation/articles/virtual-machines-linux-add-disk/)的说明。


## 捕获 VM

1. 若准备就绪可捕获 VM，使用 SSH 客户端连接到该 VM。

2. 在 SSH 窗口中，键入以下命令。请注意，**waagent** 的输出结果可能会因此实用程序的版本而略有差异：

	`sudo waagent -deprovision+user`

	此命令将尝试清除系统并使其适用于重新预配。此操作执行以下任务：

	- 删除 SSH 主机密钥（如果在配置文件中 Provisioning.RegenerateSshHostKeyPair 为“y”）
	- 清除 /etc/resolv.conf 中的 nameserver 配置
	- 从 /etc/shadow 中删除 `root` 用户的密码（如果在配置文件中 Provisioning.DeleteRootPassword 为“y”）
	- 删除缓存的 DHCP 客户端租赁
	- 将主机名重置为 localhost.localdomain
	- 删除上次预配的用户帐户（从 /var/lib/waagent 获得）和关联数据。

	>[AZURE.NOTE]取消预配会删除文件和数据，目的是使映像“一般化”。仅在要捕获为映像的 VM 上运行此命令。无法确保映像中的所有敏感信息均已清除，或者说无法确保该映像适合再分发给第三方。

3. 键入 **y** 继续。添加 **-force** 参数即可免除此确认步骤。

4. 键入 **exit** 关闭 SSH 客户端。

	>[AZURE.NOTE]后续步骤假定你已在客户端计算机上[安装 Azure CLI](/documentation/articles/xplat-cli-install/)。

5. 从客户端计算机中打开 Azure CLI 并登录到你的 Azure 订阅。有关详细信息，请阅读[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)。

6. 请确保你在资源管理器模式下：

	`azure config mode arm`

7. 停止你已使用以下命令取消预配的 VM：

	`azure vm stop –g <your-resource-group-name> -n <your-virtual-machine-name>`

8. 使用以下命令一般化 VM：

	`azure vm generalize –g <your-resource-group-name> -n <your-virtual-machine-name>`

9. 现在使用以下命令捕获映像和本地文件模板：

	`azure vm capture <your-resource-group-name> <your-virtual-machine-name> <your-vhd-name-prefix> -t <your-template-file-name.json>`

	此命令使用你为 VM 磁盘指定的 VHD 名称前缀创建通用 OS 映像。默认情况下，映像 VHD 文件在原始 VM 所用的相同存储帐户中创建。**-t** 选项创建可用于从映像创建新 VM 的本地 JSON 文件模板。

>[AZURE.TIP]若要查找映像的位置，请打开 JSON 文件模板。在 **storageProfile** 中，查找**系统**容器中**映像**的 **uri**。例如，OS 磁盘映像的 uri 类似于 `https://clixxxxxxxxxxxxxxxxxxxx.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/vhds/your-prefix-osDisk.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd`。

## 从捕获的映像部署新的 VM
现在通过模板使用映像创建新的 Linux VM。下列步骤演示如何使用 Azure CLI 和通过 `azure vm capture` 命令创建的 JSON 文件模板在新的虚拟网络中创建 VM。

### 创建网络资源

要使用模板，首先需要为新的 VM 设置虚拟网络和 NIC。我们建议你为这些资源创建新的资源组。运行与下面类似的命令，为你的资源替换名称和相应的 Azure 位置（这些命令中的 "chinanorth"）：

	azure group create <your-new-resource-group-name> -l "chinanorth"

	azure network vnet create <your-new-resource-group-name> <your-vnet-name> -l "chinanorth"

	azure network vnet subnet create <your-new-resource-group-name> <your-vnet-name> <your-subnet-name>

	azure network public-ip create <your-new-resource-group-name> <your-ip-name> -l "chinanorth"

	azure network nic create <your-new-resource-group-name> <your-nic-name> -k <your-subnetname> -m <your-vnet-name> -p <your-ip-name> -l "chinanorth"

要使用在捕获过程中保存的 JSON 从映像部署 VM，你需要 NIC 的 Id。通过运行以下命令获取它。

	azure network nic show <your-new-resource-group-name> <your-nic-name>

输出中的 **Id** 是与此类似的一个字符串。

	/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/<your-new-resource-group-name>/providers/Microsoft.Network/networkInterfaces/<your-nic-name>



### 新建部署
现在运行以下命令，从捕获的 VM 映像和你保存的模板 JSON 文件创建 VM。

	azure group deployment create <your-new-resource-group-name> <your-new-deployment-name> -f <your-template-file-name.json>

系统将提示你提供新的 VM 名称、管理员用户名和密码，以及你之前创建的 NIC 的 Id。

	info:    Executing command group deployment create
	info:    Supply values for the following parameters
	vmName: mynewvm
	adminUserName: myadminuser
	adminPassword: ********
	networkInterfaceId: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resource Groups/mynewrg/providers/Microsoft.Network/networkInterfaces/mynewnic

如果部署成功，你将看到与以下内容类似的输出。

	+ Initializing template configurations and parameters
	+ Creating a deployment
	info:    Created template deployment "dlnewdeploy"
	+ Waiting for deployment to complete
	data:    DeploymentName     : mynewdeploy
	data:    ResourceGroupName  : mynewrg
	data:    ProvisioningState  : Succeeded
	data:    Timestamp          : 2015-10-29T16:35:47.3419991Z
	data:    Mode               : Incremental
	data:    Name                Type          Value


	data:    ------------------  ------------  -------------------------------------

	data:    vmName              String        mynewvm


	data:    vmSize              String        Standard_D1


	data:    adminUserName       String        myadminuser


	data:    adminPassword       SecureString  undefined


	data:    networkInterfaceId  String        /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/mynewrg/providers/Microsoft.Network/networkInterfaces/mynewnic
	info:    group deployment create command OK

### 验证部署

现在将 SSH 连接到你创建的虚拟机以验证部署并开始使用新的 VM。若要通过 SSH 连接，找到通过运行以下命令创建的 VM 的 IP 地址：

	azure network public-ip show <your-new-resource-group-name> <your-ip-name>

公共 IP 地址在命令输出中列出。默认情况下你通过 SSH 在端口 22 上连接到 Linux VM。

## 使用模板创建更多 VM

使用捕获的映像和模板按照前面部分所述的步骤部署更多 VM。

* 确保你的 VM 映像位于将托管 VM 之 VHD 的存储帐户中
* 复制模板 JSON 文件并为每个 VM 之 VHD 的 **uri** 输入一个唯一值
* 在相同或不同的虚拟网络中新建 NIC
* 使用修改后的模板 JSON 文件在你设置虚拟网络的资源组中创建部署

如果你希望网络在你从映像创建 VM 时自动设置，请从 GitHub 使用 [101-vm-from-user-image template](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)。此模板会从你的自定义映像创建 VM 以及必要的虚拟网络、公共 IP 地址和 NIC 资源。

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

## 使用 azure vm create 命令

通常需要使用资源管理器模板从映像创建 VM。但是，您可以使用带 **--os-disk-vhd** (**-d**) 参数的 **azure vm create** 命令_强制_创建 VM。

通过映像运行 **azure vm create** 之前，执行下列操作:

1.	针对部署新建资源组或确定一个现有的资源组。

2.	为新的 VM 创建公共 IP 地址资源和 NIC 资源。有关使用 CLI 创建虚拟网络、公共 IP 地址和 NIC 的步骤，请参阅本文前面的内容。（**azure vm create** 也可以新建 NIC，但你需要为虚拟网络和子网传递其他参数。）

3.	确保将映像 VHD 复制到没有文件夹的 blob 容器位置（虚拟目录）。默认情况下捕获的映像存储在存储 blob 容器的嵌套文件夹中（URI 类似于 `https://clixxxxxxxxxxxxxxxxxxxx.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/vhds/your-prefix-osDisk.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd`）。**azure vm create** 命令当前只能从存储在 blob 容器顶层的操作系统磁盘 VHD 创建 VM。例如，你可能会将映像 VHD 复制到 `https://yourstorage.blob.core.chinacloudapi.cn/vhds/your-prefix-OsDisk.vhd`。

然后运行与下面类似的命令。

	azure vm create <your-resource-group-name> <your-new-vm-name> eastus Linux -o <your-storage-account-name> -d "https://yourstorage.blob.core.chinacloudapi.cn/vhds/your-prefix-OsDisk.vhd" -z Standard_A1 -u <your-admin-name> -p <your-admin-password> -f <your-nic-name>
	
对于其他命令选项，运行 `azure help vm create`。

## 后续步骤

要使用 CLI 管理 VM，请参阅[使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)中的任务。

<!---HONumber=Mooncake_1207_2015-->