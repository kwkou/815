<properties 
   pageTitle="在 Resource Manager 中使用 Azure CLI 创建内部负载平衡器 | Azure"
   description="了解如何在 Resource Manager 中使用 Azure CLI 创建内部负载平衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carolz"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.date="02/09/2016"
   wacn.date="08/29/2016" />  


# 开始使用 Azure CLI 创建内部负载平衡器

[AZURE.INCLUDE [load-balancer-get-started-ilb-arm-selectors-include/](../../includes/load-balancer-get-started-ilb-arm-selectors-include.md)]
<BR>
[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include/](../../includes/load-balancer-get-started-ilb-intro-include.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代 [classic deployment model](/documentation/articles/load-balancer-get-started-ilb-classic-cli/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include/](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## 创建内部负载平衡器需要什么？

需要创建和配置以下对象以部署负载平衡器：

- 前端 IP 配置 - 配置传入网络流量的专用 IP 地址。

- 后端地址池 - 包含从负载平衡器接收流量的网络接口 (NIC)。

- 负载平衡规则 - 包含将传入网络流量端口映射到后端池中接收网络流量的端口的规则。

- 入站 NAT 规则 - 包含将负载平衡器上的公共端口映射到后端地址池中特定虚拟机的端口的规则。

- 探测器 - 包含用于检查与后端地址池关联的虚拟机的可用性的运行状况探测器。

你可以在以下网页中获取有关 Azure Resource Manager 的负载平衡器组件的详细信息：[Azure Resource Manager 对负载平衡器的支持](/documentation/articles/load-balancer-arm/)。

## 将 CLI 设置为使用 Resource Manager


1. 如果你从未使用过 Azure CLI，请参阅[安装 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。


2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	预期输出：

		info:    New mode is arm

## 逐步创建内部负载平衡器 

以下步骤将基于上述方案创建内部负载平衡器：

### 步骤 1 

下载 [Azure 命令行界面](/downloads/)的最新版本（如果你尚未这样做）。

### 步骤 2 

在安装该软件之后，对你的帐户进行身份验证。

	azure login -e AzureChinaCloud

身份验证过程将请求 Azure 订阅的用户名和密码。

### 步骤 3

将命令工具更改为 Azure Resource Manager 模式。

	azure config mode arm

## 创建资源组

Azure Resource Manager 中的所有资源将关联到资源组。创建资源组（如果你尚未这样做）。

	azure group create <resource group name> <location>


## 创建内部负载平衡器集 


### 步骤 1 

使用 `azure network lb create` 命令创建内部负载平衡器。在以下方案中，将在中国东部区域中创建一个名为 nrprg 的资源组。
 	
	azure network lb create -n nrprg -l chinaeast

>[AZURE.NOTE] 内部负载平衡器的所有资源（如虚拟网络和虚拟网络子网）必须都在同一资源组中并在同一区域中。


### 步骤 2 

为内部负载平衡器创建前端 IP 地址。所用的 IP 地址必须在虚拟网络的子网范围内。

	
	azure network lb frontend-ip create -g nrprg -l ilbset -n feilb -a 10.0.0.7 -e nrpvnetsubnet -m nrpvnet

使用的参数：

**-g** - 资源组 
**-l** - 内部负载平衡器集的名称 
**-n** - 前端 IP 的名称 
**-a** - 子网范围内的专用 IP 地址。
**-e** - 子网名称 
**-m** - 虚拟网络名称

### 步骤 3 

创建后端地址池。

	azure network lb address-pool create -g nrprg -l ilbset -n beilb

使用的参数：

**-g** - 资源组 
**-l** - 内部负载平衡器集的名称 
**-n** - 后端地址池的名称

定义前端 IP 地址和后端地址池后，可以创建负载平衡器规则、入站 NAT 规则和自定义运行状况探测器。

### 步骤 4


为内部负载平衡器创建负载平衡器规则。按照上面的方案，该命令将创建负载平衡器规则，以侦听前端池中的端口 1433，并还使用端口 1433 将经过负载平衡的网络流量发送到后端地址池。

	azure network lb rule create -g nrprg -l ilbset -n ilbrule -p tcp -f 1433 -b 1433 -t feilb -o beilb

使用的参数：

**-g** - 资源组 
**-l** - 内部负载平衡器集的名称 
**-n** - 负载平衡器规则的名称 
**-p** - 用于规则的协议 
**-f** - 侦听负载平衡器前端的传入网络流量的端口 
**-b** - 后端地址池中接收网络流量的端口

### 步骤 5

创建入站 NAT 规则。入站 NAT 规则用于在负载平衡器中创建将转到特定虚拟机实例的终结点。
按照上面的示例，为远程桌面访问创建了 2 个 NAT 规则。

	azure network lb inbound-nat-rule create -g nrprg -l ilbset -n NATrule1 -p TCP -f 5432 -b 3389
	
	azure network lb inbound-nat-rule create -g nrprg -l ilbset -n NATrule2 -p TCP -f 5433 -b 3389

使用的参数：

**-g** - 资源组 
**-l** - 内部负载平衡器集的名称 
**-n** - 入站 NAT 规则的名称 
**-p** - 用于规则的协议 
**-f** - 侦听负载平衡器前端的传入网络流量的端口 
**-b** - 后端地址池中接收网络流量的端口

### 步骤 5 

为负载平衡器创建运行状况探测器。运行状况探测器将检查所有虚拟机实例，以确保它可以发送网络流量。探测器检查失败的虚拟机实例将从负载平衡器中删除，直到它恢复联机状态并且探测器检查为正常。

	azure network lb probe create -g nrprg -l ilbset -n ilbprobe -p tcp -i 300 -c 4

**-g** - 资源组 
**-l** - 内部负载平衡器集的名称 
**-n** - 运行状况探测器的名称 
**-p** - 运行状况探测器所用的协议 
**-i** - 探测时间间隔（以秒为单位）
**-c** - 检查次数

>[AZURE.NOTE] Azure Platform 对各种管理方案使用一个公开可路由的静态 IPv4 地址。该 IP 地址为 168.63.129.16。此 IP 地址不应被任何防火墙阻止，因为这会导致意外行为。对于 Azure 内部负载平衡，此 IP 地址用于监视负载平衡器中的探测器，以确定负载平衡集中虚拟机的运行状况状态。如果网络安全组用于将流量限制到内部负载平衡集中的 Azure 虚拟机或应用于虚拟网络子网，请确保添加网络安全规则以允许来自 168.63.129.16 的流量。

## 创建 NIC

你需要创建 NIC（或修改现有 NIC），并将其关联到 NAT 规则、负载平衡器规则和探测器。

### 步骤 1 

创建名为 *lb-nic1-be* 的 NIC，并将其与 *rdp1* NAT 规则和 *beilb* 后端地址池相关联。
	
	azure network nic create -g nrprg -n lb-nic1-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/beilb" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1" chinaeast

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

创建名为 *lb-nic2-be* 的 NIC，并将其与 *rdp2* NAT 规则和 *beilb* 后端地址池相关联。

 	azure network nic create -g nrprg -n lb-nic2-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/beilb" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp2" chinaeast

### 步骤 3 

创建名为 *DB1* 的虚拟机 (VM)，并将其与名为 *lb-nic1-be* 的 NIC 相关联。名为 *web1nrp* 的存储帐户在运行以下命令之前已创建。

	azure vm create --resource-group nrprg --name DB1 --location chinaeast --vnet-name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic1-be --availset-name nrp-avset --storage-account-name web1nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

>[AZURE.IMPORTANT] 负载平衡器中的 VM 需要在同一可用性集中。使用 `azure availset create` 创建可用性集。

### 步骤 4

创建名为 *DB2* 的虚拟机 (VM)，并将其与名为 *lb-nic2-be* 的 NIC 相关联。名为 *web1nrp* 的存储帐户在运行以下命令之前已创建。

	azure vm create --resource-group nrprg --name DB2 --location chinaeast --vnet-	name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic2-be --availset-name nrp-avset --storage-account-name web2nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

## 删除负载平衡器 


若要删除负载平衡器，请使用以下命令

	azure network lb delete -g nrprg -n ilbset 

其中 **nrprg** 是资源组，**ilbset** 是内部负载平衡器名称。


## 后续步骤

[使用源 IP 关联配置负载平衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载平衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0822_2016-->