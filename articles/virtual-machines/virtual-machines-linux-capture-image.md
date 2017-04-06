<properties
    pageTitle="捕获用作模板的 Azure Linux VM | Azure"
    description="了解如何捕获并通用化使用 Azure Resource Manager 部署模型创建的、基于 Linux 的 Azure 虚拟机 (VM) 的映像。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="e608116f-f478-41be-b787-c2ad91b5a802"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/02/2017"
    wacn.date="03/20/2017"
    ms.author="iainfou" />  


# 捕获 Azure 上运行的 Linux 虚拟机
遵循本文中的步骤可在 Resource Manager 部署模型中通用化和捕获 Azure Linux 虚拟机 (VM)。通用化 VM 时，需删除个人帐户信息，并准备好要用作映像的 VM。然后捕获 OS 的通用化虚拟硬盘 (VHD) 映像、附加数据磁盘的 VHD 以及新 VM 部署的 [Resource Manager 模板](/documentation/articles/resource-group-overview/)。本文详述了如何使用适用于 VM 的 Azure CLI 1.0 捕获 VM 映像。

若要使用映像创建 VM，请为每个新 VM 设置网络资源，然后使用模板（JavaScript 对象表示法 (JSON) 文件）从捕获的 VHD 映像部署该 VM。这样，便可以如同使用 Azure 应用商店中的映像一样，复制该 VM 及其当前软件配置。

> [AZURE.TIP]
如果想要创建现有 Linux VM 的副本（包括其专用化备份或调试状态），请参阅[创建在 Azure 上运行的 Linux 虚拟机副本](/documentation/articles/virtual-machines-linux-copy-vm/)。如果想要从本地 VM 上载 Linux VHD，请参阅[上载自定义磁盘映像并从其创建 Linux VM](/documentation/articles/virtual-machines-linux-upload-vhd/)。

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](#before-you-begin)：用于经典部署模型和资源管理部署模型（本文）的 CLI
- Azure CLI 2.0 - 不支持 Azure 中国区的虚拟机。

## <a name="before-you-begin"></a> 准备工作
请确保符合以下先决条件：

* **在 Resource Manager 部署模型中创建的 Azure VM** - 如果尚未创建 Linux VM，可以使用[门户](/documentation/articles/virtual-machines-linux-quick-create-portal/)、[Azure CLI](/documentation/articles/virtual-machines-linux-quick-create-cli/) 或 [Resource Manager 模板](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)。
  
    根据需要配置 VM。例如，[添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)、应用更新，并安装应用程序。
* **Azure CLI** - 在本地计算机上安装 [Azure CLI](/documentation/articles/xplat-cli-install/)。

## 步骤 1：删除 Azure Linux 代理
首先，在 Linux VM 上结合 **deprovision** 参数运行 **waagent** 命令。此命令将删除文件与数据，使 VM 准备好通用化。有关详细信息，请参阅 [Azure Linux 代理用户指南](/documentation/articles/virtual-machines-linux-agent-user-guide/)。

1. 使用 SSH 客户端连接到 Linux VM。
2. 在 SSH 窗口中，键入以下命令：

        sudo waagent -deprovision+user

    > [AZURE.NOTE]
    仅在要捕获为映像的 VM 上运行此命令。不保证映像中的所有敏感信息被清除，或者映像适合用于分发。
 
3. 键入 **y** 继续。添加 **-force** 参数即可免除此确认步骤。
4. 完成该命令后，键入 **exit**。此步骤将关闭 SSH 客户端。

## <a name="capture-the-vm" id="step-2-capture-the-vm"></a> 步骤 2：捕获 VM
使用 Azure CLI 来通用化和捕获 VM。在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 **myResourceGroup**、**myVnet** 和 **myVM**。

1. 从本计算机打开 Azure CLI 并[登录到你的 Azure 订阅](/documentation/articles/xplat-cli-connect/)。
2. 请确保你处于 Resource Manager 模式。

        azure config mode arm

3. 使用以下命令关闭已取消预配的 VM：

        azure vm deallocate -g myResourceGroup -n myVM

4. 使用以下命令一般化 VM：

        azure vm generalize -g myResourceGroup -n myVM

5. 现在，运行 **azure vm capture** 命令来捕获 VM。以下示例将捕获名称以 **MyVHDNamePrefix** 开头的映像 VHD，其中，**-t** 选项指定模板 **MyTemplate.json** 的路径。

        azure vm capture -g myResourceGroup -n myVM -p myVHDNamePrefix -t myTemplate.json

    > [AZURE.IMPORTANT]
    默认情况下，映像 VHD 文件在原始 VM 所用的相同存储帐户中创建。使用*同一个存储帐户*来存储从映像创建的所有新 VM 的 VHD。

6. 若要查找捕获的映像的位置，请在文本编辑器中打开 JSON 模板。在 **storageProfile** 中，查找**系统**容器中**映像**的 **uri**。例如，OS 磁盘映像的 URI 类似于 `https://xxxxxxxxxxxxxx.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/vhds/MyVHDNamePrefix-osDisk.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd`

## 步骤 3：从捕获的映像创建 VM
现在，通过模板使用映像创建 Linux VM。下列步骤演示如何使用 Azure CLI 和捕获的 JSON 文件模板在新虚拟网络中创建 VM。

### 创建网络资源
要使用模板，首先需要为新的 VM 设置虚拟网络和 NIC。建议在 VM 映像的存储位置中，为这些资源创建一个资源组。运行类似于下面的命令，并替换资源名称和相应的 Azure 位置（这些命令中的“chinaeast”）：

    azure group create myResourceGroup1 -l "chinaeast"

    azure network vnet create myResourceGroup1 myVnet -l "chinaeast"

    azure network vnet subnet create myResourceGroup1 myVnet mySubnet

    azure network public-ip create myResourceGroup1 myPublicIP -l "chinaeast"

    azure network nic create myResourceGroup1 myNIC -k mySubnet -m myVnet -p myPublicIP -l "chinaeast"

### 获取 NIC 的 ID
若要使用在捕获期间保存的 JSON 从映像部署 VM，需要获取 NIC 的 ID。运行以下命令可获取该 ID：

    azure network nic show myResourceGroup1 myNIC

输出中的 **ID** 类似于 `/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MyResourceGroup1/providers/Microsoft.Network/networkInterfaces/myNic`

### 创建 VM
现在运行以下命令，从捕获的 VM 映像创建 VM。使用 **-f** 参数指定所保存的模板 JSON 文件的路径。

    azure group deployment create myResourceGroup1 MyDeployment -f MyTemplate.json

在命令输出中，系统将提示提供新 VM 名称、管理员用户名和密码，以及以前创建的 NIC 的 ID。

    info:    Executing command group deployment create
    info:    Supply values for the following parameters
    vmName: myNewVM
    adminUserName: myAdminuser
    adminPassword: ********
    networkInterfaceId: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resource Groups/myResourceGroup1/providers/Microsoft.Network/networkInterfaces/myNic

以下示例显示成功部署后看到的内容：

    + Initializing template configurations and parameters
    + Creating a deployment
    info:    Created template deployment xxxxxxx
    + Waiting for deployment to complete
    data:    DeploymentName     : MyDeployment
    data:    ResourceGroupName  : MyResourceGroup1
    data:    ProvisioningState  : Succeeded
    data:    Timestamp          : xxxxxxx
    data:    Mode               : Incremental
    data:    Name                Type          Value

    data:    ------------------  ------------  -------------------------------------

    data:    vmName              String        myNewVM

    data:    vmSize              String        Standard_D1

    data:    adminUserName       String        myAdminuser

    data:    adminPassword       SecureString  undefined

    data:    networkInterfaceId  String        /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MyResourceGroup1/providers/Microsoft.Network/networkInterfaces/myNic
    info:    group deployment create command OK

### 验证部署
现在将 SSH 连接到你创建的虚拟机以验证部署并开始使用新的 VM。若要通过 SSH 连接，找到通过运行以下命令创建的 VM 的 IP 地址：

    azure network public-ip show myResourceGroup1 myPublicIP

公共 IP 地址在命令输出中列出。默认情况下你通过 SSH 在端口 22 上连接到 Linux VM。

## 创建更多 VM
根据上一部分中所述的步骤，使用捕获的映像和模板来部署更多 VM。用于从映像创建 VM 的其他选项包括使用快速入门模板，或运行 **azure vm create** 命令。

### 使用捕获的模板
若要使用捕获的映像和模板，请遵循以下步骤（上一部分中有详细说明）：

* 确保 VM 映像位于托管 VM 的 VHD 的同一存储帐户中。
* 复制模板 JSON 文件，指定新 VM 的一个或多个 VHD 的 OS 磁盘的唯一名称。例如，在 **storageProfile** 中 **vhd** 下面的 **uri** 中，指定 **osDisk** VHD 的唯一名称，类似于 `https://xxxxxxxxxxxxxx.blob.core.chinacloudapi.cn/vhds/MyNewVHDNamePrefix-osDisk.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd`
* 在相同或不同的虚拟网络中创建 NIC。
* 使用修改后的模板 JSON 文件，在设置虚拟网络的资源组中创建部署。

### 使用快速入门模板
如果要在从映像创建 VM 时自动设置网络，可在模板中指定这些资源。有关示例，请参阅 GitHub 中的 [101-vm-from-user-image 模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)。此模板会从你的自定义映像创建 VM 以及必要的虚拟网络、公共 IP 地址和 NIC 资源。若要在 Azure 门户预览中演练如何使用该模板，请参阅 [How to create a virtual machine from a custom image using a Resource Manager template](http://codeisahighway.com/how-to-create-a-virtual-machine-from-a-custom-image-using-an-arm-template/)（如何使用 Resource Manager 模板从自定义映像创建虚拟机）。

### 使用 azure vm create 命令
通常，最简单的方法是使用 Resource Manager 模板从映像创建 VM。不过，也可以结合 **-Q** (**--image-urn**) 参数使用 **azure vm create** 命令*强制*创建 VM。如果使用此方法，则还要传递 **-d** (**--os-disk-vhd**) 参数来指定新 VM 的 OS .vhd 文件位置。此文件必须位于存储映像 VHD 文件的存储帐户的 vhds 容器中。该命令自动将新 VM 的 VHD 复制到 **vhds** 容器。

对映像运行 **azure vm create** 之前，请完成以下步骤：

1. 创建一个资源组或指定一个现有的资源组供部署。
2. 为新的 VM 创建公共 IP 地址资源和 NIC 资源。有关使用 CLI 创建虚拟网络、公共 IP 地址和 NIC 的步骤，请参阅本文前面的内容。（**azure vm create** 也可以创建 NIC，但你需要为虚拟网络和子网传递其他参数。）

然后运行一个可将 URI 传递给新 OS VHD 文件和现有映像的命令。本示例在中国东部区域创建 Standard\_A1 大小的 VM。

    azure vm create -g myResourceGroup1 -n myNewVM -l chinaeast -y Linux \
    -z Standard_A1 -u myAdminname -p myPassword -f myNIC \
    -d "https://xxxxxxxxxxxxxx.blob.core.chinacloudapi.cn/vhds/MyNewVHDNamePrefix.vhd" \
    -Q "https://xxxxxxxxxxxxxx.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/vhds/MyVHDNamePrefix-osDisk.vhd"

对于其他命令选项，运行 `azure help vm create`。

## 后续步骤
要使用 CLI 管理 VM，请参阅[使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)中的任务。

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: add information about CLI 2.0-->