
<properties
   pageTitle="使用 Azure CLI 创建完整的 Linux 环境 | Azure"
   description="使用 Azure CLI 从头开始创建存储、Linux VM、虚拟网络和子网、负载均衡器、NIC、公共 IP 和网络安全组。"
   services="virtual-machines-linux"
   documentationCenter="virtual-machines"
   authors="iainfoulds"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>  


<tags
   ms.service="virtual-machines-linux"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-linux"
   ms.workload="infrastructure"
   ms.date="08/23/2016"
   wacn.date="12/12/2016"
   ms.author="iainfou"/>

# 使用 Azure CLI 创建完整的 Linux 环境

在本文中，我们将构建一个简单网络，其中包含一个负载均衡器，以及一对可用于开发和简单计算的 VM。将以逐条命令的方式完成整个过程，直到创建两个可以从 Internet 上的任何位置连接的有效且安全的 Linux VM。然后，便可以继续构建更复杂的网络和环境。

在此过程中，将了解 Resource Manager 部署模型提供的依赖性层次结构及其提供的功能。明白系统是如何构建的以后，即可使用 [Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)更快速地重新构建系统。此外，了解环境的各个部分如何彼此配合运行后，可以更轻松创建模板以实现自动化。

该环境包含：

- 两个位于可用性集中的 VM。
- 端口 80 上有一个带负载均衡规则的负载均衡器。
- 网络安全组 (NSG) 规则，阻止 VM 接受不需要的流量。

![基本环境概述](./media/virtual-machines-linux-create-cli-complete/environment_overview.png)

若要创建此自定义环境，需要在 Resource Manager 模式 (`azure config mode arm`) 下安装最新的 [Azure CLI](/documentation/articles/xplat-cli-install/)。此外，还需要安装 JSON 解析工具。本示例使用 [jq](https://stedolan.github.io/jq/)。

## 快速命令
可以使用以下快速命令构建自定义环境。有关构建环境时每条命令的作用的详细信息，请参阅[下面的详细演练步骤](#detailed-walkthrough)。

创建资源组：

	azure group create TestRG -l chinaeast

使用 JSON 分析器验证资源组：

	azure group show TestRG --json | jq '.'

创建存储帐户：

	azure storage account create -g TestRG -l chinaeast --kind Storage --sku-name GRS computeteststore

使用 JSON 分析器验证存储帐户：

	azure storage account show -g TestRG computeteststore --json | jq '.'

创建虚拟网络：

	azure network vnet create -g TestRG -n TestVNet -a 192.168.0.0/16 -l chinaeast

创建子网：

	azure network vnet subnet create -g TestRG -e TestVNet -n FrontEnd -a 192.168.1.0/24

使用 JSON 分析器验证虚拟网络和子网：

	azure network vnet show TestRG TestVNet --json | jq '.'

创建公共 IP 地址：

	azure network public-ip create -g TestRG -n TestLBPIP -l chinaeast -d testlb -a static -i 4

创建负载均衡器：

	azure network lb create -g TestRG -n TestLB -l chinaeast

创建负载均衡器的前端 IP 池并关联公共 IP：

	azure network lb frontend-ip create -g TestRG -l TestLB -n TestFrontEndPool -i TestLBPIP

创建负载均衡器的后端 IP 池：

	azure network lb address-pool create -g TestRG -l TestLB -n TestBackEndPool

创建负载均衡器的 SSH 入站 NAT 规则：

	azure network lb inbound-nat-rule create -g TestRG -l TestLB -n VM1-SSH -p tcp -f 4222 -b 22
	azure network lb inbound-nat-rule create -g TestRG -l TestLB -n VM2-SSH -p tcp -f 4223 -b 22

创建负载均衡器的 Web 入站 NAT 规则：

	azure network lb rule create -g TestRG -l TestLB -n WebRule -p tcp -f 80 -b 80 \
	     -t TestFrontEndPool -o TestBackEndPool

创建负载均衡器运行状况探测：

	azure network lb probe create -g TestRG -l TestLB -n HealthProbe -p "http" -f healthprobe.aspx -i 15 -c 4

使用 JSON 分析器验证负载均衡器、IP 池和 NAT 规则：

	azure network lb show -g TestRG -n TestLB --json | jq '.'

创建第一个网络接口卡 (NIC)：

	azure network nic create -g TestRG -n LB-NIC1 -l chinaeast --subnet-vnet-name TestVNet --subnet-name FrontEnd \
	    -d "/subscriptions/########-####-####-####-############/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool" \
	    -e "/subscriptions/########-####-####-####-############/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM1-SSH"

创建第二个 NIC：

	azure network nic create -g TestRG -n LB-NIC2 -l chinaeast --subnet-vnet-name TestVNet --subnet-name FrontEnd \
	    -d "/subscriptions/########-####-####-####-############/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool" \
	    -e "/subscriptions/########-####-####-####-############/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM2-SSH"

使用 JSON 分析器验证 NIC：

	azure network nic show TestRG LB-NIC1 --json | jq '.'
	azure network nic show TestRG LB-NIC2 --json | jq '.'

创建 NSG：

	azure network nsg create -g TestRG -n TestNSG -l chinaeast

为 NSG 添加入站规则：

	azure network nsg rule create --protocol tcp --direction inbound --priority 1000 \
	    --destination-port-range 22 --access allow -g TestRG -a TestNSG -n SSHRule
	azure network nsg rule create --protocol tcp --direction inbound --priority 1001 \
	    --destination-port-range 80 --access allow -g TestRG -a TestNSG -n HTTPRule

使用 JSON 分析器验证 NSG 和入站规则：

azure network nsg show -g TestRG -n TestNSG --json | jq '.'

将 NSG 绑定到 NIC：

	azure network nic set -g TestRG -n LB-NIC1 -o TestNSG
	azure network nic set -g TestRG -n LB-NIC2 -o TestNSG

创建可用性集：

	azure availset create -g TestRG -n TestAvailSet -l chinaeast

创建第一个 Linux VM：

	azure vm create \
	    --resource-group TestRG \
	    --name TestVM1 \
	    --location chinaeast \
	    --os-type linux \
	    --availset-name TestAvailSet \
	    --nic-name LB-NIC1 \
	    --vnet-name TestVnet \
	    --vnet-subnet-name FrontEnd \
	    --storage-account-name computeteststore \
    --image-urn canonical:UbuntuServer:16.04.0-LTS:latest \
	    --ssh-publickey-file ~/.ssh/id_rsa.pub \
	    --admin-username ops

创建第二个 Linux VM：

	azure vm create \
	    --resource-group TestRG \
	    --name TestVM2 \
	    --location chinaeast \
	    --os-type linux \
	    --availset-name TestAvailSet \
	    --nic-name LB-NIC2 \
	    --vnet-name TestVnet \
	    --vnet-subnet-name FrontEnd \
	    --storage-account-name computeteststore \
    --image-urn canonical:UbuntuServer:16.04.0-LTS:latest \
	    --ssh-publickey-file ~/.ssh/id_rsa.pub \
	    --admin-username ops

使用 JSON 分析器验证构建的所有组件：

	azure vm show -g TestRG -n TestVM1 --json | jq '.'
	azure vm show -g TestRG -n TestVM2 --json | jq '.'

将构建的环境导出到模板，以便快速重新创建新实例：

	azure resource export TestRG

## <a name="detailed-walkthrough"></a>详细演练
下面的详细步骤说明构建环境时每条命令的作用。了解这些概念有助于构建自己的自定义开发或生产环境。

## 创建资源组并选择部署位置

Azure 资源组是逻辑部署实体，包含用于启用资源部署逻辑管理的配置信息和元数据。

	azure group create TestRG chinaeast

输出：
```bash                        
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
```

## 创建存储帐户

需要为 VM 磁盘和要添加的任何其他数据磁盘创建存储帐户。创建资源组后，应立即创建存储帐户。

此处，使用 `azure storage account create` 命令传递帐户的位置、控制该帐户的资源组，以及所需的存储支持类型。

	azure storage account create \  
	--location chinaeast \
	--resource-group TestRG \
	--kind Storage --sku-name GRS \
	computeteststore

输出：

	info:    Executing command storage account create
	+ Creating storage account
	info:    storage account create command OK
	ahmet•~/workspace/keygen» azure group show testrg
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

若要通过 `azure group show` 命令检查资源组，可在 [jq](https://stedolan.github.io/jq/) 工具中结合使用 `--json` Azure CLI 选项。（可以使用 **jsawk** 或用于分析 JSON 的任何语言库。）

	azure group show TestRG --json | jq                                                                                      

输出：

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

若要使用 CLI 检查存储帐户，首先需要使用以下命令的其他形式设置帐户名和密钥。将下例中的存储帐户名替换为所选的名称：

	AZURE_STORAGE_CONNECTION_STRING="$(azure storage account connectionstring show computeteststore --resource-group testrg --json | jq -r '.string')"

然后，可以轻松查看存储信息：

azure 存储容器列表

输出：

	info:    Executing command storage container list
	+ Getting storage containers
	data:    Name  Public-Access  Last-Modified
	data:    ----  -------------  -----------------------------
	data:    vhds  Off            Sun, 27 Sep 2015 19:03:54 GMT
	info:    storage container list command OK

## 创建虚拟网络和子网

接下来，需要创建在 Azure 中运行的虚拟网络，以及可在其中安装 VM 的子网。

	azure network vnet create -g TestRG -n TestVNet -a 192.168.0.0/16 -l chinaeast

输出：

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

同样，可以了解如何使用 `azure group show` 的 --json 选项和 **jq** 构建资源。现在，已经创建 `storageAccounts` 资源和 `virtualNetworks` 资源。

	azure group show TestRG --json | jq '.'

输出：

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

现在，需在部署 VM 的 `TestVnet` 虚拟网络中创建子网。使用 `azure network vnet subnet create` 命令以及已创建的资源：`TestRG` 资源组和 `TestVNet` 虚拟网络。添加子网名称 `FrontEnd` 和子网地址前缀 `192.168.1.0/24`，如下所示：

	azure network vnet subnet create -g TestRG -e TestVNet -n FrontEnd -a 192.168.1.0/24

输出：

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

由于子网以逻辑方式出现在虚拟网络中，可使用稍微不同的命令来查找子网信息。使用的命令为 `azure network vnet show`，但将继续使用 **jq** 检查 JSON 输出。

	azure network vnet show TestRG TestVNet --json | jq '.'

输出：

	{
	  "subnets": [
	    {
	      "ipConfigurations": [],
	      "addressPrefix": "192.168.1.0/24",
	      "provisioningState": "Succeeded",
	      "name": "FrontEnd",
	      "etag": "W/\"974f3e2c-028e-4b35-832b-a4b16ad25eb6\"",
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
	  "etag": "W/\"974f3e2c-028e-4b35-832b-a4b16ad25eb6\"",
	  "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet",
	  "name": "TestVNet",
	  "location": "chinaeast"
	}

## 创建公共 IP 地址 (PIP)

现在，需要创建分配给负载均衡器的公共 IP 地址 (PIP)。使用该地址可以通过 `azure network public-ip create` 命令从 Internet 连接到 VM。由于默认地址是动态的，因此可使用 `-d testsubdomain` 选项在 **chinacloudapp.cn** 域中创建一个命名的 DNS 条目。

	azure network public-ip create -d testsubdomain TestRG TestPIP chinaeast

输出：

	info:    Executing command network public-ip create
	+ Looking up the public ip "TestPIP"
	+ Creating public ip address "TestPIP"
	+ Looking up the public ip "TestPIP"
	data:    Id                              : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestPIP
	data:    Name                            : TestPIP
	data:    Type                            : Microsoft.Network/publicIPAddresses
	data:    Location                        : chinaeast
	data:    Provisioning state              : Succeeded
	data:    Allocation method               : Dynamic
	data:    Idle timeout                    : 4
	data:    Domain name label               : testsubdomain
	data:    FQDN                            : testsubdomain.chinaeast.chinacloudapp.cn
	info:    network public-ip create command OK

公共 IP 地址也是顶级资源，因此可以使用 `azure group show` 查看。

	azure group show TestRG --json | jq '.'

输出：

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

可以查看更多资源详细信息，包括使用完整的 `azure network public-ip show` 命令查看子域的完全限定域名 (FQDN)。公共 IP 地址资源已经以逻辑方式分配，但尚未分配有特定地址。若要获取 IP 地址，需要一个负载均衡器，但该负载均衡器尚未创建。

	azure network public-ip show TestRG TestPIP --json | jq '.'

输出：

	{
	"tags": {},
	"publicIpAllocationMethod": "Dynamic",
	"dnsSettings": {
	    "domainNameLabel": "testsubdomain",
	    "fqdn": "testsubdomain.chinaeast.chinacloudapp.cn"
	},
	"idleTimeoutInMinutes": 4,
	"provisioningState": "Succeeded",
	"etag": "W/\"c63154b3-1130-49b9-a887-877d74d5ebc5\"",
	"id": "/subscriptions/guid/resourceGroups/testrg/providers/Microsoft.Network/publicIPAddresses/testpip",
	"name": "testpip",
	"location": "chinaeast"
	}

## 创建负载均衡器和 IP 池
创建负载均衡器时，可以将流量分散到多个 VM。负载均衡器还可以在执行维护或承受重负载时运行多个 VM 来响应用户请求，为应用程序提供冗余。

我们将创建具有以下特点的负载均衡器：

	azure network lb create -g TestRG -n TestLB -l chinaeast

	azure network lb create -g TestRG -n TestLB -l chinaeast

输出：

	info:    Executing command network lb create
	+ Looking up the load balancer "TestLB"
	+ Creating load balancer "TestLB"
	data:    Id                              : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB
	data:    Name                            : TestLB
	data:    Type                            : Microsoft.Network/loadBalancers
	data:    Location                        : chinaeast
	data:    Provisioning state              : Succeeded
	info:    network lb create command OK

我们的负载均衡器很空，因此让我们创建一些 IP 池。我们想要为负载均衡器创建两个 IP 池：一个用于前端，一个用于后端。前端 IP 池将公开显示。它也是我们将前面创建的 PIP 分配到的位置。然后我们使用后端池作为 VM 要连接到的位置。这样，流量便可以通过负载均衡器流向 VM。

首先，创建前端 IP 池：

	azure network lb frontend-ip create -g TestRG -l TestLB -n TestFrontEndPool -i TestLBPIP

输出：

	info:    Executing command network lb frontend-ip create
	+ Looking up the load balancer "TestLB"
	+ Looking up the public ip "TestLBPIP"
	+ Updating load balancer "TestLB"
	data:    Name                            : TestFrontEndPool
	data:    Provisioning state              : Succeeded
	data:    Private IP allocation method    : Dynamic
	data:    Public IP address id            : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestLBPIP
	info:    network lb frontend-ip create command OK

请注意我们如何使用 `--public-ip-name` 开关传入前面创建的 TestLBPIP。通过将公共 IP 地址分配给负载均衡器，可以通过 Internet 访问 VM。

接下来，让我们创建第二个 IP 池，用于传输后端流量：

	azure network lb address-pool create -g TestRG -l TestLB -n TestBackEndPool

输出：

	info:    Executing command network lb address-pool create
	+ Looking up the load balancer "TestLB"
	+ Updating load balancer "TestLB"
	data:    Name                            : TestBackEndPool
	data:    Provisioning state              : Succeeded
	info:    network lb address-pool create command OK

可以通过查看 `azure network lb show` 和 JSON 输出来了解负载均衡器的工作情况：

	azure network lb show TestRG TestLB --json | jq '.'

输出：

	{
	  "etag": "W/\"29c38649-77d6-43ff-ab8f-977536b0047c\"",
	  "provisioningState": "Succeeded",
	  "resourceGuid": "f1446acb-09ba-44d9-b8b6-849d9983dc09",
	  "outboundNatRules": [],
	  "inboundNatPools": [],
	  "inboundNatRules": [],
	  "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB",
	  "name": "TestLB",
	  "type": "Microsoft.Network/loadBalancers",
	  "location": "chinaeast",
	  "frontendIPConfigurations": [
	    {
	      "etag": "W/\"29c38649-77d6-43ff-ab8f-977536b0047c\"",
	      "name": "TestFrontEndPool",
	      "provisioningState": "Succeeded",
	      "publicIPAddress": {
	        "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestLBPIP"
	      },
	      "privateIPAllocationMethod": "Dynamic",
	      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/frontendIPConfigurations/TestFrontEndPool"
	    }
	  ],
	  "backendAddressPools": [
	    {
	      "etag": "W/\"29c38649-77d6-43ff-ab8f-977536b0047c\"",
	      "name": "TestBackEndPool",
	      "provisioningState": "Succeeded",
	      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool"
	    }
	  ],
	  "loadBalancingRules": [],
	  "probes": []
	}

## 创建负载均衡器 NAT 规则
若要获取流经负载均衡器的流量，需要创建 NAT 规则来指定入站或出站操作。可以指定要使用的协议，然后根据需要将外部端口映射到内部端口。针对我们的环境，让我们创建一些规则，以允许通过负载均衡器对 VM 进行 SSH 访问。将 TCP 端口 4222 和 4223 设置为定向到 VM 上的 TCP 端口 22（稍后将会创建）：

	azure network lb inbound-nat-rule create -g TestRG -l TestLB -n VM1-SSH -p tcp -f 4222 -b 22

输出：

	info:    Executing command network lb inbound-nat-rule create
	+ Looking up the load balancer "TestLB"
	warn:    Using default enable floating ip: false
	warn:    Using default idle timeout: 4
	warn:    Using default frontend IP configuration "TestFrontEndPool"
	+ Updating load balancer "TestLB"
	data:    Name                            : VM1-SSH
	data:    Provisioning state              : Succeeded
	data:    Protocol                        : Tcp
	data:    Frontend port                   : 4222
	data:    Backend port                    : 22
	data:    Enable floating IP              : false
	data:    Idle timeout in minutes         : 4
	data:    Frontend IP configuration id    : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/frontendIPConfigurations/TestFrontEndPool
	info:    network lb inbound-nat-rule create command OK

对于第二个 NAT 规则中的 SSH 访问，重复该过程：

	azure network lb inbound-nat-rule create -g TestRG -l TestLB -n VM2-SSH -p tcp -f 4223 -b 22

让我们继续为 TCP 端口 80 创建 NAT 规则，并将该规则挂接到 IP 池。如果将规则挂接到 IP 池，而不是将规则逐个挂接到 VM，则可以直接在 IP 池中添加或删除 VM。然后，负载均衡器会自动调整流量流：

	azure network lb rule create -g TestRG -l TestLB -n WebRule -p tcp -f 80 -b 80 \
	     -t TestFrontEndPool -o TestBackEndPool

输出：

	info:    Executing command network lb rule create
	+ Looking up the load balancer "TestLB"
	warn:    Using default idle timeout: 4
	warn:    Using default enable floating ip: false
	warn:    Using default load distribution: Default
	+ Updating load balancer "TestLB"
	data:    Name                            : WebRule
	data:    Provisioning state              : Succeeded
	data:    Protocol                        : Tcp
	data:    Frontend port                   : 80
	data:    Backend port                    : 80
	data:    Enable floating IP              : false
	data:    Load distribution               : Default
	data:    Idle timeout in minutes         : 4
	data:    Frontend IP configuration id    : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/frontendIPConfigurations/TestFrontEndPool
	data:    Backend address pool id         : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool
	info:    network lb rule create command OK

## 创建负载均衡器运行状况探测

运行状况探测定期检查受负载均衡器后面的 VM，以确保它们可以根据定义操作和响应请求。否则，将从操作中删除这些 VM，确保不会将用户定向到它们。可以针对运行状况探测定义自定义检查，以及间隔和超时值。有关运行状况探测的详细信息，请参阅 [Load Balancer probes](/documentation/articles/load-balancer-custom-probe-overview/)（负载均衡器探测）。

	azure network lb probe create -g TestRG -l TestLB -n HealthProbe -p "http" -f healthprobe.aspx -i 15 -c 4

输出：

	info:    Executing command network lb probe create
	warn:    Using default probe port: 80
	+ Looking up the load balancer "TestLB"
	+ Updating load balancer "TestLB"
	data:    Name                            : HealthProbe
	data:    Provisioning state              : Succeeded
	data:    Protocol                        : Tcp
	data:    Port                            : 80
	data:    Interval in seconds             : 15
	data:    Number of probes                : 4
	info:    network lb probe create command OK

此处我们指定了 15 秒的运行状况检查间隔。在负载均衡器将该主机视为不再正常运行之前，我们最多可能会错过四个探测（1 分钟）。

## 验证负载均衡器

现已完成负载均衡器配置。以下是执行的步骤：

1. 首先，创建了负载均衡器。
2. 其次，创建前端 IP 池并为其分配公共 IP 地址。
3. 再次，创建 VM 可以连接到的后端 IP 池。
4. 接下来，创建允许通过 SSH 连接到 VM 以进行管理的 NAT 规则，以及允许对 Web 应用使用 TCP 端口 80 的规则。
5. 最后，添加了一个运行状况探测来定期检查 VM。此运行状况探测可以确保用户不会尝试访问不再正常运行和不再提供内容的 VM。

让我们查看负载均衡器现在的情形：

	azure network lb show -g TestRG -n TestLB --json | jq '.'

输出：

	{
	  "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
	  "provisioningState": "Succeeded",
	  "resourceGuid": "f1446acb-09ba-44d9-b8b6-849d9983dc09",
	  "outboundNatRules": [],
	  "inboundNatPools": [],
	  "inboundNatRules": [
	    {
	      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
	      "name": "VM1-SSH",
	      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM1-SSH",
	      "frontendIPConfiguration": {
	        "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/frontendIPConfigurations/TestFrontEndPool"
	      },
	      "protocol": "Tcp",
	      "frontendPort": 4222,
	      "backendPort": 22,
	      "idleTimeoutInMinutes": 4,
	      "enableFloatingIP": false,
	      "provisioningState": "Succeeded"
	    },
	    {
	      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
	      "name": "VM2-SSH",
	      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM2-SSH",
	      "frontendIPConfiguration": {
	        "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/frontendIPConfigurations/TestFrontEndPool"
	      },
	      "protocol": "Tcp",
	      "frontendPort": 4223,
	      "backendPort": 22,
	      "idleTimeoutInMinutes": 4,
	      "enableFloatingIP": false,
	      "provisioningState": "Succeeded"
	    }
	  ],
	  "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB",
	  "name": "TestLB",
	  "type": "Microsoft.Network/loadBalancers",
	  "location": "chinaeast",
	  "frontendIPConfigurations": [
	    {
	      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
	      "name": "TestFrontEndPool",
	      "provisioningState": "Succeeded",
	      "publicIPAddress": {
	        "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestLBPIP"
	      },
	      "privateIPAllocationMethod": "Dynamic",
	      "loadBalancingRules": [
	        {
	          "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/loadBalancingRules/WebRule"
	        }
	      ],
	      "inboundNatRules": [
	        {
	          "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM1-SSH"
	        },
	        {
	          "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM2-SSH"
	        }
	      ],
	      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/frontendIPConfigurations/TestFrontEndPool"
	    }
	  ],
	  "backendAddressPools": [
	    {
	      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
	      "name": "TestBackEndPool",
	      "provisioningState": "Succeeded",
	      "loadBalancingRules": [
	        {
	          "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/loadBalancingRules/WebRule"
	        }
	      ],
	      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool"
	    }
	  ],
	  "loadBalancingRules": [
	    {
	      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
	      "name": "WebRule",
	      "provisioningState": "Succeeded",
	      "enableFloatingIP": false,
	      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/loadBalancingRules/WebRule",
	      "frontendIPConfiguration": {
	        "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/frontendIPConfigurations/TestFrontEndPool"
	      },
	      "backendAddressPool": {
	        "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool"
	      },
	      "protocol": "Tcp",
	      "loadDistribution": "Default",
	      "frontendPort": 80,
	      "backendPort": 80,
	      "idleTimeoutInMinutes": 4
	    }
	  ],
	  "probes": [
	    {
	      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
      "name": "HealthProbe",
      "provisioningState": "Succeeded",
      "numberOfProbes": 4,
      "intervalInSeconds": 15,
      "port": 80,
      "protocol": "Tcp",
      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/probes/HealthProbe"
        }
    ]
    }

## 创建用于 Linux VM 的 NIC

 由于可以对 NIC 使用应用规则，因此能以编程方式使用 NIC。可以创建多个规则。在下面的 `azure network nic create` 命令中，要将 NIC 挂接到负载后端 IP 池，并与 NAT 规则关联以允许 SSH 流量。为此，需要指定 Azure 订阅的订阅 ID 来取代 `<GUID>`：

	azure network nic create -g TestRG -n LB-NIC1 -l chinaeast --subnet-vnet-name TestVNet --subnet-name FrontEnd \
	     -d /subscriptions/<GUID>/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool \
	     -e /subscriptions/<GUID>/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM1-SSH

输出：

	info:    Executing command network nic create
	+ Looking up the subnet "FrontEnd"
	+ Looking up the network interface "LB-NIC1"
	+ Creating network interface "LB-NIC1"
	data:    Id                              : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/LB-NIC1
	data:    Name                            : LB-NIC1
	data:    Type                            : Microsoft.Network/networkInterfaces
	data:    Location                        : chinaeast
	data:    Provisioning state              : Succeeded
	data:    Enable IP forwarding            : false
	data:    IP configurations:
	data:      Name                          : Nic-IP-config
	data:      Provisioning state            : Succeeded
	data:      Private IP address            : 192.168.1.4
	data:      Private IP allocation method  : Dynamic
	data:      Subnet                        : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
	data:      Load balancer backend address pools:
	data:        Id                          : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool
	data:      Load balancer inbound NAT rules:
	data:        Id                          : /subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM1-SSH
	data:
	info:    network nic create command OK

可以通过直接检查资源来查看详细信息。使用 `azure network nic show` 命令检查资源：

	azure network nic show TestRG LB-NIC1 --json | jq '.'

输出：

	{
	  "etag": "W/\"fc1eaaa1-ee55-45bd-b847-5a08c7f4264a\"",
	  "provisioningState": "Succeeded",
	  "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/LB-NIC1",
	  "name": "LB-NIC1",
	  "type": "Microsoft.Network/networkInterfaces",
	  "location": "chinaeast",
	  "ipConfigurations": [
	    {
	      "etag": "W/\"fc1eaaa1-ee55-45bd-b847-5a08c7f4264a\"",
	      "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/LB-NIC1/ipConfigurations/Nic-IP-config",
	      "loadBalancerBackendAddressPools": [
	        {
	          "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool"
	        }
	      ],
	      "loadBalancerInboundNatRules": [
	        {
	          "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM1-SSH"
	        }
	      ],
	      "privateIPAddress": "192.168.1.4",
	      "privateIPAllocationMethod": "Dynamic",
	      "subnet": {
	        "id": "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
	      },
	      "provisioningState": "Succeeded",
	      "name": "Nic-IP-config"
	    }
	  ],
	  "dnsSettings": {
	    "appliedDnsServers": [],
	    "dnsServers": []
	  },
	  "enableIPForwarding": false,
	  "resourceGuid": "a20258b8-6361-45f6-b1b4-27ffed28798c"
	}

现在，我们将创建第二个 NIC 并同样将其挂接到后端 IP 池。这一次，第二个 NAT 规则将允许 SSH 流：

	azure network nic create -g TestRG -n LB-NIC2 -l chinaeast --subnet-vnet-name TestVNet --subnet-name FrontEnd \
	    -d  /subscriptions/<GUID>/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/backendAddressPools/TestBackEndPool \
	    -e /subscriptions/<GUID>/resourceGroups/TestRG/providers/Microsoft.Network/loadBalancers/TestLB/inboundNatRules/VM2-SSH

## 创建网络安全组和规则

现在，将创建 NSG 和用于控制 NIC 访问权限的入站规则。

	azure network nsg create TestRG TestNSG chinaeast

添加 NSG 的入站规则，允许端口 22 上的入站连接（以支持 SSH）：

	azure network nsg rule create --protocol tcp --direction inbound --priority 1000 \
	    --destination-port-range 22 --access allow TestRG TestNSG SSHRule

	azure network nsg rule create --protocol tcp --direction inbound --priority 1001 \
	    --destination-port-range 80 --access allow -g TestRG -a TestNSG -n HTTPRule

> [AZURE.NOTE] 入站规则是入站网络连接的筛选器。在本示例中，我们将 NSG 绑定到 VM 虚拟 NIC，这意味着任何对端口 22 的请求都将在 VM 上传递到 NIC。此入站规则与网络连接相关，而不与终结点相关（终结点与经典部署相关）。若要打开端口，必须将 `--source-port-range` 保持设置为“*”（默认值）才能接受来自**任何**请求端口的入站请求。端口通常是动态的。

## 绑定到 NIC

将 NSG 绑定到 NIC：

	azure network nic set -g TestRG -n LB-NIC1 -o TestNSG

	azure network nic set -g TestRG -n LB-NIC2 -o TestNSG

## 创建可用性集
可用性集有助于将 VM 分散到容错域和升级域。为 VM 创建可用性集：

	azure availset create -g TestRG -n TestAvailSet -l chinaeast

容错域定义共享通用电源和网络交换机的一组虚拟机。默认情况下，在可用性集中配置的虚拟机隔离在最多三个容错域中。思路是其中一个容错域中的硬件问题不会影响运行应用的每个 VM。将多个 VM 放入一个可用性集时，Azure 会自动将它们分散到容错域。

升级域表示虚拟机组以及可同时重新启动的基础物理硬件。在计划内维护期间，升级域的重新启动顺序可能不会按序进行，但一次只重新启动一个升级域。同样，将多个 VM 放入一个可用性站点时，Azure 会自动将它们分散到升级域。

有关详细信息，请阅读[管理 VM 可用性](/documentation/articles/virtual-machines-linux-manage-availability/)。

## 创建 Linux VM

已经创建存储和网络资源，支持可访问 Internet 的 VM。现在，创建 VM 并使用不含密码的 SSH 密钥保护其安全。在此情况下，我们需要基于最新的 LTS 创建 Ubuntu VM。我们将根据 [finding Azure VM images](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)（查找 Azure VM 映像）中所述，使用 `azure vm image list` 来查找该映像信息。

我们使用命令 `azure vm image list westeurope canonical | grep LTS` 选择了映像。在此示例中，使用 `canonical:UbuntuServer:16.04.0-LTS:16.04.201608150`。对于最后一个字段，我们将传递 `latest`，以便将来可随时获取最新的内部版本。（使用的字符串是 `canonical:UbuntuServer:16.04.0-LTS:16.04.201608150`）。

已使用 **ssh-keygen -t rsa -b 2048** 在 Linux 或 Mac 上创建 ssh rsa 公钥和私钥对的任何人都熟悉下一个步骤。如果 `~/.ssh` 目录中没有任何证书密钥对，可以创建证书密钥对：

- 使用 `azure vm create --generate-ssh-keys` 选项自动创建。
- [根据说明手动自行创建](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)。

或者，可以在创建 VM 之后，使用 --admin-password 方法对 SSH 连接进行身份验证。此方法通常不太安全。我们使用 `azure vm create` 命令并结合所有资源和信息来创建 VM：

	azure vm create \            
	    --resource-group TestRG \
	    --name TestVM1 \
	    --location chinaeast \
	    --os-type linux \
	    --availset-name TestAvailSet \
	    --nic-name LB-NIC1 \
	    --vnet-name TestVnet \
	    --vnet-subnet-name FrontEnd \
	    --storage-account-name computeteststore \
	    --image-urn canonical:UbuntuServer:16.04.0-LTS:latest \
	    --ssh-publickey-file ~/.ssh/id_rsa.pub \
	    --admin-username ops

输出：

	info:    Executing command vm create
	+ Looking up the VM "TestVM1"
	info:    Verifying the public key SSH file: /home/ahmet/.ssh/id_rsa.pub
	info:    Using the VM Size "Standard_DS1"
	info:    The [OS, Data] Disk or image configuration requires storage account
	+ Looking up the storage account computeteststore
	+ Looking up the availability set "TestAvailSet"
	info:    Found an Availability set "TestAvailSet"
	+ Looking up the NIC "LB-NIC1"
	info:    Found an existing NIC "LB-NIC1"
	info:    Found an IP configuration with virtual network subnet id "/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd" in the NIC "LB-NIC1"
	info:    This is an NIC without publicIP configured
	info:    The storage URI 'https://computeteststore.blob.core.chinacloudapi.cn/' will be used for boot diagnostics settings, and it can be overwritten by the parameter input of '--boot-diagnostics-storage-uri'.
	info:    vm create command OK

可以使用默认的 SSH 密钥立即连接到 VM。请确保指定适当的端口，因为我们要通过负载均衡器传递流量。（对于第一个 VM，设置 NAT 规则以将端口 4222 转发到 VM）：

	ssh ops@testlb.chinaeast.chinacloudapp.cn -p 4222

输出：

	The authenticity of host '[testlb.chinaeast.chinacloudapp.cn]:4222 ([xx.xx.xx.xx]:4222)' can't be established.
	ECDSA key fingerprint is 94:2d:d0:ce:6b:fb:7f:ad:5b:3c:78:93:75:82:12:f9.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added '[testlb.chinaeast.chinacloudapp.cn]:4222,[xx.xx.xx.xx]:4222' (ECDSA) to the list of known hosts.
	Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-34-generic x86_64)
	
* 文档：https://help.ubuntu.com	
* 管理：https://landscape.canonical.com
* 支持：https://ubuntu.com/advantage
	
    使用 Ubuntu 优势云来宾获取云支持：

      http://www.ubuntu.com/business/services/cloud
	
	可以更新 0 个包。
	0 项更新是安全更新。
	
	
Ubuntu 系统附带的程序是免费软件；每个程序的具体分发条款在 /usr/share/doc/*/copyright 中的单个文件中说明。

Ubuntu 附带了在适用法律允许的范围内绝对不担保的声明。

若要以管理员（用户“root”）身份运行命令，请使用“sudo <command>”。
有关详细信息，请参阅“man sudo\_root”。

    ops@TestVM1：~$

以相同的方式继续创建第二个 VM：

	azure vm create \            
	    --resource-group TestRG \
	    --name TestVM2 \
	    --location chinaeast \
	    --os-type linux \
	    --availset-name TestAvailSet \
	    --nic-name LB-NIC2 \
	    --vnet-name TestVnet \
	    --vnet-subnet-name FrontEnd \
	    --storage-account-name computeteststore \
	    --image-urn canonical:UbuntuServer:16.04.0-LTS:latest \
	    --ssh-publickey-file ~/.ssh/id_rsa.pub \
	    --admin-username ops

现在，可以使用 `azure vm show testrg testvm` 命令检查创建的内容。此时，已在 Azure 中运行了一个位于负载均衡器后面的 Ubuntu VM，只能使用 SSH 密钥对登录到该 VM（因为密码已禁用）。可以安装 nginx 或 httpd、部署 Web 应用，以及查看流量是否通过负载均衡器流向两个 VM。

	azure vm show TestRG TestVM1

输出：

	info:    Executing command vm show
	+ Looking up the VM "TestVM1"
	+ Looking up the NIC "LB-NIC1"
	data:    Id                              :/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/TestVM1
	data:    ProvisioningState               :Succeeded
	data:    Name                            :TestVM1
	data:    Location                        :chinaeast
	data:    Type                            :Microsoft.Compute/virtualMachines
	data:
	data:    Hardware Profile:
	data:      Size                          :Standard_DS1
	data:
	data:    Storage Profile:
	data:      Image reference:
	data:        Publisher                   :canonical
	data:        Offer                       :UbuntuServer
	data:        Sku                         :16.04.0-LTS
	data:        Version                     :latest
	data:
	data:      OS Disk:
	data:        OSType                      :Linux
	data:        Name                        :clib45a8b650f4428a1-os-1471973896525
	data:        Caching                     :ReadWrite
	data:        CreateOption                :FromImage
	data:        Vhd:
	data:          Uri                       :https://computeteststore.blob.core.chinacloudapi.cn/vhds/clib45a8b650f4428a1-os-1471973896525.vhd
	data:
	data:    OS Profile:
	data:      Computer Name                 :TestVM1
	data:      User Name                     :ops
	data:      Linux Configuration:
	data:        Disable Password Auth       :true
	data:
	data:    Network Profile:
	data:      Network Interfaces:
	data:        Network Interface #1:
	data:          Primary                   :true
	data:          MAC Address               :00-0D-3A-24-D4-AA
	data:          Provisioning State        :Succeeded
	data:          Name                      :LB-NIC1
	data:          Location                  :chinaeast
	data:
	data:    AvailabilitySet:
	data:      Id                            :/subscriptions/guid/resourceGroups/TestRG/providers/Microsoft.Compute/availabilitySets/TESTAVAILSET
	data:
	data:    Diagnostics Profile:
	data:      BootDiagnostics Enabled       :true
	data:      BootDiagnostics StorageUri    :https://computeteststore.blob.core.chinacloudapi.cn/
	data:
	data:      Diagnostics Instance View:
	info:    vm show command OK

## 将环境导出为模板
现已构建此环境，如果要使用相同的参数创建与其相符的额外开发环境或生产环境，该怎么办？ Resource Manager 使用定义了所有环境参数的 JSON 模板。通过引用此 JSON 模板构建出整个环境。可以[手动构建 JSON 模板](/documentation/articles/resource-group-authoring-templates/)，也可以通过直接导出现有环境来为自己创建 JSON 模板：

	azure group export TestRG

此命令在当前工作目录中创建 `TestRG.json` 文件。从此模板创建环境时，系统会提示输入所有资源名称，包括负载均衡器、网络接口或 VM 的名称。可以通过向前面所示的 `azure group export` 命令中添加 `-p` 或 `--includeParameterDefaultValue` 参数，在模板文件中填充这些名称。请编辑 JSON 模板以指定资源名称，或[创建 parameters.json 文件](/documentation/articles/resource-group-authoring-templates/#parameters)来指定资源名称。

使用模板创建环境：

	azure group deployment create -f TestRG.json -g NewRGFromTemplate

可能需要阅读[有关通过模板进行部署的详细信息](/documentation/articles/resource-group-template-deploy-cli/)。了解如何对环境进行增量更新、如何使用参数文件，以及如何从单个存储位置访问模板。

## 后续步骤

现在，已准备好开始使用多个网络组件和 VM。可以使用本文介绍的核心组件，通过此示例环境构建应用程序。

<!---HONumber=Mooncake_Quality_Review_1118_2016-->