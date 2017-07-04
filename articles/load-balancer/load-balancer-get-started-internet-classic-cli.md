<properties
    pageTitle="创建面向 Internet 的负载均衡器 - Azure CLI 经典 | Azure"
    description="了解如何使用 Azure CLI 在经典部署模型中创建面向 Internet 的负载均衡器"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="e433a824-4a8a-44d2-8765-a74f52d4e584"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="04/17/2017"
    ms.author="kumud"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="9ed15784fbc04f9ddd57cebef6def4443a253aae"
    ms.lasthandoff="04/07/2017" />

# <a name="get-started-creating-an-internet-facing-load-balancer-classic-in-the-azure-cli"></a>开始在 Azure CLI 中创建面向 Internet 的负载均衡器（经典）
> [AZURE.SELECTOR]
- [Azure 经典管理门户](/documentation/articles/load-balancer-get-started-internet-classic-portal/)
- [PowerShell](/documentation/articles/load-balancer-get-started-internet-classic-ps/)
- [Azure CLI](/documentation/articles/load-balancer-get-started-internet-classic-cli/)
- [Azure 云服务](/documentation/articles/load-balancer-get-started-internet-classic-cloud/)

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

> [AZURE.IMPORTANT]
> 在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：Azure Resource Manager 部署模型和经典部署模型。 在使用任何 Azure 资源之前，请确保了解 [部署模型和工具](/documentation/articles/azure-classic-rm/) 。 可以通过单击本文顶部的选项卡来查看不同工具的文档。 本文介绍经典部署模型。 还可以[了解如何使用 Azure Resource Manager 创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)。

[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## <a name="step-by-step-creating-an-internet-facing-load-balancer-using-cli"></a>使用 CLI 逐步创建面向 Internet 的负载均衡器

本指南演示如何基于上述方案创建 Internet 负载均衡器。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到经典模式，如下所示。

        azure config mode asm

    预期输出：

        info:    New mode is asm

## <a name="create-endpoint-and-load-balancer-set"></a>创建终结点和负载均衡器集

此方案假定已创建虚拟机“web1”和“web2”。
本指南将使用端口 80 作为公用端口和本地端口创建负载均衡器集。 还将在端口 80 上配置探测端口，并将负载均衡器集命名为“lbset”。

### <a name="step-1"></a>步骤 1

使用 `azure network vm endpoint create` 为虚拟机“web1”创建第一个终结点和负载均衡器集。

    azure vm endpoint create web1 80 --local-port 80 --protocol tcp --probe-port 80 --load-balanced-set-name lbset

### <a name="step-2"></a>步骤 2

将第二个虚拟机“web2”添加到负载均衡器集。

    azure vm endpoint create web2 80 --local-port 80 --protocol tcp --probe-port 80 --load-balanced-set-name lbset

### <a name="step-3"></a>步骤 3

使用 `azure vm show` 验证负载均衡器配置。

    azure vm show web1

输出将为：

    data:    DNSName "contoso.chinacloudapp.cn"
    data:    Location "China East"
    data:    VMName "web1"
    data:    IPAddress "10.0.0.5"
    data:    InstanceStatus "ReadyRole"
    data:    InstanceSize "Standard_D1"
    data:    Image "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-2015
    6-en.us-127GB.vhd"
    data:    OSDisk hostCaching "ReadWrite"
    data:    OSDisk name "joaoma-1-web1-0-201509251804250879"
    data:    OSDisk mediaLink "https://XXXXXXXXXXXXXXX.blob.core.chinacloudapi.cn
    /vhds/joaomatest-web1-2015-09-25.vhd"
    data:    OSDisk sourceImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Se
    r-2012-R2-20150916-en.us-127GB.vhd"
    data:    OSDisk operatingSystem "Windows"
    data:    OSDisk iOType "Standard"
    data:    ReservedIPName ""
    data:    VirtualIPAddresses 0 address "XXXXXXXXXXXXXXXX"
    data:    VirtualIPAddresses 0 name "XXXXXXXXXXXXXXXXXXXX"
    data:    VirtualIPAddresses 0 isDnsProgrammed true
    data:    Network Endpoints 0 loadBalancedEndpointSetName "lbset"
    data:    Network Endpoints 0 localPort 80
    data:    Network Endpoints 0 name "tcp-80-80"
    data:    Network Endpoints 0 port 80
    data:    Network Endpoints 0 loadBalancerProbe port 80
    data:    Network Endpoints 0 loadBalancerProbe protocol "tcp"
    data:    Network Endpoints 0 loadBalancerProbe intervalInSeconds 15
    data:    Network Endpoints 0 loadBalancerProbe timeoutInSeconds 31
    data:    Network Endpoints 0 protocol "tcp"
    data:    Network Endpoints 0 virtualIPAddress "XXXXXXXXXXXX"
    data:    Network Endpoints 0 enableDirectServerReturn false
    data:    Network Endpoints 1 localPort 5986
    data:    Network Endpoints 1 name "PowerShell"
    data:    Network Endpoints 1 port 5986
    data:    Network Endpoints 1 protocol "tcp"
    data:    Network Endpoints 1 virtualIPAddress "XXXXXXXXXXXX"
    data:    Network Endpoints 1 enableDirectServerReturn false
    data:    Network Endpoints 2 localPort 3389
    data:    Network Endpoints 2 name "Remote Desktop"
    data:    Network Endpoints 2 port 58081
    info:    vm show command OK

## <a name="create-a-remote-desktop-endpoint-for-a-virtual-machine"></a>为虚拟机创建远程桌面终结点

你可以使用 `azure vm endpoint create`创建远程桌面终结点，将网络流量从公共端口转发到特定虚拟机的本地端口。

    azure vm endpoint create web1 54580 -k 3389

## <a name="remove-virtual-machine-from-load-balancer"></a>从负载均衡器中删除虚拟机

你必须从虚拟机中删除关联到负载均衡器集的终结点。 删除终结点后，虚拟机将不再属于该负载均衡器集。

使用上面的示例，你可以使用命令 `azure vm endpoint delete` 从负载均衡器“lbset”中删除为虚拟机“web1”创建的终结点。

    azure vm endpoint delete web1 tcp-80-80

> [AZURE.NOTE]
> 你可以使用命令 `azure vm endpoint --help` 浏览管理终结点的更多选项

## <a name="next-steps"></a>后续步骤

[开始配置内部负载均衡器](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[配置负载均衡器的空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)


<!--Update_Description: update meta properties; wording update-->