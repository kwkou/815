<!-- ARM: tested -->

<properties
   pageTitle="使用 Azure CLI 从头开始创建 Linux VM | Azure"
   description="使用 Azure CLI 从头开始创建 Linux VM、存储、虚拟网络和子网、NIC、公共 IP 和网络安全组。"
   services="virtual-machines-linux"
   documentationCenter="virtual-machines"
   authors="vlivech"
   manager="squillace"
   editor=""
   tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/04/2016"
	wacn.date="06/06/2016"/>

# 使用 Azure CLI 从头开始创建 Linux VM

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

若要创建 Linux VM，你需要安装 Resource Manager 模式 (`azure config mode arm`) 的 [Azure CLI](/documentation/articles/xplat-cli-install) 以及 JSON 分析工具，在本文档中，我们将使用 [jq](https://stedolan.github.io/jq/)。

## 快速命令

	# Create the Resource Group
	chrisL@fedora$ azure group create TestRG chinaeast
	
	# Create the Storage Account
	chrisL@fedora$ azure storage account create \  
	--location chinaeast \
	--resource-group TestRG \
	--type GRS \
	computeteststore
	
	# Verify the RG using the JSON parser
	chrisL@fedora$ azure group show testrg --json | jq '.'
	
	# Create the Virtual Network
	chrisL@fedora$ azure network vnet create -g TestRG -n TestVNet -a 192.168.0.0/16 -l chinaeast
	
	# Verify the RG
	chrisL@fedora$ azure group show testrg --json | jq '.'
	
	# Create the Subnet
	chrisL@fedora$ azure network vnet subnet create -g TestRG -e TestVNet -n FrontEnd -a 192.168.1.0/24
	
	# Verify the VNet and Subnet
	chrisL@fedora$ azure network vnet show testrg testvnet --json | jq '.'
	
	# Create the NIC
	chrisL@fedora$ azure network nic create -g TestRG -n TestNIC -l chinaeast -a 192.168.1.101 -m TestVNet -k FrontEnd
	
	# Verify the NIC
	chrisL@fedora$ azure network nic show testrg testnic --json | jq '.'
	
	# Create the NSG
	chrisL@fedora$ azure network nsg create testrg testnsg chinaeast
	
	# Add an inbound rule for the NSG
	chrisL@fedora$ azure network nsg rule create --protocol tcp --direction inbound --priority 1000  --destination-port-range 22 --access allow testrg testnsg testnsgrule
	
	# Creat the Public facing NIC
	chrisL@fedora$ azure network public-ip create -d testsubdomain testrg testpip chinaeast
	
	# Verify the NIC
	chrisL@fedora$ azure network public-ip show testrg testpip --json | jq '.'
	
	# Associate the Public IP to the NIC
	chrisL@fedora$ azure network nic set --public-ip-name testpip testrg testnic
	
	# Bind the NSG to the NIC
	chrisL@fedora$ azure network nic set --network-security-group-name testnsg testrg testnic
	
	# Create the Linux VM
	chrisL@fedora$ azure vm create \            
	    --resource-group testrg \
	    --name testvm \
	    --location chinaeast \
	    --os-type linux \
	    --nic-name testnic \
	    --vnet-name testvnet \
	    --vnet-subnet-name FrontEnd \
	    --storage-account-name computeteststore \
	    --image-urn canonical:UbuntuServer:14.04.3-LTS:latest \
	    --ssh-publickey-file ~/.ssh/id_rsa.pub \
	    --admin-username ops
	
	# Verify everything built
	chrisL@fedora$ azure vm show testrg testvm

## 详细演练

### 介绍

本文使用 VNetwork 子网中的一个 Linux VM 来构建部署。它以强制性的逐条命令方式逐步完成整个基本部署，直到创建一个可通过 Internet 从任何位置连接的有效且安全的 Linux VM。

在此过程中，你将了解 Resource Manager 部署模型提供给你的依赖性层次结构及其提供的功能。了解系统的构建方法后，你可以使用更直接的 Azure CLI 命令，更快地重新构建系统（请参阅[此文](/documentation/articles/virtual-machines-linux-quick-create-cli)，以了解如何使用 `azure vm quick-create` 命令处理大致相同的部署），或者继续掌握如何设计和自动化整个网络与应用程序部署，并使用 [Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates)进行更新。了解部署的部件如何彼此配合运行后，你可以更轻松地创建模板来将它们自动化。

让我们构建一个简单的网络，其中包含一个可用于部署和简单计算的 VM；在构建过程中，我们将解释具体的操作。然后你可以继续构建更复杂的网络和部署。

### 创建资源组并选择部署位置

Azure 资源组是逻辑部署实体，其中包含用于启用资源部署逻辑管理的配置及其他元数据。

	chrisL@fedora$ azure group create TestRG chinaeast                        
	info:    Executing command group create
	+ Getting resource group TestRG
	+ Creating resource group TestRG
	info:    Created resource group TestRG
	data:    Id:                  /subscriptions/<yoursub>/resourceGroups/TestRG
	data:    Name:                TestRG
	data:    Location:            chinaeast
	data:    Provisioning State:  Succeeded
	data:    Tags: null
	data:
	info:    group create command OK

### 创建存储帐户

你需要对 VM 磁盘、任何想要添加的额外数据磁盘及其他方案使用存储帐户。简而言之，你始终要在创建资源组之后紧接着创建存储帐户。

我们在此处使用 `azure storage account create` 命令，并传递帐户的位置、将要控制该帐户的资源组，以及所需的存储支持类型。

	chrisL@fedora$ azure storage account create \  
	--location chinaeast \
	--resource-group TestRG \
	--type GRS \
	computeteststore
	info:    Executing command storage account create
	+ Creating storage account
	info:    storage account create command OK
	rasquill•~/workspace/keygen» azure group show testrg
	info:    Executing command group show
	+ Listing resource groups
	+ Listing resources for the group
	data:    Id:                  /subscriptions/<guid>/resourceGroups/TestRG
	data:    Name:                TestRG
	data:    Location:            chinaeast
	data:    Provisioning State:  Succeeded
	data:    Tags: null
	data:    Resources:
	data:
	data:      Id      : /subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Storage/storageAccounts/computeteststore
	data:      Name    : computeteststore
	data:      Type    : storageAccounts
	data:      Location: chinaeast
	data:      Tags    :
	data:
	data:    Permissions:
	data:      Actions: *
	data:      NotActions:
	data:
	info:    group show command OK

让我们结合 `--json` Azure CLI 选项使用 [jq](https://stedolan.github.io/jq/) 工具（可以使用 **jsawk** 或偏好的语言库来分析 JSON）以及 `azure group show` 命令来检查资源组。

	chrisL@fedora$ azure group show testrg --json | jq '.'                                                                                        
	{
	  "tags": {},
	  "id": "/subscriptions/<guid>/resourceGroups/TestRG",
	  "name": "TestRG",
	  "provisioningState": "Succeeded",
	  "location": "chinaeast",
	  "properties": {
	    "provisioningState": "Succeeded"
	  },
	  "resources": [
	    {
	      "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Storage/storageAccounts/computeteststore",
	      "name": "computeteststore",
	      "type": "storageAccounts",
	      "location": "chinaeast",
	      "tags": null
	    }
	  ],
	  "permissions": [
	    {
	      "actions": [
	        "*"
	      ],
	      "notActions": []
	    }
	  ]
	}

若要调查使用 CLI 的存储帐户，首先需要使用以下命令的变体，将本文中的存储帐户名称替换为你自己的名称，以设置帐户名称和密钥。

	AZURE_STORAGE_CONNECTION_STRING="$(azure storage account connectionstring show computeteststore --resource-group testrg --json | jq -r '.string')"

然后就能轻松地查看存储信息：

	chrisL@fedora$ azure storage container list
	info:    Executing command storage container list
	+ Getting storage containers
	data:    Name  Public-Access  Last-Modified
	data:    ----  -------------  -----------------------------
	data:    vhds  Off            Sun, 27 Sep 2015 19:03:54 GMT
	info:    storage container list command OK

### 创建虚拟网络和子网

你需要创建可在其中安装 VM 的 Azure 虚拟网络和子网。

	chrisL@fedora$ azure network vnet create -g TestRG -n TestVNet -a 192.168.0.0/16 -l chinaeast
	info:    Executing command network vnet create
	+ Looking up virtual network "TestVNet"
	+ Creating virtual network "TestVNet"
	+ Loading virtual network state
	data:    Id                              : /subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
	data:    Name                            : TestVNet
	data:    Type                            : Microsoft.Network/virtualNetworks
	data:    Location                        : chinaeast
	data:    ProvisioningState               : Succeeded
	data:    Address prefixes:
	data:      192.168.0.0/16
	info:    network vnet create command OK

同样，让我们了解如何使用 `azure group show` 的 --json 选项以及 **jq** 构建资源。现在，我们有了一个 `storageAccounts` 资源和一个 `virtualNetworks` 资源。

	chrisL@fedora$ azure group show testrg --json | jq '.'
	{
	  "tags": {},
	  "id": "/subscriptions/<guid>/resourceGroups/TestRG",
	  "name": "TestRG",
	  "provisioningState": "Succeeded",
	  "location": "chinaeast",
	  "properties": {
	    "provisioningState": "Succeeded"
	  },
	  "resources": [
	    {
	      "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet",
	      "name": "TestVNet",
	      "type": "virtualNetworks",
	      "location": "chinaeast",
	      "tags": null
	    },
	    {
	      "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Storage/storageAccounts/computeteststore",
	      "name": "computeteststore",
	      "type": "storageAccounts",
	      "location": "chinaeast",
	      "tags": null
	    }
	  ],
	  "permissions": [
	    {
	      "actions": [
	        "*"
	      ],
	      "notActions": []
	    }
	  ]
	}

现在，让我们在要部署 VM 的 `TestVnet` 虚拟网络中创建子网。我们使用 `azure network vnet subnet create` 命令以及已创建的资源：`TestRG` 资源组、`TestVNet` 虚拟网络，并添加子网名称 `FrontEnd` 和子网地址前缀 `192.168.1.0/24`，如下所示。

	chrisL@fedora$ azure network vnet subnet create -g TestRG -e TestVNet -n FrontEnd -a 192.168.1.0/24
	info:    Executing command network vnet subnet create
	+ Looking up the subnet "FrontEnd"
	+ Creating subnet "FrontEnd"
	+ Looking up the subnet "FrontEnd"
	data:    Id                              : /subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
	data:    Type                            : Microsoft.Network/virtualNetworks/subnets
	data:    ProvisioningState               : Succeeded
	data:    Name                            : FrontEnd
	data:    Address prefix                  : 192.168.1.0/24
	data:
	info:    network vnet subnet create command OK

由于子网以逻辑方式出现在虚拟网络中，我们使用稍微不同的命令来查找子网信息 -- `azure network vnet show`，但仍需使用 **jq** 检查 JSON 输出。

	chrisL@fedora$ azure network vnet show testrg testvnet --json | jq '.'
	{
	  "subnets": [
	    {
	      "ipConfigurations": [],
	      "addressPrefix": "192.168.1.0/24",
	      "provisioningState": "Succeeded",
	      "name": "FrontEnd",
	      "etag": "W/"974f3e2c-028e-4b35-832b-a4b16ad25eb6"",
	      "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
	    }
	  ],
	  "tags": {},
	  "addressSpace": {
	    "addressPrefixes": [
	      "192.168.0.0/16"
	    ]
	  },
	  "dhcpOptions": {
	    "dnsServers": []
	  },
	  "provisioningState": "Succeeded",
	  "etag": "W/"974f3e2c-028e-4b35-832b-a4b16ad25eb6"",
	  "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet",
	  "name": "TestVNet",
	  "location": "chinaeast"
	}

### 创建用于 Linux VM 的 NIC

由于你可以将规则应用到 NIC 的使用上，并有多个规则，因此即使对于 NIC，也能以编程方式使用。

	chrisL@fedora$ azure network nic create -g TestRG -n TestNIC -l chinaeast -a 192.168.1.101 -m TestVNet -k FrontEnd
	info:    Executing command network nic create
	+ Looking up the network interface "TestNIC"
	+ Looking up the subnet "FrontEnd"
	+ Creating network interface "TestNIC"
	+ Looking up the network interface "TestNIC"
	data:    Id                              : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC
	data:    Name                            : TestNIC
	data:    Type                            : Microsoft.Network/networkInterfaces
	data:    Location                        : chinaeast
	data:    Provisioning state              : Succeeded
	data:    Enable IP forwarding            : false
	data:    IP configurations:
	data:      Name                          : NIC-config
	data:      Provisioning state            : Succeeded
	data:      Private IP address            : 192.168.1.101
	data:      Private IP Allocation Method  : Static
	data:      Subnet                        : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
	data:
	info:    network nic create command OK

由于 NIC 资源与 VM 和网络安全组相关联，因此在检查 `TestRG` 资源组时可以将其视为顶级资源：

	chrisL@fedora$ azure group show testrg --json | jq '.'
	{
	"tags": {},
	"id": "/subscriptions/guid/resourceGroups/TestRG",
	"name": "TestRG",
	"provisioningState": "Succeeded",
	"location": "chinaeast",
	"properties": {
	    "provisioningState": "Succeeded"
	},
	"resources": [
	    {
	    "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC",
	    "name": "TestNIC",
	    "type": "networkInterfaces",
	    "location": "chinaeast",
	    "tags": null
	    },
	    {
	    "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet",
	    "name": "TestVNet",
	    "type": "virtualNetworks",
	    "location": "chinaeast",
	    "tags": null
	    },
	    {
	    "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Storage/storageAccounts/computeteststore",
	    "name": "computeteststore",
	    "type": "storageAccounts",
	    "location": "chinaeast",
	    "tags": null
	    }
	],
	"permissions": [
	    {
	    "actions": [
	        "*"
	    ],
	    "notActions": []
	    }
	]
	}

可以通过直接检查资源或使用 `azure network nic show` 命令来查看详细信息。

	chrisL@fedora$ azure network nic show testrg testnic --json | jq '.'
	{
	"ipConfigurations": [
	    {
	    "loadBalancerBackendAddressPools": [],
	    "loadBalancerInboundNatRules": [],
	    "privateIpAddress": "192.168.1.101",
	    "privateIpAllocationMethod": "Static",
	    "subnet": {
	        "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
	    },
	    "provisioningState": "Succeeded",
	    "name": "NIC-config",
	    "etag": "W/"4d29b1ca-0207-458c-b258-f298e6fc450f"",
	    "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/NIC-config"
	    }
	],
	"tags": {},
	"dnsSettings": {
	    "appliedDnsServers": [],
	    "dnsServers": []
	},
	"enableIPForwarding": false,
	"provisioningState": "Succeeded",
	"etag": "W/"4d29b1ca-0207-458c-b258-f298e6fc450f"",
	"id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC",
	"name": "TestNIC",
	"location": "chinaeast"
	}

### 创建网络安全组和规则

现在，我们将创建网络安全组 (NSG) 和用于控制 NIC 访问权限的入站规则。

	chrisL@fedora$ azure network nsg create testrg testnsg chinaeast

让我们添加 NSG 的入站规则以允许端口 22 上的入站连接（以支持 SSH）：

	chrisL@fedora$ azure network nsg rule create --protocol tcp --direction inbound --priority 1000  --destination-port-range 22 --access allow testrg testnsg testnsgrule

> [AZURE.NOTE] 入站规则是入站网络连接的筛选器。在本示例中，我们将 NSG 绑定到 VM 虚拟网络适配卡 (NIC)，这意味着任何对端口 22 的请求都将在 VM 上传递到 NIC。由于这是关于网络连接（而不是经典部署中的终结点）打开端口的规则，因此必须将 `--source-port-range` 保持设置为 '\*'（默认值）才能接受来自**任何**请求端口的入站请求（这些请求通常是动态的）。

### 创建公共 IP 地址 (PIP)

现在，让我们创建可让你从 Internet 使用 `azure network public-ip create` 命令连接到 VM 的公共 IP 地址 (PIP)。由于默认值是动态地址，因此我们使用 `-d testsubdomain` 选项在 **chinacloudapp.cn** 域中创建一个命名的 DNS 条目。

	chrisL@fedora$ azure network public-ip create -d testsubdomain testrg testpip chinaeast
	info:    Executing command network public-ip create
	+ Looking up the public ip "testpip"
	+ Creating public ip address "testpip"
	+ Looking up the public ip "testpip"
	data:    Id                              : /subscriptions/guid/resourceGroups/testrg/providers/Microsoft.Network/publicIPAddresses/testpip
	data:    Name                            : testpip
	data:    Type                            : Microsoft.Network/publicIPAddresses
	data:    Location                        : chinaeast
	data:    Provisioning state              : Succeeded
	data:    Allocation method               : Dynamic
	data:    Idle timeout                    : 4
	data:    Domain name label               : testsubdomain
	data:    FQDN                            : testsubdomain.chinaeast.chinacloudapp.cn
	info:    network public-ip create command OK

这也是顶级资源，因此你可以使用 `azure group show` 查看它。

	chrisL@fedora$ azure group show testrg --json | jq '.'
	{
	"tags": {},
	"id": "/subscriptions/guid/resourceGroups/TestRG",
	"name": "TestRG",
	"provisioningState": "Succeeded",
	"location": "chinaeast",
	"properties": {
	    "provisioningState": "Succeeded"
	},
	"resources": [
	    {
	    "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC",
	    "name": "TestNIC",
	    "type": "networkInterfaces",
	    "location": "chinaeast",
	    "tags": null
	    },
	    {
	    "id": "/subscriptions/guid/resourceGroups/testrg/providers/Microsoft.Network/publicIPAddresses/testpip",
	    "name": "testpip",
	    "type": "publicIPAddresses",
	    "location": "chinaeast",
	    "tags": null
	    },
	    {
	    "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet",
	    "name": "TestVNet",
	    "type": "virtualNetworks",
	    "location": "chinaeast",
	    "tags": null
	    },
	    {
	    "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Storage/storageAccounts/computeteststore",
	    "name": "computeteststore",
	    "type": "storageAccounts",
	    "location": "chinaeast",
	    "tags": null
	    }
	],
	"permissions": [
	    {
	    "actions": [
	        "*"
	    ],
	    "notActions": []
	    }
	]
	}

如往常一样，你可以调查更多资源的详细信息，包括使用更完整的 `azure network public-ip show` 命令查看子域的完全限定域名 (FQDN)。请注意，公共 IP 地址资源以逻辑方式分配，但尚未分配有特定地址。为此，你需要一个 VM，而我们尚未创建该 VM。

	azure network public-ip show testrg testpip --json | jq '.'
	{
	"tags": {},
	"publicIpAllocationMethod": "Dynamic",
	"dnsSettings": {
	    "domainNameLabel": "testsubdomain",
	    "fqdn": "testsubdomain.chinaeast.chinacloudapp.cn"
	},
	"idleTimeoutInMinutes": 4,
	"provisioningState": "Succeeded",
	"etag": "W/"c63154b3-1130-49b9-a887-877d74d5ebc5"",
	"id": "/subscriptions/guid/resourceGroups/testrg/providers/Microsoft.Network/publicIPAddresses/testpip",
	"name": "testpip",
	"location": "chinaeast"
	}

### 将公共 IP 和网络安全组关联到 NIC

	chrisL@fedora$ azure network nic set --public-ip-name testpip testrg testnic

将 NSG 绑定到 NIC：

	chrisL@fedora$ azure network nic set --network-security-group-name testnsg testrg testnic

### 创建 Linux VM

你已创建存储和网络资源以支持可访问 Internet 的 VM。现在，让我们创建 VM，并使用不带密码的 ssh 密钥来保护其安全。在此情况下，我们需要基于最新的 LTS 创建 Ubuntu VM。我们将根据[查找 Azure VM 映像](/documentation/articles/virtual-machines-linux-cli-ps-findimage)中所述，使用 `azure vm image list` 来查找该映像信息。我们前面使用了命令 `azure vm image list chinaeast canonical | grep LTS` 来选择映像，而现在，我们将使用 `canonical:UbuntuServer:14.04.3-LTS:14.04.201509080`，但为最后一个字段传递 `latest`，以便将来可随时获取最新的内部版本（我们使用的字符串将是 `canonical:UbuntuServer:14.04.3-LTS:latest`）。

> [AZURE.NOTE] 已使用 **ssh-keygen -t rsa -b 2048** 在 Linux 或 Mac 上创建 ssh rsa 公钥和私钥对的任何人都熟悉下一个步骤。如果 `~/.ssh` 目录中没有任何证书密钥对，你可以创建证书密钥对：
> <p>1. 使用 `azure vm create --generate-ssh-keys` 选项自动创建。
> <p>2. [根据说明手动创建](/documentation/articles/virtual-machines-linux-ssh-from-linux)。
> <p>或者，可以使用 `azure vm create --admin-username --admin-password` 选项，以使用通常较不安全的用户名和密码方法在创建 VM 之后对 ssh 连接进行身份验证。

我们通过使用 `azure vm create` 命令并结合所有资源和信息来创建 VM。

	chrisL@fedora$ azure vm create \            
	--resource-group testrg \
	--name testvm \
	--location chinaeast \
	--os-type linux \
	--nic-name testnic \
	--vnet-name testvnet \
	--vnet-subnet-name FrontEnd \
	--storage-account-name computeteststore \
	--image-urn canonical:UbuntuServer:14.04.3-LTS:latest \
	--ssh-publickey-file ~/.ssh/id_rsa.pub \
	--admin-username ops
	info:    Executing command vm create
	+ Looking up the VM "testvm"
	info:    Verifying the public key SSH file: /Users/user/.ssh/id_rsa.pub
	info:    Using the VM Size "Standard_A1"
	info:    The [OS, Data] Disk or image configuration requires storage account
	+ Looking up the storage account computeteststore
	+ Looking up the NIC "testnic"
	info:    Found an existing NIC "testnic"
	info:    Found an IP configuration with virtual network subnet id "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd" in the NIC "testnic"
	info:    This NIC IP configuration has a public ip already configured "/subscriptions/guid/resourcegroups/testrg/providers/microsoft.network/publicipaddresses/testpip", any public ip parameters if provided, will be ignored.
	+ Creating VM "testvm"
	info:    vm create command OK

现在，你可以使用默认的 ssh 密钥立即连接到 VM。

	chrisL@fedora$ ssh ops@testsubdomain.chinaeast.chinacloudapp.cn           
	The authenticity of host 'testsubdomain.chinaeast.chinacloudapp.cn (XX.XXX.XX.XXX)' can't be established.
	RSA key fingerprint is b6:a4:7g:4b:cb:cd:76:87:63:2d:84:83:ac:12:2d:cd.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'testsubdomain.chinaeast.chinacloudapp.cn,XX.XXX.XX.XXX' (RSA) to the list of known hosts.
	Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.19.0-28-generic x86_64)
	
	* Documentation:  https://help.ubuntu.com/
	
	System information as of Mon Sep 28 18:45:02 UTC 2015
	
	System load: 0.64              Memory usage: 5%   Processes:       81
	Usage of /:  45.3% of 1.94GB   Swap usage:   0%   Users logged in: 0
	
	Graph this data and manage this system at:
	    https://landscape.canonical.com/
	
	Get cloud support with Ubuntu Advantage Cloud Guest:
	    http://www.ubuntu.com/business/services/cloud
	
	0 packages can be updated.
	0 updates are security updates.
	
	
	
	The programs included with the Ubuntu system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.
	
	Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
	applicable law.
	
	ops@testvm:~$

并且可以使用 `azure vm show testrg testvm` 命令来检查创建的内容。此时，你已在 Azure 中运行了一个 Ubuntu VM，你只能使用创建的 ssh 密钥对登录；禁止使用密码。

	chrisL@fedora$ azure vm show testrg testvm
	info:    Executing command vm show
	+ Looking up the VM "testvm"
	+ Looking up the NIC "testnic"
	+ Looking up the public ip "testpip"
	data:    Id                              :/subscriptions/guid/resourceGroups/testrg/providers/Microsoft.Compute/virtualMachines/testvm
	data:    ProvisioningState               :Succeeded
	data:    Name                            :testvm
	data:    Location                        :chinaeast
	data:    FQDN                            :testsubdomain.chinaeast.chinacloudapp.cn
	data:    Type                            :Microsoft.Compute/virtualMachines
	data:
	data:    Hardware Profile:
	data:      Size                          :Standard_A1
	data:
	data:    Storage Profile:
	data:      Image reference:
	data:        Publisher                   :canonical
	data:        Offer                       :UbuntuServer
	data:        Sku                         :14.04.3-LTS
	data:        Version                     :latest
	data:
	data:      OS Disk:
	data:        OSType                      :Linux
	data:        Name                        :cli4eecdddc349d6015-os-1443465824206
	data:        Caching                     :ReadWrite
	data:        CreateOption                :FromImage
	data:        Vhd:
	data:          Uri                       :https://computeteststore.blob.core.chinacloudapi.cn/vhds/cli4eecdddc349d6015-os-1443465824206.vhd
	data:
	data:    OS Profile:
	data:      Computer Name                 :testvm
	data:      User Name                     :ops
	data:      Linux Configuration:
	data:        Disable Password Auth       :true
	data:        SSH Public Keys:
	data:          Public Key #1:
	data:            Path                    :/home/ops/.ssh/authorized_keys
	data:            Key                     :MIIBrTCCAZigAwIBAgIBATALBgkqhkiG9w0BAQUwADAiGA8yMDE1MDkyODE4MzM0
	<snip>
	data:
	data:    Network Profile:
	data:      Network Interfaces:
	data:        Network Interface #1:
	data:          Id                        :/subscriptions/guid/resourceGroups/testrg/providers/Microsoft.Network/networkInterfaces/testnic
	data:          Primary                   :true
	data:          MAC Address               :00-0D-3A-21-8E-AE
	data:          Provisioning State        :Succeeded
	data:          Name                      :testnic
	data:          Location                  :chinaeast
	data:            Private IP alloc-method :Dynamic
	data:            Private IP address      :192.168.1.101
	data:            Public IP address       :40.115.48.189
	data:            FQDN                    :testsubdomain.chinaeast.chinacloudapp.cn
	data:
	data:    Diagnostics Instance View:
	info:    vm show command OK

### 后续步骤

现在，你已准备好开始使用多个网络组件和 VM。

<!---HONumber=Mooncake_0425_2016-->