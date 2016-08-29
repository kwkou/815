<properties 
   pageTitle="使用 Azure CLI 在 Resource Manager 中创建面向 Internet 的负载平衡器 | Azure"
   description="了解如何使用 Azure CLI 在 Resource Manager 中创建面向 Internet 的负载平衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.date="02/24/2016"
   wacn.date="08/29/2016" />

# 开始使用 Azure CLI 创建面向 Internet 的负载平衡器

[AZURE.INCLUDE [load-balancer-get-started-internet-arm-selectors-include.md](../../includes/load-balancer-get-started-internet-arm-selectors-include.md)]

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include.md](../../includes/azure-arm-classic-important-include.md)] 本文介绍资源管理器部署模型。你还可以[了解如何使用经典部署创建面向 Internet 的负载平衡器](/documentation/articles/load-balancer-get-started-internet-classic-portal/)


[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

这将涵盖要创建负载平衡器所必须完成的单个任务的序列，并详细说明要怎样做才能实现目标。


## 创建面向 Internet 的负载平衡器需要什么？

需要创建和配置以下对象以部署负载平衡器。

- 前端 IP 配置 - 包含传入网络流量的公共 IP 地址。

- 后端地址池 - 包含要从负载平衡器接收网络流量的虚拟机的网络接口 (NIC)。

- 负载平衡规则 - 包含将负载平衡器上的公共端口映射到后端地址池中的端口的规则。

- 入站 NAT 规则 - 包含将负载平衡器上的公共端口映射到后端地址池中特定虚拟机的端口的规则。

- 探测器 - 包含用于检查后端地址池中虚拟机实例的可用性的运行状况探测器。

你可以在以下网页中获取有关 Azure Resource Manager 的负载平衡器组件的详细信息：[Azure Resource Manager 对负载平衡器的支持](/documentation/articles/load-balancer-arm/)。

## 将 CLI 设置为使用 Resource Manager

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。

2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	预期输出：

		info:    New mode is arm

## 为前端 IP 池创建虚拟网络和公共 IP 地址

### 步骤 1

使用名为 *NRPRG* 的资源组在中国东部位置创建名为 *NRPVnet* 的虚拟网络 (VNet)。

	azure network vnet create NRPRG NRPVnet chinaeast -a 10.0.0.0/16

使用 *NRPVnet* 中 10.0.0.0/24 的 CIDR 块创建名为 *NRPVnetSubnet* 的子网。

	azure network vnet subnet create NRPRG NRPVnet NRPVnetSubnet -a 10.0.0.0/24

### 步骤 2

使用 DNS 名称 *loadbalancernrp.chinaeast.cloudapp.azure.com* 创建要由前端 IP 池使用的名为 *NRPPublicIP* 的公共 IP 地址。下面的命令使用静态分配类型和 4 分钟的空闲超时。

	azure network public-ip create -g NRPRG -n NRPPublicIP -l chinaeast -d loadbalancernrp -a static -i 4


>[AZURE.IMPORTANT] 负载平衡器将使用公共 IP 的域标签作为其 FQDN。这与使用云服务作为负载平衡器 FQDN 的经典部署不同。
在此示例中，FQDN 将为 *loadbalancernrp.chinaeast.cloudapp.azure.com*。

## 创建负载平衡器

在以下示例中，下面的命令将在*美国东部* Azure 位置的 *NRPRG* 资源组中创建名为 *NRPlb* 的负载平衡器。

	azure network lb create NRPRG NRPlb chinaeast

## 创建前端 IP 池和后端地址池

下面的示例创建将接收负载平衡器的传入网络流量的前端 IP 池和将在其中发送经过负载平衡的网络流量的后端 IP 池。

### 步骤 1 

创建将关联在上一步中创建的公共 IP 和负载平衡器的前端 IP 池。

	azure network lb frontend-ip create nrpRG NRPlb NRPfrontendpool -i nrppublicip

### 步骤 2 

设置用于从前端 IP 池接收传入流量的后端地址池。

	azure network lb address-pool create NRPRG NRPlb NRPbackendpool

## 创建 LB 规则、NAT 规则和探测器

下面的示例将创建以下项。

- 用于将端口 21 上的所有传入流量转换到端口 22 的 NAT 规则<sup>1</sup>
- 用于将端口 23 上的所有传入流量转换到端口 22 的 NAT 规则
- 用于将端口 80 上的所有传入流量平衡到后端池中的地址端口 80 的负载平衡器规则。
- 探测规则，它将在名为 *HealthProbe.aspx* 的页上检查运行状况状态。

<sup>1</sup> NAT 规则将关联到负载平衡器后面的特定虚拟机实例。在下面的示例中，端口 21 的传入网络流量将发送到与 NAT 规则关联的端口 22 上的特定虚拟机。你必须为 NAT 规则选择协议（UDP 或 TCP）。这两种协议不能分配给同一个端口。

### 步骤 1

创建 NAT 规则。

	azure network lb inbound-nat-rule create -g nrprg -l nrplb -n ssh1 -p tcp -f 21 -b 22
	azure network lb inbound-nat-rule create -g nrprg -l nrplb -n ssh2 -p tcp -f 23 -b 22

参数：

- **-g** - 资源组名称
- **-l** - 负载平衡器名称
- **-n** - 资源的名称（无论是 nat 规则、探测器还是 lb 规则）。
- **-p** - 协议（可以是 TCP 或 UDP）
- **-f** - 要使用的前端端口（探测命令使用 -f 定义探测路径）
- **-b** - 要使用的后端端口

### 步骤 2

创建负载平衡器规则。

	azure network lb rule create nrprg nrplb lbrule -p tcp -f 80 -b 80 -t NRPfrontendpool -o NRPbackendpool
### 步骤 3

创建运行状况探测器。

	azure network lb probe create -g nrprg -l nrplb -n healthprobe -p "http" -o 80 -f healthprobe.aspx -i 15 -c 4

	
	

**-g** - 资源组 
**-l** - 负载平衡器集的名称 
**-n** - 运行状况探测器的名称 
**-p** - 运行状况探测器所用的协议 
**-i** - 探测时间间隔（以秒为单位）
**-c** - 检查次数

### 步骤 4

检查你的设置。

	azure network lb show nrprg nrplb

预期输出：

	info:    Executing command network lb show
	+ Looking up the load balancer "nrplb"
	+ Looking up the public ip "NRPPublicIP"	
	data:    Id                              : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb
	data:    Name                            : nrplb
	data:    Type                            : Microsoft.Network/loadBalancers
	data:    Location                        : chinaeast
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

你需要创建 NIC（或修改现有 NIC），并将其关联到 NAT 规则、负载平衡器规则和探测器。

### 步骤 1 

创建名为 *lb-nic1-be* 的 NIC，并将其与 *rdp1* NAT 规则和 *NRPbackendpool* 后端地址池相关联。
	
	azure network nic create -g nrprg -n lb-nic1-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1" chinaeast

参数：

- **-g** - 资源组名称
- **-n** - NIC 资源的名称
- **--subnet-name** - 子网的名称
- **--subnet-vnet-name** - 虚拟网络的名称
- **-d** - 后端池资源的 ID（以 /subscription/{subscriptionID/resourcegroups/<resourcegroup-name>/providers/Microsoft.Network/loadbalancers/<load-balancer-name>/backendaddresspools/<name-of-the-backend-pool> 开头）
- **-e** - 将关联到 NIC 资源的 NAT 规则的 ID（以 /subscriptions/####################################/resourceGroups/<resourcegroup-name>/providers/Microsoft.Network/loadBalancers/<load-balancer-name>/inboundNatRules/<nat-rule-name> 开头）


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

### 步骤 2

创建名为 *lb-nic2-be* 的 NIC，并将其与 *rdp2* NAT 规则和 *NRPbackendpool* 后端地址池相关联。

 	azure network nic create -g nrprg -n lb-nic2-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp2" chinaeast

### 步骤 3 

创建名为 *web1* 的虚拟机 (VM)，并将其与名为 *lb-nic1-be* 的 NIC 相关联。名为 *web1nrp* 的存储帐户在运行以下命令之前已创建。

	azure vm create --resource-group nrprg --name web1 --location chinaeast --vnet-name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic1-be --availset-name nrp-avset --storage-account-name web1nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

>[AZURE.IMPORTANT] 负载平衡器中的 VM 需要在同一可用性集中。使用 `azure availset create` 创建可用性集。

输出将如下所示：

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

>[AZURE.NOTE] 应显示信息性消息**这是未配置公共 IP 的 NIC**，因为为连接到 Internet 的负载平衡器创建的 NIC 使用的是负载平衡器公共 IP 地址。

由于 *lb-nic1-be* NIC 与 *rdp1* NAT 规则相关联，因此你可以使用 RDP 通过负载平衡器上的端口 3441 连接到 *web1*。

### 步骤 4

创建名为 *web2* 的虚拟机 (VM)，并将其与名为 *lb-nic2-be* 的 NIC 相关联。名为 *web1nrp* 的存储帐户在运行以下命令之前已创建。

	azure vm create --resource-group nrprg --name web2 --location chinaeast --vnet-	name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic2-be --availset-name nrp-avset --storage-account-name web2nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

## 更新现有的负载平衡器

你可以添加引用现有负载平衡器的规则。在下面的示例中，新的负载平衡器规则将添加到现有的负载平衡器 **NRPlb**

	azure network lb rule create -g nrprg -l nrplb -n lbrule2 -p tcp -f 8080 -b 8051 -t frontendnrppool -o NRPbackendpool

参数：

**-g** - 资源组名称<br> 
**-l** - 负载平衡器名称<BR> 
**-n** - 负载平衡器规则名称<BR> 
**-p** - 协议<BR> 
**-f** - 前端端口<BR> 
**-b** - 后端端口<BR> 
**-t** - 前端池名称<BR> 
**-b** - 后端池名称<BR>

## 删除负载平衡器 


若要删除负载平衡器，请使用以下命令

	azure network lb delete -g nrprg -n nrplb 

其中 **nrprg** 是资源组，**nrplb** 是负载平衡器名称。

## 后续步骤

[开始配置内部负载平衡器](/documentation/articles/load-balancer-get-started-ilb-arm-cli/)

[配置负载平衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载平衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0822_2016-->
