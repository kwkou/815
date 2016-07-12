<properties
 pageTitle="设置 Windows RDMA 群集以运行 MPI 应用程序 | Azure"
 description="了解如何创建 Windows HPC Pack 群集，以使用 Azure RDMA 网络来运行 MPI 应用。"
 services="virtual-machines-windows"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="04/14/2016"
	wacn.date="05/12/2016"/>

# 使用 HPC Pack 设置一个运行 MPI 应用程序的 Windows RDMA 群集

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

使用 [Microsoft HPC Pack](https://technet.microsoft.com/zh-cn/library/cc514029) 在 Azure 中设置一个 Windows RDMA 群集，以运行并行消息传递接口 (MPI) 应用程序。当你将基于 Windows Server 的实例设置为在 HPC Pack 群集中运行时，MPI 应用程序将通过 Azure 中基于远程直接内存访问 (RDMA) 技术的低延迟、高吞吐量网络高效地进行通信。

## HPC Pack 群集部署选项
Microsoft HPC Pack 是一个免费提供的工具，可在 Azure 中创建基于 Windows Server 的 HPC 群集。HPC Pack 中包括适用于 Windows 消息传递接口的 Microsoft 实现 (MS-MPI) 的运行时环境。

本文将介绍使用 Microsoft HPC Pack 部署群集实例的以下两种方案。

* 方案 1.部署计算密集型辅助角色实例 (PaaS)

* 方案 2.在计算密集型 VM (IaaS) 中部署计算节点

## 先决条件

* **Azure 订阅** - 如果你没有 Azure 订阅，只需要花费几分钟就能创建一个[1元帐户](/pricing/1rmb-trial/)。

* **内核配额** - 你可能需要增加内核配额才能部署 VM 的群集。例如，若要使用 HPC Pack 部署 8 个实例，则至少需要 128 个核心。若要增加配额，可免费建立[联机客户支持请求](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。

##<a name="scenario-1.-deploy-compute-intensive-worker-role-instances-(PaaS)"></a> 方案 1.部署计算密集型辅助角色实例 (PaaS)

从现有的 HPC Pack 群集，添加运行在云服务 (PaaS) 中的 Azure 辅助角色实例（Azure 节点）形式的额外计算资源。此功能（也称为从 HPC Pack“迸发到 Azure”）支持一定范围的辅助角色实例大小。

以下是从现有（通常在本地）群集迸发到 Azure 实例的注意事项和步骤。可以使用类似的过程，将辅助角色实例添加到部署在 Azure VM 中的 HPC Pack 头节点。

>[AZURE.NOTE] 有关使用 HPC Pack 迸发到 Azure 的教程，请参阅[使用 HPC Pack 设置混合群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster/)。

![迸发到 Azure][burst]

### 使用实例时的注意事项

* **代理节点** - 在每个使用计算密集型实例“迸发到 Azure”的部署中，除了你指定的 Azure 辅助角色实例外，HPC Pack 至少还会将 2 个实例自动部署为代理节点。代理节点使用分配给订阅的内核，并会与 Azure 辅助角色实例一起产生费用。

### 步骤

4. **部署和配置 HPC Pack 2012 R2 头节点**

    从 [Microsoft 下载中心](https://www.microsoft.com/download/details.aspx?id=49922)下载最新的 HPC Pack 安装包。有关 Azure 迸发部署准备工作的要求和说明，请参阅 [HPC Pack Getting Started Guide（HPC Pack 入门指南）](https://technet.microsoft.com/zh-cn/library/jj884144.aspx)和 [Burst to Azure Worker Instances with Microsoft HPC Pack（使用 Microsoft HPC Pack 迸发到 Azure 辅助角色实例）](https://technet.microsoft.com/zh-cn/library/gg481749.aspx)。

5. **在 Azure 订阅中配置管理证书**

    配置证书以保护头节点与 Azure 之间的连接。有关选项和过程，请参阅[为 HPC Pack 配置 Azure 管理证书的方案](http://technet.microsoft.com/zh-cn/library/gg481759.aspx)。对于测试部署，HPC Pack 安装默认 Microsoft HPC Azure 管理证书，你可以将该证书快速上载到 Azure 订阅。

6. **创建新的云服务和存储帐户**

    使用 Azure 经典管理门户可以在提供计算密集型实例的区域中为部署创建云服务和存储帐户。

7. **创建 Azure 节点模板**

    在 HPC 群集管理器中使用“创建节点模板”向导。有关步骤，请参阅“使用 Microsoft HPC Pack 部署 Azure 节点的步骤”中的[创建 Azure 节点模板](http://technet.microsoft.com/zh-cn/library/gg481758.aspx#BKMK_Templ)。

    对于初始测试，我们建议在模板中配置手动可用性策略。

8. **向群集添加节点**

    在 HPC 群集管理器中使用“添加节点”向导。有关详细信息，请参阅[向 Windows HPC 群集中添加 Azure 节点](http://technet.microsoft.com/zh-cn/library/gg481758.aspx#BKMK_Add)。

9. **启动（预配）节点并使它们联机以运行作业**

    在 HPC 群集管理器中选择节点，并使用“启动”操作。完成预配后，在 HPC 群集管理器中选择节点并使用“联机”操作。节点将准备运行作业。

10. **向群集提交作业**

    使用 HPC Pack 作业提交工具来运行群集作业。请参阅 [Microsoft HPC Pack：作业管理](http://technet.microsoft.com/zh-cn/library/jj899585.aspx)。

11. **停止（取消预配）节点**

    完成运行作业后，使节点脱机，并在 HPC 群集管理器中使用“停止”操作。





## 方案 2.在计算密集型 VM (IaaS) 中部署计算节点

在此方案中，你可以将 HPC Pack 头节点和群集计算节点部署到 Azure 虚拟网络中已加入 Active Directory 域的 VM 中。HPC Pack 提供了许多 [Azure VM 中的部署选项](/documentation/articles/virtual-machines-windows-hpcpack-cluster-options/)，包括自动执行部署脚本和 Azure 快速入门模板。作为示例，以下注意事项和步骤将指导你使用 [HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)自动执行此过程中的大部分操作。

![Azure VM 中的群集][iaas]



### 步骤

1. **创建一个群集头节点，并通过在客户端计算机上运行 HPC Pack IaaS 部署脚本来计算节点 VM**

    从[ Microsoft 下载中心](https://www.microsoft.com/download/details.aspx?id=49922)下载 HPC Pack IaaS 部署脚本包。

    若要准备客户端计算机、创建脚本配置文件和运行脚本，请参阅[使用 HPC Pack IaaS 部署脚本创建 HPC 群集](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)。若要部署计算节点，请注意以下附加事项：
    
    * **虚拟网络** - 确保在实例可用的区域中指定一个新的虚拟网络。

    * **Windows Server 操作系统** - 若要支持 RDMA 连接，请为 VM 指定 Windows Server 2012 R2 或 Windows Server 2012 操作系统。

    * **云服务** - 我们建议你在一个云服务中部署头节点，在另一个云服务中部署计算节点。

    * **头节点大小** - 对于此方案，请考虑为头节点至少使用 A4 大小（特大）。

    * **HpcVmDrivers 扩展** - 当你部署运行 Windows Server 操作系统的计算节点时，部署脚本会自动安装 Azure VM 代理和 HpcVmDrivers 扩展。HpcVmDrivers 扩展将驱动程序安装在计算节点 VM 上，以便它们可以连接到 RDMA 网络。

    * **群集网络配置** - 部署脚本将自动设置拓扑 5 中的 HPC Pack 群集（企业网络上的所有节点）。此拓扑是 VM 中所有 HPC Pack 群集部署所必需的。以后请不要更改群集网络拓扑。

2. **使计算节点联机以运行作业**

    在 HPC 群集管理器中选择节点，并使用“联机”操作。节点将准备运行作业。

3. **向群集提交作业**

    连接到头节点以提交作业，或将本地计算机设置为执行此操作。有关信息，请参阅[向 Azure 中的 HPC 群集提交作业](/documentation/articles/virtual-machines-windows-hpcpack-cluster-submit-jobs/)。

4. **使节点脱机并停止（释放）节点**

    完成运行作业后，请在 HPC 群集管理器中使节点脱机。然后，使用 Azure 管理工具来关闭这些节点。



## 在实例上运行 MPI 应用程序

### 示例：在 HPC Pack 群集上运行 mpipingpong

若要验证计算密集型实例的 HPC Pack 部署，可以在群集上运行 HPC Pack **mpipingpong** 命令。**mpipingpong** 反复在配对节点之间发送数据包以计算启用 RDMA 的应用程序网络的延迟和吞吐量度量以及统计信息。本示例演示了使用群集 **mpiexec** 命令运行 MPI 作业（在本例中为 **mpipingpong**）的典型模式。

本示例假设你已在“迸发到 Azure”配置中添加了 Azure 节点（本文中的 [方案 1](#scenario-1.-deploy-compute-intensive-worker-role-instances-(PaaS))）。如果你在 Azure VM 群集中部署了 HPC Pack，则需要修改命令语法以指定不同的节点组，并设置其他环境变量以将网络流量定向到 RDMA 网络。


在群集上运行 mpipingpong：


1. 在头节点或正确配置的客户端计算机上，打开“命令提示符”。

2. 若要估计 4 节点的 Azure 迸发部署中节点对之间的延迟时间，请键入以下命令提交使用小数据包大小和大量迭代来运行 mpipingpong 的作业：

    ```
    job submit /nodegroup:azurenodes /numnodes:4 mpiexec -c 1 -affinity mpipingpong -p 1:100000 -op -s nul
    ```

    该命令将返回提交的作业的 ID。

    如果你在 Azure VM 上部署了 HPC Pack 群集，请指定包含单个云服务中部署的计算节点 VM 的节点组，并按如下所示修改 **mpiexec** 命令：

    ```
    job submit /nodegroup:vmcomputenodes /numnodes:4 mpiexec -c 1 -affinity -env MSMPI\_DISABLE\_SOCK 1 -env MSMPI\_PRECONNECT all -env MPICH\_NETMASK 172.16.0.0/255.255.0.0 mpipingpong -p 1:100000 -op -s nul
    ```

3. 完成该作业后，如果要查看输出（在此例中为作业的任务 1 的输出），请键入以下命令：

    ```
    task view <JobID>.1
    ```

    其中 &lt;JobID&gt; 为已提交作业的 ID。

    输出将包含类似于下面的延迟结果。

    ![弹性延迟][pingpong1]

4. 若要估计 Azure 迸发节点对之间的吞吐量，请键入以下命令提交使用大数据包大小和少量迭代来运行 **mpipingpong** 的作业：

    ```
    job submit /nodegroup:azurenodes /numnodes:4 mpiexec -c 1 -affinity mpipingpong -p 4000000:1000 -op -s nul
    ```

    该命令将返回提交的作业的 ID。

    在 Azure VM 上部署的 HPC Pack 群集中，修改步骤 2 中所示的命令。

5. 完成该作业后，如果要查看输出（在此例中为作业的任务 1 的输出），请键入以下命令：

    ```
    task view <JobID>.1
    ```

  输出将包含类似于下面的吞吐量结果。

  ![弹性吞吐量][pingpong2]


### MPI 应用程序注意事项


下面是在 Azure 实例上运行 MPI 应用程序的注意事项。某些注意事项只适用于 Azure 节点（在“迸发到 Azure”配置中添加的辅助角色实例）的部署。

* Azure 会定期对云服务中的辅助角色实例进行重新预配，且不会事先通知（例如，出于系统维护目的，或者在实例失败时）。如果在实例正在运行 MPI 作业时对它进行重新预配，则该实例将会丢失其所有数据，并且返回到首次部署它时的状态，这可能导致该 MPI 作业失败。你用于单个 MPI 作业的节点越多，该作业的运行时间就会越长，在某一作业正在运行时重新预配其中一个实例的可能性就越大。如果你在部署中将单个节点指定为文件服务器，则也应考虑此情况。

* 部署到 Azure 实例的应用程序将受到与应用程序关联的许可条款的限制。请向商业应用程序的供应商咨询有关在云中运行的许可或其他限制。并非所有供应商都提供即付即用许可。


* Azure 实例需要进一步设置才能访问本地节点、共享和许可证服务器。例如，要使 Azure 节点能够访问本地许可证服务器，你可以配置站点到站点 Azure 虚拟网络。


* 若要在 Azure 实例上运行 MPI 应用程序，必须通过运行 **hpcfwutil** 命令，向这些实例上的 Windows 防火墙注册每个 MPI 应用程序。这样便可以在防火墙动态分配的端口上进行 MPI 通信。

    >[AZURE.NOTE] 对于“迸发到 Azure”部署，你还可以配置一个防火墙例外命令，以便在添加到群集的所有新 Azure 节点上自动运行。在运行 **hpcfwutil** 命令并确认你的应用程序正常工作后，可以将该命令添加到 Azure 节点的启动脚本。有关详细信息，请参阅 [Use a Startup Script for Azure Nodes（对 Azure 节点使用启动脚本）](https://technet.microsoft.com/zh-cn/library/jj899632.aspx)。



* HPC Pack 使用 CCP\_MPI\_NETMASK 群集环境变量为 MPI 通信指定可接受地址范围。从 HPC Pack 2012 R2 开始，CCP\_MPI\_NETMASK 群集环境变量仅影响已加入域的群集计算节点（在本地或 Azure VM 中）之间的 MPI 通信。该变量将被已添加到“迸发到 Azure”配置中的节点忽略。


* MPI 作业不能跨部署在不同云服务中的 Azure 实例运行（例如，不能在使用不同节点模板的“迸发到 Azure”部署中或部署在多个云服务中的 Azure VM 计算节点上运行）。如果你有使用不同节点模板启动的多个 Azure 节点部署，则 MPI 作业必须仅在一组 Azure 节点上运行。


* 在你向群集添加 Azure 节点并且使它们处于联机状态时，HPC 作业计划程序服务将会立即尝试启动这些节点上的作业。如果你只有一部分的工作负荷可以在 Azure 上运行，请确保更新或创建作业模板以定义可在 Azure 上运行的作业类型。例如，若要确保使用某一作业模板提交的作业仅在 Azure 节点上运行，你可以向该作业模板中添加“节点组”属性并且选择 AzureNodes 作为所需值。若要为你的 Azure 节点创建自定义组，可以使用 Add-HpcGroup HPC PowerShell cmdlet。


## 后续步骤

* 使用 HPC Pack 的替代方法是使用 Azure Batch 服务进行开发，以便在 Azure 中的计算节点池上运行 MPI 应用程序。请参阅 [Use multi-instance tasks to run Message Passing Interface (MPI) applications in Azure Batch（在 Azure Batch 中使用多实例任务来执行消息传递接口 (MPI) 应用程序）](/documentation/articles/batch-mpi/)。

<!--Image references-->
[burst]: ./media/virtual-machines-windows-classic-hpcpack-rdma-cluster/burst.png
[iaas]: ./media/virtual-machines-windows-classic-hpcpack-rdma-cluster/iaas.png
[pingpong1]: ./media/virtual-machines-windows-classic-hpcpack-rdma-cluster/pingpong1.png
[pingpong2]: ./media/virtual-machines-windows-classic-hpcpack-rdma-cluster/pingpong2.png

<!---HONumber=Mooncake_0503_2016-->