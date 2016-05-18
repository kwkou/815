<properties
   pageTitle="用于部署 Linux HPC 群集的 PowerShell 脚本 | Azure"
   description="运行 PowerShell 脚本，以在 Azure 基础结构服务中部署 Linux HPC Pack 群集"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""
   tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-linux"
	ms.date="04/05/2016"
	wacn.date="05/12/2016"/>

# 使用 HPC Pack IaaS 部署脚本创建 Linux 高性能计算 (HPC) 群集

在客户端计算机上运行 HPC Pack IaaS 部署 PowerShell 脚本，以在 Azure 基础结构服务 (IaaS) 中部署适用于 Linux 工作负荷的完整 HPC 群集。如果你想要在 Azure 中部署适用于 Windows 工作负荷的 HPC Pack 群集，请参阅 [Create a Windows HPC cluster with the HPC Pack IaaS deployment script（使用 HPC Pack IaaS 部署脚本创建 Windows HPC 群集）](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script)。

[AZURE.INCLUDE [virtual-machines-common-classic-hpcpack-cluster-powershell-script](../includes/virtual-machines-common-classic-hpcpack-cluster-powershell-script.md)]

##<a name="Example-configuration-files"></a>示例配置文件

### 示例 1

以下配置文件将创建新的域林并部署 HPC Pack 群集，其中包含 1 个具有本地数据库的头节点和 20 个 Linux 计算节点。所有云服务直接在“中国东部”位置创建。Linux 计算节点在 4 个云服务和 4 个存储帐户中创建（即，MyLnxCNService01 和 mylnxstorage01 中的 MyLnxCN-0001 到 MyLnxCN-0005；MyLnxCNService02 和 mylnxstorage02 中的 MyLnxCN-0006 到 MyLnxCN-0010；MyLnxCNService03 和 mylnxstorage03 中的 MyLnxCN-0011 到 MyLnxCN-0015；MyLnxCNService04 和 mylnxstorage04 中的 MyLnxCN-0016 到 MyLnxCN-0020）。计算节点是基于 OpenLogic CentOS 7.0 版 Linux 映像创建的。

	<?xml version="1.0" encoding="utf-8" ?>
	<IaaSClusterConfig>
	  <Subscription>
	    <SubscriptionName>Subscription-1</SubscriptionName>
	    <StorageAccount>mystorageaccount</StorageAccount>
	  </Subscription>
	  <Location>China East</Location>  
	  <VNet>
	    <VNetName>MyVNet</VNetName>
	    <SubnetName>Subnet-1</SubnetName>
	  </VNet>
	  <Domain>
	    <DCOption>NewDC</DCOption>
	    <DomainFQDN>hpc.local</DomainFQDN>
	    <DomainController>
	      <VMName>MyDCServer</VMName>
	      <ServiceName>MyHPCService</ServiceName>
	      <VMSize>Large</VMSize>
	    </DomainController>
	  </Domain>
	  <Database>
	    <DBOption>LocalDB</DBOption>
	  </Database>
	  <HeadNode>
	    <VMName>MyHeadNode</VMName>
	    <ServiceName>MyHPCService</ServiceName>
	    <VMSize>ExtraLarge</VMSize>
	  </HeadNode>
	  <LinuxComputeNodes>
	    <VMNamePattern>MyLnxCN-%0001%</VMNamePattern>
	    <ServiceNamePattern>MyLnxCNService%01%</ServiceNamePattern>
	    <MaxNodeCountPerService>5</MaxNodeCountPerService>
	    <StorageAccountNamePattern>mylnxstorage%01%</StorageAccountNamePattern>
	    <VMSize>Medium</VMSize>
	    <NodeCount>20</NodeCount>
	    <ImageName>5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20150325 </ImageName>
	    <SSHKeyPairForRoot>
	      <PfxFile>d:\mytestcert1.pfx</PfxFile>
	      <Password>MyPsw!!2</Password>
	    </SSHKeyPairForRoot>
	  </LinuxComputeNodes>
	</IaaSClusterConfig>

## 故障排除

* **“虚拟网络不存在”错误** - 如果你运行 HPC Pack IaaS 部署脚本以在 Azure 中的一个订阅下同时部署多个群集，则一个或多个部署可能会失败并显示错误“虚拟网络 VNet\_Name 不存在”。如果发生此错误，请对失败的部署重新运行该脚本。

* **从 Azure 虚拟网络访问 Internet 时出现问题** - 如果你通过使用部署脚本创建 HPC Pack 群集和新的域控制器，或者你手动将头节点 VM 提升为域控制器，则在将 Azure 虚拟网络中的 VM 连接到 Internet 时可能会遇到问题。如果已在域控制器上自动配置转发器 DNS 服务器，但此转发器 DNS 服务器未正确解析，则会出现这种情况。

    若要解决此问题，请登录到域控制器，删除转发器配置设置或配置一个有效的转发器 DNS 服务器。为此，请在服务器管理器中单击“工具”>“DNS”以打开 DNS 管理器，然后双击“转发器”。
    
## 后续步骤

* 有关使用脚本创建群集和运行 HPC 工作负荷的教程，请参阅 [Run NAMD with Microsoft HPC Pack on Linux compute nodes in Azure（在 Azure 中的 Linux 计算节点上使用 Microsoft HPC Pack 运行 NAMD）](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster-namd)或 [Run OpenFOAM with Microsoft HPC Pack on Linux compute nodes in Azure（在 Azure 中的 Linux 计算节点上使用 Microsoft HPC Pack 运行 OpenFOAM）](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster-openfoam)。

<!---HONumber=Mooncake_0503_2016-->