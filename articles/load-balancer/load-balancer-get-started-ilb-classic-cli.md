<properties
    pageTitle="创建内部负载均衡器 - Azure CLI 经典 | Azure"
    description="了解如何在经典部署模型中使用 Azure CLI 创建内部负载均衡器"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="becbbbde-a118-4269-9444-d3153f00bf34"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="04/17/2017"
    ms.author="kumud"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="6519254a2e988af5d20672df2470511a1403cb0d"
    ms.lasthandoff="04/07/2017" />

# <a name="get-started-creating-an-internal-load-balancer-classic-using-the-azure-cli"></a>开始使用 Azure CLI 创建内部负载均衡器（经典）
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/load-balancer-get-started-ilb-classic-ps/)
- [Azure CLI](/documentation/articles/load-balancer-get-started-ilb-classic-cli/)
- [云服务](/documentation/articles/load-balancer-get-started-ilb-classic-cloud/)

[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

> [AZURE.IMPORTANT]
> Azure 具有用于创建和处理资源的两个不同的部署模型：[Resource Manager 和经典](/documentation/articles/resource-manager-deployment-model/)。  本文介绍使用经典部署模型的情况。 Azure 建议大多数新部署使用 Resource Manager 模型。 了解如何[使用 Resource Manager 模型执行这些步骤](/documentation/articles/load-balancer-get-started-ilb-arm-cli/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## <a name="to-create-an-internal-load-balancer-set-for-virtual-machines"></a>为虚拟机创建内部负载均衡器集

若要创建内部负载均衡器集和要将其流量发送到的服务器，必须执行以下操作：

1. 创建内部负载均衡实例，该实例将是要在负载均衡集的服务器上进行负载均衡的传入流量的终结点。
2. 添加与虚拟机对应的终结点以接收传入流量。
3. 配置将发送要进行负载均衡的流量的服务器，以将其流量发送到内部负载均衡实例的虚拟 IP (VIP) 地址。

## <a name="step-by-step-creating-an-internal-load-balancer-using-cli"></a>使用 CLI 逐步创建内部负载均衡器

本指南演示如何基于上述方案创建内部负载均衡器。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到经典模式，如下所示。

        azure config mode asm

    预期输出：

        info:    New mode is asm

## <a name="create-endpoint-and-load-balancer-set"></a>创建终结点和负载均衡器集

此方案假定虚拟机“DB1”和“DB2”在一个名为“mytestcloud”的云服务中。 这两个虚拟机均使用一个包含子网“subnet-1”的名为“testvnet”的虚拟网络。

本指南将使用端口 1433 作为专用端口和本地端口创建内部负载均衡器集。

这是一种常见方案，其中在使用内部负载均衡器的后端有 SQL 虚拟机，以确保不会使用公共 IP 地址直接公开数据库服务器。

### <a name="step-1"></a>步骤 1

使用 `azure network service internal-load-balancer add`创建内部负载均衡器集。

    azure service internal-load-balancer add --serviceName mytestcloud --internalLBName ilbset --subnet-name subnet-1 --static-virtualnetwork-ipaddress 192.168.2.7

有关详细信息，请查看 `azure service internal-load-balancer --help` 。

你可以使用命令 `azure service internal-load-balancer list` *云服务名称*查看内部负载均衡器属性。

下面是数据的示例：

    azure service internal-load-balancer list my-testcloud
    info:    Executing command service internal-load-balancer list
    + Getting cloud service deployment
    data:    Name    Type     SubnetName  StaticVirtualNetworkIPAddress
    data:    ------  -------  ----------  -----------------------------
    data:    ilbset  Private  subnet-1    192.168.2.7
    info:    service internal-load-balancer list command OK

### <a name="step-2"></a>步骤 2

当你添加第一个终结点时，可配置内部负载均衡器集。 在此步骤中，你会将终结点、虚拟机和探测器端口关联到内部负载均衡器集。

    azure vm endpoint create db1 1433 --local-port 1433 --protocol tcp --probe-port 1433 --probe-protocol tcp --probe-interval 300 --probe-timeout 600 --internal-load-balancer-name ilbset

### <a name="step-3"></a>步骤 3

使用 `azure vm show` *虚拟机名称*

    azure vm show DB1

输出将为：

    azure vm show DB1
    info:    Executing command vm show
    + Getting virtual machines
    data:    DNSName "mytestcloud.chinacloudapp.cn"
    data:    Location "China East"
    data:    VMName "DB1"
    data:    IPAddress "192.168.2.4"
    data:    InstanceStatus "ReadyRole"
    data:    InstanceSize "Standard_D1"
    data:    Image "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-20151022-en.us-127GB.vhd"
    data:    OSDisk hostCaching "ReadWrite"
    data:    OSDisk name "db1-DB1-0-201511120457370846"
    data:    OSDisk mediaLink "https://XXXX.blob.core.chinacloudapi.cn/vhd"
    data:    OSDisk sourceImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-20151022-en.us-127GB.vhd"
    data:    OSDisk operatingSystem "Windows"
    data:    OSDisk iOType "Standard"
    data:    ReservedIPName ""
    data:    VirtualIPAddresses 0 address "137.116.64.107"
    data:    VirtualIPAddresses 0 name "db1ContractContract"
    data:    VirtualIPAddresses 0 isDnsProgrammed true
    data:    VirtualIPAddresses 1 address "192.168.2.7"
    data:    VirtualIPAddresses 1 name "ilbset"
    data:    Network Endpoints 0 localPort 5986
    data:    Network Endpoints 0 name "PowerShell"
    data:    Network Endpoints 0 port 5986
    data:    Network Endpoints 0 protocol "tcp"
    data:    Network Endpoints 0 virtualIPAddress "137.116.64.107"
    data:    Network Endpoints 0 enableDirectServerReturn false
    data:    Network Endpoints 1 localPort 3389
    data:    Network Endpoints 1 name "Remote Desktop"
    data:    Network Endpoints 1 port 60173
    data:    Network Endpoints 1 protocol "tcp"
    data:    Network Endpoints 1 virtualIPAddress "137.116.64.107"
    data:    Network Endpoints 1 enableDirectServerReturn false
    data:    Network Endpoints 2 localPort 1433
    data:    Network Endpoints 2 name "tcp-1433-1433"
    data:    Network Endpoints 2 port 1433
    data:    Network Endpoints 2 loadBalancerProbe port 1433
    data:    Network Endpoints 2 loadBalancerProbe protocol "tcp"
    data:    Network Endpoints 2 loadBalancerProbe intervalInSeconds 300
    data:    Network Endpoints 2 loadBalancerProbe timeoutInSeconds 600
    data:    Network Endpoints 2 protocol "tcp"
    data:    Network Endpoints 2 virtualIPAddress "192.168.2.7"
    data:    Network Endpoints 2 enableDirectServerReturn false
    data:    Network Endpoints 2 loadBalancerName "ilbset"
    info:    vm show command OK

## <a name="create-a-remote-desktop-endpoint-for-a-virtual-machine"></a>为虚拟机创建远程桌面终结点

你可以使用 `azure vm endpoint create`创建远程桌面终结点，将网络流量从公共端口转发到特定虚拟机的本地端口。

    azure vm endpoint create web1 54580 -k 3389

## <a name="remove-virtual-machine-from-load-balancer"></a>从负载均衡器中删除虚拟机

可以通过删除关联的终结点从内部负载均衡器集中删除虚拟机。 删除终结点后，虚拟机将不再属于该负载均衡器集。

使用上面的示例，你可以使用命令 `azure vm endpoint delete`从内部负载均衡器“ilbset”中删除为虚拟机“DB1”创建的终结点。

    azure vm endpoint delete DB1 tcp-1433-1433

有关详细信息，请查看 `azure vm endpoint --help` 。

## <a name="next-steps"></a>后续步骤

[使用源 IP 关联配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[配置负载均衡器的空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!--Update_Description: update meta properties; wording update -->