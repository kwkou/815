

[Docker](https://www.docker.com/) 是最常用的虚拟化技术之一，它使用 [Linux 容器](http://zh.wikipedia.org/wiki/LXC)而不是虚拟机作为在共享资源上隔离应用程序数据和执行计算的方法。可以在 [Azure Linux 代理](/documentation/articles/virtual-machines-linux-agent-user-guide/)中使用 [Azure Docker VM 扩展](https://github.com/Azure/azure-docker-extension/blob/master/README.md)，以创建可在 Azure 上为应用程序托管任意数量的容器的 Docker VM。

本主题介绍：

+ [Docker 和 Linux 容器]
+ [如何对 Azure 使用 Docker VM 扩展]
+ [适用于 Linux 和 Windows 的虚拟机扩展]

若要立即创建启用 Docker 的 VM，请参阅：

+ [如何从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展]

若要了解有关该扩展及其工作原理的详细信息，请参阅 [Docker 扩展用户指南](https://github.com/Azure/azure-docker-extension/blob/master/README.md)。

##<a name="Docker-and-Linux-Containers"></a> Docker 和 Linux 容器
[Docker](https://www.docker.com/) 是最常用的虚拟化方法之一，它使用 [Linux 容器](http://wikipedia.org/wiki/LXC)而不是虚拟机作为在共享资源上隔离数据和执行计算的方法，并提供其他服务使你可以快速生成或汇编应用程序，然后在其他 Docker 容器之间分发应用程序。

Docker 和 Linux 容器不是诸如 Windows Hyper-V 和 Linux 上 [KVM](http://wikipedia.org/wiki/Hypervisor)（还有许多其他例子）之类的[虚拟机监控程序](http://www.linux-kvm.org/page/Main_Page)。虚拟机监控程序将虚拟化底层操作系统，使整个操作系统（称为*虚拟机*）像应用程序一样在虚拟机监控程序中运行。

Docker 及其他*容器*技术使用 Linux 内核的进程和文件系统隔离功能，只将内核功能公开给其他隔离容器，因而可真正减少启动时间以及处理和存储开销。

下表从较高的层面描述了虚拟机监控程序与 Linux 容器之间存在的功能差异类型。请注意，根据你自身的应用程序需求，某些功能的作用或大或小。

| 功能 | 虚拟机监控程序 | 容器 |
| :------------- |-------------| ----------- |
| 进程隔离 | 完整性或大或小 | 如果获取 root 权限，容器主机可能会泄密 |
| 磁盘上需有内存 | 完整操作系统加上应用 | 仅限应用要求 |
| 启动所需时间 | 明显更长：操作系统引导和应用程序加载 | 明显更短：只需启动应用，因为内核已在运行 |
| 容器自动化 | 根据操作系统和应用差别很大 | [Docker 映像库](https://registry.hub.docker.com/)；其他

若要查看容器及其优点的综合讨论，请参阅 [Docker 高级白板](http://channel9.msdn.com/Blogs/Regular-IT-Guy/Docker-High-Level-Whiteboard)。

有关 Docker 的定义及其实际工作原理的详细信息，请参阅[什么是 Docker？](https://www.docker.com/whatisdocker/)

#### Docker 和 Linux 容器的安全最佳实践

因为容器对主计算机内核进行共享访问，因此，如果恶意代码可以获取 root 权限，则也可以获取对其他容器的访问权限，而不仅仅是主机。为了以强于默认配置的力度保护容器系统，[Docker 建议](https://docs.docker.com/articles/security/)同时使用附加的组原则或[基于角色的安全性](http://zh.wikipedia.org/wiki/RBAC)（例如 [SELinux](http://selinuxproject.org/page/Main_Page) 或 [AppArmor](http://wiki.apparmor.net/index.php/Main_Page)），并尽可能减少授予容器的内核功能。除此之外，Internet 上也提供了其他许多介绍如何使用 Docker 等容器实现安全性的文档。

##<a name="How-to-use-the-Docker-VM-Extension-with-Azure"></a> 如何对 Azure 使用 Docker VM 扩展

Docker VM 扩展是在你创建的 VM 实例中安装的组件，它会自行安装 Docker 引擎并管理与 VM 的远程通信。安装 VM 扩展的方式有两种：使用经典管理门户创建 VM，或通过 Azure 命令行界面 (Azure CLI) 创建 VM。

可以使用门户预览将 Docker VM 扩展添加到任何兼容的 Linux VM（目前，支持此扩展的唯一映像是七月份后推出的 Ubuntu 14.04 LTS 映像）。但是，如果使用 Azure CLI 命令行，则可以在创建 VM 实例时安装 Docker VM 扩展，以及创建并上载你的 Docker 通信证书。

若要立即创建启用 Docker 的 VM，请参阅：

+ [如何从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展]

##<a name="Virtual-Machine-Extensions-For-Linux-and-Windows"></a> 适用于 Linux 和 Windows 的虚拟机扩展
[Azure 的 Docker VM 扩展](https://github.com/Azure/azure-docker-extension/blob/master/README.md)只是提供特殊行为的众多 VM 扩展之一，还有许多正在开发中。例如，许多 [Linux VM 代理扩展](/documentation/articles/virtual-machines-linux-agent-user-guide/)功能可让你修改和管理虚拟机，包括安全功能、内核和网络功能，等等。例如，VMAccess 扩展可让你重置管理员密码或 SSH 密钥。

有关完整列表，请参阅 Azure VM 扩展：[Windows](/documentation/articles/virtual-machines-windows-extensions-features/) 或者 [Linux](/documentation/articles/virtual-machines-linux-extensions-features/)。

<!--Anchors-->
[如何从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展]: /documentation/articles/virtual-machines-linux-classic-cli-use-docker/
[Docker 和 Linux 容器]: #Docker-and-Linux-Containers
[如何对 Azure 使用 Docker VM 扩展]: #How-to-use-the-Docker-VM-Extension-with-Azure
[适用于 Linux 和 Windows 的虚拟机扩展]: #Virtual-Machine-Extensions-For-Linux-and-Windows
