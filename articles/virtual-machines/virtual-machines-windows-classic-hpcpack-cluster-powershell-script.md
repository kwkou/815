

<properties
   pageTitle="用于部署 Windows HPC 群集的 PowerShell 脚本 | Azure"
   description="运行 PowerShell 脚本，以在 Azure 虚拟机中部署 Windows HPC Pack 群集"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""
   tags="azure-service-management,hpc-pack"/>  

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="big-compute"
   ms.date="07/07/2016"
   wacn.date="12/26/2016"
   ms.author="danlep"/>

# 使用 HPC Pack IaaS 部署脚本创建 Windows 高性能计算 (HPC) 群集

运行 HPC Pack IaaS 部署 PowerShell 脚本，以便在 Azure 虚拟机中部署适用于 Windows 工作负荷的完整 HPC 群集。群集包含运行 Windows Server 和 Microsoft HPC Pack 的已加入 Active Directory 的头节点，以及你指定的其他 Windows 计算资源。你还可以使用 Azure Resource Manager 模板来部署 HPC Pack 群集。相关示例，请参阅[创建 HPC 群集](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster/)和[使用自定义计算节点映像创建 HPC 群集](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster-custom-image/)。

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]

[AZURE.INCLUDE [virtual-machines-common-classic-hpcpack-cluster-powershell-script](../../includes/virtual-machines-common-classic-hpcpack-cluster-powershell-script.md)]

## <a name="Example-configuration-files"></a> 示例配置文件

在下面的示例中，将订阅 ID 或名称以及帐户和服务名称替换为你自己的值。

### 示例 1

以下配置文件将部署 1 个 HPC Pack 群集，其中包含 1 个具有本地数据库的头节点和 5 个运行 Windows Server 2012 R2 操作系统的计算节点。所有云服务直接在“中国北部”位置创建。头节点充当域林的域控制器。

	<?xml version="1.0" encoding="utf-8" ?>
	<IaaSClusterConfig>
	  <Subscription>
	    <SubscriptionId>08701940-C02E-452F-B0B1-39D50119F267</SubscriptionId>
	    <StorageAccount>mystorageaccount</StorageAccount>
	  </Subscription>
	  <Location>China North</Location>  
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
	    <OSVersion>WindowsServer2012R2</OSVersion>
	  </ComputeNodes>
	</IaaSClusterConfig>

### 示例 2

以下配置文件将在现有域林中部署一个 HPC Pack 群集。该群集包含 1 个具有本地数据库的头节点和 12 个应用了 BGInfo VM 扩展的计算节点。域林中的所有 VM 均已禁用 Windows 更新的自动安装。所有云服务直接在“中国东部”位置创建。计算节点在 3 个云服务和 3 个存储帐户中创建：_MyHPCCNService01_ 和 _mycnstorage01_ 中的 _MyHPCCN-0001_ 到 _MyHPCCN-0005_；_MyHPCCNService02_ 和 _mycnstorage02_ 中的 _MyHPCCN-0006_ 到 _MyHPCCN0010_；_MyHPCCNService03_ 和 _mycnstorage03_ 中的 _MyHPCCN-0011_ 到 _MyHPCCN-0012_。计算节点基于从计算节点捕获的现有专用映像创建。已启用自动扩展和收缩服务，并采用默认的扩展和收缩间隔。

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

### 示例 3

以下配置文件将在现有域林中部署一个 HPC Pack 群集。该群集包含 1 个头节点、 1 个具有 500 GB 数据磁盘的数据库服务器、2 个运行 Windows Server 2012 R2 操作系统的中转节点，以及 5 个运行 Windows Server 2012 R2 操作系统的计算节点。云服务 MyHPCCNService 在地缘组 *MyIBAffinityGroup* 中创建，其他云服务在地缘组 *MyAffinityGroup* 中创建。已在头节点上启用了 HPC 作业计划程序 REST API 和 HPC Web 门户。

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



### 示例 4

以下配置文件将在现有域林中部署一个 HPC Pack 群集。该群集包含 2 个具有本地数据库的头节点，此外还将创建 2 个 Azure 节点模板并为 Azure 节点模板 _AzureTemplate1_ 创建 3 个中等大小的 Azure 节点。配置头节点后，将在其上运行脚本文件。


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

## 故障排除


* **“虚拟网络不存在”错误** - 如果运行脚本在 Azure 中的一个订阅下同时部署多个群集，则一个或多个部署可能会失败并显示错误“虚拟网络 *VNet\_Name* 不存在”。如果发生此错误，请对失败的部署再次运行该脚本。

* **从 Azure 虚拟网络访问 Internet 时出现问题** - 如果使用部署脚本创建具有新域控制器的群集，或将头节点 VM 手动提升为域控制器，则将 VM 连接到 Internet 时可能会遇到问题。如果已在域控制器上自动配置转发器 DNS 服务器，但此转发器 DNS 服务器未正确解析，则会出现此问题。

    若要解决此问题，请登录到域控制器，删除转发器配置设置或配置一个有效的转发器 DNS 服务器。若要配置此设置，请在服务器管理器中单击“工具”>“DNS”以打开 DNS 管理器，然后双击“转发器”。

## 后续步骤

* 尝试在群集上运行测试工作负荷。例如，请参阅 HPC Pack [入门指南](https://technet.microsoft.com/zh-cn/library/jj884144)。

* 有关编写群集部署脚本和运行 HPC 工作负荷的教程，请参阅[开始在 Azure 中使用 HPC Pack 群集运行 Excel 和 SOA 工作负荷](/documentation/articles/virtual-machines-windows-excel-cluster-hpcpack/)。

* 尝试使用 HPC Pack 的工具来启动、停止、添加和删除所创建群集中的计算节点。请参阅[在 Azure 中管理 HPC Pack 群集的计算节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-manage/)。

* 若要完成设置以将本地计算机中的作业提交到群集，请参阅[将本地计算机中的 HPC 作业提交到 Azure 中的 HPC Pack 群集](/documentation/articles/virtual-machines-windows-hpcpack-cluster-submit-jobs/)。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->