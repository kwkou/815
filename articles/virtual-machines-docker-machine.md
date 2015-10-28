<properties pageTitle="如何在 Azure 上使用 docker-machine" description="演示如何在 Azure 上启动和运行 Ubuntu 上的 Docker 计算机。" services="virtual-machines" documentationCenter="virtual-machines" authors="squillace" manager="timlt" editor="tysonn"/>

<tags ms.service="virtual-machines" ms.date="05/25/2015" wacn.date="08/29/2015"/>

# 如何将 docker-machine 与 Azure 一起使用

本主题介绍如何将 [Docker](https://www.docker.com/) 与[计算机](https://github.com/docker/machine)和 [Azure CLI](https://github.com/Azure/azure-xplat-cli) 结合使用来创建 Azure 虚拟机，以便快速轻松地从运行 Ubuntu 的计算机管理 Linux 容器。为了演示，本教程将说明如何同时部署 [busybox Docker Hub 映像](https://registry.hub.docker.com/_/busybox/)和 [nginx Docker Hub 映像](https://registry.hub.docker.com/_/nginx/)，并配置容器将 Web 请求路由到 nginx 容器。（Docker **计算机**文档介绍如何针对其他平台修改这些说明。）

完成本教程有一些先决条件。你将需要安装以下项：

1. [npm](https://docs.npmjs.com/) 和 [Node.js](http://nodejs.org/)
2. [Azure CLI](https://github.com/Azure/azure-xplat-cli)
3. [Docker 客户端](https://docs.docker.com/installation/)

如果按此顺序安装这些项，Ubuntu 计算机就会具备使用本教程的条件。（本教程应在很大程度上也适用于任何其他基于 dpkg 的发行版本，例如 Debian。如果你发现额外的要求或步骤，请在评论中让我们知道。）

## 获取 docker-machine - 或生成它

熟悉 **docker-machine** 的最快方法是直接从[版本共享](https://github.com/docker/machine/releases)下载相应的发行版。本教程中的客户端计算机在 x64 计算机上运行 Ubuntu，因此 **docker-machine_linux amd64** 映像是所使用的那一个。

你还可以按照[提供给虚拟机](https://github.com/docker/machine#contributing)的步骤自己构建 **docker-machine**。为执行此生成你应准备好下载多达 1 GB 或更多的内容，但通过这样做你可以按所需方式准确地自定义自己的体验。

> [AZURE.NOTE]你或许可以创建指向它的平台版本的[符号链接](http://zh.wikipedia.org/wiki/Symbolic_link)，但本教程使用二进制文件直接非常清楚地演示行为。结果是，不是使用 **docker-machine** 文档所演示的 `docker-machine env` 等命令，本教程改用 `docker-machine_linux-amd64 env`。是创建符号链接还是直接使用二进制文件名称直接取决于你，但如果你更改所用的名称，请记得修改下面的说明中的名称。

<br />

>  无论你使用哪种方法，你都必须直接在命令行上调用二进制文件或将二进制文件放在路径（例如 **/usr/local/bin**）上。请记住，确保通过键入 `chmod +x` &lt;*`binaryName`*&gt; 将它标记为可执行文件，其中 &lt;*`binaryName`*&gt; 是 Docker 计算机可执行文件的名称。本教程使用 **docker-machine_linux-amd64**。

## 为 docker、计算机和 Azure 创建证书和密钥文件

现在，你必须创建 Azure 需要用来确认你的身份和权限的证书和密钥文件，以及 **docker-machine** 需要用来与 Azure 虚拟机进行通信以远程创建和管理容器的证书和密钥文件。如果你的目录中已有这些文件 - 可能用于 docker - 你可以重复使用它们。但是，测试 **docker-machine** 的最佳做法是在一个单独的目录中创建它们，并使 docker-machine 指向它们。

> [AZURE.NOTE]如果你结果是反复试用 **docker-machine**，请务必重复使用相同的证书和密钥文件。**docker-machine** 也创建一组客户端证书（它创建的所有内容都可以在 `~/.docker/machine` 中查看）。如果将这些证书移到另一台计算机，你也需要移动 **docker-machine** 证书文件夹。如果要在另一个平台上使用 **docker-machine**（例如，只为了解其所有工作原理），这会有所不同。

如果你有 Linux 分发的经验，则可能已将这些文件提供给特定位置中的计算机使用，[Docker HTTPS 文档很好地说明了这些步骤](https://docs.docker.com/articles/https/)。但是，以下是此步骤的最简单形式。

1. 创建一个本地文件夹，并在命令行中导航到该文件夹，然后键入：

		openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem
		openssl pkcs12 -export -out mycert.pfx -in mycert.pem -name "My Certificate"

	在此处准备好输入你的证书的导出密码并捕获该证书供将来使用。然后键入：

		openssl x509 -inform pem -in mycert.pem -outform der -out mycert.cer

2. 将证书的 .cer 文件上载到 Azure。在 [Azure 门户](https://manage.windowsazure.cn)中，单击服务区域左下角的**“设置”**（下面显示）

	![][portalsettingsitem]

	然后单击**“管理证书”**：

	![][managementcertificatesitem]

	然后单击**“上载”**（在页面底部）![][uploaditem] 以上载在上一步中创建的 **mycert.cer** 文件。

3. 在门户的同一个**“设置”**窗格中，单击**“订阅”**，并捕获在创建 VM 时要使用的订阅 ID，因为你将在下一步中使用它。（你还可以在命令行上使用 Azure CLI 命令 `azure account list` 查找订阅 ID，该命令显示你的帐户中拥有的每个订阅的订阅 ID。）

4. 在 Azure 上使用 **docker-machine create** 命令创建 docker 宿主 VM。该命令需要你在上一步中刚捕获的订阅 ID 和在步骤 1 中创建的 **.pem** 文件的路径。本主题使用“machine-name”作为 Azure 云服务（和你的 VM）名称，但你应将其替换为你自己的选项，并在需要 VM 名称的任何其他步骤中记住使用你的云服务名称。（请记住，在此示例中，我们将使用完整二进制文件名称，而不是 **docker-machine** 符号链接。）

		docker-machine_linux-amd64 create \
	    -d azure \
	    --azure-subscription-id="<subscription ID acquired above>" \
	    --azure-subscription-cert="mycert.pem" \
	    machine-name

	由于前两个步骤需要创建一个新的 VM 并加载 Linux Azure 代理以及更新新的 VM，你应看到如下内容。

		INFO[0001] Creating Azure machine...
	    INFO[0049] Waiting for SSH...
	    modprobe: FATAL: Module aufs not found.
	    + sudo -E sh -c sleep 3; apt-get update
	    + sudo -E sh -c sleep 3; apt-get install -y -q linux-image-extra-3.13.0-36-generic
	    E: Unable to correct problems, you have held broken packages.
	    modprobe: FATAL: Module aufs not found.
	    Warning: tried to install linux-image-extra-3.13.0-36-generic (for AUFS)
	     but we still have no AUFS.  Docker may not work. Proceeding anyways!
	    + sleep 10
	    + [ https://get.docker.com/ = https://get.docker.com/ ]
	    + sudo -E sh -c apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
	    gpg: requesting key A88D21E9 from hkp server keyserver.ubuntu.com
	    gpg: key A88D21E9: public key "Docker Release Tool (releasedocker) <docker@dotcloud.com>" imported
	    gpg: Total number processed: 1
	    gpg:               imported: 1  (RSA: 1)
	    + sudo -E sh -c echo deb https://get.docker.com/ubuntu docker main > /etc/apt/sources.list.d/docker.list
	    + sudo -E sh -c sleep 3; apt-get update; apt-get install -y -q lxc-docker
	    + sudo -E sh -c docker version
	    INFO[0368] "machine-name" has been created and is now the active machine.
	    INFO[0368] To point your Docker client at it, run this in your shell: $(docker-machine_linux-amd64 env machine-name)

    > [AZURE.NOTE]由于正在创建 VM，它可能需要几分钟才能处于就绪状态。在等待时，你可以通过使用 Azure CLI 键入 `azure vm list` 来查看新 Docker 主机的状态，直到你看到具有 **ReadyRole** 状态的 VM。

5. 为终端会话设置 docker 和计算机环境变量。反馈的最后一行建议你立即运行 **env** 命令以导出直接在特定计算机上使用 docker 客户端所必需的环境变量。

		$(docker-machine_linux-amd64 env machine-name)

	这么做后，你将不需要任何特殊命令即可使用你的本地 docker 客户端连接到 Docker **计算机**创建的 VM。

	    $ docker info
	    Containers: 0
	    Images: 0
	    Storage Driver: devicemapper
	     Pool Name: docker-8:1-131736-pool
	     Pool Blocksize: 65.54 kB
	     Backing Filesystem: extfs
	     Data file: /dev/loop0
	     Metadata file: /dev/loop1
	     Data Space Used: 305.7 MB
	     Data Space Total: 107.4 GB
	     Metadata Space Used: 729.1 kB
	     Metadata Space Total: 2.147 GB
	     Udev Sync Supported: false
	     Data loop file: /var/lib/docker/devicemapper/devicemapper/data
	     Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
	     Library Version: 1.02.82-git (2013-10-04)
	    Execution Driver: native-0.2
	    Kernel Version: 3.13.0-36-generic
	    Operating System: Ubuntu 14.04.1 LTS
	    CPUs: 1
	    Total Memory: 1.639 GiB
	    Name: machine-name
	    ID: W3FZ:BCZW:UX24:GDSV:FR4N:N3JW:XOC2:RI56:IWQX:LRTZ:3G4P:6KJK
	    WARNING: No swap limit support

> [AZURE.NOTE]本教程演示创建一个 VM 的 **docker-machine**。但是，你可以重复执行这些步骤创建所需的任意数量的虚拟机。如果这样做，则使用 docker 切换 VM 的最佳方式是以内联方式使用 **env** 命令，以便为每个单独的命令设置 **docker** 环境变量。例如，若要对不同 VM 使用 **docker info**，可以键入 `docker $(docker-machine env <VM name>) info`，**env** 命令会填写要用于该 VM 的 docker 连接信息。

## 至此已完成。让我们来使用 docker 和 Docker Hub 映像远程运行一些应用程序。

现在可以以正常方式使用 docker 在容器中创建应用程序。最容易演示的是 [busybox](https://registry.hub.docker.com/_/busybox/)：

	    $  docker run busybox echo hello world
	    Unable to find image 'busybox:latest' locally
	    511136ea3c5a: Pull complete
	    df7546f9f060: Pull complete
	    ea13149945cb: Pull complete
	    4986bf8c1536: Pull complete
	    busybox:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
	    Status: Downloaded newer image for busybox:latest
	    hello world

但是，你可能想要创建可立即在 Internet 上看到的应用程序，例如 [Docker Hub](https://registry.hub.docker.com/) 中的 [nginx](https://registry.hub.docker.com/_/nginx/)。

> [AZURE.NOTE]请记住使用 **-P** 选项让 **docker** 随机将端口分配给该映像，并使用 **-d** 以确保该容器在后台继续运行。（如果你忘记，你将启动 nginx，然后它将立即关闭。不要忘记！）

	$ docker run --name machinenginx -P -d nginx
    Unable to find image 'nginx:latest' locally
    30d39e59ffe2: Pull complete
    c90d655b99b2: Pull complete
    d9ee0b8eeda7: Pull complete
    3225d58a895a: Pull complete
    224fea58b6cc: Pull complete
    ef9d79968cc6: Pull complete
    f22d05624ebc: Pull complete
    117696d1464e: Pull complete
    2ebe3e67fb76: Pull complete
    ad82b43d6595: Pull complete
    e90c322c3a1c: Pull complete
    4b5657a3d162: Pull complete
    511136ea3c5a: Already exists
    nginx:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
    Status: Downloaded newer image for nginx:latest
    5883e2ff55a4ba0aa55c5c9179cebb590ad86539ea1d4d367d83dc98a4976848

若要在 Internet 上查看它，请在 Azure VM 的端口 80 上创建公共终结点，并将该端口映射到 nginx 容器的端口。首先，使用 `docker ps -a` 找到容器，并查找 **docker** 已将哪些端口分配给容器的端口 80。（下面显示的信息已编辑为仅显示端口信息；你将看到更多信息。）

	$ docker ps -a
    IMAGE               PORTS
    nginx:latest        0.0.0.0:49153->80/tcp, 0.0.0.0:49154->443/tcp
    busybox:latest

我们可以看到 docker 已将容器的端口 80 分配给 VM 的端口 49153。我们现在可以使用 Azure CLI 将外部云服务的公共端口 80 映射到 VM 上的端口 49153。然后，Docker 将确保将 VM 端口 49153 上的入站 tcp 流量路由到 nginx 容器。

	$ azure vm endpoint create machine-name 80 49153
    info:    Executing command vm endpoint create
    + Getting virtual machines
    + Reading network configuration
    + Updating network configuration
    info:    vm endpoint create command OK

打开你最喜欢的浏览器进行查看。

![][nginx]

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤
转到 [Docker 用户指南](https://docs.docker.com/userguide/)并在 Windows Azure 上创建一些应用程序。或者，在 Azure 上玩 [**docker** 和 swarm](https://github.com/docker/swarm)](virtual-machines-docker-swarm)，并了解如何将 swarm 用于 docker 和 Azure。

<!--Image references-->
[nginx]: ./media/virtual-machines-docker-machine/nginxondocker.png
[portalsettingsitem]: ./media/virtual-machines-docker-machine/portalsettingsitem.png
[managementcertificatesitem]: ./media/virtual-machines-docker-machine/managementcertificatesitem.png
[uploaditem]: ./media/virtual-machines-docker-machine/uploaditem.png

<!--Link references-->
[Link 1 to another azure.microsoft.com documentation topic]: /documentation/articles/virtual-machines-windows-tutorial
[Link 2 to another azure.microsoft.com documentation topic]: /documentation/articles/web-sites-custom-domain-name
[Link 3 to another azure.microsoft.com documentation topic]: /documentation/articles/storage-whatis-account
<!---HONumber=67-->