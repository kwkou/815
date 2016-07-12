<properties
   pageTitle="连接到安全专用群集 | Azure"
   description="本文介绍如何保护独立群集或专用群集内部的通信，以及客户端与群集之间的通信。"
   services="service-fabric"
   documentationCenter=".net"
   authors="dsk-2015"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/10/2016"
   wacn.date="07/04/2016"/>

# 使用证书保护独立的 Windows 群集

本文介绍如何使用 X.509 证书保护独立 Windows 群集的各个节点之间的通信，以及如何对连接到此群集的客户端进行身份验证。这可确保只有经过授权的用户才能访问该群集和部署的应用程序，以及执行管理任务。创建群集时，应在该群集上启用证书安全性。有关节点到节点安全性、客户端到节点安全性和基于角色的访问控制的详细信息，请参阅 [Cluster security scenarios](/documentation/articles/service-fabric-cluster-security/)（群集安全方案）。

## 需要哪些证书？

首先，[将独立群集包下载到](/documentation/articles/service-fabric-cluster-creation-for-windows-server/#downloadpackage)群集中的节点之一。在下载的包，你会看到 **ClusterConfig.X509.json** 文件。在“编辑”模式下打开该文件，然后在 **properties** 选项卡下面查看 **security** 所在的节：

    "security": {
        "metadata": "The Credential type X509 indicates this is cluster is secured using X509 Certificates. The thumbprint format is - d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
        "ClusterCredentialType": "X509",
        "ServerCredentialType": "X509",
        "CertificateInformation": {
			"ClusterCertificate": {
                "Thumbprint": "[Thumbprint]",
                "ThumbprintSecondary": "[Thumbprint]",
                "X509StoreName": "My"
            },
            "ServerCertificate": {
                "Thumbprint": "[Thumbprint]",
                "ThumbprintSecondary": "[Thumbprint]",
                "X509StoreName": "My"
            },
            "ClientCertificateThumbprints": [{
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": false
            }, {
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": true
            }],
            "ClientCertificateCommonNames": [{
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint" : "[Thumbprint]",
                "IsAdmin": true
            }]
        }
    },

此节描述保护独立 Windows 群集所需的所有证书。通过将 **ClusterCredentialType** 和 **ServerCredentialType** 的值设置为 *X509* 来启用基于证书的安全性。

>[AZURE.NOTE] [指纹](https://en.wikipedia.org/wiki/Public_key_fingerprint)是证书的主标识。请阅读 [How to retrieve thumbprint of a certificate](https://msdn.microsoft.com/zh-cn/library/ms734695.aspx)（如何检索证书的指纹），以找出所创建的证书的指纹。

下表列出了在设置群集时所需的实际证书：

|**CertificateInformation 设置**|**说明**|
|-----------------------|--------------------------|
|ClusterCertificate|需要使用此证书来保护群集节点之间的通信。可以使用两个不同的证书（一个主证书和一个辅助证书）进行故障转移。在 **Thumbprint** 节中设置主证书的指纹，在 **ThumbprintSecondary** 变量中设置辅助证书的指纹。|
|ServerCertificate|当客户端尝试连接到此群集时，系统会向客户端提供此证书。为方便起见，你可以选择对 *ClusterCertificate* 和 *ServerCertificate* 使用相同的证书。可以使用两个不同的服务器证书（一个主证书和一个辅助证书）进行故障转移。在 **Thumbprint** 节中设置主证书的指纹，在 **ThumbprintSecondary** 变量中设置辅助证书的指纹。 |
|ClientCertificateThumbprints|这是需要在经过身份验证的客户端上安装的一组证书。可以在想要允许访问群集及其上运行的应用程序的计算机上安装许多不同的客户端证书。在 **CertificateThumbprint** 变量中设置每个证书的指纹。如果将 **IsAdmin** 设置为 *true*，则安装了此证书的客户端可以针对群集执行各种管理活动。如果 **IsAdmin** 为 *false*，则该客户端只能访问该群集上运行的应用程序。|
|ClientCertificateCommonNames|在 **CertificateCommonName** 中设置第一个客户端证书的公用名。**CertificateIssuerThumbprint** 是此证书的颁发者的指纹。请阅读 [Working with certificates](https://msdn.microsoft.com/zh-cn/library/ms731899.aspx)（使用证书），以了解有关公用名和颁发者的详细信息。|


## 安装证书

若要保护群集之间的通信，首先需要获取群集节点的 X.509 证书。此外，如果只想允许经过授权的计算机/用户连接到此群集，需要获得并安装这些潜在客户端计算机的证书。请阅读 [How to obtain a certificate](https://msdn.microsoft.com/zh-cn/library/aa702761.aspx)（如何获得证书），以了解有关获取 X.509 证书的步骤。需要创建一个 **.pfx** 证书，以便能够存储你的私钥。或者，如果你有 Azure 订阅，则可以遵循[获取 X.509 证书](/documentation/articles/service-fabric-secure-azure-cluster-with-certs/#acquirecerts)部分中的步骤创建证书。获取此证书后，可以使用以下步骤在群集节点上安装该证书。请注意，这些步骤假设你的节点上已安装最新版本的 Windows PowerShell 3.x。需要在每个节点上，针对群集证书和服务器证书以及任何辅助证书重复这些步骤。

- 将 .pfx 文件复制到节点。

- 以管理员身份打开 PowerShell 窗口并输入以下命令：
	
		Import-PfxCertificate -Exportable -CertStoreLocation Cert:\LocalMachine\My `
		-FilePath $PfxFilePath -Password (ConvertTo-SecureString -String $password -AsPlainText -Force)

将 *$password* 替换为用来创建此证书的密码。将 $PfxFilePath 替换为复制到此节点的 .pfx 文件的完整路径。

- 通过运行以下脚本设置对证书的访问控制，以便 Service Fabric 进程可以使用它：

		$cert = get-item Cert:\LocalMachine\My<CertificateThumbprint>
		$cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName

		#Note down the string output by this command, this is the container name for your private key. 

		$privateKeyFilePath = 'C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys<PrivateKeyContainerName>'
	
		#Observe the access rights currently assigned to this certificate.
		get-acl $privateKeyFilePath | fl

		#Set the ACL so that the Service Fabric process can access it.
		$acl = get-acl $privateKeyFilePath
		$acl.SetAccessRule((New-Object System.Security.Accesscontrol.FileSystemAccessRule("NT AUTHORITY\NetworkService","FullControl","Allow")))
		Set-Acl $privateKeyFilePath $acl -ErrorAction Stop 
	

使用这些步骤还可以在你想要允许其访问群集的计算机上安装客户端证书。


## 后续步骤

配置 ClusterConfig.X509.json 文件的 **security** 节后，可以转到[创建群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/#createcluster)部分来配置节点并创建独立群集。创建群集时，请记得使用 **ClusterConfig.X509.json** 文件。例如，你的命令可能如下所示：

	cd $ServiceFabricDeployAnywhereFolder
	.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -AcceptEULA $true -Verbose


使安全的独立 Windows 群集成功运行并且设置了可连接到该群集的、经过身份验证的客户端之后，请遵循[使用 PowerShell 连接到安全群集](/documentation/articles/service-fabric-connect-to-secure-cluster/#connectsecurecluster)部分中的步骤连接到该群集。









<!---HONumber=Mooncake_0627_2016-->