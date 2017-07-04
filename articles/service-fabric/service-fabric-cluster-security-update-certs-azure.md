<properties
    pageTitle="管理 Azure Service Fabric 群集中的证书 | Azure"
    description="介绍如何向 Service Fabric 群集添加新的证书、滚动更新证书和从群集中删除证书。"
    services="service-fabric"
    documentationcenter=".net"
    author="ChackDan"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="91adc3d3-a4ca-46cf-ac5f-368fb6458d74"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/09/2017"
    wacn.date="04/24/2017"
    ms.author="chackdan"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="a737a615ad6db164a4eff978558753008db5a421"
    ms.lasthandoff="04/14/2017" />

# <a name="add-or-remove-certificates-for-a-service-fabric-cluster-in-azure"></a>在 Azure 中添加或删除 Service Fabric 群集的证书
建议先了解 Service Fabric 使用 X.509 证书的方式，并熟悉[群集安全性应用场景](/documentation/articles/service-fabric-cluster-security/)。 在继续下一步之前，必须先了解群集证书的定义和用途。

在创建群集期间配置证书安全性时，Service Fabric 允许指定两个群集证书（主要证书和辅助证书）以及客户端证书。 请参阅[通过门户创建 Azure 群集](/documentation/articles/service-fabric-cluster-creation-via-portal/)或[通过 Azure Resource Manager 创建 Azure 群集](/documentation/articles/service-fabric-cluster-creation-via-arm/)，了解在创建时进行相关设置的详细信息。 如果创建时只指定了一个群集证书，将使用该证书作为主要证书。 在创建群集后，可以添加一个新证书作为辅助证书。

> [AZURE.NOTE]
> 对于安全群集，始终至少需要部署一个有效的（未吊销或过期）群集证书（主证书或辅助证书），否则，群集无法正常运行。 在所有有效证书过期前的 90 天，系统将针对节点生成警告跟踪和警告运行状况事件。 Service Fabric 当前不会针对此主题发送电子邮件或其他任何通知。 
> 
> 

## <a name="add-a-secondary-cluster-certificate-using-the-portal"></a>使用门户添加辅助群集证书

无法通过 Azure 门户添加辅助群集证书。为此必须使用 Azure PowerShell。本文档稍后将概述该过程。

## <a name="swap-the-cluster-certificates-using-the-portal"></a>使用门户交换群集证书

成功部署辅助群集证书后，如果想要交换主证书和辅助证书，请导航到“安全”边栏选项卡，然后从上下文菜单中选择“与主证书交换”选项，将辅助证书交换为主证书。

![交换证书][Delete_Swap_Cert]

## <a name="remove-a-cluster-certificate-using-the-portal"></a>使用门户删除群集证书

对于安全群集，始终至少需要部署一个有效的（未吊销或过期）证书（主证书或辅助证书），否则，群集无法正常运行。

若要删除群集安全性使用的辅助证书，请导航到“安全”边栏选项卡，然后从辅助证书的上下文菜单中选择“删除”选项。

如果目的是删除标记为主证书的证书，则首先需要将该证书交换为辅助证书，然后在升级完成后删除辅助证书。

## <a name="add-a-secondary-certificate-using-resource-manager-powershell"></a>使用 Resource Manager Powershell 添加辅助证书

以下步骤假设读者熟悉 Resource Manager 的工作原理，已使用 Resource Manager 模板至少部署了一个 Service Fabric 群集，并且已准备好用于设置群集的模板。 此外，还有一个前提就是，你可以熟练使用 JSON。

> [AZURE.NOTE]
> 如需可参考或入手的示例模板和参数，请从此 [git-repo](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample) 下载。 
> 
> 

### <a name="edit-your-resource-manager-template"></a>编辑 Resource Manager 模板

为了便于参考，示例 5-VM-1-NodeTypes-Secure_Step2.JSON 包含我们将进行的所有编辑。 该示例位于 [git-repo](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample)。

**请确保执行所有步骤**

**步骤 1：**打开用于部署群集的 Resource Manager 模板。 （如果已从上述存储库下载此示例，则使用 5-VM-1-NodeTypes-Secure_Step1.JSON 部署安全的群集，然后打开该模板）。

**步骤 2：**向模板的参数部分添加**两个新参数**“secCertificateThumbprint”和“secCertificateUrlValue”，类型为“string”。 可复制以下代码片段，并将其添加到该模板。 根据具体的模板源，可能已经存在这些定义，如果是这样，请转至下一步。 

	   "secCertificateThumbprint": {
	      "type": "string",
	      "metadata": {
	        "description": "Certificate Thumbprint"
	      }
	    },
	    "secCertificateUrlValue": {
	      "type": "string",
	      "metadata": {
	        "description": "Refers to the location URL in your key vault where the certificate was uploaded, it is should be in the format of https://<name of the vault>.vault.azure.cn:443/secrets/<exact location>"
	      }
	    },


**步骤 3：**对 **Microsoft.ServiceFabric/clusters** 资源进行更改 - 在模板中找到 "Microsoft.ServiceFabric/clusters" 资源定义。 在该定义的属性下，找到 "Certificate" JSON 标记，如以下 JSON 代码片段所示。

	      "properties": {
	        "certificate": {
	          "thumbprint": "[parameters('certificateThumbprint')]",
	          "x509StoreName": "[parameters('certificateStoreValue')]"
	        }

添加新标记“thumbprintSecondary”并为其指定值“[parameters('secCertificateThumbprint')]”。  

资源定义现在应如下所示（根据具体的模板源，有时与下面的代码片段不完全相同）。 


	      "properties": {
	        "certificate": {
	            "thumbprint": "[parameters('certificateThumbprint')]",
	            "thumbprintSecondary": "[parameters('secCertificateThumbprint')]",
	            "x509StoreName": "[parameters('certificateStoreValue')]"
         	}

如果要**滚动更新证书**，请将新证书指定为主要证书，并将当前的主要证书移为辅助证书。 这样就可以通过一个部署步骤，将当前主要证书滚动更新为新证书。


	      "properties": {
	        "certificate": {
	          "thumbprint": "[parameters('secCertificateThumbprint')]",
			  "thumbprintSecondary": "[parameters('certificateThumbprint')]",
	          "x509StoreName": "[parameters('certificateStoreValue')]"
	     }

**步骤 4：**对**所有** **Microsoft.Compute/virtualMachineScaleSets** 资源定义进行更改 - 查找 Microsoft.Compute/virtualMachineScaleSets 资源定义。 滚动到 "publisher": "Microsoft.Azure.ServiceFabric"，位于 "virtualMachineProfile" 下。

在 Service Fabric 发布服务器设置中，应看到类似如下的内容。

![Json_Pub_Setting1][Json_Pub_Setting1]

向其中添加新的证书项


               "certificateSecondary": {
                    "thumbprint": "[parameters('secCertificateThumbprint')]",
                    "x509StoreName": "[parameters('certificateStoreValue')]"
                    }
                  },


属性现在应如下所示

![Json_Pub_Setting2][Json_Pub_Setting2]

如果要**滚动更新证书**，请将新证书指定为主要证书，并将当前的主要证书移为辅助证书。 这样就可以通过一个部署步骤，将当前证书滚动更新为新证书。 

               "certificate": {
                   "thumbprint": "[parameters('secCertificateThumbprint')]",
                   "x509StoreName": "[parameters('certificateStoreValue')]"
                     },
               "certificateSecondary": {
                    "thumbprint": "[parameters('certificateThumbprint')]",
                    "x509StoreName": "[parameters('certificateStoreValue')]"
                    }
                  },


属性现在应如下所示

![Json_Pub_Setting3][Json_Pub_Setting3]


**步骤 5：**对**所有** **Microsoft.Compute/virtualMachineScaleSets** 资源定义进行更改 - 查找 Microsoft.Compute/virtualMachineScaleSets 资源定义。 滚动到 "vaultCertificates":，位于 "OSProfile" 下。 你应该会看到类似下面的屏幕。


![Json_Pub_Setting4][Json_Pub_Setting4]

向其添加 secCertificateUrlValue。 使用以下代码片段：


                  {
                    "certificateStore": "[parameters('certificateStoreValue')]",
                    "certificateUrl": "[parameters('secCertificateUrlValue')]"
                  }


现在，生成的 Json 应如下所示。
![Json_Pub_Setting5][Json_Pub_Setting5]


> [AZURE.NOTE]
> 请确保已对模板中的 Nodetypes/Microsoft.Compute/virtualMachineScaleSets 资源定义重复执行了步骤 4 和 5。 如果未针对其中一个资源定义执行这些步骤，证书将不会安装在 VMSS 上，并且群集会发生不可预知的结果，包括群集关闭（最终群集无法使用有效的证书来确保安全）。 因此，在继续下一步之前请仔细检查。
> 
> 


### <a name="edit-your-template-file-to-reflect-the-new-parameters-you-added-above"></a>编辑模板文件，反映前面添加的新参数
如果参考了 [git-repo](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample) 中的示例，便可开始更改示例 5-VM-1-NodeTypes-Secure.paramters_Step2.JSON 

编辑 Resource Manager 模板参数文件，添加 secCertificateThumbprint 和 secCertificateUrlValue 的两个新参数。 


	    "secCertificateThumbprint": {
	      "value": "thumbprint value"
	    },
	    "secCertificateUrlValue": {
	      "value": "Refers to the location URL in your key vault where the certificate was uploaded, it is should be in the format of https://<name of the vault>.vault.azure.cn:443/secrets/<exact location>"
	     },


### <a name="deploy-the-template-to-azure"></a>将模板部署到 Azure

- 现在，可以将模板部署到 Azure。请打开 Azure PS 版本 1（或更高版本）的命令提示符。
- 登录到 Azure 帐户，选择特定的 Azure 订阅。对于有权访问多个 Azure 订阅的用户而言，这是一个重要步骤。



	Login-AzureRmAccount -EnvironmentName AzureChinaCloud 
	Select-AzureRmSubscription -SubscriptionId <Subcription ID>


部署模板之前先进行测试。 使用群集当前部署到的同一个资源组。


	Test-AzureRmResourceGroupDeployment -ResourceGroupName <Resource Group that your cluster is currently deployed to> -TemplateFile <PathToTemplate>


将模板部署到该资源组。 使用群集当前部署到的同一个资源组。 运行 New-AzureRmResourceGroupDeployment 命令。 无需指定模式，因为默认值为 **incremental**。

> [AZURE.NOTE]
> 如果将 Mode 设置为 Complete，可能会无意中删除不在模板中的资源。 因此请不要在此方案中使用该模式。
> 
> 

	New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName <Resource Group that your cluster is currently deployed to> -TemplateFile <PathToTemplate>

下面是已填充数据的同一个 Powershell 命令示例。


	$ResouceGroup2 = "chackosecure5"
	$TemplateFile = "C:\GitHub\Service-Fabric\ARM Templates\Cert Rollover Sample\5-VM-1-NodeTypes-Secure_Step2.json"
	$TemplateParmFile = "C:\GitHub\Service-Fabric\ARM Templates\Cert Rollover Sample\5-VM-1-NodeTypes-Secure.parameters_Step2.json"

	New-AzureRmResourceGroupDeployment -ResourceGroupName $ResouceGroup2 -TemplateParameterFile $TemplateParmFile -TemplateUri $TemplateFile -clusterName $ResouceGroup2


部署完成后，使用新证书连接到群集，然后执行一些查询。 如果能够执行这些查询， 则可以删除旧证书。 

如果使用自签名证书，请务必将它们导入本地 TrustedPeople 证书存储。

	######## Set up the certs on your local box
	Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople -FilePath c:\Mycertificates\chackdanTestCertificate9.pfx -Password (ConvertTo-SecureString -String abcd123 -AsPlainText -Force)
	Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My -FilePath c:\Mycertificates\chackdanTestCertificate9.pfx -Password (ConvertTo-SecureString -String abcd123 -AsPlainText -Force)


以下快速参考提供了用于连接到安全群集的命令 

	$ClusterName= "chackosecure5.chinaeast.cloudapp.chinacloudapi.cn:19000"
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

## <a name="deploying-application-certificates-to-the-cluster"></a>将应用程序证书部署到群集。

可以使用上面步骤 5 中所述的相同步骤将证书从 Key Vault 部署到节点。 只需定义并使用不同的参数即可。


## <a name="adding-or-removing-client-certificates"></a>添加或删除客户端证书

除了群集证书以外，还可以添加客户端证书用于在 Service Fabric 群集上执行管理操作。

可以添加两种类型的客户端证书 - 管理证书或只读证书。 然后，可以使用这些证书在群集上控制对管理操作和查询操作的访问。 默认情况下，群集证书将添加到允许的管理证书列表。

可以指定任意数量的客户端证书。 每次执行添加/删除操作都会更新 Service Fabric 群集的配置


### <a name="adding-client-certificates---admin-or-read-only-via-portal"></a>通过门户添加管理或只读客户端证书

1. 导航到“安全”边栏选项卡，然后选择顶部的“+ 身份验证”按钮。
2. 在“添加身份验证”边栏选项卡中选择“身份验证类型”-“只读客户端”或“管理客户端”
3. 现在选择授权方法。 向 Service Fabric 指出是要使用使用者名称还是指纹来查找此证书。 一般来说，使用使用者名称授权方法并不是一种良好的安全做法。 

![添加客户端证书][Add_Client_Cert]

### <a name="deletion-of-client-certificates---admin-or-read-only-using-the-portal"></a>使用门户删除管理或只读客户端证书

若要删除群集安全性使用的辅助证书，请导航到“安全”边栏选项卡，然后从特定证书的上下文菜单中选择“删除”选项。



## <a name="next-steps"></a>后续步骤
有关群集管理的详细信息，请阅读以下文章：

* [Service Fabric 群集升级过程和用户预期](/documentation/articles/service-fabric-cluster-upgrade/)
* [为客户端设置基于角色的访问](/documentation/articles/service-fabric-cluster-security-roles/)

<!--Image references-->
[Delete_Swap_Cert]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_09.PNG
[Add_Client_Cert]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_13.PNG
[Json_Pub_Setting1]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_14.PNG
[Json_Pub_Setting2]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_15.PNG
[Json_Pub_Setting3]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_16.PNG
[Json_Pub_Setting4]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_17.PNG
[Json_Pub_Setting5]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_18.PNG
<!--Update_Description: wording update;add anchors to sub titles-->