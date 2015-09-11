<properties
      pageTitle="设置 Linux RDMA 群集以运行 MPI 应用程序 | Windows Azure"
      description="了解如何创建 A8 或 A9 大小 VM 的 Linux 群集以使用 RDMA 运行 MPI 应用。"
      services="virtual-machines"
      documentationCenter=""
      authors="dlepow"
      manager="timlt"
      editor=""/>
<tags
      ms.service="virtual-machines"
      ms.date="07/17/2015"
      wacn.date="08/29/2015"/>

# 设置 Linux RDMA 群集以运行 MPI 应用程序

本文介绍如何使用 [A8 和 A9 大小虚拟机](/documentation/articles/virtual-machines-a8-a9-a10-a11-specs)在 Azure 中设置 Linux RDMA 群集以运行并行消息传递接口 (MPI) 应用程序。当你配置了基于 Linux 的 A8 和 A9大小 VM 以运行支持的 MPI 实现时，MPI 应用程序将通过 Azure 中基于远程直接内存访问 (RDMA) 技术的低延迟、高吞吐量网络高效地进行通信。

>[AZURE.NOTE]目前，在 SUSE Linux Enterprise Server 12 (SLES 12) 上运行的 Intel MPI 库版本 5.0 支持 Azure Linux RDMA。
>
> Azure 还提供了 A10 和 A11 计算密集型实例，其处理能力与 A8 和 A9 实例相同，但无需连接到 RDMA 后端网络。若要在 Azure 中运行 MPI 工作负荷，通常使用 A8 和 A9 实例可获得最佳性能。


## Linux 群集部署选项

以下是可用于创建 Linux RDMA 群集的方法（使用或不使用作业计划程序）。

* **HPC Pack** - 你可以在 Azure 中创建 Microsoft HPC Pack 群集，然后添加运行支持的 Linux 分发（从 HPC Pack 2012 R2 Update 2 开始支持）的计算节点。可以将某些 Linux 节点配置为访问 RDMA 网络。若要开始使用，请参阅 <!--[-->HPC Pack 文档<!--](/documentation/articles/virtual-machines-linux-cluster-hpcpack)-->。

* **Azure CLI 脚本** - 你可以使用适用于 Mac、Linux 和 Windows 的 [Azure 命令行界面](/documentation/articles/xplat-cli) (CLI) 生成自己的脚本，以部署创建 Linux 群集所需的虚拟网络和其他必需的组件。CLI 在 Azure 服务管理 (asm) 模式下将依次部署群集节点，因此，如果你要部署许多计算节点，可能需要几分钟才能完成部署。有关示例，请参阅本文其余部分中的步骤。

* **Azure 资源管理器模板** - 通过创建简单的 Azure 资源管理器 JSON 模板文件和运行 Azure CLI arm 模式命令或使用 Azure 预览版门户，你可以部署多个 A8 和 A9 Linux VM，并定义虚拟网络、静态 IP 地址、DNS 设置和其他资源来创建可利用 RDMA 网络和运行 MPI 工作负荷的计算群集。你可以[创建自己的模板](/documentation/articles/resource-group-authoring-templates)或查看[“Azure 快速入门模板”页](https://azure.microsoft.com/documentation/templates/)获取由 Microsoft 或社区提供的模板以部署所需的解决方案。资源管理器模板通常提供最快且最可靠的方式来部署 Linux 群集。

## 在 Azure 服务管理中使用 Azure CLI 脚本进行部署

以下步骤将帮助你使用 Azure CLI 部署 SLES 12 VM、安装 Intel MPI 库和其他自定义项、创建自定义 VM 映像，然后编写脚本部署 A8 或 A9 VM 的群集。

### 先决条件

* **客户端计算机** - 你需要基于 Mac、Linux 或 Windows 的客户端计算机，以便与 Azure 通信。

* **Azure 订阅** - 如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](http://www.windowsazure.cn/pricing/1rmb-trial/)。

* **内核配额** - 你可能需要增加内核配额才能部署 A8 或 A9 VM 的群集。例如，如果你要按本文所示部署 8 个 A9 VM，则至少需要 128 个内核。若要增加配额，可免费[建立联机客户支持请求](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。

* **Azure CLI** - [安装](/documentation/articles/xplat-cli-install) Azure CLI 并[对其进行配置](/documentation/articles/xplat-cli-connect)，以便在客户端计算机上使用你的 Azure 订阅。


### 预配 SLES 12 VM

使用 Azure CLI 登录到 Azure 后，运行 `azure config list` 命令以确认 CLI 在 Azure 服务管理 (asm) 模式下。如果它不是，请通过运行以下命令设置模式：

```
azure config mode asm
```

键入以下内容以列出你有权使用的所有订阅：

```
azure account list
```

通过将 `Current` 设为 `true` 来标识当前的活动订阅。如果这不是你要用于创建群集的订阅，请将相应的订阅编号设为活动订阅：

```he
azure account set <subscription-number>
```

若要查看 Azure 中公开提供的 SLES 12 HPC 映像，请运行如下命令（如果你的 shell 环境支持 **grep**）：

```
azure vm image list | grep "suse.*hpc"
```

>[AZURE.NOTE]已为 SLES 12 HPC 映像预配置了必需的用于 Azure 的 Linux RDMA 驱动程序。

现在，通过运行如下命令使用提供的 SLES 12 HPC 映像预配 A9 大小 VM：

```
azure vm create -g <username> -p <password> -c <cloud-service-name> -l <location> -z A9 -n <vm-name> -e 10004 b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-v20150708
```

其中

* 大小（在此示例中为 A9）可以是 A8 或 A9

* 外部 SSH 端口号（在此示例中为 10004）是任何有效的端口号；内部 SSH 端口号将设为 22

* 将在 location 指定的 Azure 区域中创建新的云服务；请指定提供 A8 和 A9 实例的位置，例如“West US”

* 映像名称当前可以是`b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-v20150708`（免费）或 `b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-priority-v20150708`（用于 SUSE 优先支持，将付费）

### 自定义 VM

在 VM 完成预配后，使用 VM 的外部 IP 地址（或 DNS 名称）和你配置的外部端口号通过 SSH 登录到 VM，然后对其进行自定义。有关连接详细信息，请参阅[如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)。

>[AZURE.NOTE]Windows Azure 不提供对 Linux VM 的根访问权限。若要在以用户身份连接时获得管理访问权限，可以使用 `sudo –s`。

**更新** - 你可以使用 **zypper** 和 NFS 实用工具安装更新。

>[AZURE.IMPORTANT]目前，我们建议你不要应用内核更新，这可能会在使用 Linux RDMA 驱动程序时导致问题。

**Intel MPI** - 从 [Intel.com 站点](https://software.intel.com/zh-cn/intel-mpi-library/)下载并安装 Intel MPI 库 5.0 运行时。向 Intel 注册后，请点击确认电子邮件中的链接转到相关网页，并复制相应版本的 Intel MPI 所对应的 .tgz 文件的下载链接。

运行如下命令以在 VM 上安装 Intel MPI：

```

$ wget <download link for your registration>
$ tar xvzf <tar-file>
$ cd <mpi-directory>
$ sudo ./install.sh
```

**锁定内存** - 要使 MPI 代码锁定可用于 RDMA 的内存，需要在 /etc/security/limits.conf 文件中添加或更改以下设置：

```
<User or group name> hard    memlock <memory required for your application in KB>
<User or group name> soft    memlock <memory required for your application in KB>
```

>[AZURE.NOTE]出于测试目的，你也可以将 memlock 设置为不受限制。例如：`<User or group name>    hard    memlock unlimited`。


**SSH 密钥** - 在群集中的所有计算节点之间为用于创建此 VM 的用户名和密码建立信任。使用以下命令创建 SSH 密钥：

```
$ ssh-keygen
```

将公钥保存在默认位置并记住输入的通行短语。

```
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

在 /.ssh 目录中编辑或创建 ssh_config 文件。提供将在 Azure 中使用的专用网络的 IP 地址范围：

```
host 10.32.0.*
StrictHostKeyChecking no
```

或者，可以按以下方式列出群集中每个 VM 的专用网络 IP 地址：

```
host 10.32.0.1
 StrictHostKeyChecking no
host 10.32.0.2
 StrictHostKeyChecking no
host 10.32.0.3
 StrictHostKeyChecking no
```

>[AZURE.NOTE]如果未按上面所示指定特定 IP 地址或范围，配置 `StrictHostKeyChecking no` 就会产生潜在的安全风险。

**应用程序** - 在捕获映像之前，将所需的所有应用程序复制到此 VM 上或执行其他自定义设置。

### 捕获映像

若要捕获映像，首先在 Linux VM 中运行以下命令：

```
$ sudo waagent -deprovision
```

然后，从客户端计算机运行以下 Azure CLI 命令，以捕获映像。有关详细信息，请参阅[如何捕获 Linux 虚拟机以用作模板](/documentation/articles/virtual-machines-linux-capture-image)。

```
azure vm shutdown <vm-name>

azure vm capture -t <vm-name> <image-name>

```

运行这些命令之后，将捕获 VM 映像供你使用并将删除 VM。现已准备好用于部署群集的自定义映像。

### 使用映像部署群集

使用你环境的相应值修改以下脚本，并从客户端计算机运行它。由于 ASM 部署方法将依次部署 VM，因此将需要几分钟时间才能部署此脚本中建议的 8 个 A9 VM。

```
      ### Script to create a compute cluster without a scheduler in a VNet in Azure
      ### Create a custom private network in Azure
      ### Replace 10.32.0.0 with your virtual network address space
      ### Replace <network-name> with your network identifier
      ### Select a region where A8 and A9 VMs are available, such as West US
      ### See Azure Pricing pages for prices and availability of A8 and A9 VMs
      
      azure network vnet create -l "West US" –e 10.32.0.0 <network-name>
      
      ### Create a cloud service. All the A8 and A9 instances need to be in the same cloud service for Linux RDMA to work across InfiniBand.
      ### Note: The current maximum number of VMs in a cloud service is 50. If you need to provision more than 50 VMs in the same cloud service in your cluster, contact Azure Support.
      
      azure service create <cloud-service-name> --location "West US" –s <subscription-ID>
      
      ### Define a prefix naming scheme for compute nodes, e.g., cluster11, cluster12, etc.
      
      vmname=cluster
      
      ### Define a prefix for external port numbers. If you want to turn off external ports and use only internal ports to communicate between compute nodes via port 22, don’t use this option. Since port numbers up to 10000 are reserved, use numbers after 10000. Leave external port on for rank 0 and head node.
      
      portnumber=101
      
      ### In this cluster there will be 8 size A9 nodes, named cluster11 to cluster18. Specify your captured image in <image-name>.
      
      for (( i=11; i<19; i++ )); do
            azure vm create -g <username> -p <password> -c <cloud-service-name> -z A9 -n $vmname$i -e $portnumber$i <image-name>
      done
      
      ### Save this script and run it at the CLI prompt to provision your cluster
```

## 配置和运行 Intel MPI

若要在 Azure Linux RDMA 上运行 MPI 应用程序，你需要配置特定于 Intel MPI 的某些环境变量。下面是用于配置变量和运行应用程序的示例脚本。

```
      #!/bin/bash -x
      source /opt/intel/impi_latest/bin64/mpivars.sh
      
      export I_MPI_FABRICS=shm:dapl
      
      # THIS IS A MANDATORY ENVIRONMENT VARIABLE AND MUST BE SET BEFORE RUNNING ANY JOB
      # Setting the variable to shm:dapl gives best performance for some applications
      # If your application doesn’t take advantage of shared memory and MPI together, then set only dapl
      
      export I_MPI_DAPL_PROVIDER=ofa-v2-ib0
      
      # THIS IS A MANDATORY ENVIRONMENT VARIABLE AND MUST BE SET BEFORE RUNNING ANY JOB
      
      export I_MPI_DYNAMIC_CONNECTION=0
      
      # THIS IS A MANDATORY ENVIRONMENT VARIABLE AND MUST BE SET BEFORE RUNNING ANY JOB
      
      # Command line to run the job
      
      mpirun -n <number-of-cores> -ppn <core-per-node> -hostfile <hostfilename>  /path <path to the application exe> <arguments specific to the application>
      
      #end
```

主机文件的格式如下所示。针对群集中的每个节点添加一行。指定前面定义的虚拟网络中的专用 IP 地址，而不是 DNS 名称。

```
private ip address1:16 [e.g. 10.32.0.1:16]
private ip address2:16
...
```

## 安装 Intel MPI 后验证基本双节点群集

你可以运行以下 Intel MPI 命令，以通过使用 pingpong 基准来验证群集配置。

```
/opt/intel/impi_latest/bin64/mpirun -hosts <host1>, <host2> -ppn 1 -n 2 -env I_MPI_FABRICS dapl -env I_MPI_DAPL_PROVIDER=ofa-v2-ib0 -env I_MPI_DYNAMIC_CONNECTION=0

/opt/intel/impi_latest/bin64/IMB-MPI1 pingpong
```

你应在具有两个节点的工作群集上看到类似如下内容的输出：

```
      #------------------------------------------------------------
      #    Intel (R) MPI Benchmarks 4.0 Update 1, MPI-1 part
      #------------------------------------------------------------
      # Date                  : Fri Jul 17 23:16:46 2015
      # Machine               : x86_64
      # System                : Linux
      # Release               : 3.12.39-44-default
      # Version               : #5 SMP Thu Jun 25 22:45:24 UTC 2015
      # MPI Version           : 3.0
      # MPI Thread Environment:
      # New default behavior from Version 3.2 on:
      # the number of iterations per message size is cut down
      # dynamically when a certain run time (per message size sample)
      # is expected to be exceeded. Time limit is defined by variable
      # "SECS_PER_SAMPLE" (=> IMB_settings.h)
      # or through the flag => -time
      
      # Calling sequence was:
      # /opt/intel/impi_latest/bin64/IMB-MPI1 pingpong
      # Minimum message length in bytes:   0
      # Maximum message length in bytes:   4194304
      #
      # MPI_Datatype                   :   MPI_BYTE
      # MPI_Datatype for reductions    :   MPI_FLOAT
      # MPI_Op                         :   MPI_SUM
      #
      #
      # List of Benchmarks to run:
      # PingPong
      #---------------------------------------------------
      # Benchmarking PingPong
      # #processes = 2
      #---------------------------------------------------
            #bytes #repetitions      t[usec]   Mbytes/sec
                  0         1000         2.23         0.00
                  1         1000         2.26         0.42
                  2         1000         2.26         0.85
                  4         1000         2.26         1.69
                  8         1000         2.26         3.38
                 16         1000         2.36         6.45
                 32         1000         2.57        11.89
                 64         1000         2.36        25.81
                128         1000         2.64        46.19
                256         1000         2.73        89.30
                512         1000         3.09       157.99
               1024         1000         3.60       271.53
               2048         1000         4.46       437.57
               4096         1000         6.11       639.23
               8192         1000         7.49      1043.47
              16384         1000         9.76      1600.76
              32768         1000        14.98      2085.77
              65536          640        25.99      2405.08
             131072          320        50.68      2466.64
             262144          160        80.62      3101.01
             524288           80       145.86      3427.91
            1048576           40       279.06      3583.42
            2097152           20       543.37      3680.71
            4194304           10      1082.94      3693.63

      # All processes entering MPI_Finalize

```

## 网络拓扑注意事项

* 在 Linux VM 上，已为 RDMA 网络流量保留 Eth1。不要更改引用此网络的配置文件中的任何 Eth1 设置或任何信息。

* 在 Azure 中，不支持 IP over Infiniband (IB)。仅支持 RDMA over IB。

* 在 Linux VM 上，已为常规 RDMA 网络流量保留 Eth0。

## 后续步骤

* 在 Linux 群集上尝试部署和运行 Linux MPI 应用程序。

* 有关 Intel MPI 的指南，请参阅 [Intel MPI 库文档](https://software.intel.com/zh-cn/articles/intel-mpi-library-documentation/)。

<!---HONumber=67-->