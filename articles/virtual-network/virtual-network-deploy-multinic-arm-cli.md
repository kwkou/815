<properties
    pageTitle="创建具有多个 NIC 的 VM — Azure CLI 2.0 | Azure"
    description="了解如何使用 Azure CLI 2.0 创建具有多个 NIC 的 VM。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="8e906a4b-8583-4a97-9416-ee34cfa09a98"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/02/2016"
    wacn.date="03/31/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017" />  


# 使用 Azure CLI 2.0 创建具有多个 NIC 的 VM

[AZURE.INCLUDE [virtual-network-deploy-multinic-arm-selectors-include.md](../../includes/virtual-network-deploy-multinic-arm-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-deploy-multinic-intro-include.md](../../includes/virtual-network-deploy-multinic-intro-include.md)]

> [AZURE.NOTE]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是[经典部署模型](/documentation/articles/virtual-network-deploy-multinic-classic-cli/)。
>

## <a name="create"></a>创建 VM

可使用 Azure CLI 2.0（详见本文）或 [Azure CLI 1.0](/documentation/articles/virtual-network-deploy-multinic-cli-nodejs/) 完成此任务。在以下步骤中，"" 中的变量值使用本方案的设置创建资源。根据需要更改环境值。

1. 安装 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)（若尚未安装）。
2. 完成[为 Linux VM 创建 SSH 公钥和私钥对](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)中的步骤，为 Linux VM 创建 SSH 公钥/私钥对。
3. 从命令行界面，使用 `az login` 命令登录。
4. 执行 Linux 或 Mac 计算机上的脚本进行 VM 创建。该脚本创建一个资源组、一个包含两个子网的虚拟网络 (VNet)、两个 NIC 和一个附加有两个 NIC 的 VM。其中一个 NIC 连接到一个子网，并分配有一个静态公共 IP 地址和一个静态专用 IP 地址。另一个 NIC 连接到其他子网，并分配有一个静态专用 IP 地址，未分配公共 IP 地址。NIC、公共 IP 地址、虚拟网络和 VM 资源均必须位于同一位置和订阅。虽然资源并非都必须位于同一资源组，但在以下脚本中如此。

        #!/bin/sh

        RgName="Multi-NIC-VM"
        Location="chinanorth"
        az group create --name $RgName --location $Location

        # Create a public IP address resource with a static IP address using the `--allocation-method Static` option.
        # If you do not specify this option, the address is allocated dynamically. The address is assigned to the
        # resource from a pool of IP adresses unique to each Azure region. 
        # Download and view the file from https://www.microsoft.com/download/details.aspx?id=42064 that lists
        # the ranges for each region.

        PipName="PIP-WEB"

        az network public-ip create \
        --name $PipName \
        --resource-group $RgName \
        --location $Location \
        --allocation-method Static

        # Create a virtual network with one subnet

        VnetName="VNet1"
        VnetPrefix="10.0.0.0/16"
        VnetSubnet1Name="Front-End"
        VnetSubnet1Prefix="10.0.0.0/24"

        az network vnet create \
        --name $VnetName \
        --resource-group $RgName \
        --location $Location \
        --address-prefix $VnetPrefix \
        --subnet-name $VnetSubnet1Name \
        --subnet-prefix $VnetSubnet1Prefix

        # Create a second subnet within the VNet

        VnetSubnet2Name="Back-end"
        VnetSubnet2Prefix="10.0.1.0/24"

        az network vnet subnet create \
        --vnet-name $VnetName \
        --resource-group $RgName \
        --name $VnetSubnet2Name \
        --address-prefix $VnetSubnet2Prefix

        # Create a network interface connected to one of the subnets. The NIC is assigned a single dynamic private and
        # public IP address by default, but you can instead, assign static addresses, or no public IP address at all.
        # You can also assign multiple private or public IP addresses to each NIC. To learn more about IP addressing
        # options for NICs, enter the `az network nic create -h` command.

        Nic1Name="NIC-FE"
        PrivateIpAddress1="10.0.0.5"

        az network nic create \
        --name $Nic1Name \
        --resource-group $RgName \
        --location $Location \
        --subnet $VnetSubnet1Name \
        --vnet-name $VnetName \
        --private-ip-address $PrivateIpAddress1 \
        --public-ip-address $PipName

        # Create a second network interface and connect it to the other subnet. Though multiple NICs attached to the same
        # VM can be connected to different subnets, the subnets must all be within the same VNet. Add additional NICs as necessary.

        Nic2Name="NIC-BE"
        PrivateIpAddress2="10.0.1.5"

        az network nic create \
        --name $Nic2Name \
        --resource-group $RgName \
        --location $Location \
        --subnet $VnetSubnet2Name \
        --vnet-name $VnetName \
        --private-ip-address $PrivateIpAddress2 \

        # Create a VM and attach the two NICs.

        VmName="WEB"

        # Replace the value for the following **VmSize** variable with a value from the
        # https://docs.microsoft.com/azure/virtual-machines/virtual-machines-linux-sizes article. Not all VM sizes support
        # more than one NIC, so be sure to select a VM size that supports the number of NICs you want to attach to the VM.
        # You must create the VM with at least two NICs if you want to add more after VM creation. If you create a VM with
        # only one NIC, you can't add additional NICs to the VM after VM creation, regardless of how many NICs the VM supports.
        # The VM size specified in the following variable supports two NICs.

        VmSize="Standard_DS2"

        # Replace the value for the OsImage variable value with a value for *urn* from the output returned by entering the
        # `az vm image list` command.

        OsImage="credativ:Debian:8:latest"

        Username="adminuser"

        # Replace the following value with the path to your public key file.

        SshKeyValue="~/.ssh/id_rsa.pub"

        # Before executing the following command, add variable names of additional NICs you may have added to the script that
        # you want to attach to the VM. If creating a Windows VM, remove the **ssh-key-value** line and you'll be prompted for
        # the password you want to configure for the VM.

        az vm create \
        --name $VmName \
        --resource-group $RgName \
        --image $OsImage \
        --location $Location \
        --size $VmSize \
        --nics $Nic1Name $Nic2Name \
        --admin-username $Username \
        --ssh-key-value $SshKeyValue

    除了创建具有两个 NIC 的 VM，该脚本还创建：
    - 单个高级托管磁盘（默认情况下），但对于可以创建的磁盘类型，可以有其他选择。有关详细信息，请阅读[使用 Azure CLI 2.0 创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/) 一文。
    - 一个包含两个子网和单个公共 IP 地址的虚拟网络。也可以使用*现有*虚拟网络、子网、NIC 或公共 IP 地址资源。若要了解如何使用现有网络资源，而不是创建其他资源，请输入 `az vm create -h`。

## <a name = "validate"></a>验证 VM 创建和 NIC

1. 输入命令 `az resource list --resouce-group Multi-NIC-VM --output table`，查看由脚本创建的资源的列表。返回的输出中应该有六个资源：两个 NIC、一个磁盘、一个公共 IP 地址、一个虚拟网络和一个虚拟机。
2. 输入命令 `az network public-ip show --name PIP-WEB --resource-group Multi-NIC-VM --output table`。在返回的输出中，记下 **IpAddress** 的值，并且 **PublicIpAllocationMethod** 的值为 *Static*。
3. 在运行以下命令前，请删除 <>，将 *Username* 替换为用于脚本中 **Username** 变量的名称，并将 *ipAddress* 替换为上一步中的 **ipAddress**。运行以下命令以连接到 VM：`ssh -i ~/.ssh/azure_id_rsa <Username>@<ipAddress>`。
4. 连接到 VM 后，运行 `sudo ifconfig` 命令以查看 *eth0* 和 *eth1* 接口。Azure DHCP 服务器已为每个 NIC 分配了脚本中指定的静态专用 IP 地址。删除 VM 之前，分配给 NIC 的 IP 和 MAC 地址不会更改。建议不要更改操作系统内的 IP 寻址，因为它可能会禁用与计算机的连接。公共 IP 地址不会显示在操作系统中，因为它们是 Azure 基础结构转换为专用 IP 地址或从专用 IP 地址转换的网络地址。

## <a name= "clean-up"></a>删除 VM 和关联的资源

如果创建资源组只是为了完成本文中的步骤，则可以通过使用 `az group delete --name Multi-NIC-VM` 命令删除资源组来删除所有资源。

>[AZURE.WARNING]
在删除资源组之前，请确认在资源组中除了本文中的脚本创建的资源外没有其他资源。运行 `az resource list --resource-group Multi-NIC-VM` 命令以查看资源组中的资源。

如果不会在生产环境中使用 VM，建议删除资源。VM、公共 IP 地址和磁盘资源在预配后会产生费用。

## 后续步骤

任何网络流量都可流入和流出本文中创建的 VM。可以在 NSG 中定义入站和出站规则，以限制可以流入和流出每个网络接口和/或每个子网的流量。若要详细了解 NSG，请阅读 [NSG 概述](/documentation/articles/virtual-networks-nsg/)一文。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: change from CLI 1.0 to CLI 2.0-->