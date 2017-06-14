<properties
    pageTitle="使用虚拟机规模集在 Azure 中创建高度可用的应用 | Azure"
    description="使用虚拟机规模集和 Azure CLI 在 Linux VM 上创建和部署高度可用的应用程序。"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="Thraka"
    manager="timlt"
    editor=""
    tags="" />
<tags
    ms.assetid=""
    ms.service="virtual-machine-scale-sets"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="na"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.date="04/05/2017"
    wacn.date="05/15/2017"
    ms.author="adegeo"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="89524108d60c46a8325cc89badc0667d76d80c3d"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-highly-available-application-on-linux-with-virtual-machine-scale-sets"></a>使用虚拟机规模集在 Linux 上创建高度可用的应用程序
本教程演示如何在虚拟机规模集中创建高度可用的应用程序。 还介绍如何在规模集中自动进行虚拟机配置。 

## <a name="step-1---create-a-resource-group"></a>步骤 1 - 创建资源组
若要完成本教程，请确保已安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。 如果尚未登录到 Azure 订阅，请使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录，然后按照屏幕上的说明进行操作。

使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建资源组。 以下示例在 `chinanorth` 位置创建名为 `myResourceGroupVMSS` 的资源组：

    az group create --name myResourceGroupVMSS --location chinanorth

## <a name="step-2---define-your-app"></a>步骤 2 - 定义应用程序
使用创建高度可用、负载均衡的应用程序时所用教程中的同一 **cloud-init** 配置。 有关使用 **cloud-init** 的详细信息，请参阅[在创建期间使用 cloud-init 自定义 Linux VM](/documentation/articles/virtual-machines-linux-using-cloud-init/)。

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

## <a name="step-3---create-scale-set"></a>第 3 步 - 创建规模集
利用虚拟机规模集，可以部署和管理一组相同的、自动缩放的虚拟机。 规模集使用的组件与教程中[在 Azure 上构建高度可用的应用程序](/documentation/articles/virtual-machines-linux-tutorial-load-balance-nodejs/)相关部分中介绍的相同。 这些组件包括可用性集、容错域和更新域以及负载均衡器。

规模集将自动为用户创建和管理这些资源。 可基于定义的规则，自动增加或减少规模集中 VM 的数目。 可[使用自定义映像](/documentation/articles/virtual-machines-linux-capture-image/)作为虚拟机的源，或者在部署过程中使用本教程中介绍的 **cloud-init** 来配置 VM。

使用 [az vmss create](https://docs.microsoft.com/zh-cn/cli/azure/vmss#create) 创建虚拟机规模集。 以下示例创建名为 `myScaleSet` 的规模集：

    az vmss create \
      --resource-group myResourceGroupVMSS \
      --name myScaleSet \
      --image Canonical:UbuntuServer:14.04.3-LTS:latest \
      --upgrade-policy-mode automatic \
      --custom-data cloud-init.txt \
      --admin-username azureuser \
      --use-unmanaged-disk \
      --generate-ssh-keys      

创建和配置所有的规模集资源和 VM 需要几分钟时间。

## <a name="step-4---configure-firewall"></a>步骤 4 - 配置防火墙
已自动创建一个负载均衡器，作为虚拟机规模集的一部分。 负载均衡器使用负载均衡器规则将流量分配到一组定义的 VM。 若要允许通信流到达 Web 应用，请使用 [az network lb probe create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/probe#create) 创建一个规则。 以下示例创建一个名为 `myLoadBalancerRuleWeb` 的规则：

    az network lb rule create \
      --resource-group myResourceGroupVMSS \
      --name myLoadBalancerRuleWeb \
      --lb-name myScaleSetLB \
      --backend-pool-name myScaleSetLBBEPool \
      --backend-port 80 \
      --frontend-ip-name loadBalancerFrontEnd \
      --frontend-port 80 \
      --protocol tcp

## <a name="step-5---test-your-app"></a>步骤 5 - 测试应用
使用 [az network public-ip show](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#show) 获取负载均衡器的公共 IP 地址。 以下示例获取创建为规模集一部分的 `myScaleSetLBPublicIP` 的 IP 地址：

    az network public-ip show \
        --resource-group myResourceGroupVMSS \
        --name myScaleSetLBPublicIP \
        --query [ipAddress] \
        --output tsv

将公共 IP 地址输入到 Web 浏览器中。 将显示应用，包括负载均衡器将流量分发到的 VM 的主机名：

![运行 Node.js 应用](./media/virtual-machines-linux-tutorial-load-balance-nodejs/running-nodejs-app.png)

强制刷新 Web 浏览器，以查看负载均衡器在规模集中运行应用的所有 VM 之间分配流量。

## <a name="step-6---management-tasks"></a>步骤 6 - 管理任务
在规模集的整个生命周期内，可能需要运行一个或多个管理任务。 此外，可能还需要创建自动执行各种生命周期任务的脚本。 Azure CLI 2.0 提供一种用于执行这些任务的快速方法。 以下是一些常见任务。

### <a name="increase-or-decrease-vm-instances"></a>增加或减少 VM 实例
可以使用 [az vmss scale](https://docs.microsoft.com/zh-cn/cli/azure/vmss#scale) 手动增加或减少规模集中虚拟机的数目。 以下示例将规模集中 VM 的数目增加到 `5`：

    az vmss scale --resource-group myResourceGroupVMSS --name myScaleSet --new-capacity 5

利用自动缩放规则，可以定义如何根据网络流量或 CPU 使用率等需求，增加或减少规模集中 VM 的数目。 目前，不能在 Azure CLI 2.0 中设置这些规则。 使用 [Azure 门户](https://portal.azure.cn)配置自动缩放。

### <a name="get-connection-info"></a>获取连接信息
若要获取有关规模集中 VM 的连接信息，请使用 [az vmss list-instance-connection-info](https://docs.microsoft.com/zh-cn/cli/azure/vmss#list-instance-connection-info)。 此命令为每个允许采用 SSH 进行连接的 VM 输出公共 IP 地址和端口：

    az vmss list-instance-connection-info --resource-group myResourceGroupVMSS --name myScaleSet

### <a name="delete-resource-group"></a>删除资源组
删除资源组会删除其包含的所有资源。

    az group delete --name myResourceGroupVMSS

## <a name="next-steps"></a>后续步骤
在本教程中，使用 **cloud-init** 定义 Web 应用，并在部署过程中配置每个 VM。 有关捕获 VM 以将其用作规模集中的源映像的信息，请参阅[如何通用化和捕获 Linux 虚拟机](/documentation/articles/virtual-machines-linux-capture-image/)。

若要详细了解本教程中介绍的一些虚拟机规模集功能，请参阅以下信息：

- [Azure 虚拟机规模集概述](/documentation/articles/virtual-machine-scale-sets-overview/)
- [Azure 负载均衡器概述](/documentation/articles/load-balancer-overview/)
- [使用网络安全组控制网络流量](/documentation/articles/virtual-networks-nsg/)