<properties
	pageTitle="Resource Manager 模式下的 Azure CLI 命令 | Azure"
	description="用于在 Resource Manager 部署模型中管理资源的 Azure 命令行界面 (CLI) 命令"
	services="virtual-machines-linux,virtual-machines-windows,virtual-network,mobile-services,cloud-services"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="multiple"
	ms.date="05/05/2016"
	wacn.date="06/27/2016"/>

# Resource Manager 模式下的 Azure CLI 命令

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是[经典部署模型](/documentation/articles/virtual-machines-command-line-tools/)。

本文提供经常用于在 Azure Resource Manager 部署模型中创建和管理 Azure 资源的 Azure 命令行界面 (CLI) 命令的语法和选项。通过在 Resource Manager (arm) 模式下运行 CLI 可以访问这些命令。本参考内容并不完整，你的 CLI 版本可能会显示略微不同的命令或参数。

若要开始，请先[安装 Azure CLI](/documentation/articles/xplat-cli-install/)，然后使用工作或学校帐户[连接到你的 Azure 订阅](/documentation/articles/xplat-cli-connect/)。

要在资源管理器模式下在命令行中查看当前的命令语法和选项，请键入 `azure help`；要显示某个命令的帮助，请键入 `azure help [command]`。你还可以在创建和管理具体 Azure 服务的说明文档中找到 CLI 示例。

可选参数显示在方括号中（例如，`[parameter]`）。其他所有参数都是必需的。

除了此处记录的特定于命令的可选参数外，还有三个可用于显示详细输出（例如请求选项和状态代码）的可选参数。`-v` 参数提供详细输出，而 `-vv` 参数提供更详细的输出。`--json` 选项将以原始的 json 格式输出结果。

## 设置 Resource Manager 模式

使用以下命令启用 Azure CLI Resource Manager 命令。

	azure config mode arm

>[AZURE.NOTE] Azure 资源管理器模式与 Azure 服务管理模式互斥。即在一种模式下创建的资源不能从另一种模式进行管理。

## 命令性和声明性方法

与 [Azure 服务管理模式](/documentation/articles/virtual-machines-command-line-tools/)一样，Azure CLI 的 Resource Manager 模式可提供命令让你在命令行上强制创建资源。例如，如果键入 `azure group create <groupname> <location>`，则会要求 Azure 创建资源组；如果键入 `azure group deployment create <resourcegroup> <deploymentname>`，则会指示 Azure 创建包含任意项数的部署，并将其放在组中。由于每种类型的资源都有强制命令，你可以将这些命令链接在一起，以创建相当复杂的部署。

但是，使用用于描述资源组的资源组模板是一种强大得多的声明性方法，它允许你针对（几乎）任何目的自动完成包含（几乎）任意数量的资源的复杂部署。使用模板时，唯一的强制性命令是单一部署。有关模板、资源和资源组的一般概述，请参阅 [Azure 资源组概述](/documentation/articles/resource-group-overview/)。


## azure account：管理你的帐户信息
该工具使用你的 Azure 订阅信息连接到你的帐户。

**列出导入的订阅**

	account list [options]

**显示有关订阅的详细信息**

	account show [options] [subscriptionNameOrId]

**设置当前订阅**

	account set [options] <subscriptionNameOrId>

**删除订阅或环境，或清除所有存储的帐户和环境信息**

	account clear [options]

**用于管理帐户环境的命令**

	account env list [options]
	account env show [options] [environment]
	account env add [options] [environment]
	account env set [options] [environment]
	account env delete [options] [environment]

## azure ad：用于显示 Active Directory 对象的命令

**用于显示 Active Directory 应用程序的命令**

	ad app create [options]
	ad app delete [options] <object-id>

**用于显示 Active Directory 组的命令**

	ad group list [options]
	ad group show [options]

**用于提供 Active Directory 子组或成员信息的命令**

	ad group member list [options] [objectId]

**用于显示 Active Directory 服务主体的命令**

	ad sp list [options]
	ad sp show [options]
	ad sp create [options] <application-id>
	ad sp delete [options] <object-id>

**用于显示 Active Directory 用户的命令**

	ad user list [options]
	ad user show [options]

##<a name="azure-availset-commands-to-manage-your-availability-sets"></a> azure availset：用于管理可用性集的命令

**在资源组中创建可用性集**

	availset create [options] <resource-group> <name> <location> [tags]

**列出资源组中的可用性集**

	availset list [options] <resource-group>

**获取资源组中的一个可用性集**

	availset show [options] <resource-group> <name>

**删除资源组中的一个可用性集**

	availset delete [options] <resource-group> <name>

## azure config：用于管理本地设置的命令

**列出 Azure CLI 配置设置**

	conconfig list [options]

**删除配置设置**

	conconfig delete [options] <name>

**更新配置设置**

	conconfig set <name> <value>

**将 Azure CLI 工作模式设置为 arm 或 asm**

	conconfig mode [options] <modename>

## azure group：用于管理资源组的命令

**创建新的资源组**

	group create [options] <name> <location>

**向资源组设置标记**

	group set [options] <name> <tags>

**删除资源组**

	group delete [options] <name>

**列出订阅的资源组**

	group list [options]

**显示订阅的资源组**

	group show [options] <name>

**用于管理资源组日志的命令**

	group log show [options] [name]

**用于管理资源组中的部署的命令**

	group deployment create [options] [resource-group] [name]
	group deployment list [options] <resource-group> [state]
	group deployment show [options] <resource-group> [deployment-name]
	group deployment stop [options] <resource-group> [deployment-name]

**用于管理本地或库资源组模板的命令**

	group template list [options]
	group template show [options] <name>
	group template download [options] [name] [file]
	group template validate [options] <resource-group>

## azure insights：与监视 Insights（事件、警报规则、自动缩放设置、度量值）相关的命令

**检索订阅、correlationId、资源组、资源或资源提供程序的操作日志**

	insights logs list [options]

## azure location：用于获取所有资源类型的可用位置的命令

**列出可用位置**

	location list [options]

## azure network：用于管理网络资源的命令

**用于管理虚拟网络的命令**

	network vnet create [options] <resource-group> <name> <location>
用于创建新的虚拟网络。在以下示例中，我们将为中国北部区域的资源组 myresourcegroup 创建名为 newvnet 的虚拟网络。


	azure network vnet create myresourcegroup newvnet "China North"
	info:    Executing command network vnet create
	+ Looking up virtual network "newvnet"
	+ Creating virtual network "newvnet"
	 Loading virtual network state
	data:    Id:                   /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/newvnet
	data:    Name:                 newvnet
	data:    Type:                 Microsoft.Network/virtualNetworks
	data:    Location:             chinanorth
	data:    Tags:
	data:    Provisioning state:   Succeeded
	data:    Address prefixes:
	data:     10.0.0.0/8
	data:    DNS servers:
	data:    Subnets:
	data:
	info:    network vnet create command OK


参数选项：

 	-h, --help                                 output usage information
 	-v, --verbose                              use verbose output
	--json                                     use json output
 	-g, --resource-group <resource-group>      the name of the resource group
 	-n, --name <name>                          the name of the virtual network
 	-l, --location <location>                  the location
 	-a, --address-prefixes <address-prefixes>  the comma separated list of address prefixes for this virtual network
      For example -a 10.0.0.0/24,10.0.1.0/24.
      Default value is 10.0.0.0/8

	-d, --dns-servers <dns-servers>            the comma separated list of DNS servers IP addresses
 	-t, --tags <tags>                          the tags set on this virtual network.
      Can be multiple. In the format of "name=value".
      Name is required and value is optional.
      For example, -t tag1=value1;tag2
	 -s, --subscription <subscription>          the subscription identifier

<BR>

	network vnet set [options] <resource-group> <name>

更新资源组中的虚拟网络配置。

	azure network vnet set myresourcegroup newvnet

	info:    Executing command network vnet set
	+ Looking up virtual network "newvnet"
	+ Updating virtual network "newvnet"
	+ Loading virtual network state
	data:    Id:                   /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/newvnet
	data:    Name:                 newvnet
	data:    Type:                 Microsoft.Network/virtualNetworks
	data:    Location:             chinanorth
	data:    Tags:
	data:    Provisioning state:   Succeeded
	data:    Address prefixes:
	data:     10.0.0.0/8
	data:    DNS servers:
	data:    Subnets:
	data:
	info:    network vnet set command OK

参数选项：

	   -h, --help                                 output usage information
	   -v, --verbose                              use verbose output
	   --json                                     use json output
	   -g, --resource-group <resource-group>      the name of the resource group
	   -n, --name <name>                          the name of the virtual network
	   -a, --address-prefixes <address-prefixes>  the comma separated list of address prefixes for this virtual network.
        For example -a 10.0.0.0/24,10.0.1.0/24.
        This list will be appended to the current list of address prefixes.
        The address prefixes in this list should not overlap between them.
        The address prefixes in this list should not overlap with existing address prefixes in the vnet.

	   -d, --dns-servers [dns-servers]            the comma separated list of DNS servers IP addresses.
        This list will be appended to the current list of DNS server IP addresses.

	   -t, --tags <tags>                          the tags set on this virtual network.
        Can be multiple. In the format of "name=value".
        Name is required and value is optional. For example, -t tag1=value1;tag2.
        This list will be appended to the current list of tags

	   --no-tags                                  remove all existing tags
	   -s, --subscription <subscription>          the subscription identifier
<BR>

	network vnet list [options] <resource-group>

使用该命令可列出资源组中的所有虚拟网络。


	C:\>azure network vnet list myresourcegroup

	info:    Executing command network vnet list
	+ Listing virtual networks
		data:    ID
       Name      Location  Address prefixes  DNS servers
	data:    -------------------------------------------------------------------
	------  --------  --------  ----------------  -----------
	data:    /subscriptions/###############################/resourceGroups/
	wvnet   newvnet   chinanorth    10.0.0.0/8
	info:    network vnet list command OK

参数选项：


      -h, --help                             output usage information
      -v, --verbose                          use verbose output
      --json                                 use json output
      -g, --resource-group <resource-group>  the name of the resource group
      -s, --subscription <subscription>      the subscription identifier

<BR>

	network vnet show [options] <resource-group> <name>

该命令显示资源组中的虚拟网络属性。

	azure network vnet show -g myresourcegroup -n newvnet

	info:    Executing command network vnet show
	+ Looking up virtual network "newvnet"
	data:    Id:                   /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/newvnet
	data:    Name:                 newvnet
	data:    Type:                 Microsoft.Network/virtualNetworks
	data:    Location:             chinanorth
	data:    Tags:
	data:    Provisioning state:   Succeeded
	data:    Address prefixes:
	data:     10.0.0.0/8
	data:    DNS servers:
	data:    Subnets:
	data:
	info:    network vnet show command OK
<BR>

	network vnet delete [options] <resource-group> <name>

该命令删除虚拟网络。

	azure network vnet delete myresourcegroup newvnetX

	info:    Executing command network vnet delete
	+ Looking up virtual network "newvnetX"
	Delete virtual network newvnetX? [y/n] y
	+ Deleting virtual network "newvnetX"
	info:    network vnet delete command OK

参数选项：

     -h, --help                             output usage information
     -v, --verbose                          use verbose output
     --json                                 use json output
     -g, --resource-group <resource-group>  the name of the resource group
     -n, --name <name>                      the name of the virtual network
     -q, --quiet                            quiet mode, do not ask for delete confirmation
     -s, --subscription <subscription>      the subscription identifier


**用于管理虚拟网络子网的命令**

	network vnet subnet create [options] <resource-group> <vnet-name> <name>
该命令允许将另一个子网添加到现有的虚拟网络。

	azure network vnet subnet create -g myresourcegroup --vnet-name newvnet -n subnet --address-prefix 10.0.1.0/24

	info:    Executing command network vnet subnet create
	+ Looking up the subnet "subnet"
	+ Creating subnet "subnet"
	+ Looking up the subnet "subnet"
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/newvnet/subnets/subnet
	data:    Name:                      subnet
	data:    Type:                      Microsoft.Network/virtualNetworks/subnets
	data:    Provisioning state:        Succeeded
	data:    Address prefix:            10.0.1.0/24
	info:    network vnet subnet create command OK

参数选项：

     -h, --help                                                       output usage information
     -v, --verbose                                                    use verbose output
		 --json                                                           use json output
	 -g, --resource-group <resource-group>                            the name of the resource group
	 -e, --vnet-name <vnet-name>                                      the name of the virtual network
     -n, --name <name>                                                the name of the subnet
     -a, --address-prefix <address-prefix>                            the address prefix
     -w, --network-security-group-id <network-security-group-id>      the network security group identifier.
           e.g. /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/networkSecurityGroups/<nsg-name>
     -o, --network-security-group-name <network-security-group-name>  the network security group name
     -s, --subscription <subscription>                                the subscription identifier

<BR>

	network vnet subnet set [options] <resource-group> <vnet-name> <name>

在资源组中设置特定的虚拟网络子网。


	C:\>azure network vnet subnet set -g myresourcegroup --vnet-name newvnet -n subnet1

	info:    Executing command network vnet subnet set
	+ Looking up the subnet "subnet1"
	+ Setting subnet "subnet1"
	+ Looking up the subnet "subnet1"
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/newvnet/subnets/subnet1
	data:    Name:                      subnet1
	data:    Type:                      Microsoft.Network/virtualNetworks/subnets
	data:    Provisioning state:        Succeeded
	data:    Address prefix:            10.0.1.0/24
	info:    network vnet subnet set command OK
<BR>

	network vnet subnet list [options] <resource-group> <vnet-name>

列出资源组中特定虚拟网络的所有虚拟网络子网。

	azure network vnet subnet set -g myresourcegroup --vnet-name newvnet -n subnet1

	info:    Executing command network vnet subnet set
	+ Looking up the subnet "subnet1"
	+ Setting subnet "subnet1"
	+ Looking up the subnet "subnet1"
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/newvnet/subnets/subnet1
	data:    Name:                      subnet1
	data:    Type:                      Microsoft.Network/virtualNetworks/subnets
	data:    Provisioning state:        Succeeded
	data:    Address prefix:            10.0.1.0/24
	info:    network vnet subnet set command OK
<BR>

	network vnet subnet show [options] <resource-group> <vnet-name> <name>

显示虚拟网络子网属性

	azure network vnet subnet show -g myresourcegroup --vnet-name newvnet -n subnet1

	info:    Executing command network vnet subnet show
	+ Looking up the subnet "subnet1"
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft
	.Network/virtualNetworks/newvnet/subnets/subnet1
	data:    Name:                      subnet1
	data:    Type:                      Microsoft.Network/virtualNetworks/subnets
	data:    Provisioning state:        Succeeded
	data:    Address prefix:            10.0.1.0/24
	info:    network vnet subnet show command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-e, --vnet-name <vnet-name>            the name of the virtual network
	-n, --name <name>                      the name of the subnet
	-s, --subscription <subscription>      the subscription identifier
<BR>

	network vnet subnet delete [options] <resource-group> <vnet-name> <subnet-name>

从现有虚拟网络中删除子网。

	azure network vnet subnet delete -g myresourcegroup --vnet-name newvnet -n subnet1

	info:    Executing command network vnet subnet delete
	+ Looking up the subnet "subnet1"
	Delete subnet "subnet1"? [y/n] y
	+ Deleting subnet "subnet1"
	info:    network vnet subnet delete command OK

参数选项：

 	-h, --help                             output usage information
 	-v, --verbose                          use verbose output
 	--json                                 use json output
 	-g, --resource-group <resource-group>  the name of the resource group
 	-e, --vnet-name <vnet-name>            the name of the virtual network
 	-n, --name <name>                      the subnet name
 	-s, --subscription <subscription>      the subscription identifier
 	-q, --quiet                            quiet mode, do not ask for delete confirmation

**用于管理负载平衡器的命令**

	network lb create [options] <resource-group> <name> <location>

创建负载平衡器集。

	azure network lb create -g myresourcegroup -n mylb -l chinanorth

	info:    Executing command network lb create
	+ Looking up the load balancer "mylb"
	+ Creating load balancer "mylb"
	+ Looking up the load balancer "mylb"
	data:    Id:                           /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb
	data:    Name:                         mylb
	data:    Type:                         Microsoft.Network/loadBalancers
	data:    Location:                     chinanorth
	data:    Provisioning state:           Succeeded
	info:    network lb create command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-n, --name <name>                      the name of the load balancer
	-l, --location <location>              the location
	-t, --tags <tags>                      the list of tags.
     Can be multiple. In the format of "name=value".
     Name is required and value is optional. For example, -t tag1=value1;tag2
	-s, --subscription <subscription>      the subscription identifier
<BR>

	network lb list [options] <resource-group>

列出资源组中的负载平衡器资源。

	azure network lb list myresourcegroup

	info:    Executing command network lb list
	+ Getting the load balancers
	data:    Name  Location
	data:    ----  --------
	data:    mylb  chinanorth
	info:    network lb list command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-s, --subscription <subscription>      the subscription identifier
<BR>

	network lb show [options] <resource-group> <name>

显示资源组中特定负载平衡器的负载平衡器信息

	azure network lb show myresourcegroup mylb -v

	info:    Executing command network lb show
	verbose: Looking up the load balancer "mylb"
	data:    Id:                           /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb
	data:    Name:                         mylb
	data:    Type:                         Microsoft.Network/loadBalancers
	data:    Location:                     chinanorth
	data:    Provisioning state:           Succeeded
	info:    network lb show command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-n, --name <name>                      the name of the load balancer
	-s, --subscription <subscription>      the subscription identifier

<BR>

	network lb delete [options] <resource-group> <name>

删除负载平衡器资源。

	azure network lb delete  myresourcegroup mylb

	info:    Executing command network lb delete
	+ Looking up the load balancer "mylb"
	Delete load balancer "mylb"? [y/n] y
	+ Deleting load balancer "mylb"
	info:    network lb delete command OK

参数选项：

 	-h, --help                             output usage information
 	-v, --verbose                          use verbose output
 	--json                                 use json output
 	-g, --resource-group <resource-group>  the name of the resource group
 	-n, --name <name>                      the name of the load balancer
 	-q, --quiet                            quiet mode, do not ask for delete confirmation
 	-s, --subscription <subscription>      the subscription identifier

**用于管理负载平衡器探测的命令**

	network lb probe create [options] <resource-group> <lb-name> <name>

在负载平衡器中创建运行状况状态探测配置。请记住，若要运行此命令，负载平衡器需要一个前端 IP 资源（发出命令“azure network frontend-ip”可向负载平衡器分配 IP 地址）。

	azure network lb probe create -g myresourcegroup --lb-name mylb -n mylbprobe --protocol tcp --port 80 -i 300

	info:    Executing command network lb probe create
	+ Looking up the load balancer "mylb"
	+ Updating load balancer "mylb"
	info:    network lb probe create command OK

参数选项：

 	-h, --help                             output usage information
 	-v, --verbose                          use verbose output
 	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the probe
	-p, --protocol <protocol>              the probe protocol
	-o, --port <port>                      the probe port
	-f, --path <path>                      the probe path
	-i, --interval <interval>              the probe interval in seconds
	-c, --count <count>                    the number of probes
	-s, --subscription <subscription>      the subscription identifier

<BR>

	network lb probe set [options] <resource-group> <lb-name> <name>

使用新值更新现有负载平衡器探测。

	azure network lb probe set -g myresourcegroup -l mylb -n mylbprobe -p mylbprobe1 -p TCP -o 443 -i 300

	info:    Executing command network lb probe set
		+ Looking up the load balancer "mylb"
	+ Updating load balancer "mylb"
	info:    network lb probe set command OK

参数选项

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the probe
	-e, --new-probe-name <new-probe-name>  the new name of the probe
	-p, --protocol <protocol>              the new value for probe protocol
	-o, --port <port>                      the new value for probe port
	-f, --path <path>                      the new value for probe path
	-i, --interval <interval>              the new value for probe interval in seconds
	-c, --count <count>                    the new value for number of probes
	-s, --subscription <subscription>      the subscription identifier
<BR>

	network lb probe list [options] <resource-group> <lb-name>

列出负载平衡器集的探测属性。

	C:\>azure network lb probe list -g myresourcegroup -l mylb

	info:    Executing command network lb probe list
	+ Looking up the load balancer "mylb"
	data:    Name       Protocol  Port  Path  Interval  Count
	data:    ---------  --------  ----  ----  --------  -----
	data:    mylbprobe  Tcp       443         300       2
	info:    network lb probe list command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-s, --subscription <subscription>      the subscription identifier


	network lb probe delete [options] <resource-group> <lb-name> <name>

删除为负载平衡器创建的探测。

	azure network lb probe delete -g myresourcegroup -l mylb -n mylbprobe

	info:    Executing command network lb probe delete
	+ Looking up the load balancer "mylb"
	Delete a probe "mylbprobe?" [y/n] y
	+ Updating load balancer "mylb"
	info:    network lb probe delete command OK

**用于管理负载平衡器前端 IP 配置的命令**

	network lb frontend-ip create [options] <resource-group> <lb-name> <name>

为现有的负载平衡器集创建前端 IP 配置。

	azure network lb frontend-ip create -g myresourcegroup --lb-name mylb -n myfrontendip -o Dynamic -e subnet -m newvnet

	info:    Executing command network lb frontend-ip create
	+ Looking up the load balancer "mylb"
	+ Looking up the subnet "subnet"
	+ Creating frontend IP configuration "myfrontendip"
	+ Looking up the load balancer "mylb"
	data:    Id:                           /subscriptions/###############################/resourceGroups/Myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb
	/frontendIPConfigurations/myfrontendip
	data:    Name:                         myfrontendip
	data:    Type:                         Microsoft.Network/loadBalancers/frontendIPConfigurations
	data:    Provisioning state:           Succeeded
	data:    Private IP allocation method: Dynamic
	data:    Private IP address:           10.0.1.4
	data:    Subnet:                       id=/subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/newvnet/subnets/subnet
	data:    Public IP address:
	data:    Inbound NAT rules
	data:    Outbound NAT rules
	data:    Load balancing rules
	data:
	info:    network lb frontend-ip create command OK

<BR>

	network lb frontend-ip set [options] <resource-group> <lb-name> <name>

用于更新现有的前端 IP 配置。以下命令将名为 mypubip5 的公共 IP 添加到名为 myfrontendip 的现有负载平衡器前端 IP。

	azure network lb frontend-ip set -g myresourcegroup --lb-name mylb -n myfrontendip -i mypubip5

	info:    Executing command network lb frontend-ip set
	+ Looking up the load balancer "mylb"
	+ Looking up the public ip "mypubip5"
	+ Updating load balancer "mylb"
	+ Looking up the load balancer "mylb"
	data:    Id:                           /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/frontendIPConfigurations/myfrontendip
	data:    Name:                         myfrontendip
	data:    Type:                         Microsoft.Network/loadBalancers/frontendIPConfigurations
	data:    Provisioning state:           Succeeded
	data:    Private IP allocation method: Dynamic
	data:    Private IP address:
	data:    Subnet:
	data:    Public IP address:            id=/subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/publicIPAddresses/mypubip5
	data:    Inbound NAT rules
	data:    Outbound NAT rules
	data:    Load balancing rules
	data:
	info:    network lb frontend-ip set command OK

参数选项：

	-h, --help                                                         output usage information
	-v, --verbose                                                      use verbose output
	--json                                                             use json output
	-g, --resource-group <resource-group>                              the name of the resource group
	-l, --lb-name <lb-name>                                            the name of the load balancer
	-n, --name <name>                                                  the name of the frontend ip configuration
	-a, --private-ip-address <private-ip-address>                      the private ip address
	-o, --private-ip-allocation-method <private-ip-allocation-method>  the private ip allocation method [Static, Dynamic]
	-u, --public-ip-id <public-ip-id>                                  the public ip identifier.
	e.g. /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/publicIPAddresses/<public-ip-name>
	-i, --public-ip-name <public-ip-name>                              the public ip name.
	This public ip must exist in the same resource group as the lb.
	Please use public-ip-id if that is not the case.
	-b, --subnet-id <subnet-id>                                        the subnet id.
	e.g. /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/VirtualNetworks/<vnet-name>/subnets/<subnet-name>
	-e, --subnet-name <subnet-name>                                    the subnet name
	-m, --vnet-name <vnet-name>                                        the virtual network name.
	This virtual network must exist in the same resource group as the lb.
	Please use subnet-id if that is not the case.
	-s, --subscription <subscription>                                  the subscription identifier

<BR>

	network lb frontend-ip list [options] <resource-group> <lb-name>

列出针对负载平衡器配置的所有前端 IP 资源。

	azure network lb frontend-ip list -g myresourcegroup -l mylb

	info:    Executing command network lb frontend-ip list
	+ Looking up the load balancer "mylb"
	data:    Name         Provisioning state  Private IP allocation method  Subnet
	data:    -----------  ------------------  ----------------------------  ------
	data:    myprivateip  Succeeded           Dynamic
	info:    network lb frontend-ip list command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-s, --subscription <subscription>      the subscription identifier
<BR>

	network lb frontend-ip delete [options] <resource-group> <lb-name> <name>

删除与负载平衡器关联的前端 IP 对象

	network lb frontend-ip delete -g myresourcegroup -l mylb -n myfrontendip
	info:    Executing command network lb frontend-ip delete
	+ Looking up the load balancer "mylb"
	Delete frontend ip configuration "myfrontendip"? [y/n] y
	+ Updating load balancer "mylb"

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the frontend ip configuration
	-q, --quiet                            quiet mode, do not ask for delete confirmation
	-s, --subscription <subscription>      the subscription identifier

**用于管理负载平衡器后端地址池的命令**

	network lb address-pool create [options] <resource-group> <lb-name> <name>

为负载平衡器创建后端地址池。

	azure network lb address-pool create -g myresourcegroup --lb-name mylb -n myaddresspool

	info:    Executing command network lb address-pool create
	+ Looking up the load balancer "mylb"
	+ Updating load balancer "mylb"
	+ Looking up the load balancer "mylb"
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourgroup/providers/Microso.Network/loadBalancers/mylb/backendAddressPools/myaddresspool
	data:    Name:                      myaddresspool
	data:    Type:                      Microsoft.Network/loadBalancers/backendAddressPools
	data:    Provisioning state:        Succeeded
	data:    Backend IP configurations:
	data:    Load balancing rules:
	data:
	info:    network lb address-pool create command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the backend address pool
	-s, --subscription <subscription>      the subscription identifier

<BR>

	network lb address-pool add [options] <resource-group> <lb-name> <name>

负载平衡器根据后端地址池范围来确定哪些资源正在使用 Azure 资源管理器来从其终结点路由传入的网络流量。在创建并命名后端地址池范围后（请参阅命令“azure network lb address-pool create”），需要添加现在已由名为“网络接口”的资源定义的终结点。

若要配置后端地址范围，你至少需要一个“网络接口”（有关详细信息，请参阅“azure network lb nic”命令行）。

在以下示例中，使用了以前创建的“nic1”网络接口来创建后端地址池范围。

	azure network lb address-pool add -g myresourcegroup -l mylb -n mybackendpool -a nic1

	info:    Executing command network lb address-pool add
	+ Looking up the load balancer "mylb"
	+ Getting network interfaces
	+ Updating network interface "nic1"
	+ Looking up the load balancer "mylb"
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/mybackendpool
	data:    Name:                      mybackendpool
	data:    Type:                      Microsoft.Network/loadBalancers/backendAddressPools
	data:    Provisioning state:        Succeeded
	data:    Backend IP configurations:
	data:     id=/subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/networkInterfaces/nic1/ipConfigurations/NIC-config
	data:    Load balancing rules:
	data:
	info:    network lb address-pool add command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the backend address pool
	-i, --vm-id <vm-id>                    the virtual machine identifier.
	e.g. "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines/<vm-name>,/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines/<vm-name>"
	-m, --vm-name <vm-name>                the name of the virtual machine
	-d, --nic-id <nic-id>                  the network interface identifier.
	e.g. ""/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/networkInterfaces/<nic-name>"
	-a, --nic-name <nic-name>              the name of the network interface
	-s, --subscription <subscription>      the subscription identifier

<BR>

	network lb address-pool remove [options] <resource-group> <lb-name> <name>

从后端 IP 地址池范围中删除网络接口。

	azure network lb address-pool remove -g myresourcegroup -l mylb -n mybackendpool -a nic1

	info:    Executing command network lb address-pool remove
	+ Looking up the load balancer "mylb"
	+ Getting network interfaces
	+ Updating network interface "nic1"
	+ Looking up the load balancer "mylb"
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/mybackendpool
	data:    Name:                      mybackendpool
	data:    Type:                      Microsoft.Network/loadBalancers/backendAddressPools
	data:    Provisioning state:        Succeeded
	data:    Backend IP configurations:
	data:    Load balancing rules:
	data:
	info:    network lb address-pool remove command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the backend address pool
	-i, --vm-id <vm-id>                    the virtual machine identifier.
	e.g. "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines/<vm-name>,/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines/<vm-name>"
	-m, --vm-name <vm-name>                the name of the virtual machine
	-d, --nic-id <nic-id>                  the network interface identifier.
	e.g. ""/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/networkInterfaces/<nic-name>"
	-a, --nic-name <nic-name>              the name of the network interface
	-s, --subscription <subscription>      the subscription identifier
<BR>

	network lb address-pool list [options] <resource-group> <lb-name>

列出特定资源组的后端 IP 地址池范围

	azure network lb address-pool list -g myresourcegroup -l mylb

	info:    Executing command network lb address-pool list
	+ Looking up the load balancer "mylb"
	data:    Name           Provisioning state
	data:    -------------  ------------------
	data:    mybackendpool  Succeeded
	info:    network lb address-pool list command OK

参数选项：

 	-h, --help                             output usage information
 	-v, --verbose                          use verbose output
 	--json                                 use json output
 	-g, --resource-group <resource-group>  the name of the resource group
 	-l, --lb-name <lb-name>                the name of the load balancer
 	-s, --subscription <subscription>      the subscription identifier

<BR> network lb address-pool delete [选项] <resource-group> <lb-name> <name>

从负载平衡器中删除后端 IP 池范围资源。

	azure network lb address-pool delete -g myresourcegroup -l mylb -n mybackendpool

	info:    Executing command network lb address-pool delete
	+ Looking up the load balancer "mylb"
	Delete backend address pool "mybackendpool"? [y/n] y
	+ Updating load balancer "mylb"
	info:    network lb address-pool delete command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the backend address pool
	-q, --quiet                            quiet mode, do not ask for delete confirmation
	-s, --subscription <subscription>      the subscription identifier

**用于管理负载平衡器规则的命令**

	network lb rule create [options] <resource-group> <lb-name> <name>

创建负载平衡器规则。

你可以创建负载平衡器规则，用于配置负载平衡器的前端终结点以及要接收传入网络流量的后端地址池范围。设置还包括前端 IP 终结点的端口，以及后端地址池范围的端口。

以下示例演示了如何创建负载平衡器规则、侦听端口 80 TCP 的前端终结点，以及发送到后端地址池范围的端口 8080 的负载平衡网络流量。

	azure network lb rule create -g myresourcegroup -l mylb -n mylbrule -p tcp -f 80 -b 8080 -i 10


	info:    Executing command network lb rule create
	+ Looking up the load balancer "mylb"
	+ Updating load balancer "mylb"
	+ Loading rule state
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/loadBalancingRules/mylbrule
	data:    Name:                      mylbrule
	data:    Type:                      Microsoft.Network/loadBalancers/loadBalancingRules
	data:    Provisioning state:        Succeeded
	data:    Frontend IP configuration: /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/frontendIPConfigurations/myfrontendip
	data:    Backend address pool:      id=/subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/mybackendpool
	data:    Protocol:                  Tcp
	data:    Frontend port:             80
	data:    Backend port:              8080
	data:    Enable floating IP:        false
	data:    Idle timeout in minutes:   10
	data:    Probes
	data:
	info:    network lb rule create command OK

<BR>

	network lb rule set [options] <resource-group> <lb-name> <name>

更新特定资源组中设置的现有负载平衡器规则。在以下示例中，我们已将规则名称从 mylbrule 更改为 mynewlbrule。

	azure network lb rule set -g myresourcegroup -l mylb -n mylbrule -r mynewlbrule -p tcp -f 80 -b 8080 -i 10 -t myfrontendip -o mybackendpool

	info:    Executing command network lb rule set
	+ Looking up the load balancer "mylb"
	+ Updating load balancer "mylb"
	+ Loading rule state
	data:    Id:                        /subscriptions/###############################/resourceGroups/yresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/loadBalancingRules/mynewlbrule
	data:    Name:                      mynewlbrule
	data:    Type:                      Microsoft.Network/loadBalancers/loadBalancingRules
	data:    Provisioning state:        Succeeded
	data:    Frontend IP configuration: /subscriptions/###############################/resourceGroups/yresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/frontendIPConfigurations/myfrontendip
	data:    Backend address pool:      id=/subscriptions/###############################/resourceGroups/yresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/mybackendpool
	data:    Protocol:                  Tcp
	data:    Frontend port:             80
	data:    Backend port:              8080
	data:    Enable floating IP:        false
	data:    Idle timeout in minutes:   10
	data:    Probes
	data:
	info:    network lb rule set command OK

参数选项：

	-h, --help                                         output usage information
	-v, --verbose                                      use verbose output
	--json                                             use json output
	-g, --resource-group <resource-group>              the name of the resource group
	-l, --lb-name <lb-name>                            the name of the load balancer
	-n, --name <name>                                  the name of the rule
	-r, --new-rule-name <new-rule-name>                new rule name
	-p, --protocol <protocol>                          the rule protocol
	-f, --frontend-port <frontend-port>                the frontend port
	-b, --backend-port <backend-port>                  the backend port
	-e, --enable-floating-ip <enable-floating-ip>      enable floating point ip
	-i, --idle-timeout <idle-timeout>                  the idle timeout in minutes
	-a, --probe-name [probe-name]                      the name of the probe defined in the same load balancer
	-t, --frontend-ip-name <frontend-ip-name>          the name of the frontend ip configuration in the same load balancer
	-o, --backend-address-pool <backend-address-pool>  name of the backend address pool defined in the same load balancer
	-s, --subscription <subscription>                  the subscription identifier


	network lb rule list [options] <resource-group> <lb-name>

列出针对特定资源组中某个负载平衡器配置的所有负载平衡器规则。

	azure network lb rule list -g myresourcegroup -l mylb

	info:    Executing command network lb rule list
	+ Looking up the load balancer "mylb"
	data:    Name         Provisioning state  Protocol  Frontend port  Backend port  Enable floating IP  Idle timeout in minutes  Backend address pool  Probe data

	data:    mynewlbrule  Succeeded           Tcp       80             8080          false               10                       /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/mybackendpool
	info:    network lb rule list command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-s, --subscription <subscription>      the subscription identifier

	network lb rule delete [options] <resource-group> <lb-name> <name>

删除负载平衡器规则。

	azure network lb rule delete -g myresourcegroup -l mylb -n mynewlbrule

	info:    Executing command network lb rule delete
	+ Looking up the load balancer "mylb"
	Delete load balancing rule mynewlbrule? [y/n] y
	+ Updating load balancer "mylb"
	info:    network lb rule delete command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the rule
	-q, --quiet                            quiet mode, do not ask for delete confirmation
	-s, --subscription <subscription>      the subscription identifier

**用于管理负载平衡器入站 NAT 规则的命令**

	network lb inbound-nat-rule create [options] <resource-group> <lb-name> <name>

为负载平衡器创建入站 NAT 规则。

在以下示例中，我们从前端 IP（前面已定义。有关详细信息，请参阅“azure network frontend-ip”命令），使用入站侦听端口和负载平衡器要将网络流量发送到的出站端口，创建了一个 NAT 规则。


	azure network lb inbound-nat-rule create -g myresourcegroup -l mylb -n myinboundnat -p tcp -f 80 -b 8080 -i myfrontendip

	info:    Executing command network lb inbound-nat-rule create
	+ Looking up the load balancer "mylb"
	+ Updating load balancer "mylb"
	+ Looking up the load balancer "mylb"
	data:    Id:                        /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/inboundNatRules/myinboundnat
	data:    Name:                      myinboundnat
	data:    Type:                      Microsoft.Network/loadBalancers/inboundNatRules
	data:    Provisioning state:        Succeeded
	data:    Frontend IP Configuration: id=/subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/loadBalancers/mylb/frontendIPConfigurations/myfrontendip
	data:    Backend IP configuration
	data:    Protocol                   Tcp
	data:    Frontend port              80
	data:    Backend port               8080
	data:    Enable floating IP         false
	info:    network lb inbound-nat-rule create command OK

参数选项：

	-h, --help                                     output usage information
	-v, --verbose                                  use verbose output
	--json                                         use json output
	-g, --resource-group <resource-group>          the name of the resource group
	-l, --lb-name <lb-name>                        the name of the load balancer
	-n, --name <name>                              the name of the inbound NAT rule
	-p, --protocol <protocol>                      the rule protocol [tcp,udp]
	-f, --frontend-port <frontend-port>            the frontend port [0-65535]
	-b, --backend-port <backend-port>              the backend port [0-65535]
	-e, --enable-floating-ip <enable-floating-ip>  enable floating point ip [true,false]
	-i, --frontend-ip <frontend-ip>                the name of the frontend ip configuration
	-m, --vm-id <vm-id>                            the VM id.
	e.g. /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines/<vm-name>
	-a, --vm-name <vm-name>                        the VM name.This VM must exist in the same resource group as the lb.
	Please use vm-id if that is not the case.
	this parameter will be ignored if --vm-id is specified
	-s, --subscription <subscription>              the subscription identifier
<BR>

	network lb inbound-nat-rule set [options] <resource-group> <lb-name> <name>

更新现有的入站 NAT 规则。在以下示例中，我们已将入站侦听端口从 80 更改为 81。

	azure network lb inbound-nat-rule set -g group-1 -l mylb -n myinboundnat -p tcp -f 81 -b 8080 -i myfrontendip

	info:    Executing command network lb inbound-nat-rule set
	+ Looking up the load balancer "mylb"
	+ Updating load balancer "mylb"
	+ Looking up the load balancer "mylb"
	data:    Id:                        /subscriptions/###############################/resourceGroups/group-1/providers/Microsoft.Network/loadBalancers/mylb/inboundNatRules/myinboundnat
	data:    Name:                      myinboundnat
	data:    Type:                      Microsoft.Network/loadBalancers/inboundNatRules
	data:    Provisioning state:        Succeeded
	data:    Frontend IP Configuration: id=/subscriptions/###############################/resourceGroups/group-1/providers/Microsoft.Network/loadBalancers/mylb/frontendIPConfigurations/myfrontendip
	data:    Backend IP configuration
	data:    Protocol                   Tcp
	data:    Frontend port              81
	data:    Backend port               8080
	data:    Enable floating IP         false
	info:    network lb inbound-nat-rule set command OK

参数选项：

	-h, --help                                     output usage information
	-v, --verbose                                  use verbose output
	--json                                         use json output
	-g, --resource-group <resource-group>          the name of the resource group
	-l, --lb-name <lb-name>                        the name of the load balancer
	-n, --name <name>                              the name of the inbound NAT rule
	-p, --protocol <protocol>                      the rule protocol [tcp,udp]
	-f, --frontend-port <frontend-port>            the frontend port [0-65535]
	-b, --backend-port <backend-port>              the backend port [0-65535]
	-e, --enable-floating-ip <enable-floating-ip>  enable floating point ip [true,false]
	-i, --frontend-ip <frontend-ip>                the name of the frontend ip configuration
	-m, --vm-id [vm-id]                            the VM id.
	e.g. /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines/<vm-name>
	-a, --vm-name <vm-name>                        the VM name.
	This virtual machine must exist in the same resource group as the lb.
	Please use vm-id if that is not the case
	-s, --subscription <subscription>              the subscription identifier
<BR>

	network lb inbound-nat-rule list [options] <resource-group> <lb-name>

列出负载平衡器的所有入站 NAT 规则。

	azure network lb inbound-nat-rule list -g myresourcegroup -l mylb

	info:    Executing command network lb inbound-nat-rule list
	+ Looking up the load balancer "mylb"
	data:    Name          Provisioning state  Protocol  Frontend port  Backend port  Enable floating IP  Idle timeout in minutes  Backend IP configuration
	data:    ------------  ------------------  --------  -------------  ------------  ------------------  -----------------------  ---
	---------------------
	data:    myinboundnat  Succeeded           Tcp       81             8080          false               4

	info:    network lb inbound-nat-rule list command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-s, --subscription <subscription>      the subscription identifier
<BR>

	network lb inbound-nat-rule delete [options] <resource-group> <lb-name> <name>

删除特定资源组中负载平衡器的 NAT 规则。

	azure network lb inbound-nat-rule delete -g myresourcegroup -l mylb -n myinboundnat

	info:    Executing command network lb inbound-nat-rule delete
	+ Looking up the load balancer "mylb"
	Delete inbound NAT rule "myinboundnat?" [y/n] y
	+ Updating load balancer "mylb"
	info:    network lb inbound-nat-rule delete command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-l, --lb-name <lb-name>                the name of the load balancer
	-n, --name <name>                      the name of the inbound NAT rule
	-q, --quiet                            quiet mode, do not ask for delete confirmation
	-s, --subscription <subscription>      the subscription identifier

**用于管理公共 IP 地址的命令**

	network public-ip create [options] <resource-group> <name> <location>

创建公共 IP 资源。你将要创建公共 IP 资源并将其关联到一个域名。

	azure network public-ip create -g myresourcegroup -n mytestpublicip1 -l chinaeast -d azureclitest -a "Dynamic"
	info:    Executing command network public-ip create
	+ Looking up the public ip "mytestpublicip1"
	+ Creating public ip address "mytestpublicip1"
	+ Looking up the public ip "mytestpublicip1"
	data:    Id:                   /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/publicIPAddresses/mytestpublicip1
	data:    Name:                 mytestpublicip1
	data:    Type:                 Microsoft.Network/publicIPAddresses
	data:    Location:             chinaeast
	data:    Provisioning state:   Succeeded
	data:    Allocation method:    Dynamic
	data:    Idle timeout:         4
	data:    Domain name label:    azureclitest
	data:    FQDN:                 azureclitest.chinaeast.chinacloudapp.cn
	info:    network public-ip create command OK


参数选项：

	-h, --help                                   output usage information
	-v, --verbose                                use verbose output
	--json                                       use json output
	-g, --resource-group <resource-group>        the name of the resource group
	-n, --name <name>                            the name of the public ip
	-l, --location <location>                    the location
	-d, --domain-name-label <domain-name-label>  the domain name label.
	This set DNS to <domain-name-label>.<location>.chinacloudapp.cn
	-a, --allocation-method <allocation-method>  the allocation method [Static][Dynamic]
	-i, --idletimeout <idletimeout>              the idle timeout in minutes
	-f, --reverse-fqdn <reverse-fqdn>            the reverse fqdn
	-t, --tags <tags>                            the list of tags.
	Can be multiple. In the format of "name=value".
	Name is required and value is optional.
	For example, -t tag1=value1;tag2
	-s, --subscription <subscription>            the subscription identifier
<br>

	network public-ip set [options] <resource-group> <name>

更新现有的公共 IP 资源的属性。在以下示例中，我们已将公共 IP 地址从动态更改为静态。

	azure network public-ip set -g group-1 -n mytestpublicip1 -d azureclitest -a "Static"
	info:    Executing command network public-ip set
	+ Looking up the public ip "mytestpublicip1"
	+ Updating public ip address "mytestpublicip1"
	+ Looking up the public ip "mytestpublicip1"
	data:    Id:                   /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/publicIPAddresses/mytestpublicip1
	data:    Name:                 mytestpublicip1
	data:    Type:                 Microsoft.Network/publicIPAddresses
	data:    Location:             chinaeast
	data:    Provisioning state:   Succeeded
	data:    Allocation method:    Static
	data:    Idle timeout:         4
	data:    IP Address:           (static IP address)
	data:    Domain name label:    azureclitest
	data:    FQDN:                 azureclitest.chinaeast.chinacloudapp.cn
	info:    network public-ip set command OK

参数选项：

	-h, --help                                   output usage information
	-v, --verbose                                use verbose output
	--json                                       use json output
	-g, --resource-group <resource-group>        the name of the resource group
	-n, --name <name>                            the name of the public ip
	-d, --domain-name-label [domain-name-label]  the domain name label.
	This set DNS to <domain-name-label>.<location>.chinacloudapp.cn
	-a, --allocation-method <allocation-method>  the allocation method [Static][Dynamic]
	-i, --idletimeout <idletimeout>              the idle timeout in minutes
	-f, --reverse-fqdn [reverse-fqdn]            the reverse fqdn
	-t, --tags <tags>                            the list of tags.
	Can be multiple. In the format of "name=value".
	Name is required and value is optional.
	For example, -t tag1=value1;tag2
	--no-tags                                    remove all existing tags
	-s, --subscription <subscription>            the subscription identifier

<br>

	network public-ip list [options] <resource-group> 
	
列出资源组中的所有公共 IP 资源。

	azure network public-ip list -g myresourcegroup

	info:    Executing command network public-ip list
	+ Getting the public ip addresses
	data:    Name             Location  Allocation  IP Address    Idle timeout  DNS Name
	data:    ---------------  --------  ----------  ------------  ------------  -------------------------------------------
	data:    mypubip5         chinanorth    Dynamic                   4             "domain name".chinanorth.chinacloudapp.cn
	data:    myPublicIP       chinaeast    Dynamic                   4             "domain name".chinaeast.chinacloudapp.cn
	data:    mytestpublicip   chinaeast    Dynamic                   4             "domain name".chinaeast.chinacloudapp.cn
	data:    mytestpublicip1  chinaeast   Static (Static IP address) 4             azureclitest.chinaeast.chinacloudapp.cn

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-s, --subscription <subscription>      the subscription identifier
<BR>
	network public-ip show [选项] <resource-group> <name>

显示资源组中公共 IP 资源的公共 IP 属性。

	azure network public-ip show -g myresourcegroup -n mytestpublicip

	info:    Executing command network public-ip show
	+ Looking up the public ip "mytestpublicip1"
	data:    Id:                   /subscriptions/###############################/resourceGroups/myresourcegroup/providers/Microsoft.Network/publicIPAddresses/mytestpublicip
	data:    Name:                 mytestpublicip
	data:    Type:                 Microsoft.Network/publicIPAddresses
	data:    Location:             chinaeast
	data:    Provisioning state:   Succeeded
	data:    Allocation method:    Static
	data:    Idle timeout:         4
	data:    IP Address:           (static IP address)
	data:    Domain name label:    azureclitest
	data:    FQDN:                 azureclitest.chinaeast.chinacloudapp.cn
	info:    network public-ip show command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-n, --name <name>                      the name of the public IP
	-s, --subscription <subscription>      the subscription identifier


	network public-ip delete [options] <resource-group> <name>

删除公共 IP 资源。

	azure network public-ip delete -g group-1 -n mypublicipname
	info:    Executing command network public-ip delete
	+ Looking up the public ip "mypublicipname"
	Delete public ip address "mypublicipname"? [y/n] y
	+ Deleting public ip address "mypublicipname"
	info:    network public-ip delete command OK

参数选项：

	-h, --help                             output usage information
	-v, --verbose                          use verbose output
	--json                                 use json output
	-g, --resource-group <resource-group>  the name of the resource group
	-n, --name <name>                      the name of the public IP
	-q, --quiet                            quiet mode, do not ask for delete confirmation
	-s, --subscription <subscription>      the subscription identifier


**用于管理网络接口的命令**

	network nic create [options] <resource-group> <name> <location>

创建可用于负载平衡器或关联到虚拟机的名为网络接口 (NIC) 的资源。

	azure network nic create -g myresourcegroup -l chinaeast -n testnic1 --subnet-name subnet-1 --subnet-vnet-name myvnet

	info:    Executing command network nic create
	+ Looking up the network interface "testnic1"
	+ Looking up the subnet "subnet-1"
	+ Creating network interface "testnic1"
	+ Looking up the network interface "testnic1"
	data:    Id:                     /subscriptions/c4a17ddf-aa84-491c-b6f9-b90d882299f7/resourceGroups/group-1/providers/Microsoft.Network/networkInterfaces/testnic1
	data:    Name:                   testnic1
	data:    Type:                   Microsoft.Network/networkInterfaces
	data:    Location:               chinaeast
	data:    Provisioning state:     Succeeded
	data:    IP configurations:
	data:       Name:                         NIC-config
	data:       Provisioning state:           Succeeded
	data:       Private IP address:           10.0.0.5
	data:       Private IP Allocation Method: Dynamic
	data:       Subnet:                       /subscriptions/c4a17ddf-aa84-491c-b6f9-b90d882299f7/resourceGroups/group-1/providers/Microsoft.Network/virtualNetworks/myVNET/subnets/Subnet-1

参数选项：

	-h, --help                                                       output usage information
	-v, --verbose                                                    use verbose output
	--json                                                           use json output
	-g, --resource-group <resource-group>                            the name of the resource group
	-n, --name <name>                                                the name of the network interface
	-l, --location <location>                                        the location
	-w, --network-security-group-id <network-security-group-id>      the network security group identifier.
	e.g. /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/networkSecurityGroups/<nsg-name>
	-o, --network-security-group-name <network-security-group-name>  the network security group name.
	This network security group must exist in the same resource group as the nic.
	Please use network-security-group-id if that is not the case.
	-i, --public-ip-id <public-ip-id>                                the public IP identifier.
	e.g. /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/publicIPAddresses/<public-ip-name>
	-p, --public-ip-name <public-ip-name>                            the public IP name.
	This public ip must exist in the same resource group as the nic.
	Please use public-ip-id if that is not the case.
	-a, --private-ip-address <private-ip-address>                    the private IP address
	-u, --subnet-id <subnet-id>                                      the subnet identifier.
	e.g. /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/virtualNetworks/<vnet-name>/subnets/<subnet-name>
	--subnet-name <subnet-name>                                  the subnet name
	-m, --subnet-vnet-name <subnet-vnet-name>                        the vnet name under which subnet-name exists
	-t, --tags <tags>                                                the comma seperated list of tags.
	Can be multiple. In the format of "name=value".
	Name is required and value is optional.
	For example, -t tag1=value1;tag2
	-s, --subscription <subscription>                                the subscription identifier
	data:
	info:    network nic create command OK

<BR>

	network nic set [options] <resource-group> <name>

	network nic list [options] <resource-group>
	network nic show [options] <resource-group> <name>
	network nic delete [options] <resource-group> <name>

**用于管理网络安全组的命令**

	network nsg create [options] <resource-group> <name> <location>
	network nsg set [options] <resource-group> <name>
	network nsg list [options] <resource-group>
	network nsg show [options] <resource-group> <name>
	network nsg delete [options] <resource-group> <name>

**用于管理网络安全组规则的命令**

	network nsg rule create [options] <resource-group> <nsg-name> <name>
	network nsg rule set [options] <resource-group> <nsg-name> <name>
	network nsg rule list [options] <resource-group> <nsg-name>
	network nsg rule show [options] <resource-group> <nsg-name> <name>
	network nsg rule delete [options] <resource-group> <nsg-name> <name>

**用于管理流量管理器配置文件的命令**

	network traffic-manager profile create [options] <resource-group> <name>
	network traffic-manager profile set [options] <resource-group> <name>
	network traffic-manager profile list [options] <resource-group>
	network traffic-manager profile show [options] <resource-group> <name>
	network traffic-manager profile delete [options] <resource-group> <name>
	network traffic-manager profile is-dns-available [options] <resource-group> <relative-dns-name>

**用于管理流量管理器终结点的命令**

	network traffic-manager profile endpoint create [options] <resource-group> <profile-name> <name> <endpoint-location>
	network traffic-manager profile endpoint set [options] <resource-group> <profile-name> <name>
	network traffic-manager profile endpoint delete [options] <resource-group> <profile-name> <name>

**用于管理虚拟网络网关的命令**

	network gateway list [options] <resource-group>

## azure provider：用于管理资源提供程序注册的命令

**列出 ARM 中当前已注册的提供程序**

	provider list [options]

**显示有关请求的提供程序命名空间的详细信息**

	provider show [options] <namespace>

**将提供程序注册到订阅**

	provider register [options] <namespace>

**从订阅中取消注册提供程序**

	provider unregister [options] <namespace>

## azure resource：用于管理资源的命令

**在资源组中创建资源**

	resource create [options] <resource-group> <name> <resource-type> <location> <api-version>

**更新资源组中不带任何模板或参数的资源**

	resource set [options] <resource-group> <name> <resource-type> <properties> <api-version>

**列出资源**

	resource list [options] [resource-group]

**获取资源组或订阅中的一个资源**

	resource show [options] <resource-group> <name> <resource-type> <api-version>

**删除资源组中的资源**

	resource delete [options] <resource-group> <name> <resource-type> <api-version>

## azure role：用于管理 Azure 角色的命令

**获取所有可用的角色定义**

	role list [options]

**获取一个可用的角色定义**

	role show [options] [name]

**用于管理角色分配的命令**

	role assignment create [options] [objectId] [upn] [mail] [spn] [role] [scope] [resource-group] [resource-type] [resource-name]
	role assignment list [options] [objectId] [upn] [mail] [spn] [role] [scope] [resource-group] [resource-type] [resource-name]
	role assignment delete [options] [objectId] [upn] [mail] [spn] [role] [scope] [resource-group] [resource-type] [resource-name]

## azure storage：用于管理存储对象的命令

**用于管理存储帐户的命令**

	storage account list [options]
	storage account show [options] <name>
	storage account create [options] <name>
	storage account set [options] <name>
	storage account delete [options] <name>

**用于管理存储帐户密钥的命令**

	storage account keys list [options] <name>
	storage account keys renew [options] <name>

**用于显示存储连接字符串的命令**

	storage account connectionstring show [options] <name>

**用于管理存储容器的命令**

	storage container list [options] [prefix]
	storage container show [options] [container]
	storage container create [options] [container]
	storage container delete [options] [container]
	storage container set [options] [container]

**用于管理存储容器的共享访问签名的命令**

	storage container sas create [options] [container] [permissions] [expiry]

**用于管理存储容器的存储访问策略的命令**

	storage container policy create [options] [container] [name]
	storage container policy show [options] [container] [name]
	storage container policy list [options] [container]
	storage container policy set [options] [container] [name]
	storage container policy delete [options] [container] [name]

**用于管理存储 Blob 的命令**

	storage blob list [options] [container] [prefix]
	storage blob show [options] [container] [blob]
	storage blob delete [options] [container] [blob]
	storage blob upload [options] [file] [container] [blob]
	storage blob download [options] [container] [blob] [destination]

**用于管理 Blob 复制操作的命令**

	storage blob copy start [options] [sourceUri] [destContainer]
	storage blob copy show [options] [container] [blob]
	storage blob copy stop [options] [container] [blob] [copyid]

**用于管理存储 Blob 的共享访问签名的命令**

	storage blob sas create [options] [container] [blob] [permissions] [expiry]

**用于管理存储文件共享的命令**

	storage share create [options] [share]
	storage share show [options] [share]
	storage share delete [options] [share]
	storage share list [options] [prefix]

**用于管理存储文件的命令**

	storage file list [options] [share] [path]
	storage file delete [options] [share] [path]
	storage file upload [options] [source] [share] [path]
	storage file download [options] [share] [path] [destination]

**用于管理存储文件目录的命令**

	storage directory create [options] [share] [path]
	storage directory delete [options] [share] [path]

**用于管理存储队列的命令**

	storage queue create [options] [queue]
	storage queue list [options] [prefix]
	storage queue show [options] [queue]
	storage queue delete [options] [queue]

**用于管理存储队列的共享访问签名的命令**

	storage queue sas create [options] [queue] [permissions] [expiry]

**用于管理存储队列的存储访问策略的命令**

	storage queue policy create [options] [queue] [name]
	storage queue policy show [options] [queue] [name]
	storage queue policy list [options] [queue]
	storage queue policy set [options] [queue] [name]
	storage queue policy delete [options] [queue] [name]

**用于管理存储日志记录属性的命令**

	storage logging show [options]
	storage logging set [options]

**用于管理存储度量值属性的命令**

	storage metrics show [options]
	storage metrics set [options]

**用于管理存储表的命令**

	storage table create [options] [table]
	storage table list [options] [prefix]
	storage table show [options] [table]
	storage table delete [options] [table]

**用于管理存储表的共享访问签名的命令**

	storage table sas create [options] [table] [permissions] [expiry]

**用于管理存储表的存储访问策略的命令**

	storage table policy create [options] [table] [name]
	storage table policy show [options] [table] [name]
	storage table policy list [options] [table]
	storage table policy set [options] [table] [name]
	storage table policy delete [options] [table] [name]

## azure tag：用于管理资源管理器标记的命令

**添加标记**

	tag create [options] <name> <value>

**删除整个标记或标记值**

	tag delete [options] <name> <value>

**列出标记信息**

	tag list [options]

**获取标记**

	tag show [options] [name]

## azure vm：用于管理 Azure 虚拟机的命令

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

**创建 VM**

	vm create [options] <resource-group> <name> <location> <os-type>

**使用默认资源创建 VM**

    vm quick-create [options] <resource-group> <name> <location> <os-type> <image-urn> <admin-username> <admin-password>
    
>[AZURE.TIP]从 CLI 版本 0.10 开始，你可以为某些常用的应用商店映像提供“UbuntuLTS”或“Win2012R2Datacenter”之类的简短别名作为 `image-urn`。针对选项运行 `azure help vm quick-create`。另外，从版本 0.10 开始，`azure vm quick-create` 默认使用高级存储，前提是所选区域提供该存储。

**列出帐户中的虚拟机**

	vm list [options]

**获取资源组中的一个虚拟机**

	vm show [options] <resource-group> <name>

**删除资源组中的一个虚拟机**

	vm delete [options] <resource-group> <name>

**关闭资源组中的一个虚拟机**

	vm stop [options] <resource-group> <name>

**重新启动资源组中的一个虚拟机**

	vm restart [options] <resource-group> <name>

**启动资源组中的一个虚拟机**

	vm start [options] <resource-group> <name>

**关闭资源组中的一个虚拟机并释放计算资源**

	vm deallocate [options] <resource-group> <name>

**列出可用的虚拟机大小**

	vm sizes [options]

**捕获用作 OS 映像或 VM 映像的 VM**

	vm capture [options] <resource-group> <name> <vhd-name-prefix>

**将 VM 的状态设置为通用化**

	vm generalize [options] <resource-group> <name>

**获取 VM 的实例视图**

	vm get-instance-view [options] <resource-group> <name>

**可让你重置虚拟机上的远程桌面访问或 SSH 设置，以及重置具有管理员或 sudo 权限的帐户的密码**

	vm reset-access [options] <resource-group> <name>

**使用新数据更新 VM**

	vm set [options] <resource-group> <name>

**用于管理虚拟机数据磁盘的命令**

	vm disk attach-new [options] <resource-group> <vm-name> <size-in-gb> [vhd-name]
	vm disk detach [options] <resource-group> <vm-name> <lun>
	vm disk attach [options] <resource-group> <vm-name> [vhd-url]

**用于管理 VM 资源扩展的命令**

	vm extension set [options] <resource-group> <vm-name> <name> <publisher-name> <version>
	vm extension get [options] <resource-group> <vm-name>

**用于管理 Docker 虚拟机的命令**

	vm docker create [options] <resource-group> <name> <location> <os-type>

**用于管理 VM 映像的命令**

	vm image list-publishers [options] <location>
	vm image list-offers [options] <location> <publisher>
	vm image list-skus [options] <location> <publisher> <offer>
	vm image list [options] <location> <publisher> [offer] [sku]

<!---HONumber=Mooncake_0620_2016-->