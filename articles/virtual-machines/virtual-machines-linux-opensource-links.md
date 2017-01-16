<properties
	pageTitle="Azure 上的 Linux 和开源计算 | Azure"
	description="列出 Azure 上的 Linux 和开源计算文章，包括基本的 Linux 用法，关于在 Azure 上运行或上载 Linux 映像的一些基本概念，以及关于特定技术或优化的其他内容。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.devlang="NA"
	ms.topic="article"
	ms.tgt_pltfrm="vm-linux"
	ms.workload="infrastructure-services"
	ms.date="06/27/2016"
	wacn.date="12/12/2016"
	ms.author="rasquill"/>



# Azure 上的 Linux 和开源计算

查找在经典部署模型中创建和管理基于 Linux 的虚拟机所需的所有文档。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-classic-include.md)]

## 入门
- [Azure 上的 Linux 简介](/documentation/articles/virtual-machines-linux-intro-on-azure/)
- [使用经典部署模型创建 Azure 虚拟机的常见问题](/documentation/articles/virtual-machines-linux-classic-faq/)
- [关于虚拟机的映像](/documentation/articles/virtual-machines-linux-classic-about-images/)
- [上载自己的发行版映像](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)（以及使用 [Azure 认可的发行版](/documentation/articles/virtual-machines-linux-endorsed-distros/)的说明）
- [使用 Azure 经典管理门户登录到 Linux VM](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

## 设置

- [安装 Azure 命令行接口 (Azure CLI)](/documentation/articles/xplat-cli-install/)


## 教程

- [在 Azure 中的 Linux 虚拟机上安装 LAMP 堆栈](/documentation/articles/virtual-machines-linux-create-lamp-stack/)
- [Azure 虚拟机上的 Ruby on Rails Web 应用程序](/documentation/articles/virtual-machines-linux-classic-ruby-rails-web-app/)
- [如何：为 AMQP 和服务总线安装 Apache Qpid Proton-C](/documentation/articles/service-bus-amqp-apache/)

### 数据库
- [优化 Azure 上的 MySQL 的性能](/documentation/articles/virtual-machines-linux-classic-optimize-mysql/)
- [MySQL 群集](/documentation/articles/virtual-machines-linux-classic-mysql-cluster/)
- [在 Azure 上同时运行 Cassandra 和 Linux 并通过 Node.js 访问](/documentation/articles/virtual-machines-linux-classic-cassandra-nodejs/)
- [创建多主的 MariaDb 群集](/documentation/articles/virtual-machines-linux-classic-mariadb-mysql-cluster/)

### Ubuntu
- [如何：MySQL 群集](/documentation/articles/virtual-machines-linux-classic-mysql-cluster/)
- [如何：Node.js 和 Cassandra](/documentation/articles/virtual-machines-linux-classic-cassandra-nodejs/)

### OpenSUSE
- [如何：安装和运行 MySQL](/documentation/articles/virtual-machines-linux-classic-mysql-on-opensuse/)

### CoreOS
- [如何：在 Azure 上使用 CoreOS](https://coreos.com/os/docs/latest/booting-on-azure.html)

## 规划
- [Azure 基础结构服务实施准则](/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/)
- [选择 Linux 用户名](/documentation/articles/virtual-machines-linux-usernames/)
- [如何在经典部署模型中为虚拟机配置可用性集](/documentation/articles/virtual-machines-linux-classic-configure-availability/)
- [如何在 Azure VM 上安排计划内的维护](/documentation/articles/virtual-machines-linux-planned-maintenance-schedule/)
- [管理虚拟机的可用性](/documentation/articles/virtual-machines-linux-manage-availability/)
- [Azure 中 Linux 虚拟机的计划内维护](/documentation/articles/virtual-machines-linux-planned-maintenance/)


## 部署
- [创建运行 Linux 的自定义虚拟机](/documentation/articles/virtual-machines-linux-classic-createportal/)
- [基础知识：捕获 Linux VM 以创建模板](/documentation/articles/virtual-machines-linux-classic-capture-image/)
- [有关未认可发行版的信息](/documentation/articles/virtual-machines-linux-create-upload-generic/)


## 管理

- [SSH](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)
- [如何为 Linux 重置密码或 SSH 属性](/documentation/articles/virtual-machines-linux-classic-reset-access/)
- [使用 Root](/documentation/articles/virtual-machines-linux-use-root-privileges/)


## Azure 资源

- [Azure Linux 代理](/documentation/articles/virtual-machines-linux-agent-user-guide/)
- [Azure VM 扩展和功能](/documentation/articles/virtual-machines-windows-extensions-features/)
- [将自定义数据注入到 VM 中以用于 Cloud-init](/documentation/articles/virtual-machines-windows-classic-inject-custom-data/)


## 存储

- [将数据磁盘附加到 Linux VM](/documentation/articles/virtual-machines-linux-classic-attach-disk/)
- [从 Linux VM 分离数据磁盘](/documentation/articles/virtual-machines-linux-classic-detach-disk/)
- [RAID](/documentation/articles/virtual-machines-linux-configure-raid/)


## 网络
- [如何在 Azure 中的经典虚拟机上设置终结点](/documentation/articles/virtual-machines-linux-classic-setup-endpoints/)


## 故障排除
- [对基于 Linux 的 Azure 虚拟机的 Secure Shell (SSH) 连接进行故障排除](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection/)


## 引用

- [Azure 服务管理 (asm) 模式下的 Azure CLI 命令](/documentation/articles/virtual-machines-command-line-tools/)
- [Azure 服务管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)




## 常规链接
下面是 Microsoft 博客、Technet 页面和外部站点（而非上述 Azure.com 文档）的链接。由于 Azure 和开源计算都在快速发展，因此几乎可以肯定以下链接都已过时，*尽管*我们将尽最大努力继续添加更新的主题并删除过时的主题。如果我们有遗漏，请在评论中告诉我们，或者向我们的 [GitHub 存储库](https://github.com/wacn/techcontent)提交拉取请求。

- [使用 Docker 容器在 Linux 上运行 ASP.NET 5](http://blogs.msdn.com/b/webdev/archive/2015/01/14/running-asp-net-5-applications-in-linux-containers-with-docker.aspx)
- [如何从 OpenLogic 部署 CentOS VM 映像](https://azure.microsoft.com/blog/2013/01/11/deploying-openlogic-centos-images-on-windows-azure-virtual-machines/)
- [SUSE 更新基础结构](https://forums.suse.com/showthread.php?5622-New-Update-Infrastructure)
- [通过 12 个步骤在 Azure 上构建具有高可用性的 Linux](http://blogs.technet.com/b/keithmayer/archive/2014/10/03/quick-start-guide-building-highly-available-linux-servers-in-the-cloud-on-microsoft-azure.aspx)
- [使用 Azure CLI、node.js、jhawk 在 Azure 上自动预配 Linux](http://blogs.technet.com/b/keithmayer/archive/2014/11/24/step-by-step-automated-provisioning-for-linux-in-the-cloud-with-microsoft-azure-xplat-cli-json-and-node-js-part-1.aspx)

### FreeBSD
- [在 Azure 中运行 FreeBSD](https://azure.microsoft.com/blog/2014/05/22/running-freebsd-in-azure/)
- [轻松部署 FreeBSD](http://msopentech.com/blog/2014/10/24/easy-deploy-freebsd-microsoft-azure-vm-depot/)
- [部署自定义的 FreeBSD 映像](http://msopentech.com/blog/2014/05/14/deploy-customize-freebsd-virtual-machine-image-microsoft-azure/)

### NoSQL

- [Slideshare (MSOpenTech)：在 Azure 上体验 CouchDb](http://www.slideshare.net/brianbenz/experiences-using-couchdb-inside-microsofts-azure-team)
- [使用 node.js、CORS 和 Grunt 运行 CouchDB 即服务](http://msopentech.com/blog/2013/12/19/tutorial-building-multi-tier-windows-azure-web-application-use-cloudants-couchdb-service-node-js-cors-grunt-2/)

- [Azure Redis Cache Service 中的 Redis on Windows](http://msopentech.com/blog/2014/05/12/redis-on-windows/)
- [推出适用于 Redis 预览版的 ASP.NET 会话状态提供程序](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)

- [博客：现已在 Azure 库中推出 RavenHQ](https://azure.microsoft.com/blog/2014/08/12/ravenhq-now-available-in-the-azure-store/)

### 大数据
- [在 Azure Linux VM 上安装 Hadoop](http://blogs.msdn.com/b/benjguin/archive/2013/04/05/how-to-install-hadoop-on-windows-azure-linux-virtual-machines.aspx)

### 关系数据库
- [Azure 中的 MySQL 高可用性体系结构](http://download.microsoft.com/download/6/1/C/61C0E37C-F252-4B33-9557-42B90BA3E472/MySQL_HADR_solution_in_Azure.pdf)
- [使用 ILB 通过 corosync、pg\_bouncer 安装 Postgres](https://github.com/chgeuer/postgres-azure)

### 开发、管理和优化

进行开发、管理和优化涉及的方面十分广泛，而且各种情况变化很快，因此应将以下列表看作是学习的起点。

- [视频：Azure 虚拟机：使用 Chef、Puppet 和 Docker 管理 Linux VM](https://azure.microsoft.com/blog/2014/12/15/azure-virtual-machines-using-chef-puppet-and-docker-for-managing-linux-vms/)

- [使用 CoreOS 和 Weave 自动部署 Kubernetes 群集的完整指南](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/coreos/azure/README.md#kubernetes-on-azure-with-coreos-and-weave)
- [Kubernetes Visualizer](https://azure.microsoft.com/blog/2014/08/28/hackathon-with-kubernetes-on-azure/)

- [适用于 Azure 的 Jenkins Slave 插件](http://msopentech.com/blog/2014/09/23/announcing-jenkins-slave-plugin-azure/)
- [GitHub repo：适用于 Azure 的 Jenkins Storage 插件](https://github.com/jenkinsci/windows-azure-storage-plugin)

- [第三方：适用于 Azure 的 Hudson Slave 插件](http://wiki.hudson-ci.org/display/HUDSON/Azure+Slave+Plugin)
- [第三方：适用于 Azure 的 Hudson Storage 插件](https://github.com/hudson3-plugins/windows-azure-storage-plugin)

- [视频：Chef 是什么及其工作原理](https://msopentech.com/blog/2014/03/31/using-chef-to-manage-azure-resources/)

- [视频：如何在 Linux VM 上使用 Azure 自动化](http://channel9.msdn.com/Shows/Azure-Friday/Azure-Automation-104-managing-Linux-and-creating-Modules-with-Joe-Levy)

- [博客：如何执行 Powershell DSC for Linux](http://blogs.technet.com/b/privatecloud/archive/2014/05/19/powershell-dsc-for-linux-step-by-step.aspx)
- [GitHub：Docker 客户端 DSC](https://github.com/anweiss/DockerClientDSC)

- [Ubuntu Juju](https://juju.ubuntu.com/docs/config-azure.html)

- [适用于 Azure 的打包程序插件](https://github.com/msopentech/packer-azure)

<!---HONumber=Mooncake_Quality_Review_1118_2016-->