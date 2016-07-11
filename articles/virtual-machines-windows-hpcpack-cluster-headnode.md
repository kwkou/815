<properties
 pageTitle="在 Azure VM 中创建 HPC Pack 头节点 | Azure"
 description="了解如何使用 Azure 门户预览和 Resource Manager 部署模型在 Azure VM 中创建 Microsoft HPC Pack 头节点。"
 services="virtual-machines-windows"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,hpc-pack"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="05/19/2016"
	wacn.date="07/11/2016"/>

# 在 Azure VM 中使用应用商店映像创建 HPC Pack 群集的头节点


使用 Azure 库中的 [Microsoft HPC Pack 虚拟机映像](https://azure.microsoft.com/marketplace/partners/microsoft/hpcpack2012r2onwindowsserver2012r2/)，通过 Azure 门户预览创建 HPC 群集的头节点。此 HPC Pack VM 映像基于预安装了 HPC Pack 2012 R2 Update 3 的 Windows Server 2012 R2 Datacenter。使用此头节点在 Azure 中进行 HPC Pack 的概念证明部署。然后，可以向该群集添加计算节点，以运行 HPC 工作负荷。



>[AZURE.TIP]若要在 Azure 中部署包括头节点和计算节点的完整 HPC Pack 群集，建议使用自动化的方法，如 [HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script)，或使用 Resource Manager 模板，如 [HPC Pack cluster for Windows workloads](https://azure.microsoft.com/marketplace/partners/microsofthpc/newclusterwindowscn/)（适用于 Windows 工作负荷的 HPC Pack 群集）模板。请参阅 [Azure 中的 HPC Pack 群集选项](/documentation/articles/virtual-machines-windows-hpcpack-cluster-options)获取其他模板。


## 规划注意事项

如下图所示，将在 Azure 虚拟网络的 Active Directory 域中部署头节点。

![HPC Pack 头节点][headnode]

* **Active Directory 域** - 必须先将 HPC Pack 头节点加入到 Azure 中的 Active Directory 域，再在 VM 上启动 HPC 服务。如本文所示，若要进行概念证明部署，可以先将为头节点创建的 VM 提升为域控制器，然后再启动 HPC 服务。另一种方法是在 Azure 中部署单独的域控制器和林，并将头节点 VM 加入到该域林。

* **Azure 虚拟网络** - 如本文所示，当在 Azure 门户预览中使用 Resource Manager 部署模型部署 HPC Pack 头节点时，需要指定或创建 Azure 虚拟网络。之后，需要使用虚拟网络将群集计算节点 VM 添加到 HPC 群集，或者将头节点加入到现有的 Active Directory 域。

    
## 创建头节点的步骤

以下是在 Azure 门户预览中使用 Resource Manager 部署模型为 HPC Pack 头节点创建 Azure VM 的高级步骤。


1. 如果希望使用单独的域控制器 VM 在 Azure 中创建新的 Active Directory 林，其中一种方法是使用 [Resource Manager 模板](https://azure.microsoft.com/documentation/templates/active-directory-new-domain-ha-2-dc/)。如果是进行简单的 HPC Pack 概念证明部署，则可以忽略这种方法。而需要按照本文后续部分所述，将头节点 VM 本身配置为域控制器。
    
2. 若要在 [Azure 门户预览](https://portal.azure.cn)中创建 HPC Pack 头节点，请从 Azure 库中选择 HPC Pack 2012 R2 映像。若要实现此操作，请单击“新建”，并在应用商店中搜索“HPC Pack”。选择“Windows Server 2012 R2 上的 HPC Pack 2012 R2”映像。

3. 在“Windows Server 2012 R2 上的 HPC Pack 2012 R2”页上，选择“Resource Manager”部署模型，然后单击“创建”。

    ![HPC Pack 映像][marketplace]

4. 使用门户配置设置并创建 VM。如果你不熟悉 Azure，请按照[在 Azure 门户预览中创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial)教程中的说明进行操作。若要进行概念证明部署，通常可以接受很多默认或推荐的设置。

    **虚拟网络注意事项**

   * 如果希望将头节点加入到 Azure 中的现有 Active Directory 域，请确保在创建头节点 VM 时为该域指定了虚拟网络。
   
   * 如果希望创建新的虚拟网络，请在“设置”中，指定专用网络地址范围（如 10.0.0.0/16）和子网地址范围（如 10.0.0.0/24）。
    
4. 创建 VM 并运行 VM 之后，通过远程桌面[连接到 VM](/documentation/articles/virtual-machines-windows-connect-logon)。 

5. 将 VM 加入到现有的域林，或在 VM 上创建一个新的域林。

    * 如果使用现有的域林在 Azure 虚拟网络中创建了 VM，请使用标准的 Server Manager 或 Windows PowerShell 工具将其加入到该域林。然后重新启动。

    * 如果未使用现有域林在新的虚拟网络中创建 VM，则将该 VM 提升为域控制器。若要实现此操作，请使用标准的 Server Manager 或 Windows PowerShell 工具来安装和配置 Active Directory 域服务角色。有关详细步骤，请参阅[安装新的 Windows Server 2012 Active Directory 林](https://technet.microsoft.com/zh-cn/library/jj574166.aspx)。

5. VM 运行且加入到 Active Directory 林后，在头节点启动 HPC Pack 服务。为此，请按以下步骤操作：

    a.使用一个属于本地管理员组的域帐户连接到 VM。例如，可以使用创建头节点 VM 时设置的管理员帐户。

    b.对于默认头节点配置，以管理员身份启动 Windows PowerShell 并键入以下命令：

    ```
    & $env:CCP_HOME\bin\HPCHNPrepare.ps1 -DBServerInstance ".\ComputeCluster"
    ```

    HPC Pack 服务启动可能需要几分钟时间。

    对于其他头节点配置选项，请键入 `get-help HPCHNPrepare.ps1`。


## 后续步骤

* 现在即可使用 HPC Pack 群集的头节点。例如，启动 HPC 群集管理器，并完成 [Deployment To-do List（部署待办事项列表）](https://technet.microsoft.com/zh-cn/library/jj884141.aspx)。

* 在云服务中添加 [Azure 迸发节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-burst)，按需提高群集计算容量。 

* 尝试在群集上运行测试工作负荷。例如，请参阅 HPC Pack [入门指南](https://technet.microsoft.com/zh-cn/library/jj884144)。

<!--Image references-->
[headnode]: ./media/virtual-machines-windows-hpcpack-cluster-headnode/headnode.png
[marketplace]: ./media/virtual-machines-windows-hpcpack-cluster-headnode/marketplace.png

<!---HONumber=Mooncake_0704_2016-->