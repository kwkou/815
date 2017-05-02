<properties
    pageTitle="创建具有静态公共 IP 地址的 VM — Azure CLI 2.0 | Azure"
    description="了解如何使用 Azure 命令行接口 (CLI) 2.0 创建具有静态公共 IP 地址的 VM。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="55bc21b0-2a45-4943-a5e7-8d785d0d015c"
    ms.service="virtual-network"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/15/2016"
    wacn.date="05/02/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="e01b221f15a937d2e96bdb7ebe001d5e90bdfebb"
    ms.lasthandoff="04/22/2017" />

# <a name="create-a-vm-with-a-static-public-ip-address-using-the-azure-cli-20"></a>使用 Azure CLI 2.0 创建具有静态公共 IP 地址的 VM
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/virtual-network-deploy-static-pip-arm-portal/)
- [PowerShell](/documentation/articles/virtual-network-deploy-static-pip-arm-ps/)
- [Azure CLI 2.0](/documentation/articles/virtual-network-deploy-static-pip-arm-cli/)
- [Azure CLI 1.0](/documentation/articles/virtual-network-deploy-static-pip-cli-nodejs/)
- [模板](/documentation/articles/virtual-network-deploy-static-pip-arm-template/)
- [PowerShell（经典）](/documentation/articles/virtual-networks-reserved-public-ip/)

[AZURE.INCLUDE [virtual-network-deploy-static-pip-intro-include.md](../../includes/virtual-network-deploy-static-pip-intro-include.md)]

Azure 具有用于创建和处理资源的两个不同的部署模型： [资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。 本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。

[AZURE.INCLUDE [virtual-network-deploy-static-pip-scenario-include.md](../../includes/virtual-network-deploy-static-pip-scenario-include.md)]

## <a name = "create"></a>创建 VM

可以使用 Azure CLI 2.0（本文）或 [Azure CLI 1.0](/documentation/articles/virtual-network-deploy-static-pip-cli-nodejs/) 完成此任务。 在以下步骤中，"" 中的变量值使用本方案的设置创建资源。 根据需要更改环境值。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

1. 安装 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2) （若尚未安装）。
2. 通过完成[为 Linux VM 创建 SSH 公钥和私钥对](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)中的步骤创建适用于 Linux VM 的 SSH 公钥和私钥对。
3. 从命令行界面，使用 `az login`命令登录。
4. 执行 Linux 或 Mac 计算机上的脚本进行 VM 创建。 Azure 公共 IP 地址、虚拟网络、网络接口和 VM 资源必须存在于同一位置。 虽然资源并非都必须位于同一资源组，但在以下脚本中如此。

        RgName="IaaSStory"
        Location="chinanorth"

        # Create a resource group.

        az group create \
        --name $RgName \
        --location $Location

        # Create a public IP address resource with a static IP address using the --allocation-method Static option.
        # If you do not specify this option, the address is allocated dynamically. The address is assigned to the
        # resource from a pool of IP adresses unique to each Azure region. The DnsName must be unique within the
        # Azure location it's created in. Download and view the file from https://www.microsoft.com/download/details.aspx?id=41653#
        # that lists the ranges for each region.

        PipName="PIPWEB1"
        DnsName="iaasstoryws1"
        az network public-ip create \
        --name $PipName \
        --resource-group $RgName \
        --location $Location \
        --allocation-method Static \
        --dns-name $DnsName

        # Create a virtual network with one subnet

        VnetName="TestVNet"
        VnetPrefix="192.168.0.0/16"
        SubnetName="FrontEnd"
        SubnetPrefix="192.168.1.0/24"
        az network vnet create \
        --name $VnetName \
        --resource-group $RgName \
        --location $Location \
        --address-prefix $VnetPrefix \
        --subnet-name $SubnetName \
        --subnet-prefix $SubnetPrefix

        # Create a network interface connected to the VNet with a static private IP address and associate the public IP address
        # resource to the NIC.

        NicName="NICWEB1"
        PrivateIpAddress="192.168.1.101"
        az network nic create \
        --name $NicName \
        --resource-group $RgName \
        --location $Location \
        --subnet $SubnetName \
        --vnet-name $VnetName \
        --private-ip-address $PrivateIpAddress \
        --public-ip-address $PipName

        # Create a new VM with the NIC

        VmName="WEB1"

        # Replace the value for the VmSize variable with a value from the
        # https://www.azure.cn/documentation/articles/virtual-machines-linux-sizes/ article.
        VmSize="Standard_DS1"

        # Replace the value for the OsImage variable with a value for *urn* from the output returned by entering
        # the `az vm image list` command. 

        OsImage="credativ:Debian:8:latest"
        Username='adminuser'

        # Replace the following value with the path to your public key file.
        SshKeyValue="~/.ssh/id_rsa.pub"

        az vm create \
        --name $VmName \
        --resource-group $RgName \
        --image $OsImage \
        --location $Location \
        --size $VmSize \
        --nics $NicName \
        --admin-username $Username \
        --ssh-key-value $SshKeyValue \
        # If creating a Windows VM, remove the previous line and you'll be prompted for the password you want to configure for the VM.
        --use-unmanaged-disk

除了创建 VM，该脚本还创建：
- 使用非托管磁盘，并自动创建 Azure 存储账户。 有关详细信息，请阅读[使用 Azure CLI 2.0 创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/) 一文。
- 虚拟网络、子网、NIC 和公共 IP 地址资源。 也可以使用 *现有* 虚拟网络、子网、NIC 或公共 IP 地址资源。 若要了解如何使用现有网络资源，而不是创建其他资源，请输入 `az vm create -h`。

## <a name = "validate"></a>验证 VM 创建和公共 IP 地址

1. 输入命令 `az resource list --resouce-group IaaSStory --output table` ，查看由脚本创建的资源的列表。 返回的输出中应该有五个资源：网络接口、磁盘、公共 IP 地址、虚拟网络和虚拟机。
2. 输入命令 `az network public-ip show --name PIPWEB1 --resource-group IaaSStory --output table`。 在返回的输出中，记下 **IpAddress** 的值，并且 **PublicIpAllocationMethod** 的值是 *Static*。
3. 在执行以下命令前，请删除 <>，将 *Username* 替换为用于脚本中 **Username** 变量的名称，并将 *ipAddress* 替换为上一步中的 **ipAddress**。 输入以下命令以连接到 VM：`ssh -i ~/.ssh/azure_id_rsa <Username>@<ipAddress>`。 

## <a name= "clean-up"></a>删除 VM 和关联的资源

如果生产环境中不会使用此练习中创建的资源，建议将其删除。 VM、公共 IP 地址和磁盘资源在预配后将产生费用。 若要删除此练习中创建的资源，请完成以下步骤：

1. 若要查看资源组中的资源，请运行 `az resource list --resource-group IaaSStory` 命令。
2. 请确认在资源组中除了本文中脚本创建的资源外没有其他资源。 
3. 若要删除本练习中创建的所有资源，请运行 `az group delete -n IaaSStory` 命令。 该命令将删除资源组及其包含的所有资源。

## <a name="next-steps"></a>后续步骤

任何网络流量都可流入和流出本文中创建的 VM。 可以在 NSG 中定义入站和出站规则，以限制可以流入和流出网络接口和/或子网的流量。 若要了解有关 NSG 的详细信息，请阅读 [NSG 概述](/documentation/articles/virtual-networks-nsg/)一文。

<!--Update_Description: wording update-->