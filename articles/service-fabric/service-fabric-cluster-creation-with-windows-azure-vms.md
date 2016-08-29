<properties
   pageTitle="使用运行 Windows 的 Azure VM 创建独立群集 | Azure"
   description="了解如何创建和管理运行 Windows Server 的 Azure 虚拟机上的 Azure Service Fabric 群集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="dsk-2015"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="08/05/2016"
   wacn.date="08/29/2016"/>



# 使用运行 Windows Server 的 Azure 虚拟机创建三节点独立 Service Fabric 群集

本文介绍如何使用用于 Windows Server 的独立 Service Fabric 安装程序在基于 Windows 的 Azure 虚拟机 (VM) 上创建群集。这是一种[创建和管理在 Windows Server 上运行的群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)的特殊情况，其中 VM 是[运行 Windows Server 的 Azure VM](/documentation/articles/virtual-machines-windows-hero-tutorial/)，但您不会创建[基于云的 Azure Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-via-portal/)。区别在于根据以下步骤创建的独立 Service Fabric 群集由您完全管理，而基于云的 Azure Service Fabric 群集则由 Service Fabric 资源提供程序管理和升级。


## 创建独立群集的步骤

1. 登录到 Azure 门户，在资源组中创建新的 Windows Server 2012 R2 Datacenter VM。有关更多详细信息，请阅读[在 Azure 门户中创建 Windows VM](/documentation/articles/virtual-machines-windows-hero-tutorial/) 一文。
2. 向同一资源组再添加两个 Windows Server 2012 R2 Datacenter VM。确保每个 VM 在创建后都有相同的管理员用户名和密码。创建以后，你会在同一虚拟网络中看到所有三个 VM。
3. 使用[服务器管理器的“本地服务器”仪表板](https://technet.microsoft.com/zh-cn/library/jj134147.aspx)连接到每个 VM 并关闭 Windows 防火墙。这确保网络流量可以在计算机之间通信。同时连接到每台计算机，通过打开命令提示符并键入 `ipconfig` 来获取 IP 地址。另外，也可以在 Azure 门户中选择资源组的虚拟网络资源，这样即可查看每台计算机的 IP 地址。
4. 连接到其中一台 VM，测试是否可以成功地 ping 其他两台 VM。
5. 连接到其中一台 VM，并将 [Windows Server 独立 Service Fabric 包下载](http://go.microsoft.com/fwlink/?LinkId=730690)到此虚拟机的新文件夹中，然后提取该包。
6. 在记事本中打开 *ClusterConfig.Unsecure.MultiMachine.json* 文件，使用计算机的三个 IP 地址编辑每个节点。更改顶部的群集名称，然后保存文件。下方显示了群集清单的部分示例。


	    {
	        "name": "TestCluster",
	        "clusterManifestVersion": "1.0.0",
	        "apiVersion": "2015-01-01-alpha",
	        "nodes": [
	        {
	            "nodeName": "vm0",
	        	"metadata": "Replace the localhost with valid IP address or FQDN below",
	            "iPAddress": "10.7.0.5",
	            "nodeTypeRef": "NodeType0",
	            "faultDomain": "fd:/dc1/r0",
	            "upgradeDomain": "UD0"
	        },
	        {
	            "nodeName": "vm1",
	        	"metadata": "Replace the localhost with valid IP address or FQDN below",
	            "iPAddress": "10.7.0.4",
	            "nodeTypeRef": "NodeType0",
	            "faultDomain": "fd:/dc2/r0",
	            "upgradeDomain": "UD1"
	        },
	        {
	            "nodeName": "vm2",
	        	"metadata": "Replace the localhost with valid IP address or FQDN below",
	            "iPAddress": "10.7.0.6",
	            "nodeTypeRef": "NodeType0",
	            "faultDomain": "fd:/dc3/r0",
	            "upgradeDomain": "UD2"
	        }
	    ],


7. 打开 [PowerShell ISE 窗口](https://msdn.microsoft.com/zh-cn/powershell/scripting/core-powershell/ise/introducing-the-windows-powershell-ise)。导航到您在其中提取已下载的独立安装程序包并保存群集清单文件的文件夹。运行以下 PowerShell 命令。


    	.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab


8. 您会看到 PowerShell 运行，此时请连接到每台计算机并创建群集。大约一分钟后，可以通过某个计算机 IP 地址（例如通过使用 `http://10.7.0.5:19080/Explorer/index.html`）连接到 Service Fabric Explorer，以便检查群集是否正常运行。由于该群集是使用 Azure VM 的独立群集，因此若要确保其安全，则需[将证书部署到 Azure VM](/documentation/articles/service-fabric-windows-cluster-x509-security/)，或者将某台计算机设置为 [Windows Server Active Directory (AD) 控制器，以便进行 Windows 身份验证](/documentation/articles/service-fabric-windows-cluster-windows-security/)，就像在本地执行相关操作一样。


## 后续步骤
- [在 Windows Server 或 Linux 上创建独立的 Service Fabric 群集](/documentation/articles/service-fabric-deploy-anywhere/)
- [向独立 Service Fabric 群集添加或删除节点](/documentation/articles/service-fabric-cluster-windows-server-add-remove-nodes/)
- [Windows 独立群集的配置设置](/documentation/articles/service-fabric-cluster-manifest/)
- [使用 Windows 安全性保护 Windows 上的独立群集](/documentation/articles/service-fabric-windows-cluster-windows-security/)
- [使用 X509 证书保护 Windows 上的独立群集](/documentation/articles/service-fabric-windows-cluster-x509-security/)

<!---HONumber=Mooncake_0822_2016-->