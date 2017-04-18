<properties
    pageTitle="创建具有静态公共 IP 地址的 VM — Azure CLI 2.0 | Azure"
    description="了解如何使用 Azure 命令行接口 (CLI) 2.0 创建具有静态公共 IP 地址的 VM。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="55bc21b0-2a45-4943-a5e7-8d785d0d015c"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/15/2016"
    wacn.date="03/31/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017" />  


# 使用 Azure CLI 2.0 创建具有静态公共 IP 地址的 VM
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/virtual-network-deploy-static-pip-arm-portal/)
- [PowerShell](/documentation/articles/virtual-network-deploy-static-pip-arm-ps/)
- [Azure CLI 2.0](/documentation/articles/virtual-network-deploy-static-pip-arm-cli/)
- [Azure CLI 1.0](/documentation/articles/virtual-network-deploy-static-pip-cli-nodejs/)
- [模板](/documentation/articles/virtual-network-deploy-static-pip-arm-template/)
- [PowerShell（经典）](/documentation/articles/virtual-networks-reserved-public-ip/)

[AZURE.INCLUDE [virtual-network-deploy-static-pip-intro-include.md](../../includes/virtual-network-deploy-static-pip-intro-include.md)]

> [AZURE.NOTE]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。

[AZURE.INCLUDE [virtual-network-deploy-static-pip-scenario-include.md](../../includes/virtual-network-deploy-static-pip-scenario-include.md)]

## <a name = "create"></a>创建 VM

可使用 Azure CLI 2.0（详见本文）或 [Azure CLI 1.0](/documentation/articles/virtual-network-deploy-static-pip-cli-nodejs/) 完成此任务。在以下步骤中，"" 中的变量值使用本方案的设置创建资源。根据需要更改环境值。

1. 安装 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)（若尚未安装）。
2. 完成[为 Linux VM 创建 SSH 公钥和私钥对](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)中的步骤，为 Linux VM 创建 SSH 公钥/私钥对。
3. 从命令行界面，使用 `az login` 命令登录。
4. 执行 Linux 或 Mac 计算机上的脚本进行 VM 创建。Azure 公共 IP 地址、虚拟网络、网络接口和 VM 资源必须存在于同一位置。虽然资源并非都必须位于同一资源组，但在以下脚本中如此。

        #!/bin/sh

        RgName="IaaSStory"
        Location="chinanorth"
        az group create --name $RgName --location $Location

        # Create a public IP address resource with a static IP address
        PipName="PIPWEB1"
        # Note: The value below must be unique within the azure location it's created in.
        DnsName="iaasstoryws1"

        az network public-ip create \
        --name $PipName \
        --resource-group $RgName \
        --location $Location \

        # The following option allocates a static public IP address to the resource. If you do not specify it, the address is
        # allocated dynamically. The address is assigigned to the resource from a pool of IP adresses unique to each Azure regions.
        # Download and view the file from https://www.microsoft.com/download/details.aspx?id=42064 to see the ranges for each region.
        --allocation-method Static \

        --dns-name $DnsName \

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
        # https://docs.microsoft.com/azure/virtual-machines/virtual-machines-linux-sizes article.
        VmSize="Standard_DS1"

        # Replace the value for the OsImage variable value with a value for *urn* from the output returned by entering the
        # `az vm image list` command. 
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

        # If creating a Windows VM, remove the next line and you'll be prompted for the password you want to configure for the VM.
        --ssh-key-value $SshKeyValue \
        --use-unmanaged-disk

    除了创建 VM，该脚本还创建：
    - 使用非托管磁盘，并自动创建 Azure 存储账户。有关详细信息，请阅读[使用 Azure CLI 2.0 创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/) 一文。
    - 虚拟网络、子网、NIC 和公共 IP 地址资源。也可以使用*现有*虚拟网络、子网、NIC 或公共 IP 地址资源。若要了解如何使用现有网络资源，而不是创建其他资源，请输入 `az vm create -h`。

## <a name = "validate"></a>验证 VM 创建和公共 IP 地址

1. 输入命令 `az resource list --resouce-group IaaSStory --output table`，查看由脚本创建的资源的列表。返回的输出中应该有五个资源：网络接口、磁盘、公共 IP 地址、虚拟网络和虚拟机。
2. 输入命令 `az network public-ip show --name PIPWEB1 --resource-group IaaSStory --output table`。在返回的输出中，记下 **IpAddress** 的值，并且 **PublicIpAllocationMethod** 的值为 *Static*。
3. 在执行以下命令前，请删除 <>，将 *Username* 替换为用于脚本中 **Username** 变量的名称，并将 *ipAddress* 替换为上一步中的 **ipAddress**。运行以下命令以连接到 VM：`ssh -i ~/.ssh/azure_id_rsa <Username>@<ipAddress>`。

## <a name= "clean-up"></a>删除 VM 和关联的资源

如果创建资源组只是为了完成本文中的步骤，则可以通过使用 `az group delete -n IaaSStory` 命令删除资源组来删除所有资源。

>[AZURE.WARNING]
在删除资源组之前，请确认在资源组中除了本文中的脚本创建的资源外没有其他资源。运行 `az resource list --resouce-group IaaSStory` 命令以查看资源组中的资源。

如果不会在生产环境中使用 VM，建议删除资源。VM、公共 IP 地址和磁盘资源在预配后会产生费用。

## 后续步骤

任何网络流量都可流入和流出本文中创建的 VM。可以在 NSG 中定义入站和出站规则，以限制可以流入和流出网络接口和/或子网的流量。若要详细了解 NSG，请阅读 [NSG 概述](/documentation/articles/virtual-networks-nsg/)一文。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: change from CLI 1.0 to CLI 2.0-->