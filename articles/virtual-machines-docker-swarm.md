<properties
   pageTitle="在 Azure 上将 docker 与 swarm 一起使用入门"
   description="介绍如何使用 Docker VM 扩展创建一组 VM，以及如何使用 swarm 创建 Docker 群集。"
   services="virtual-machines"
   documentationCenter="virtual-machines-linux"
   authors="squillace"
   manager="timlt"
   editor="tysonn"
   tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="01/04/2016"
	wacn.date="02/26/2016"/>

# 如何将 docker 与 swarm 一起使用

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

本主题介绍非常简单的方法，以将 [docker](https://www.docker.com/) 与 [swarm](https://github.com/docker/swarm) 一起使用在 Azure 上创建 swarm 托管群集。它在 Azure 中创建四个虚拟机，其中一个虚拟机充当 swarm 管理器，而另外三个虚拟机则为 docker 主机群集的一部分。完成时，可以使用 swarm 查看群集，然后开始在其上使用 docker。此外，本主题中的 Azure CLI 调用将使用服务管理 (asm) 模式。

> [AZURE.NOTE] 本主题会将 docker 与 swarm 和 Azure CLI 一起使用，*而不*使用 **docker-machine** 以显示不同工具如何协同工作，但又保持独立。**docker-machine** 具有 **--swarm** 开关，让你能够使用 **docker-machine** 直接将节点添加到 swarm 中。有关示例，请参阅 [docker-machine](https://github.com/docker/machine) 文档。如果你错过了针对 Azure VM 运行的 **docker-machine**，请参阅[如何在 Azure 上使用 docker-machine](/documentation/articles/virtual-machines-linux-docker-machine/)。

## 使用 Azure 虚拟机创建 docker 主机

本主题将创建四个 VM，但你可以使用所需的任何数量。调用以下命令，并将 *&lt;password&gt;* 替换为你选择的密码。

    azure vm docker create swarm-master -l "China East" -e 22 $imagename ops <password>
    azure vm docker create swarm-node-1 -l "China East" -e 22 $imagename ops <password>
    azure vm docker create swarm-node-2 -l "China East" -e 22 $imagename ops <password>
    azure vm docker create swarm-node-3 -l "China East" -e 22 $imagename ops <password>

完成后，应能够使用 **azure vm list** 查看你的 Azure VM：

    $ azure vm list | grep "swarm-[mn]"
    data:    swarm-master     ReadyRole           China East       swarm-master.chinacloudapp.cn                               100.78.186.65
    data:    swarm-node-1     ReadyRole           China East       swarm-node-1.chinacloudapp.cn                               100.66.72.126
    data:    swarm-node-2     ReadyRole           China East       swarm-node-2.chinacloudapp.cn                               100.72.18.47  
    data:    swarm-node-3     ReadyRole           China East       swarm-node-3.chinacloudapp.cn                               100.78.24.68  

## 在 swarm 主 VM 上安装 swarm

本主题使用[从 docker swarm 安装容器模型文档](https://github.com/docker/swarm#1---docker-image) - 但你也可以使用 SSH 连接到 **swarm-master**。在此模型中，将下载 **swarm** 作为运行 swarm 的 docker 容器。下面，我们将执行此步骤*使用 docker 从笔记本电脑远程*连接到 **swarm-master** VM，并指示它使用群集 ID 创建命令 **swarm create**。通过群集 ID，**swarm** 可以发现 swarm 组的成员。（你也可以克隆存储库并自己生成它，从而可以对其完全控制并启用调试。）

    $ docker --tls -H tcp://swarm-master.chinacloudapp.cn:2376 run --rm swarm create
    Unable to find image 'swarm:latest' locally
    511136ea3c5a: Pull complete
    a8bbe4db330c: Pull complete
    9dfb95669acc: Pull complete
    0b3950daf974: Pull complete
    633f3d9a9685: Pull complete
    bba5f98a0414: Pull complete
    defbc1ab4462: Pull complete
    92d78d321ff2: Pull complete
    swarm:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
    Status: Downloaded newer image for swarm:latest
    36731c17189fd8f450c395db8437befd

最后一行是群集 ID；请将其复制到某处，因为当你将节点 VM 加入 swarm 主机以创建“swarm”时将再次使用它。在此示例中，群集 ID 为 **36731c17189fd8f450c395db8437befd**。

> [AZURE.NOTE] 只是为了清楚起见，我们将使用本地 docker 安装连接到 Azure 中的 **swarm-master** VM，并指示 **swarm-master** 下载、安装和运行 **create** 命令，该命令会返回群集 ID，稍后我们会将该 ID 用于发现目的。
<!-- -->
> 若要确认此 ID，请运行 `docker -H tcp://`*&lt;hostname&gt;* ` images` 以列出 **swarm-master** 计算机和另一个节点上的容器进程以进行比较（因为我们在运行前一 swarm 命令时使用了 **--rm** 开关，在它完成后将删除该容器，因此使用 **docker ps-a** 不会返回任何内容）。


        $ docker --tls -H tcp://swarm-master.chinacloudapp.cn:2376 images
        REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
        swarm               latest              92d78d321ff2        11 days ago         7.19 MB
        $ docker --tls -H tcp://swarm-node-1.chinacloudapp.cn:2376 images
        REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
        $
<P />
> 如果你熟悉 **docker**，就会知道其他节点没有任何条目，因为还没有下载并运行任何映像。

## 将节点 VM 加入 docker 群集

对于每个节点，使用 Azure CLI 列出终结点信息。下面我们对 **swarm-node-1** docker 主机执行该操作以获取该节点的 docker 端口。

    $ azure vm endpoint list swarm-node-1
    info:    Executing command vm endpoint list
    + Getting virtual machines
    data:    Name    Protocol  Public Port  Private Port  Virtual IP      EnableDirectServerReturn  Load Balanced
    data:    ------  --------  -----------  ------------  --------------  ------------------------  -------------
    data:    docker  tcp       2376         2376          138.91.112.194  false                     No
    data:    ssh     tcp       22           22            138.91.112.194  false                     No
    info:    vm endpoint list command OK


使用 **docker** 和 `-H` 选项将 docker 客户端指向你的节点 VM，通过传递群集 ID 和节点的 docker 端口将该节点加入要创建的 swarm（后者使用 **--addr**）：

    $ docker --tls -H tcp://swarm-node-1.chinacloudapp.cn:2376 run -d swarm join --addr=138.91.112.194:2376 token://36731c17189fd8f450c395db8437befd
    Unable to find image 'swarm:latest' locally
    511136ea3c5a: Pull complete
    a8bbe4db330c: Pull complete
    9dfb95669acc: Pull complete
    0b3950daf974: Pull complete
    633f3d9a9685: Pull complete
    bba5f98a0414: Pull complete
    defbc1ab4462: Pull complete
    92d78d321ff2: Pull complete
    swarm:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
    Status: Downloaded newer image for swarm:latest
    bbf88f61300bf876c6202d4cf886874b363cd7e2899345ac34dc8ab10c7ae924

这看起来不错。若要确认 **swarm** 正在 **swarm-node-1** 上运行，请键入：

    $ docker --tls -H tcp://swarm-node-1.chinacloudapp.cn:2376 ps -a
        CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
        bbf88f61300b        swarm:latest        "/swarm join --addr=   13 seconds ago      Up 12 seconds       2375/tcp            angry_mclean

对群集中的所有其他节点重复上述操作。在本示例中，我们对 **swarm-node-2** 和 **swarm-node-3** 执行该操作。

## 开始管理 swarm 群集

    $ docker --tls -H tcp://swarm-master.chinacloudapp.cn:2376 run -d -p 2375:2375 swarm manage token://36731c17189fd8f450c395db8437befd
    d7e87c2c147ade438cb4b663bda0ee20981d4818770958f5d317d6aebdcaedd5

然后可以列出群集中的节点：

    ralph@local:~$ docker --tls -H tcp://swarm-master.chinacloudapp.cn:2376 run --rm swarm list token://73f8bc512e94195210fad6e9cd58986f
    54.149.104.203:2375
    54.187.164.89:2375
    92.222.76.190:2375

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤

转到 swarm 上运行操作。若要寻找灵感，请参阅 [https://github.com/docker/swarm/](https://github.com/docker/swarm/)。

<!-- links -->

[docker-machine-azure]: /documentation/articles/virtual-machines-linux-docker-machine/
 

<!---HONumber=Mooncake_0215_2016-->