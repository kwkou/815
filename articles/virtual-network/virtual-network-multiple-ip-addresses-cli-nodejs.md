<properties
    pageTitle="使用 Azure CLI 1.0 且具有多个 IP 地址的 VM | Azure"
    description="了解如何使用 Azure CLI 1.0 将多个 IP 地址分配给虚拟机 | Resource Manager。"
    services="virtual-network"
    documentationcenter="na"
    author="anavinahar"
    manager="narayan"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="11/17/2016"
    wacn.date="05/02/2017"
    ms.author="annahar"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="cda5161993658aa537382efe4fc26c1308400931"
    ms.lasthandoff="04/22/2017" />

# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-azure-cli-10"></a>使用 Azure CLI 1.0 将多个 IP 地址分配给虚拟机

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

本文介绍如何使用 Azure CLI 1.0 通过 Azure Resource Manager 部署模型创建虚拟机 (VM)。 无法将多个 IP 地址分配到通过经典部署模型创建的资源。 若要详细了解 Azure 部署模型，请阅读[了解部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-template-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name = "create"></a>创建具有多个 IP 地址的 VM

可以使用 Azure CLI 1.0（本文）或 [Azure CLI 2.0](/documentation/articles/virtual-network-multiple-ip-addresses-cli/) 完成此任务。 下面的步骤说明如何根据方案中所述，创建具有多个 IP 地址的示例 VM。 在自己的实施项目中，需要更改变量名称和 IP 地址类型。

1. 按照[安装和配置 Azure CLI](/documentation/articles/cli-install-nodejs/) 一文中的步骤安装并配置 Azure CLI 1.0，然后使用 `azure-login` 命令登录到 Azure 帐户。

2. 创建资源组：

        RgName=myResourceGroup
        Location=chinaeast
        azure group create --name $RgName --location $Location

3. 创建虚拟网络：

        azure network vnet create --resource-group $RgName --location $Location --name myVNet \
        --address-prefixes 10.0.0.0/16

4. 在虚拟网络中创建子网：

        azure network vnet subnet create --name mySubnet --resource-group $RgName --vnet-name myVNet \
        --address-prefix 10.0.0.0/24

5. 为 VM 创建存储帐户。 运行以下命令之前，请使用唯一名称替换 mystorageaccount。 该名称在 Azure 中必须唯一。

        az storage account create --resource-group $RgName --location $Location --name mystorageaccount \
        --kind Storage --sku Standard_LRS

6. 创建 IP 配置和 NIC，并将 IP 配置分配给 NIC。 可以根据需要添加、删除或更改配置。 方案所述的配置如下所示：

    **IPConfig-1**

    输入以下命令，创建：

    - 具有静态公共 IP 地址的公共 IP 地址资源
    - 一个 NIC，将公共 IP 地址和一个静态专用 IP 地址分配给它。

    使用 Azure 位置中唯一的名称替换 mypublicdns。

        azure network public-ip create --resource-group $RgName --location $Location --name myPublicIP1 \
        --domain-name-label mypublicdns --allocation-method Static

        azure network nic create --resource-group $RgName --location $Location --name myNic1 \
        --private-ip-address 10.0.0.4 --subnet-name mySubnet --subnet-vnet-name myVNet \
        --subnet-name mySubnet --public-ip-name myPublicIP1

    > [AZURE.NOTE]
    > 公共 IP 地址会产生少许费用。 有关 IP 地址定价的详细信息，请阅读 [IP 地址定价](/pricing/details/reserved-ip-addresses/)页。 可在一个订阅中使用的公共 IP 地址数有限制。 有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。

    **IPConfig-2**

    输入以下命令，创建具有一个静态公共 IP 地址和一个静态专用 IP 地址的新公共 IP 地址资源和新 IP 配置：

        azure network public-ip create --resource-group $RgName --location $Location --name myPublicIP2
        --domain-name-label mypublicdns2 --allocation-method Static

        azure network nic ip-config create --resource-group $RgName --nic-name myNic1 --name IPConfig-2
        --private-ip-address 10.0.0.5 --public-ip-name myPublicIP2

    **IPConfig-3**

    输入以下命令，创建具有静态专用 IP 地址且没有公共 IP 地址的 IP 配置：

        azure network nic ip-config create --resource-group $RgName --nic-name myNic1 --private-ip-address 10.0.0.6 \
        --name IPConfig-3

    >[AZURE.NOTE] 
    >尽管本文是将所有 IP 配置分配到单个 NIC，但你也可以将多个 IP 配置分配到 VM 中的任意 NIC。 若要了解如何创建具有多个 NIC 的 VM，请阅读“创建具有多个 NIC 的 VM”一文。

7. 创建 Linux VM 

        az vm create --resource-group $RgName --name myVM1 --location $Location --nics myNic1 \
        --image UbuntuLTS --ssh-key-value ~/.ssh/id_rsa.pub --admin-username azureuser

    例如，若要将 VM 大小更改为标准 DS2 v2，只需将以下属性 `--vm-size Standard_DS3_v2` 添加到步骤 6 中的 `azure vm create` 命令。

8. 输入以下命令查看 NIC 和关联的 IP 配置：

        azure network nic show --resource-group $RgName    --name myNic1

9. 将专用 IP 地址添加到 VM 操作系统，只需完成本文 [将 IP 地址添加到 VM 操作系统](#os-config) 部分针对操作系统的步骤即可。

## <a name="add"></a>将 IP 地址添加到 VM

完成以下步骤，可将其他专用和公共 IP 地址添加到现有 NIC。 示例根据本文所述的 [方案](#Scenario) 生成。

1. 打开 Azure CLI，在单个 CLI 会话中完成本部分的剩余步骤。 如果尚未安装并配置 Azure CLI，请完成[安装和配置 Azure CLI](/documentation/articles/cli-install-nodejs/) 一文中的步骤，然后登录到你的 Azure 帐户。

2. 根据要求完成以下部分之一中的步骤：

    - **添加专用 IP 地址**

        若要将专用 IP 地址添加到 NIC，必须使用以下命令创建 IP 配置。 静态地址必须是未使用的子网地址。

            azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic1 \
            --private-ip-address 10.0.0.7 --name IPConfig-4

        使用唯一的配置名称和专用 IP 地址创建任意数目的配置（适用于具有静态 IP 地址的配置）。

    - **添加公共 IP 地址**

        将公共 IP 地址关联到新 IP 配置或现有 IP 配置即可添加它。 根据需要，完成以下任一部分中的步骤。

        > [AZURE.NOTE]
        > 公共 IP 地址会产生少许费用。 有关 IP 地址定价的详细信息，请阅读 [IP 地址定价](/pricing/details/reserved-ip-addresses/)页。 可在一个订阅中使用的公共 IP 地址数有限制。 有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
        >

        **将资源关联到新 IP 配置**

        每次在新 IP 配置中添加公共 IP 地址时，还必须添加专用 IP 地址，因为所有 IP 配置都必须具有专用 IP 地址。 可添加现有公共 IP 地址资源，也可创建新的公共 IP 地址资源。 若要新建此类资源，请输入以下命令：

            azure network public-ip create --resource-group myResourceGroup --location chinaeast --name myPublicIP3 \
            --domain-name-label mypublicdns3

         若要新建具有静态专用 IP 地址和关联的 myPublicIP3 公共 IP 地址资源的 IP 配置，请输入下面的命令：

            azure network nic ip-config create --resource-group myResourceGroup --nic-name myNic --name IPConfig-4 \
            --private-ip-address 10.0.0.8 --public-ip-name myPublicIP3

        **将资源关联到现有 IP 配置**

        公共 IP 地址资源只能关联到尚未与任何公共 IP 地址资源关联的 IP 配置。 输入以下命令即可确定某个 IP 配置是否具有关联的公共 IP 地址：

            azure network nic ip-config list --resource-group myResourceGroup --nic-name myNic1

        在返回的输出中查找类似 IPConfig-3 下面的行：

            Name               Provisioning state  Primary  Private IP allocation Private IP version  Private IP address  Subnet    Public IP
            default-ip-config  Succeeded           true     Static                IPv4                10.0.0.4            mySubnet  myPublicIP
            IPConfig-2         Succeeded           false    Static                IPv4                10.0.0.5            mySubnet  myPublicIP2
            IPConfig-3         Succeeded           false    Static                IPv4                10.0.0.6            mySubnet

        *IpConfig-3* 的 **Public IP** 列为空，这表示该 IP 配置当前没有任何关联的公共 IP 地址资源。 可将现有公共 IP 地址资源添加到 IpConfig-3，或输入以下命令进行创建：

            azure network public-ip create --resource-group  myResourceGroup --location chinaeast \
            --name myPublicIP3 --domain-name-label mypublicdns3 --allocation-method Static

        输入以下命令，将公共 IP 地址资源关联到名为 *IPConfig-3*的现有 IP 配置：

            azure network nic ip-config set --resource-group myResourceGroup --nic-name myNic1 --name IPConfig-3 \
            --public-ip-name myPublicIP3

3. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源：

        azure network nic ip-config list --resource-group myResourceGroup --nic-name myNic1

    返回的输出与以下内容类似：

        Name               Provisioning state  Primary  Private IP allocation Private IP version  Private IP address  Subnet    Public IP

        default-ip-config  Succeeded           true     Static                IPv4                10.0.0.4            mySubnet  myPublicIP
        IPConfig-2         Succeeded           false    Static                IPv4                10.0.0.5            mySubnet  myPublicIP2
        IPConfig-3         Succeeded           false    Static                IPv4                10.0.0.6            mySubnet  myPublicIP3

4. 根据本文[将 IP 地址添加到 VM 操作系统](#os-config)部分中的说明，将添加到 NIC 的专用 IP 地址添加到 VM 操作系统。 请勿向操作系统添加公共 IP 地址。

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]

<!--Update_Description: wording update-->