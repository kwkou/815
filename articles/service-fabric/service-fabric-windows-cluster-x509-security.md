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
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="07/08/2016"
   wacn.date="08/08/2016"
   ms.author="dkshir"/>

# 使用 X.509 证书在 Windows 上保护独立群集

本文介绍如何使用 X.509 证书保护独立 Windows 群集的各个节点之间的通信，以及如何对连接到此群集的客户端进行身份验证。这可确保只有经过授权的用户才能访问该群集和部署的应用程序，以及执行管理任务。创建群集时，应在该群集上启用证书安全性。

有关节点到节点安全性、客户端到节点安全性和基于角色的访问控制等群集安全性的详细信息，请参阅[群集安全方案](/documentation/articles/service-fabric-cluster-security/)。

## 需要哪些证书？

首先是[下载独立群集包](/documentation/articles/service-fabric-cluster-creation-for-windows-server/#downloadpackage)，将其下载到群集中的某个节点。在下载的包中，你会找到 **ClusterConfig.X509.MultiMachine.json** 文件。打开文件，然后查看 **properties** 部分下面的 **security** 部分：

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

该部分描述保护独立 Windows 群集所需的证书。若要启用基于证书的安全性，可将 **ClusterCredentialType** 和 **ServerCredentialType** 的值设置为 X509。

>[AZURE.NOTE] [指纹](https://en.wikipedia.org/wiki/Public_key_fingerprint)是证书的主标识。请阅读 [How to retrieve thumbprint of a certificate](https://msdn.microsoft.com/zh-cn/library/ms734695.aspx)（如何检索证书的指纹），以找出所创建的证书的指纹。

下表列出了在设置群集时所需的证书：

|**CertificateInformation 设置**|**说明**|
|-----------------------|--------------------------|
|ClusterCertificate|需要使用此证书来保护群集节点之间的通信。可以使用两个不同的证书（一个主证书和一个辅助证书）进行升级。在 **Thumbprint** 节中设置主证书的指纹，在 **ThumbprintSecondary** 变量中设置辅助证书的指纹。|
|ServerCertificate|当客户端尝试连接到此群集时，系统会向客户端提供此证书。为方便起见，你可以选择对 ClusterCertificate 和 ServerCertificate 使用相同的证书。可以使用两个不同的服务器证书（一个主证书和一个辅助证书）进行升级。在 **Thumbprint** 节中设置主证书的指纹，在 **ThumbprintSecondary** 变量中设置辅助证书的指纹。 |
|ClientCertificateThumbprints|这是需要在经过身份验证的客户端上安装的一组证书。可以在想要允许其访问群集的计算机上安装许多不同的客户端证书。在 **CertificateThumbprint** 变量中设置每个证书的指纹。如果将 **IsAdmin** 设置为 true，则安装了此证书的客户端可以针对群集执行管理员管理活动。如果 **IsAdmin** 为 false，则使用此证书的客户端只能执行用户访问权限（通常为只读）所允许的操作。有关角色的详细信息，请阅读[基于角色的访问控制 (RBAC)](/documentation/articles/service-fabric-cluster-security/#role-based-access-control-rbac) |
|ClientCertificateCommonNames|在 **CertificateCommonName** 中设置第一个客户端证书的公用名。**CertificateIssuerThumbprint** 是此证书的颁发者的指纹。请阅读[使用证书](https://msdn.microsoft.com/zh-cn/library/ms731899.aspx)，以了解有关公用名和颁发者的详细信息。|

下面是群集配置示例，其中提供了群集证书、服务器证书和客户端证书。


	 {
	    "name": "SampleCluster",
	    "clusterManifestVersion": "1.0.0",
	    "apiVersion": "2015-01-01-alpha",
	    "nodes": [{
	        "nodeName": "vm0",
			"metadata": "Replace the localhost below with valid IP address or FQDN",
	        "iPAddress": "10.7.0.5",
	        "nodeTypeRef": "NodeType0",
	        "faultDomain": "fd:/dc1/r0",
	        "upgradeDomain": "UD0"
	    }, {
	      "nodeName": "vm1",
	      		"metadata": "Replace the localhost with valid IP address or FQDN",
	        "iPAddress": "10.7.0.4",
	        "nodeTypeRef": "NodeType0",
	        "faultDomain": "fd:/dc1/r1",
	        "upgradeDomain": "UD1"
	    }, {
	        "nodeName": "vm2",
	      "iPAddress": "10.7.0.6",
	      		"metadata": "Replace the localhost with valid IP address or FQDN",
	        "nodeTypeRef": "NodeType0",
	        "faultDomain": "fd:/dc1/r2",
	        "upgradeDomain": "UD2"
	    }],
	    "diagnosticsFileShare": {
	        "etlReadIntervalInMinutes": "5",
	        "uploadIntervalInMinutes": "10",
	        "dataDeletionAgeInDays": "7",
	        "etwStoreConnectionString": "file:c:\\ProgramData\\SF\\FileshareETW",
	        "crashDumpConnectionString": "file:c:\\ProgramData\\SF\\FileshareCrashDump",
	        "perfCtrConnectionString": "file:c:\\ProgramData\\SF\\FilesharePerfCtr"
	    },
	    "properties": {
	        "security": {
	            "metadata": "The Credential type X509 indicates this is cluster is secured using X509 Certificates. The thumbprint format is - d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
	            "ClusterCredentialType": "X509",
	            "ServerCredentialType": "X509",
	            "CertificateInformation": {
	                "ClusterCertificate": {
	                    "Thumbprint": "a8 13 67 58 f4 ab 89 62 af 2b f3 f2 79 21 be 1d f6 7f 43 26",
	                    "X509StoreName": "My"
	                },
	                "ServerCertificate": {
	                    "Thumbprint": "a8 13 67 58 f4 ab 89 62 af 2b f3 f2 79 21 be 1d f6 7f 43 26",
	                    "X509StoreName": "My"
	                },
	                "ClientCertificateThumbprints": [{
	                    "CertificateThumbprint": "c4 c18 8e aa a8 58 77 98 65 f8 61 4a 0d da 4c 13 c5 a1 37 6e",
	                    "IsAdmin": false
	                }, {
	                    "CertificateThumbprint": "71 de 04 46 7c 9e d0 54 4d 02 10 98 bc d4 4c 71 e1 83 41 4e",
	                    "IsAdmin": true
	                }]
	            }
	        },
	        "reliabilityLevel": "Bronze",
	        "nodeTypes": [{
	            "name": "NodeType0",
	            "clientConnectionEndpointPort": "19000",
	            "clusterConnectionEndpoint": "19001",
	            "httpGatewayEndpointPort": "19080",
	            "applicationPorts": {
	                "startPort": "20001",
	                "endPort": "20031"
	            },
	            "ephemeralPorts": {
	                "startPort": "20032",
	                "endPort": "20062"
	            },
	            "isPrimary": true
	        }
	         ],
	        "fabricSettings": [{
	            "name": "Setup",
	            "parameters": [{
	                "name": "FabricDataRoot",
	                "value": "C:\\ProgramData\\SF"
	            }, {
	                "name": "FabricLogRoot",
	                "value": "C:\\ProgramData\\SF\\Log"
	            }]
	        }]
	    }
	}


## 获取 X.509 证书
若要保护群集内部的通信，首先需要获取群集节点的 X.509 证书。此外，如果只想允许经过授权的计算机/用户连接到此群集，需要获得并安装客户端计算机的证书。

对于运行生产工作负荷的群集，应使用[证书颁发机构 (CA)](https://en.wikipedia.org/wiki/Certificate_authority) 签名的 X.509 证书来保护群集。有关如何获取这些证书的详细信息，请转到[如何：获取证书](http://msdn.microsoft.com/zh-cn/library/aa702761.aspx)。

对于用于测试的群集，可以选择使用自签名证书。

## 可选：创建自签名证书
若要创建能够得到适当保护的自签名证书，一种方式是使用 CertSetup.ps1 脚本，该脚本位于 C:\\Program Files\\Microsoft SDKs\\Service Fabric\\ClusterSetup\\Secure 目录的 Service Fabric SDK 文件夹中。编辑此文件，用其创建具有合适名称的证书。

现在，请将证书导出到使用受保护密码的 PFX 文件。首先需获取证书的指纹。运行 certmgr.exe 应用程序。导航到 **Local Computer\\Personal** 文件夹，找到刚创建的证书。双击证书将其打开，选择“详细信息”选项卡，然后向下滚动到“指纹”字段。将指纹值复制到下面的 PowerShell 命令中，删除空格。将 $pswd 值更改为一个合适的安全密码，以确保其受保护，然后运行 PowerShell：


	$pswd = ConvertTo-SecureString -String "1234" -Force ¨CAsPlainText
	Get-ChildItem -Path cert:\localMachine\my<Thumbprint> | Export-PfxCertificate -FilePath C:\mypfx.pfx -Password $pswd


若要查看安装在计算机上的证书的详细信息，可运行以下 PowerShell 命令：


	$cert = Get-Item Cert:\LocalMachine\My<Thumbprint>
	Write-Host $cert.ToString($true)


或者，如果你有 Azure 订阅，则可以遵循[获取 X.509 证书](/documentation/articles/service-fabric-secure-azure-cluster-with-certs/#acquirecerts)部分中的步骤创建证书。

## 安装证书
有了证书以后，即可将其安装在群集节点上。你的节点上需安装最新版本的 Windows PowerShell 3.x。需要在每个节点上，针对群集证书和服务器证书以及任何辅助证书重复这些步骤。

1. 将 .pfx 文件复制到节点。
2. 以管理员身份打开 PowerShell 窗口并输入以下命令。将 $pswd 替换为用来创建此证书的密码。将 $PfxFilePath 替换为复制到此节点的 .pfx 文件的完整路径。


	    $pswd = "1234"
	    $PfcFilePath ="C:\mypfx.pfx"
	    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\LocalMachine\My -FilePath $PfxFilePath -Password (ConvertTo-SecureString -String $pswd -AsPlainText -Force)


3. 接下来，你需要通过运行以下脚本设置对此证书的访问控制，以便在网络服务帐户下运行的 Service Fabric 进程可以使用它。为服务帐户提供证书指纹以及“NETWORK SERVICE”。你可以使用 certmgr.exe 工具来查看证书上的“管理私钥”，以确保证书上的 ACL 是正确的。


	    param
	    (
	        [Parameter(Position=1, Mandatory=$true)]
	        [ValidateNotNullOrEmpty()]
	        [string]$pfxThumbPrint,

	        [Parameter(Position=2, Mandatory=$true)]
	        [ValidateNotNullOrEmpty()]
	        [string]$serviceAccount
	        )

	        $cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object -FilterScript { $PSItem.ThumbPrint -eq $pfxThumbPrint; };

	        # Specify the user, the permissions and the permission type
	        $permission = "$($serviceAccount)","FullControl","Allow"
	        $accessRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $permission;

	        # Location of the machine related keys
	        $keyPath = $env:ProgramData + "\Microsoft\Crypto\RSA\MachineKeys";
	        $keyName = $cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName;
	        $keyFullPath = $keyPath + $keyName;

	        # Get the current acl of the private key
	        $acl = (Get-Item $keyFullPath).GetAccessControl('Access')

	        # Add the new ace to the acl of the private key
	        $acl.SetAccessRule($accessRule);

	        # Write back the new acl
	        Set-Acl -Path $keyFullPath -AclObject $acl -ErrorAction Stop

	        #Observe the access rights currently assigned to this certificate.
	        get-acl $keyFullPath| fl


4. 针对每个服务器证书重复上述步骤。你还可以使用这些步骤在需要让其访问群集的计算机上安装客户端证书。

## 创建安全群集
配置 **ClusterConfig.X509.MultiMachine.json** 文件的 **security** 部分后，可以转到[创建群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/#createcluster)部分来配置节点并创建独立群集。创建群集时，请记得使用 **ClusterConfig.X509.MultiMachine.json** 文件。例如，你的命令可能如下所示：


	.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -AcceptEULA $true


使安全且独立的 Windows 群集成功运行并设置可连接到该群集的、经过身份验证的客户端之后，即可按照[使用 PowerShell 连接到安全群集](/documentation/articles/service-fabric-connect-to-secure-cluster/#connectsecurecluster)部分的步骤连接到该群集。例如：


	Connect-ServiceFabricCluster -ConnectionEndpoint 10.7.0.4:19000 -KeepAliveIntervalInSec 10 -X509Credential -ServerCertThumbprint 057b9544a6f2733e0c8d3a60013a58948213f551 -FindType FindByThumbprint -FindValue 057b9544a6f2733e0c8d3a60013a58948213f551 -StoreLocation CurrentUser -StoreName My


如果你已登录到群集中的某台计算机，由于已在本地安装证书，因此你可以直接运行 Powershell 命令连接到该群集，然后就会显示节点的列表：


	Connect-ServiceFabricCluster
	Get-ServiceFabricNode

若要删除群集，请调用以下命令：


	.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json   -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab

<!---HONumber=Mooncake_0801_2016-->