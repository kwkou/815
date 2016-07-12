<properties
	pageTitle="使用适用于 Azure 上的 Linux 的 Docker VM 扩展"
	description="介绍 Docker 以及 Azure 虚拟机扩展，并说明如何在 Azure 上，使用 Azure CLI 通过命令行以编程方式创建用作 Docker 主机的虚拟机。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="01/04/2016"
	wacn.date="02/26/2016"/>

# 从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

本主题说明如何通过 Azure CLI 中的服务管理 (asm) 模式，在任何平台上创建包含 Docker VM 扩展的 VM。[Docker](https://www.docker.com/) 是最常用的虚拟化技术之一，它使用 [Linux 容器](http://zh.wikipedia.org/wiki/LXC)而不是虚拟机作为在共享资源上隔离数据和执行计算的方法。可以在 [Azure Linux 代理](/documentation/articles/virtual-machines-linux-agent-user-guide/)中使用 Docker VM 扩展，以创建可在 Azure 上为应用程序托管任意数量的容器的 Docker VM。若要查看容器及其优点的综合讨论，请参阅 [Docker 高级白板](http://channel9.msdn.com/Blogs/Regular-IT-Guy/Docker-High-Level-Whiteboard)。


##如何对 Azure 使用 Docker VM 扩展
若要对 Azure 使用 Docker VM，你必须安装 0.8.6 版本以上的 [Azure 命令行界面 ](https://github.com/Azure/azure-sdk-tools-xplat)(Azure CLI) （截至本文编写时的最新版本为 0.10.0）。可以在 Mac、Linux 和 Windows 上安装 Azure CLI。


在 Azure 上使用 Docker 的整个过程相当简单：

+ 在要从中控制 Azure 的计算机上安装 Azure CLI 及其依赖项（在 Windows 上，这是作为虚拟机运行的 Linux 分发版）
+ 使用 Azure CLI Docker 命令在 Azure 中创建 VM Docker 主机
+ 使用本地 Docker 命令在 Azure 中的 Docker VM 内管理 Docker 容器。


### 安装 Azure 命令行界面 (Azure CLI)

若要安装和配置 Azure CLI，请参阅[如何安装 Azure 命令行界面](/documentation/articles/xplat-cli-install/)。若要确认安装，请在命令提示符下键入 `azure`，片刻之后，你就会看到 Azure CLI ASCII 图文，其中列出了可用的基本命令。如果安装正常，你应该可以键入 `azure help vm`，并看到列出的其中一个命令是“docker”。

> [AZURE.NOTE] Docker 提供适用于 Windows 的工具 [Docker Machine](https://docs.docker.com/installation/windows/)，你也可以使用该安装程序自动创建可以配合用作 docker 主机的 Azure VM 使用的 docker 客户端。

### 将 Azure CLI 连接到你的 Azure 帐户
在使用 Azure CLI 之前，必须将 Azure 帐户凭据与平台上的 Azure CLI 相关联。[如何连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)部分说明了如何下载和导入 **.publishsettings** 文件，或者将 Azure CLI 与组织 ID 相关联。在Bash或终端会话下键入一下Azure CLI命令来链接到Azure账户,并将其中的<username\>和<password\>分别替换为您自己的用户名和密码。

`azure login --environment AzureChinaCloud --username <username> --password <password>`

登录成功后将服务模式切换为**asm**模式

`azure config mode asm`

> [AZURE.NOTE] 由于使用一个或其他多个身份验证方法的行为有一些差异，因此请务必阅读上述文档，以了解不同的功能。

### 安装 Docker 并对 Azure 使用 Docker VM 扩展
遵循 [Docker 安装说明](https://docs.docker.com/installation/#installation)在计算机本地安装 Docker。

若要将 Docker 与 Azure 虚拟机配合使用，VM 使用的 Linux 映像上必须已安装 [Azure Linux VM 代理](/documentation/articles/virtual-machines-linux-agent-user-guide/)。目前，只有两种类型的映像提供此代理：

+ Azure 映像库中的 Ubuntu 映像，或

+ 使用已安装并配置的 Azure Linux VM 代理创建的自定义 Linux 映像。有关如何使用 Azure VM 代理构建自定义 Linux VM 的详细信息，请参阅 [Azure Linux VM 代理](/documentation/articles/virtual-machines-linux-agent-user-guide/)。

### 使用 Azure 映像库

从 Bash 或终端会话键入以下 Azure CLI 命令找到 VM 映像库中要使用的最新 Ubuntu 映像

`azure vm image list | grep Ubuntu-14_04`

选择其中一个映像名称（例如 `b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20160516-en-us-30GB`），然后使用以下命令创建使用该映像的新 VM。

	
	azure vm docker create -e 22 -l "China North" <vm-cloudservice name> "b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20160516-en-us-30GB" <username> <password>
	

其中：

+ *&lt;vm-cloudservice name&gt;* 是将成为 Azure 中 Docker 容器主计算机的 VM 名称。

+  *&lt;username&gt;* 是 VM 默认 root 用户的用户名

+ *&lt;password&gt;* 是 *username* 帐户的密码，需符合 Azure 的复杂性标准

> [AZURE.NOTE] 目前，密码必须至少有 8 个字符，包含一个小写字母和一个大写字母、一个数字以及下列特殊字符之一：`!@#$%^&+=`。不，上一个句子末尾的句号并不是特殊字符。

如果命令成功，根据使用的确切参数和选项，你应会可以看到如下所示的内容：

![](./media/virtual-machines-linux-classic-cli-use-docker/dockercreateresults.png)

> [AZURE.NOTE] 创建虚拟机可能需要几分钟的时间，但当其设置完成后（状态值为 `ReadyRole`），Docker 守护程序（Docker 服务）便会启动，你可以连接到 Docker 容器主机。

若要测试你在 Azure 中创建的 Docker VM，请键入

	sudo docker --tls -H tcp://<vm-name-you-used>.chinacloudapp.cn:2376 info

其中，*&lt;vm-name-you-used&gt;* 是你在 `azure vm docker create` 调用中使用的虚拟机名称。你应会看到如下所示的内容，这表示 Docker 主机 VM 已在 Azure 中启动和运行，并且正等待你的命令。

现在，可以尝试使用你的 Docker 客户端进行连接以获取信息（在某些 Docker 客户端设置（如 Mac 上的 Docker 客户端设置）中可能需要使用 `sudo`）：

	sudo docker --tls -H tcp://testsshasm.chinacloudapp.cn:2376 info
	Password:
	Containers: 0
	Images: 0
	Storage Driver: devicemapper
	Pool Name: docker-8:1-131781-pool
	Pool Blocksize: 65.54 kB
	Backing Filesystem: extfs
	Data file: /dev/loop0
	Metadata file: /dev/loop1
	Data Space Used: 1.821 GB
	Data Space Total: 107.4 GB
	Data Space Available: 28 GB
	Metadata Space Used: 1.479 MB
	Metadata Space Total: 2.147 GB
	Metadata Space Available: 2.146 GB
	Udev Sync Supported: true
	Deferred Removal Enabled: false
	Data loop file: /var/lib/docker/devicemapper/devicemapper/data
	Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
	Library Version: 1.02.77 (2012-10-15)
	Execution Driver: native-0.2
	Logging Driver: json-file
	Kernel Version: 3.19.0-28-generic
	Operating System: Ubuntu 14.04.3 LTS
	CPUs: 1
	Total Memory: 1.637 GiB
	Name: testsshasm
	WARNING: No swap limit support

为了确认它一直在工作，你可以检查 Docker 扩展的 VM：

	azure vm extension get testsshasm
	info: Executing command vm extension get
	+ Getting virtual machines
	data: Publisher Extension name ReferenceName Version State
	data: -------------------- --------------- ------------------------- ------- ------
	data: Microsoft.Azure.E... DockerExtension DockerExtension 1.* Enable
	info: vm extension get command OK

### Docker 主机 VM 身份验证

除了创建 Docker VM 以外，`azure vm docker create` 命令还会自动创建所需的证书，使 Docker 客户端计算机能够使用 HTTPS 连接到 Azure 容器主机；这些证书将适当地存储在客户端和主计算机上。在后续尝试时，现有证书将被重复使用并与新主机共享。

默认情况下，证书放置在 `~/.docker` 中，而 Docker 将配置为在端口 **2376** 上运行。如果你要使用不同的端口或目录，则可以使用下列其中一个 `azure vm docker create` 命令行选项将 Docker 容器主机 VM 配置为使用不同端口或不同证书来连接客户端：

	
	-dp, --docker-port [port]              Port to use for docker [2376]
	-dc, --docker-cert-dir [dir]           Directory containing docker certs [.docker/]
	

主机上的 Docker 守护程序配置为使用由 `azure vm docker create` 命令生成的证书侦听和验证指定端口上的客户端连接。客户端计算机必须使用这些证书来获取对 Docker 主机的访问权限。

> [AZURE.NOTE] 不使用这些证书运行的联网主机很容易受到任何可连接到该计算机的用户的攻击。在修改默认配置之前，请确保了解这样做会对计算机和应用程序造成的风险。

## 后续步骤

* 现在，你可以转到 [Docker 用户指南]并开始使用你的 Docker VM。

* Azure Docker VM 扩展同样支持 Docker Compose，可以使用声明 YAML 文件来让一个 developer-modeled 应用程序跨越任意环境，并生成一致的部署。请参阅[开始使用 Docker 和 Compose，在 Azure 虚拟机上定义和运行多容器应用程序]

<!--Anchors-->
[Subheading 1]: #subheading-1
[Subheading 2]: #subheading-2
[Subheading 3]: #subheading-3
[Next steps]: #next-steps

[How to use the Docker VM Extension with Azure]: #How-to-use-the-Docker-VM-Extension-with-Azure
[Virtual Machine Extensions for Linux and Windows]: #Virtual-Machine-Extensions-For-Linux-and-Windows
[Container and Container Management Resources for Azure]: #Container-and-Container-Management-Resources-for-Azure



<!--Link references-->
[Link 1 to another azure.microsoft.com documentation topic]: /documentation/articles/virtual-machines-windows-hero-tutorial/
[Link 2 to another azure.microsoft.com documentation topic]: /documentation/articles/web-sites-custom-domain-name/
[Link 3 to another azure.microsoft.com documentation topic]: /documentation/articles/storage-whatis-account/

[Docker 用户指南]: https://docs.docker.com/userguide/
 
[开始使用 Docker 和 Compose，在 Azure 虚拟机上定义和运行多容器应用程序]: /documentation/articles/virtual-machines-linux-docker-compose-quickstart/

<!---HONumber=Mooncake_0215_2016-->