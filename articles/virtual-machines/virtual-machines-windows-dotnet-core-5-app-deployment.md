<properties
    pageTitle="使用虚拟机扩展自动化应用程序部署 | Azure"
    description="Azure 虚拟机 DotNet Core 教程"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="79c91304-6c1b-4354-a185-fecc129b139d"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure-services"
    ms.date="11/21/2016"
    wacn.date="04/17/2017"
    ms.author="nepeters"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="2545cbde724ae26de9c63937f6fdd35aee3a203d"
    ms.lasthandoff="04/06/2017" />

# <a name="application-deployment-with-azure-resource-manager-templates-for-windows-vms"></a>适用于 Windows VM 的使用 Azure Resource Manager 模板的应用程序部署

确定所有 Azure 基础结构要求并转换成部署模板后，需要解决实际的应用程序部署需求。 此处的应用程序部署是指将实际应用程序二进制文件安装到 Azure 资源上。 以音乐应用商店示例而言，需要在每个虚拟机上安装并配置 .Net Core 和 IIS。 需要将音乐应用商店二进制文件安装到虚拟机上，并预先创建音乐应用商店数据库。

本文档详细说明虚拟机扩展如何自动化 Azure 虚拟机的应用程序部署和配置。 所有依赖项和独特配置都已突出显示。 为了获得最佳体验，请将一个解决方案实例预先部署到 Azure 订阅，然后将它与 Azure Resource Manager 模板配合运行。 可以在 [Windows 上的音乐应用商店部署](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows)中找到完整模板。

## <a name="configuration-script"></a>配置脚本
虚拟机扩展是针对虚拟机执行的专用程序，可提供配置自动化。 许多特定的目的（例如防病毒、日志记录配置和 Docker 配置）都有相应的扩展。 使用自定义脚本扩展可对虚拟机运行任何脚本。 在音乐应用商店示例中，需依赖自定义脚本扩展来配置 Windows 虚拟机以及安装音乐应用商店应用程序。

在详细说明如何在 Azure Resource Manager 模板中声明虚拟机扩展之前，我们先了解所要运行的脚本。 此脚本配置用于托管音乐应用商店应用程序的 Windows 虚拟机。 在运行时，该脚本将安装全部所需的软件、从源代码管理安装音乐应用商店应用程序，以及准备数据库。 

> 此示例用于演示目的。

    <#
        .SYNOPSIS
            Downloads and configures .Net Core Music Store application sample across IIS and Azure SQL DB.
    #>

    Param (
        [string]$user,
        [string]$password,
        [string]$sqlserver
    )

    # Firewall
    netsh advfirewall firewall add rule name="http" dir=in action=allow protocol=TCP localport=80

    # Folders
    New-Item -ItemType Directory c:\temp
    New-Item -ItemType Directory c:\music

    # Install iis
    Install-WindowsFeature web-server -IncludeManagementTools

    # Install dot.net core sdk
    Invoke-WebRequest http://go.microsoft.com/fwlink/?LinkID=615460 -outfile c:\temp\vc_redistx64.exe
    Start-Process c:\temp\vc_redistx64.exe -ArgumentList '/quiet' -Wait
    Invoke-WebRequest https://go.microsoft.com/fwlink/?LinkID=809122 -outfile c:\temp\DotNetCore.1.0.0-SDK.Preview2-x64.exe
    Start-Process c:\temp\DotNetCore.1.0.0-SDK.Preview2-x64.exe -ArgumentList '/quiet' -Wait
    Invoke-WebRequest https://go.microsoft.com/fwlink/?LinkId=817246 -outfile c:\temp\DotNetCore.WindowsHosting.exe
    Start-Process c:\temp\DotNetCore.WindowsHosting.exe -ArgumentList '/quiet' -Wait

    # Download music app
    Invoke-WebRequest  https://github.com/Microsoft/dotnet-core-sample-templates/raw/master/dotnet-core-music-windows/music-app/music-store-azure-demo-pub.zip -OutFile c:\temp\musicstore.zip
    Expand-Archive C:\temp\musicstore.zip c:\music

    # Azure SQL connection sting in environment variable
    [Environment]::SetEnvironmentVariable("Data:DefaultConnection:ConnectionString", "Server=$sqlserver;Database=MusicStore;Integrated Security=False;User Id=$user;Password=$password;MultipleActiveResultSets=True;Connect Timeout=30",[EnvironmentVariableTarget]::Machine)

    # Pre-create database
    $env:Data:DefaultConnection:ConnectionString = "Server=$sqlserver;Database=MusicStore;Integrated Security=False;User Id=$user;Password=$password;MultipleActiveResultSets=True;Connect Timeout=30"
    Start-Process 'C:\Program Files\dotnet\dotnet.exe' -ArgumentList 'c:\music\MusicStore.dll'

    # Configure iis
    Remove-WebSite -Name "Default Web Site"
    Set-ItemProperty IIS:\AppPools\DefaultAppPool\ managedRuntimeVersion ""
    New-Website -Name "MusicStore" -Port 80 -PhysicalPath C:\music\ -ApplicationPool DefaultAppPool
    & iisreset

## <a name="vm-script-extension"></a>VM 脚本扩展
通过将扩展资源包含在 Azure Resource Manager 模板中，可在构建时对虚拟机运行 VM 扩展。 可以通过使用 Visual Studio 中的“添加新资源”向导或者在模板中插入有效的 JSON 来添加该扩展。 脚本扩展资源嵌套在虚拟机资源中，如以下示例中所示。

单击以下链接可查看 Resource Manager 模板中的 JSON 示例 – [VM 脚本扩展](https://github.com/Microsoft/dotnet-core-sample-templates/blob/master/dotnet-core-music-windows/azuredeploy.json#L339)。 

>[AZURE.NOTE]
> 必须修改下载的模板，以适应 Azure 中国云环境。 例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”，将“database.windows.net”替换为“database.chinacloudapi.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

请注意，在以下 JSON 中，脚本存储在 GitHub 中。 此脚本也可以存储在 Azure Blob 存储中。 此外，Azure Resource Manager 模板允许构建脚本执行字符串，使模板参数值可用作脚本执行参数。 在本例中，数据是在部署模板时提供的，然后，可在执行脚本时使用这些值。

    {
      "apiVersion": "2015-06-15",
      "type": "extensions",
      "name": "config-app",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
        "[variables('musicstoresqlName')]"
      ],
      "tags": {
        "displayName": "config-app"
      },
      "properties": {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.4",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "fileUris": [
            "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
          ]
        },
        "protectedSettings": {
          "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.chinacloudapi.cn')]"
        }
      }
    }

如上所述，还可将自定义脚本存储在 Azure Blob 存储中。 可通过两种方式将脚本资源存储在 blob 存储中：公开容器/脚本并遵循上述方法，或将其保存在专用 blob 存储中；后者需要向 CustomScriptExtension 资源定义提供 storageAccountName 和 storageAccountKey。

在下例中，进一步执行了操作。 虽然可在部署期间将存储帐户名称和密钥提供为参数或变量，但 Resource Manager 模板提供了 `listKeys` 函数；该函数可按编程方式获取存储帐户密钥并在部署时将其插入到模板中。

在下面的 CustomScriptExtension 资源定义示例中，自定义脚本已上传到另一个 `mysa999rgname` 资源组中名为 `mystorageaccount9999` 的 Azure 存储帐户。 在部署包含此资源的模板时，`listKeys` 函数会按编程方式获取资源组 `mysa999rgname` 中的存储帐户 `mystorageaccount9999` 的存储帐户密钥，并将其代为插入到模板中。

    {
      "apiVersion": "2015-06-15",
      "type": "extensions",
      "name": "config-app",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
        "[variables('musicstoresqlName')]"
      ],
      "tags": {
        "displayName": "config-app"
      },
      "properties": {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.7",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "fileUris": [
            "https://mystorageaccount9999.blob.core.chinacloudapi.cn/container/configure-music-app.ps1"
          ]
        },
        "protectedSettings": {
          "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.chinacloudapi.cn')]",
          "storageAccountName": "mystorageaccount9999",
          "storageAccountKey": "[listKeys(resourceId('mysa999rgname','Microsoft.Storage/storageAccounts', mystorageaccount9999), '2015-06-15').key1]"
        }
      }
    }

此方法的主要优点是，在存储帐户密钥更改的情况下，无需更改模板和部署参数。

有关使用自定义脚本扩展的详细信息，请参阅[使用 Resource Manager 模板自定义脚本扩展](/documentation/articles/virtual-machines-windows-extensions-customscript/)。

## <a name="next-step"></a>后续步骤
<hr>

[探索更多 Azure Resource Manager 模板](https://github.com/Azure/azure-quickstart-templates)
<!--Update_Description: adding customScriptExtension Resource-->