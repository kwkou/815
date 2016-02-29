<properties
	pageTitle="Azure 上的 Linux 和开源计算"
	description="列出 Azure 上的 Linux 和开源计算文章，包括基本的 Linux 用法、一些关于在 Azure 上运行或上载 Linux 映像的基本概念，以及关于特定技术或优化的其他内容。"
	services="virtual-machines"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="01/03/2016"
	wacn.date=""/>



# Azure 上的 Linux 和开源计算

本文档尝试在一个位置列出由 Microsoft 及其合作伙伴编写的关于在 Microsoft Azure 上运行基于 Linux 的虚拟机以及其他开源计算环境和应用程序的所有主题。

因为 Azure 和开源计算都在快速发展，所以几乎可以肯定地说，本文档已过时了，*尽管*我们将尽最大努力来持续添加更新的主题并删除过时的主题。如果我们有遗漏，请在评论中告诉我们，或者向我们的 [GitHub 存储库](https://github.com/Azure/azure-content/)提交一个拉取请求。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]


## 一般说明
在本页的右侧对各个部分进行了拆分。链接可能会出现在多个部分中，因为主题可能是关于多个概念、发行版或技术的。 此外，有多个主题介绍了各种 Linux 选项、映像存储库、案例研究以及关于如何上载你自己的自定义映像的操作指南主题：

- [MSOpenTech VM Depot](https://vmdepot.msopentech.com/List/Index)
- [事件和演示：Microsoft 开放性 CEE](http://www.opennessatcee.com/)
- [如何：上载自己的发行版映像](/documentation/articles/virtual-machines-linux-create-upload-vhd)（以及使用 [Azure 认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distributions)的说明）
- [注意：在 Azure 中运行时的一般 Linux 要求](/documentation/articles/virtual-machines-linux-create-upload-vhd-generic)
- [注意：Azure 上的 Linux 的一般简介](/documentation/articles/virtual-machines-linux-introduction)


## 发行版

Linux 有大量的分发，通常按包管理系统进行划分：有些是基于 dpkg 的，如 Debian 和 Ubuntu，还有一些是基于 rpm 的，如 CentOS、SUSE 和 RedHat。有些公司作为 Microsoft 的正式合作伙伴也提供了发行版映像并且是经认可的。其他一些是由社区提供的。本部分中的发行版具有关于它们的正式文章，即使它们只是在其他技术的示例中使用也是如此。

### Ubuntu

Ubuntu 是一种非常流行的经 Azure 认可的 Linux 分发，它基于 dpkg 和 apt-get 包管理系统。

1. [如何：上载自己的 Ubuntu 映像](/documentation/articles/virtual-machines-linux-create-upload-vhd-ubuntu)
2. [如何：Ubuntu LAMP 堆栈](/documentation/articles/virtual-machines-linux-install-lamp-stack)
3. [如何：MySQL 群集](/documentation/articles/virtual-machines-linux-mysql-cluster)
4. [如何：Node.js 和 Cassandra](/documentation/articles/virtual-machines-linux-nodejs-running-cassandra)
6. [钻研：使用 Docker 容器在 Linux 上运行 ASP.NET 5](http://blogs.msdn.com/b/webdev/archive/2015/01/14/running-asp-net-5-applications-in-linux-containers-with-docker.aspx)


### [Debian](https://vmdepot.msopentech.com/List/Index?sort=Featured&search=Debian)

对于 Linux 和基于 dpkg 和 apt-get 包管理系统的开源世界来说，Debian 是一个重要的分发。MSOpenTech VM Depot 有多个可使用的映像。

### CentOS

CentOS Linux 分发是从 Red Hat Enterprise Linux (RHEL) 的源代码派生的一个稳定的、可预测、可管理且可重现的平台。

1. [MSOpenTech VM Depot](https://vmdepot.msopentech.com/List/Index?sort=Featured&search=centos)
3. [如何：为 Azure 准备自定义的基于 CentOS 的 VM](/documentation/articles/virtual-machines-linux-create-upload-vhd-centos)
4. [博客：如何从 OpenLogic 部署 CentOS VM 映像](http://azure.microsoft.com/blog/2013/01/11/deploying-openlogic-centos-images-on-windows-azure-virtual-machines/)
6. [如何：为 AMQP 和服务总线安装 Apache Qpid Proton-C](/documentation/articles/service-bus-amqp-apache)

### SUSE Linux Enterprise Server 和 openSUSE

9. [MSOpenTech VM Depot](https://vmdepot.msopentech.com/List/Index?sort=Featured&search=OpenSUSE)
11. [如何：安装和运行 MySQL](/documentation/articles/virtual-machines-linux-mysql-use-opensuse)
12. [如何：准备自定义 SLES 或 openSUSE VM](/documentation/articles/virtual-machines-linux-create-upload-vhd-suse)  
13. [[SUSE 论坛] 如何：移动到新的修补程序服务器](https://forums.suse.com/showthread.php?5622-New-Update-Infrastructure)

### CoreOS

CoreOS 是一个小型的经优化的发行版，适用于具有高度的自定义控制的纯计算规模。

11. [何：在 Azure 上使用 CoreOS](/documentation/articles/virtual-machines-linux-coreos-how-to)
12. [如何：开始在 Azure 上的 CoreOS 上使用 Fleet 和 Docker](/documentation/articles/virtual-machines-linux-coreos-fleet-get-started)


### Oracle Linux
  2. [为 Azure 准备 Oracle Linux 虚拟机](/documentation/articles/virtual-machines-linux-create-upload-vhd-oracle)

### FreeBSD

12. [MSOpenTech VM Depot](https://vmdepot.msopentech.com/List/Index?sort=Date&search=FreeBSD)
13. [博客：在 Azure 中运行 FreeBSD](http://azure.microsoft.com/blog/2014/05/22/running-freebsd-in-azure/)
14. [博客：轻松部署 FreeBSD](http://msopentech.com/blog/2014/10/24/easy-deploy-freebsd-microsoft-azure-vm-depot/)
15. [博客：部署自定义的 FreeBSD 映像](http://msopentech.com/blog/2014/05/14/deploy-customize-freebsd-virtual-machine-image-microsoft-azure/)
17. [如何：安装 Azure Linux 代理](/documentation/articles/virtual-machines-linux-agent-user-guide)

## 基础知识

1. [基础知识：Azure 命令行界面 (Azure CLI)](/documentation/articles/xplat-cli-install)
<!-- 4. [The basics: Certificate Use and Management](http://msdn.microsoft.com/zh-cn/library/azure/gg981929.aspx)-->
5. [基础知识：选择 Linux 用户名](/documentation/articles/virtual-machines-linux-usernames)
6. [基础知识：使用 Azure 管理门户登录到 Linux VM](/documentation/articles/virtual-machines-linux-how-to-log-on)
7. [基础知识：SSH](/documentation/articles/virtual-machines-linux-use-ssh-key)
8. [基础知识：如何为 Linux 重置密码或 SSH 属性](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh)
9. [基础知识：使用 Root](/documentation/articles/virtual-machines-linux-use-root-privileges)
10. [基础知识：将数据磁盘附加到 Linux VM](/documentation/articles/virtual-machines-linux-how-to-attach-disk)
11. [基础知识：从 Linux VM 分离数据磁盘](/documentation/articles/virtual-machines-linux-how-to-detach-disk)
12. [博客中的基础知识：优化使用 Linux 和 Azure 时的存储、磁盘和性能](http://blogs.msdn.com/b/igorpag/archive/2014/10/23/azure-storage-secrets-and-linux-i-o-optimizations.aspx)
13. [基础知识：RAID](/documentation/articles/virtual-machines-linux-configure-raid)
14. [基础知识：捕获 Linux VM 来生成模板](/documentation/articles/virtual-machines-linux-capture-image)
15. [基础知识：Azure Linux 代理](/documentation/articles/virtual-machines-linux-agent-user-guide)
16. [基础知识：Azure VM 扩展和功能](/documentation/articles/virtual-machines-extensions-features)
17. [基础知识：将自定义数据注入到 VM 中以用于 Cloud-init](/documentation/articles/virtual-machines-how-to-inject-custom-data)
18. [博客中的基础知识：通过 12 个步骤在 Azure 上构建高度可用的 Linux](http://blogs.technet.com/b/keithmayer/archive/2014/10/03/quick-start-guide-building-highly-available-linux-servers-in-the-cloud-on-microsoft-azure.aspx)
19. [博客中的基础知识：使用 Azure CLI、node.js、jhawk 在 Azure 上自动预配 Linux](http://blogs.technet.com/b/keithmayer/archive/2014/11/24/step-by-step-automated-provisioning-for-linux-in-the-cloud-with-microsoft-azure-xplat-cli-json-and-node-js-part-1.aspx)
20. [基础知识：Azure Docker VM 扩展](/documentation/articles/virtual-machines-docker-vm-extension)
23. [Azure 服务管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx) 参考
24. [Azure 上的 GlusterFS](http://dastouri.azurewebsites.net/gluster-on-azure-part-1/)

## 社区映像和存储库
3. [MSOpenTech VM Depot](https://vmdepot.msopentech.cn/List/Index) &mdash; 适用于社区提供的虚拟机映像。
4. [GitHub](https://github.com/Azure/) &mdash; 适用于 Azure CLI 以及许多其他工具和项目。
5. [Docker Hub 注册表](https://registry.hub.docker.com/) &mdash; Docker 容器映像的注册表。

## 语言和平台
### [Azure Java 开发人员中心](/develop/java/)

1. [映像](https://vmdepot.msopentech.com/List/Index?sort=Featured&search=java)
2. [如何：使用 AMQP 1.0 通过 Java 使用 Service Bus](/documentation/articles/service-bus-amqp-java)
3. [如何：使用 Azure 管理门户在 Linux 上设置 Tomcat7](/documentation/articles/virtual-machines-linux-setup-tomcat7-linux)
4. [视频：Azure Java SDK for Service Management](http://channel9.msdn.com/Shows/Cloud+Cover/Episode-157-The-Java-SDK-for-Azure-Management-with-Brady-Gaster)
5. [博客：适用于 Java 的 Azure 管理库入门](http://azure.microsoft.com/blog/2014/09/15/getting-started-with-the-azure-java-management-libraries/)
5. [GitHub 存储库：适用于 Eclipse with Java 的 Azure 工具包](https://github.com/MSOpenTech/WindowsAzureToolkitForEclipseWithJava)
6. [参考：适用于 Eclipse with Java 的 Azure 工具包](/documentation/articles/azure-toolkit-for-eclipse)
7. [GitHub 存储库：适用于 IntelliJ IDEA 和 Android Studio 的 MS Open Tech Tools 插件](https://github.com/MSOpenTech/msopentech-tools-for-intellij)
7. [博客：MSOpenTech 对 OpenJDK 做出了贡献](http://msopentech.com/blog/2014/10/21/ms-open-techs-first-contribution-openjdk/)

### JVM 语言

1. [Scala：在 Azure 云服务中运行 Play Framework 应用程序](http://msopentech.com/blog/2014/09/25/tutorial-running-play-framework-applications-microsoft-azure-cloud-services-2/)

### SDK 类型、安装和升级
4. [Azure 服务管理 SDK：Java](http://azure.github.io/azure-sdk-for-java/)
5. [Azure 服务管理 SDK：Go](https://github.com/MSOpenTech/azure-sdk-for-go)
5. [Azure 服务管理 SDK：Ruby](https://github.com/MSOpenTech/azure-sdk-for-ruby)
    - [如何：在 Rails 上安装 Ruby](/documentation/articles/virtual-machines-ruby-rails-web-app-linux)
6. [Azure 服务管理 SDK：Python](https://github.com/Azure/azure-sdk-for-python)
    - [如何：Django Hello World Web 应用程序 (Mac-Linux)](/documentation/articles/virtual-machines-python-django-web-app-linux)
7. [Azure 服务管理 SDK：Node.js](https://github.com/MSOpenTech/azure-sdk-for-node)
8. [Azure 服务管理 SDK：PHP](https://github.com/MSOpenTech/azure-sdk-for-php)
    - [如何：在 Azure VM 上安装 LAMP 堆栈](/documentation/articles/virtual-machines-linux-install-lamp-stack)
    - [视频：在 Azure VM 上安装 LAMP 堆栈](http://channel9.msdn.com/Shows/Azure-Friday/LAMP-stack-on-Azure-VMs-with-Guy-Bowerman)
9. [Azure 服务管理 SDK：.NET](https://github.com/Azure/azure-sdk-for-net)
10. [博客：Mono、ASP.NET 5、Linux 和 Docker](http://blogs.msdn.com/b/webdev/archive/2015/01/14/running-asp-net-5-applications-in-linux-containers-with-docker.aspx)

## 示例和脚本

可查找此部分来快速填写。如果你有任何建议，请向我们发送 PR 或者在下文中的意见部分留言。

2. [Patrick Chanezon 的 Azure Linux GitHub 存储库](https://github.com/chanezon/azure-linux)


## 数据

本部分包含几种不同的存储方法和技术（包括非 SQL、关系型和大数据）的相关信息。

### NoSQL

1. [博客：8 种适用于 Azure 的开源非 SQL 数据库](http://openness.microsoft.com/blog/2014/11/03/open-source-nosql-databases-microsoft-azure/)
2. Couchdb
    - [Slideshare (MSOpenTech)：在 Azure 上体验 CouchDb](http://www.slideshare.net/brianbenz/experiences-using-couchdb-inside-microsofts-azure-team)
    - [博客：使用 node.js、CORS 和 Grunt 运行 CouchDB 即服务](http://msopentech.com/blog/2013/12/19/tutorial-building-multi-tier-windows-azure-web-application-use-cloudants-couchdb-service-node-js-cors-grunt-2/)
3. MongoDB
    - [如何：使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js 应用程序](/documentation/articles/web-sites-dotnet-store-data-mongodb-vm)
4. Cassandra
    - [如何：在 Azure 上将 Cassandra 与 Linux 一起运行以及通过 Node.js 对其进行访问](/documentation/articles/virtual-machines-linux-nodejs-running-cassandra)
5. Redis
    - [博客：Azure Redis 缓存服务中 Windows 上的 Redis](http://msopentech.com/blog/2014/05/12/redis-on-windows/)
    - [博客：宣布推出适用于 Redis 预览版的 ASP.NET 会话状态提供程序](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)

### 大型数据
2. Hadoop/Cloudera  
	- [博客：在 Azure Linux VM 上安装 Hadoop](http://blogs.msdn.com/b/benjguin/archive/2013/04/05/how-to-install-hadoop-on-windows-azure-linux-virtual-machines.aspx)
3. [Azure HDInsight](/documentation/services/hdinsight/) -- Azure 上的一项完全托管的 Hadoop 服务。

### 关系数据库
2. MySQL
    - [如何：安装和运行 MySQL](/documentation/articles/virtual-machines-linux-mysql-use-opensuse)
    - [如何：优化 Azure 上的 MySQL 的性能](/documentation/articles/virtual-machines-linux-optimize-mysql-perf)
    - [如何：MySQL 群集](/documentation/articles/virtual-machines-linux-mysql-cluster)
    - [如何：使用应用商店创建 MySQL 数据库](/documentation/articles/store-php-create-mysql-database)
    - [如何：具有 Python 和 Visual Studio 的 Azure Web 应用上的 Django 和 MySQL](/documentation/articles/web-sites-python-ptvs-django-mysql)
    - [Microsoft Azure 中的 MySQL 高可用性体系结构](http://download.microsoft.com/download/6/1/C/61C0E37C-F252-4B33-9557-42B90BA3E472/MySQL_HADR_solution_in_Azure.pdf)
7. MariaDB
    - [如何：创建多主的 MariaDb 群集](/documentation/articles/virtual-machines-mariadb-cluster)
8. [使用 ILB 通过 corosync、pg\_bouncer 安装 Postgres](https://github.com/chgeuer/postgres-azure)


## 身份验证和加密

身份验证和加密是软件开发中的重要主题，并且网上有很多很多主题介绍了如何针对这两者使用合适的安全技术。我们将介绍一些基本的用法来快速启动和运行 Linux 和开源工作负荷，并指出用来重置或删除 Azure 上的远程安全功能的工具。这些是基本过程，我们很快将添加更复杂的方案。

4. [基础知识：证书的使用和管理](http://msdn.microsoft.com/zh-cn/library/azure/gg981929.aspx)
7. [基础知识：SSH](/documentation/articles/virtual-machines-linux-use-ssh-key)
8. [基础知识：如何为 Linux 重置密码或 SSH 属性](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh)
9. [基础知识：使用 Root](/documentation/articles/virtual-machines-linux-use-root-privileges)

## Linux 高性能计算 (HPC)

4.	[教程：Azure 的 HPC Pack 群集中的 Linux 计算节点入门](/documentation/articles/virtual-machines-linux-cluster-hpcpack)
6.	[教程：设置 Linux RDMA 群集以运行 MPI 应用程序](/documentation/articles/virtual-machines-linux-cluster-rdma)


## 开发运营、管理和优化

本部分开头介绍了一个博客条目，其中包含一系列视频 -- [视频：Azure 虚拟机：使用 Chef、Puppet 和 Docker 管理 Linux VM](http://azure.microsoft.com/blog/2014/12/15/azure-virtual-machines-using-chef-puppet-and-docker-for-managing-linux-vms/)。然而，进行开发、管理和优化涉及的面十分广泛，而且各种情况变化很快，因此在一开始的时候，你可以考虑将下表作为学习的目标。

1. Docker
	- [适用于 Azure 上 的 Linux 的 Docker VM 扩展](/documentation/articles/virtual-machines-docker-vm-extension)
	- [从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展](/documentation/articles/virtual-machines-docker-with-xplat-cli)
	- [如何在 Azure 上使用 docker-machine](/documentation/articles/virtual-machines-docker-machine)
	- [如何在 Azure 上将 docker 与 swarm 一起使用](/documentation/articles/virtual-machines-docker-swarm)
	- [在 Azure 上使用 Docker 和 Compose 入门](/documentation/articles/virtual-machines-docker-compose-quickstart)

2. [在 CoreOS 上使用Fleet](/documentation/articles/virtual-machines-linux-coreos-how-to)
3. Deis
	- [GitHub 存储库：在 Azure 中的 CoreOS 群集上安装 Deis](https://github.com/chanezon/azure-linux/tree/master/coreos/deis)
4. Kubernetes
	- [使用 CoreOS 和 Weave 实现 Kubernetes 群集部署自动化的完整指南](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/coreos/azure/README.md#kubernetes-on-azure-with-coreos-and-weave)
	- [Kubernetes 可视化工具](http://azure.microsoft.com/blog/2014/08/28/hackathon-with-kubernetes-on-azure)
5. Jenkins 和 Hudson
	- [博客：适用于 Azure 的 Jenkins Slave 插件](http://msopentech.com/blog/2014/09/23/announcing-jenkins-slave-plugin-azure/)
	- [GitHub repo：适用于 Azure 的 Jenkins Storage 插件](https://github.com/jenkinsci/windows-azure-storage-plugin)
	- [第三方：适用于 Azure 的 Hudson Slave 插件](http://wiki.hudson-ci.org/display/HUDSON/Azure+Slave+Plugin)
	- [第三方：适用于 Azure 的 Hudson Storage 插件](https://github.com/hudson3-plugins/windows-azure-storage-plugin)
10. Chef
	- [Chef 和虚拟机](/documentation/articles/virtual-machines-automation-with-chef)

13. Powershell DSC for Linux
    - [博客：如何执行 Powershell DSC for Linux](http://blogs.technet.com/b/privatecloud/archive/2014/05/19/powershell-dsc-for-linux-step-by-step.aspx)
    - [GitHub：Docker 客户端 DSC](https://github.com/anweiss/DockerClientDSC)
13. [Ubuntu Juju](https://juju.ubuntu.com/docs/config-azure.html)
14. [适用于 Azure 的打包程序插件](https://github.com/msopentech/packer-azure)

## 支持、故障排除以及“它就是不工作”

1. Microsoft 支持文档
	- [支持：支持 Microsoft Azure 上的 Linux 映像](http://support2.microsoft.com/kb/2941892)

<!--Anchors-->
[Distros]: #distros
[The Basics]: #basics
[Community Images and Repositories]: #images
[Languages and Platforms]: #langsandplats
[Samples and Scripts]: #samples
[Auth and Encryption]: #security
[Devops, Management, and Optimization]: #devops
[Support, Troubleshooting, and "It Just Doesn't Work"]: #supportdebug

<!--Link references--In actual articles, you only need a single period before the slash. -->
[How to use docker-machine on Azure]: virtual-machines-docker-machine.md
[How to use docker with swarm on Azure]: virtual-machines-docker-swarm.md

<!---HONumber=Mooncake_0215_2016-->