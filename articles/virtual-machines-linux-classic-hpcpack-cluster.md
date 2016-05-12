<properties
 pageTitle="HPC Pack 群集中的 Linux 计算 VM | Azure"
 description="了解如何在 Azure 中为 Linux 高性能计算 (HPC) 工作负荷创建和使用 HPC Pack 群集。"
 services="virtual-machines-linux"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,azure-resource-manager,hpc-pack"/>
<tags
	ms.service="virtual-machines-linux"
	ms.date="03/21/2016"
	wacn.date="05/12/2016"/>

# Azure 的 HPC Pack 群集中的 Linux 计算节点入门

在 Azure 中设置 Windows HPC Pack 群集，该群集包含运行 Windows Server 的头节点和运行受支持 Linux 分发版的多个计算节点。了解可用于在群集的 Linux 节点与 Windows 头节点之间移动数据的选项。了解如何将 Linux HPC 作业提交到群集。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]。


下图从较高层面显示了你要创建和使用的 HPC Pack 群集。

![包含 Linux 节点的 HPC 群集][scenario]

>[AZURE.TIP]如果你想要在本地 HPC Pack 群集中使用 Linux 节点，请参阅 [TechNet guidance（TechNet 指南）](https://technet.microsoft.com/zh-cn/library/mt595803.aspx)。

## 使用 Linux 计算节点部署 HPC Pack 群集

下面是在 Azure 中创建包含 Linux 计算节点的 HPC Pack 群集的两种建议方法：

* **PowerShell 脚本** - 使用 [Microsoft HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script) (**New-HpcIaaSCluster.ps1**) 在经典部署模型中自动化群集部署。此 Azure PowerShell 脚本使用 Azure 库中的 HPC Pack VM 映像进行快速部署，并提供一组全面的配置参数使部署轻松且灵活。脚本可部署 Azure 虚拟网络、存储帐户、云服务、域控制器、可选的单独 SQL Server 数据库服务器、群集头节点、计算节点、中转站节点、Azure PaaS（“迸发”）节点和 Linux 计算节点。

有关 HPC Pack 群集部署选项的概述，请参阅 [Getting started guide for HPC Pack 2012 R2 and HPC Pack 2012（HPC Pack 2012 R2 和 HPC Pack 2012 入门指南）](https://technet.microsoft.com/zh-cn/library/jj884144.aspx)和 [Options to create and manage a high peformance computing (HPC) cluster in Azure with Microsoft HPC Pack（使用 Microsoft HPC Pack 在 Azure 中创建和管理高性能计算 (HPC) 群集时可用的选项）](/documentation/articles/virtual-machines-linux-hpcpack-cluster-options)。

### 先决条件

* **Azure 订阅** - 你可以使用 Azure 全球或 Azure 中国服务中的订阅。如果你没有帐户，只需花费几分钟就能创建一个[1元帐户](/pricing/1rmb-trial/)。

* **内核配额** - 你可能需要增加内核配额，尤其是在你选择部署具有多核 VM 大小的多个群集节点时需要。若要增加配额，可免费[建立联机客户支持请求](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。

* **Linux 分发版** - HPC Pack 目前支持计算节点的以下 Linux 分发版：Ubuntu Server 14.04、CentOS 6.6 或 7.0、Red Hat Enterprise Linux 6.7 或 7.2 和 SUSE Linux Enterprise Server 12。你可以使用这些分发版的应用商店版本，或提供自己的版本。

如果使用 HPC Pack IaaS 部署脚本部署群集，则还要满足其他先决条件：

* **客户端计算机** - 你需要基于 Windows 的客户端计算机，以便运行群集部署脚本。

* **Azure PowerShell** - 在客户端计算机上[安装并配置 Azure PowerShell](/documentation/articles/powershell-install-configure)（版本 0.8.10 或更高版本）。

* **HPC Pack IaaS 部署脚本** - 从 [Microsoft 下载中心](https://www.microsoft.com/download/details.aspx?id=44949)下载并解压缩最新版本的脚本。可以通过运行 `New-HPCIaaSCluster.ps1 -Version` 检查脚本的版本。本文基于版本 4.4.0 或更高版本的脚本。

### 部署方案 2.使用 IaaS 部署脚本

HPC Pack IaaS 部署脚本使用描述 HPC 群集基础结构的 XML 配置文件作为输入。以下示例配置文件将部署一个小型群集，该群集由头节点和两个大小为 A7 的 CentOS 7 Linux 计算节点组成。请根据环境和所需的群集配置修改该文件。有关配置文件中的元素的详细信息，请参阅脚本文件夹中的 Manual.rtf 文件和 [Create an HPC cluster with the HPC Pack IaaS deployment script（使用 HPC Pack IaaS 部署脚本创建 HPC 群集）](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script)。

	<?xml version="1.0" encoding="utf-8" ?>
	<IaaSClusterConfig>
	  <Subscription>
	    <SubscriptionName>Subscription-1</SubscriptionName>
	    <StorageAccount>allvhdsje</StorageAccount>
	  </Subscription>
	  <Location>Japan East</Location>  
	  <VNet>
	    <VNetName>centos7rdmavnetje</VNetName>
	    <SubnetName>CentOS7RDMACluster</SubnetName>
	  </VNet>
	  <Domain>
	    <DCOption>HeadNodeAsDC</DCOption>
	    <DomainFQDN>hpc.local</DomainFQDN>
	  </Domain>
	  <Database>
	    <DBOption>LocalDB</DBOption>
	  </Database>
	  <HeadNode>
	    <VMName>CentOS7RDMA-HN</VMName>
	    <ServiceName>centos7rdma-je</ServiceName>
	  <VMSize>A4</VMSize>
	  <EnableRESTAPI />
	  <EnableWebPortal />
	  </HeadNode>
	  <LinuxComputeNodes>
	    <VMNamePattern>CentOS7RDMA-LN%1%</VMNamePattern>
	    <ServiceName>centos7rdma-je</ServiceName>
	    <VMSize>A7</VMSize>
	    <NodeCount>2</NodeCount>
	    <ImageName>5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20150325</ImageName>
	  </LinuxComputeNodes>
	</IaaSClusterConfig>

**运行 HPC Pack IaaS 部署脚本**

1. 在客户端计算机上以管理员身份打开 PowerShell 控制台。

2. 将目录更改到脚本文件夹（在此示例中为 E:\\IaaSClusterScript）。

    	cd E:\IaaSClusterScript

3. 运行以下命令以部署 HPC Pack 群集。本示例假定配置文件位于 E:\\HPCDemoConfig.xml。

    	.\New-HpcIaaSCluster.ps1 -ConfigFile E:\HPCDemoConfig.xml -AdminUserName MyAdminName

    由于未指定 **-LogFile** 参数，此脚本将自动生成日志文件。日志不是实时写入的，而是在验证和部署结束时收集的。如果 PowerShell 进程已停止但脚本仍在运行，一些日志将会丢失。

    a.由于在上述命令中未指定 **AdminPassword**，系统将提示你为用户 MyAdminName 输入密码。

    b.然后，此脚本将开始验证配置文件。这将需要几十秒到几分钟时间，具体取决于网络连接。

    ![验证][validate]

    c.通过验证后，此脚本将列出要为 HPC 群集创建的资源。输入 Y 以继续。

    ![资源][resources]

    d.此脚本开始部署 HPC Pack 群集并完成配置，而无需进一步手动步骤。这可能需要几分钟。

    ![部署][deploy]

## 连接到头节点

成功完成部署之后，请使用部署群集时提供的域凭据（例如，hpc\\clusteradmin）[通过远程桌面连接到头节点](/documentation/articles/virtual-machines-windows-classic-connect-logon)。

在头节点上，启动 HPC 群集管理器来查看 HPC Pack 群集的状态。你可以用处理 Windows 计算节点的相同方式管理和监视 Linux 计算节点。例如，在“资源管理”中，你将看到列出的 Linux 节点（这些节点都是使用 **LinuxNode** 模板部署的）。

![节点管理][management]

你还会在“热度地图”视图中看到 Linux 节点。

![热度地图][heatmap]

## 如何在具有 Linux 节点的群集中移动数据

要在群集的 Linux 节点和 Windows 头节点之间移动数据有几种选择。以下是三种常用方法。

* **Azure 文件** - 公开用于在 Azure 存储中存储数据文件的托管 SMB 文件共享。Windows 节点和 Linux 节点都可以同时装载 Azure 文件共享作为某个驱动器或文件夹，即使这些节点部署在不同虚拟网络中也是如此。

* **头节点 SMB 共享** - 在 Linux 节点上装载头节点的标准 Windows 共享文件夹。

* **头节点 NFS 服务器** - 为混合 Windows 和 Linux 环境提供文件共享解决方案。

### Azure 文件存储

[Azure 文件](/home/features/storage/files/)服务使用标准 SMB 2.1 协议公开文件共享。Azure VM 和云服务可通过装载的共享在应用程序组件之间共享文件数据，本地应用程序可通过文件存储 API 来访问共享中的文件数据。

有关创建 Azure 文件共享以及将其装入头节点的详细步骤，请参阅 [Get started with Azure File storage on Windows（在 Windows 上开始使用 Azure 文件存储）](/documentation/articles/storage-dotnet-how-to-use-files)。若要在 Linux 节点上载入 Azure 文件共享，请参阅 [How to use Azure File Storage with Linux（如何通过 Linux 使用 Azure 文件存储）](/documentation/articles/storage-how-to-use-files-linux)。若要设置持久性连接，请参阅 [Persisting connections to Azure Files（将连接保存到 Azure 文件中）](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)。

在此示例中，我们将在存储帐户 allvhdsje 上创建一个名为 rdma 的 Azure 文件共享。若要在头节点上装入该共享，请打开命令提示符并输入以下命令：

	> cmdkey /add:allvhdsje.file.core.chinacloudapi.cn /user:allvhdsje /pass:<storageaccountkey>
	> net use Z: \\allvhdje.file.core.chinacloudapi.cn\rdma /persistent:yes

在此示例中，allvhdsje 是存储帐户名称，storageaccountkey 是存储帐户密钥，rdma 是 Azure 文件共享名称。该 Azure 文件共享将装载到头节点的 Z: 上。

若要在 Linux 节点上装载 Azure 文件共享，请在头节点上运行 **clusrun** 命令。**[Clusrun](https://technet.microsoft.com/zh-cn/library/cc947685.aspx)** 是有用的 HPC Pack 工具，用于在多个节点上执行管理任务。（另请参阅本文中的[用于 Linux 节点的 CLusrun](#CLusrun-for-Linux-nodes)。）

打开 Windows PowerShell 窗口并输入以下命令。

	PS > clusrun /nodegroup:LinuxNodes mkdir -p /rdma
	
	PS > clusrun /nodegroup:LinuxNodes mount -t cifs //allvhdsje.file.core.chinacloudapi.cn/rdma /rdma -o vers=2.1`,username=allvhdsje`,password=<storageaccountkey>'`,dir_mode=0777`,file_mode=0777

第一个命令在 LinuxNodes 组中的所有节点上创建名为 /rdma 的文件夹。第二个命令将 Azure 文件共享 allvhdsjw.file.core.chinacloudapi.cn/rdma 装载到 /rdma 文件夹上，并将目录和文件模式位设置为 777。在第二个命令中，allvhdsje 是存储帐户名称，storageaccountkey 是存储帐户密钥。

>[AZURE.NOTE]第二个命令中的“\`”符号是 PowerShell 的转义符号。“\`,”表示“,”（逗号字符）是命令的一部分。

### 头节点共享

此外，还可以在 Linux 节点上装载头节点的共享文件夹。这是最简单的共享文件方法，但头节点和所有 Linux 节点必须部署在同一虚拟网络中。下面是相关步骤。

1. 在头节点上创建一个文件夹并向具有读/写权限的所有用户共享它。例如，在头节点上将 D:\\OpenFOAM 共享为 \\CentOS7RDMA-HN\\OpenFOAM。在此处，CentOS7RDMA-HN 是头节点的主机名。

    ![文件共享权限][fileshareperms]

    ![文件共享][filesharing]

2. 打开 Windows PowerShell 窗口并运行以下命令来装载共享文件夹。

	PS > clusrun /nodegroup:LinuxNodes mkdir -p /openfoam
	
	PS > clusrun /nodegroup:LinuxNodes mount -t cifs //CentOS7RDMA-HN/OpenFOAM /openfoam -o vers=2.1`,username=<username>`,password='<password>'`,dir_mode=0777`,file\_mode=0777

第一个命令在 LinuxNodes 组中的所有节点上创建名为 /openfoam 的文件夹。第二个命令将共享文件夹 //CentOS7RDMA-HN/OpenFOAM 装载到该文件夹上，并将目录和文件模式位设置为 777。该命令中的用户名和密码应是头节点上的群集用户的用户名和密码。（请参阅 [Add or remove cluster users（添加或删除群集用户）](https://technet.microsoft.com/zh-cn/library/ff919330.aspx)。）

>[AZURE.NOTE]第二个命令中的“\`”符号是 PowerShell 的转义符号。“\`,”表示“,”（逗号字符）是命令的一部分。


### NFS 服务器

NFS 服务使你能够在运行 Windows Server 2012 操作系统的计算机之间使用 SMB 协议共享和迁移文件，并在基于 Linux 的计算机之间使用 NFS 协议共享和迁移文件。NFS 服务器和所有其他节点必须部署在同一虚拟网络中。与 SMB 共享相比，它提供了与 Linux 节点更好的兼容性；例如，它支持文件链接。

1. 若要安装和设置 NFS 服务器，请按照[网络文件系统第一个共享端到端的服务器](http://blogs.technet.com/b/filecab/archive/2012/10/08/server-for-network-file-system-first-share-end-to-end.aspx)中的步骤进行操作。

    例如，创建具有以下属性的名为 nfs 的 NFS 共享。

    ![NFS 授权][nfsauth]

    ![NFS 共享权限][nfsshare]

    ![NFS NTFS 权限][nfsperm]

    ![NFS 管理属性][nfsmanage]

2. 打开 Windows PowerShell 窗口并运行以下命令来装载 NFS 共享。

  ```
  PS > clusrun /nodegroup:LinuxNodes mkdir -p /nfsshare
  PS > clusrun /nodegroup:LinuxNodes mount CentOS7RDMA-HN:/nfs /nfsshared
  ```

  第一个命令在 LinuxNodes 组中的所有节点上创建名为 /nfsshared 的文件夹。第二个命令将 NFS 共享 CentOS7RDMA-HN:/nfs 装载到该文件夹上。在此处，CentOS7RDMA-HN:/nfs 是 NFS 共享的远程路径。

## 如何提交作业
有多种方法可将作业提交到 HPC Pack 群集：

* HPC 群集管理器或 HPC 作业管理器 GUI

* HPC Web 门户

* REST API

通过 HPC Pack GUI 工具和 HPC Web 门户将作业提交到 Azure 中的群集的方法与 Windows 计算节点相同。请参阅 [HPC Pack Job Manager（HPC Pack 作业管理器）](https://technet.microsoft.com/zh-cn/library/ff919691.aspx)和 [How to submit jobs from an on-premises client computer（如何从本地客户端计算机提交作业）](/documentation/articles/virtual-machines-windows-hpcpack-cluster-submit-jobs)。

若要通过 REST API 提交作业，请参阅[在 Microsoft HPC Pack 中通过使用 REST API 创建和提交作业](http://social.technet.microsoft.com/wiki/contents/articles/7737.creating-and-submitting-jobs-by-using-the-rest-api-in-microsoft-hpc-pack-windows-hpc-server.aspx)。若要从 Linux 客户端提交作业，另请参阅 [HPC Pack SDK](https://www.microsoft.com/download/details.aspx?id=47756) 中的 Python 示例。

##<a name="CLusrun-for-Linux-nodes"></a> 用于 Linux 节点的 Clusrun

HPC Pack **clusrun** 工具可用于通过命令提示符或 HPC 群集管理器在 Linux 节点上执行命令。下面是一些基本示例。

* 显示群集中所有节点的当前用户名。

    	> clusrun whoami

* 在 linuxnodes 组中的所有节点上安装 **gdb** 调试器工具与 **yum**，然后在 10 分钟后重启节点。

    	> clusrun /nodegroup:linuxnodes yum install gdb -y; shutdown -r 10

* 创建一个在群集的每个 Linux 节点上每秒显示 1 到 10 中的一个数字的 shell 脚本，运行该脚本并立即显示节点的输出。

    	> clusrun /interleaved /nodegroup:linuxnodes echo "for i in {1..10}; do echo \\"\$i\\"; sleep 1; done" ^> script.sh; chmod +x script.sh; ./script.sh

>[AZURE.NOTE] 在 **clusrun** 命令中可能需要使用某些转义符。如此示例中所示，在命令提示符下使用 ^ 来转义“>”符号。

## 后续步骤

* 尝试扩展群集，使之拥有更多的节点，或者尝试在群集上运行 Linux 工作负荷。有关示例，请参阅[在 Azure 中的 Linux 计算节点上使用 Microsoft HPC Pack 运行 NAMD](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster-namd)。

<!--Image references-->
[scenario]: ./media/virtual-machines-linux-classic-hpcpack-cluster/scenario.png
[portal]: ./media/virtual-machines-linux-classic-hpcpack-cluster/portal.png
[validate]: ./media/virtual-machines-linux-classic-hpcpack-cluster/validate.png
[resources]: ./media/virtual-machines-linux-classic-hpcpack-cluster/resources.png
[deploy]: ./media/virtual-machines-linux-classic-hpcpack-cluster/deploy.png
[management]: ./media/virtual-machines-linux-classic-hpcpack-cluster/management.png
[heatmap]: ./media/virtual-machines-linux-classic-hpcpack-cluster/heatmap.png
[fileshareperms]: ./media/virtual-machines-linux-classic-hpcpack-cluster/fileshare1.png
[filesharing]: ./media/virtual-machines-linux-classic-hpcpack-cluster/fileshare2.png
[nfsauth]: ./media/virtual-machines-linux-classic-hpcpack-cluster/nfsauth.png
[nfsshare]: ./media/virtual-machines-linux-classic-hpcpack-cluster/nfsshare.png
[nfsperm]: ./media/virtual-machines-linux-classic-hpcpack-cluster/nfsperm.png
[nfsmanage]: ./media/virtual-machines-linux-classic-hpcpack-cluster/nfsmanage.png

<!---HONumber=Mooncake_0503_2016-->