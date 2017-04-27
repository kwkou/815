<properties
    pageTitle="根据模板创建 Azure Service Fabric 群集 | Azure"
    description="本文介绍如何通过使用 Azure Resource Manager、Azure Key Vault 和 Azure Active Directory (Azure AD) 进行客户端身份验证，在 Azure 中设置一个安全的 Service Fabric 群集。"
    services="service-fabric"
    documentationcenter=".net"
    author="chackdan"
    manager="timlt"
    editor="chackdan"
    translationtype="Human Translation" />
<tags
    ms.assetid="15d0ab67-fc66-4108-8038-3584eeebabaa"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/22/2017"
    wacn.date="04/24/2017"
    ms.author="chackdan"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="d8233d9507689ad3bede8eabe91389fc3f3ef151"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-service-fabric-cluster-by-using-azure-resource-manager"></a>使用 Azure Resource Manager 创建 Service Fabric 群集
> [AZURE.SELECTOR]
- [Azure Resource Manager](/documentation/articles/service-fabric-cluster-creation-via-arm/)
- [Azure 门户](/documentation/articles/service-fabric-cluster-creation-via-portal/)

本指南逐步介绍如何使用 Azure Resource Manager 在 Azure 中设置安全的 Azure Service Fabric 群集。 本文篇幅较长。 然而，除非已完全熟悉内容，否则请确保仔细阅读每个步骤。

该指南包含下列步骤：

* 设置 Azure 密钥保管库 以上传证书，保护群集和应用程序的安全
* 使用 Azure Resource Manager 在 Azure 中创建受保护的群集
* 通过使用 Azure Active Directory (Azure AD) 进行群集管理，对用户进行身份验证

安全的群集可防止未经授权访问管理操作。 包括部署、升级和删除应用程序、服务及其所包含的数据。 不安全的群集是任何人都可以随时连接并执行管理操作的群集。 虽然可创建不安全的群集，但强烈建议一开始就创建安全的群集。 因为不安全的群集无法在事后受到保护 - 若要保护群集，必须创建新群集。

无论是 Linux 或 Windows 群集，创建安全群集的概念是相同的。 有关创建安全的 Linux 群集的详细信息和协助脚本，请参阅[在 Linux 上创建安全群集](#secure-linux-clusters)。

## <a name="sign-in-to-your-azure-account"></a>登录到 Azure 帐户
本指南使用 [Azure PowerShell][azure-powershell]。 开始新的 PowerShell 会话时，请登录到 Azure 帐户并选择订阅，然后执行 Azure 命令。

请登录到 Azure 帐户：


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud


选择订阅：


	Get-AzureRmSubscription
	Set-AzureRmContext -SubscriptionId <guid>

## <a name="set-up-a-key-vault"></a>设置密钥保管库
本部分讨论如何在 Azure 中为 Service Fabric 群集以及 Service Fabric 应用程序创建密钥保管库。 有关 Azure 密钥保管库 的完整指南，请参阅 [Key Vault 入门指南][key-vault-get-started]。

Service Fabric 使用 X.509 证书保护群集，提供应用程序安全功能。Azure 密钥保管库用于管理 Azure 中 Service Fabric 群集的证书。在 Azure 中部署群集时，负责创建 Service Fabric 群集的 Azure 资源提供程序将从密钥保管库提取证书，然后将其安装在群集 VM 上。

下图演示密钥保管库、Service Fabric 群集与 Azure 资源提供程序（在创建群集时使用密钥保管库中存储的证书）之间的关系：

![证书安装图示][cluster-security-cert-installation]

### <a name="create-a-resource-group"></a>创建资源组
第一个步骤是专门为密钥保管库创建资源组。 建议将密钥保管库置于其资源组中。 这样可在不丢失密钥和机密的情况下删除计算和存储资源组，包括具有 Service Fabric 群集的资源组。 包含密钥保管库的资源组必须与正在使用它的群集位于_同一区域_。

如果计划在多个区域部署群集，则建议以适当的方式对资源组和密钥保管库命名，以便通过名称了解其所属的区域。  


		New-AzureRmResourceGroup -Name mycluster-keyvault -Location 'China East'

输出应如下所示：

        	WARNING: The output object type of this cmdlet is going to be modified in a future release.

		ResourceGroupName : mycluster-keyvault
		Location          : chinaeast
		ProvisioningState : Succeeded
		Tags              :
		ResourceId        : /subscriptions/<guid>/resourceGroups/mycluster-keyvault


<a id="new-key-vault"></a>

### <a name="create-a-key-vault-in-the-new-resource-group"></a>在新资源组中创建密钥保管库
_必须针对部署启用_密钥保管库，使计算资源提供程序能够从中获取证书并将其安装在虚拟机实例上：

		New-AzureRmKeyVault -VaultName 'myvault' -ResourceGroupName 'mycluster-keyvault' -Location 'China East' -EnabledForDeployment
	
输出应如下所示：
	
		Vault Name                       : myvault
		Resource Group Name              : mycluster-keyvault
		Location                         : China East
		Resource ID                      : /subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault
		Vault URI                        : https://myvault.vault.azure.cn
		Tenant ID                        : <guid>
		SKU                              : Standard
		Enabled For Deployment?          : False
		Enabled For Template Deployment? : False
		Enabled For Disk Encryption?     : False
		Access Policies                  :
		                                   Tenant ID                :    <guid>
		                                   Object ID                :    <guid>
		                                   Application ID           :
		                                   Display Name             :    
		                                   Permissions to Keys      :    get, create, delete, list, update, import, backup, restore
		                                   Permissions to Secrets   :    all
	
	
		Tags                             :

<a id="existing-key-vault"></a>

## <a name="use-an-existing-key-vault"></a>使用现有的密钥保管库

若要使用现有密钥保管库，则_必须针对部署启用_该密钥保管库，使计算资源提供程序能够从中获取证书并将其安装在群集节点上：

    Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -EnabledForDeployment

<a id="add-certificate-to-key-vault"></a>

## <a name="add-certificates-to-your-key-vault"></a>将证书添加到密钥保管库

证书在 Service Fabric 中用于提供身份验证和加密，为群集及其应用程序提供全方位的保护。 有关如何在 Service Fabric 中使用证书的详细信息，请参阅 [Service Fabric 群集安全方案][service-fabric-cluster-security]。

### <a name="cluster-and-server-certificate-required"></a>群集和服务器证书（必需）
需要使用此证书来保护群集以及防止未经授权访问群集。 此证书通过两种方式保护群集：

* 群集身份验证：在群集联合的情况下对节点间的通信进行身份验证。 只有可以使用此证书自我证明身份的节点才能加入群集。
* 服务器身份验证：在管理客户端上对群集管理终结点进行身份验证，使管理客户端知道它正在与真正的群集通信。 此证书还通过 HTTPS 为 HTTPS 管理 API 和 Service Fabric Explorer 提供 SSL。

为满足这些用途，该证书必须符合以下要求：

* 证书必须包含私钥。
* 必须为密钥交换创建证书，并且该证书可导出到个人信息交换 (.pfx) 文件。
* 证书的使用者名称必须与用于访问 Service Fabric 群集的域匹配。 只有满足此匹配，才能为群集的 HTTPS 管理终结点和 Service Fabric Explorer 提供 SSL。 无法从证书颁发机构 (CA) 处获取针对 .cloudapp.chinacloudapi.cn 域的 SSL 证书。 必须获取群集的自定义域名。 从 CA 请求证书时，该证书的使用者名称必须与用于群集的自定义域名匹配。

### <a name="application-certificates-optional"></a>应用程序证书（可选）
可以出于应用程序安全目的，在群集上安装任意数量的附加证书。 在创建群集之前，请考虑需要在节点上安装证书的应用程序安全方案，例如：

* 加密和解密应用程序配置值。
* 在复制期间跨节点加密数据。

### <a name="formatting-certificates-for-azure-resource-provider-use"></a>格式化证书以供 Azure 资源提供程序使用
可直接通过密钥保管库添加和使用私有密钥文件 (.pfx)。 但是，Azure 计算资源提供程序需要以特殊 JavaScript 对象表示法 (JSON) 格式存储的密钥。 格式包含作为 base 64 编码的字符串和私钥密码的 .pfx 文件。 若要满足这些要求，必须将密钥放入 JSON 字符串，然后在密钥保管库中将其存储为“机密”。

为了简化此过程，[GitHub 上提供了][service-fabric-rp-helpers] PowerShell 模块。 若要使用该模块，请执行以下操作：

1. 将存储库的整个内容下载到本地目录。
2. 转到本地目录。
2. 将 ServiceFabricRPHelpers 模块导入 PowerShell 窗口：

     	Import-Module "C:\..\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1"


此 PowerShell 模块中的 `Invoke-AddCertToKeyVault` 命令自动将证书私钥的格式设置为 JSON 字符串，并将它上传到密钥保管库。 使用该命令可将群集证书与任何其他应用程序证书添加到密钥保管库。 针对要在群集中安装的其他任何证书重复此步骤。

#### <a name="uploading-an-existing-certificate"></a>上传现有证书

	 Invoke-AddCertToKeyVault -SubscriptionId <guid> -ResourceGroupName mycluster-keyvault -Location "China East" -VaultName myvault -CertificateName mycert -Password "<password>" -UseExistingCertificate -ExistingPfxFilePath "C:\path\to\mycertkey.pfx"
	

如果收到如此处所示的错误，通常意味着发生了资源 URL 冲突。 若要解决此冲突，请更改密钥保管库名称。

	    Set-AzureKeyVaultSecret : The remote name could not be resolved: 'chinaeastkv.vault.azure.cn'
	    At C:\Users\chackdan\Documents\GitHub\Service-Fabric\Scripts\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1:440 char:11
	    + $secret = Set-AzureKeyVaultSecret -VaultName $VaultName -Name $Certif ...
	    +           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	        + CategoryInfo          : CloseError: (:) [Set-AzureKeyVaultSecret], WebException
	        + FullyQualifiedErrorId : Microsoft.Azure.Commands.KeyVault.SetAzureKeyVaultSecret


冲突解决后，输出应如下所示：

	        Switching context to SubscriptionId <guid>
	        Ensuring ResourceGroup chinaeast-mykeyvault in China East
	        WARNING: The output object type of this cmdlet is going to be modified in a future release.
	        Using existing valut mychinaeastvault in China East
	        Reading pfx file from C:\path\to\key.pfx
	        Writing secret to mychinaeastvault in vault mychinaeastvault


	    Name  : CertificateThumbprint
	    Value : E21DBC64B183B5BF355C34C46E03409FEEAEF58D

	    Name  : SourceVault
	    Value : /subscriptions/<guid>/resourceGroups/chinaeast-mykeyvault/providers/Microsoft.KeyVault/vaults/mychinaeastvault

	    Name  : CertificateURL
	    Value : https://mychinaeastvault.vault.azure.net:443/secrets/mycert/4d087088df974e869f1c0978cb100e47


>[AZURE.NOTE]
>需要三个上述字符串：CertificateThumbprint、SourceVault 和 CertificateURL，以便设置安全的 Service Fabric 群集并获取用于保护应用程序安全的应用程序证书。 如果不保存字符串，可能稍后难以通过查询密钥保管库来检索它们。

<a id="add-self-signed-certificate-to-key-vault"></a>

#### <a name="creating-a-self-signed-certificate-and-uploading-it-to-the-key-vault"></a>创建自签名证书并将其上传到密钥保管库

如果已将证书上传到密钥保管库，可跳过此步骤。 此步骤用于生成新的自签名证书并将其上传到密钥保管库。 在以下脚本中更改参数然后运行该参数后，系统会提示用户输入证书密码。  

	$ResouceGroup = "chackochinaeastkv"
	$VName = "chackokv2"
	$SubID = "6c653126-e4ba-42cd-a1dd-f7bf96ae7a47"
	$locationRegion = "chinaeast" 
	$newCertName = "chackotestcertificate1"
	$dnsName = "www.mycluster.chinaeast.mydomain.com" #The certificate's subject name must match the domain used to access the Service Fabric cluster.
	$localCertPath = "C:\MyCertificates" # location where you want the .PFX to be stored

	 Invoke-AddCertToKeyVault -SubscriptionId $SubID -ResourceGroupName $ResouceGroup -Location $locationRegion -VaultName $VName -CertificateName $newCertName -CreateSelfSignedCertificate -DnsName $dnsName -OutputPath $localCertPath


如果收到如此处所示的错误，通常意味着发生了资源 URL 冲突。 若要解决此冲突，请更改密钥保管库名称、RG 名称等。

	    Set-AzureKeyVaultSecret : The remote name could not be resolved: 'chinaeastkv.vault.azure.cn'
	    At C:\Users\chackdan\Documents\GitHub\Service-Fabric\Scripts\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1:440 char:11
	    + $secret = Set-AzureKeyVaultSecret -VaultName $VaultName -Name $Certif ...
	    +           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	        + CategoryInfo          : CloseError: (:) [Set-AzureKeyVaultSecret], WebException
	        + FullyQualifiedErrorId : Microsoft.Azure.Commands.KeyVault.SetAzureKeyVaultSecret


冲突解决后，输出应如下所示：

	PS C:\Users\chackdan\Documents\GitHub\Service-Fabric\Scripts\ServiceFabricRPHelpers> Invoke-AddCertToKeyVault -SubscriptionId $SubID -ResourceGroupName $ResouceGroup -Location $locationRegion -VaultName $VName -CertificateName $newCertName -Password $certPassword -CreateSelfSignedCertificate -DnsName $dnsName -OutputPath $localCertPath
	Switching context to SubscriptionId 6c343126-e4ba-52cd-a1dd-f8bf96ae7a47
	Ensuring ResourceGroup chackochinaeastkv in chinaeast
	WARNING: The output object type of this cmdlet will be modified in a future release.
	Creating new vault chinaeastkv1 in chinaeast
	Creating new self signed certificate at C:\MyCertificates\chackonewcertificate1.pfx
	Reading pfx file from C:\MyCertificates\chackonewcertificate1.pfx
	Writing secret to chackonewcertificate1 in vault chinaeastkv1


	Name  : CertificateThumbprint
	Value : 96BB3CC234F9D43C25D4B547sd8DE7B569F413EE

	Name  : SourceVault
	Value : /subscriptions/6c653126-e4ba-52cd-a1dd-f8bf96ae7a47/resourceGroups/chackochinaeastkv/providers/Microsoft.KeyVault/vaults/chinaeastkv1

	Name  : CertificateURL
	Value : https://chinaeastkv1.vault.azure.cn:443/secrets/chackonewcertificate1/ee247291e45d405b8c8bbf81782d12bd


>[AZURE.NOTE]
>需要三个上述字符串：CertificateThumbprint、SourceVault 和 CertificateURL，以便设置安全的 Service Fabric 群集并获取用于保护应用程序安全的应用程序证书。 如果不保存字符串，可能稍后难以通过查询密钥保管库来检索它们。

 此时，应具备以下元素：

* 密钥保管库资源组。
* 密钥保管库及其 URL（在上面的 PowerShell 输出中称为 SourceVault）。
* 密钥保管库中的群集服务器身份验证证书及其 URL。
* 密钥保管库中的应用程序证书及其 URL。


<a id="add-AAD-for-client"></a>

## <a name="set-up-azure-active-directory-for-client-authentication"></a>为客户端身份验证设置 Azure Active Directory

通过 Azure AD，组织（称为租户）可管理用户对应用程序的访问。 应用程序分为采用基于 Web 的登录 UI 的应用程序和采用本地客户端体验的应用程序。 本文假设已创建了一个租户。 如果未创建，请先阅读[如何获取 Azure Active Directory 租户][active-directory-howto-tenant]。

Service Fabric 群集提供其管理功能的各种入口点，包括基于 Web 的 [Service Fabric Explorer][service-fabric-visualizing-your-cluster] 和 [Visual Studio][service-fabric-manage-application-in-visual-studio]。 因此，需要创建两个 Azure AD 应用程序来控制对群集的访问：一个 Web 应用程序和一个本机应用程序。

为了简化涉及到配置 Azure AD 与 Service Fabric 群集的一些步骤，我们创建了一组 Windows PowerShell 脚本。

> [AZURE.NOTE]
> 在创建群集之前，请完成以下步骤。 因为脚本需要群集名称和终结点，这些值应是规划的值，而不是已创建的值。

1. [将脚本下载][sf-aad-ps-script-download]到计算机。
2. 右键单击 zip 文件，选择“属性”，“解除阻止”复选框，然后单击“应用”。
3. 解压缩 zip 文件。
4. 运行 `SetupApplications.ps1` 并提供 TenantId、ClusterName 和 WebApplicationReplyUrl 作为参数。 例如：


    	.\SetupApplications.ps1 -TenantId '690ec069-8200-4068-9d01-5aaf188e557a' -ClusterName 'mycluster' -WebApplicationReplyUrl 'https://mycluster.chinaeast.cloudapp.chinacloudapi.cn:19080/Explorer/index.html'

    执行 PowerShell 命令 `Get-AzureSubscription`，可找到租户 ID。 执行此命令，为每个订阅显示 TenantId。

    将 ClusterName 用作脚本创建的 Azure AD 应用程序的前缀。 它不需要完全匹配实际的群集名称。 旨在更加轻松地将 Azure AD 项目映射到其配合使用的 Service Fabric 群集。

    WebApplicationReplyUrl 是 Azure AD 在完成登录过程之后返回给用户的默认终结点。 将此终结点设置为群集的 Service Fabric Explorer 的终结点，默认值为：

    https://&lt;cluster_domain&gt;:19080/Explorer

    系统将提示登录到具有 Azure AD 租户管理权限的帐户。 完成此操作后，脚本会创建 Web 和本机应用程序来代表 Service Fabric 群集。 在 [Azure 经典门户][azure-classic-portal]中查看租户的应用程序时，应会看到两个新条目：

   * *ClusterName*\_Cluster
   * *ClusterName*\_Client

   在下一部分创建群集时该脚本显示 Azure Resource Manager 模板所需的 JSON，因此最好不要关闭 PowerShell 窗口。


	"azureActiveDirectory": {
	  "tenantId":"<guid>",
	  "clusterApplication":"<guid>",
	  "clientApplication":"<guid>"
	},

## <a name="create-a-service-fabric-cluster-resource-manager-template"></a>创建 Service Fabric 群集 Resource Manager 模板
在本部分中，在 Service Fabric 群集 Resource Manager 模板中使用上述 PowerShell 命令的输出。

可以从 [GitHub 上的 Azure 快速入门模板库][azure-quickstart-templates]获取示例 Resource Manager 模板。 这些模板可用作群集模板的起点。

### <a name="create-the-resource-manager-template"></a>创建 Resource Manager 模板
本指南使用 [5 节点安全群集][service-fabric-secure-cluster-5-node-1-nodetype]示例模板和模板参数。 将 `azuredeploy.json` 和 `azuredeploy.parameters.json` 下载到计算机，在偏好的文本编辑器中打开这两个文件。

### <a name="add-certificates"></a>添加证书
通过引用包含证书密钥的密钥保管库将证书添加到群集 Resource Manager 模板。 我们建议将密钥保管库值放在 Resource Manager 模板参数文件中。 以便可以重复使用 Resource Manager 模板文件，且无需特定于某个部署的值。

#### <a name="add-all-certificates-to-the-virtual-machine-scale-set-osprofile"></a>将所有证书都添加到虚拟机规模集 osProfile
必须在规模集资源 (Microsoft.Compute/virtualMachineScaleSets) 的 osProfile 节中配置在群集中安装的每个证书。 该操作会指示资源提供程序在 VM 上安装证书。 此安装包括群集证书和打算用于应用程序的任何应用程序安全证书：

	{
	  "apiVersion": "2016-03-30",
	  "type": "Microsoft.Compute/virtualMachineScaleSets",
	  ...
	  "properties": {
	    ...
	    "osProfile": {
	      ...
	      "secrets": [
	        {
	          "sourceVault": {
	            "id": "[parameters('sourceVaultValue')]"
	          },
	          "vaultCertificates": [
	            {
	              "certificateStore": "[parameters('clusterCertificateStorevalue')]",
	              "certificateUrl": "[parameters('clusterCertificateUrlValue')]"
	            },
	            {
	              "certificateStore": "[parameters('applicationCertificateStorevalue')",
	              "certificateUrl": "[parameters('applicationCertificateUrlValue')]"
	            },
	            ...
	          ]
	        }
	      ]
	    }
	  }
	}

#### <a name="configure-the-service-fabric-cluster-certificate"></a>配置 Service Fabric 群集证书
必须在 Service Fabric 群集资源 (Microsoft.ServiceFabric/clusters) 和 Service Fabric 扩展为虚拟机规模集资源中的虚拟机规模集配置群集身份验证证书。 通过此安排，Service Fabric 资源提供程序便可以将该证书配置为用于群集身份验证及管理终结点的服务器身份验证。

##### <a name="virtual-machine-scale-set-resource"></a>虚拟机规模集资源：

	{
	  "apiVersion": "2016-03-30",
	  "type": "Microsoft.Compute/virtualMachineScaleSets",
	  ...
	  "properties": {
	    ...
	    "virtualMachineProfile": {
	      "extensionProfile": {
	        "extensions": [
	          {
	            "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
	            "properties": {
	              ...
	              "settings": {
	                ...
	                "certificate": {
	                  "thumbprint": "[parameters('clusterCertificateThumbprint')]",
	                  "x509StoreName": "[parameters('clusterCertificateStoreValue')]"
	                },
	                ...
	              }
	            }
	          }
	        ]
	      }
	    }
	  }
	}

##### <a name="service-fabric-resource"></a>Service Fabric 资源：

	{
	  "apiVersion": "2016-03-01",
	  "type": "Microsoft.ServiceFabric/clusters",
	  "name": "[parameters('clusterName')]",
	  "location": "[parameters('clusterLocation')]",
	  "dependsOn": [
	    "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
	  ],
	  "properties": {
	    "certificate": {
	      "thumbprint": "[parameters('clusterCertificateThumbprint')]",
	      "x509StoreName": "[parameters('clusterCertificateStoreValue')]"
	    },
	    ...
	  }
	}

### <a name="insert-azure-ad-configuration"></a>插入 Azure AD 配置
先前创建的 Azure AD 配置可直接插入 Resource Manager 模板。 但是，我们建议先将值提取到参数文件，以便重复使用 Resource Manager 模板文件，且无需特定于某个部署的值。

	{
	  "apiVersion": "2016-03-01",
	  "type": "Microsoft.ServiceFabric/clusters",
	  "name": "[parameters('clusterName')]",
	  ...
	  "properties": {
	    "certificate": {
	      "thumbprint": "[parameters('clusterCertificateThumbprint')]",
	      "x509StoreName": "[parameters('clusterCertificateStorevalue')]"
	    },
	    ...
	    "azureActiveDirectory": {
	      "tenantId": "[parameters('aadTenantId')]",
	      "clusterApplication": "[parameters('aadClusterApplicationId')]",
	      "clientApplication": "[parameters('aadClientApplicationId')]"
	    },
	    ...
	  }
	}

### <a name="configure-arm"></a>配置 Resource Manager 模板参数
最后，使用密钥保管库和 Azure AD PowerShell 命令的输出值填充参数文件：

	{
	    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
	    "contentVersion": "1.0.0.0",
	    "parameters": { 
	        ...
	        "clusterCertificateStoreValue": {
	            "value": "My"
	        },
	        "clusterCertificateThumbprint": {
	            "value": "<thumbprint>"
	        },
	        "clusterCertificateUrlValue": {
	            "value": "https://myvault.vault.azure.cn:443/secrets/myclustercert/4d087088df974e869f1c0978cb100e47"
	        },
	        "applicationCertificateStorevalue": {
	            "value": "My"
	        },
	        "applicationCertificateUrlValue": {
	            "value": "https://myvault.vault.azure.cn:443/secrets/myapplicationcert/2e035058ae274f869c4d0348ca100f08"
	        },
	        "sourceVaultvalue": {
	            "value": "/subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault"
	        },
	        "aadTenantId": {
	            "value": "<guid>"
	        },
	        "aadClusterApplicationId": {
	            "value": "<guid>"
	        },
	        "aadClientApplicationId": {
	            "value": "<guid>"
	        },
	        ...
	    }
	}

此时，应具备以下元素：

* 密钥保管库资源组
  * 密钥保管库
  * 群集服务器身份验证证书
  * 数据加密证书
* Azure Active Directory 租户
  * 用于基于 Web 的管理和 Service Fabric Explorer 的 Azure AD 应用程序
  * 用于本机客户端管理的 Azure AD 应用程序
  * 用户及其分配的角色
* Service Fabric 群集 Resource Manager 模板
  * 通过密钥保管库配置的证书
  * 配置的 Azure Active Directory

下图演示密钥保管库和 Azure AD 配置在 Resource Manager 模板中的作用。

![Resource Manager 依赖关系图][cluster-security-arm-dependency-map]

## <a name="create-the-cluster"></a>创建群集
现在可以使用 [Azure 资源模板部署][resource-group-template-deploy]来创建群集。

#### <a name="test-it"></a>测试
运行以下 PowerShell 命令，使用 parameters 文件测试 Resource Manager 模板：

    Test-AzureRmResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json

#### <a name="deploy-it"></a>部署
如果 Resource Manager 模板通过测试，请运行以下 PowerShell 命令，使用 parameters 文件来部署 Resource Manager 模板：

    New-AzureRmResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json

<a name="assign-roles"></a>

## <a name="assign-users-to-roles"></a>将用户分配到角色
创建用于表示群集的应用程序后，请将用户分配到 Service Fabric 支持的角色：只读和管理员。 可通过使用 [Azure 经典管理门户][azure-classic-portal]来分配角色。

1. 在 Azure 经典管理门户中，转到你的租户，然后选择“应用程序”。
2. 选择名称类似于 `myTestCluster_Cluster` 的 Web 应用程序。
3. 单击“用户”选项卡。
4. 选择要分配的用户，然后单击屏幕底部的“分配”按钮。

    ![将用户分配到角色按钮][assign-users-to-roles-button]
5. 选择要分配给用户的角色。

    ![“分配用户”对话框][assign-users-to-roles-dialog]

> [AZURE.NOTE]
> 有关 Service Fabric 中角色的详细信息，请参阅[适用于 Service Fabric 客户端的基于角色的访问控制](/documentation/articles/service-fabric-cluster-security-roles/)。
>
>

## <a name="next-steps"></a>后续步骤
此时，已创建一个使用 Azure Active Directory 进行管理身份验证的安全群集。 接下来，请[连接到该群集](/documentation/articles/service-fabric-connect-to-secure-cluster/)，了解如何[管理应用程序机密](/documentation/articles/service-fabric-application-secret-management/)。

## <a name="troubleshoot-setting-up-azure-active-directory-for-client-authentication"></a>排查用于客户端身份验证的 Azure Active Directory 的设置问题
如果设置用于客户端身份验证的 Azure AD 时遇到问题，请参阅本部分中可能的解决方法。

### <a name="service-fabric-explorer-prompts-you-to-select-a-certificate"></a>Service Fabric Explorer 提示选择证书
#### <a name="problem"></a>问题
成功登录到 Service Fabric Explorer 中的 Azure AD 后，浏览器返回到主页，但会出现提示用户选择证书的消息。

![SFX - 选择证书对话框][sfx-select-certificate-dialog]

#### <a name="reason"></a>原因
未在 Azure AD 群集应用程序中为用户分配角色。 因此，Service Fabric 群集的 Azure AD 身份验证失败。 Service Fabric Explorer 将故障回复到证书身份验证。

#### <a name="solution"></a>解决方案
遵循有关设置 Azure AD 的说明操作，并为用户分配角色。 此外，我们建议打开“访问应用需要的用户分配”，如 `SetupApplications.ps1` 所示。

### <a name="connection-with-powershell-fails-with-an-error-the-specified-credentials-are-invalid"></a>使用 PowerShell 连接失败并出现错误：“指定的凭据无效”
#### <a name="problem"></a>问题
使用 PowerShell 以“AzureActiveDirectory”安全模式连接到群集时，成功登录到 Azure AD 后，连接失败并显示错误：“指定的凭据无效”。

#### <a name="solution"></a>解决方案
解决方案同上。

### <a name="service-fabric-explorer-returns-a-failure-when-you-sign-in-aadsts50011"></a>登录时，Service Fabric Explorer 返回失败信息：“AADSTS50011”
#### <a name="problem"></a>问题
用户尝试登录到 Service Fabric Explorer 中的 Azure AD 时，页面返回故障：“AADSTS50011：回复地址 &lt;url&gt; 与针对应用程序 &lt;guid&gt; 配置的回复地址不匹配”。

![SFX 回复地址不匹配][sfx-reply-address-not-match]

#### <a name="reason"></a>原因
代表 Service Fabric Explorer 的群集 (web) 应用程序尝试针对 Azure AD 进行身份验证，在执行请求的过程中提供了重定向返回 URL。 但是，该 URL 并未列在 Azure AD 应用程序的“回复 URL”列表中。

#### <a name="solution"></a>解决方案
在群集 (web) 应用程序的“配置”选项卡上，将 Service Fabric Explorer 的 URL 添加到“回复 URL”列表，或替换列表中的某个项。 完成后，保存所做的更改。

![Web 应用程序回复 URL][web-application-reply-url]

### <a name="connect-the-cluster-by-using-azure-ad-authentication-via-powershell"></a>使用 Azure AD 身份验证通过 PowerShell 连接群集
若要连接 Service Fabric 群集，请使用以下 PowerShell 命令示例：

    Connect-ServiceFabricCluster -ConnectionEndpoint <endpoint> -KeepAliveIntervalInSec 10 -AzureActiveDirectory -ServerCertThumbprint <thumbprint>

若要了解有关 Connect-servicefabriccluster cmdlet 的信息，请参阅 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx)。

### <a name="can-i-reuse-the-same-azure-ad-tenant-in-multiple-clusters"></a>是否可将同一个 Azure AD 租户用于多个群集？
是的。 请记得将 Service Fabric Explorer 的 URL 添加到群集 (Web) 应用程序。 否则 Service Fabric Explorer 无法正常工作。

### <a name="why-do-i-still-need-a-server-certificate-while-azure-ad-is-enabled"></a>为何启用 Azure AD 时仍然需要服务器证书？
FabricClient 和 FabricGateway 执行相互身份验证。 使用 Azure AD 身份验证时，Azure AD 集成可将客户端标识提供给服务器，服务器证书将用于验证服务器标识。 有关 Service Fabric 证书的详细信息，请参阅 [X.509 证书和 Service Fabric][x509-certificates-and-service-fabric]。

<!-- Links -->
[azure-powershell]: /documentation/articles/powershell-install-configure/
[key-vault-get-started]: /documentation/articles/key-vault-get-started/
[aad-graph-api-docs]:https://msdn.microsoft.com/zh-cn/library/azure/ad/graph/api/api-catalog
[azure-classic-portal]: https://manage.windowsazure.cn
[service-fabric-rp-helpers]: https://github.com/ChackDan/Service-Fabric/tree/master/Scripts/ServiceFabricRPHelpers
[service-fabric-cluster-security]: /documentation/articles/service-fabric-cluster-security/
[active-directory-howto-tenant]: /documentation/articles/active-directory-howto-tenant/
[service-fabric-visualizing-your-cluster]: /documentation/articles/service-fabric-visualizing-your-cluster/
[service-fabric-manage-application-in-visual-studio]: /documentation/articles/service-fabric-manage-application-in-visual-studio/
[sf-aad-ps-script-download]:http://servicefabricsdkstorage.blob.core.windows.net/publicrelease/MicrosoftAzureServiceFabric-AADHelpers.zip
[azure-quickstart-templates]: https://github.com/Azure/azure-quickstart-templates
[service-fabric-secure-cluster-5-node-1-nodetype]: https://github.com/Azure/azure-quickstart-templates/blob/master/service-fabric-secure-cluster-5-node-1-nodetype/
[resource-group-template-deploy]: /documentation/articles/resource-group-template-deploy/
[x509-certificates-and-service-fabric]: /documentation/articles/service-fabric-cluster-security/#x509-certificates-and-service-fabric

<!-- Images -->
[cluster-security-arm-dependency-map]: ./media/service-fabric-cluster-creation-via-arm/cluster-security-arm-dependency-map.png
[cluster-security-cert-installation]: ./media/service-fabric-cluster-creation-via-arm/cluster-security-cert-installation.png
[assign-users-to-roles-button]: ./media/service-fabric-cluster-creation-via-arm/assign-users-to-roles-button.png
[assign-users-to-roles-dialog]: ./media/service-fabric-cluster-creation-via-arm/assign-users-to-roles.png
[sfx-select-certificate-dialog]: ./media/service-fabric-cluster-creation-via-arm/sfx-select-certificate-dialog.png
[sfx-reply-address-not-match]: ./media/service-fabric-cluster-creation-via-arm/sfx-reply-address-not-match.png
[web-application-reply-url]: ./media/service-fabric-cluster-creation-via-arm/web-application-reply-url.png
<!--update: add linux content;wording update;add anchors to sub titles-->