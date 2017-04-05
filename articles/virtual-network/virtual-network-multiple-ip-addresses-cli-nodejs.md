<properties
    pageTitle="使用 Azure CLI 1.0 且具有多个 IP 地址的 VM | Azure"
    description="了解如何使用 Azure CLI 1.0 将多个 IP 地址分配给虚拟机 | Resource Manager。"
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


# 使用 Azure CLI 1.0 将多个 IP 地址分配给虚拟机

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

本文介绍如何使用 Azure CLI 1.0 通过 Azure Resource Manager 部署模型创建虚拟机 (VM)。无法将多个 IP 地址分配给通过经典部署模型创建的资源。若要了解有关 Azure 部署模型的详细信息，请阅读[了解部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

[AZURE.INCLUDE [virtual-network-preview](../../includes/virtual-network-preview.md)]

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-template-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name = "create"></a>创建具有多个 IP 地址的 VM

可使用 Azure CLI 1.0（详见本文）或 [Azure CLI 2.0](/documentation/articles/virtual-network-multiple-ip-addresses-cli/) 完成此任务。以下步骤说明如何根据本方案所述，创建具有多个 IP 地址的示例 VM。根据实现的需要，更改变量名称和 IP 地址类型。

1. 按照[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/) 文章中的步骤安装和配置 Azure CLI 1.0，然后通过 `azure-login` 命令登录到 Azure 帐户。

2. 登录后，在 PowerShell 中运行以下命令进行预览版注册（不可通过 CLI 注册），然后选择相应的订阅：

        Register-AzureRmProviderFeature  -FeatureName AllowMultipleIpConfigurationsPerNic -ProviderNamespace Microsoft.Network
        Register-AzureRmProviderFeature  -FeatureName AllowLoadBalancingonSecondaryIpconfigs -ProviderNamespace Microsoft.Network
        Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network

    请不要尝试完成剩余步骤，直至运行 ```Get-AzureRmProviderFeature``` 命令时看到以下输出：

        FeatureName                            ProviderName      RegistrationState
        -----------                            ------------      -----------------
        AllowLoadBalancingOnSecondaryIpConfigs Microsoft.Network Registered       
        AllowMultipleIpConfigurationsPerNic    Microsoft.Network Registered       

    >[AZURE.NOTE] 
    这可能需要几分钟的时间。

3. [创建资源组](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-resource-groups-and-choose-deployment-locations)，然后创建[虚拟网络和子网](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-virtual-network-and-subnet)。按如下所示，更改 ``` --address-prefixes ``` 和 ```--address-prefix``` 字段，满足本文列出的具体方案要求：

        --address-prefixes 10.0.0.0/16
        --address-prefix 10.0.0.0/24

    >[AZURE.NOTE] 
    上面引用的文章使用“中国北部”作为创建资源的位置，但本文使用“中国西北部”。相应更改位置。

4. 为 VM [创建存储帐户](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-storage-account)。

5. 创建要分配给 NIC 的 NIC 和 IP 配置。可根据需要添加、删除或更改配置。方案所述的配置如下所示：

    **IPConfig-1**

    输入以下命令，创建：

    - 一个具有静态公共 IP 地址的公共 IP 地址资源
    - 一个具有公共 IP 地址资源和动态专用 IP 地址的 IP 配置

        azure network public-ip create --resource-group myResourceGroup --location chinaeast --name myPublicIP --domain-name-label mypublicdns --allocation-method Static

    > [AZURE.NOTE]
    公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。

        azure network nic create --resource-group myResourceGroup --location chinaeast --subnet-vnet-name myVnet --subnet-name mySubnet --name myNic1 --public-ip-name myPublicIP

    **IPConfig-2**

     输入以下命令，创建具有一个静态公共 IP 地址和一个静态专用 IP 地址的新公共 IP 地址资源和新 IP 配置：

        azure network public-ip create --resource-group myResourceGroup --location chinaeast --name myPublicIP2 --domain-name-label mypublicdns2 --allocation-method Static

        azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-2 --private-ip-address 10.0.0.5 --public-ip-name myPublicIP2

    **IPConfig-3**

    输入以下命令，创建具有一个动态专用 IP 地址且没有公共 IP 地址的 IP 配置：

        azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-3

    >[AZURE.NOTE] 
    尽管本文将所有 IP 配置都分配给一个 NIC，但还可将多个 IP 配置分配给 VM 中的任何 NIC。若要了解如何创建具有多个 NIC 的 VM，请阅读“创建具有多个 NIC 的 VM”一文。

6. [创建 Linux VM](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-the-linux-vms)。请务必删除 ```  --availset-name myAvailabilitySet \ ``` 属性，此方案不需要它。根据方案使用相应的位置。

    >[AZURE.WARNING] 
    如果选择的位置不支持 VM 大小，“创建 VM”一文中的步骤 6 会失败。运行以下命令，以便获取中国东部 VM 的完整列表，例如：`azure vm sizes --location chinaeast` 此位置名称可根据方案更改。

    例如，若要将 VM 大小更改为标准 DS2 v2，只需在步骤 6 中将以下属性 ```  --vm-size Standard_DS3_v2``` 到添加 ``` azure vm create ``` 命令。

7. 输入以下命令，查看 NIC 和关联的 IP 配置：

        azure network nic show --resource-group myResourceGroup --name myNic1

8. 将专用 IP 地址添加到 VM 操作系统，只需完成本文[将 IP 地址添加到 VM 操作系统](#os-config)部分针对操作系统的步骤即可。

## <a name="add"></a>将 IP 地址添加到 VM

完成以下步骤，可将其他专用和公共 IP 地址添加到现有 NIC。示例根据本文所述的[方案](#Scenario)生成。

1. 打开 Azure CLI，在单个 CLI 会话中完成本部分的剩余步骤。如果还没有安装和配置 Azure CLI，请完成[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/) 一文中的步骤，并登录 Azure 帐户。

2. 按照**创建具有多个 IP 地址的 VM** 部分中的步骤 2，注册公共预览版。

3. 根据要求，完成以下任一部分中的步骤：

    **添加专用 IP 地址**
	
    若要将专用 IP 地址添加到 NIC，必须使用以下命令创建 IP 配置。如果想要添加动态专用 IP 地址，请在输入命令前删除 ```-PrivateIpAddress 10.0.0.7```。指定的静态 IP 地址必须是子网的未使用地址。

        azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 --private-ip-address 10.0.0.7 --name IPConfig-4

    使用唯一配置名称和专用 IP 地址（用于具有静态 IP 地址的配置），根据需要创建多个配置。

    **添加公共 IP 地址**
	
    将公共 IP 地址关联到新 IP 配置或现有 IP 配置即可添加它。根据需要，完成以下任一部分中的步骤。

    > [AZURE.NOTE]
    公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
    >

    **将资源关联到新 IP 配置**
	
    每次在新 IP 配置中添加公共 IP 地址时，还必须添加专用 IP 地址，因为所有 IP 配置都必须具有专用 IP 地址。可添加现有公共 IP 地址资源，也可创建新的公共 IP 地址资源。若要新建，请输入以下命令：

          azure network public-ip create --resource-group myResourceGroup --location chinaeast --name myPublicIP3 --domain-name-label mypublicdns3

    若要创建具有动态专用 IP 地址和关联的 *myPublicIP3* 公共 IP 地址资源的新 IP 配置，请输入以下命令：

        azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic --name IPConfig-4 --public-ip-name myPublicIP3

    **将资源关联到现有 IP 配置**
    公共 IP 地址资源仅可关联到尚未关联公共 IP 地址资源的 IP 配置。可输入以下命令，确定某个 IP 配置是否具有关联的公共 IP 地址：

        azure network nic ip-config list --resource-group myResourceGroup --nic-name myNic1

    在返回的输出中查找类似如下所示的行：
	
        Name               Provisioning state  Primary  Private IP allocation  Private IP version  Private IP address  Subnet    Public IP
        -----------------  ------------------  -------  ---------------------  ------------------  ------------------  --------  -----------
        default-ip-config  Succeeded           true     Dynamic                IPv4                10.0.0.4            mySubnet  myPublicIP
        IPConfig-2         Succeeded           false    Static                 IPv4                10.0.0.5            mySubnet  myPublicIP2
        IPConfig-3         Succeeded           false    Dynamic                IPv4                10.0.0.6            mySubnet
     
    *IpConfig-3* 的**公共 IP** 列为空白，因此，当前没有公共 IP 地址资源与其关联。可将现有公共 IP 地址资源添加到 IpConfig-3，或输入以下命令进行创建：

        azure network public-ip create --resource-group  myResourceGroup --location chinaeast --name myPublicIP3 --domain-name-label mypublicdns3 --allocation-method Static

    输入以下命令，将公共 IP 地址资源关联到名为 *IPConfig-3* 的现有 IP 配置：

        azure network nic ip-config set --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-3 --public-ip-name myPublicIP3

7. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源：

        azure network nic ip-config list --resource-group myResourceGroup --nic-name myNic1

    您应该会看到与下面类似的输出：
	
        Name               Provisioning state  Primary  Private IP allocation  Private IP version  Private IP address  Subnet    Public IP
        -----------------  ------------------  -------  ---------------------  ------------------  ------------------  --------  -----------
        default-ip-config  Succeeded           true     Dynamic                IPv4                10.0.0.4            mySubnet  myPublicIP
        IPConfig-2         Succeeded           false    Static                 IPv4                10.0.0.5            mySubnet  myPublicIP2
        IPConfig-3         Succeeded           false    Dynamic                IPv4                10.0.0.6            mySubnet  myPublicIP3
     
9. 根据本文[将 IP 地址添加到 VM 操作系统](#os-config)部分中的说明，将添加到 NIC 的专用 IP 地址添加到 VM 操作系统。请勿向操作系统添加公共 IP 地址。

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]

<!---HONumber=Mooncake_0327_2017-->