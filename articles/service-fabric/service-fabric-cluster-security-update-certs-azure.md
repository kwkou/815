<properties
   pageTitle="在 Azure 中添加、滚动更新和删除 Service Fabric 群集内使用的证书 | Azure"
   description="介绍如何上载辅助群集证书，然后滚动更新旧的主要证书。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/15/2016"
   wacn.date="10/24/2016"
   ms.author="chackdan"/>  


# 在 Azure 中添加或删除 Service Fabric 群集的证书

建议读者先阅读 [Cluster security scenarios](/documentation/articles/service-fabric-cluster-security/)（群集安全方案），了解 Service Fabric 使用 X.509 证书的方式。在继续下一步之前，必须先了解群集证书的定义和用途。

在创建群集期间配置证书安全性时，Service Fabric 允许指定两个群集证书（主要证书和辅助证书）。有关详细信息，请参阅 [creating an azure cluster via portal](/documentation/articles/service-fabric-cluster-creation-via-portal/)（通过门户创建 Azure 群集）或 [creating an azure cluster via Azure Resource Manager](/documentation/articles/service-fabric-cluster-creation-via-arm/)（通过 Azure Resource Manager 创建 Azure 群集）。如果通过 Resource Manager 进行部署并且只指定了一个群集证书，将使用该证书作为主要证书。在创建群集后，可以添加一个新证书作为辅助证书。

>[AZURE.NOTE] 对于安全群集，始终至少需要部署一个有效的（未吊销或过期）证书（主要或辅助），否则，群集无法正常运行。在所有有效证书过期前的 90 天，系统将针对节点生成警告跟踪和警告运行状况事件。Service Fabric 当前不会针对此主题发送电子邮件或其他任何通知。


## 使用门户添加辅助证书
若要添加另一个证书作为辅助证书，必须将该证书上载到 Azure 密钥保管库，然后将它部署到群集中的 VM。有关更多信息，请参阅 [Deploy certificates to VMs from a customer-managed key vault](http://blogs.technet.com/b/kv/archive/2015/07/14/vm_2d00_certificates.aspx)（将证书从客户管理的密钥保管库部署到 VM）。

1. 有关操作方法，请参阅 [Upload an X.509 certificate to the key vault](/documentation/articles/service-fabric-secure-azure-cluster-with-certs/#step-2-upload-the-x509-certificate-to-the-key-vault)（将 X.509 证书上载到密钥保管库）。

2. 登录到 [Azure 门户预览](https://portal.azure.cn/)并浏览到要将此证书添加到的群集资源。
3. 在“设置”下面，单击“安全性”显示“群集安全性”边栏选项卡。
4. 单击边栏选项卡顶部的“+证书”按钮，转到“添加证书”边栏选项卡。
5. 从下拉列表中选择“辅助证书指纹”，为上载到密钥保管库的辅助证书填写证书指纹。

>[AZURE.NOTE]
与执行群集创建工作流期间的不同之处在于，此处无需填充有关密钥保管库的详细信息，因为系统假设当时在此边栏选项卡中操作，已将证书部署到 VM，并且证书已在 VMSS 实例的本地证书存储中提供。

单击“证书”。随即开始部署，“群集安全性”边栏选项卡中会显示一个蓝色状态栏。

![门户中的证书指纹屏幕截图][SecurityConfigurations_02]

部署成功完成后，可以使用主要证书或辅助证书在群集上执行管理操作。

![正在部署证书的屏幕截图][SecurityConfigurations_03]

以下屏幕截图显示完成部署后的安全性边栏选项卡外观。

![部署后的证书指纹屏幕截图][SecurityConfigurations_08]


现在，可以使用刚刚添加的证书建立连接，在群集上执行操作。

>[AZURE.NOTE]
目前无法在门户中交换主要证书与辅助证书，该功能正在开发中。只要存在有效群集证书，群集就能正常运行。

## 添加辅助证书并使用 Resource Manager Powershell 将其交换为主要证书

以下步骤假设读者熟悉 Resource Manager 的工作原理，已使用 Resource Manager 模板至少部署了一个 Service Fabric 群集，并且已准备好用于设置群集的模板。此外，假设读者可以熟练使用 JSON。

>[AZURE.NOTE]
如需可用作参考或用作起点的示例模板与参数，可从此 [git-repo](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample)下载。

#### 编辑 Resource Manager 模板 

如果以前使用了 [git-repo](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample) 中的示例作为参考，会发现 5-VM-1-NodeTypes-Secure\_Step2.JSON 示例有所更改。使用 5-VM-1-NodeTypes-Secure\_Step1.JSON 部署安全群集

1. 打开用于部署群集的 Resource Manager 模板。
2. 添加“字符串”类型的新参数“secCertificateThumbprint”。如果使用的 Resource Manager 模板是在创建群集时从门户下载的，或者是从快速入门模板下载的，则只需搜索该参数，就会发现它已做了定义。
3. 找到“Microsoft.ServiceFabric/clusters”资源定义。在属性下面找到“Certificate”JSON 标记，如以下 JSON 代码片段所示。

	      "properties": {
	        "certificate": {
	          "thumbprint": "[parameters('certificateThumbprint')]",
	          "x509StoreName": "[parameters('certificateStoreValue')]"
	        }


4. 添加新标记“thumbprintSecondary”并为其指定值“[parameters('secCertificateThumbprint')]”。

资源定义现在应如下所示（根据具体的模板源，有时与下面的代码片段不完全相同）。在下面可以看到，执行的操作是指定一个新证书作为主要证书，同时将当前主要证书交换为辅助证书。这样就可以通过一个部署步骤，将当前证书滚动更新为新证书。



	      "properties": {
	        "certificate": {
	            "thumbprint": "[parameters('certificateThumbprint')]",
	            "thumbprintSecondary": "[parameters('secCertificateThumbprint')]",
	            "x509StoreName": "[parameters('certificateStoreValue')]"
	        },



#### 编辑模板文件，反映前面添加的新参数

如果以前使用了 [git-repo](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample) 中的示例作为参考，便可以开始在示例 5-VM-1-NodeTypes-Secure.paramters\_Step2.JSON 中进行更改


编辑 Resource Manager 模板参数文件，为 secCertificate 添加新参数，将现有的主要证书详细信息与辅助证书交换，然后将主要证书详细信息替换为新证书详细信息。


	    "secCertificateThumbprint": {
	      "value": "OLD Primary Certificate Thumbprint"
	    },
	   "secSourceVaultValue": {
	      "value": "OLD Primary Certificate Key Vault location"
	    },
	    "secCertificateUrlValue": {
	      "value": "OLD Primary Certificate location in the key vault"
	     },
	    "certificateThumbprint": {
	      "value": "New Certificate Thumbprint"
	    },
	    "sourceVaultValue": {
	      "value": "New Certificate Key Vault location"
	    },
	    "certificateUrlValue": {
	      "value": "New Certificate location in the key vault"
	     },



### 将模板部署到 Azure

1. 现在，可以将模板部署到 Azure。请打开 Azure PS 版本 1（或更高版本）的命令提示符。
2. 登录到 Azure 帐户，选择特定的 Azure 订阅。对于有权访问多个 Azure 订阅的用户而言，这是一个重要步骤。



	Login-AzureRmAccount -EnvironmentName AzureChinaCloud 
	Select-AzureRmSubscription -SubscriptionId <订阅 ID>



部署模板之前先进行测试。使用群集当前部署到的同一个资源组。


	Test-AzureRmResourceGroupDeployment -ResourceGroupName <Resource Group that your cluster is currently deployed to> -TemplateFile <PathToTemplate>



将模板部署到该资源组。使用群集当前部署到的同一个资源组。运行 New-AzureRmResourceGroupDeployment 命令。无需指定模式，因为默认值为 **incremental**。

>[AZURE.NOTE]
如果将 Mode 设置为 Complete，可能会无意中删除不在模板中的资源。因此请不要在此方案中使用该模式。
   


	New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName <Resource Group that your cluster is currently deployed to> -TemplateFile <PathToTemplate>


下面是已填充数据的同一个 Powershell 命令示例。


	$ResouceGroup2 = "chackosecure5"
	$TemplateFile = "C:\GitHub\Service-Fabric\ARM Templates\Cert Rollover Sample\5-VM-1-NodeTypes-Secure_Step2.json"
	$TemplateParmFile = "C:\GitHub\Service-Fabric\ARM Templates\Cert Rollover Sample\5-VM-1-NodeTypes-Secure.parameters_Step2.json"

	New-AzureRmResourceGroupDeployment -ResourceGroupName $ResouceGroup2 -TemplateParameterFile $TemplateParmFile -TemplateUri $TemplateFile -clusterName $ResouceGroup2



部署完成后，使用新证书连接到群集，然后执行一些查询。如果能够执行这些查询，则可以删除旧的主要证书。

如果使用自签名证书，请务必将它们导入本地 TrustedPeople 证书存储。


	######## Set up the certs on your local box
	Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople -FilePath c:\Mycertificates\chackdanTestCertificate9.pfx -Password (ConvertTo-SecureString -String abcd123 -AsPlainText -Force)
	Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My -FilePath c:\Mycertificates\chackdanTestCertificate9.pfx -Password (ConvertTo-SecureString -String abcd123 -AsPlainText -Force)


以下快速参考提供了用于连接到安全群集的命令

	$ClusterName= "chackosecure5.chinaeast.chinacloudapp.cn:19000"
	$CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7SD1D630F8F3" 

	Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
	    -X509Credential `
	    -ServerCertThumbprint $CertThumbprint  `
	    -FindType FindByThumbprint `
	    -FindValue $CertThumbprint `
	    -StoreLocation CurrentUser `
	    -StoreName My

以下快速参考提供了用于获取群集运行状况的命令

	Get-ServiceFabricClusterHealth 

 
## 使用门户删除旧证书
下面是删除旧证书，以使群集不使用它的过程：

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)并导航到群集的安全设置。
2. 右键单击要删除的证书
3. 选择“删除”并根据提示操作。

[SecurityConfigurations_05]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_05.png


## 后续步骤
有关群集管理的详细信息，请阅读以下文章：

- [Service Fabric 群集升级过程与期望](/documentation/articles/service-fabric-cluster-upgrade/)
- [Setup role-based access for clients（为客户端设置基于角色的访问）](/documentation/articles/service-fabric-cluster-security-roles/)


<!--Image references-->

[SecurityConfigurations_02]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_02.png  
[SecurityConfigurations_03]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_03.png  
[SecurityConfigurations_05]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_05.png  
[SecurityConfigurations_08]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_08.png

<!---HONumber=Mooncake_1017_2016-->