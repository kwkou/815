<properties
    pageTitle="创建内部负载均衡器 - Azure CLI | Azure"
    description="了解如何在 Resource Manager 中使用 Azure CLI 创建内部负载均衡器"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    tags="azure-resource-manager" />
<tags
    ms.assetid="c7a24e92-b4da-43c0-90f2-841c1b7ce489"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="03/03/2017"
    ms.author="kumud" />  

# 使用 Azure CLI 创建内部负载均衡器

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/load-balancer-get-started-ilb-arm-portal/)
- [PowerShell](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)
- [Azure CLI](/documentation/articles/load-balancer-get-started-ilb-arm-cli/)
- [模板](/documentation/articles/load-balancer-get-started-ilb-arm-template/)

[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

> [AZURE.NOTE]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是[经典部署模型](/documentation/articles/load-balancer-get-started-ilb-classic-cli/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## 使用 Azure CLI 部署解决方案

以下步骤说明如何使用 Azure Resource Manager 和 CLI 创建面向 Internet 的负载均衡器。借助 Azure Resource Manager，可单独创建和配置每个资源，再将其合成一个新资源。

需要创建和配置以下对象以部署负载均衡器：

* **前端 IP 配置**：包含传入网络流量的公共 IP 地址
* **后端地址池**：包含使虚拟机可以从负载均衡器接收网络流量的网络接口 (NIC)
* **负载均衡规则**：所含规则可将负载均衡器上的公共端口映射到后端地址池的端口上
* **入站 NAT 规则**：所含规则可将负载均衡器上的公共端口映射到后端地址池中特定虚拟机的端口
* **探测器**：包含用于检查后端地址池中虚拟机实例的可用性的运行状况探测器

有关详细信息，请参阅 [Azure Resource Manager 对负载均衡器的支持](/documentation/articles/load-balancer-arm/)。

## 将 CLI 设置为使用 Resource Manager

1. 如果从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。按照说明执行，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到 Resource Manager 模式，如下所示：

        azure config mode arm

    预期输出：

        info:    New mode is arm

## 逐步创建内部负载均衡器集

1. 登录 Azure。

        azure login -e AzureChinaCloud

    在系统提示时输入 Azure 凭据。

2. 将命令工具更改为 Azure Resource Manager 模式。

        azure config mode arm

## 创建资源组

Azure Resource Manager 中的所有资源将与资源组关联。创建资源组（如果你尚未这样做）。

    azure group create <resource group name> <location>

## 创建内部负载均衡器集

1. 创建内部负载均衡器

    在以下方案中，将在中国东部区域中创建一个名为 nrprg 的资源组。

        azure network lb create --name nrprg --location chinaeast

    > [AZURE.NOTE]
    内部负载均衡器的所有资源（如虚拟网络和虚拟网络子网）必须都在同一资源组中并在同一区域中。
    >
    

2. 为内部负载均衡器创建前端 IP 地址。

    所用的 IP 地址必须在虚拟网络的子网范围内。

        azure network lb frontend-ip create --resource-group nrprg --lb-name ilbset --name feilb --private-ip-address 10.0.0.7 --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet


3. 创建后端地址池。

        azure network lb address-pool create --resource-group nrprg --lb-name ilbset --name beilb


    定义前端 IP 地址和后端地址池后，可以创建负载均衡器规则、入站 NAT 规则和自定义运行状况探测器。

4. 为内部负载均衡器创建负载均衡器规则。

    按照前面的步骤执行时，该命令将创建负载均衡器规则，以侦听前端池中的端口 1433，还使用端口 1433 将经过负载均衡的网络流量发送到后端地址池。

        azure network lb rule create --resource-group nrprg --lb-name ilbset --name ilbrule --protocol tcp --frontend-port 1433 --backend-port 1433 --frontend-ip-name feilb --backend-address-pool-name beilb
5. 创建入站 NAT 规则。

    入站 NAT 规则用于在负载均衡器中创建要转到特定虚拟机实例的终结点。前面的步骤为远程桌面创建了两个 NAT 规则。

        azure network lb inbound-nat-rule create --resource-group nrprg --lb-name ilbset --name NATrule1 --protocol TCP --frontend-port 5432 --backend-port 3389

        azure network lb inbound-nat-rule create --resource-group nrprg --lb-name ilbset --name NATrule2 --protocol TCP --frontend-port 5433 --backend-port 3389


6. 为负载均衡器创建运行状况探测器。

    运行状况探测器将检查所有虚拟机实例，以确保它们可以发送网络流量。探测器检查失败的虚拟机实例将从负载均衡器中删除，直到它恢复联机状态并且探测器检查确定它运行正常。

        azure network lb probe create --resource-group nrprg --lb-name ilbset --name ilbprobe --protocol tcp --interval 300 --count 4

    > [AZURE.NOTE]
    Azure Platform 对各种管理方案使用一个公开可路由的静态 IPv4 地址。该 IP 地址为 168.63.129.16。此 IP 地址不应被任何防火墙阻止，因为这可能会导致意外行为。对于 Azure 内部负载均衡，此 IP 地址用于监视负载均衡器中的探测器，以确定负载均衡集中虚拟机的运行状况状态。如果网络安全组用于将流量限制到内部负载均衡集中的 Azure 虚拟机或应用于虚拟网络子网，请确保添加网络安全规则以允许来自 168.63.129.16 的流量。
    >
    
## 创建 NIC

你需要创建 NIC（或修改现有 NIC），并将其关联到 NAT 规则、负载均衡器规则和探测器。

1. 创建名为 *lb-nic1-be* 的 NIC，然后将其与 *rdp1* NAT 规则和 *beilb* 后端地址池相关联。

        azure network nic create --resource-group nrprg --name lb-nic1-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet --lb-address-pool-ids "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/beilb" --lb-inbound-nat-rule-ids "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1" --location chinaeast


    预期输出：

        info:    Executing command network nic create
        + Looking up the network interface "lb-nic1-be"
        + Looking up the subnet "nrpvnetsubnet"
        + Creating network interface "lb-nic1-be"
        + Looking up the network interface "lb-nic1-be"
        data:    Id                              : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/networkInterfaces/lb-nic1-be
        data:    Name                            : lb-nic1-be
        data:    Type                            : Microsoft.Network/networkInterfaces
        data:    Location                        : chinaeast
        data:    Provisioning state              : Succeeded
        data:    Enable IP forwarding            : false
        data:    IP configurations:
        data:      Name                          : NIC-config
        data:      Provisioning state            : Succeeded
        data:      Private IP address            : 10.0.0.4
        data:      Private IP Allocation Method  : Dynamic
        data:      Subnet                        : /subscriptions/####################################/resourceGroups/NRPRG/providers/Microsoft.Network/virtualNetworks/NRPVnet/subnets/NRPVnetSubnet
        data:      Load balancer backend address pools
        data:        Id                          : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool
        data:      Load balancer inbound NAT rules:
        data:        Id                          : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1
        data:
        info:    network nic create command OK

2. 创建名为 *lb-nic2-be* 的 NIC，然后将其与 *rdp2* NAT 规则和 *beilb* 后端地址池相关联。

        azure network nic create --resource-group nrprg --name lb-nic2-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet --lb-address-pool-ids "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/beilb" --lb-inbound-nat-rule-ids "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp2" --location chinaeast

3. 创建名为 *DB1* 的虚拟机，然后将其与名为 *lb-nic1-be* 的 NIC 相关联。名为 *web1nrp* 的存储帐户在以下命令运行之前已创建：

        azure vm create --resource--resource-grouproup nrprg --name DB1 --location chinaeast --vnet-name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic1-be --availset-name nrp-avset --storage-account-name web1nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

    > [AZURE.IMPORTANT]
    负载均衡器中的 VM 需要在同一可用性集中。使用 `azure availset create` 创建可用性集。
    >
    

4. 创建名为 *DB2* 的虚拟机 (VM)，然后将其与名为 *lb-nic2-be* 的 NIC 相关联。名为 *web1nrp* 的存储帐户在运行以下命令之前已创建。

        azure vm create --resource--resource-grouproup nrprg --name DB2 --location chinaeast --vnet-name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic2-be --availset-name nrp-avset --storage-account-name web2nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

## 删除负载均衡器

若要删除负载均衡器，请使用以下命令：

        azure network lb delete --resource-group nrprg --name ilbset


## 后续步骤

[使用源 IP 关联配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description:update meta properties; wording update; update code with full-parameter format -->