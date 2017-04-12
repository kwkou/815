<properties
    pageTitle="使用 Azure CLI 2.0 且具有多个 IP 地址的 VM | Azure"
    description="了解如何使用 Azure CLI 2.0 将多个 IP 地址分配给虚拟机 | Resource Manager。"
    services="virtual-network"
    documentationcenter="na"
    author="anavinahar"
    manager="narayan"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="11/17/2016"
    wacn.date="03/31/2017"
    ms.author="annahar" />  


# 使用 Azure CLI 2.0 将多个 IP 地址分配给虚拟机

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

本文介绍如何使用 Azure CLI 2.0 通过 Azure Resource Manager 部署模型创建虚拟机 (VM)。无法将多个 IP 地址分配给通过经典部署模型创建的资源。若要了解有关 Azure 部署模型的详细信息，请阅读[了解部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

[AZURE.INCLUDE [virtual-network-preview](../../includes/virtual-network-preview.md)]

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-template-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name = "create"></a>创建具有多个 IP 地址的 VM

可使用 Azure CLI 2.0（详见本文）或 [Azure CLI 1.0](/documentation/articles/virtual-network-multiple-ip-addresses-cli-nodejs/) 完成此任务。根据需要更改环境值。以下步骤说明如何根据本方案所述，创建具有多个 IP 地址的示例 VM。根据实现的需要，更改 "" 中的变量值和 IP 地址类型。

1. 安装 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)（若尚未安装）。
2. 完成[为 Linux VM 创建 SSH 公钥和私钥对](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)中的步骤，为 Linux VM 创建 SSH 公钥/私钥对。
3. 在命令行界面中，通过命令 `az login` 登录，然后选择当前使用的订阅。
4. <a name="register"></a>运行以下 PowerShell 命令进行预览版注册（无法通过 CLI 注册）：

        Register-AzureRmProviderFeature  -FeatureName AllowMultipleIpConfigurationsPerNic    -ProviderNamespace Microsoft.Network
        Register-AzureRmProviderFeature  -FeatureName AllowLoadBalancingonSecondaryIpconfigs -ProviderNamespace Microsoft.Network
        Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network

    请不要尝试完成剩余步骤，直至运行 ```Get-AzureRmProviderFeature``` 命令时看到以下输出：

        FeatureName                            ProviderName      RegistrationState
        -----------                            ------------      -----------------
        AllowLoadBalancingOnSecondaryIpConfigs Microsoft.Network Registered       
        AllowMultipleIpConfigurationsPerNic    Microsoft.Network Registered       

    >[AZURE.NOTE] 
    注册可能会耗时几分钟。
5. 执行 Linux 或 Mac 计算机上的脚本进行 VM 创建。该脚本将创建 1 个资源组、1 个虚拟网络 (VNet)、1 个具有 3 个 IP 配置的 NIC，以及附加 2 个 NIC 的 VM。NIC、公共 IP 地址、虚拟网络和 VM 资源均必须位于同一位置和订阅。虽然资源并非都必须位于同一资源组，但在以下脚本中如此。

        #!/bin/sh

        RgName="myResourceGroup"
        Location="chinaeast"
        az group create --name $RgName --location $Location

        # Create a public IP address resource with a static IP address using the `--allocation-method Static` option. If you
        # do not specify this option, the address is allocated dynamically. The address is assigned to the resource from a pool
        # of IP adresses unique to each Azure region. Download and view the file from
        # https://www.microsoft.com/download/details.aspx?id=42064 that lists the ranges for each region.

        PipName="myPublicIP"

        # This name must be unique within an Azure location.
        DnsName="myDNSName"

        az network public-ip create \
        --name $PipName \
        --resource-group $RgName \
        --location $Location \
        --dns-name $DnsName\
        --allocation-method Static

        # Create a virtual network with one subnet

        VnetName="myVnet"
        VnetPrefix="10.0.0.0/16"
        VnetSubnetName="mySubnet"
        VnetSubnetPrefix="10.0.0.0/24"

        az network vnet create \
        --name $VnetName \
        --resource-group $RgName \
        --location $Location \
        --address-prefix $VnetPrefix \
        --subnet-name $VnetSubnetName \
        --subnet-prefix $VnetSubnetPrefix

        # Create a network interface connected to the subnet and associate the public IP address to it. Azure will create the
        # first IP configuration with a dynamic private IP address and will associate the public IP address resource to it.

        NicName="MyNic1"

        az network nic create \
        --name $NicName \
        --resource-group $RgName \
        --location $Location \
        --subnet $VnetSubnet1Name \
        --vnet-name $VnetName \
        --public-ip-address $PipName

        # Create a second public IP address, a second IP configuration, and associate it to the NIC. This configuration has a
        # static public IP address and a static private IP address.

        az network public-ip create --resource-group $RgName --location $Location --name myPublicIP2 --dns-name mypublicdns2 --allocation-method Static
        az network nic ip-config create --resource-group $RgName --nic-name $NicName --name IPConfig-2 --private-ip-address 10.0.0.5 --public-ip-name myPublicIP2

        # Create a third IP configuration, and associate it to the NIC. This configuration has a dynamic private IP address and
        # no public IP address.

        azure network nic ip-config create --resource-group $RgName --nic-name $NicName --name IPConfig-3

        # Note: Though this article assigns all IP configurations to a single NIC, you can also assign multiple IP configurations
        # to any NIC in a VM. To learn how to create a VM with multiple NICs, read the Create a VM with multiple NICs 
        # article: https://docs.microsoft.com/azure/networking/virtual-network-deploy-multinic-arm-cli.

        # Create a VM and attach the NIC.

        VmName="myVm"

        # Replace the value for the following **VmSize** variable with a value from the
        # https://docs.microsoft.com/azure/virtual-machines/virtual-machines-linux-sizes article. The script fails if the VM size
        # is not supported in the location you select. Run the `azure vm sizes --location chinaeast` command to get a full list
        # of VMs in US West Central, for example.

        VmSize="Standard_DS1"

        # Replace the value for the OsImage variable value with a value for *urn* from the output returned by entering the
        # `az vm image list` command.

        OsImage="credativ:Debian:8:latest"

        Username="adminuser"

        # Replace the following value with the path to your public key file. If you're creating a Windows VM, remove the following
        # line and you'll be prompted for the password you want to configure for the VM.

        SshKeyValue="~/.ssh/id_rsa.pub"

        az vm create \
        --name $VmName \
        --resource-group $RgName \
        --image $OsImage \
        --location $Location \
        --size $VmSize \
        --nics $NicName \
        --admin-username $Username \
        --ssh-key-value $SshKeyValue

    除了使用带 3 个 IP 配置的 NIC 创建 VM，脚本还会创建：- 单个高级托管磁盘（默认）；但也可选择创建其他磁盘类型。有关详细信息，请阅读[使用 Azure CLI 2.0 创建Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)一文。
    - 具有 1 个子网和 2 个公共 IP 地址的虚拟网络。或者，可使用*现有的*虚拟网络、子网、NIC 或公共 IP 地址资源。若要了解如何使用现有网络资源，而不是创建其他资源，请输入 `az vm create -h`。

    > [AZURE.NOTE]
    公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。

7. 创建 VM 后，输入 `az network nic show --name MyNic1 --resource-group myResourceGroup` 命令查看 NIC 配置。输入 `az network nic ip-config list --nic-name MyNic1 --resource-group myResourceGroup --output table`，查看与 NIC 关联的 IP 配置的列表。

8. 将专用 IP 地址添加到 VM 操作系统，只需完成本文[将 IP 地址添加到 VM 操作系统](#os-config)部分针对操作系统的步骤即可。

## <a name="add"></a>将 IP 地址添加到 VM

完成以下步骤，可将其他专用和公共 IP 地址添加到现有 NIC。示例根据本文所述的[方案](#Scenario)生成。

1. 打开命令行界面，在单个会话中完成本部分的剩余步骤。若尚未安装和配置 Azure CLI，请完成 [Azure CLI 2.0 安装](https://docs.microsoft.com/cli/azure/install-az-cli2)文章中的步骤，然后通过 `az-login` 命令登录 Azure 帐户。

2. 按照本文中**创建具有多个 IP 地址的 VM** 部分的[步骤 4](#register)，注册公共预览版。

3. 根据要求，完成以下任一部分中的步骤：

    **添加专用 IP 地址**
	
    若要向 NIC 添加专用 IP 地址，必须使用以下命令创建 IP 配置。如果想要添加动态专用 IP 地址，请在输入命令前删除 `-PrivateIpAddress 10.0.0.7`。指定的静态 IP 地址必须是子网的未使用地址。

        az network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 --private-ip-address 10.0.0.7 --name IPConfig-4

    使用唯一配置名称和专用 IP 地址（用于具有静态 IP 地址的配置），根据需要创建多个配置。

    **添加公共 IP 地址**
	
    将公共 IP 地址关联到新 IP 配置或现有 IP 配置即可添加它。根据需要，完成以下任一部分中的步骤。

    > [AZURE.NOTE]
    公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
    >

    **将资源关联到新 IP 配置**
	
    每次在新 IP 配置中添加公共 IP 地址时，还必须添加专用 IP 地址，因为所有 IP 配置都必须具有专用 IP 地址。可添加现有公共 IP 地址资源，也可创建新的公共 IP 地址资源。若要新建，请输入以下命令：

          az network public-ip create --resource-group myResourceGroup --location chinaeast --name myPublicIP3 --dns-name mypublicdns3

    若要创建具有动态专用 IP 地址和关联的 *myPublicIP3* 公共 IP 地址资源的新 IP 配置，请输入以下命令：

        az network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-5 --public-ip-address myPublicIP3

    **将资源关联到现有 IP 配置**
    公共 IP 地址资源仅可关联到尚未关联公共 IP 地址资源的 IP 配置。可输入以下命令，确定某个 IP 配置是否具有关联的公共 IP 地址：

        az network nic ip-config list --resource-group myResourceGroup --nic-name myNic1 --query "[?provisioningState=='Succeeded'].{ Name: name, PublicIpAddressId: publicIpAddress.id }" --output table

    返回的输出：

        Name        PublicIpAddressId
        --------    ------------------------------------------------------------------------------------------------------------
        ipconfig1   /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP1
        IPConfig-2  /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP2
        IPConfig-3  

    由于输出中 *IpConfig-3* 的 **PublicIpAddressId** 列空白，因此其当前未关联公共 IP 地址资源。可将现有公共 IP 地址资源添加到 IpConfig-3，或输入以下命令进行创建：

        az network public-ip create --resource-group  myResourceGroup --location chinaeast --name myPublicIP3 --dns-name mypublicdns3 --allocation-method Static

    输入以下命令，将公共 IP 地址资源关联到名为 *IPConfig-3* 的现有 IP 配置：

        az network nic ip-config update --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-3 --public-ip myPublicIP3

4. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源 ID：

        az network nic ip-config list --resource-group myResourceGroup --nic-name myNic1 --query "[?provisioningState=='Succeeded'].{ Name: name, PrivateIpAddress: privateIpAddress, PrivateIpAllocationMethod: privateIpAllocationMethod, PublicIpAddressId: publicIpAddress.id }" --output table

    返回的输出：

        Name        PrivateIpAddress    PrivateIpAllocationMethod    PublicIpAddressId
        --------    ------------------  ---------------------------  ------------------------------------------------------------------------------------------------------------
        ipconfig1   10.0.0.4            Dynamic                      /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP1
        IPConfig-2  10.0.0.5            Static                       /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP2
        IPConfig-3  10.0.0.6            Dynamic                      /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP3

5. 根据本文[将 IP 地址添加到 VM 操作系统](#os-config)部分中的说明，将添加到 NIC 的专用 IP 地址添加到 VM 操作系统。请勿向操作系统添加公共 IP 地址。

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: change from CLI 1.0 to CLI 2.0-->