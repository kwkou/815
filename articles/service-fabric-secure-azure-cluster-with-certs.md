<properties
   pageTitle="使用证书保护 Service Fabric 群集 | Azure"
   description="如何使用 X.509 证书保护 Service Fabric 群集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/27/2016"
   wacn.date="07/07/2016"/>

# 使用证书保护 Azure 上的 Service Fabric 群集

Azure Service Fabric 群集是你拥有的资源。为了防止未经授权访问资源，必须保护资源，尤其是其中有正在运行的生产工作负荷时。若要使用 X.509 证书设置安全的 Service Fabric 群集，至少需要一个服务器 X.509 证书，你要将它上载到 Azure 密钥保管库并在创建群集的过程中用于。

有关 Service Fabric 如何使用 X.509 证书的详细信息，请参阅 [Cluster security scenarios](/documentation/articles/service-fabric-cluster-security/)（群集安全方案）。

共有三个不同的步骤：

1. 获取 X.509 证书。
2. 将证书上载到 Azure 密钥保管库。
3. 将证书的位置和详细信息提供给 Service Fabric 群集创建过程。

<a id="acquirecerts"></a>
## 步骤 1：获取 X.509 证书

对于运行生产工作负荷的群集，必须使用[证书颁发机构 (CA)](https://en.wikipedia.org/wiki/Certificate_authority) 签名的 X.509 证书来保护群集。有关如何获取这些证书的详细信息，请转到[如何：获取证书](http://msdn.microsoft.com/zh-cn/library/aa702761.aspx)。

对于仅用于测试的群集，可以选择使用自签名证书。下面的步骤 2.5 说明如何执行该操作。

## 步骤 2：将 X.509 证书上载到密钥保管库

这是一个复杂的过程，因此我们要将一个 PowerShell 模块上载到 Git 存储库，它将为完成此过程。

### 步骤 2.1
确保 Azure PowerShell 1.0+ 已安装在计算机上。如果尚未安装，请按照 [How to install and configure Azure PowerShell.](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell）中所述的步骤进行安装。

### 步骤 2.2
将 *ServiceFabricRPHelpers* 文件夹从此 [Git 存储库](https://github.com/ChackDan/Service-Fabric/tree/master/Scripts/ServiceFabricRPHelpers)复制到你的计算机。

### 步骤 2.3
打开 PowerShell 窗口并转到模块下载到的目录。然后使用以下命令导入该模块。


	Import-Module .\ServiceFabricRPHelpers.psm1


### 步骤 2.4
如果要使用以前获取的证书，请遵循此步骤中的过程。否则，请跳到步骤 2.5，该步骤说明了如何创建自签名证书，并将自签名证书部署到密钥保管库。

可以使用现有的资源组和密钥保管库来存储证书，或者，如果资源组和/或密钥保管库不存在，你可以新建一个。必须先使用此脚本将现有密钥保管库配置为支持部署。


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

	Set-AzureRmKeyVaultAccessPolicy -VaultName <Name of the Vault> -ResourceGroupName <string> -EnabledForDeployment


若要将证书上载到资源组和密钥保管库，请运行以下脚本。如果资源组和密钥保管库尚不存在，该脚本将予以创建。


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	Invoke-AddCertToKeyVault -SubscriptionId <your subscription id> -ResourceGroupName <string> -Location <region> -VaultName <Name of the Vault> -CertificateName <Name of the Certificate> -Password <Certificate password> -UseExistingCertificate -ExistingPfxFilePath <Full path to the .pfx file>

以下是已填充脚本的示例。


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	Invoke-AddCertToKeyVault -SubscriptionId 35389201-c0b3-405e-8a23-9f1450994307 -ResourceGroupName chackdankeyvault4doc -Location westus -VaultName chackdankeyvault4doc  -CertificateName chackdantestcertificate2 -Password abcd123 -UseExistingCertificate -ExistingPfxFilePath C:\MyCertificates\ChackdanTestCertificate.pfx


该脚本成功完成时，你将看到类似于下面的输出，执行步骤 3（配置安全群集）时将用到这些数据。


	Certificate Thumbprint: 2118C3BCE6541A54A0236E14ED2CCDD77EA4567A

	SourceVault /Resource ID of the key vault :  /subscriptions/35389201-c0b3-405e-8a23-9f1450994307/resourceGroups/chackdankeyvault4doc/providers/Microsoft.KeyVault/vaults/chackdankeyvault4doc

	Certificate URL /URL to the certificate location in the key vault : https://chackdankeyvalut4doc.vault.chinacloudapi.cn:443/secrets/chackdantestcertificate3/ebc8df6300834326a95d05d90e0701ea


现在你已拥有设置安全群集所需的信息。请转到步骤 3。

### 步骤 2.5
如果你没有证书，但想要创建新的自签名证书并将它上载到密钥保管库，请执行以下步骤。自签名证书只应该用于测试群集，而不应该用于产品群集。

可以使用现有的资源组和密钥保管库来存储证书，或者，如果资源组和/或密钥保管库不存在，你可以新建一个。必须先使用此脚本将现有密钥保管库配置为支持部署。

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	Set-AzureRmKeyVaultAccessPolicy -VaultName <Name of the Vault> -ResourceGroupName <string> -EnabledForDeployment


以下脚本将创建新的资源组和/或密钥保管库（如果尚不存在）、创建自签名证书并将其上载到密钥保管库，然后将新证书输出到 *OutputPath*。


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	Invoke-AddCertToKeyVault -SubscriptionId <you subscription id> -ResourceGroupName <string> -Location <region> -VaultName <Name of the Vault> -CertificateName <Name of the Certificate> -Password <Certificate password> -CreateSelfSignedCertificate -DnsName <string- see note below.> -OutputPath <Full path to the .pfx file>

*DnsName* 字符串指定一个或多个 DNS 名称，以便在 CloneCert 参数中未指定要复制的证书时放入证书的使用者可选名称扩展中。第一个 DNS 名称还将保存为使用者名称。如果未指定任何签名证书，则第一个 DNS 名称还将保存为颁发者名称。*Invoke-AddCertToKeyVault* cmdlet 使用 [New-SelfSignedCertificate](https://technet.microsoft.com/zh-cn/library/hh848633.aspx) cmdlet 来创建自签名证书。

以下是已填充脚本的示例。


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	Invoke-AddCertToKeyVault -SubscriptionId 35389201-c0b3-405e-8a23-9f1450994307 -ResourceGroupName chackdankeyvault4doc -Location westus -VaultName chackdankeyvault4doc  -CertificateName chackdantestcertificate3 -Password abcd123 -CreateSelfSignedCertificate -DnsName www.chackdan.westus.azure.com -OutputPath C:\MyCertificates


脚本成功完成时，你将看到类似于下面的输出。执行步骤 3 时需要用到这些数据。


	Certificate Thumbprint: 64881409F4D86498C88EEC3697310C15F8F1540F

	SourceVault /Resource ID of the key vault : /subscriptions/35389201-c0b3-405e-8a23-9f1450994307/resourceGroups/chackdankeyvault4doc/providers/Microsoft.KeyVault/vaults/chackdankeyvault4doc

	Certificate URL /URL to the certificate location in the key vault: https://chackdankeyvalut4doc.vault.chinacloudapi.cn:443/secrets/chackdantestcertificate3/fvc8df6300834326a95d05d90e0720ea


## 步骤 3：设置安全群集
<!--
将证书上载到 Azure 密钥保管库之后，可以创建使用这些证书保护的群集。此步骤对应于群集创建过程的[步骤 3：配置安全性](/documentation/articles/service-fabric-cluster-creation-via-portal/#step-3--configure-security)，说明如何设置安全配置。

>[AZURE.NOTE]
所需的证书在“安全配置”下的“节点类型”级别指定。必须为群集中的每个节点类型指定此配置。尽管本文档演练如何使用门户执行此操作，但你可以使用 Azure Resource Manager 模板来实现相同的目的。

![Azure 门户预览中“安全配置”的屏幕截图][SecurityConfigurations_01]-->

你可以使用 [Azure Resource Manager 模板](/documentation/articles/service-fabric-cluster-creation-via-arm/)来为群集中的每个节点类型进行安全配置。

### 必需参数

- **安全模式**：选择“X509 证书”来设置使用 X.509 证书保护的群集。
- **群集保护级别**：请参阅此[保护级别文档](https://msdn.microsoft.com/zh-cn/library/aa347692.aspx)，了解每个值的含义。尽管我们允许在此处使用三个值（EncryptAndSign、Sign 和 None），但除非你知道的意图，否则最好保留默认值 EncryptAndSign。
- **源保管库**：这是密钥保管库的资源 ID。其格式应该是：

    ```
    /subscriptions/<Sub ID>/resourceGroups/<Resource group name>/providers/Microsoft.KeyVault/vaults/<vault name>
    ```

- **证书 URL**：这是密钥保管库中证书所上载到的位置 URL。其格式应该是：

	```
    https://<name of the vault>.vault.chinacloudapi.cn:443/secrets/<exact location>
	```
	```
    https://chackdan-kmstest-eastus.vault.chinacloudapi.cn:443/secrets/MyCert/6b5cc15a753644e6835cb3g3486b3812
	```

- **证书指纹**：这是证书的指纹，可以在前面指定的 URL 中找到。

### 可选参数

 可以选择性地指定客户端计算机的、用于在群集上执行操作的其他证书。默认情况下，在必需参数中指定的指纹将添加到能够执行客户端操作的已授权指纹列表中。

**管理客户端**：此信息用于验证连接到群集管理终结点的客户端是否可以提供正确的凭据，以便在群集上执行管理和只读操作。你可以指定多个想要授权其执行管理操作的证书。

- **授权方**：向 Service Fabric 指出是要使用使用者名称还是指纹来查找此证书。使用使用者名称授权不是良好的安全实践，但是可以增加操作的弹性。
- **使用者名称**：仅当已指定按使用者名称授权时才需要。
- **颁发者指纹**：当客户端向服务器提供自己的凭据时，能够让服务器执行额外的检查。

**只读客户端**：此信息用于验证连接到群集管理终结点的客户端是否可以提供正确的凭据，以便在群集上执行只读操作。你可以指定多个想要授权其执行只读操作的证书。

- **授权方**：向 Service Fabric 指出是要使用使用者名称还是指纹来查找此证书。使用使用者名称授权不是良好的安全实践，但是可以增加操作的弹性。
- **使用者名称**：仅当已指定按使用者名称授权时才需要。
- **颁发者指纹**：当客户端向服务器提供自己的凭据时，能够让服务器执行额外的检查。
<!--
## 后续步骤
在群集上配置证书安全性之后，请继续执行[步骤 4：完成群集创建](/documentation/articles/service-fabric-cluster-creation-via-portal/#step-4--complete-the-cluster-creation)中的群集创建过程。

为群集创建证书安全性之后，可以[更新证书](/documentation/articles/service-fabric-cluster-security-update-certs-azure/)。
-->

<!--Image references-->
[SecurityConfigurations_01]: ./media/service-fabric-cluster-azure-secure-with-certs/SecurityConfigurations_01.png
[SecurityConfigurations_02]: ./media/service-fabric-cluster-azure-secure-with-certs/SecurityConfigurations_02.png

<!---HONumber=Mooncake_0627_2016-->