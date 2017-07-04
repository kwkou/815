<properties
    pageTitle="Azure PowerShell 脚本示例 - 使用 FTP 将文件上载到 Web 应用 | Azure"
    description="Azure PowerShell 脚本示例 - 使用 FTP 将文件上载到 Web 应用"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="b7d46d6f-44fd-454c-8008-87dab6eefbc1"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="79cf3bd7cb4c59b1a57195a629d87d1351fcce99"
    ms.lasthandoff="04/14/2017" />

# <a name="upload-files-to-a-web-app-using-ftp"></a>使用 FTP 将文件上载到 Web 应用

此示例脚本使用其相关资源，在应用服务中创建 Web 应用，然后使用 FTP（通过 [WebClient.UploadFile()](https://msdn.microsoft.com/zh-cn/library/ms144229.aspx)）部署 Web 应用代码。

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)中的说明安装 Azure PowerShell，然后运行 `Login-AzureRmAccount -EnvironmentName AzureChinaCloud` 创建与 Azure 的连接。

## <a name="sample-script"></a>示例脚本

    $appdirectory="<Replace with your app directory>"
    $webappname="mywebapp$(Get-Random)"
    $location="China North"

    # Create a resource group.
    New-AzureRmResourceGroup -Name myResourceGroup -Location $location

    # Create an App Service plan in `Free` tier.
    New-AzureRmAppServicePlan -Name $webappname -Location $location `
    -ResourceGroupName myResourceGroup -Tier Free

    # Create a web app.
    New-AzureRmWebApp -Name $webappname -Location $location -AppServicePlan $webappname `
    -ResourceGroupName myResourceGroup

    # Get publishing profile for the web app
    $xml = (Get-AzureRmWebAppPublishingProfile -Name $webappname `
    -ResourceGroupName myResourceGroup `
    -OutputFile null)

    # Extract connection information from publishing profile
    $username = $xml.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@userName").value
    $password = $xml.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@userPWD").value
    $url = $xml.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@publishUrl").value

    # Upload files recursively 
    Set-Location $appdirectory
    $webclient = New-Object -TypeName System.Net.WebClient
    $webclient.Credentials = New-Object System.Net.NetworkCredential($username,$password)
    $files = Get-ChildItem -Path $appdirectory -Recurse | Where-Object{!($_.PSIsContainer)}
    foreach ($file in $files)
    {
        $relativepath = (Resolve-Path -Path $file.FullName -Relative).Replace(".\", "").Replace('\', '/')
        $uri = New-Object System.Uri("$url/$relativepath")
        "Uploading to " + $uri.AbsoluteUri
        $webclient.UploadFile($uri, $file.FullName)
    } 
    $webclient.Dispose()

## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用以下命令删除资源组、Web 应用以及所有相关资源。

    Remove-AzureRmResourceGroup -Name $webappname -Force

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Resources/v3.5.0/new-azurermresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzureRmAppServicePlan](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermappserviceplan) | 创建应用服务计划。 |
| [New-AzureRmWebApp](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermwebapp) | 创建 Web 应用。 |
| [Get-AzureRmWebAppPublishingProfile](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/get-azurermwebapppublishingprofile) | 获取 Web 应用的发布配置文件。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

可以在 [Azure PowerShell 示例](/documentation/articles/app-service-powershell-samples/)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。