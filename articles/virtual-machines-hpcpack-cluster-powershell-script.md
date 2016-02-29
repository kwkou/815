<properties
   pageTitle="用于部署 HPC Pack 群集的 PowerShell 脚本 | Microsoft Azure"
   description="运行 Windows PowerShell 脚本，以在 Azure 基础结构服务中部署完整的 HPC Pack 群集"
   services="virtual-machines"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""
   tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines"
	ms.date="01/08/2016"
	wacn.date="02/26/2016"/>

# 使用 HPC Pack IaaS 部署脚本在 Azure VM 中创建高性能计算 (HPC) 群集

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]



在客户端计算机上运行 HPC Pack IaaS 部署 PowerShell 脚本，以在 Azure 基础结构服务 (IaaS) 中部署完整的 HPC Pack 群集。此脚本提供多种部署选项，并且可添加运行受支持 Linux 分发包或 Windows Server 操作系统的群集计算节点。

根据你的环境和选择，该脚本可以创建所有群集基础结构，包括 Azure 虚拟网络、存储帐户、云服务、域控制器、远程或本地 SQL 数据库、头节点、代理节点、计算节点和 Azure 云服务（“迸发”或 PaaS）节点。此外，该脚本还可以使用预先存在的 Azure 基础结构，然后创建 HPC 群集头节点、代理节点、计算节点和 Azure 迸发节点。


有关规划 HPC Pack 群集的背景信息，请参阅 HPC Pack TechNet 库中的[产品评估和规划](https://technet.microsoft.com/zh-cn/library/jj899596.aspx)及[入门](https://technet.microsoft.com/zh-cn/library/jj899590.aspx)内容。

## 先决条件

* **Azure 订阅** - 你可以使用 Azure 全球或 Azure 中国服务中的订阅。你的订阅限制只会影响你可以部署的群集节点数量和类型。有关信息，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits)。


* **安装并配置了 Azure PowerShell 0.8.7 或更高版本的 Windows 客户端计算机** - 请参阅[安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。该脚本将在 Azure 服务管理中运行。


* **HPC Pack IaaS 部署脚本** - 从 [Microsoft 下载中心](https://www.microsoft.com/download/details.aspx?id=44949)下载并解压缩最新版本的脚本。通过运行 `New-HPCIaaSCluster.ps1 -Version` 检查脚本的版本。本文基于版本 4.4.0 的脚本。

* **脚本配置文件** - 需要创建脚本用来配置 HPC 群集的 XML 文件。有关信息和示例，请参阅本文后面的部分。


## 语法

	New-HPCIaaSCluster.ps1 [-ConfigFile] <String> [-AdminUserName]<String> [[-AdminPassword] <String>] [[-HPCImageName] <String>] [[-LogFile] <String>] [-Force] [-NoCleanOnFailure] [-PSSessionSkipCACheck] [<CommonParameters>]

>[AZURE.NOTE]你必须以管理员身份运行脚本。

### Parameters

* **ConfigFile** - 指定描述 HPC 群集的配置文件的路径。有关详细信息，请参阅本主题中的[配置文件](#Configuration-file)，或参阅包含该脚本的文件夹中的 Manual.rtf 文件。

* **AdminUserName** - 指定用户名。如果域林是由脚本创建的，则此用户名将成为所有 VM 的本地管理员用户名以及域管理员名称。如果域林已存在，则此参数会将域用户指定为安装 HPC Pack 的本地管理员用户名。

* **AdminPassword** - 指定管理员的密码。如果未在命令行中指定，脚本将提示你输入密码。

* **HPCImageName**（可选）- 指定用于部署 HPC 群集的 HPC Pack VM 映像名称。它必须是 Microsoft 通过 Azure 库提供的 HPC Pack 映像。如果未指定（在大多数情况下建议不要指定），脚本将选择最新发布的 HPC Pack 映像。

    >[AZURE.NOTE] 指定无效的 HPC Pack 映像会导致部署失败。

* **LogFile**（可选）- 指定部署日志文件路径。如果未指定，脚本将在运行脚本的计算机的 temp 目录中创建一个日志文件。

* **Force**（可选）- 抑制所有确认提示。

* **NoCleanOnFailure**（可选）- 指定将不删除未成功部署的 Azure VM。在重新运行脚本以继续部署之前，你必须手动删除这些 VM，否则部署可能失败。

* **PSSessionSkipCACheck**（可选）- 对于每个包含此脚本所部署的 VM 的云服务，Azure 将自动生成自签名证书，云服务中的所有 VM 使用此证书作为默认的 Windows 远程管理 (WinRM) 证书。若要在这些 Azure VM 中部署 HPC 功能，脚本默认情况下将在客户端计算机的“本地计算机\\受信任的根证书颁发机构”存储中临时安装这些证书，以抑制执行脚本期间发生的“不受信任的 CA”安全错误，并在完成脚本后删除这些证书。如果指定此参数，将不会在客户端计算机中安装证书，并且会抑制安全警告。

    >[AZURE.IMPORTANT] 对于生产部署，不建议指定此参数。

### 示例

以下示例将使用配置文件 *MyConfigFile.xml* 创建一个新的 HPC Pack 群集，并指定用于安装该群集的管理凭据。

	New-HPCIaaSCluster.ps1 -ConfigFile MyConfigFile.xml -AdminUserName <username> -AdminPassword <password>

### 其他注意事项

* 该脚本使用 Azure 库中的 HPC Pack VM 映像创建群集头节点。最新映像基于装有 HPC Pack 2012 R2 Update 3 的 Windows Server 2012 R2 Datacenter。

* （可选）该脚本可以启用通过 HPC Pack Web 门户或 HPC Pack REST API 提交作业。

* 如果你想要安装其他软件或配置其他设置，该脚本可以选择在头节点上运行自定义的配置前和配置后脚本。


## 配置文件

部署脚本的配置文件是一个 XML 文件。架构文件 HPCIaaSClusterConfig.xsd 位于 HPC Pack IaaS 部署脚本文件夹中。**IaaSClusterConfig** 是配置文件的根元素，其中包含部署脚本文件夹中 Manual.rtf 文件详细描述的子元素。有关不同方案的示例文件，请参阅本文中的[示例配置文件](#Example-configuration-files)。

## 示例配置文件

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
	    <VMSize>A8</VMSize>
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

以下配置文件将创建新的域林并部署 HPC Pack 群集，其中包含 1 个具有本地数据库的头节点和 20 个 Linux 计算节点。所有云服务直接在“中国东部”位置创建。Linux 计算节点在 4 个云服务和 4 个存储帐户中创建（即，_MyLnxCNService01_ 和 _mylnxstorage01_ 中的 _MyLnxCN-0001_ 到 _MyLnxCN-0005_；_MyLnxCNService02_ 和 _mylnxstorage02_ 中的 _MyLnxCN-0006_ 到 _MyLnxCN-0010_；_MyLnxCNService03_ 和 _mylnxstorage03_ 中的 _MyLnxCN-0011_ 到 _MyLnxCN-0015_；_MyLnxCNService04_ 和 _mylnxstorage04_ 中的 _MyLnxCN-0016_ 到 _MyLnxCN-0020_）。计算节点是基于 OpenLogic CentOS 7.0 版 Linux 映像创建的。

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

### 示例 4

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

### 示例 5

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

## 已知问题


* **“虚拟网络不存在”错误** - 如果你运行 HPC Pack IaaS 部署脚本以在 Azure 中的一个订阅下同时部署多个群集，则一个或多个部署可能会失败并显示错误“虚拟网络 *VNet\_Name* 不存在”。如果发生此错误，请对失败的部署重新运行该脚本。

* **从 Azure 虚拟网络访问 Internet 时出现问题** - 如果你通过使用部署脚本创建 HPC Pack 群集和新的域控制器，或者你手动将 VM 提升为域控制器，则在将 Azure 虚拟网络中的 VM 连接到 Internet 时可能会遇到问题。如果已在域控制器上自动配置转发器 DNS 服务器，但此转发器 DNS 服务器未正确解析，则会出现这种情况。

    若要解决此问题，请登录到域控制器，删除转发器配置设置或配置一个有效的转发器 DNS 服务器。为此，请在服务器管理器中单击“工具”>“DNS”以打开 DNS 管理器，然后双击“转发器”。

* **从大小为 A8 或 A9 的 VM 访问 RDMA 网络时出现问题** - 如果你通过使用部署脚本添加大小为 A8 或 A9 的计算节点 VM 或代理节点 VM，则在将这些 VM 连接到 RDMA 应用程序网络时可能会遇到问题。发生这种情况的原因之一是，在将大小为 A8 或 A9 的 VM 添加到群集时未正确安装 HpcVmDrivers 扩展。例如，该扩展可能会停滞在安装状态。

    若要解决此问题，请先检查 VM 中扩展的状态。如果该扩展未正确安装，请尝试从 HPC 群集中删除节点，然后再重新添加节点。例如，你可以通过使用 Add-HpcIaaSNode.ps1 脚本添加计算节点 VM。


## 后续步骤

* 尝试在群集上运行测试工作负荷。例如，请参阅 HPC Pack [入门指南](https://technet.microsoft.com/zh-cn/library/jj884144)。

* 有关使用脚本创建群集和运行 HPC 工作负荷的教程，请参阅[开始使用 Azure 中的 HPC Pack 群集运行 Excel 和 SOA 工作负荷](/documentation/articles/virtual-machines-excel-cluster-hpcpac)、[在 Azure 中的 Linux 计算节点上使用 Microsoft HPC Pack 运行 NAMD](/documentation/articles/virtual-machines-linux-cluster-hpcpack-namd) 或[在 Azure 中的 Linux 计算节点上使用 Microsoft HPC Pack 运行 OpenFOAM](/documentation/articles/virtual-machines-linux-cluster-hpcpack-openfoam)。

* 尝试使用 HPC Pack 的工具来启动、停止、添加和删除所创建群集中的计算节点。请参阅[在 Azure 中管理 HPC Pack 群集的计算节点](/documentation/articles/virtual-machines-hpcpack-cluster-node-manage)

<!---HONumber=Mooncake_0215_2016-->