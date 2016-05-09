<properties
 pageTitle="在 Linux VM 上运行 OpenFOAM 与 HPC Pack | Azure"
 description="在 Azure 上部署 Microsoft HPC Pack 群集，并在多个 Linux 计算节点上跨 RDMA 网络运行 OpenFOAM 作业。"
 services="virtual-machines"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-linux"
	ms.date="11/25/2015"
	wacn.date="01/29/2016"/>

# 在 Azure 中的 Linux RDMA 群集上运行 OpenFoam 和 Microsoft HPC Pack

本文介绍如何在 Azure 上部署 Microsoft HPC Pack 群集，以及如何在多个跨 Azure 远程直接内存访问 (RDMA) 网络进行连接的 Linux 计算节点上运行 [OpenFoam](http://openfoam.com/) 作业和 Intel MPI。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]资源管理器模型。

OpenFOAM（Open Field Operation and Manipulation 的缩写）是免费提供的开源计算流体动力学 (CFD) 软件包，广泛用于商业和学术组织的工程和科学项目中。它包括各种网格工具，最主要的工具是 snappyHexMesh，这是一个并行处理式网格器，适用于复杂的 CAD 几何以及预处理和后处理。几乎所有进程都是并行运行的，因此用户可以根据自己的需要充分利用计算机硬件。

Microsoft HPC Pack 可提供在 Azure 虚拟机群集上运行各种大型 HPC 和并行应用程序的功能，包括 MPI 应用程序。从 Microsoft HPC Pack 2012 R2 Update 2 开始，HPC Pack 还支持在 HPC Pack 群集中部署的 Linux 计算节点 VM 上运行 Linux HPC 应用程序。有关将 Linux 计算节点与 HPC Pack 一起使用的简介，请参阅 [Azure 的 HPC Pack 群集中的 Linux 计算节点入门](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster)。

>[AZURE.NOTE]本文假设你已对 Linux 系统管理以及如何在 Linux HPC 群集上运行 MPI 工作负荷有所了解。

## 先决条件

*   **具有 Linux 计算节点的 HPC Pack 群集** - 请参阅 [Azure 的 HPC Pack 群集中的 Linux 计算节点入门](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster)，了解使用 Azure PowerShell 脚本和 Azure 应用商店中的 HPC Pack 映像，在 Azure 上部署 HPC Pack 群集与 Linux 计算节点的先决条件和步骤。

    下面是一个示例 XML 配置文件，可以与脚本配合使用以部署基于 Azure 的 HPC Pack 群集，群集中包含一个 A7 大小的 Windows Server 2012 R2 头节点和 2 个 A7 大小的 SUSE Linux Enterprise Server 12 计算节点。请将订阅和服务名称替换为相应值。

    >[AZURE.NOTE]目前，在 Azure 中实现 Linux RDMA 联网只能在特定的 VM 上进行，这些 VM 是通过 Azure 应用商店中启用了 RDMA 的 SUSE Linux Enterprise Server 12 映像 (b4590d9e3ed742e4a1d46e5424aa335e\_\_suse-sles-12-hpc-v20150708) 创建的。

	
	    <?xml version="1.0" encoding="utf-8" ?>
	    <IaaSClusterConfig>
	      <Subscription>
	        <SubscriptionName>Subscription-1</SubscriptionName>
	        <StorageAccount>allvhdsje</StorageAccount>
	      </Subscription>
	      <Location>Japan East</Location>  
	      <VNet>
	        <VNetName>suse12rdmavnet</VNetName>
	        <SubnetName>SUSE12RDMACluster</SubnetName>
	      </VNet>
	      <Domain>
	        <DCOption>HeadNodeAsDC</DCOption>
	        <DomainFQDN>hpclab.local</DomainFQDN>
	      </Domain>
	      <Database>
	        <DBOption>LocalDB</DBOption>
	      </Database>
	      <HeadNode>
	        <VMName>SUSE12RDMA-HN</VMName>
	        <ServiceName>suse12rdma-je</ServiceName>
	        <VMSize>A7</VMSize>
	        <EnableRESTAPI />
	        <EnableWebPortal />
	      </HeadNode>
	      <LinuxComputeNodes>
	        <VMNamePattern>SUSE12RDMA-LN%1%</VMNamePattern>
	        <ServiceName>suse12rdma-je</ServiceName>
	        <VMSize>A7</VMSize>
	        <NodeCount>2</NodeCount>
	        <ImageName>b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-v20150708</ImageName>
	      </LinuxComputeNodes>
	    </IaaSClusterConfig>
	

    **其他须知项**

    *   将所有 Linux 计算节点部署在一个云服务中，以便使用节点之间的 RDMA 网络连接。

    *   部署 Linux 节点以后，若需通过 SSH 进行连接以执行其他管理任务，可在 Azure 门户中查找每个 Linux VM 的 SSH 连接详细信息。
        
*   **Intel MPI** - 若要在 Azure 的 Linux 计算节点上运行 OpenFOAM，你需要 [Intel.com site](https://software.intel.com/zh-cn/intel-mpi-library/) 提供的 Intel MPI Library 5 运行时。在后面的步骤中，你需要在 Linux 计算节点上安装 Intel MPI。为此，向 Intel 注册后，请点击确认电子邮件中的链接转到相关网页，并复制相应版本的 Intel MPI 所对应的 .tgz 文件的下载链接。本文基于 Intel MPI 版本 5.0.3.048。

*   **OpenFOAM Source Pack** - 从 [OpenFOAM Foundation 站点](http://www.openfoam.org/download/source.php)下载 Linux 版 OpenFOAM Source Pack 软件。本文基于 2.3.1 版的 Source Pack（以 OpenFOAM-2.3.1.tgz 形式供用户下载）。请按本文后面的说明，在 Linux 计算节点上对 OpenFOAM 进行解包和编译。

*   **EnSight**（可选）- 若要查看 OpenFOAM 模拟结果，请下载 Windows 版的 [EnSight](https://www.ceisoftware.com/download/) 可视化和分析程序并将其安装在 HPC Pack 群集的头节点上。EnSight 站点提供了许可和下载信息。


## 在计算节点之间建立互信关系

在多个 Linux 节点上运行跨节点作业需要节点彼此信任（通过 **rsh** 或 **ssh**）。使用 Microsoft HPC Pack IaaS 部署脚本创建 HPC Pack 群集时，此脚本会自动为指定的管理员帐户建立永久性互信关系。对于在群集域中创建的非管理员用户，分配作业时必须在节点之间建立临时互信关系，并在作业完成后销毁互信关系。若要为每个用户执行此操作，请向群集提供一个 HPC Pack 用于建立信任关系的 RSA 密钥对。

### 生成一个 RSA 密钥对

通过运行 Linux **ssh keygen** 命令，可以很容易生成一个 RSA 密钥对，其中包含一个公钥和一个私钥。

1.	登录到 Linux 计算机。

2.	运行以下命令。

	
	    ssh-keygen -t rsa
	    

    >[AZURE.NOTE]按 **Enter** 以使用默认设置，直至命令完成。请勿在此处输入密码；系统提示输入密码时，只需按 **Enter** 即可。

    ![生成一个 RSA 密钥对][keygen]

3.	将目录切换到 ~/.ssh 目录。私钥存储在 id\_rsa 中，公钥存储在 id\_rsa.pub 中。

    ![私钥和公钥][keys]

### 将密钥对添加到 HPC Pack 群集中
1.	使用 HPC Pack 管理员帐户（运行部署脚本时设置的管理员帐户），以远程桌面连接方式连接至头节点。

2. 使用 Windows Server 标准程序，在群集的 Active Directory 域中创建一个域用户帐户。例如，在头节点上使用 Active Directory 用户和计算机工具。本文中的示例假设你创建了一个名为 hpclab\\hpcuser 域用户。

3.	创建一个名为 C:\\cred.xml 的文件，将 RSA 密钥数据复制到此文件中。你可以在本文末尾的示例文件中找到此文件的一个示例。

	
	    <ExtendedData>
	        <PrivateKey>Copy the contents of private key here</PrivateKey>
	        <PublicKey>Copy the contents of public key here</PublicKey>
	    </ExtendedData>
	    

4.	打开命令提示符，输入以下命令，为 hpclab\\hpcuser 帐户设置凭据数据。使用 **extendeddata** 参数传递你为关键数据创建的 C:\\cred.xml 文件的名称。

	
	    hpccred setcreds /extendeddata:c:\cred.xml /user:hpclab\hpcuser /password:<UserPassword>
	    

    此命令成功完成后，没有输出。为你需要运行作业的用户帐户设置凭据后，请将 cred.xml 文件存储在安全位置，或者删除 cred.xml 文件。

5.	如果你在一个 Linux 节点上生成 RSA 密钥对，请记住在使用完成后删除这些密钥。如果找到一个现有 id\_rsa 文件或 id\_rsa.pub 文件，HPC Pack 并不会建立互信关系。

>[AZURE.IMPORTANT]我们不建议在共享群集上以群集管理员的身份运行 Linux 作业，因为由管理员提交的作业会在 Linux 节点的根帐户下运行。由非管理员用户提交的作业会在本地 Linux 用户帐户（名称与作业用户相同）下运行，HPC Pack 会在分配了此作业的所有节点内为这位 Linux 用户建立互信关系。在运行作业之前，你可以在 Linux 节点上手动设置 Linux 用户，也可以在提交作业时由 HPC Pack 自动创建用户。如果 HPC Pack 创建了一个用户，则作业完成后 HPC Pack 会删除此用户。在节点上完成作业后删除密钥，以减少安全威胁。

## 为 Linux 节点设置文件共享

现在，在头节点上对一个文件夹设置标准 SMB 共享，然后在所有 Linux 节点上装载此共享文件夹，就可以支持 Linux 节点使用一个通用路径访问应用程序文件。你可以根据需要使用其他文件共享选项，例如 Azure 文件共享（建议用于多种方案）或 NFS 共享。请参阅 [Azure 的 HPC Pack 群集中的 Linux 计算节点入门](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster)中的文件共享信息和详细步骤。

1.	在头节点上创建一个文件夹，然后通过设置读/写权限与所有人共享。例如，在头节点上将 C:\\OpenFOAM 共享为 \\\SUSE12RDMA-HN\\OpenFOAM。在此处，*SUSE12RDMA-HN* 是头节点的主机名。

2.	打开 Windows PowerShell 窗口并运行以下命令来装载共享文件夹。

    ```
    clusrun /nodegroup:LinuxNodes mkdir -p /openfoam

    clusrun /nodegroup:LinuxNodes mount -t cifs //SUSE12RDMA-HN/OpenFOAM /openfoam -o vers=2.1`,username=<username>`,password='<password>’`,dir_mode=0777`,file_mode=0777
    ```

第一个命令在 LinuxNodes 组中的所有节点上创建名为 /openfoam 的文件夹。第二个命令将共享文件夹 //SUSE12RDMA-HN/OpenFOAM 装载到 dir\_mode 和 file\_mode 位设置为 777 的 Linux 节点上。该命令中的*用户名*和*密码*应是头节点上的用户的凭据。

>[AZURE.NOTE]第二个命令中的 “`” 符号是 PowerShell 的转义符号。“`,” 表示 “,”（逗号字符）是命令的一部分。

## 安装 MPI 和 OpenFOAM

若要在 RDMA 网络上以 MPI 作业的形式运行 OpenFOAM，你需要使用 Intel MPI 库编译 OpenFOAM。

你需要先运行多个 **clusrun** 命令，在所有 Linux 节点上安装 Intel MPI 库和 OpenFOAM。使用以前配置的头节点共享在 Linux 节点中共享安装文件。

>[AZURE.IMPORTANT]这些安装和编译步骤属于示例，需要对 Linux 系统管理有一定的了解，以确保正确安装相关内容（尤其是编译器和库）。你可能需要根据 Intel MPI 和 OpenFOAM 的版本修改某些环境变量或其他设置。有关详细信息，请参阅 [Intel MPI Library for Linux 安装指南](http://scc.ustc.edu.cn/zlsc/tc4600/intel/2015.1.133/impi/INSTALL.html)和 [OpenFOAM Source Pack 安装](http://www.openfoam.org/download/source.php)。


### 安装 Intel MPI

将下载的 Intel MPI 安装包（在此示例中为 l\_mpi\_p\_5.0.3.048.tgz）保存到头节点上的 C:\\OpenFoam 中，使得 Linux 节点能够从 /openfoam 访问此文件。然后，运行 **clusrun** 在所有 Linux 节点上安装 Intel MPI Library。

1.  以下命令复制安装包并将其解压缩到每个节点上的 /opt/intel。

	
	    clusrun /nodegroup:LinuxNodes mkdir -p /opt/intel
	
	    clusrun /nodegroup:LinuxNodes cp /openfoam/l_mpi_p_5.0.3.048.tgz /opt/intel/
	
	    clusrun /nodegroup:LinuxNodes tar -xzf /opt/intel/l_mpi_p_5.0.3.048.tgz -C /opt/intel/
	    

2.  若要以无提示方式安装 Intel MPI Library，请使用 silent.cfg 文件。你可以在本文末尾的示例文件中找到一个示例。将此文件放在共享文件夹 /openfoam 中。有关 silent.cfg 文件的详细信息，请参阅 [Intel MPI Library for Linux 安装指南 - 无提示安装](http://scc.ustc.edu.cn/zlsc/tc4600/intel/2015.1.133/impi/INSTALL.html#silentinstall)。

    >[AZURE.TIP]请确保将你的 silent.cfg 文件另存为带有 Linux 换行（仅 LF，而不是 CR LF）的文本文件。这可确保其在 Linux 节点上正常运行。

3.  在静默模式下安装 Intel MPI Library。

	
	    clusrun /nodegroup:LinuxNodes bash /opt/intel/l_mpi_p_5.0.3.048/install.sh --silent /openfoam/silent.cfg
	    
    
### 配置 MPI

测试时，应将以下行添加到每个 Linux 节点的 /etc/security/limits.conf 中：

	
	*               hard    memlock         unlimited
	*               soft    memlock         unlimited
	

更新 limits.conf 文件之后，请重新启动 Linux 节点。例如，使用以下 **clusrun** 命令。

	
	clusrun /nodegroup:LinuxNodes systemctl reboot
	

重新启动之后，请确保将共享文件夹以 /openfoam 形式装入。

### 编译并安装 OpenFOAM

将下载的 OpenFOAM Source Pack 安装包（在此示例中为 OpenFOAM-2.3.1.tgz）保存到头节点上的 C:\\OpenFoam 中，使得 Linux 节点能够从 /openfoam 访问此文件。然后，运行 **clusrun** 在所有 Linux 节点上编译 OpenFOAM。


1.  在每个 Linux 节点上创建 /opt/OpenFOAM 文件夹，将源包复制到该文件夹中，然后解压缩。

	
	    clusrun /nodegroup:LinuxNodes mkdir -p /opt/OpenFOAM
	
	    clusrun /nodegroup:LinuxNodes cp /openfoam/OpenFOAM-2.3.1.tgz /opt/OpenFOAM/
	
	    clusrun /nodegroup:LinuxNodes tar -xzf /opt/OpenFOAM/OpenFOAM-2.3.1.tgz -C /opt/OpenFOAM/
	    

2.  若要通过 Intel MPI Library 编译 OpenFOAM，请先针对 Intel MPI 和 OpenFOAM 设置某些环境变量。使用名为 settings.sh 的 bash 脚本来执行该操作。你可以在本文末尾的示例文件中找到一个示例。将此文件（保存时带有 Linux 换行）置于共享文件夹 /openfoam 中。此文件还包含可以随后用来运行 OpenFOAM 作业的 MPI 和 OpenFOAM 运行时设置。

3. 安装编译 OpenFOAM 所需的相关程序包。你可能需要添加存储库，具体取决于你的 Linux 分发。运行类似于以下命令的 **clusrun** 命令：

	
	    clusrun /nodegroup:LinuxNodes zypper ar http://download.opensuse.org/distribution/13.2/repo/oss/suse/ opensuse
	    
	    clusrun /nodegroup:LinuxNodes zypper -n --gpg-auto-import-keys install --repo opensuse --force-resolution -t pattern devel_C_C++
	    
    
    必要时可对每个 Linux 节点使用 ssh，以便运行相关命令来确认这些命令是否正常运行。

4.  运行以下命令以编译 OpenFOAM。编译过程需一段时间才能完成，并且会生成大量可供标准输出的日志信息，因此请使用 **/interleaved** 选项来交错显示输出。

    ```
    clusrun /nodegroup:LinuxNodes /interleaved source /openfoam/settings.sh `&`& /opt/OpenFOAM/OpenFOAM-2.3.1/Allwmake
    ```
    
    >[AZURE.NOTE]命令中的“\`”符号是 PowerShell 的转义符号。“\`&”表示“&”是命令的一部分。

## 准备运行 OpenFOAM 作业

现在，请准备好在 2 个 Linux 节点上运行名为 sloshingTank3D 的 MPI 作业，该作业是 OpenFoam 示例之一。

### 设置运行时环境

在头节点的 Windows PowerShell 窗口运行以下命令，以便在所有 Linux 节点上设置 MPI 和 OpenFOAM 的运行时环境。（此命令仅适用于 SUSE Linux。）

	
	clusrun /nodegroup:LinuxNodes cp /openfoam/settings.sh /etc/profile.d/
	

### 准备示例数据

使用以前配置的头节点共享在 Linux 节点中共享文件（装载为 /openfoam）。

1.  通过 SSH 连接到 Linux 计算节点之一。

2.  如果你尚未执行此操作，请运行以下命令以设置 OpenFOAM 运行时环境。

	
	    $ source /openfoam/settings.sh
	    
    
3.  将 sloshingTank3D 示例复制到共享文件夹，然后导航到该文件夹。

	
	    $ cp -r $FOAM_TUTORIALS/multiphase/interDyMFoam/ras/sloshingTank3D /openfoam/
	
	    $ cd /openfoam/sloshingTank3D
	    

4.  使用此示例的默认参数时，可能需要数十分钟或更长的时间才能运行，因此你可能需要修改部分参数，使其运行速度加快。一种简单的方法是修改 system/controlDict 文件中的时间步骤变量 deltaT 和 writeInterval，该文件存储与时间控制以及解决方案数据的读取和写入相关的所有输入数据。例如，你可以将 deltaT 的值从 0.05 更改为 0.5，将 writeInterval 的值从 0.05 更改为 0.5。

    ![修改步骤变量][step_variables]

5.  在 system/decomposeParDict 文件中指定变量的所需值。此示例使用 2 个 Linux 节点，每个节点 8 个内核，因此可将 numberOfSubdomains 设置为 16，将 hierarchicalCoeffs 的 n 设置为 (1 1 16)，这表示使用 16 个流程并行运行 OpenFOAM。有关如何并行运行 OpenFOAM 的详细信息，请参阅 [OpenFOAM 用户指南：3.4 并行运行应用程序](http://cfd.direct/openfoam/user-guide/running-applications-parallel/#x12-820003.4)。

    ![对过程进行分解][decompose]

6.  从 sloshingTank3D 目录运行以下命令，对示例数据进行准备。

	
	    $ . $WM_PROJECT_DIR/bin/tools/RunFunctions
	
	    $ m4 constant/polyMesh/blockMeshDict.m4 > constant/polyMesh/blockMeshDict
	
	    $ runApplication blockMesh
	
	    $ cp 0/alpha.water.org 0/alpha.water
	
	    $ runApplication setFields  
	    
    
7.  在头节点上，你会看到示例数据文件已复制到 C:\\OpenFoam\\sloshingTank3D 中。（C:\\OpenFoam 是头节点上的共享文件夹。）

    ![头节点上的数据文件][data_files]

### mpirun 的主机文件

在此步骤中，你将创建一个可供 **mpirun** 命令使用的主机文件（计算节点的列表）。

1.	在其中一个 Linux 节点的 /openfoam 下创建名为 hostfile 的新文件，使用户可以在所有 Linux 节点的 /openfoam/hostfile 位置访问此文件。

2.	将 Linux 节点名称写入此文件中。在此示例中，该文件如下所示：

	       
	    SUSE12RDMA-LN1
	    SUSE12RDMA-LN2
	    
    
    >[AZURE.TIP]你还可以在头节点的 C:\\OpenFoam\\hostfile 中创建此文件。如果这样做，你可以将你的脚本另存为带有 Linux 换行（仅 LF，而不是 CR LF）的文本文件。这可确保其在 Linux 节点上正常运行。

    **Bash 脚本包装**

    如果你有多个 Linux 节点，并且你的作业仅在部分节点上运行，则最好不要使用固定的主机文件，因为你不知道哪些节点会分配给你的作业。在这种情况下，可以为 **mpirun** 编写一个 bash 脚本包装，以便自动创建主机文件。你可以在本文末尾的示例文件中找到一个示例性的名为 hpcimpirun.sh 的 bash 脚本包装，然后将其另存为 /openfoam/hpcimpirun.sh。此示例脚本执行以下任务：

    1.	设置 **mpirun** 的环境变量以及其他命令参数，以便通过 RDMA 网络运行 MPI 作业。在本示例中，设置如下：

        *	I\_MPI\_FABRICS=shm:dapl
        *	I\_MPI\_DAPL\_PROVIDER=ofa-v2-ib0
        *	I\_MPI\_DYNAMIC\_CONNECTION=0

    2.	根据环境变量 $CCP\_NODES\_CORES 创建主机文件，该变量是在激活作业时由 HPC 头节点设置的。

        $CCP\_NODES\_CORES 的格式遵循以下模式：

        ```
        <Number of nodes> <Name of node1> <Cores of node1> <Name of node2> <Cores of node2>...`
        ```

        其中

        * `<Number of nodes>`：分配给此作业的节点数。  
        
        * `<Name of node_n_...>`：分配给此作业的每个节点的名称。
        
        * `<Cores of node_n_...>`：分配给此作业的节点上的内核数。

        例如，如果作业需要 2 个节点才能运行，则 $CCP\_NODES\_CORES 将类似于

	
	        2 SUSE12RDMA-LN1 8 SUSE12RDMA-LN2 8
	        
        
    3.	调用 **mpirun** 命令并将 2 个参数追加到命令行。

        * `--hostfile <hostfilepath>: <hostfilepath>` - 该脚本创建的主机文件的路径

        * `-np ${CCP_NUMCPUS}: ${CCP_NUMCPUS}` - HPC Pack 头节点设置的环境变量，用于存储分配给此作业的内核的总数。在本示例中，它指定 **mpirun** 的进程数。


## 提交 OpenFOAM 作业

现在，你可以在 HPC 群集管理器中提交作业。部分作业任务需要将脚本 hpcimpirun.sh 传递到命令行中。

1. 连接到你的群集头节点，然后启动 HPC 群集管理器。

2. 在“资源管理”中，确保 Linux 计算节点处于“联机”状态。如果节点未处于联机状态，请选择它们并单击“联机”。

3.  在“作业管理”中，单击“新作业”。

4.  为作业输入一个名称，如 _sloshingTank3D_。

    ![作业详细信息][job_details]

5.	在“作业资源”中，将资源类型选为“节点”，并将“最小数量”设置为 2。这样就会在 2 个 Linux 节点上运行此作业，每个节点有 8 个内核（仅限本示例）。

    ![作业资源][job_resources]

6.	向作业添加 4 个任务，任务的命令行和设置如下。

    >[AZURE.NOTE]运行 `source /openfoam/settings.sh` 将设置 OpenFOAM 和 MPI 运行时环境，因此下述每个任务会在调用 OpenFOAM 命令之前调用它。

    *   **任务 1**。运行 **decomposePar**，以便生成并行运行 **interDyMFoam** 所需的数据文件。
    
        *   为任务分配 1 个节点

        *   **命令行** - `source /openfoam/settings.sh && decomposePar -force > /openfoam/decomposePar${CCP_JOBID}.log`
    
        *   **工作目录** - /openfoam/sloshingTank3D
        
        请参阅下图。以相似的方式配置剩余任务。

        ![任务 1 详细信息][task_details1]

    *   **任务 2**。并行运行 **interDyMFoam** 以计算示例。

        *   为任务分配 2 个节点

        *   **命令行** - `source /openfoam/settings.sh && /openfoam/hpcimpirun.sh interDyMFoam -parallel > /openfoam/interDyMFoam${CCP_JOBID}.log`

        *   **工作目录** - /openfoam/sloshingTank3D

    *   **任务 3**。运行 **reconstructPar**，将每个 processor\_N\_ 目录的时间目录集合并到一个时间目录集中。

        *   为任务分配 1 个节点

        *   **命令行** - `source /openfoam/settings.sh && reconstructPar > /openfoam/reconstructPar${CCP_JOBID}.log`

        *   **工作目录** - /openfoam/sloshingTank3D

    *   **任务 4**。并行运行 **foamToEnsight**，将 OpenFOAM 结果文件转换成 EnSight 格式，然后将 EnSight 文件置于示例目录的名为 Ensight 的目录中。

        *   为任务分配 2 个节点

        *   **命令行** - `source /openfoam/settings.sh && /openfoam/hpcimpirun.sh foamToEnsight -parallel > /openfoam/foamToEnsight${CCP_JOBID}.log`

        *   **工作目录** - /openfoam/sloshingTank3D

6.	按任务顺序的升序将依赖项添加到这些任务中。

    ![任务依赖项][task_dependencies]

7.	单击“提交”运行此作业。

    默认情况下，HPC Pack 将以当前登录的用户帐户提交作业。单击“提交”后，你可能会看到一个对话框，提示你输入用户名和密码。

    ![作业凭据][creds]

    在某些情况下，HPC Pack 会记住你之前输入的用户信息，并不会显示此对话框。为了使 HPC Pack 再次显示此对话框，在“命令提示符”窗口中输入以下命令，然后提交此作业。

	
	    hpccred delcreds
	    

8.	根据你为示例设置的参数的不同，此作业可能需要数十分钟到数小时的时间才能完成。在热度地图中，你会看到该作业在 2 个 Linux 节点上运行。

    ![热度地图][heat_map]

    每个节点上启动了 8 个进程。

    ![Linux 进程][linux_processes]

9.  作业结束时，可在 C:\\OpenFoam\\sloshingTank3D 下的文件夹中找到作业结果，在 C:\\OpenFoam 中找到日志文件。


## 在 EnSight 中查看结果

（可选）使用 [EnSight](https://www.ceisoftware.com/) 将 OpenFOAM 作业的结果可视化并对其进行分析。有关在 EnSight 中实现可视化和动画效果的详细信息，请参阅此[视频指南](http://www.ceisoftware.com/wp-content/uploads/screencasts/vof_visualization/vof_visualization.html)。

1.  在头节点上安装 EnSight 后，启动它。

2.  打开 C:\\OpenFoam\\sloshingTank3D\\EnSight\\sloshingTank3D.case。

    你会在查看器中看到一个储槽。

    ![EnSight 中的罐][tank]

3.	从 **internalMesh** 创建“等值面”，然后选择变量 **alpha\_water**。

    ![创建等值面][isosurface]

4.	为前一步骤中创建的 **Isosurface\_part** 设置颜色。例如，将其设置为水蓝色。

    ![编辑等值面颜色][isosurface_color]

5.  从“罐壁”创建“等值体”，方法是：在“部件”面板中选择“罐壁”，然后单击工具栏中的“等值面”按钮。

6.	在对话框中，选择“等值体”作为“类型”，然后将“等值体范围”的最小值设置为 0.5。单击“使用所选部件创建”创建等值体。

7.	为前一步骤中创建的 **Iso\_volume\_part** 设置颜色。例如，将其设置为深水蓝色。

8.	设置“罐壁”的颜色。例如，将其设置为透明的白色。

9. 现在，请单击“播放”查看模拟结果。

    ![罐的结果][tank_result]

## 示例文件


### 示例 cred.xml 文件

	
	<ExtendedData>
	  <PrivateKey>-----BEGIN RSA PRIVATE KEY-----
	MIIEpQIBAAKCAQEAxJKBABhnOsE9eneGHvsjdoXKooHUxpTHI1JVunAJkVmFy8JC
	qFt1pV98QCtKEHTC6kQ7tj1UT2N6nx1EY9BBHpZacnXmknpKdX4Nu0cNlSphLpru
	lscKPR3XVzkTwEF00OMiNJVknq8qXJF1T3lYx3rW5EnItn6C3nQm3gQPXP0ckYCF
	Jdtu/6SSgzV9kaapctLGPNp1Vjf9KeDQMrJXsQNHxnQcfiICp21NiUCiXosDqJrR
	AfzePdl0XwsNngouy8t0fPlNSngZvsx+kPGh/AKakKIYS0cO9W3FmdYNW8Xehzkc
	VzrtJhU8x21hXGfSC7V0ZeD7dMeTL3tQCVxCmwIDAQABAoIBAQCve8Jh3Wc6koxZ
	qh43xicwhdwSGyliZisoozYZDC/ebDb/Ydq0BYIPMiDwADVMX5AqJuPPmwyLGtm6
	9hu5p46aycrQ5+QA299g6DlF+PZtNbowKuvX+rRvPxagrTmupkCswjglDUEYUHPW
	05wQaNoSqtzwS9Y85M/b24FfLeyxK0n8zjKFErJaHdhVxI6cxw7RdVlSmM9UHmah
	wTkW8HkblbOArilAHi6SlRTNZG4gTGeDzPb7fYZo3hzJyLbcaNfJscUuqnAJ+6pT
	iY6NNp1E8PQgjvHe21yv3DRoVRM4egqQvNZgUbYAMUgr30T1UoxnUXwk2vqJMfg2
	Nzw0ESGRAoGBAPkfXjjGfc4HryqPkdx0kjXs0bXC3js2g4IXItK9YUFeZzf+476y
	OTMQg/8DUbqd5rLv7PITIAqpGs39pkfnyohPjOe2zZzeoyaXurYIPV98hhH880uH
	ZUhOxJYnlqHGxGT7p2PmmnAlmY4TSJrp12VnuiQVVVsXWOGPqHx4S4f9AoGBAMn/
	vuea7hsCgwIE25MJJ55FYCJodLkioQy6aGP4NgB89Azzg527WsQ6H5xhgVMKHWyu
	Q1snp+q8LyzD0i1veEvWb8EYifsMyTIPXOUTwZgzaTTCeJNHdc4gw1U22vd7OBYy
	nZCU7Tn8Pe6eIMNztnVduiv+2QHuiNPgN7M73/x3AoGBAOL0IcmFgy0EsR8MBq0Z
	ge4gnniBXCYDptEINNBaeVStJUnNKzwab6PGwwm6w2VI3thbXbi3lbRAlMve7fKK
	B2ghWNPsJOtppKbPCek2Hnt0HUwb7qX7Zlj2cX/99uvRAjChVsDbYA0VJAxcIwQG
	TxXx5pFi4g0HexCa6LrkeKMdAoGAcvRIACX7OwPC6nM5QgQDt95jRzGKu5EpdcTf
	g4TNtplliblLPYhRrzokoyoaHteyxxak3ktDFCLj9eW6xoCZRQ9Tqd/9JhGwrfxw
	MS19DtCzHoNNewM/135tqyD8m7pTwM4tPQqDtmwGErWKj7BaNZARUlhFxwOoemsv
	R6DbZyECgYEAhjL2N3Pc+WW+8x2bbIBN3rJcMjBBIivB62AwgYZnA2D5wk5o0DKD
	eesGSKS5l22ZMXJNShgzPKmv3HpH22CSVpO0sNZ6R+iG8a3oq4QkU61MT1CfGoMI
	a8lxTKnZCsRXU1HexqZs+DSc+30tz50bNqLdido/l5B4EJnQP03ciO0=
	-----END RSA PRIVATE KEY-----</PrivateKey>
	  <PublicKey>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDEkoEAGGc6wT16d4Ye+yN2hcqigdTGlMcjUlW6cAmRWYXLwkKoW3WlX3xAK0oQdMLqRDu2PVRPY3qfHURj0EEellpydeaSekp1fg27Rw2VKmEumu6Wxwo9HddXORPAQXTQ4yI0lWSerypckXVPeVjHetbkSci2foLedCbeBA9c/RyRgIUl227/pJKDNX2Rpqly0sY82nVWN/0p4NAyslexA0fGdBx+IgKnbU2JQKJeiwOomtEB/N492XRfCw2eCi7Ly3R8+U1KeBm+zH6Q8aH8ApqQohhLRw71bcWZ1g1bxd6HORxXOu0mFTzHbWFcZ9ILtXRl4Pt0x5Mve1AJXEKb username@servername;</PublicKey>
	</ExtendedData>
	
### 示例 silent.cfg 文件

	
	
		# Patterns used to check silent configuration file
		#
		# anythingpat - any string
		# filepat     - the file location pattern (/file/location/to/license.lic)
		# lspat       - the license server address pattern (0123@hostname)
		# snpat       - the serial number pattern (ABCD-01234567)
		
		# accept EULA, valid values are: {accept, decline}
		ACCEPT_EULA=accept
		
		# optional error behavior, valid values are: {yes, no}
		CONTINUE_WITH_OPTIONAL_ERROR=yes
		
		# install location, valid values are: {/opt/intel, filepat}
		PSET_INSTALL_DIR=/opt/intel
		
		# continue with overwrite of existing installation directory, valid values are: {yes, no}
		CONTINUE_WITH_INSTALLDIR_OVERWRITE=yes
		
		# list of components to install, valid values are: {ALL, DEFAULTS, anythingpat}
		COMPONENTS=DEFAULTS
		
		# installation mode, valid values are: {install, modify, repair, uninstall}
		PSET_MODE=install
		
		# directory for non-RPM database, valid values are: {filepat}
		#NONRPM_DB_DIR=filepat
		
		# Serial number, valid values are: {snpat}
		#ACTIVATION_SERIAL_NUMBER=snpat
		
		# License file or license server, valid values are: {lspat, filepat}
		#ACTIVATION_LICENSE_FILE=
		
		# Activation type, valid values are: {exist_lic, license_server, license_file, trial_lic, serial_number}
		ACTIVATION_TYPE=trial_lic
		
		# Path to the cluster description file, valid values are: {filepat}
		#CLUSTER_INSTALL_MACHINES_FILE=filepat
		
		# Intel(R) Software Improvement Program opt-in, valid values are: {yes, no}
		PHONEHOME_SEND_USAGE_DATA=no
		
		# Perform validation of digital signatures of RPM files, valid values are: {yes, no}
		SIGNING_ENABLED=yes
		
		# Select yes to enable mpi-selector integration, valid values are: {yes, no}
		ENVIRONMENT_REG_MPI_ENV=no
		
		# Select yes to update ld.so.conf, valid values are: {yes, no}
		ENVIRONMENT_LD_SO_CONF=no
	
	

### 示例 settings.sh 脚本

	
	
		#!/bin/bash
		
		# impi
		source /opt/intel/impi/5.0.3.048/bin64/mpivars.sh
		export MPI_ROOT=$I_MPI_ROOT
		export I_MPI_FABRICS=shm:dapl
		export I_MPI_DAPL_PROVIDER=ofa-v2-ib0
		export I_MPI_DYNAMIC_CONNECTION=0
		
		# openfoam
		export FOAM_INST_DIR=/opt/OpenFOAM
		source /opt/OpenFOAM/OpenFOAM-2.3.1/etc/bashrc
		export WM_MPLIB=INTELMPI
	


###实例 hpcimpirun.sh 脚本

	
	
		#!/bin/bash
		
		# The path of this script
		SCRIPT_PATH="$( dirname "${BASH_SOURCE[0]}" )"
		
		# Set mpirun runtime evironment
		source /opt/intel/impi/5.0.3.048/bin64/mpivars.sh
		export MPI_ROOT=$I_MPI_ROOT
		export I_MPI_FABRICS=shm:dapl
		export I_MPI_DAPL_PROVIDER=ofa-v2-ib0
		export I_MPI_DYNAMIC_CONNECTION=0
		
		# mpirun command
		MPIRUN=mpirun
		# Argument of "--hostfile"
		NODELIST_OPT="--hostfile"
		# Argument of "-np"
		NUMPROCESS_OPT="-np"
		
		# Get node information from ENVs
		NODESCORES=(${CCP_NODES_CORES})
		COUNT=${#NODESCORES[@]}
		
		if [ ${COUNT} -eq 0 ]
		then
			# CCP_NODES_CORES is not found or is empty, just run the mpirun without hostfile arg.
			${MPIRUN} $*
		else
			# Create the hostfile file
			NODELIST_PATH=${SCRIPT_PATH}/hostfile_$$
		
			# Get every node name and write into the hostfile file
			I=1
			while [ ${I} -lt ${COUNT} ]
			do
				echo "${NODESCORES[${I}]}" >> ${NODELIST_PATH}
				let "I=${I}+2"
			done
		
			# Run the mpirun with hostfile arg
			${MPIRUN} ${NUMPROCESS_OPT} ${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*
		
			RTNSTS=$?
			rm -f ${NODELIST_PATH}
		fi
		
		exit ${RTNSTS}
	
	





<!--Image references-->
[keygen]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/keygen.png
[keys]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/keys.png
[step_variables]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/step_variables.png
[data_files]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/data_files.png
[decompose]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/decompose.png
[job_details]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/job_details.png
[job_resources]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/job_resources.png
[task_details1]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/task_details1.png
[task_dependencies]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/task_dependencies.png
[creds]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/creds.png
[heat_map]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/heat_map.png
[tank]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/tank.png
[tank_result]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/tank_result.png
[isosurface]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/isosurface.png
[isosurface_color]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/isosurface_color.png
[linux_processes]: ./media/virtual-machines-linux-classic-hpcpack-cluster-openfoam/linux_processes.png

<!---HONumber=Mooncake_0118_2016-->