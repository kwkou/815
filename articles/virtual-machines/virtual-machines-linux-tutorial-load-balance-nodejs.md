<properties
    pageTitle="教程 - 在 Azure VM 上生成高可用性应用程序 | Azure"
    description="了解如何在 Azure 中使用负载均衡器在 3 个 Linux VM 之间创建高可用性和安全性的应用程序"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="iainfoulds"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="03/27/2017"
    wacn.date="05/15/2017"
    ms.author="iainfou"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="65568a1b5caa611e224b92e47c980662679aed20"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="build-a-load-balanced-highly-available-application-on-linux-virtual-machines-in-azure"></a>在 Azure 中的 Linux 虚拟机上生成负载均衡的高可用性应用程序
在本教程中，你将创建高可用性应用程序，以适应维护事件。 该应用使用 1 个负载均衡器、1 个可用性集和 3 个 Linux 虚拟机 (VM)。 本教程将生成 1 个 Node.js 应用，尽管可以使用本教程通过相同的高可用性组件和准则部署其他应用程序框架。

## <a name="step-1---azure-prerequisites"></a>步骤 1 - Azure 先决条件
若要完成本教程，请打开终端窗口，并确保已安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。 如果尚未登录到 Azure 订阅，请使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录，然后按照屏幕上的说明进行操作：

    az login

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 需要使用 [](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建一个资源组才能创建任何其他 Azure 资源。 以下示例在 `chinanorth` 位置创建名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

## <a name="step-2---create-availability-set"></a>步骤 2 - 创建可用性集
可以在逻辑容错和更新域之间创建虚拟机。 每个逻辑域都是基础 Azure 数据中心中的硬件的一部分。 创建两个或多个 VM 时，将在这些域中分配计算和存储资源。 如果硬件组件正在进行维护，此分配将维持你的应用的可用性。 可用性集可用于定义这些逻辑容错和更新域。

使用 [az vm availability-set create](https://docs.microsoft.com/zh-cn/cli/azure/vm/availability-set#create) 创建可用性集。 以下示例创建一个名为 `myAvailabilitySet` 的可用性集：

    az vm availability-set create \
        --resource-group myResourceGroup \
        --name myAvailabilitySet \
        --platform-fault-domain-count 3 \
        --platform-update-domain-count 2

## <a name="step-3---create-load-balancer"></a>步骤 3 - 创建负载均衡器
Azure 负载均衡器使用负载均衡器规则将流量分配到一组定义的 VM。 运行状况探测器监视每个 VM 上的给定端口，并仅将流量分配给可操作的 VM。

### <a name="create-a-public-ip-address"></a>创建公共 IP 地址
若要在 Internet 上访问你的应用，请为负载均衡器分配公共 IP 地址。 使用 [az network public-ip create](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#create) 创建公共 IP 地址。 下例创建名为 `myPublicIP` 的公共 IP 地址：

    az network public-ip create --resource-group myResourceGroup --name myPublicIP

### <a name="create-a-load-balancer"></a>创建负载均衡器
使用 [az network lb create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb#create) 创建负载均衡器。 下例使用 `myPublicIP` 地址创建一个名为 `myLoadBalancer` 的负载均衡器：

    az network lb create \
        --resource-group myResourceGroup \
        --name myLoadBalancer \
        --frontend-ip-name myFrontEndPool \
        --backend-pool-name myBackEndPool \
        --public-ip-address myPublicIP

### <a name="create-a-health-probe"></a>创建运行状况探测器
若要允许负载均衡器监视应用的状态，可以使用运行状况探测器。 运行状况探测器基于其对运行状况检查的响应，从负载均衡器中动态添加或删除 VM。 默认情况下，在 15 秒时间间隔内发生两次连续的故障后，将从负载均衡器分布中删除 VM。 可以为应用创建基于协议或特定运行状况检查页面的运行状况探测器。 下例创建一个 TCP 探测器，尽管你可以在此示例中进行扩展，例如添加 `--path healthcheck.js`。 但必须创建 `healthcheck.js` 并为负载均衡器返回 **HTTP 200 OK** 响应，以保持主机的旋转。

使用 [az network lb probe create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/probe#create) 创建运行状况探测器。 下例创建一个 `myHealthProbe` 运行状况探测器，用于监视每个 VM：

    az network lb probe create \
        --resource-group myResourceGroup \
        --lb-name myLoadBalancer \
        --name myHealthProbe \
        --protocol tcp \
        --port 80

### <a name="create-a-load-balancer-rule"></a>创建负载均衡器规则
负载均衡器规则用于定义将流量分配给 VM 的方式。

使用 [az network lb rule create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/rule#create) 创建负载均衡器规则。 以下示例创建一个名为 `myLoadBalancerRule` 的规则、使用 `myHealthProbe` 运行状况探测器并平衡端口 `80` 上的流量：

    az network lb rule create \
        --resource-group myResourceGroup \
        --lb-name myLoadBalancer \
        --name myLoadBalancerRule \
        --protocol tcp \
        --frontend-port 80 \
        --backend-port 80 \
        --frontend-ip-name myFrontEndPool \
        --backend-pool-name myBackEndPool \
        --probe-name myHealthProbe

## <a name="step-4---configure-networking"></a>步骤 4 - 配置网络
每个 VM 都有一个或多个连接到某个虚拟网络的虚拟网络接口卡 (NIC)。 该虚拟网络基于定义的访问规则筛选流量，确保安全。

### <a name="create-a-virtual-network"></a>创建虚拟网络
若要为 VM 提供网络连接，请使用 [az network vnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#create) 创建虚拟网络。 以下示例创建一个名为 `myVnet` 的虚拟网络和名为 `mySubnet` 的子网：

    az network vnet create --resource-group myResourceGroup --name myVnet --subnet-name mySubnet

### <a name="configure-network-security"></a>配置网络安全
网络安全组使用规则控制与 VM 之间的来往通信流。 你可以定义这些规则，例如允许端口 80 上的流量。 当负载均衡器在 VM 之间分配流量时，创建网络安全组规则可确保仅允许许可的流量。

使用 [az network nsg create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg#create) 创建网络安全组。 以下示例创建名为 `myNetworkSecurityGroup` 的网络安全组：

    az network nsg create --resource-group myResourceGroup --name myNetworkSecurityGroup

若要允许 Web 流量到达你的应用，请使用 [az network nsg rule create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg/rule#create) 创建网络安全组规则。 以下示例创建一个名为 `myNetworkSecurityGroupRule` 的网络安全组规则：

    az network nsg rule create \
        --resource-group myResourceGroup \
        --nsg-name myNetworkSecurityGroup \
        --name myNetworkSecurityGroupRule \
        --priority 1001 \
        --protocol tcp \
        --destination-port-range 80

### <a name="create-virtual-network-interface-cards"></a>创建虚拟网络接口卡 
负载均衡器使用虚拟 NIC 资源（而不是实际的 VM）运行。 虚拟 NIC 连接到负载均衡器后会被附加到 VM。

使用 [az network nic create](https://docs.microsoft.com/zh-cn/cli/azure/network/nic#create) 创建虚拟 NIC。 以下示例创建三个虚拟 NIC。 （你在以下步骤中为应用创建的每个 VM 各使用一个虚拟 NIC）：

    for i in `seq 1 3`; do
        az network nic create \
            --resource-group myResourceGroup \
            --name myNic$i \
            --vnet-name myVnet \
            --subnet mySubnet \
            --network-security-group myNetworkSecurityGroup \
            --lb-name myLoadBalancer \
            --lb-address-pools myBackEndPool
    done

## <a name="step-5---build-your-app"></a>步骤 5 - 生成应用
**cloud-init** 是自定义 VM 的常用方法。 可使用 **cloud-init** 安装程序包和写入文件。 由于 **cloud-init** 在初始部署期间运行，因此没有其他步骤可让你的应用运行。 VM 完成部署且该应用运行后，负载均衡器将开始分配流量。 有关使用 **cloud-init** 的详细信息，请参阅[在创建期间使用 cloud-init 自定义 Linux VM](/documentation/articles/virtual-machines-linux-using-cloud-init/)。

以下 **cloud-init** 配置安装 **nodejs** 和 **npm**，然后安装并配置 **nginx** 作为你的应用的 Web 代理。 配置还会创建一个简单的“Hello World” Node.js 应用，然后使用 **Express** 初始化并启动应用。 如果想要使用其他应用程序框架，请相应地调整程序包和已部署的应用程序。

创建名为 `cloud-init.txt` 的文件并粘贴下面的配置：

    #cloud-config
    package_upgrade: true
    packages:
      - nginx
      - nodejs
      - npm
    write_files:
      - owner: www-data:www-data
      - path: /etc/nginx/sites-available/default
        content: |
          server {
            listen 80;
            location / {
              proxy_pass http://localhost:3000;
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection keep-alive;
              proxy_set_header Host $host;
              proxy_cache_bypass $http_upgrade;
            }
          }
      - owner: azureuser:azureuser
      - path: /home/azureuser/myapp/index.js
        content: |
          var express = require('express')
          var app = express()
          var os = require('os');
          app.get('/', function (req, res) {
            res.send('Hello World from host ' + os.hostname() + '!')
          })
          app.listen(3000, function () {
            console.log('Hello world app listening on port 3000!')
          })
    runcmd:
      - nginx -s reload
      - cd "/home/azureuser/myapp"
      - npm init
      - npm install express -y
      - nodejs index.js

## <a name="step-6---create-virtual-machines"></a>步骤 6 - 创建虚拟机
所有基础组件准备就绪后，现在可以创建高可用性 VM 以运行你的应用。

### <a name="create-vms"></a>创建 VM
使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建 VM。 以下示例创建三个 VM，并生成 SSH 密钥（如果它们尚不存在）：

    for i in `seq 1 3`; do
        az vm create \
            --resource-group myResourceGroup \
            --name myVM$i \
            --availability-set myAvailabilitySet \
            --nics myNic$i \
            --image Canonical:UbuntuServer:14.04.3-LTS:latest \
            --admin-username azureuser \
            --generate-ssh-keys \
            --custom-data cloud-init.txt \
            --use-unmanaged-disk \
            --no-wait
    done

创建和配置所有三个 VM 需要几分钟时间。 在每个 VM 上运行应用时，负载均衡器运行状况探测器会自动检测。 应用运行后，负载均衡器规则将开始分布流量。

### <a name="test-your-app"></a>测试应用
使用 [az network public-ip show](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#show) 获取负载均衡器的公共 IP 地址。 以下示例获取前面创建的 `myPublicIP` 的 IP 地址：

    az network public-ip show \
        --resource-group myResourceGroup \
        --name myPublicIP \
        --query [ipAddress] \
        --output tsv

将公共 IP 地址输入到 Web 浏览器中。 将显示应用，包括负载均衡器将流量分发到的 VM 的主机名：

![运行 Node.js 应用](./media/virtual-machines-linux-tutorial-load-balance-nodejs/running-nodejs-app.png)

强制刷新 Web 浏览器，以查看负载均衡器在运行应用的所有三个 VM 之间分配流量。

## <a name="step-7---management-tasks"></a>步骤 7 - 管理任务
建议对运行应用的 VM 执行维护，例如安装 OS 更新。 若要应对应用增加的流量，建议添加更多 VM。 本部分演示了如何在负载均衡器中删除或添加 VM。

### <a name="remove-a-vm-from-the-load-balancer"></a>从负载均衡器中删除 VM
使用 [az network nic ip-config address-pool remove](https://docs.microsoft.com/zh-cn/cli/azure/network/nic/ip-config/address-pool#remove) 从后端地址池中删除 VM。 以下示例从 `myLoadBalancer` 中删除 **myVM2** 的虚拟 NIC：

    az network nic ip-config address-pool remove \
        --resource-group myResourceGroup \
        --nic-name myNic2 \
        --ip-config-name ipConfig1 \
        --lb-name myLoadBalancer \
        --address-pool myBackEndPool 

强制刷新 Web 浏览器，以查看负载均衡器在运行应用的其余两个 VM 之间分配流量。 现在可以对 VM 执行维护，例如安装 OS 更新或执行 VM 重新启动。

### <a name="add-a-vm-to-the-load-balancer"></a>将 VM 添加到负载均衡器
执行 VM 维护后，或如果需要扩展容量，请使用 [az network nic ip-config address-pool add](https://docs.microsoft.com/zh-cn/cli/azure/network/nic/ip-config/address-pool#add) 将 VM 添加到后端地址池。 以下示例将 **myVM2** 的虚拟 NIC 添加到 `myLoadBalancer`：

    az network nic ip-config address-pool add \
        --resource-group myResourceGroup \
        --nic-name myNic2 \
        --ip-config-name ipConfig1 \
        --lb-name myLoadBalancer \
        --address-pool myBackEndPool

## <a name="next-steps"></a>后续步骤
本教程使用单个 Azure 资源构建高度可用的应用程序基础结构。 还可以使用虚拟机规模集自动增加或减少应用上运行的 VM 数量。

若要详细了解本教程中介绍的一些高可用性功能，请参阅以下信息：

- [管理 Linux 虚拟机的可用性](/documentation/articles/virtual-machines-windows-manage-availability/)
- [Azure 负载均衡器概述](/documentation/articles/load-balancer-overview/)
- [使用网络安全组控制网络流量](/documentation/articles/virtual-networks-nsg/)
- [Azure CLI 示例脚本](/documentation/articles/virtual-machines-windows-cli-samples/)