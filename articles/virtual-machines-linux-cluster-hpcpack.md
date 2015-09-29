<properties
 pageTitle="HPC Pack 群集中的 Linux 计算节点入门 | Windows Azure"
 description="了解如何编写脚本以部署 Azure 中包含运行 Windows Server 的头节点和 Linux 计算节点的 HPC Pack 群集。"
 services="virtual-machines"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""/>
<tags
ms.service="virtual-machines"
 ms.date="07/27/2015"
 wacn.date="09/15/2015"/>

# Azure 的 HPC Pack 群集中的 Linux 计算节点入门

本文介绍如何使用 Azure PowerShell 脚本在 Azure 中设置 Windows HPC Pack 群集，该群集包含运行 Windows Server 的头节点和运行 CentOS Linux 分发的多个计算节点。我们还会介绍几种将数据文件移到 Linux 计算节点的方法。你可以使用此群集在 Azure 中运行 Linux HPC 工作负荷。

下图在较高级别显示了将创建的 HPC Pack 群集。

![具有 Linux 节点的 HPC 群集][scenario]

## 使用 Linux 计算节点部署 HPC Pack 群集

你将使用 Windows HPC Pack IaaS 部署脚本 (**New-HpcIaaSCluster.ps1**) 在 Azure 基础结构服务 (IaaS) 中自动执行群集部署。此 Azure PowerShell 脚本使用 Azure 应用商店中的 HPC Pack VM 映像进行快速部署，并提供一组全面的配置参数使部署轻松且灵活。你可以使用脚本部署 Azure 虚拟网络、存储帐户、云服务、域控制器、可选的单独 SQL Server 数据库服务器、群集头节点、计算节点、代理节点、Azure PaaS（“迸发”）节点和 Linux 计算节点（[HPC Pack 2012 R2 Update 2](https://technet.microsoft.com/zh-cn/library/mt269417.aspx) 中引入的 Linux 支持）。

有关 HPC Pack 群集部署选项的概述，请参阅 [HPC Pack 2012 R2 和 HPC Pack 2012 入门指南](https://technet.microsoft.com/zh-cn/library/jj884144.aspx)。

### 先决条件

* **客户端计算机** - 你需要基于 Windows 的客户端计算机，以便运行群集部署脚本。

* **Azure PowerShell** - 在客户端计算机上[安装并配置 Azure PowerShell](/documentation/articles/powershell-install-configure)（版本 0.8.10 或更高版本）。

* **HPC Pack IaaS 部署脚本** - 从 [Microsoft 下载中心](https://www.microsoft.com/zh-cn/download/details.aspx?id=44949)下载并解压缩最新版本的脚本。可以通过运行 `New-HPCIaaSCluster.ps1 –Version` 检查脚本的版本。本文基于版本 4.4.0 或更高版本的脚本。

* **Azure 订阅** - 你可以使用 Azure 全球或 Azure 中国服务中的订阅。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](http://azure.microsoft.com/pricing/free-trial/)。

* **内核配额** - 你可能需要增加内核配额，尤其是在你选择部署具有多核 VM 大小的多个群集节点时需要。对于本文中的示例，你将至少需要 24 个内核。若要增加配额，可免费[建立联机客户支持请求](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。

### 创建配置文件
HPC Pack IaaS 部署脚本使用描述 HPC 群集基础结构的 XML 配置文件作为输入。若要部署由一个头节点和 2 个 Linux 计算节点组成的小群集，请将你环境的值代入下面的示例配置文件。有关配置文件的详细信息，请参阅脚本文件夹中的 Manual.rtf 文件或[脚本文档](https://msdn.microsoft.com/zh-cn/library/azure/dn864734.aspx)。

```
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
```

以下是配置文件中的元素的简要说明。

* **IaaSClusterConfig** - 配置文件的根元素。

* **Subscription** - 用于部署 HPC Pack 群集的 Azure 订阅。使用以下命令确保 Azure 订阅名称已配置并且在客户端计算机中唯一。在此示例中，我们使用 Azure 订阅“Subscription-1”。

    ```
    PS > Get-AzureSubscription –SubscriptionName <SubscriptionName>
    ```

    >[AZURE.NOTE]或者，可以使用订阅 ID 指定要使用的订阅。请参阅脚本文件夹中的 Manual.rtf 文件。

* **StorageAccount** - HPC Pack 群集的所有永久性数据将存储到指定的存储帐户（在本示例中为 allvhdsje）。如果该存储帐户不存在，则脚本将在 **Location** 指定的区域中创建它。

* **Location** - 将在其中部署 HPC Pack 群集的 Azure 区域（在本示例中为日本东部）。

* **VNet** - 将在其中创建 HPC 群集的虚拟网络和子网的设置。在运行此脚本之前你可以自己创建虚拟网络和子网，否则此脚本将使用地址空间 192.168.0.0/20 创建虚拟网络，使用地址空间 192.168.0.0/23 创建子网。在本示例中，此脚本将创建虚拟网络 centos7rdmavnetje 和子网 CentOS7RDMACluster。

* **Domain** - HPC Pack 群集的 Active Directory 域设置。此脚本创建的所有 Windows VM 都将加入该域。目前，此脚本支持三个域选项：ExistingDC、NewDC 和 HeadNodeAsDC。在本示例中，我们将头节点配置为域控制器。完全限定域名是 hpc.local。

* **Database** - HPC Pack 群集的数据库设置。目前，此脚本支持三个数据库选项：ExistingDB、NewRemoteDB 和 LocalDB。在本示例中，我们将在头节点上创建本地数据库。

* **HeadNode** - HPC Pack 头节点的设置。在本示例中，我们将在云服务 centos7rdma-je 中创建名为 CentOS7RDMA-HN 的 A7 大小头节点。为了支持远程（未加入域）客户端计算机进行 HPC 作业提交，此脚本将启用 HPC 作业计划程序 REST API 和 HPC Web 门户。

* **LinuxComputeNodes** - HPC Pack Linux 计算节点的设置。在本示例中，我们将在云服务 centos7rdma-je 中创建两个 A7 大小 CentOS 7 Linux 计算节点（CentOS7RDMA-LN1 和 CentOS7RDMA-LN2）。

### 有关 Linux 计算节点的其他注意事项

* HPC Pack 目前支持计算节点的以下 Linux 分发：Ubuntu Server 14.10、CentOS 6.6、CentOS 7.0 和 SUSE Linux Enterprise Server 12。

* 本文中的示例使用 Azure 应用商店中提供的特定 CentOS 版本来创建群集。如果要使用其他可用映像，请使用 **get-azurevmimage** Azure PowerShell cmdlet 查找所需的映像。例如，若要列出所有 CentOS 7.0 映像，请运行以下命令：```
    get-azurevmimage | ?{$_.Label -eq "OpenLogic 7.0"}
    ```

    找到所需的映像，然后替换配置文件中的 **ImageName** 值。

* 对 A8 和 A9 大小 VM 支持 RDMA 连接的 Linux 映像可用。如果你指定的映像安装并启用了 Linux RDMA 驱动程序，则 HPC Pack IaaS 部署脚本将部署这些驱动程序。例如，你可以为当前 SUSE Linux Enterprise Server 12（已针对应用商店中的高性能计算映像进行优化）指定映像名称 `b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-v20150708`。


* 若要在从支持的映像创建的 Linux VM 上启用 Linux RDMA 以运行 MPI 作业，请在群集部署后根据应用程序需求在 Linux 节点上安装并配置特定的 MPI 库。有关如何在 Azure 上的 Linux 节点中使用 RDMA 的详细信息，请参阅[设置 Linux RDMA 群集以运行 MPI 应用程序](/documentation/articles/virtual-machines-linux-cluster-rdma)。

* 请确保在一个服务内部署所有 Linux RDMA 节点，以便节点之间的 RDMA 网络连接可以正常工作。


## 运行 HPC Pack IaaS 部署脚本

1. 在客户端计算机上以管理员身份打开 PowerShell 控制台。

2. 将目录更改到脚本文件夹（在此示例中为 E:\\IaaSClusterScript）。

    ```
cd E:\IaaSClusterScript
```

3. 运行以下命令以部署 HPC Pack 群集。本示例假定配置文件位于 E:\\HPCDemoConfig.xml。

    ```
    .\New-HpcIaaSCluster.ps1 –ConfigFile E:\HPCDemoConfig.xml –AdminUserName MyAdminName
```

    由于未指定 **-LogFile** 参数，此脚本将自动生成日志文件。日志不是实时写入，而是在验证和部署结束时收集，因此如果在运行此脚本时停止了 PowerShell 进程，则某些日志将丢失。

    a.由于在上述命令中未指定 **AdminPassword**，系统将提示你为用户 *MyAdminName* 输入密码。

    b.然后，此脚本将开始验证配置文件。这将需要几十秒到几分钟时间，具体取决于网络连接。

    ![验证][validate]

    c.通过验证后，此脚本将列出要为 HPC 群集创建的资源。输入 *Y* 以继续。

    ![资源][resources]

    d.然后，此脚本将开始部署 HPC Pack 群集并将完成配置，而无需进一步手动步骤。这可能需要几分钟。

    ![部署][deploy]

4. 在配置成功完成后，通过远程桌面进入头节点并打开 HPC 群集管理器以检查 HPC Pack 群集状态。你可以用处理 Windows 计算节点的相同方式管理和监视 Linux 计算节点。例如，你将看到在“节点管理”中列出的 Linux 节点。

    ![节点管理][management]

    你还会在“热度地图”视图中看到 Linux 节点。

    ![热度地图][heatmap]

## 如何在具有 Linux 节点的群集中移动数据

要在群集的 Linux 节点和 Windows 头节点之间移动数据有几种选择。以下是三种常用方法。

* **Azure 文件** - 公开用于在 Azure 存储中存储数据文件的文件共享。Windows 节点和 Linux 节点都可以同时装载 Azure 文件共享作为某个驱动器或文件夹，即使这些节点部署在不同虚拟网络中也是如此。

* **头节点 SMB 共享** - 在 Linux 节点上装载头节点的共享文件夹。

* **头节点 NFS 服务器** - 为混合 Windows 和 Linux 环境提供文件共享解决方案。

### Azure 文件

[Azure 文件](https://azure.microsoft.com/services/storage/files/)服务使用标准 SMB 2.1 协议公开文件共享。Azure VM 和云服务可通过装载的共享在应用程序组件之间共享文件数据，本地应用程序可通过文件存储 API 来访问共享中的文件数据。有关详细信息，请参阅[如何通过 PowerShell 和 .NET 使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files)。

若要创建 Azure 文件共享，请参阅 [Windows Azure 文件服务简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)中的详细步骤 。若要设置持久性连接，请参阅[将连接保存到 Windows Azure 文件中](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)。

在此示例中，我们将在存储帐户 allvhdsje 上创建一个名为 rdma 的 Azure 文件共享。为了在头节点上装载该共享，我们打开命令窗口并输入以下命令：

```
> cmdkey /add:allvhdsje.file.core.windows.net /user:allvhdsje /pass:<storageaccountkey>
> net use Z: \\allvhdje.file.core.windows.net\rdma /persistent:yes
```

在此示例中，allvhdsje 是存储帐户名称，storageaccountkey 是存储帐户密钥，rdma 是 Azure 文件共享名称。该 Azure 文件共享将装载到头节点的 Z: 上。

若要在 Linux 节点上装载 Azure 文件共享，请在头节点上运行 **clusrun** 命令。**[Clusrun](https://technet.microsoft.com/zh-cn/library/cc947685.aspx)** 是有用的 HPC Pack 工具，用于在多个节点上执行管理任务。（另请参阅本文中的[用于 Linux 节点的 CLusrun](#CLusrun-for-Linux-nodes)。）

打开 Windows PowerShell 窗口并输入以下命令。

```
PS > clusrun /nodegroup:LinuxNodes mkdir -p /rdma

PS > clusrun /nodegroup:LinuxNodes mount -t cifs //allvhdsje.file.core.windows.net/rdma /rdma -o vers=2.1`,username=allvhdsje`,password=<storageaccountkey>'`,dir_mode=0777`,file_mode=0777
```

第一个命令在 LinuxNodes 组中的所有节点上创建名为 /rdma 的文件夹。第二个命令将 Azure 文件共享 allvhdsjw.file.core.windows.net/rdma 装载到 /rdma 文件夹上，并将目录和文件模式位设置为 777。在第二个命令中，allvhdsje 是存储帐户名称，storageaccountkey 是存储帐户密钥。

>[AZURE.NOTE]第二个命令中的 “`” 符号是 PowerShell 的转义符号。“`,” 表示 “,”（逗号）是命令的一部分。

### 头节点共享

此外，还可以在 Linux 节点上装载头节点的共享文件夹。这是最简单的共享文件方法，但头节点和所有 Linux 节点必须部署在同一虚拟网络中。下面是相关步骤。

1. 在头节点上创建一个文件夹并向具有读/写权限的所有用户共享它。例如，我们在头节点上将 D:\\OpenFOAM 共享为 \\CentOS7RDMA-HN\\OpenFOAM。在此处，CentOS7RDMA-HN 是头节点的主机名。

    ![文件共享权限][fileshareperms]

    ![文件共享][filesharing]

2. 打开 Windows PowerShell 窗口并运行以下命令来装载共享文件夹。

```
PS > clusrun /nodegroup:LinuxNodes mkdir -p /openfoam

PS > clusrun /nodegroup:LinuxNodes mount -t cifs //CentOS7RDMA-HN/OpenFOAM /openfoam -o vers=2.1`,username=<username>,password='<password>’,dir_mode=0777`,file_mode=0777
```

第一个命令在 LinuxNodes 组中的所有节点上创建名为 /openfoam 的文件夹。第二个命令将共享文件夹 //CentOS7RDMA-HN/OpenFOAM 装载到该文件夹上，并将目录和文件模式位设置为 777。该命令中的用户名和密码应是头节点上的用户的用户名和密码。

>[AZURE.NOTE]第二个命令中的 “`” 符号是 PowerShell 的转义符号。“`,” 表示 “,”（逗号）是命令的一部分。


### NFS 服务器

NFS 服务使用户能够在运行 Windows Server 2012 操作系统的计算机之间使用 SMB 协议共享和迁移文件，并在基于 Linux 的计算机之间使用 NFS 协议共享和迁移文件。NFS 服务器和所有其他节点必须部署在同一虚拟网络中。与 SMB 共享相比，它提供了与 Linux 节点更好的兼容性；例如，它支持文件链接。

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
有多种方法可将作业提交到 HPC Pack 群集

* HPC 群集管理器或 HPC 作业管理器 GUI

* HPC Web 门户

* REST API

通过 HPC Pack GUI 工具和 HPC Web 门户将作业提交到 Azure 中的群集的方法与 Windows 计算节点相同。请参阅 [HPC Pack 作业管理器](https://technet.microsoft.com/zh-cn/library/ff919691.aspx)和[如何从本地客户端提交作业](https://msdn.microsoft.com/zh-cn/library/azure/dn689084.aspx)。

若要通过 REST API 提交作业，请参阅[在 Microsoft HPC Pack 中通过使用 REST API 创建和提交作业](http://social.technet.microsoft.com/wiki/contents/articles/7737.creating-and-submitting-jobs-by-using-the-rest-api-in-microsoft-hpc-pack-windows-hpc-server.aspx)。若要从 Linux 客户端提交作业，另请参阅 [HPC Pack SDK](https://www.microsoft.com/download/details.aspx?id=47756) 中的 Python 示例。

## 用于 Linux 节点的 Clusrun

HPC Pack **clusrun** 工具可用于通过命令窗口或 HPC 群集管理器在 Linux 节点上执行命令。下面是一些示例。

* 显示群集中所有节点的当前用户名

    ```
> clusrun whoami
```

* 在 linuxnodes 组中的所有节点上安装 **gdb** 调试器工具第 i 个 **yum**，然后在 10 分钟后重启这些节点

    ```
> clusrun /nodegroup:linuxnodes yum install gdb –y; shutdown –r 10
```

* 创建一个在群集节点上每秒显示 1 到 10 的 shell 脚本，运行该脚本并立即显示每个节点的输出。

    ```
> clusrun /interleaved echo "for i in {1..10}; do echo \\"\$i\\"; sleep 1; done" ^> script.sh; chmod +x script.sh; ./script.sh```

>[AZURE.NOTE]在 clusrun 命令中可能需要使用某些转义符。在命令窗口中使用 ^，在 PowerShell 中使用 ` 来转换特殊字符。例如，在 PowerShell 中，逗号和分号字符必须分别通过 `, 和 `; 进行转换。在命令窗口中这些字符不需要转换。




## 后续步骤

* 使用 **clusrun** 将 Linux 应用程序安装到 Linux 计算节点上并向 HPC Pack 群集提交作业。

* 尝试将群集向上扩展为包含更多节点，或部署 [A8 或 A9](/documentation/articles/virtual-machines-a8-a9-a10-a11-specs) 大小计算节点以运行 MPI 工作负荷。


<!--Image references-->
[scenario]: ./media/virtual-machines-linux-cluster-hpcpack/scenario.png
[validate]: ./media/virtual-machines-linux-cluster-hpcpack/validate.png
[resources]: ./media/virtual-machines-linux-cluster-hpcpack/resources.png
[deploy]: ./media/virtual-machines-linux-cluster-hpcpack/deploy.png
[management]: ./media/virtual-machines-linux-cluster-hpcpack/management.png
[heatmap]: ./media/virtual-machines-linux-cluster-hpcpack/heatmap.png
[fileshareperms]: ./media/virtual-machines-linux-cluster-hpcpack/fileshare1.png
[filesharing]: ./media/virtual-machines-linux-cluster-hpcpack/fileshare2.png
[nfsauth]: ./media/virtual-machines-linux-cluster-hpcpack/nfsauth.png
[nfsshare]: ./media/virtual-machines-linux-cluster-hpcpack/nfsshare.png
[nfsperm]: ./media/virtual-machines-linux-cluster-hpcpack/nfsperm.png
[nfsmanage]: ./media/virtual-machines-linux-cluster-hpcpack/nfsmanage.png

<!---HONumber=69-->