<properties
   pageTitle="使用 Azure CLI 创建多 VM 部署 | Windows Azure"
   description="了解如何使用经典部署模型和 Azure CLI 创建多 VM 部署。"
   services="virtual-machines"
   documentationCenter="nodejs"
   authors="AlanSt"
   manager="timlt"
   editor=""
   tags="azure-service-management"/>

   <tags
   ms.service="virtual-machines"
   ms.date="02/20/2015"
   wacn.date="02/17/2016"/>

# 使用 Azure CLI 创建多 VM 部署

> [AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]

下面的脚本将向你展示如何使用 Azure 命令行界面 (Azure CLI) 在 VNET 中配置多 VM 多云服务部署。

下图说明了在该脚本完成后你的部署的外观：

![](./media/virtual-machines-create-multi-vm-deployment-xplat-cli/multi-vm-xplat-cli.png)

该脚本在云服务 **servercs** 中创建一个 VM (**servervm**) 并附加两个数据磁盘，在云服务 **workercs** 中创建两个 VM（**clientvm1、clientvm2**）。两个云服务都放置在 VNET **samplevnet** 中。**servercs** 云服务还配置了一个用于外部连接的终结点。

## 用于实现此目标的 CLI 脚本
用于实现此目标的代码比较简单明了：

>[AZURE.NOTE]你可能需要将云服务名称 servercs 和 workercs 更改为唯一的云服务名称

    azure network vnet create samplevnet -l "China North"
    azure vm create -l "China North" -w samplevnet -e 10000 -z Small -n servervm servercs b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_10-amd64-server-20150202-zh-CN-30GB azureuser Password@1
    azure vm create -l "China North" -w samplevnet -e 10001 -z Small –n clientvm1 clientcs b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_10-amd64-server-20150202-zh-CN-30GB azureuser Password@1
    azure vm create -l "China North" -w samplevnet -e 10002 -c -z Small -n clientvm2 clientcs b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_10-amd64-server-20150202-zh-CN-30GB azureuser Password@1
    azure vm disk attach-new servervm 100
    azure vm disk attach-new servervm 500
    azure vm endpoint create servervm 443 443 -n https -o tcp

下面是用于拆除它的代码：

    azure vm delete -b -q servervm
    azure vm delete -b -q clientvm1
    azure vm delete -b -q clientvm2
    azure network vnet delete -q samplevnet

*–q 选项禁止以交互方式确认删除对象操作，-b 清除与 VM 关联的磁盘 / blob。*

## 使用的命令的一般形式

虽然你可以在任何 Azure CLI 命令中使用 –help 选项来了解更多信息，但是上面使用的每个命令的一般形式如下所示：

    azure network vnet create -l <Region> <VNet_name>
    azure network vnet delete -q <VNet_name>

    azure vm create -l <Region> -w <Vnet_name> -e <SSH_port> -z <VM_size> -n <VM_name> <Cloud_service_name> <VM_image> <Username> <Password>
    azure vm delete -b -q <VM_name>
    azure vm disk attach-new <VM_name>
    azure vm endpoint create <VM_name> <External_port> <Internal_port> -n <Endpoint_name> -o <TCP/UDP>

## 后续步骤


* [Azure 上的 Linux 和开源计算](/documentation/articles/virtual-machines-linux-opensource)
* [如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)
 

<!---HONumber=79-->