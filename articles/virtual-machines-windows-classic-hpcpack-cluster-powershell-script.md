<properties
   pageTitle="用于部署 Windows HPC Pack 群集的 PowerShell 脚本 | Azure"
   description="运行 Windows PowerShell 脚本，以在 Azure 基础结构服务中部署 Windows HPC Pack 群集"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""
   tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="04/05/2016"
	wacn.date="05/24/2016"/>

# 使用 HPC Pack IaaS 部署脚本在 Azure VM 中创建 Windows 高性能计算 (HPC) 群集

在客户端计算机上运行 HPC Pack IaaS 部署 PowerShell 脚本，以在 Azure 基础结构服务 (IaaS) 中部署完整的 Windows 工作负荷 HPC Pack 群集。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

[AZURE.INCLUDE [virtual-machines-common-classic-hpcpack-cluster-powershell-script](../includes/virtual-machines-common-classic-hpcpack-cluster-powershell-script.md)]

##<a name="Example-configuration-files"></a> 示例配置文件

### 示例 1

以下配置文件将在现有域林中部署一个 HPC Pack 群集。该群集包含 1 个具有本地数据库的头节点和 12 个应用了 BGInfo VM 扩展的计算节点。对于域林中的所有 VM，禁用了 Windows 更新的自动安装。所有云服务直接在“中国东部”位置创建。计算节点在 3 个云服务和 3 个存储帐户中创建（即，_MyHPCCNService01_ 和 _mycnstorage01_ 中的 _MyHPCCN-0001_ 到 _MyHPCCN-0005_；_MyHPCCNService02_ 和 _mycnstorage02_ 中的 _MyHPCCN-0006_ 到 _MyHPCCN0010_；_MyHPCCNService03_ 和 _mycnstorage03_ 中的 _MyHPCCN-0011_ 到 _MyHPCCN-0012_）。计算节点是基于从计算节点捕获的现有专用映像创建的。已启用自动增长和收缩服务，该服务采用默认的增长和收缩间隔。

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
	     <NoWindowsAutoUpdate />
	  </Domain>
	  <Database>
	    <DBOption>LocalDB</DBOption>
	  </Database>
	  <HeadNode>
	    <VMName>MyHeadNode</VMName>
	    <ServiceName>MyHPCService</ServiceName>
	    <VMSize>ExtraLarge</VMSize>
	  </HeadNode>
	  <Certificates>
	    <Certificate>
	      <Id>1</Id>
	      <PfxFile>d:\mytestcert1.pfx</PfxFile>
	      <Password>MyPsw!!2</Password>
	    </Certificate>
	  </Certificates>
	  <ComputeNodes>
	    <VMNamePattern>MyHPCCN-%0001%</VMNamePattern>
	<ServiceNamePattern>MyHPCCNService%01%</ServiceNamePattern>
	<MaxNodeCountPerService>5</MaxNodeCountPerService>
	<StorageAccountNamePattern>mycnstorage%01%</StorageAccountNamePattern>
	    <VMSize>Medium</VMSize>
	    <NodeCount>12</NodeCount>
	    <ImageName HPCPackInstalled="true">MyHPCComputeNodeImage</ImageName>
	    <VMExtensions>
	       <VMExtension>
	          <ExtensionName>BGInfo</ExtensionName>
	          <Publisher>Microsoft.Compute</Publisher>
	          <Version>1.*</Version>
	       </VMExtension>
	    </VMExtensions>
	  </ComputeNodes>
	  <AutoGrowShrink>
	    <CertificateId>1</CertificateId>
	  </AutoGrowShrink>
	</IaaSClusterConfig>

### 示例 2

以下配置文件将在现有域林中部署一个 HPC Pack 群集。该群集包含 1 个头节点、1 台具有 500GB 数据磁盘的数据库服务器、2 个运行 Windows Server 2012 R2 操作系统的代理节点，和 5 个运行 Windows Server 2012 R2 操作系统的计算节点。云服务 MyHPCCNService 是在地缘组 *MyIBAffinityGroup* 中创建的，其他所有云服务是在地缘组 *MyAffinityGroup* 中创建的。已在头节点上启用了 HPC 作业计划程序 REST API 和 HPC Web 门户。

	<?xml version="1.0" encoding="utf-8" ?>
	<IaaSClusterConfig>
	  <Subscription>
	    <SubscriptionName>Subscription-1</SubscriptionName>
	    <StorageAccount>mystorageaccount</StorageAccount>
	  </Subscription>
	  <AffinityGroup>MyAffinityGroup</AffinityGroup>
	  <Location>China East</Location>  
	  <VNet>
	    <VNetName>MyVNet</VNetName>
	    <SubnetName>Subnet-1</SubnetName>
	  </VNet>    
	  <Domain>
	    <DCOption>ExistingDC</DCOption>
	    <DomainFQDN>hpc.local</DomainFQDN>
	  </Domain>
	  <Database>
	    <DBOption>NewRemoteDB</DBOption>
	    <DBVersion>SQLServer2014_Enterprise</DBVersion>
	    <DBServer>
	      <VMName>MyDBServer</VMName>
	      <ServiceName>MyHPCService</ServiceName>
	      <VMSize>ExtraLarge</VMSize>
	      <DataDiskSizeInGB>500</DataDiskSizeInGB>
	    </DBServer>
	  </Database>
	  <HeadNode>
	    <VMName>MyHeadNode</VMName>
	    <ServiceName>MyHPCService</ServiceName>
	    <VMSize>ExtraLarge</VMSize>
	    <EnableRESTAPI />
	    <EnableWebPortal />
	  </HeadNode>
	  <ComputeNodes>
	    <VMNamePattern>MyHPCCN-%0000%</VMNamePattern>
	    <ServiceName>MyHPCCNService</ServiceName>
	    <VMSize>A7</VMSize>
	<NodeCount>5</NodeCount>
	<AffinityGroup>MyIBAffinityGroup</AffinityGroup>
	  </ComputeNodes>
	  <BrokerNodes>
	    <VMNamePattern>MyHPCBN-%0000%</VMNamePattern>
	    <ServiceName>MyHPCBNService</ServiceName>
	    <VMSize>Medium</VMSize>
	    <NodeCount>2</NodeCount>
	  </BrokerNodes>
	</IaaSClusterConfig>

### 示例 3

以下配置文件将部署一个 HPC Pack 群集，其中包含一个具有本地数据库的头节点，以及 5 个运行 Windows Server 2008 R2 操作系统的计算节点。所有云服务直接在“中国东部”位置创建。头节点充当域林的域控制器。

	<?xml version="1.0" encoding="utf-8" ?>
	<IaaSClusterConfig>
	  <Subscription>
	    <SubscriptionId>08701940-C02E-452F-B0B1-39D50119F267</SubscriptionId>
	    <StorageAccount>mystorageaccount</StorageAccount>
	  </Subscription>
	  <Location>China East</Location>  
	  <VNet>
	    <VNetName>MyVNet</VNetName>
	    <SubnetName>Subnet-1</SubnetName>
	  </VNet>
	  <Domain>
	    <DCOption>HeadNodeAsDC</DCOption>
	    <DomainFQDN>hpc.local</DomainFQDN>
	  </Domain>
	  <Database>
	    <DBOption>LocalDB</DBOption>
	  </Database>
	  <HeadNode>
	    <VMName>MyHeadNode</VMName>
	    <ServiceName>MyHPCService</ServiceName>
	    <VMSize>ExtraLarge</VMSize>
	  </HeadNode>
	  <ComputeNodes>
	    <VMNamePattern>MyHPCCN-%1000%</VMNamePattern>
	    <ServiceName>MyHPCCNService</ServiceName>
	    <VMSize>Medium</VMSize>
	    <NodeCount>5</NodeCount>
	    <OSVersion>WindowsServer2008R2</OSVersion>
	  </ComputeNodes>
	</IaaSClusterConfig>

### 示例 4

以下配置文件将在现有域林中部署一个 HPC Pack 群集。该群集包含 1 个具有本地数据库的头节点，此外将创建两个 Azure 节点模板，并为 Azure 节点模板 _AzureTemplate1_ 创建 3 个中等大小的 Azure 节点。配置头节点后，将在头节点上运行脚本文件。

	<?xml version="1.0" encoding="utf-8" ?>
	<IaaSClusterConfig>
	  <Subscription>
	    <SubscriptionName>Subscription-1</SubscriptionName>
	    <StorageAccount>mystorageaccount</StorageAccount>
	  </Subscription>
	  <AffinityGroup>MyAffinityGroup</AffinityGroup>
	  <Location>China East</Location>  
	  <VNet>
	    <VNetName>MyVNet</VNetName>
	    <SubnetName>Subnet-1</SubnetName>
	  </VNet>
	  <Domain>
	    <DCOption>ExistingDC</DCOption>
	    <DomainFQDN>hpc.local</DomainFQDN>
	  </Domain>
	  <Database>
	    <DBOption>LocalDB</DBOption>
	  </Database>
	  <HeadNode>
	    <VMName>MyHeadNode</VMName>
	    <ServiceName>MyHPCService</ServiceName>
	<VMSize>ExtraLarge</VMSize>
	    <PostConfigScript>c:\MyHNPostActions.ps1</PostConfigScript>
	  </HeadNode>
	  <Certificates>
	    <Certificate>
	      <Id>1</Id>
	      <PfxFile>d:\mytestcert1.pfx</PfxFile>
	      <Password>MyPsw!!2</Password>
	    </Certificate>
	    <Certificate>
	      <Id>2</Id>
	      <PfxFile>d:\mytestcert2.pfx</PfxFile>
	    </Certificate>    
	  </Certificates>
	  <AzureBurst>
	    <AzureNodeTemplate>
	      <TemplateName>AzureTemplate1</TemplateName>
	      <SubscriptionId>bb9252ba-831f-4c9d-ae14-9a38e6da8ee4</SubscriptionId>
	      <CertificateId>1</CertificateId>
	      <ServiceName>mytestsvc1</ServiceName>
	      <StorageAccount>myteststorage1</StorageAccount>
	      <NodeCount>3</NodeCount>
	      <RoleSize>Medium</RoleSize>
	    </AzureNodeTemplate>
	    <AzureNodeTemplate>
	      <TemplateName>AzureTemplate2</TemplateName>
	      <SubscriptionId>ad4b9f9f-05f2-4c74-a83f-f2eb73000e0b</SubscriptionId>
	      <CertificateId>1</CertificateId>
	      <ServiceName>mytestsvc2</ServiceName>
	      <StorageAccount>myteststorage2</StorageAccount>
	      <Proxy>
	        <UsesStaticProxyCount>false</UsesStaticProxyCount>     
	        <ProxyRatio>100</ProxyRatio>
	        <ProxyRatioBase>400</ProxyRatioBase>
	      </Proxy>
	      <OSVersion>WindowsServer2012</OSVersion>
	    </AzureNodeTemplate>
	  </AzureBurst>
	</IaaSClusterConfig>

## 故障排查


* **“虚拟网络不存在”错误** - 如果你运行 HPC Pack IaaS 部署脚本以在 Azure 中的一个订阅下同时部署多个群集，则一个或多个部署可能会失败并显示错误“虚拟网络 *VNet\_Name* 不存在”。如果发生此错误，请对失败的部署重新运行该脚本。

* **从 Azure 虚拟网络访问 Internet 时出现问题** - 如果你通过使用部署脚本创建 HPC Pack 群集和新的域控制器，或者你手动将头节点 VM 提升为域控制器，则在将 Azure 虚拟网络中的 VM 连接到 Internet 时可能会遇到问题。如果已在域控制器上自动配置转发器 DNS 服务器，但此转发器 DNS 服务器未正确解析，则会出现这种情况。

    若要解决此问题，请登录到域控制器，删除转发器配置设置或配置一个有效的转发器 DNS 服务器。为此，请在服务器管理器中单击“工具”>“DNS”以打开 DNS 管理器，然后双击“转发器”。

* **从 VM 访问 RDMA 网络时出现问题** - 如果你通过使用部署脚本添加计算节点 VM 或代理节点 VM，则在将这些 VM 连接到 RDMA 应用程序网络时可能会遇到问题。发生这种情况的原因之一是，在 VM 添加到群集时未正确安装 HpcVmDrivers 扩展。例如，该扩展可能会停滞在安装状态。

    若要解决此问题，请先检查 VM 中扩展的状态。如果该扩展未正确安装，请尝试从 HPC 群集中删除节点，然后再重新添加节点。例如，你可以通过使用 Add-HpcIaaSNode.ps1 脚本添加计算节点 VM。


## 后续步骤

* 尝试在群集上运行测试工作负荷。例如，请参阅 HPC Pack [入门指南](https://technet.microsoft.com/zh-cn/library/jj884144)。

* 有关使用脚本创建群集和运行 HPC 工作负荷的教程，请参阅[开始使用 Azure 中的 HPC Pack 群集运行 Excel 和 SOA 工作负荷](/documentation/articles/virtual-machines-windows-excel-cluster-hpcpack/)。

* 尝试使用 HPC Pack 的工具来启动、停止、添加和删除所创建群集中的计算节点。请参阅[在 Azure 中管理 HPC Pack 群集的计算节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-manage/)。

* 你也可以使用 Azure 资源管理器模板部署 HPC Pack 集群。

<!---HONumber=Mooncake_0215_2016-->