
Azure 提供出色的云解决方案，以虚拟机为基础构建（基于物理计算机硬件的模拟），可实现软件部署的灵活移动，以及相比于物理硬件大大优化的资源整合。在过去几年内，Linux 容器技术已显著拓展出了许多可用于开发和管理分布式软件的方式，这在很大程度上要归功于访问容器的 [Docker](https://www.docker.com) 方法以及 docker 生态系统。容器中的应用程序代码独立于主机 Azure VM 以及同一 VM 上的其他容器，这样除了 Azure VM 已经提供的灵活性之外，你还能在应用程序级别获得更大的开发和部署灵活性。

**但是，这个新闻已经过时了。** *最新* 的新闻是 Azure 可带来更多的 Docker 好处：

- 可通过 Azure CLI [创建 Docker 主机](/documentation/articles/virtual-machines-linux-classic-cli-use-docker/)。
- 可与许多专有和开放源配置管理工具集成

而且由于可通过编程方式在 Azure 上创建 VM 和 Linux 容器，因此还可以使用 VM 和容器*协调*工具来创建多组虚拟机 (VM)，并在 Linux 容器和很快即将支持的 [Windows 容器](https://msdn.microsoft.com/zh-cn/virtualization/windowscontainers/about/about_overview)中部署应用程序。

本文不仅从较高水平讨论了这些概念，还包含了很多链接，可帮助你了解与 Azure 上的容器和群集使用相关的详细信息、教程和产品。如果你已了解全部内容，而只是需要这些链接，这些链接就在[使用容器的工具](#tools-for-working-with-containers)中。

## 虚拟机与容器之间的区别

虚拟机是在[虚拟机监控程序](http://zh.wikipedia.org/wiki/Hypervisor)提供的单独硬件虚拟化环境中运行。在 Azure 中，[虚拟机](/home/features/virtual-machines/)服务可为你处理所有事务：你只需选择操作系统并将其配置为按你所需方式运行（或上载你自己的自定义 VM 映像），由此创建虚拟机即可。虚拟机是经过时间考验、“久经沙场”的技术，可使用许多工具来管理操作系统并配置你所安装和运行的应用程序。虚拟机中运行的所有内容对主机操作系统隐藏，而且从虚拟机内运行的应用程序或用户的角度来看，虚拟机似乎是一个自治物理计算机。

[Linux 容器](http://en.wikipedia.org/wiki/LXC)中包括使用 docker 工具创建和托管的对象以及其他方法，不需要或不使用虚拟机监控程序来提供隔离。相反，容器主机会利用 Linux 内核的流程和文件系统隔离功能，仅向容器（及其应用程序）显示特定的内核功能和其自己的隔离文件系统（最低程度上）。从容器内运行的应用程序的角度来看，该容器似乎是一个独特的操作系统实例。包含在容器内的应用程序无法查看该容器以外的流程或其他任何资源。

因为在该隔离和执行模型中，Docker 主机计算机的内核是共享的，而且由于容器的磁盘要求现在不包括整个操作系统，所以容器的启动时间和所需的磁盘存储开销都非常非常少。

这一点很厉害。

但是，对于在 Windows 上运行的应用程序，Windows 容器可提供与 Linux 容器相同的好处。Windows 容器支持 Docker 映像格式和 Docker API，但还可以使用 PowerShell 进行管理。两种容器运行时随 Windows 容器、Windows Server 容器和 Hyper-V 容器提供。Hyper-V 容器通过在超级优化虚拟机中托管每个容器提供了附加隔离层。若要了解有关 Windows 容器的详细信息，请参阅[关于 Windows 容器](https://msdn.microsoft.com/zh-cn/virtualization/windowscontainers/about/about_overview)。若要在 Azure 中试用 Windows 容器，请参阅 [Windows 容器 Azure 快速入门](https://msdn.microsoft.com/virtualization/windowscontainers/quick_start/azure_setup)。

这一点也很厉害。

### 是不是好得让人不敢相信？

其实，既是又不是。容器和其他任何技术一样，无法像有魔法一般消除分布式应用程序所需的所有繁琐工作。但是同时，容器又的确改变了：

- 应用程序代码进行广泛开发和共享的速度
- 代码测试的速度和可信度
- 代码部署的速度和可信度

即便如此，请记住，容器是在容器主机（一种操作系统）上和 Azure（指 Azure 虚拟机）中执行。即使已经喜欢容器的设计理念，你仍然需要一个 VM 基础结构用于托管容器，但好处是容器可以在任意 VM 上运行（不过举例来说，容器是否希望 Linux 或 Windows 执行环境将具有重要意义）。

## 容器适合于哪些方面？

它们适用于许多情况，但鼓励（正如执行 [Azure 云服务](/home/features/cloud-services/)）创建单一服务、以[微服务]为导向的分布式应用程序，其中的应用程序设计是基于更多小规模的组合部件，而非基于更大规模、耦合更强的组件。

在公有云环境（例如 Azure）中尤其如此，你可随时随地租用 VM。你不仅能获得独立且快速的部署和协调工具，还能做出更高效的应用程序基础结构决策。

例如，当前你可能有一个包含 9 个大规模 Azure VM 的部署，适用于高度可用的分布式应用程序。如果此应用程序的组件可在容器中进行部署，则你可能只能使用 4 个 VM 并将你的应用程序组件部署在 20 个容器内，以实现冗余和负载平衡。

当然，这只是一个示例，但是如果你能在自己的方案中做到这一点，那么就能利用更多容器而非更多 Azure VM 来调整以适应用量尖峰，并能比从前更高效地使用剩余的总体 CPU 负载。

此外，还有许多方案不适合于微服务方法；你最了解微服务和容器是否能为你带来帮助。

### 容器为开发人员带来的好处

一般而言，很容易能看出容器技术代表了技术的进步，但还有其他更具体的优点好处。我们以 Docker 容器为例展开讨论。本主题不会立即深入探究 Docker（有关这部分内容，请参阅[什么是 Docker？](https://www.docker.com/whatisdocker/)，或查阅[wikipedia](http://wikipedia.org/wiki/Docker_%28software%29)），但 Docker 及其生态系统可为开发人员和 IT 专业人士带来巨大好处。

开发人员很快就会开始喜欢 Docker 容器，因为首先它有助于轻松使用 Linux 容器：

- 他们可以使用简单的增量命令来创建易于部署的固定映像，并可使用 dockerfile 自动构建这些映像。
- 通过对[公有](https://registry.hub.docker.com/)或[私有 docker 注册表](/documentation/articles/virtual-machines-linux-docker-registry-in-blob-storage/)使用简单的 [Git](https://git-scm.com/) 式推送和拉取命令，就可对这些映像进行共享。 
- 他们可以考虑用单独的应用程序组件来代替计算机
- 他们可以使用大量适合于 docker 容器的工具以及不同的基础映像。

### 容器可为运营和 IT 专业人士带来的好处

IT 和运营专家还可以从容器与虚拟机的组合中获益。

- 包含在容器内的服务独立于 VM 主机执行环境
- 包含在容器内的代码能证实是相同的
- 包含在容器内的服务可快速启动、停止并在开发、测试和生产环境之间快速移动

这些以及其他更多特性对大型企业具有很大的吸引力，这些企业中的专业信息技术组织的工作是将资源（包括纯粹的处理能力）合理分配到多项任务中，这些任务不仅是为了维持业务，更是要提高客户满意度并实现目标。小型企业、ISV 和初创企业也具有完全相同的要求，但他们对此的描述可能有所不同。

## 虚拟机适合于哪些方面？

虚拟机是云计算的支柱，这一点不会发生改变。如果虚拟机启动速度更慢、磁盘占用空间更大且不能直接映射到微服务体系结构，则它们具有非常重要的好处：

1. 默认情况下，它们可为主机计算机提供更稳健的默认安全保护
2. 它们支持任一种常用的操作系统和应用程序配置
3. 它们提供长久持续的工具生态系统以用于命令和控制
4. 它们提供执行环境以托管容器

最后一项很重要，因为包含在容器内的应用程序仍然需要特定的操作系统和 CPU 类型，具体取决于应用程序的调用。务必要牢记的一点是，在 VM 上安装容器是因为其中包含你希望部署的应用程序；容器不能代替 VM 或操作系统。

## VM 和容器的高级功能对比

下表从非常高的水平描述了 VM 与 Linux 容器之间存在的功能差异类型（没有太多额外工作）。请注意，一些功能可能或多或少都能满足需要，具体取决于你自己的应用程序需求，而且与所有软件一样，额外工作可提供增强的功能支持，在安全性方面尤其如此。

| 功能 | VM | 容器 |
| :------------- |-------------| ----------- |
| “默认”安全支持 | 达到更高程度 | 程度略低 |
| 所需的磁盘内存 | 完整操作系统加上应用 | 仅限应用要求 |
| 启动所需时间 | 明显更长：操作系统引导和应用程序加载 | 明显短很多：仅应用程序需要启动，因为内核已经在运行 |
| 可移植性 | 通过适当的准备实现可移植 | 在映像格式内可移植；通常更小 | 
| 映像自动化 | 相差很大，具体取决于操作系统和应用程序 | [Docker 注册表](https://registry.hub.docker.com/)；其他

## 创建和管理多组 VM 和容器

这时候，任何架构师、开发人员或 IT 运营专家可能会想“我可以自动完成所有这些事情；这真的是数据中心即服务！”。

你说得对，的确可以这样，而且有任意数量的系统，你可能已经使用了其中的许多系统，可管理 Azure VM 组和使用脚本（通常使用 CustomScriptingExtension for Windows 或 [CustomScriptingExtension for Linux](http://azure.microsoft.com/blog/2014/08/20/automate-linux-vm-customization-tasks-using-customscript-extension/)）注入自定义代码。你可以（可能已经）使用 PowerShell 或者 Azure CLI 实现 Azure 自动部署。

随后，这些功能通常迁移到 [Puppet](https://puppetlabs.com/) 和 [Chef](https://www.chef.io/) 等工具，以实现大规模的自动创建和配置 VM。（此处有一些链接指向[在 Azure 中使用这些工具](#tools-for-working-with-containers)的说明。）

### 部署和管理整个组的 Azure VM 和容器

有多个常用系统可部署整个组的 VM，并在其上安装 Docker（或其他 Linux 容器主机系统）作为可自动化组。有关直接链接，请参阅下文的[容器和工具](#containers-and-vm-technologies)节。有许多系统都或多或少能实现这一点，此列表不能尽录。根据你的技能组合和具体情况，它们可能有用，也可能不发挥作用。

Docker 有自己的 VM 创建工具集 ([docker-machine](/documentation/articles/virtual-machines-linux-docker-machine/)) 和一个负载平衡、docker-container 群集管理工具 ([swarm](/documentation/articles/virtual-machines-linux-docker-swarm/))。此外，[Azure Docker VM 扩展程序](https://github.com/Azure/azure-docker-extension/blob/master/README.md)附带了默认的 [`docker-compose`](https://docs.docker.com/compose/) 支持，此功能可在多个容器内部署已配置的应用程序容器。

另外，你还可以尝试使用 [Mesosphere 的数据中心操作系统 (DCOS)](http://docs.mesosphere.com/install/azurecluster/)。DCOS 基于开放源 [mesos](http://mesos.apache.org/)“分布式系统内核”，有助于你将数据中心视为一个可寻址服务进行处理。DCOS 拥有适合于多个重要系统（[Spark](http://spark.apache.org/) 和 [Kafka](http://kafka.apache.org/) 及其他）的内置程序包，以及 [Marathon](https://mesosphere.github.io/marathon/)（一种容器控制系统）和 [Chronos](https://mesosphere.com/blog/2015/04/30/chronos-2-3-3-released/)（一种分布式计划程序）等内置服务。Mesos 是借鉴了从 Twitter、AirBnb 和其他网络级企业学到的经验。

另外，[kubernetes](http://azure.microsoft.com/blog/2014/08/28/hackathon-with-kubernetes-on-azure) 是一种用于 VM 和容器组管理的开放源系统，借鉴了从 Google 学习的经验。你甚至可以将 [kubernetes 与 weave 配合使用，以提供网络支持](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/coreos/azure/README.md#kubernetes-on-azure-with-coreos-and-weave)。

[Deis](http://deis.io/overview/) 是一种开放源“平台即服务”(PaaS)，可帮助在你自己的服务器上轻松部署和管理应用程序。Deis 构建在 Docker 和 CoreOS 的基础之上，可提供轻型 PaaS，其中包含一个受 Heroku 启发的工作流。

[CoreOS](https://coreos.com/os/docs/latest/booting-on-azure.html) 是一种 Linux 分发，可提供优化的占用空间和 Docker 支持，且具有名为 [rkt](https://github.com/coreos/rkt) 的自有容器系统，还有一个名为 [fleet](https://coreos.com/using-coreos/clustering/) 的容器组管理工具。

Ubuntu 是另一个非常受欢迎的 Linux 分发，可以非常好地支持 Docker，还支持 [Linux（LXC 式）群集](https://help.ubuntu.com/lts/serverguide/lxc.html)。

##<a name="tools-for-working-with-containers"></a> 使用 Azure VM 和容器所需的工具

使用容器和 Azure VM 时需借助工具。本节仅列出了部分与容器和组相关的最有用或最重要的概念和工具，以及与其搭配使用的更大配置和协调工具。

> [AZURE.NOTE]此领域正迅速发生着令人惊奇的变化，尽管我们尽最大努力使本主题及其中的链接保持最新，依然无法做到详尽。请确保搜索有兴趣的科目，以便掌握最新动态！

###<a name="containers-and-vm-technologies"></a> 容器和 VM 技术

部分 Linux 容器技术：

- [Docker](https://www.docker.com)
- [LXC](https://linuxcontainers.org/)
- [CoreOS 和 rkt](https://github.com/coreos/rkt)
- [开放容器项目](http://opencontainers.org/)
- [RancherOS](http://rancher.com/rancher-os/)

Windows 容器链接：

- [Windows 容器](https://msdn.microsoft.com/zh-cn/virtualization/windowscontainers/about/about_overview)

Visual Studio Docker 链接：

- [Visual Studio 2015 RC Tools for Docker - 预览版](https://visualstudiogallery.msdn.microsoft.com/6f638067-027d-4817-bcc7-aa94163338f0)

Docker 工具：

- [Docker 后台程序](https://docs.docker.com/installation/#installation)
- Docker 客户端
	- [Windows Docker Client on Chocolatey](https://chocolatey.org/packages/docker)
	- [Docker 安装说明](https://docs.docker.com/installation/#installation)


Azure 上的 Docker：

- [适用于 Azure 上 的 Linux 的 Docker VM 扩展](/documentation/articles/virtual-machines-linux-dockerextension/)
- [Azure Docker VM 扩展用户指南](https://github.com/Azure/azure-docker-extension/blob/master/README.md)
- [从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展](/documentation/articles/virtual-machines-linux-classic-cli-use-docker/)
- [如何在 Azure 上使用 docker-machine](/documentation/articles/virtual-machines-linux-docker-machine/)
- [如何在 Azure 上将 docker 与 swarm 一起使用](/documentation/articles/virtual-machines-linux-docker-swarm/)
- [在 Azure 上使用 Docker 和 Compose 入门](/documentation/articles/virtual-machines-linux-docker-compose-quickstart/)
- [对包含在容器内的应用程序的 `compose`](https://github.com/Azure/azure-docker-extension#11-public-configuration-keys) 提供内置支持
- [在 Azure 上实施 Docker 专有注册表](/documentation/articles/virtual-machines-linux-docker-registry-in-blob-storage/)

Linux 分发和 Azure 示例：

- [CoreOS](https://coreos.com/os/docs/latest/booting-on-azure.html)

配置、群集管理和容器协调：

- [CoreOS 上的 Fleet](https://coreos.com/using-coreos/clustering/)

-	Kubernetes
	- [使用 CoreOS 和 Weave 实现 Kubernetes 群集部署自动化的完整指南](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/coreos/azure/README.md#kubernetes-on-azure-with-coreos-and-weave)
	- [Kubernetes 可视化工具](http://azure.microsoft.com/blog/2014/08/28/hackathon-with-kubernetes-on-azure)
	
-	[Mesos](http://mesos.apache.org/)
	-	[Mesosphere 的数据中心操作系统 (DCOS)](http://beta-docs.mesosphere.com/install/azurecluster/)
	
-	[Jenkins](https://jenkins-ci.org/) 和 [Hudson](http://hudson-ci.org/)
	- [博客：适用于 Azure 的 Jenkins Slave 插件](http://msopentech.com/blog/2014/09/23/announcing-jenkins-slave-plugin-azure/)
	- [GitHub repo：适用于 Azure 的 Jenkins Storage 插件](https://github.com/jenkinsci/windows-azure-storage-plugin)
	- [第三方：适用于 Azure 的 Hudson Slave 插件](http://wiki.hudson-ci.org/display/HUDSON/Azure+Slave+Plugin)
	- [第三方：适用于 Azure 的 Hudson Storage 插件](https://github.com/hudson3-plugins/windows-azure-storage-plugin)
	
-	[Chef](https://docs.chef.io/index.html)
	- [Chef 和虚拟机](/documentation/articles/virtual-machines-windows-extensions-features/)
	- [视频：Chef 是什么及其工作原理](https://msopentech.com/blog/2014/03/31/using-chef-to-manage-azure-resources/)

-	[Azure 自动化](/home/features/automation/)
	
	
-	Powershell DSC for Linux
    - [博客：如何执行 Powershell DSC for Linux](http://blogs.technet.com/b/privatecloud/archive/2014/05/19/powershell-dsc-for-linux-step-by-step.aspx)
    - [GitHub：Docker 客户端 DSC](https://github.com/anweiss/DockerClientDSC)

## 后续步骤

了解 [Docker](https://www.docker.com) 和 [Windows 容器](https://msdn.microsoft.com/zh-cn/virtualization/windowscontainers/about/about_overview)。

<!--Anchors-->
[microservices]: http://martinfowler.com/articles/microservices.html
[微服务]: http://martinfowler.com/articles/microservices.html
<!--Image references-->
