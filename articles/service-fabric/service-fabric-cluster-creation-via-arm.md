<properties
   pageTitle="使用 Azure Resource Manager 创建安全 Service Fabric 群集 | Azure"
   description="本文介绍如何在 Azure 中设置一个使用 Azure Resource Manager、Azure 密钥保管库和 Azure Active Directory (AAD) 进行客户端身份验证的安全 Service Fabric 群集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="chackdan"
   manager="timlt"
   editor="vturecek"/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="12/08/2016"
   wacn.date="04/11/2017"
   ms.author="chackdan"/>  


# 使用 Azure Resource Manager 在 Azure 中创建 Service Fabric 群集

> [AZURE.SELECTOR]
- [Azure 资源管理器](/documentation/articles/service-fabric-cluster-creation-via-arm/)
- [Azure 门户](/documentation/articles/service-fabric-cluster-creation-via-portal/)

本指南逐步介绍如何使用 Azure Resource Manager 在 Azure 中设置安全的 Azure Service Fabric 群集，我们意识到文档很长，但除非已熟悉步骤和内容，否则请执行所有步骤。

其中包括以下步骤：

* 设置可保护群集和应用程序安全的 Key Vault 并上传证书。
* 使用 Azure Resource Manager 在 Azure 中创建受保护的群集。
* 使用 Azure Active Directory (AAD) 对用户进行身份验证以进行群集管理。

安全的群集是防止未经授权访问管理操作的群集，这些操作包括部署、升级和删除应用程序、服务及其包含的数据。不安全的群集是任何人都可以随时连接并执行管理操作的群集。尽管可以创建不安全的群集，但**强烈建议创建安全的群集**。不安全的群集**无法在事后受到保护** - 要保护群集，必须创建新群集。

## 登录到 Azure
本指南使用 [Azure PowerShell][azure-powershell]。开始新的 PowerShell 会话时，请登录到 Azure 帐户并选择订阅，然后执行 Azure 命令。

登录到 Azure 帐户：


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud


选择订阅：


	Get-AzureRmSubscription
	Set-AzureRmContext -SubscriptionId <guid>


## 设置密钥保管库
本部分逐步介绍如何为 Azure 中的 Service Fabric 群集以及为 Service Fabric 应用程序创建密钥保管库。有关密钥保管库的完整指南，请参阅 [Key Vault getting started guide][key-vault-get-started]（密钥保管库入门指南）。

Service Fabric 使用 X.509 证书保护群集，提供应用程序安全功能。Azure 密钥保管库用于管理 Azure 中 Service Fabric 群集的证书。在 Azure 中部署群集时，负责创建 Service Fabric 群集的 Azure 资源提供程序将从密钥保管库提取证书，然后将其安装在群集 VM 上。

下图演示密钥保管库、Service Fabric 群集与 Azure 资源提供程序（在创建群集时使用密钥保管库中存储的证书）之间的关系：

![证书安装][cluster-security-cert-installation]  


### 创建资源组。
第一个步骤是专门针对密钥保管库创建资源组。建议将密钥保管库放入其自身的资源组。这样，便可以删除计算和存储资源组（包括具有 Service Fabric 群集的资源组），而不会丢失密钥和密码。包含 Key Vault 的资源组**必须与正在使用它的群集位于同一区域**。

如果计划在多个区域部署群集，建议在命名资源组和 keyvault 时，使其名称可告知其所属的区域。


		New-AzureRmResourceGroup -Name mycluster-keyvault -Location 'China East'
应看到如下所示的输出。警告：此 cmdlet 的输出对象类型会在将来的版本中进行修改。
	
		ResourceGroupName : mycluster-keyvault
		Location          : chinaeast
		ProvisioningState : Succeeded
		Tags              :
		ResourceId        : /subscriptions/<guid>/resourceGroups/mycluster-keyvault


<a id="new-key-vault"></a>

### 创建新的 Key Vault
在新资源组中创建密钥保管库。**必须针对部署启用** Key Vault，使计算资源提供程序能够从中获取证书并在虚拟机实例上安装：



		New-AzureRmKeyVault -VaultName 'myvault' -ResourceGroupName 'mycluster-keyvault' -Location 'China East' -EnabledForDeployment
	
应看到如下所示的输出。
	
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

## 使用现有的 Key Vault

如果拥有 Key Vault 并且想要使用，必须针对部署启用它。**必须针对部署启用** Key Vault，使计算资源提供程序能够从中获取证书并在群集节点上安装：



	Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -EnabledForDeployment


<a id="add-certificate-to-key-vault"></a>

## 将证书添加到密钥保管库

证书在 Service Fabric 中用于提供身份验证和加密，为群集及其应用程序提供全方位的保护。有关如何在 Service Fabric 中使用证书的详细信息，请参阅 [Service Fabric cluster security scenarios][service-fabric-cluster-security]（Service Fabric 群集安全方案）。

### 群集和服务器证书（必需）
需要使用此证书来保护群集以及防止未经授权访问群集。此证书通过多种方式保护群集：

* **群集身份验证：** 在群集联合的情况下对节点间的通信进行身份验证。只有可以使用此证书自我证明身份的节点才能加入群集。
* **服务器身份验证：** 在管理客户端上对群集管理终结点进行身份验证，使管理客户端知道它正在与真正的群集通信。此证书还通过 HTTPS 为 HTTPS 管理 API 和 Service Fabric Explorer 提供 SSL。

为满足这些用途，该证书必须符合以下要求：

 - 证书必须包含私钥。
 - 必须为密钥交换创建证书，并且该证书可导出到个人信息交换 (.pfx) 文件。
 - 证书的使用者名称必须与用于访问 Service Fabric 群集的域匹配。只有符合此要求，才能为群集的 HTTPS 管理终结点和 Service Fabric Explorer 提供 SSL。无法从证书颁发机构 (CA) 获取 `.chinacloudapp.cn` 域的 SSL 证书。必须获取群集的自定义域名。在从 CA 请求证书时，该证书的使用者名称必须与用于群集的自定义域名匹配。

### 应用程序证书（可选）
可以出于应用程序安全目的，在群集上安装任意数量的附加证书。在创建群集之前，请考虑需要在节点上安装证书的应用程序安全方案，例如：

 - 加密和解密应用程序配置值
 - 在复制期间跨节点加密数据

### 格式化证书以供 Azure 资源提供程序使用
可以直接通过密钥保管库添加和使用私钥文件 (.pfx)。但是，Azure 计算资源提供程序要求以特殊 JSON 格式存储密钥，其中包含 .pfx 作为 Base-64 编码字符串和私钥密码。若要满足这些要求，必须将密钥放入 JSON 字符串，然后在密钥保管库中将其存储为 *机密* 。

为了简化此过程，[GitHub 上提供了][service-fabric-rp-helpers]一个 PowerShell 模块。请遵循以下步骤使用该模块：

1. 将存储库的整个内容下载到本地目录。
2. 导航到本地目录
2. 在 PowerShell 窗口中导入 ServiceFabricRPHelpers 模块：

		Import-Module "C:..\\ServiceFabricRPHelpers\\ServiceFabricRPHelpers.psm1"

     
此 PowerShell 模块中的 `Invoke-AddCertToKeyVault` 命令自动将证书私钥的格式设置为 JSON 字符串，并将它上载到密钥保管库。使用该字符串可将群集证书与任何其他应用程序证书添加到密钥保管库。针对要在群集中安装的其他任何证书重复此步骤。

#### 上传现有证书 

	 Invoke-AddCertToKeyVault -SubscriptionId <guid> -ResourceGroupName mycluster-keyvault -Location "China East" -VaultName myvault -CertificateName mycert -Password "<password>" -UseExistingCertificate -ExistingPfxFilePath "C:\path\to\mycertkey.pfx"
	

如果收到如下所示的错误，这通常意味着资源 URL 发生冲突，因此，请更改 keyvault 名称。


	Set-AzureKeyVaultSecret : The remote name could not be resolved: 'chinaeastkv.vault.azure.cn'
	At C:\Users\chackdan\Documents\GitHub\Service-Fabric\Scripts\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1:440 char:11
	+ $secret = Set-AzureKeyVaultSecret -VaultName $VaultName -Name $Certif ...
	+           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	    + CategoryInfo          : CloseError: (:) [Set-AzureKeyVaultSecret], WebException
	    + FullyQualifiedErrorId : Microsoft.Azure.Commands.KeyVault.SetAzureKeyVaultSecret
    


成功完成后，应看到如下所示的输出。


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


**记下上面的三个字符串 - CertificateThumbprint、SourceVault 和 CertificateURL。** 需要使用这些字符串来设置安全的 Service Fabric 群集，用于应用程序安全的任何应用程序证书也需要这些字符串。如果没有在某个位置保存这三个字符串，很难在稍后通过查询 keyvault 获取它们。


<a id="add-self-signed-certificate-to-key-vault"></a>

#### 创建自签名证书并上传到 keyvault

如果已将证书上传到 keyvault，则跳过此步骤，此步骤用于生成新的自签名证书并将其上传到 keyvault。更改以下参数并运行脚本。应提示输入证书密码。



	$ResouceGroup = "chackochinaeastkv"
	$VName = "chackokv2"
	$SubID = "6c653126-e4ba-42cd-a1dd-f7bf96ae7a47"
	$locationRegion = "chinaeast" 
	$newCertName = "chackotestcertificate1"
	$dnsName = "www.mycluster.chinaeast.mydomain.com" #The certificate's subject name must match the domain used to access the Service Fabric cluster.
	$localCertPath = "C:\MyCertificates" # location where you want the .PFX to be stored

	 Invoke-AddCertToKeyVault -SubscriptionId $SubID -ResourceGroupName $ResouceGroup -Location $locationRegion -VaultName $VName -CertificateName $newCertName -CreateSelfSignedCertificate -DnsName $dnsName -OutputPath $localCertPath


如果收到如下所示的错误，这通常意味着资源 URL 发生冲突，因此，请更改 keyvault 名称、RG 名称等。

<p><font color="red">
Set-AzureKeyVaultSecret : The remote name could not be resolved: 'chinaeastkv.vault.azure.cn'
At C:\Users\chackdan\Documents\GitHub\Service-Fabric\Scripts\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1:440 char:11
+ $secret = Set-AzureKeyVaultSecret -VaultName $VaultName -Name $Certif ...
+           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzureKeyVaultSecret], WebException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.KeyVault.SetAzureKeyVaultSecret
    
</font></p>

成功完成后，应看到如下所示的输出。


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



**记下上面的三个字符串 - CertificateThumbprint、SourceVault 和 CertificateURL。** 需要使用这些字符串来设置安全的 Service Fabric 群集，用于应用程序安全的任何应用程序证书也需要这些字符串。如果没有在某个位置保存这三个字符串，很难在稍后通过查询 keyvault 获取它们。



 此时，应该已经拥有以下设置并准备好在 Azure 中执行操作：

* 密钥保管库资源组
* Key Vault 及其 URL（在上面的 powershell 输出中称为源保管库）。
* Keyvault 中的群集服务器身份验证证书及其 URL
* Keyvault 中的应用程序证书及其 URL


<a id="add-AAD-for-client"></a>

## 为客户端身份验证设置 Azure Active Directory

AAD 可让组织（称为租户）管理用户对应用程序的访问，这些应用程序划分为提供基于 Web 的 UI 的应用程序，以及提供本机客户端体验的应用程序。本文假设已创建了一个租户。如果未创建，请先阅读 [How to get an Azure Active Directory tenant][active-directory-howto-tenant]（如何获取 Azure Active Directory 租户）。

Service Fabric 群集提供其管理功能的各种入口点，包括基于 Web 的 [Service Fabric Explorer][service-fabric-visualizing-your-cluster] 和 [Visual Studio][service-fabric-manage-application-in-visual-studio]。因此，要创建两个 AAD 应用程序来控制对群集的访问：一个 Web 应用程序和一个本机应用程序。

为了简化涉及到配置 AAD 与 Service Fabric 群集的一些步骤，我们创建了一组 Windows PowerShell 脚本。

>[AZURE.NOTE] 必须在创建群集 *之前* 执行这些步骤；因此，在脚本需要群集名称和终结点的情况下，这些应该是计划的值，而不是所创建的值。

1. [将脚本下载][sf-aad-ps-script-download]到计算机。
2. 右键单击 zip 文件，选择“属性”，选中“取消阻止”复选框，然后单击“应用”。
3. 解压缩 zip 文件。
4. 运行 `SetupApplications.ps1` 并提供 TenantId、ClusterName 和 WebApplicationReplyUrl 作为参数。例如：


    	.\SetupApplications.ps1 -TenantId '690ec069-8200-4068-9d01-5aaf188e557a' -ClusterName 'mycluster' -WebApplicationReplyUrl 'https://mycluster.chinaeast.cloudapp.chinacloudapi.cn:19080/Explorer/index.html'


    执行 PowerShell 命令 `Get-AzureSubscription`，可找到 **TenantId**。此命令会显示每个订阅的 **TenantId**。

    脚本创建的 AAD 应用程序带有 **ClusterName** 前缀。它不需要完全符合实际群集名称，因为它只是让你更轻松地将 AAD 项目映射到与其配合使用的 Service Fabric 群集。

    **WebApplicationReplyUrl** 是 AAD 在完成登录过程之后将用户返回到的默认终结点。你应该将此项设置为群集的 Service Fabric Explorer 终结点，默认值为：

    https://&lt;cluster_domain&gt;:19080/Explorer  


    系统将提示登录到具有 AAD 租户管理权限的帐户。完成此操作后，脚本将继续创建 Web 和本机应用程序来代表 Service Fabric 群集。在 [Azure 经典门户][azure-classic-portal]中查看租户的应用程序时，应会看到两个新条目：

    - *ClusterName*\_Cluster
    - *ClusterName*\_Client

    该脚本将列显在下一部分创建群集时 Azure Resource Manager 模板所需的 Json，因此请不要关闭 PowerShell 窗口。


	"azureActiveDirectory": {
	  "tenantId":"<guid>",
	  "clusterApplication":"<guid>",
	  "clientApplication":"<guid>"
	},


## 创建 Service Fabric 群集 Resource Manager 模板
在本部分中，在 Service Fabric 群集 Resource Manager 模板中使用上述 PowerShell 命令的输出。

可以从 [GitHub 上的 Azure 快速入门模板库][azure-quickstart-templates]获取示例 Resource Manager 模板。这些模板可用作群集模板的起点。

### 创建 Resource Manager 模板
本指南使用 [5 节点安全群集][service-fabric-secure-cluster-5-node-1-nodetype-wad]示例模板和模板参数。将 `azuredeploy.json` 和 `azuredeploy.parameters.json` 下载到计算机，在偏好的文本编辑器中打开这两个文件。

### 添加证书
证书通过引用包含证书密钥的密钥保管库添加到群集 Resource Manager 模板。建议将这些密钥保管库值放在 Resource Manager 模板参数文件中，以便可以重复使用 Resource Manager 模板文件，避免输入特定于部署的值。

#### 将所有证书添加到 VMSS osProfile
必须在 VMSS 资源 (Microsoft.Compute/virtualMachineScaleSets) 的 osProfile 节中配置应在群集中安装的每个证书。这样就会指示资源提供程序在 VM 上安装证书。这包括群集证书，以及打算用于应用程序的任何应用程序安全证书：


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


#### 配置 Service Fabric 群集证书
此外，还必须在 Service Fabric 群集资源 (Microsoft.ServiceFabric/clusters) 中，以及 VMSS 资源的 VMSS Service Fabric 扩展中配置群集身份验证证书。这样，Service Fabric 资源提供程序便可以将该证书配置为用于群集身份验证及管理终结点的服务器身份验证。

##### VMSS 资源：


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


##### Service Fabric 资源：


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


### 插入 AAD 配置
可将前面创建的 AAD 配置直接插入 Resource Manager 模板，不过建议最好先将参数值提取到 parameters 文件，以便可以重复使用 Resource Manager 模板文件，避免输入特定于部署的值。


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

最后，使用密钥保管库和 AAD PowerShell 命令的输出值填充参数文件：


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
	            "value": "https://myvault.vault.chinalcoudapi.cn:443/secrets/myclustercert/4d087088df974e869f1c0978cb100e47"
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

此时，应已准备好以下各项：

 - 密钥保管库资源组
    - 密钥保管库
    - 群集服务器身份验证证书
    - 数据加密证书
 - Azure Active Directory 租户
    - 用于基于 Web 的管理和 Service Fabric Explorer 的 AAD 应用程序
    - 用于本机客户端管理的 AAD 应用程序
    - 已分配角色的用户
 - Service Fabric 群集 Resource Manager 模板
    - 通过密钥保管库配置的证书
    - 配置的 Azure Active Directory

下图演示密钥保管库和 AAD 配置在 Resource Manager 模板中的作用。

![Resource Manager 依赖关系图][cluster-security-arm-dependency-map]  


## 创建群集
现在可以使用 [Azure 资源模板部署][resource-group-template-deploy]来创建群集。

#### 测试
运行以下 PowerShell 命令，使用 parameters 文件测试 Resource Manager 模板：


	Test-AzureRmResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json


#### 部署
如果 Resource Manager 模板通过测试，请运行以下 PowerShell 命令，使用 parameters 文件来部署 Resource Manager 模板：


	New-AzureRmResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json


## 将用户分配到角色

创建用于表示群集的应用程序后，需要将用户分配到 Service Fabric 支持的角色：只读和管理员。可以使用 [Azure 经典门户][azure-classic-portal]执行此操作。

1. 导航到你的租户，然后选择“应用程序”。
2. 选择名称类似于 `myTestCluster_Cluster` 的 Web 应用程序。
3. 单击“用户”选项卡。
4. 选择要分配的用户，然后单击屏幕底部的“分配”按钮。

    ![将用户分配到角色按钮][assign-users-to-roles-button]  


5. 选择要分配给用户的角色。

    ![将用户分配到角色][assign-users-to-roles-dialog]  


>[AZURE.NOTE] 有关 Service Fabric 中角色的详细信息，请参阅 [Role-based access control for Service Fabric clients](/documentation/articles/service-fabric-cluster-security-roles/)（适用于 Service Fabric 客户端的基于角色的访问控制）。


## 后续步骤
此时，已创建一个使用 Azure Active Directory 进行管理身份验证的安全群集。接下来，请[连接到该群集](/documentation/articles/service-fabric-connect-to-secure-cluster/)，了解如何[管理应用程序机密](/documentation/articles/service-fabric-application-secret-management/)。

## 排查用于客户端身份验证的 Azure Active Directory 的设置问题

如果设置用于客户端身份验证的 Azure Active Directory 时遇到问题，请参阅下面建议的可能解决方法。

### Service Fabric Explorer 提示选择证书
#### 问题
在 Service Fabric Explorer 的 AAD 登录页上成功登录后，浏览器返回主页，但出现提示选择证书的对话框。

![SFX - 选择证书对话框][sfx-select-certificate-dialog]  


#### 原因
未在 AAD 群集应用程序中为用户分配角色。因此，Service Fabric 群集的 AAD 身份验证失败。Service Fabric Explorer 将故障回复到证书身份验证。

#### 解决方案
遵循有关设置 AAD 的说明操作，并为用户分配角色。另外，建议启用“访问应用需要进行用户分配”，就像使用 `SetupApplications.ps1` 时一样。

### 使用 PowerShell 连接失败并出现错误：“指定的凭据无效”
#### 问题
使用 PowerShell 以“AzureActiveDirectory”安全模式连接到群集时，在 AAD 登录页上成功登录后，连接失败并显示错误：“指定的凭据无效”。

#### 解决方案
同上。

### Service Fabric Explorer 登录返回错误代码：AADSTS50011
#### 问题
在 Service Fabric Explorer 的 AAD 登录页上成功登录后，页面返回登录失败 -“AADSTS50011: 回复地址 &lt;url&gt; 与针对应用程序 &lt;guid&gt; 配置的回复地址不匹配”。

![SFX 回复地址不匹配][sfx-reply-address-not-match]  


#### 原因
代表 Service Fabric Explorer 的群集 (Web) 应用程序尝试针对 AAD 进行身份验证，在执行请求的过程中提供了重定向返回 URL。但是，该 URL 并未列在 AAD 应用程序的“回复 URL”列表中。

#### 解决方案
将 Service Fabric Explorer 的 URL 添加到群集 (Web) 应用程序“配置”选项卡中的“回复 URL”，或替换列表中的某个项。然后保存。

![Web 应用程序回复 URL][web-application-reply-url]  


### 如何通过 PowerShell 将群集与 AAD 身份验证连接
使用以下 PowerShell 命令示例连接 Service Fabric 群集：


	Connect-ServiceFabricCluster -ConnectionEndpoint <endpoint> -KeepAliveIntervalInSec 10 -AzureActiveDirectory -ServerCertThumbprint <thumbprint>


若要了解有关 Connect-ServiceFabricCluster cmdlett 的信息，请参阅 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx)。

### 是否可将同一个 AAD 租户用于多个群集？
是的。但请记得将 Service Fabric Explorer 的 URL 添加到群集 (Web) 应用程序，否则 Service Fabric Explorer 无法正常工作。

### 为何启用 AAD 时仍然需要服务器证书？
FabricClient 和 FabricGateway 执行相互身份验证。使用 AAD 身份验证时，AAD 集成可将客户端标识提供给服务器，服务器证书将用于验证服务器标识。有关如何在 Service Fabric 上使用证书的详细信息，请参阅 [X.509 证书和 Service Fabric][x509-certificates-and-service-fabric]

<!-- Links -->
[azure-powershell]: /documentation/articles/powershell-install-configure/
[key-vault-get-started]: /documentation/articles/key-vault-get-started/
[aad-graph-api-docs]: https://msdn.microsoft.com/zh-cn/library/azure/ad/graph/api/api-catalog
[azure-classic-portal]: https://manage.windowsazure.cn
[service-fabric-rp-helpers]: https://github.com/ChackDan/Service-Fabric/tree/master/Scripts/ServiceFabricRPHelpers
[service-fabric-cluster-security]: /documentation/articles/service-fabric-cluster-security/
[active-directory-howto-tenant]: /documentation/articles/active-directory-howto-tenant/
[service-fabric-visualizing-your-cluster]: /documentation/articles/service-fabric-visualizing-your-cluster/
[service-fabric-manage-application-in-visual-studio]: /documentation/articles/service-fabric-manage-application-in-visual-studio/
[sf-aad-ps-script-download]: http://servicefabricsdkstorage.blob.core.windows.net/publicrelease/MicrosoftAzureServiceFabric-AADHelpers.zip
[azure-quickstart-templates]: https://github.com/Azure/azure-quickstart-templates
[service-fabric-secure-cluster-5-node-1-nodetype-wad]: https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype
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

<!---HONumber=Mooncake_0116_2017-->
<!--update: add sections("上传现有证书","创建自签名证书并上传到 keyvault","如何通过 PowerShell 将群集与 AAD 身份验证连接")-->