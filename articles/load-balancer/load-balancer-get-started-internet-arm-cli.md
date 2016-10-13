<properties 
   pageTitle="使用 Azure CLI 在 Resource Manager 中创建面向 Internet 的负载均衡器 | Azure"
   description="了解如何使用 Azure CLI 在 Resource Manager 中创建面向 Internet 的负载均衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="sdwheeler"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="08/31/2016"
   wacn.date="10/10/2016" />  


# 开始使用 Azure CLI 创建面向 Internet 的负载均衡器

[AZURE.INCLUDE [load-balancer-get-started-internet-arm-selectors-include.md](../../includes/load-balancer-get-started-internet-arm-selectors-include.md)]

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include.md](../../includes/azure-arm-classic-important-include.md)] 本文介绍资源管理器部署模型。你还可以[了解如何使用经典部署创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-classic-portal/)


[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## 使用 Azure CLI 部署解决方案

以下步骤说明如何使用 Azure Resource Manager 和 CLI 创建面向 Internet 的负载均衡器。借助 Azure Resource Manager，可单独创建和配置每个资源，再将其合成一个新资源。

需要创建和配置以下对象才能部署负载均衡器。

- 前端 IP 配置 - 包含传入网络流量的公共 IP 地址。

- 后端地址池 - 包含要从负载均衡器接收网络流量的虚拟机的网络接口 (NIC)。

- 负载均衡规则 - 包含将负载均衡器上的公共端口映射到后端地址池中的端口的规则。

- 入站 NAT 规则 - 包含将负载均衡器上的公共端口映射到后端地址池中特定虚拟机的端口的规则。

- 探测器 - 包含用于检查后端地址池中虚拟机实例的可用性的运行状况探测器。

有关详细信息，请参阅 [Azure Resource Manager 对负载均衡器的支持](/documentation/articles/load-balancer-arm/)。

## 将 CLI 设置为使用 Resource Manager

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。

2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

        azure config mode arm

    预期输出：

        info:    New mode is arm

## 为前端 IP 池创建虚拟网络和公共 IP 地址

1. 使用名为 *NRPRG* 的资源组在中国东部位置创建名为 *NRPVnet* 的虚拟网络 (VNet)。

	azure network vnet create NRPRG NRPVnet chinaeast -a 10.0.0.0/16

    使用 *NRPVnet* 中 10.0.0.0/24 的 CIDR 块创建名为 *NRPVnetSubnet* 的子网。

        azure network vnet subnet create NRPRG NRPVnet NRPVnetSubnet -a 10.0.0.0/24

2. 使用 DNS 名称 *loadbalancernrp.eastus.cloudapp.azure.com* 创建要由前端 IP 池使用的名为 *NRPPublicIP* 的公共 IP 地址。下面的命令使用静态分配类型和 4 分钟的空闲超时。

	azure network public-ip create -g NRPRG -n NRPPublicIP -l chinaeast -d loadbalancernrp -a static -i 4

    >[AZURE.IMPORTANT] 负载均衡器将使用公共 IP 的域标签作为其 FQDN。这与经典部署不同，后者使用云服务座位负载均衡器 FQDN。
    该实例中，FQDN 是 *loadbalancernrp.eastus.cloudapp.azure.com*。

## 创建负载均衡器

以下命令会在*中国东部* Azure 位置的 *NRPRG* 资源组中创建 *NRPlb* 负载均衡器。

	azure network lb create NRPRG NRPlb chinaeast

## 创建前端 IP 池和后端地址池

下例会创建前端和后端 IP 池，前者接收负载均衡器上的传入网络流量，后者发送负载均衡的网络流量。

1. 创建前端 IP 池，它与负载均衡器和上一步中创建的公共 IP 相关联。

        azure network lb frontend-ip create nrpRG NRPlb NRPfrontendpool -i nrppublicip

2. 设置后端地址池，它用于接收前端 IP 池的传入流量。

        azure network lb address-pool create NRPRG NRPlb NRPbackendpool

## 创建 LB 规则、NAT 规则和探测器

下例会创建以下项。

- 用于将端口 21 上的所有传入流量转换到端口 22 的 NAT 规则<sup>1</sup>
- 用于将端口 23 上的所有传入流量转换到端口 22 的 NAT 规则
- 用于将端口 80 上的所有传入流量平衡到后端池中的地址端口 80 的负载均衡器规则。
- 用于检查 *HealthProbe.aspx* 页面上的运行状况状态的探测规则。

<sup>1</sup> NAT 规则将关联到负载均衡器后面的特定虚拟机实例。到达端口 21 的网络流量发送至端口 22 上与该 NAT 规则关联的特定虚拟机。必须为 NAT 规则指定协议（UDP 或 TCP）。这两种协议不能分配给同一个端口。

1. 创建 NAT 规则。

        azure network lb inbound-nat-rule create -g nrprg -l nrplb -n ssh1 -p tcp -f 21 -b 22
        azure network lb inbound-nat-rule create -g nrprg -l nrplb -n ssh2 -p tcp -f 23 -b 22

    参数：
    * **-g** - 资源组名称
    * **-l** - 负载均衡器名称
    * **-n** - 资源名称（Nat 规则、探测或 LB 规则皆可）。
    * **-p** - 协议（可以是 TCP 或 UDP）
    * **-f** - 要使用的前端端口（探测命令使用 -f 定义探测路径）
    * **-b** - 要使用的后端端口

2. 创建负载均衡器规则。

        azure network lb rule create nrprg nrplb lbrule -p tcp -f 80 -b 80 -t NRPfrontendpool -o NRPbackendpool

3. 创建运行状况探测器。

        azure network lb probe create -g nrprg -l nrplb -n healthprobe -p "http" -o 80 -f healthprobe.aspx -i 15 -c 4

    参数：
    * **-g** - 资源组
    * **-l** - 负载均衡器规则集的名称
    * **-n** - 运行状况探测器的名称
    * **-p** - 运行状况探测器所用的协议
    * **-i** - 探测间隔（以秒计）
    * **-c** - 检查数

4. 检查你的设置。

        azure network lb show nrprg nrplb

    预期输出：

        info:    Executing command network lb show
        + Looking up the load balancer "nrplb"
        + Looking up the public ip "NRPPublicIP"
        data:    Id                              : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb
        data:    Name                            : nrplb
        data:    Type                            : Microsoft.Network/loadBalancers
        data:    Location                        : eastus
        data:    Provisioning State              : Succeeded
        data:    Frontend IP configurations:
        data:      Name                          : NRPfrontendpool
        data:      Provisioning state            : Succeeded
        data:      Public IP address id          : /subscriptions/####################################/resourceGroups/NRPRG/providers/Microsoft.Network/publicIPAddresses/NRPPublicIP
        data:      Public IP allocation method   : Static
        data:      Public IP address             : 40.114.13.145
        data:
        data:    Backend address pools:
        data:      Name                          : NRPbackendpool
        data:      Provisioning state            : Succeeded
        data:
        data:    Load balancing rules:
        data:      Name                          : HTTP
        data:      Provisioning state            : Succeeded
        data:      Protocol                      : Tcp
        data:      Frontend port                 : 80
        data:      Backend port                  : 80
        data:      Enable floating IP            : false
        data:      Idle timeout in minutes       : 4
        data:      Frontend IP configuration     : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/frontendIPConfigurations/NRPfrontendpool
        data:      Backend address pool          : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool
        data:
        data:    Inbound NAT rules:
        data:      Name                          : ssh1
        data:      Provisioning state            : Succeeded
        data:      Protocol                      : Tcp
        data:      Frontend port                 : 21
        data:      Backend port                  : 22
        data:      Enable floating IP            : false
        data:      Idle timeout in minutes       : 4
        data:      Frontend IP configuration     : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/frontendIPConfigurations/NRPfrontendpool
        data:
        data:      Name                          : ssh2
        data:      Provisioning state            : Succeeded
        data:      Protocol                      : Tcp
        data:      Frontend port                 : 23
        data:      Backend port                  : 22
        data:      Enable floating IP            : false
        data:      Idle timeout in minutes       : 4
        data:      Frontend IP configuration     : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/frontendIPConfigurations/NRPfrontendpool
        data:
        data:    Probes:
        data:      Name                          : healthprobe
        data:      Provisioning state            : Succeeded
        data:      Protocol                      : Http
        data:      Port                          : 80
        data:      Interval in seconds           : 15
        data:      Number of probes              : 4
        data:
        info:    network lb show command OK

## 创建 NIC

你需要创建 NIC（或修改现有 NIC），并将其关联到 NAT 规则、负载均衡器规则和探测器。

1. 创建名为 *lb-nic1-be* 的 NIC，并将其与 *rdp1* NAT 规则和 *NRPbackendpool* 后端地址池相关联。

        azure network nic create -g nrprg -n lb-nic1-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1" eastus

    参数：

    * **-g** - 资源组名称
    * **-n** - NIC 资源的名称
    * **--subnet-name** - 子网的名称
    * **--subnet-vnet-name** - 虚拟网络的名称
    * **-d** - 后端池资源的 ID（以 /subscription/{subscriptionID/resourcegroups/<resourcegroup-name>/providers/Microsoft.Network/loadbalancers/<load-balancer-name>/backendaddresspools/<name-of-the-backend-pool> 开头）
    * **-e** - 要关联到 NIC 资源的 NAT 规则的 ID（以 /subscriptions/####################################/resourceGroups/<resourcegroup-name>/providers/Microsoft.Network/loadBalancers/<load-balancer-name>/inboundNatRules/<nat-rule-name> 开头）

    预期输出：

        info:    Executing command network nic create
        + Looking up the network interface "lb-nic1-be"
        + Looking up the subnet "nrpvnetsubnet"
        + Creating network interface "lb-nic1-be"
        + Looking up the network interface "lb-nic1-be"
        data:    Id                              : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/networkInterfaces/lb-nic1-be
        data:    Name                            : lb-nic1-be
        data:    Type                            : Microsoft.Network/networkInterfaces
        data:    Location                        : eastus
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

2. 创建名为 *lb-nic2-be* 的 NIC，并将其与 *rdp2* NAT 规则和 *NRPbackendpool* 后端地址池相关联。

 	    azure network nic create -g nrprg -n lb-nic2-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp2" chinaeast

3. 创建名为 *web1* 的虚拟机 (VM)，并将其与名为 *lb-nic1-be* 的 NIC 相关联。名为 *web1nrp* 的存储帐户在运行以下命令之前已创建。

	    azure vm create --resource-group nrprg --name web1 --location chinaeast --vnet-name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic1-be --availset-name nrp-avset --storage-account-name web1nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

    >[AZURE.IMPORTANT] 负载均衡器中的 VM 需要在同一可用性集中。使用 `azure availset create` 创建可用性集。

    输出应如下所示：

        info:    Executing command vm create
        + Looking up the VM "web1"
        Enter username: azureuser
        Enter password for azureuser: *********
        Confirm password: *********
        info:    Using the VM Size "Standard_A1"
        info:    The [OS, Data] Disk or image configuration requires storage account
        + Looking up the storage account web1nrp
        + Looking up the availability set "nrp-avset"
        info:    Found an Availability set "nrp-avset"
        + Looking up the NIC "lb-nic1-be"
        info:    Found an existing NIC "lb-nic1-be"
        info:    Found an IP configuration with virtual network subnet id "/subscriptions/####################################/resourceGroups/NRPRG/providers/Microsoft.Network/virtualNetworks/NRPVnet/subnets/NRPVnetSubnet" in the NIC "lb-nic1-be"
        info:    This is a NIC without publicIP configured
        + Creating VM "web1"
        info:    vm create command OK

    >[AZURE.NOTE] 应显示信息性消息**这是未配置公共 IP 的 NIC**，因为为连接到 Internet 的负载均衡器创建的 NIC 使用的是负载均衡器公共 IP 地址。

    由于 *lb-nic1-be* NIC 与 *rdp1* NAT 规则相关联，因此你可以使用 RDP 通过负载均衡器上的端口 3441 连接到 *web1*。

4. 创建名为 *web2* 的虚拟机 (VM)，并将其与名为 *lb-nic2-be* 的 NIC 相关联。名为 *web1nrp* 的存储帐户在运行以下命令之前已创建。

	    azure vm create --resource-group nrprg --name web2 --location chinaeast --vnet-	name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic2-be --availset-name nrp-avset --storage-account-name web2nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

## 更新现有的负载均衡器

可添加引用现有负载均衡器的规则。在下例中，向现有负载均衡器 **NRPlb** 添加了新的负载均衡器规则

    azure network lb rule create -g nrprg -l nrplb -n lbrule2 -p tcp -f 8080 -b 8051 -t frontendnrppool -o NRPbackendpool

参数：

* **-g** - 资源组名称
* **-l** - 负载均衡器名称
* **-n** - 负载均衡器规则名称
* **-p** - 协议
* **-f** - 前端端口
* **-b** - 后端端口
* **-t** - 前端池名称
* **-b** - 后端池名称

## 删除负载均衡器

以下命令可删除负载均衡器：

    azure network lb delete -g nrprg -n nrplb

其中 **nrprg** 是资源组，**nrplb** 是负载均衡器名称。

## 后续步骤

[开始配置内部负载均衡器](/documentation/articles/load-balancer-get-started-ilb-arm-cli/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0926_2016-->