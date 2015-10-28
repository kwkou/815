<properties
	pageTitle="如何使用 CoreOS | Windows Azure"
	description="介绍 CoreOS、如何在 Azure 上创建 CoreOS 虚拟机群集及其基本用法。"
	services="virtual-machines"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="08/03/2015"
	wacn.date="09/18/2015"/>

# 如何在 Azure 上使用 CoreOS

本主题介绍 [CoreOS] 并演示如何在 Azure 上创建三个 CoreOS 虚拟机构成的群集，以帮助你快速了解此操作系统。它使用非常基本的 CoreOS 部署元素和来自 [CoreOS 与 Azure]、[Tim Park 的 CoreOS 教程]和 [Patrick Chanezon 的 CoreOS 教程]中的示例，演示了解 CoreOS 部署的基本结构及成功运行三个虚拟机构成的群集的绝对最低要求。

>[AZURE.NOTE]本文介绍了如何通过 Azure 命令行界面来使用服务管理命令创建 CoreOS VM。若要在 Azure 资源管理器中开始使用 CoreOS，请尝试此<!--[-->快速入门模板<!--](/documentation/templates/coreos-with-fleet-multivm)-->。

## <a id='intro'>CoreOS、群集和 Linux 容器</a>

CoreOS 是 Linux 的轻量级版本，旨在支持快速创建使用 Linux 容器作为唯一打包机制的可能的大型 VM 群集（包括 [Docker] 容器）。CoreOS 旨在支持：

+ 高度自动化
+ 更轻松且更一致的应用程序部署
+ 应用程序级别和系统级别的可伸缩性

概括而言，支持这些目标的 CoreOS 功能包括：

1. 单一打包系统：CoreOS 仅运行可在 Linux 容器中运行，以取得部署速度、一致性和方便性的 Linux 容器映像
2. 系统会自动执行操作系统更新，这样一来，操作系统会作为单个实体进行更新，并且可以轻松回退到已知状态
3. 内置用于动态 VM 和群集通信与管理的 [etcd](https://github.com/coreos/etcd) 和 [fleet](https://github.com/coreos/fleet) 守护程序（服务）

这是对 CoreOS 及其功能的概要说明。有关 CoreOS 更完整的信息，请参阅 [CoreOS 概述]。

## <a id='security'>安全注意事项</a>
目前，CoreOS 假定可以通过 SSH 进入群集的人员有权对群集进行管理。因此，在不进行修改的情况下，CoreOS 群集在测试和开发环境中的表现也相当出色，但你应该在所有生产环境中应用更多安全措施。

## <a id='usingcoreos'>如何在 Azure 上使用 CoreOS</a>

本部分介绍如何使用 [Azure 命令行界面 (Azure CLI)]，创建拥有三个 CoreOS 虚拟机的 Azure 云服务。基本步骤如下所示：

1. 创建 SSH 证书和密钥，以保护与 CoreOS 虚拟机的通信安全
2. 获取群集的 etcd id 以互相通信
3. 创建 [YAML] 格式的 cloud-config 文件
4. 使用 Azure CLI 创建新的 Azure 云服务和三个 CoreOS VM
5. 在 Azure VM 中测试你的 CoreOS 群集
6. 在 localhost 中测试你的 CoreOS 群集

### 创建用于通信的公钥和私钥

按照[如何在 Azure 上通过 Linux 使用 SSH](/documentation/articles/virtual-machines-linux-use-ssh-key) 中的说明，创建用于 SSH 的公钥和私钥。（可在下面的说明中找到基本步骤。） 你将使用这些密钥连接到群集中的 VM，以验证它们是否可以正常工作并互相通信。

> [AZURE.NOTE]本主题假定你没有这些密钥，为了让你明了操作过程，将要求你创建 `myPrivateKey.pem` 和 `myCert.pem` 文件。如果已将公钥和私钥对保存到 `~/.ssh/id_rsa`，只需键入 `openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out myCert.pem` 即可获取需要上载到 Azure 的 .pem 文件。

1. 在工作目录中，键入 `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout myPrivateKey.key -out myCert.pem` 以创建私钥和与之关联的 X.509 证书。

2. 若要判断私钥的所有者是否可以读取或写入文件，请键入 `chmod 600 myPrivateKey.key`。

你的工作目录中现在应同时包含 `myPrivateKey.key` 和 `myCert.pem` 文件。


### 获取群集的 etcd id

CoreOS 的 `etcd` 守护程序需要发现 ID，以自动查询群集中的所有节点。若要检索你的发现 ID 并将其保存到 `etcdid` 文件，请键入

```
curl https://discovery.etcd.io/new | grep ^http.* > etcdid
```

### 创建 cloud-config 文件

在相同的工作目录中，使用你最喜欢的文本编辑器创建包含以下文本的文件，并将其另存为 `cloud-config.yaml`。（可以使用你想要的任何文件名将其保存，但在下一步创建 VM 时，必须在 **azure vm create** 命令的 **--custom-data** 选项中引用此文件的名称。）

> [AZURE.NOTE]请记得键入 `cat etcdid`，以从之前创建的 `etcdid` 文件中检索 etcd 发现 id，并使用 `etcdid` 文件生成的数字替换以下 `cloud-config.yaml` 文件中的 `<token>`。如果最后无法验证群集，这可能会是你忽略了的其中一个步骤！

```
#cloud-config

coreos:
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    discovery: https://discovery.etcd.io/<token>
    # deployments across multiple cloud services will need to use $public_ipv4
    addr: $private_ipv4:4001
    peer-addr: $private_ipv4:7001
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
```

（有关 cloud-config 文件更完整的信息，请参阅 CoreOS 文档中的[使用 Cloud-Config](https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/)。）

### 使用 Azure CLI 创建新的 CoreOS VM
<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

1. 如果尚未安装 [Azure 命令行界面 (Azure CLI)]，请先执行安装并使用工作或学校 ID 登录，或下载 .publishsettings 文件并将其导入你的帐户。
2. 寻找 CoreOS 映像。若要查找随时可用的映像，请键入 `azure vm image list | grep CoreOS`，然后就会看到类似于以下内容的结果列表：

	数据: 2b171e93f07c4903bcad35bda10acf22\_\_CoreOS-Stable-522.6.0 Public Linux

3. 键入 `azure service create <cloud-service-name>`（其中的 <*cloud-service-name*> 是你的 CoreOS 云服务的名称）可创建用于基本群集的云服务。此示例使用 **`coreos-cluster`** 作为名称；你将需要重用所选名称来创建云服务内部的 CoreOS VM 实例。

注意：如果在[门户](https://manage.windowsazure.cn)中观察你到目前为止的工作，你会在资源组和域中看到你的云服务名称，如下图所示：

	![][CloudServiceInNewPortal]

4. 使用 **azure vm create** 命令可连接到你的云服务，并可在其中创建新的 CoreOS VM。你将在 **--ssh-cert** 选项中传递 X.509 证书的位置。通过键入以下命令创建你的第一个 VM 映像，请记得使用你创建的云服务名称替换 **coreos-cluster**：

```
azure vm create --custom-data=cloud-config.yaml --ssh=22 --ssh-cert=./myCert.pem --no-ssh-password --vm-name=node-1 --connect=coreos-cluster --location='China East' 2b171e93f07c4903bcad35bda10acf22__CoreOS-Stable-522.6.0 core
```

5. 通过重复步骤 4 中的命令来创建第二个节点，使用 **node-2** 替换 **--vm-name** 值，并使用 2022 替换 **--ssh** 端口值。

6. 通过重复步骤 4 中的命令来创建第三个节点，使用 **node-3** 替换 **--vm-name** 值，并使用 3022 替换 **--ssh** 端口值。

从下方的截图中，你可以看到 CoreOS 群集在门户中显示时的样子。

![][EmptyCoreOSCluster]

### 在 Azure VM 中测试你的 CoreOS 群集

若要测试你的群集，请确保你位于工作目录中，然后使用 **ssh** 连接到 **node-1**，并通过键入以下命令传递私钥：

`ssh core@coreos-cluster.chinacloudapp.cn -p 22 -i ./myPrivateKey.key`

连接后，键入 `sudo fleetctl list-machines` 以查看群集是否已识别出群集中的所有 VM。你应会收到类似于下面的响应：


	core@node-1 ~ $ sudo fleetctl list-machines
	MACHINE		IP		METADATA
	442e6cfb...	100.71.168.115	-
	a05e2d7c...	100.71.168.87	-
	f7de6717...	100.71.188.96	-


### 在 localhost 中测试你的 CoreOS 群集

最后，让我们测试你本地 Linux 客户端中的 CoreOS 群集。你也许可以使用 **npm** 来安装 **fleetctl**；或者，你可能希望安装 **fleet** 并在本地客户端上自行构建 **fleetctl**。**fleet** 需要使用 **golang**，因此，你可能需要先安装后者，则需键入：

`sudo apt-get install golang`

然后从 github 克隆 **fleet** 存储库，方法是键入：

`git clone https://github.com/coreos/fleet.git`

生成 **fleet**，方法是转到 `fleet` 目录并键入

`./build`

最后，添加 **fleet** 以便能够轻松使用（你可能需要也可能不需要 **sudo**，具体取决于你的配置）：

`cp bin/fleetctl /usr/local/bin`

确保 **fleet** 能够访问工作目录中的 `myPrivateKey.key`，方法是键入：

`ssh-add ./myPrivateKey.key`

> [AZURE.NOTE]如果你已在使用 `~/.ssh/id_rsa` 密钥，则使用 `ssh-add ~/.ssh/id_rsa` 进行添加。

现在，你已准备好使用你在 **node-1** 中使用的相同 **fleetctl** 命令进行远程测试，但需要传递部分远程参数：

`fleetctl --tunnel coreos-cluster.chinacloudapp.cn:22 list-machines`

结果应完全相同：


	MACHINE		IP		METADATA
	442e6cfb...	100.71.168.115	-
	a05e2d7c...	100.71.168.87	-
	f7de6717...	100.71.188.96	-

## 后续步骤

在 Azure 上，你现在应该拥有正常运行的三节点 CoreOS 群集。此后，你可以探索如何创建更复杂的群集、使用 Docker 和创建更有趣的应用程序。若要尝试操作几个快速示例，请参阅[在 Azure 上的 CoreOS 上使用 Fleet 入门]。

<!--Anchors-->
[CoreOS, Clusters, and Linux Containers]: #intro
[Security Considerations]: #security
[How to use CoreOS on Azure]: #usingcoreos
[Subheading 3]: #subheading-3
[Next steps]: #next-steps

<!--Image references-->
[CloudServiceInNewPortal]: ./media/virtual-machines-linux-coreos-how-to/cloudservicefromnewportal.png
[EmptyCoreOSCluster]: ./media/virtual-machines-linux-coreos-how-to/endresultemptycluster.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png


<!--Link references-->
[Azure 命令行界面 (Azure CLI)]: /documentation/articles/xplat-cli
[CoreOS]: https://coreos.com/
[CoreOS 概述]: https://coreos.com/using-coreos/
[CoreOS 与 Azure]: https://coreos.com/docs/running-coreos/cloud-providers/azure/
[Tim Park 的 CoreOS 教程]: https://github.com/timfpark/coreos-azure
[Patrick Chanezon 的 CoreOS 教程]: https://github.com/chanezon/azure-linux/tree/master/coreos/cloud-init
[Docker]: http://docker.io
[YAML]: http://yaml.org/
[在 Azure 上的 CoreOS 上使用 Fleet 入门]: /documentation/articles/virtual-machines-linux-coreos-fleet-get-started
<!---HONumber=70-->