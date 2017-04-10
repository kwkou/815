<properties
    pageTitle="使用 Azure CLI 2.0 创建 Linux 环境 | Azure"
    description="使用 Azure CLI 2.0（预览版）从头开始创建存储、Linux VM、虚拟网络和子网、负载均衡器、NIC、公共 IP 和网络安全组。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags 
    ms.assetid="4ba4060b-ce95-4747-a735-1d7c68597a1a"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="12/8/2016"
    wacn.date="04/10/2017"
    ms.author="iainfou" />

# 使用 Azure CLI 2.0（预览版）创建完整的 Linux 环境
在本文中，我们将构建一个简单网络，其中包含一个负载均衡器，以及一对可用于开发和简单计算的 VM。将以逐条命令的方式完成整个过程，直到创建两个可以从 Internet 上的任何位置连接的有效且安全的 Linux VM。然后，便可以继续构建更复杂的网络和环境。

在此过程中，将了解 Resource Manager 部署模型提供的依赖性层次结构及其提供的功能。明白系统是如何构建的以后，即可使用 [Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)更快速地重新构建系统。此外，了解环境的各个部分如何彼此配合运行后，可以更轻松创建模板以实现自动化。

该环境包含：

* 两个位于可用性集中的 VM。
* 端口 80 上有一个带负载均衡规则的负载均衡器。
* 网络安全组 (NSG) 规则，阻止 VM 接受不需要的流量。

![基本环境概述](./media/virtual-machines-linux-create-cli-complete/environment_overview.png)  


## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-create-cli-complete-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0（预览版）](#quick-commands)：用于资源管理部署模型（本文）的下一代 CLI

## <a name="quick-commands"></a> 快速命令
如果需要快速完成任务，请参阅以下部分，其中详细说明了用于将 VM 上载到 Azure 的基本命令。本文档的余下部分（[从此处开始](#detailed-walkthrough)）提供了每个步骤的更详细信息和应用背景。

在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`mystorageaccount` 和 `myVM`。

若要创建此自定义环境，需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

首先，使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建资源组。以下示例在 `chinanorth` 位置创建名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

使用非托管磁盘，需使用 [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#create) 创建存储帐户。以下示例创建名为 `mystorageaccount` 的存储帐户。（存储帐户名称必须唯一，因此，请提供自己的唯一名称。）

    az storage account create --resource-group myResourceGroup --location chinanorth \
      --name mystorageaccount --kind Storage --sku Standard_LRS

使用 [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#create) 创建虚拟网络。以下示例创建名为 `myVnet` 的虚拟网络和名为 `mySubnet` 的子网：

    az network vnet create --resource-group myResourceGroup --location chinanorth --name myVnet \
      --address-prefix 192.168.0.0/16 --subnet-name mySubnet --subnet-prefix 192.168.1.0/24

使用 [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#create) 创建一个公共 IP。以下示例创建名为 `myPublicIP`、DNS 名称为 `mypublicdns` 的公共 IP。（DNS 名称必须唯一，因此，请提供自己的唯一名称。）

    az network public-ip create --resource-group myResourceGroup --location chinanorth \
      --name myPublicIP --dns-name mypublicdns --allocation-method static --idle-timeout 4

使用 [az network lb create](https://docs.microsoft.com/cli/azure/network/lb#create) 创建负载均衡器。以下示例：

- 创建名为 `myLoadBalancer` 的负载均衡器
- 关联公共 IP `myPublicIP`
- 创建名为 `mySubnetPool` 的前端 IP 池
- 创建名为 `myBackEndPool` 的后端 IP 池

        az network lb create --resource-group myResourceGroup --location chinanorth \
          --name myLoadBalancer --public-ip-address myPublicIP \
          --frontend-ip-name myFrontEndPool --backend-pool-name myBackEndPool

使用 [az network lb inbound-nat-rule create](https://docs.microsoft.com/cli/azure/network/lb/inbound-nat-rule#create) 创建负载均衡器的 SSH 入站网络地址转换 (NAT) 规则。以下示例创建两个负载均衡器规则，分别名为 `myLoadBalancerRuleSSH1` 和 `myLoadBalancerRuleSSH2`：

    az network lb inbound-nat-rule create --resource-group myResourceGroup \
      --lb-name myLoadBalancer --name myLoadBalancerRuleSSH1 --protocol tcp \
      --frontend-port 4222 --backend-port 22 --frontend-ip-name myFrontEndPool
    az network lb inbound-nat-rule create --resource-group myResourceGroup \
      --lb-name myLoadBalancer --name myLoadBalancerRuleSSH2 --protocol tcp \
      --frontend-port 4223 --backend-port 22 --frontend-ip-name myFrontEndPool

使用 [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe#create) 创建负载均衡器运行状况探测。以下示例创建名为 `myHealthProbe` 的 TCP 探测：

    az network lb probe create --resource-group myResourceGroup --lb-name myLoadBalancer \
      --name myHealthProbe --protocol tcp --port 80 --interval 15 --threshold 4

使用 [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule#create) 创建负载均衡器的 Web 入站 NAT 规则。以下示例创建名为 `myLoadBalancerRuleWeb` 的负载均衡器规则并将其与 `myHealthProbe` 探测相关联：

    az network lb rule create --resource-group myResourceGroup --lb-name myLoadBalancer \
      --name myLoadBalancerRuleWeb --protocol tcp --frontend-port 80 --backend-port 80 \
      --frontend-ip-name myFrontEndPool --backend-pool-name myBackEndPool \
      --probe-name myHealthProbe

使用 [az network lb show](https://docs.microsoft.com/cli/azure/network/lb#show) 验证负载均衡器、IP 池和 NAT 规则：

    az network lb show --resource-group myResourceGroup --name myLoadBalancer

使用 [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg#create) 创建网络安全组。以下示例创建名为 `myNetworkSecurityGroup` 的网络安全组：

    az network nsg create --resource-group myResourceGroup --location chinanorth \
      --name myNetworkSecurityGroup

使用 [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule#create) 为网络安全组添加两个入站规则。以下示例创建两个规则，分别名为 `myNetworkSecurityGroupRuleSSH` 和 `myNetworkSecurityGroupRuleHTTP`：

    az network nsg rule create --resource-group myResourceGroup \
      --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRuleSSH \
      --protocol tcp --direction inbound --priority 1000 --source-address-prefix '*' \
      --source-port-range '*' --destination-address-prefix '*' --destination-port-range 22 \
      --access allow
    az network nsg rule create --resource-group myResourceGroup \
      --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRuleHTTP \
      --protocol tcp --direction inbound --priority 1001 --source-address-prefix '*' \
      --source-port-range '*' --destination-address-prefix '*' --destination-port-range 80 \
      --access allow

使用 [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#create) 创建第一个网络接口卡 (NIC)。以下示例创建名为 `myNic1` 的 NIC 并将其附加到负载均衡器 `myLoadBalancer` 和相应的池，还将其附加到 `myNetworkSecurityGroup`：

    az network nic create --resource-group myResourceGroup --location chinanorth --name myNic1 \
      --vnet-name myVnet --subnet mySubnet --network-security-group myNetworkSecurityGroup \
      --lb-name myLoadBalancer --lb-address-pools myBackEndPool \
      --lb-inbound-nat-rules myLoadBalancerRuleSSH1

再次使用 **az network nic create** 创建第二个 NIC。以下示例创建名为 `myNic2` 的 NIC：

    az network nic create --resource-group myResourceGroup --location chinanorth --name myNic2 \
      --vnet-name myVnet --subnet mySubnet --network-security-group myNetworkSecurityGroup \
      --lb-name myLoadBalancer --lb-address-pools myBackEndPool \
      --lb-inbound-nat-rules myLoadBalancerRuleSSH2

使用 [az vm availability-set create](https://docs.microsoft.com/cli/azure/vm/availability-set#create) 创建可用性集。以下示例创建名为 `myAvailabilitySet` 的可用性集：

    az vm availability-set create --resource-group myResourceGroup --location chinanorth \
      --name myAvailabilitySet

使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建第一个 Linux VM。以下示例使用 Azure 非托管磁盘创建名为 `myVM1` 的 VM。

    az vm create \
        --resource-group myResourceGroup \
        --name myVM1 \
        --location chinanorth \
        --availability-set myAvailabilitySet \
        --nics myNic1 \
        --vnet myVnet \
        --subnet-name mySubnet \
        --nsg myNetworkSecurityGroup \
        --image UbuntuLTS \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --admin-username azureuser \
        --use-unmanaged-disk \
        --storage-account mystorageaccount

再次使用 **az vm create** 创建第二个 Linux VM。以下示例创建名为 `myVM2` 的 VM：

    az vm create \
        --resource-group myResourceGroup \
        --name myVM2 \
        --location chinanorth \
        --availability-set myAvailabilitySet \
        --nics myNic2 \
        --vnet myVnet \
        --subnet-name mySubnet \
        --nsg myNetworkSecurityGroup \
        --image UbuntuLTS \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --admin-username azureuser \
        --use-unmanaged-disk \
        --storage-account mystorageaccount

使用 [az vm show](https://docs.microsoft.com/cli/azure/vm#show) 验证所有项是否均已正确生成：

    az vm show --resource-group myResourceGroup --name myVM1
    az vm show --resource-group myResourceGroup --name myVM2

使用 [az group export](https://docs.microsoft.com/cli/azure/group#export) 将新环境导出到模板，以便快速重新创建新实例：

    az group export --name myResourceGroup > myResourceGroup.json

## <a name="detailed-walkthrough"></a> 详细演练
下面的详细步骤说明构建环境时每条命令的作用。了解这些概念有助于构建自己的自定义开发或生产环境。

请确保已安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并已使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`mystorageaccount` 和 `myVM`。

## 创建资源组并选择部署位置
Azure 资源组是逻辑部署实体，包含用于启用资源部署逻辑管理的配置信息和元数据。使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建资源组。以下示例在 `chinanorth` 位置创建名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

默认情况下，输出采用 JSON 格式（JavaScript 对象表示法）。若要输出为列表或表（例如），请使用 [az configure --output](https://docs.microsoft.com/cli/azure/#configure)。还可以向任何命令添加 `--output` 以一次性地更改输出格式。以下示例演示 **az group create** 命令的 JSON 输出：

    {
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup",
      "location": "chinanorth",
      "name": "myResourceGroup",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null
    }

## 创建存储帐户

使用非托管磁盘，需要为 VM 磁盘和想要添加的其他任何数据磁盘创建存储帐户。

此处，我们使用 [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#create)，并传递帐户的位置、控制该帐户的资源组，以及所需的存储支持类型。以下示例创建名为 `mystorageaccount` 的存储帐户：

    az storage account create --resource-group myResourceGroup --location chinanorth \
      --name mystorageaccount --kind Storage --sku Standard_LRS

输出：

    {
      "accessTier": null,
      "creationTime": "2016-12-07T17:59:50.090092+00:00",
      "customDomain": null,
      "encryption": null,
      "id": "/subscriptions/guid/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
      "kind": "Storage",
      "lastGeoFailoverTime": null,
      "location": "chinanorth",
      "name": "mystorageaccount",
      "primaryEndpoints": {
        "blob": "https://mystorageaccount.blob.core.chinacloudapi.cn/",
        "file": "https://mystorageaccount.file.core.chinacloudapi.cn/",
        "queue": "https://mystorageaccount.queue.core.chinacloudapi.cn/",
        "table": "https://mystorageaccount.table.core.chinacloudapi.cn/"
      },
      "primaryLocation": "chinanorth",
      "provisioningState": "Succeeded",
      "resourceGroup": "myresourcegroup",
      "secondaryEndpoints": null,
      "secondaryLocation": null,
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "statusOfPrimary": "Available",
      "statusOfSecondary": null,
      "tags": {},
      "type": "Microsoft.Storage/storageAccounts"
    }

若要使用 CLI 检查存储帐户，首先需要设置帐户名和密钥。使用 [az storage account show-connection-string](https://docs.microsoft.com/cli/azure/storage/account#show-connection-string)。将下例中的存储帐户名替换为所选的名称：

    export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string --resource-group myResourceGroup --name mystorageaccount --query connectionString)"

然后，可以使用 [az storage container list](https://docs.microsoft.com/cli/azure/storage/container#list) 查看存储信息：

    az storage container list

输出：

    [
      {
        "metadata": null,
        "name": "vhds",
        "properties": {
          "etag": "\"0x8D41F912D472F94\"",
          "lastModified": "2016-12-08T17:39:35+00:00",
          "lease": {
            "duration": null,
            "state": null,
            "status": null
          },
          "leaseDuration": "infinite",
          "leaseState": "leased",
          "leaseStatus": "locked"
        }
      }
    ]

## <a name="create-a-virtual-network-and-subnet"></a> 创建虚拟网络和子网
接下来，需要创建在 Azure 中运行的虚拟网络，以及可在其中创建 VM 的子网。以下示例使用 [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#create) 创建一个名为 `myVnet`、地址前缀为 `192.168.0.0/16` 的虚拟网络和一个名为 `mySubnet`、子网地址前缀为 `192.168.1.0/24` 的子网：

    az network vnet create --resource-group myResourceGroup --location chinanorth \
      --name myVnet --address-prefix 192.168.0.0/16 \
      --subnet-name mySubnet --subnet-prefix 192.168.1.0/24

输出将该子网显示为在虚拟网络内部逻辑创建：

    {
      "addressSpace": {
        "addressPrefixes": [
          "192.168.0.0/16"
        ]
      },
      "dhcpOptions": {
        "dnsServers": []
      },
      "etag": "W/\"e95496fc-f417-426e-a4d8-c9e4d27fc2ee\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet",
      "location": "chinanorth",
      "name": "myVnet",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "resourceGuid": "ed62fd03-e9de-430b-84df-8a3b87cacdbb",
      "subnets": [
        {
          "addressPrefix": "192.168.1.0/24",
          "etag": "W/\"e95496fc-f417-426e-a4d8-c9e4d27fc2ee\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet",
          "ipConfigurations": null,
          "name": "mySubnet",
          "networkSecurityGroup": null,
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "resourceNavigationLinks": null,
          "routeTable": null
        }
      ],
      "tags": {},
      "type": "Microsoft.Network/virtualNetworks",
      "virtualNetworkPeerings": null
    }

## 创建公共 IP 地址
现在，需要创建分配给负载均衡器的公共 IP 地址 (PIP)。使用该地址可以通过 [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#create) 命令从 Internet 连接到 VM。由于默认地址是动态的，因此可使用 `--domain-name-label` 选项在 **chinacloudapp.cn** 域中创建一个命名的 DNS 条目。以下示例创建名为 `myPublicIP`、DNS 名称为 `mypublicdns` 的公共 IP。由于 DNS 名称必须唯一，因此，请提供自己的唯一 DNS 名称：

    az network public-ip create --resource-group myResourceGroup --location chinanorth \
      --name myPublicIP --dns-name mypublicdns --allocation-method static --idle-timeout 4

输出：

    {
      "publicIp": {
        "dnsSettings": {
          "domainNameLabel": "mypublicdns",
          "fqdn": "mypublicdns.chinanorth.chinacloudapp.cn",
          "reverseFqdn": ""
        },
        "idleTimeoutInMinutes": 4,
        "ipAddress": "52.174.48.109",
        "provisioningState": "Succeeded",
        "publicIPAllocationMethod": "Static",
        "resourceGuid": "78218510-ea54-42d7-b2c2-44773fc14af5"
      }
    }

公共 IP 地址资源已经以逻辑方式分配，但尚未分配有特定地址。若要获取 IP 地址，需要一个负载均衡器，但该负载均衡器尚未创建。

## 创建负载均衡器和 IP 池
创建负载均衡器时，可以将流量分散到多个 VM。负载均衡器还可以在执行维护或承受重负载时运行多个 VM 来响应用户请求，为应用程序提供冗余。以下示例使用 [az network lb create](https://docs.microsoft.com/cli/azure/network/lb#create) 创建一个名为 `myLoadBalancer` 的负载均衡器、一个名为 `myFrontEndPool` 的前端 IP 池，并挂接 `myPublicIP` 资源：

    az network lb create --resource-group myResourceGroup --location chinanorth \
      --name myLoadBalancer --public-ip-address myPublicIP --frontend-ip-name myFrontEndPool

输出：

    {
      "loadBalancer": {
        "backendAddressPools": [
          {
            "etag": "W/\"7a05df7a-5ee2-4581-b411-4b6f048ab291\"",
            "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myLoadBalancerbepool",
            "name": "myLoadBalancerbepool",
            "properties": {
              "provisioningState": "Succeeded"
            },
            "resourceGroup": "myResourceGroup"
          }
        ],
        "frontendIPConfigurations": [
          {
            "etag": "W/\"7a05df7a-5ee2-4581-b411-4b6f048ab291\"",
            "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/frontendIPConfigurations/myFrontEndPool",
            "name": "myFrontEndPool",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "provisioningState": "Succeeded",
              "publicIPAddress": {
                "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
                "resourceGroup": "myResourceGroup"
              }
            },
            "resourceGroup": "myResourceGroup"
          }
        ],
        "inboundNatPools": [],
        "inboundNatRules": [],
        "loadBalancingRules": [],
        "outboundNatRules": [],
        "probes": [],
        "provisioningState": "Succeeded",
        "resourceGuid": "f4879d84-5ae8-4e06-8b9b-1419baa875d9"
      }
    }

请注意如何使用 `--public-ip-address` 开关传入前面创建的 `myPublicIP`。通过将公共 IP 地址分配给负载均衡器，可以通过 Internet 访问 VM。

我们使用后端池作为 VM 要连接到的位置。这样，流量便可以通过负载均衡器流向 VM。让我们使用 [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool#create) 创建后端流量的 IP 池。以下示例创建名为 `myBackEndPool` 的后端池：

    az network lb address-pool create --resource-group myResourceGroup \
      --lb-name myLoadBalancer --name myBackEndPool

删节输出：

      "backendAddressPools": [
        {
          "backendIpConfigurations": null,
          "etag": "W/\"a61814bc-ce4c-4fb2-9d6a-5b1949078345\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myLoadBalancerbepool",
          "loadBalancingRules": null,
          "name": "myLoadBalancerbepool",
          "outboundNatRule": null,
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup"
        },
        {
          "backendIpConfigurations": null,
          "etag": "W/\"a61814bc-ce4c-4fb2-9d6a-5b1949078345\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool",
          "loadBalancingRules": null,
          "name": "myBackEndPool",
          "outboundNatRule": null,
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup"
        }
      ],
      "etag": "W/\"a61814bc-ce4c-4fb2-9d6a-5b1949078345\"",

## 创建负载均衡器 NAT 规则
若要获取流经负载均衡器的流量，需要创建网络地址转换 (NAT) 规则来指定入站或出站操作。可以指定要使用的协议，然后根据需要将外部端口映射到内部端口。针对我们的环境，让我们使用 [az network lb inbound-nat-rule create](https://docs.microsoft.com/cli/azure/network/lb/inbound-nat-rule#create) 创建一些规则，以允许通过负载均衡器对 VM 进行 SSH 访问。将 TCP 端口 4222 和 4223 设置为定向到 VM 上的 TCP 端口 22（稍后将会创建）。以下示例创建名为 `myLoadBalancerRuleSSH1` 的规则，用于将 TCP 端口 4222 映射到端口 22：

    az network lb inbound-nat-rule create --resource-group myResourceGroup \
      --lb-name myLoadBalancer --name myLoadBalancerRuleSSH1 --protocol tcp \
      --frontend-port 4222 --backend-port 22 --frontend-ip-name myFrontEndPool

输出：

    {
      "backendIpConfiguration": null,
      "backendPort": 22,
      "enableFloatingIp": false,
      "etag": "W/\"72843cf8-b5fb-4655-9ac8-9cbbbbf1205a\"",
      "frontendIpConfiguration": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/frontendIPConfigurations/myFrontEndPool",
        "resourceGroup": "myResourceGroup"
      },
      "frontendPort": 4222,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH1",
      "idleTimeoutInMinutes": 4,
      "name": "myLoadBalancerRuleSSH1",
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup"
    }

对于第二个 NAT 规则中的 SSH 访问，重复该过程。以下示例创建名为 `myLoadBalancerRuleSSH2` 的规则，用于将 TCP 端口 4223 映射到端口 22：

    az network lb inbound-nat-rule create --resource-group myResourceGroup \
      --lb-name myLoadBalancer --name myLoadBalancerRuleSSH2 --protocol tcp \
      --frontend-port 4223 --backend-port 22 --frontend-ip-name myFrontEndPool

## 创建负载均衡器运行状况探测
运行状况探测定期检查受负载均衡器后面的 VM，以确保它们可以根据定义操作和响应请求。否则，将从操作中删除这些 VM，确保不会将用户定向到它们。可以针对运行状况探测定义自定义检查，以及间隔和超时值。有关运行状况探测的详细信息，请参阅 [Load Balancer probes](/documentation/articles/load-balancer-custom-probe-overview/)（负载均衡器探测）。以下示例使用 [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe#create) 创建名为 `myHealthProbe` 的 TCP 运行状况探测：

    az network lb probe create --resource-group myResourceGroup --lb-name myLoadBalancer \
      --name myHealthProbe --protocol tcp --port 80 --interval 15 --threshold 4

输出：

    {
      "etag": "W/\"757018f6-b70a-4651-b717-48b511d82ba0\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/probes/myHealthProbe",
      "intervalInSeconds": 15,
      "loadBalancingRules": null,
      "name": "myHealthProbe",
      "numberOfProbes": 4,
      "port": 80,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "requestPath": null,
      "resourceGroup": "myResourceGroup"
    }

此处我们指定了 15 秒的运行状况检查间隔。在负载均衡器将该主机视为不再正常运行之前，我们最多可能会错过四个探测（1 分钟）。

让我们继续为用于传输 Web 流量的 TCP 端口 80 创建 NAT 规则，并将该规则挂接到 IP 池。如果将规则挂接到 IP 池，而不是将规则逐个挂接到 VM，则可以在 IP 池中添加或删除 VM。然后，负载均衡器会自动调整流量流。以下示例使用 [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule#create) 创建一个名为 `myLoadBalancerRuleWeb` 的规则，将 TCP 端口 80 映射到端口 80，并挂接名为 `myHealthProbe` 的运行状况探测：

    az network lb rule create --resource-group myResourceGroup --lb-name myLoadBalancer \
      --name myLoadBalancerRuleWeb --protocol tcp --frontend-port 80 --backend-port 80 \
      --frontend-ip-name myFrontEndPool --backend-pool-name myBackEndPool \
      --probe-name myHealthProbe

输出：

    {
      "backendAddressPool": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool",
        "resourceGroup": "myResourceGroup"
      },
      "backendPort": 80,
      "enableFloatingIp": false,
      "etag": "W/\"f0d77680-bf42-4d11-bfab-5d2c541bee56\"",
      "frontendIpConfiguration": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/frontendIPConfigurations/myFrontEndPool",
        "resourceGroup": "myResourceGroup"
      },
      "frontendPort": 80,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/loadBalancingRules/myLoadBalancerRuleWeb",
      "idleTimeoutInMinutes": 4,
      "loadDistribution": "Default",
      "name": "myLoadBalancerRuleWeb",
      "probe": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/probes/myHealthProbe",
        "resourceGroup": "myResourceGroup"
      },
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup"
    }

## 验证负载均衡器
现已完成负载均衡器配置。以下是执行的步骤：

1. 创建负载均衡器。
2. 创建前端 IP 池并为其分配公共 IP。
3. 创建 VM 可以连接到的后端 IP 池。
4. 创建允许通过 SSH 连接到 VM 以进行管理的 NAT 规则，以及允许对 Web 应用使用 TCP 端口 80 的规则。
5. 添加一个运行状况探测来定期检查 VM。此运行状况探测可以确保用户不会尝试访问不再正常运行和不再提供内容的 VM。

让我们使用 [az network lb show](https://docs.microsoft.com/cli/azure/network/lb#show) 查看负载均衡器现在的情形：

    az network lb show --resource-group myResourceGroup --name myLoadBalancer

输出：

    {
      "backendAddressPools": [
        {
          "backendIpConfigurations": null,
          "etag": "W/\"a0e313a1-6de1-48be-aeb6-df57188d708c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myLoadBalancerbepool",
          "loadBalancingRules": null,
          "name": "myLoadBalancerbepool",
          "outboundNatRule": null,
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup"
        },
        {
          "backendIpConfigurations": null,
          "etag": "W/\"a0e313a1-6de1-48be-aeb6-df57188d708c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool",
          "loadBalancingRules": null,
          "name": "myBackEndPool",
          "outboundNatRule": null,
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup"
        }
      ],
      "etag": "W/\"a0e313a1-6de1-48be-aeb6-df57188d708c\"",
      "frontendIpConfigurations": [
        {
          "etag": "W/\"a0e313a1-6de1-48be-aeb6-df57188d708c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/frontendIPConfigurations/LoadBalancerFrontEnd",
          "inboundNatPools": null,
          "inboundNatRules": null,
          "loadBalancingRules": null,
          "name": "LoadBalancerFrontEnd",
          "outboundNatRules": null,
          "privateIpAddress": null,
          "privateIpAllocationMethod": "Dynamic",
          "provisioningState": "Succeeded",
          "publicIpAddress": {
            "dnsSettings": null,
            "etag": null,
            "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/PublicIPmyLoadBalancer",
            "idleTimeoutInMinutes": null,
            "ipAddress": null,
            "ipConfiguration": null,
            "location": null,
            "name": null,
            "provisioningState": null,
            "publicIpAddressVersion": null,
            "publicIpAllocationMethod": null,
            "resourceGroup": "myResourceGroup",
            "resourceGuid": null,
            "tags": null,
            "type": null
          },
          "resourceGroup": "myResourceGroup",
          "subnet": null
        },
        {
          "etag": "W/\"a0e313a1-6de1-48be-aeb6-df57188d708c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/frontendIPConfigurations/myFrontEndPool",
          "inboundNatPools": null,
          "inboundNatRules": null,
          "loadBalancingRules": null,
          "name": "myFrontEndPool",
          "outboundNatRules": null,
          "privateIpAddress": null,
          "privateIpAllocationMethod": "Dynamic",
          "provisioningState": "Succeeded",
          "publicIpAddress": {
            "dnsSettings": null,
            "etag": null,
            "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
            "idleTimeoutInMinutes": null,
            "ipAddress": null,
            "ipConfiguration": null,
            "location": null,
            "name": null,
            "provisioningState": null,
            "publicIpAddressVersion": null,
            "publicIpAllocationMethod": null,
            "resourceGroup": "myResourceGroup",
            "resourceGuid": null,
            "tags": null,
            "type": null
          },
          "resourceGroup": "myResourceGroup",
          "subnet": null
        }
      ],
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer",
      "inboundNatPools": [],
      "inboundNatRules": [],
      "loadBalancingRules": [],
      "location": "chinanorth",
      "name": "myLoadBalancer",
      "outboundNatRules": [],
      "probes": [],
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "resourceGuid": "b5815801-b53d-4c22-aafc-2a9064f90f3c",
      "tags": {},
      "type": "Microsoft.Network/loadBalancers"
    }

## 创建网络安全组和规则
现在，我们创建网络安全组和用于控制 NIC 访问权限的入站规则。可将网络安全组应用到 NIC 或子网。定义用于控制传入和传出 VM 的流量流的规则。以下示例使用 [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg#create) 创建名为 `myNetworkSecurityGroup` 的网络安全组：

    az network nsg create --resource-group myResourceGroup --location chinanorth \
      --name myNetworkSecurityGroup

使用 [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule#create) 添加 NSG 的入站规则，允许端口 22 上的入站连接（以支持 SSH）。以下示例创建名为 `myNetworkSecurityGroupRuleSSH` 的规则，以便在 端口 22 上允许 TCP：

    az network nsg rule create --resource-group myResourceGroup \
      --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRuleSSH \
      --protocol tcp --direction inbound --priority 1000 \
      --source-address-prefix '*' --source-port-range '*' \
      --destination-address-prefix '*' --destination-port-range 22 --access allow

现在，添加 NSG 的入站规则，允许端口 80 上的入站连接（以支持 Web 流量）。以下示例创建名为 `myNetworkSecurityGroupRuleHTTP` 的规则，以便在 端口 80 上允许 TCP：

    az network nsg rule create --resource-group myResourceGroup \
      --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRuleHTTP \
      --protocol tcp --direction inbound --priority 1001 \
      --source-address-prefix '*' --source-port-range '*' \
      --destination-address-prefix '*' --destination-port-range 80 --access allow

> [AZURE.NOTE]
入站规则是入站网络连接的筛选器。在本示例中，我们将 NSG 绑定到 VM 虚拟 NIC，这意味着任何对端口 22 的请求都将在 VM 上传递到 NIC。此入站规则与网络连接相关，而不与终结点相关（终结点与经典部署相关）。若要打开端口，必须将 `--source-port-range` 保持设置为“*”（默认值）才能接受来自**任何**请求端口的入站请求。端口通常是动态的。

使用 [az network nsg show](https://docs.microsoft.com/cli/azure/network/nsg#show) 检查网络安全组和规则：

    az network nsg show --resource-group myResourceGroup --name myNetworkSecurityGroup

输出：

    {
      "defaultSecurityRules": [
        {
          "access": "Allow",
          "description": "Allow inbound traffic from all VMs in VNET",
          "destinationAddressPrefix": "VirtualNetwork",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowVnetInBound",
          "name": "AllowVnetInBound",
          "priority": 65000,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "sourceAddressPrefix": "VirtualNetwork",
          "sourcePortRange": "*"
        },
        {
          "access": "Allow",
          "description": "Allow inbound traffic from azure load balancer",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowAzureLoadBalancerInBound",
          "name": "AllowAzureLoadBalancerInBound",
          "priority": 65001,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "sourceAddressPrefix": "AzureLoadBalancer",
          "sourcePortRange": "*"
        },
        {
          "access": "Deny",
          "description": "Deny all inbound traffic",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/DenyAllInBound",
          "name": "DenyAllInBound",
          "priority": 65500,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        },
        {
          "access": "Allow",
          "description": "Allow outbound traffic from all VMs to all VMs in VNET",
          "destinationAddressPrefix": "VirtualNetwork",
          "destinationPortRange": "*",
          "direction": "Outbound",
          "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowVnetOutBound",
          "name": "AllowVnetOutBound",
          "priority": 65000,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "sourceAddressPrefix": "VirtualNetwork",
          "sourcePortRange": "*"
        },
        {
          "access": "Allow",
          "description": "Allow outbound traffic from all VMs to Internet",
          "destinationAddressPrefix": "Internet",
          "destinationPortRange": "*",
          "direction": "Outbound",
          "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowInternetOutBound",
          "name": "AllowInternetOutBound",
          "priority": 65001,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        },
        {
          "access": "Deny",
          "description": "Deny all outbound traffic",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Outbound",
          "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/DenyAllOutBound",
          "name": "DenyAllOutBound",
          "priority": 65500,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        }
      ],
      "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup",
      "location": "chinanorth",
      "name": "myNetworkSecurityGroup",
      "networkInterfaces": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "resourceGuid": "79c2c293-ee3e-4616-bf9c-4741ff1f708a",
      "securityRules": [
        {
          "access": "Allow",
          "description": null,
          "destinationAddressPrefix": "*",
          "destinationPortRange": "22",
          "direction": "Inbound",
          "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/myNetworkSecurityGroupRuleSSH",
          "name": "myNetworkSecurityGroupRuleSSH",
          "priority": 1000,
          "protocol": "tcp",
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        },
        {
          "access": "Allow",
          "description": null,
          "destinationAddressPrefix": "*",
          "destinationPortRange": "80",
          "direction": "Inbound",
          "etag": "W/\"2b656589-f332-4ba4-92f3-afb3be5c192c\"",
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/myNetworkSecurityGroupRuleHTTP",
          "name": "myNetworkSecurityGroupRuleHTTP",
          "priority": 1001,
          "protocol": "tcp",
          "provisioningState": "Succeeded",
          "resourceGroup": "myResourceGroup",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        }
      ],
      "subnets": null,
      "tags": {},
      "type": "Microsoft.Network/networkSecurityGroups"
    }

## 创建用于 Linux VM 的 NIC
由于可以对 NIC 使用应用规则，因此能以编程方式使用 NIC。可以创建多个规则。在下面的 [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#create) 命令中，要将 NIC 挂接到负载后端 IP 池，并将其与 NAT 规则关联以允许 SSH 流量和网络安全组。

以下示例创建名为 `myNic1` 的 NIC：

    az network nic create --resource-group myResourceGroup --location chinanorth --name myNic1 \
      --vnet-name myVnet --subnet mySubnet --network-security-group myNetworkSecurityGroup \
      --lb-name myLoadBalancer --lb-address-pools myBackEndPool \
      --lb-inbound-nat-rules myLoadBalancerRuleSSH1

输出：

    {
      "newNIC": {
        "dnsSettings": {
          "appliedDnsServers": [],
          "dnsServers": []
        },
        "enableIPForwarding": false,
        "ipConfigurations": [
          {
            "etag": "W/"a76b5c0d-14e1-4a99-afd4-5cd8ac0465ca"",
            "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic1/ipConfigurations/ipconfig1",
            "name": "ipconfig1",
            "properties": {
              "loadBalancerBackendAddressPools": [
                {
                  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool",
                  "resourceGroup": "myResourceGroup"
                }
              ],
              "loadBalancerInboundNatRules": [
                {
                  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH1",
                  "resourceGroup": "myResourceGroup"
                }
              ],
              "primary": true,
              "privateIPAddress": "192.168.1.4",
              "privateIPAllocationMethod": "Dynamic",
              "provisioningState": "Succeeded",
              "subnet": {
                "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet",
                "resourceGroup": "myResourceGroup"
              }
            },
            "resourceGroup": "myResourceGroup"
          }
        ],
        "networkSecurityGroup": {
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup",
          "resourceGroup": "myResourceGroup"
        },
        "provisioningState": "Succeeded",
        "resourceGuid": "838977cb-cf4b-4c4a-9a32-0ddec7dc818b"
      }
    }

现在，我们将创建第二个 NIC 并同样将其挂接到后端 IP 池。这一次，第二个 NAT 规则将允许 SSH 流。以下示例创建名为 `myNic2` 的 NIC：

    az network nic create --resource-group myResourceGroup --location chinanorth --name myNic2 \
      --vnet-name myVnet --subnet mySubnet --network-security-group myNetworkSecurityGroup \
      --lb-name myLoadBalancer --lb-address-pools myBackEndPool \
      --lb-inbound-nat-rules myLoadBalancerRuleSSH2

## 创建可用性集
可用性集有助于将 VM 分散到容错域和升级域。让我们使用 [az vm availability-set create](https://docs.microsoft.com/cli/azure/vm/availability-set#create) 创建 VM 的可用性集。以下示例创建名为 `myAvailabilitySet` 的可用性集：

    az vm availability-set create --resource-group myResourceGroup --location chinanorth \
      --name myAvailabilitySet

容错域定义共享通用电源和网络交换机的一组虚拟机。默认情况下，在可用性集中配置的虚拟机隔离在最多三个容错域中。思路是其中一个容错域中的硬件问题不会影响运行应用的每个 VM。将多个 VM 放入一个可用性集时，Azure 会自动将它们分散到容错域。

升级域表示虚拟机组以及可同时重新启动的基础物理硬件。在计划内维护期间，升级域的重新启动顺序可能不会按序进行，但一次只重新启动一个升级域。同样，将多个 VM 放入一个可用性站点时，Azure 会自动将它们分散到升级域。

有关详细信息，请阅读[管理 VM 可用性](/documentation/articles/virtual-machines-linux-manage-availability/)。

## 创建 Linux VM
现已创建用于支持可访问 Internet 的 VM 的网络资源。接下来，我们创建 VM 并使用不含密码的 SSH 密钥保护其安全。在此情况下，我们需要基于最新的 LTS 创建 Ubuntu VM。我们将根据 [finding Azure VM images](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)（查找 Azure VM 映像）中所述，使用 [az vm image list](https://docs.microsoft.com/cli/azure/vm/image#list) 来查找该映像信息。

我们还指定要用于身份验证的 SSH 密钥。如果没有任何 SSH 密钥，可以按照[这些说明](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)创建 SSH 密钥。或者，可以在创建 VM 之后，使用 `--admin-password` 方法对 SSH 连接进行身份验证。此方法通常不太安全。

使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 命令并结合所有资源和信息来创建 VM：以下示例使用 Azure 非托管磁盘创建名为 `myVM1` 的 VM。

    az vm create \
        --resource-group myResourceGroup \
        --name myVM1 \
        --location chinanorth \
        --availability-set myAvailabilitySet \
        --nics myNic1 \
        --vnet myVnet \
        --subnet-name mySubnet \
        --nsg myNetworkSecurityGroup \
        --image UbuntuLTS \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --admin-username azureuser \
        --use-unmanaged-disk \
        --storage-account mystorageaccount

输出：

    {
      "fqdn": "",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM1",
      "macAddress": "",
      "privateIpAddress": "",
      "publicIpAddress": "",
      "resourceGroup": "myResourceGroup"
    }

可以使用默认的 SSH 密钥立即连接到 VM。请确保指定适当的端口，因为我们要通过负载均衡器传递流量。（对于第一个 VM，设置 NAT 规则以将端口 4222 转发到 VM。）

    ssh ops@mypublicdns.chinanorth.chinacloudapp.cn -p 4222 -i ~/.ssh/id_rsa.pub

输出：

    The authenticity of host '[mypublicdns.chinanorth.chinacloudapp.cn]:4222 ([xx.xx.xx.xx]:4222)' can't be established.
    ECDSA key fingerprint is 94:2d:d0:ce:6b:fb:7f:ad:5b:3c:78:93:75:82:12:f9.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '[mypublicdns.chinanorth.chinacloudapp.cn]:4222,[xx.xx.xx.xx]:4222' (ECDSA) to the list of known hosts.
    Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-34-generic x86_64)

     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage

      Get cloud support with Ubuntu Advantage Cloud Guest:
        http://www.ubuntu.com/business/services/cloud

    0 packages can be updated.
    0 updates are security updates.

    ops@myVM1:~$

以相同的方式继续创建第二个 VM：

    az vm create \
        --resource-group myResourceGroup \
        --name myVM2 \
        --location chinanorth \
        --availability-set myAvailabilitySet \
        --nics myNic2 \
        --vnet myVnet \
        --subnet-name mySubnet \
        --nsg myNetworkSecurityGroup \
        --image UbuntuLTS \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --admin-username azureuser \
        --use-unmanaged-disk \
        --storage-account mystorageaccount

此时，已在 Azure 中运行了一个位于负载均衡器后面的 Ubuntu VM，只能使用 SSH 密钥对登录到该 VM（因为密码已禁用）。可以安装 nginx 或 httpd、部署 Web 应用，以及查看流量是否通过负载均衡器流向两个 VM。

## 将环境导出为模板
现已构建此环境，如果要使用相同的参数创建与其相符的额外开发环境或生产环境，该怎么办？ Resource Manager 使用定义了所有环境参数的 JSON 模板。通过引用此 JSON 模板构建出整个环境。可以[手动构建 JSON 模板](/documentation/articles/resource-group-authoring-templates/)，也可以通过导出现有环境来为自己创建 JSON 模板。使用 [az group export](https://docs.microsoft.com/cli/azure/group#export) 导出资源组，如下所示：

    az group export --name myResourceGroup > myResourceGroup.json

此命令在当前工作目录中创建 `myResourceGroup.json` 文件。从此模板创建环境时，系统会提示输入所有资源名称，包括负载均衡器、网络接口或 VM 的名称。可以通过在前面所示的 **az group export** 命令中添加 `--include-parameter-default-value` 参数，在模板文件中填充这些名称。请编辑 JSON 模板以指定资源名称，或[创建 parameters.json 文件](/documentation/articles/resource-group-authoring-templates/)来指定资源名称。

若要从模板创建环境，请使用 [az group deployment create](https://docs.microsoft.com/cli/azure/group/deployment#create)，如下所示：

    az group deployment create --resource-group myNewResourceGroup \
      --template-file myResourceGroup.json

可能需要阅读[有关通过模板进行部署的详细信息](/documentation/articles/resource-group-template-deploy-cli/)。了解如何对环境进行增量更新、如何使用参数文件，以及如何从单个存储位置访问模板。

## 后续步骤
现在，已准备好开始使用多个网络组件和 VM。可以使用本文介绍的核心组件，通过此示例环境构建应用程序。

<!---HONumber=Mooncake_0320_2017-->