<properties
	pageTitle="使用 MSDeploy、主机名和 SSL 证书部署 Web 应用"
	description="通过使用 MSDeploy 并设置自定义主机名和 SSL 证书，在 Azure 资源管理器模板中部署 Web 应用"
	services="app-service\web"
	documentationCenter=""
	authors="jodehavi"
	/>

<tags
	ms.service="app-service-web"
	ms.date="02/26/2016"
	wacn.date="04/26/2016"/>

# 使用 MSDeploy、自定义主机名和 SSL 证书部署 Web 应用

本指南逐步说明如何通过利用 MSDeploy，以及将自定义主机名和 SSL 证书添加到 ARM 模板，来创建 Azure Web 应用的端到端部署。

有关创建模板的详细信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。

###创建示例应用程序

你将部署一个 ASP.NET Web 应用程序。第一步是创建简单的 Web 应用程序（或者，可以选择使用现有的应用程序 - 在这种情况下，可以跳过此步骤）。

打开 Visual Studio 2015，然后选择“文件”>“新建项目”。在出现的对话框中，选择“Web”>“ASP.NET Web 应用程序”。在“模板”下选择“Web”，然后选择 MVC 模板。在“更改身份验证类型”中，选择“无身份验证”。这是为了尽量简化示例应用程序。

此时，你将获得一个可随时在部署过程中使用的基本 ASP.Net Web 应用。

###创建 MSDeploy 包

下一步是创建用于将 Web 应用部署到 Azure 的包。为此，请保存你的项目，然后从命令行运行以下命令：

	msbuild yourwebapp.csproj /t:Package /p:PackageLocation="path\to\package.zip"

这将在 PackageLocation 文件夹下创建一个压缩包。现在，应用程序可供部署，你可以构建 Azure 资源管理器模板来执行此操作。

###创建 ARM 模板
首先，让我们创建一个基本的 ARM 模板，以便创建 Web 应用程序和托管计划（请注意，为求简洁，此处并未显示参数和变量）。

	{
		"name": "[parameters('appServicePlanName')]",
		"type": "Microsoft.Web/serverfarms",
		"location": "[resourceGroup().location]",
		"apiVersion": "2014-06-01",
		"dependsOn": [ ],
		"tags": {
		    "displayName": "appServicePlan"
		},
		"properties": {
		    "name": "[parameters('appServicePlanName')]",
		    "sku": "[parameters('appServicePlanSKU')]",
		    "workerSize": "[parameters('appServicePlanWorkerSize')]",
		    "numberOfWorkers": 1
		}
	},
	{
		"name": "[variables('webAppName')]",
		"type": "Microsoft.Web/sites",
		"location": "[resourceGroup().location]",
		"apiVersion": "2015-08-01",
		"dependsOn": [
		    "[concat('Microsoft.Web/serverfarms/', parameters('appServicePlanName'))]"
		],
		"tags": {
		    "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', parameters('appServicePlanName'))]": "Resource",
		    "displayName": "webApp"
		},
		"properties": {
		    "name": "[variables('webAppName')]",
		    "serverFarmId": "[resourceId('Microsoft.Web/serverfarms/', parameters('appServicePlanName'))]"
		}
	}

接下来，需要修改 Web 应用资源，以使用嵌套的 MSDeploy 资源。这样便可以引用前面创建的包，并指示 Azure 资源管理器使用 MSDeploy 将包部署到 Azure WebApp。下面显示了包含嵌套 MSDeploy 资源的 Microsoft.Web/sites 资源：

    {
        "name": "[variables('webAppName')]",
        "type": "Microsoft.Web/sites",
        "location": "[resourceGroup().location]",
        "apiVersion": "2015-08-01",
        "dependsOn": [
            "[concat('Microsoft.Web/serverfarms/', parameters('appServicePlanName'))]"
        ],
        "tags": {
            "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', parameters('appServicePlanName'))]": "Resource",
            "displayName": "webApp"
        },
        "properties": {
            "name": "[variables('webAppName')]",
            "serverFarmId": "[resourceId('Microsoft.Web/serverfarms/', parameters('appServicePlanName'))]"
        },
        "resources": [
            {
                "name": "MSDeploy",
                "type": "extensions",
                "location": "[resourceGroup().location]",
                "apiVersion": "2015-08-01",
                "dependsOn": [
                    "[concat('Microsoft.Web/sites/', variables('webAppName'))]"
                ],
                "tags": {
                    "displayName": "webDeploy"
                },
                "properties": {
                    "packageUri": "[concat(parameters('_artifactsLocation'), '/', parameters('webDeployPackageFolder'), '/', parameters('webDeployPackageFileName'), parameters('_artifactsLocationSasToken'))]",
                    "dbType": "None",
                    "connectionString": "",
                    "setParameters": {
                        "IIS Web Application Name": "[variables('webAppName')]"
                    }
                }
            }
        ]
    }

现在，你会注意到 MSDeploy 资源使用了如下定义的 **packageUri** 属性：

	"packageUri": "[concat(parameters('_artifactsLocation'), '/', parameters('webDeployPackageFolder'), '/', parameters('webDeployPackageFileName'), parameters('_artifactsLocationSasToken'))]"

此 **packageUri** 使用的存储帐户 URI 指向包 zip 所要上载到的存储帐户。当你部署模板时，Azure 资源管理器会利用[共享访问签名](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)从存储帐户将包提取到本地。此过程将会通过一个 PowerShell 脚本自动化，该脚本将上载包，调用 Azure 管理 API 以创建所需的密钥，然后将这些密钥作为参数传入模板（ *\_artifactsLocation* 和 *\_artifactsLocationSasToken* ）。你需要针对存储容器下包所上载到的文件夹和文件名定义参数。

接下来，需要加入其他嵌套资源，以设置主机名绑定来利用自定义域。首先，需要确保拥有该主机名，然后对它进行设置，使 Azure 能够验证你是否拥有它 - 请参阅[在 Azure Web 应用中配置自定义域名](/documentation/articles/web-sites-custom-domain-name/)。完成此操作后，可以将以下内容添加到模板中的 Microsoft.Web/sites 资源部分下：

	{
		"apiVersion": "2015-08-01",
		"type": "hostNameBindings",
		"name": "www.yourcustomdomain.com",
		"dependsOn": [
		    "[concat('Microsoft.Web/sites/', variables('webAppName'))]"
		],
		"properties": {
		    "domainId": null,
		    "hostNameType": "Verified",
		    "siteName": "variables('webAppName')"
		}
	}

最后，需要添加另一个顶级资源，即 Microsoft.Web/certificates。此资源将包含你的 SSL 证书，并与 Web 应用和托管计划位于同一级别。

	{
	    "name": "[parameters('certificateName')]",
	    "apiVersion": "2014-04-01",
	    "type": "Microsoft.Web/certificates",
	    "location": "[resourceGroup().location]",
	    "properties": {
	        "pfxBlob": "pfx base64 blob",
	        "password": "some pass"
	    }
	}

需要具备有效的 SSL 证书才能设置此资源。获得有效的证书后，需要提取 base64 字符串形式的 pfx 字节。提取此信息的方法之一是使用以下 PowerShell 命令：

	$fileContentBytes = get-content 'C:\path\to\cert.pfx' -Encoding Byte

	[System.Convert]::ToBase64String($fileContentBytes) | Out-File 'pfx-bytes.txt'

然后，可将此信息作为参数传递给 ARM 部署模板。

至此，ARM 模板已准备就绪。

###部署模板

最后一步是将上述所有组成部分整合为完整的端到端部署。若要简化部署，可以利用你在 Visual Studio 中创建 Azure 资源组项目时添加的 **Deploy-AzureResourceGroup.ps1 PowerShell** 脚本来上载模板中所需的任何项目。为此，必须事先创建你要使用的存储帐户。在本示例中，我已针对要上载的 package.zip 创建了一个共享存储帐户。脚本将利用 AzCopy 将该包上载到存储帐户。在传入项目文件夹位置后，脚本会自动将该目录中的所有文件上载到指定的存储容器。调用 Deploy-AzureResourceGroup.ps1 之后，必须更新 SSL 绑定，以映射自定义主机名与 SSL 证书。

以下 PowerShell 显示了调用 Deploy-AzureResourceGroup.ps1 的完整部署：

	#Set resource group name
	$rgName = "Name-of-resource-group"

	#call deploy-azureresourcegroup script to deploy web app

	.\Deploy-AzureResourceGroup.ps1 -ResourceGroupLocation "China East" `
									-ResourceGroupName $rgName `
									-UploadArtifacts "container-name" `
									-StorageAccountName "name-of-storage-acct-for-package" `
									-StorageAccountResourceGroupName "resource-group-name-storage-acct" `
									-TemplateFile "web-app-deploy.json" `
									-TemplateParametersFile "web-app-deploy-parameters.json" `
									-ArtifactStagingDirectory "C:\path\to\packagefolder"

	#update web app to bind ssl certificate to hostname. This has to be done after creation above.

	$cert = Get-PfxCertificate -FilePath C:\path\to\certificate.pfx

	$ar = Get-AzureRmResource -Name nameofwebsite -ResourceGroupName $rgName -ResourceType Microsoft.Web/sites -ApiVersion 2014-11-01

	$props = $ar.Properties

	$props.HostNameSslStates[2].'SslState' = 1
	$props.HostNameSslStates[2].'thumbprint' = $cert.Thumbprint
	$props.hostNameSslStates[2].'toUpdate' = $true

	Set-AzureRmResource -ApiVersion 2014-11-01 -Name nameofwebsite -ResourceGroupName $rgName -ResourceType Microsoft.Web/sites -PropertyObject $props

此时，应用程序应已部署，你可以通过 https://www.yourcustomdomain.com 浏览它

<!---HONumber=Mooncake_0215_2016-->