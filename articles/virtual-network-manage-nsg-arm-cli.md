<!-- ARM: tested -->

<properties 
   pageTitle="在 Resource Manager 中使用 Azure CLI 管理 NSG | Azure"
   description="了解如何在 Resource Manager 中使用 Azure CLI 管理现有 NSG"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="03/14/2016"
	wacn.date="06/06/2016"/>

# 使用 Azure CLI 管理 NSG

[AZURE.INCLUDE [virtual-network-manage-arm-selectors-include.md](../includes/virtual-network-manage-nsg-arm-selectors-include.md)]

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

[AZURE.INCLUDE [virtual-network-manage-nsg-intro-include.md](../includes/virtual-network-manage-nsg-intro-include.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

[AZURE.INCLUDE [virtual-network-manage-nsg-arm-scenario-include.md](../includes/virtual-network-manage-nsg-arm-scenario-include.md)]

[AZURE.INCLUDE [azure-cli-prerequisites-include.md](../includes/azure-cli-prerequisites-include.md)]

## 检索信息

可以查看你的现有 NSG、检索现有 NSG 的规则和查找与 NSG 关联的资源。

### 查看现有 NSG

要查看特定资源组中 NSG 的列表，请运行 `azure network nsg list` 命令，如下所示。

	azure network nsg list --resource-group RG-NSG

预期输出：

	info:    Executing command network nsg list
	+ Getting the network security groups
	data:    Name          Location
	data:    ------------  --------
	data:    NSG-BackEnd   chinanorth
	data:    NSG-FrontEnd  chinanorth
	info:    network nsg list command OK
		 
### 列出 NSG 的所有规则

要查看名为 **NSG-FrontEnd** 的 NSG 的规则，请运行 `azure network nsg show` 命令，如下所示。

	azure network nsg show --resource-group RG-NSG --name NSG-FrontEnd

预期输出：
	
	info:    Executing command network nsg show
	+ Looking up the network security group "NSG-FrontEnd"
	data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd
	data:    Name                            : NSG-FrontEnd
	data:    Type                            : Microsoft.Network/networkSecurityGroups
	data:    Location                        : chinanorth
	data:    Provisioning state              : Succeeded
	data:    Tags                            : displayName=NSG - Front End
	data:    Security group rules:
	data:    Name                           Source IP          Source Port  Destination IP  Destination Port  Protocol  Direction  Access  Priority
	data:    -----------------------------  -----------------  -----------  --------------  ----------------  --------  ---------  ------  --------
	data:    rdp-rule                       Internet           *            *               3389              Tcp       Inbound    Allow   100
	data:    web-rule                       Internet           *            *               80                Tcp       Inbound    Allow   101
	data:    AllowVnetInBound               VirtualNetwork     *            VirtualNetwork  *                 *         Inbound    Allow   65000
	data:    AllowAzureLoadBalancerInBound  AzureLoadBalancer  *            *               *                 *         Inbound    Allow   65001
	data:    DenyAllInBound                 *                  *            *               *                 *         Inbound    Deny    65500
	data:    AllowVnetOutBound              VirtualNetwork     *            VirtualNetwork  *                 *         Outbound   Allow   65000
	data:    AllowInternetOutBound          *                  *            Internet        *                 *         Outbound   Allow   65001
	data:    DenyAllOutBound                *                  *            *               *                 *         Outbound   Deny    65500
	info:    network nsg show command OK

>[AZURE.NOTE] 还可以使用 `azure network nsg rule list --resource-group RG-NSG --nsg-name NSG-FrontEnd` 列出 **NSG-FrontEnd** NSG 中的规则。

###<a name="View-NSGs-associations"></a> 查看 NSG 关联项

若要查看与 **NSG-FrontEnd** NSG 相关联的资源，请运行 `azure network nsg show` 命令，如下所示。请注意，唯一区别是 **--json** 参数的使用。

	azure network nsg show --resource-group RG-NSG --name NSG-FrontEnd --json

查找 **networkInterfaces** 和 **subnets** 属性，如下所示：

	"networkInterfaces": [],
	...
	"subnets": [
		{
			"id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
		}
	],
	...

在上述示例中，NSG 不与任何网络接口 (NIC) 关联，而是与名为 **FrontEnd** 的子网关联。

## 管理规则

可向现有 NSG 添加规则、编辑现有规则和删除规则。

### 添加规则

若要向 **NSG-FrontEnd** NSG 添加规则，这一规则允许来自任何计算机的**入站**流量流入端口 **443**，请运行 `azure network nsg rule create` 命令，如下所示。

	azure network nsg rule create --resource-group RG-NSG \
		--nsg-name NSG-FrontEnd \
		--name allow-https \
		--description "Allow access to port 443 for HTTPS" \
		--protocol Tcp \
		--source-address-prefix * \
		--source-port-range * \
		--destination-address-prefix * \
		--destination-port-range 443 \
		--access Allow \
		--priority 102 \
		--direction Inbound		

预期输出：

	info:    Executing command network nsg rule create
	+ Looking up the network security rule "allow-https"
	+ Creating a network security rule "allow-https"
	+ Looking up the network security group "NSG-FrontEnd"
	data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/allow-https
	data:    Name                            : allow-https
	data:    Type                            : Microsoft.Network/networkSecurityGroups/securityRules
	data:    Provisioning state              : Succeeded
	data:    Description                     : Allow access to port 443 for HTTPS
	data:    Source IP                       : *
	data:    Source Port                     : *
	data:    Destination IP                  : *
	data:    Destination Port                : 443
	data:    Protocol                        : Tcp
	data:    Direction                       : Inbound
	data:    Access                          : Allow
	data:    Priority                        : 102
	info:    network nsg rule create command OK

### 更改规则

若要将上面创建的规则更改为仅允许来自 **Internet** 的入站流量，请 运行 `azure network nsg rule set` 命令，如下所示。

	azure network nsg rule set --resource-group RG-NSG \
		--nsg-name NSG-FrontEnd \
		--name allow-https \
		--source-address-prefix Internet

预期输出：

	info:    Executing command network nsg rule set
	+ Looking up the network security group "NSG-FrontEnd"
	+ Setting a network security rule "allow-https"
	+ Looking up the network security group "NSG-FrontEnd"
	data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/allow-https
	data:    Name                            : allow-https
	data:    Type                            : Microsoft.Network/networkSecurityGroups/securityRules
	data:    Provisioning state              : Succeeded
	data:    Description                     : Allow access to port 443 for HTTPS
	data:    Source IP                       : Internet
	data:    Source Port                     : *
	data:    Destination IP                  : *
	data:    Destination Port                : 443
	data:    Protocol                        : Tcp
	data:    Direction                       : Inbound
	data:    Access                          : Allow
	data:    Priority                        : 102
	info:    network nsg rule set command OK

### 删除规则

要删除上面创建的规则，请运行 `azure network nsg rule delete` 命令，如下所示。

	azure network nsg rule delete --resource-group RG-NSG \
		--nsg-name NSG-FrontEnd \
		--name allow-https \
		--quiet

>[AZURE.NOTE] **--quiet** 参数可确保无需确认删除。

预期输出：

	info:    Executing command network nsg rule delete
	+ Looking up the network security group "NSG-FrontEnd"
	+ Deleting network security rule "allow-https"
	info:    network nsg rule delete command OK

## 管理关联项

可将 NSG 关联到子网和 NIC。还可取消 NSG 与任何相关联的资源之间的关联。

### 将 NSG 关联到 NIC

要将 **NSG-FrontEnd** NSG 关联到 **TestNICWeb1** NIC，请运行 `azure network nic set` 命令，如下所示。

	azure network nic set --resource-group RG-NSG \
		--name TestNICWeb1 \
		--network-security-group-name NSG-FrontEnd

预期输出：

	info:    Executing command network nic set
	+ Looking up the network interface "TestNICWeb1"
	+ Looking up the network security group "NSG-FrontEnd"
	+ Updating network interface "TestNICWeb1"
	+ Looking up the network interface "TestNICWeb1"
	data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1
	data:    Name                            : TestNICWeb1
	data:    Type                            : Microsoft.Network/networkInterfaces
	data:    Location                        : chinanorth
	data:    Provisioning state              : Succeeded
	data:    MAC address                     : 00-0D-3A-30-A1-F8
	data:    Enable IP forwarding            : false
	data:    Tags                            : displayName=NetworkInterfaces - Web
	data:    Network security group          : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd
	data:    Virtual machine                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Compute/virtualMachines/Web1
	data:    IP configurations:
	data:      Name                          : ipconfig1
	data:      Provisioning state            : Succeeded
	data:      Public IP address             : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/publicIPAddresses/TestPIPWeb1
	data:      Private IP address            : 192.168.1.5
	data:      Private IP Allocation Method  : Dynamic
	data:      Subnet                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
	data:
	info:    network nic set command OK

###<a name="Dissociate-an-NSG-from-a-NIC"></a> 取消 NSG 与 NIC 之间的关联

要取消 **NSG-FrontEnd** NSG 与 **TestNICWeb1** NIC 之间的关联，请运行 `azure network nic set` 命令，如下所示。

	azure network nic set --resource-group RG-NSG --name TestNICWeb1 --network-security-group-id ""

>[AZURE.NOTE] 请注意 **network-security-group-id** 参数的“”（空）值。这就是删除 NSG 关联项的方式。不能对 **network-security-group-name** 参数执行相同操作。

预期结果：

	info:    Executing command network nic set
	+ Looking up the network interface "TestNICWeb1"
	+ Updating network interface "TestNICWeb1"
	+ Looking up the network interface "TestNICWeb1"
	data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1
	data:    Name                            : TestNICWeb1
	data:    Type                            : Microsoft.Network/networkInterfaces
	data:    Location                        : chinanorth
	data:    Provisioning state              : Succeeded
	data:    MAC address                     : 00-0D-3A-30-A1-F8
	data:    Enable IP forwarding            : false
	data:    Tags                            : displayName=NetworkInterfaces - Web
	data:    Virtual machine                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Compute/virtualMachines/Web1
	data:    IP configurations:
	data:      Name                          : ipconfig1
	data:      Provisioning state            : Succeeded
	data:      Public IP address             : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/publicIPAddresses/TestPIPWeb1
	data:      Private IP address            : 192.168.1.5
	data:      Private IP Allocation Method  : Dynamic
	data:      Subnet                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
	data:
	info:    network nic set command OK

###<a name="Dissociate-an-NSG-from-a-subnet"></a> 取消 NSG 与子网之间的关联

要取消 **NSG-FrontEnd** NSG 与 **FrontEnd** 子网之间的关联，请运行 `azure network vnet subnet set` 命令，如下所示。

	azure network vnet subnet set --resource-group RG-NSG \
		--vnet-name TestVNet \
		--name FrontEnd \
		--network-security-group-id ""

预期输出：

	info:    Executing command network vnet subnet set
	+ Looking up the subnet "FrontEnd"
	+ Setting subnet "FrontEnd"
	+ Looking up the subnet "FrontEnd"
	data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
	data:    Type                            : Microsoft.Network/virtualNetworks/subnets
	data:    ProvisioningState               : Succeeded
	data:    Name                            : FrontEnd
	data:    Address prefix                  : 192.168.1.0/24
	data:    IP configurations:
	data:      /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb2/ipConfigurations/ipconfig1
	data:      /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1/ipConfigurations/ipconfig1
	data:
	info:    network vnet subnet set command OK

### 将 NSG 关联到子网

要将 **NSG-FrontEnd** NSG 关联到 **FronEnd** 子网，请运行 `azure network vnet subnet set` 命令，如下所示。

	azure network vnet subnet set --resource-group RG-NSG \
		--vnet-name TestVNet \
		--name FrontEnd \
		--network-security-group-name NSG-FronEnd

>[AZURE.NOTE] 上述命令能起作用是因为 **NSG-FrontEnd** NSG 与虚拟网络 **TestVNet** 处于同一个资源组中。如果 NSG 在不同的资源组中，则需要改用 **--network-security-group-id** 参数，并提供 NSG 的完整 ID。可通过运行 **azure network nsg show --resource-group RG-NSG --name NSG-FrontEnd --json** 并查找 **id** 属性来检索其 ID。

预期输出：

		info:    Executing command network vnet subnet set
		+ Looking up the subnet "FrontEnd"
		+ Looking up the network security group "NSG-FrontEnd"
		+ Setting subnet "FrontEnd"
		+ Looking up the subnet "FrontEnd"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
		data:    Type                            : Microsoft.Network/virtualNetworks/subnets
		data:    ProvisioningState               : Succeeded
		data:    Name                            : FrontEnd
		data:    Address prefix                  : 192.168.1.0/24
		data:    Network security group          : [object Object]
		data:    IP configurations:
		data:      /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb2/ipConfigurations/ipconfig1
		data:      /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1/ipConfigurations/ipconfig1
		data:
		info:    network vnet subnet set command OK

## 删除 NSG

仅当 NSG 不与任何资源关联时，才能删除 NSG。若要删除 NSG，请按照以下步骤进行操作。

1. 若要检查与 NSG 关联的资源，请运行 `azure network nsg show`，如[查看 NSG 关联项](#View-NSGs-associations)中所示。
2. 如果 NSG 关联到任意 NIC，请为每个 NIC 运行 `azure network nic set`，如[取消 NSG 与 NIC 之间的关联](#Dissociate-an-NSG-from-a-NIC)中所示。 
3. 如果 NSG 关联到任意子网，请为每个子网运行 `azure network vnet subnet set`，如[取消 NSG 与子网之间的关联](#Dissociate-an-NSG-from-a-subnet)中所示。
4. 若要删除 NSG，请运行 `azure network nsg delete` 命令，如下所示。

		azure network nsg delete --resource-group RG-NSG --name NSG-FrontEnd --quiet

	预期输出：

		info:    Executing command network nsg delete
		+ Looking up the network security group "NSG-FrontEnd"
		+ Deleting network security group "NSG-FrontEnd"
		info:    network nsg delete command OK

## 后续步骤

- 为 NSG [启用日志记录](/documentation/articles/virtual-network-nsg-manage-log/)。

<!---HONumber=Mooncake_0516_2016-->