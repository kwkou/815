<!-- Remove script center in ACOM -->
<properties
   pageTitle="使用 Windows PowerShell 脚本发布到开发和测试环境 | Azure"
   description="了解如何使用 Windows PowerShell 脚本通过 Visual Studio 发布到开发和测试环境。"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />

<tags
    ms.assetid="5fff1301-5469-4d97-be88-c85c30f837c1"
    ms.service="multiple"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="multiple"
    ms.date="11/11/2016"
    wacn.date="02/04/2017"
    ms.author="tarcher" />

# 使用 Windows PowerShell 脚本发布到开发和测试环境

在 Visual Studio 中创建 Web 应用程序时，可以生成一个 Windows PowerShell 脚本，以后，你可以使用该脚本自动将你的网站以 Azure App Service 或虚拟机中 Web 应用的形式发布到 Azure。你可以根据需要在 Visual Studio 编辑器中编辑和扩展该 Windows PowerShell 脚本，或者将该脚本与现有的生成、测试和发布脚本相集成。

使用这些脚本，你可以设置站点的自定义版本（又称为开发与测试环境）供临时使用。例如，可以在 Azure 虚拟机中或者 Azure 网站的过渡槽中设置网站的特定版本，以运行测试套件、再现 bug、测试 bug 修复程序、试验建议的更改，或者设置自定义环境用于演示或展示。创建用于发布项目的脚本后，可以根据需要通过重新运行该脚本来重新创建相同的环境，或者结合你自己的 Web 应用程序版本运行该脚本，以创建自定义环境用于测试。

## 需要什么

- Azure SDK 2.3 或更高版本。有关详细信息，请参阅 [Visual Studio Downloads（Visual Studio 下载）](http://go.microsoft.com/fwlink/?LinkID=624384)。

无需使用 Azure SDK 就能为 Web 项目生成脚本。此功能适用于 Web 项目，而不适用于云服务中的 Web 角色。

- Azure PowerShell 0.7.4 或更高版本。有关详细信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。

- [Windows PowerShell 3.0](http://go.microsoft.com/?linkid=9811175) 或更高版本。

## 其他工具

我们还提供其他工具和资源，用于在 Visual Studio 中通过 PowerShell 进行 Azure 开发。请参阅 [PowerShell Tools for Visual Studio（适用于 Visual Studio 的 PowerShell 工具）](http://go.microsoft.com/fwlink/?LinkId=404012)。

## 生成发布脚本

你可以遵照[这些说明](/documentation/articles/virtual-machines-windows-classic-web-app-visual-studio/)，在创建新项目时为托管网站的虚拟机生成发布脚本。还可以[为 Azure App Service 中的 Web 应用生成发布脚本](/documentation/articles/web-sites-dotnet-get-started/)。

## Visual Studio 生成的脚本

Visual Studio 将生成名为 **PublishScripts** 的解决方案级文件夹，其中包含两个 Windows PowerShell 文件、一个针对虚拟机或网站的发布脚本，以及一个包含你要在脚本中使用的函数的模块。Visual Studio 还将生成 JSON 格式的文件，用于指定你要部署的项目的详细信息。

### Windows PowerShell 发布脚本

发布脚本包含有关部署到网站或虚拟机的特定发布步骤。针对 Windows PowerShell 开发，Visual Studio 提供语法颜色设置。其中还提供了函数的帮助，并且你可以自由编辑脚本中的函数，以满足不断变化的要求。

### Windows PowerShell 模块

Visual Studio 生成的 Windows PowerShell 模块包含发布脚本使用的函数。不应修改这些 Azure PowerShell 函数。有关详细信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。

### JSON 配置文件

JSON 文件是在 **Configurations** 文件夹中创建的，其中包含的配置数据用于确切指定要将哪些资源部署到 Azure。Visual Studio 生成的文件的名称为 project-name-WAWS-dev.json（如果创建的是网站），或 project name-VM-dev.json（如果创建的是虚拟机）。以下是当你创建网站时生成的 JSON 配置文件的示例。大多数值的含义都一目了然。网站名称由 Azure 生成，因此，它可能与你的项目名称不匹配。

    {
    "environmentSettings": {
    "webSite": {
    "name": "WebApplication26632",
    "location": "China East"
    },
    "databases": [
    {
    "connectionStringName": "DefaultConnection",
    "databaseName": "WebApplication26632_db",
    "serverName": "YourDatabaseServerName",
    "user": "sqluser2",
    "password": "",
    "edition": "",
    "size": "",
    "collation": "",
    "location": "China East"
    }
    ]
    }
    }

当你创建虚拟机时，JSON 配置文件类似于下面所示。请注意，创建的云服务用作虚拟机的容器。虚拟机包含通过 HTTP 和 HTTPS 进行 Web 访问时使用的普通终结点，以及用于 Web 部署的终结点，你可以从本地计算机、远程桌面和 Windows PowerShell 通过这些终结点发布到网站。

    {
    "environmentSettings": {
    "cloudService": {
    "name": "myusernamevm1",
    "affinityGroup": "",
    "location": "China East",
    "virtualNetwork": "",
    "subnet": "",
    "availabilitySet": "",
    "virtualMachine": {
    "name": "myusernamevm1",
    "vhdImage": "a699494373c04fc0bc8f2bb1389d6106__Win2K8R2SP1-Datacenter-201403.01-en.us-127GB.vhd",
    "size": "Small",
    "user": "vmuser1",
    "password": "",
    "enableWebDeployExtension": true,
    "endpoints": [
    {
    "name": "Http",
    "protocol": "TCP",
    "publicPort": "80",
    "privatePort": "80"
    },
    {
    "name": "Https",
    "protocol": "TCP",
    "publicPort": "443",
    "privatePort": "443"
    },
    {
    "name": "WebDeploy",
    "protocol": "TCP",
    "publicPort": "8172",
    "privatePort": "8172"
    },
    {
    "name": "Remote Desktop",
    "protocol": "TCP",
    "publicPort": "3389",
    "privatePort": "3389"
    },
    {
    "name": "Powershell",
    "protocol": "TCP",
    "publicPort": "5986",
    "privatePort": "5986"
    }
    ]
    }
    },
    "databases": [
    {
    "connectionStringName": "",
    "databaseName": "",
    "serverName": "",
    "user": "",
    "password": ""
    }
    ],
    "webDeployParameters": {
    "iisWebApplicationName": "Default Web Site"
    }
    }
    }

你可以编辑 JSON 配置，以更改运行发布脚本时的行为。`cloudService` 和 `virtualMachine` 节是必需的，但是，如果你不需要 `databases` 节，则可以将它删除。在 Visual Studio 生成的默认配置文件中为空的属性是可选的；在默认配置文件中具有值的属性是必需的。

如果你的网站具有多个部署环境（称为槽），而并非仅在 Azure 中有单个生产站点，则可将槽位名称包括在在 JSON 配置文件的网站名称中。例如，如果你的网站的名称为 **mysite**，该网站的一个槽的名称为 **test**，则 URI 为 mysite-test.chinacloudapp.cn，但在配置文件中使用的正确名称为 mysite(test)。只有当网站和槽已在订阅中存在时，才能这样做。如果它们不存在，请运行脚本而不指定槽位，从而创建网站，然后在 Azure 经典管理门户中创建槽位，从而使用未修改的网站名称来运行脚本。有关网站的部署槽的详细信息，请参阅 [Set up staging environments for web sites in Azure App Service（为 Azure App Service 中的网站设置过渡环境）](/documentation/articles/web-sites-staged-publishing/)。

## 如何运行发布脚本

如果你以前从未运行过 Windows PowerShell 脚本，则首先必须设置执行策略以启用要运行的脚本。这是一项安全功能，旨在为执行 Windows PowerShell 脚本时容易受到恶意软件或病毒攻击的用户提供保护。

### 运行脚本

1. 为项目创建 Web 部署包。Web 部署包是一个压缩的存档（.zip 文件），包含你要复制到网站或虚拟机的文件。可以在 Visual Studio 中为任何 Web 应用程序创建 Web 部署包。

![创建 Web 部署包](./media/vs-azure-tools-publishing-using-powershell-scripts/IC767885.png)

有关详细信息，请参阅 [How to: Create a Web Deployment Package in Visual Studio（如何：在 Visual Studio 中创建 Web 部署包）](https://msdn.microsoft.com/zh-cn/library/dd465323.aspx)。你也可根据本主题后面的**自定义和扩展发布脚本**中所述自动创建 Web 部署包。

1. 在“解决方案资源管理器”中打开脚本的上下文菜单，然后选择“使用 PowerShell ISE 打开”。

1. 如果你是第一次在此计算机上运行 Windows PowerShell 脚本，请使用管理员权限打开命令提示窗口并键入以下命令：

`Set-ExecutionPolicy RemoteSigned`

1. 使用以下命令登录到 Azure。

`Add-AzureAccount`

出现提示时，请提供你的用户名和密码。

请注意，当自动编写脚本时，这一提供 Azure 凭据的方法不起作用。应使用 .publishsettings 文件提供凭据。仅限一次使用 **Get-AzurePublishSettingsFile** 从 Azure 下载命令，此后则使用 **Import-AzurePublishSettingsFile** 导入该文件。有关详细说明，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/).

1. （可选）如果你希望创建虚拟机、数据库和网站等 Azure 资源，而不发布你的 Web 应用程序，请使用 **Publish-WebApplication.ps1** 命令，并结合设置为 JSON 配置文件的 **-Configuration** 参数。此命令行使用 JSON 配置文件来确定要创建的资源。由于它的其他命令行参数使用默认设置，因此它会创建资源，但不发布你的 Web 应用程序。-Verbose 选项可为你提供有关运行情况的更多信息。

        Publish-WebApplication.ps1 -Verbose –Configuration C:\Path\WebProject-WAWS-dev.json

1. 如以下示例之一所示，使用 **Publish-WebApplication.ps1** 命令可调用脚本并发布你的 Web 应用程序。如果需要覆盖其他任何参数（例如订阅名称、发布包名称、虚拟机凭据或数据库服务器凭据）的默认设置，可以指定这些参数。使用 **-Verbose** 选项可以查看有关发布进度的详细信息。


        Publish-WebApplication.ps1
        -Configuration C:\\Path\\WebProject-WAWS-dev-json `
        -SubscriptionName Contoso `
        -WebDeployPackage C:\\Documents\\Azure\\ADWebApp.zip `
        -DatabaseServerPassword @{Name="dbServerName";Password="adminPassword"} `
        -Verbose

如果你要创建虚拟机，则命令类似于以下形式：此示例还显示了如何为多个数据库指定凭据。对于这些脚本创建的虚拟机，SSL 证书不是来自受信任的根证书颁发机构。因此，需要使用 **-AllowUntrusted** 选项。


        Publish-WebApplication.ps1 `
        -Configuration C:\Path\ADVM-VM-test.json `
        -SubscriptionName Contoso `
        -WebDeployPackage C:\Path\ADVM.zip `
        -AllowUntrusted `
        -VMPassword @{name = "vmUserName"; password = "YourPasswordHere"} `
        -DatabaseServerPassword @{Name="server1";Password="adminPassword1"}, @{Name="server2";Password="adminPassword2"} `
        -Verbose


脚本可以创建数据库，但不会创建数据库服务器。如果你想要创建数据库服务器，可以使用 Azure 模块中的 **New-AzureSqlDatabaseServer** 函数。

## 自定义和扩展发布脚本

你可以自定义发布脚本和 JSON 配置文件。不应修改 Windows PowerShell 模块 **AzureWebAppPublishModule.psm1** 中的函数。如果你只是想要指定一个不同的数据库或更改虚拟机的某些属性，可以编辑 JSON 配置文件。如果想要扩展脚本的功能以自动生成和测试你的项目，可以在 **Publish-WebApplication.ps1** 中实现函数存根。

若要自动生成项目，请按照以下代码示例中所示，添加对 `New-WebDeployPackage` 调用 MSBuild 的代码。MSBuild 命令的路径各不相同，具体取决于你安装的 Visual Studio 版本。若要获取正确的路径，可以使用 **Get-MSBuildCmd** 函数，如本示例中所示。

### 自动生成项目

1. 在 global param 节中添加 `$ProjectFile` 参数。


        [Parameter(Mandatory = $false)]
        [ValidateScript({Test-Path $_ -PathType Leaf})]
        [String]
        $ProjectFile,

1. 将函数 `Get-MSBuildCmd` 复制到脚本文件。


        function Get-MSBuildCmd
        {
                process
        {
        
                     $path =  Get-ChildItem "HKLM:\SOFTWARE\Microsoft\MSBuild\ToolsVersions" |
                                           Sort-Object {[double]$_.PSChildName} -Descending |
                                           Select-Object -First 1 |
                                           Get-ItemProperty -Name MSBuildToolsPath |
                                           Select -ExpandProperty MSBuildToolsPath
               
                    $path = (Join-Path -Path $path -ChildPath 'msbuild.exe')
        
                return Get-Item $path
            }
        }


1. 将 `New-WebDeployPackage` 替换为以下代码，并替换构造 `$msbuildCmd` 的行中的占位符。此代码适用于 Visual Studio 2015。如果你使用的是 Visual Studio 2013，请将以下 **VisualStudioVersion** 属性更改为 `12.0`。

        function New-WebDeployPackage
        {
            #编写函数以生成 Web 应用程序并将其打包
              
        #若要生成 Web 应用程序，请使用 MsBuild.exe。有关帮助，请参阅以下网页中的 MSBuild 命令行参考：http://go.microsoft.com/fwlink/?LinkId=391339
              
        Write-VerboseWithTime 'Build-WebDeployPackage: Start'
              
        $msbuildCmd = '"{0}" "{1}" /T:Rebuild;Package /P:VisualStudioVersion=14.0 /p:OutputPath="{2}\\MSBuildOutputPath" /flp:logfile=msbuild.log,v=d' -f (Get-MSBuildCmd), $ProjectFile, $scriptDirectory
              
        Write-VerboseWithTime ('Build-WebDeployPackage: ' + $msbuildCmd)
              
        #开始执行生成命令
        $job = Start-Process cmd.exe -ArgumentList('/C "' + $msbuildCmd + '"') -WindowStyle Normal -Wait -PassThru
              
        if ($job.ExitCode -ne 0) { throw('MsBuild exited with an error.ExitCode:' + $job.ExitCode) }
        
        #获取项目名称
        $projectName = (Get-Item $ProjectFile).BaseName
              
        #构造 Web 部署 zip 包的路径
        $DeployPackageDir = '.\\MSBuildOutputPath\_PublishedWebsites{0}\_Package{0}.zip' -f $projectName
              
              
        #获取 Web 部署 zip 包的完整路径。这是使 MSDeploy 正常工作所必需的
        $WebDeployPackage = Resolve-Path –LiteralPath $DeployPackageDir
              
        Write-VerboseWithTime 'Build-WebDeployPackage: End'
              
        return $WebDeployPackage }

1. 在此行之前调用 `New-WebDeployPackage` 函数：`$Config = Read-ConfigFile $Configuration`（适用于 Web 应用）或 `$Config = Read-ConfigFile $Configuration -HasWebDeployPackage:([Bool]$WebDeployPackage)`（适用于虚拟机）。

        if($ProjectFile)
        {
        $WebDeployPackage = New-WebDeployPackage
        }

1. 从命令行通过传递 `$Project` 参数调用自定义脚本，如以下示例命令行所示。

        .\Publish-WebApplicationVM.ps1 -Configuration .\Configurations\WebApplication5-VM-dev.json `
        -ProjectFile ..\WebApplication5\WebApplication5.csproj `
        -VMPassword @{Name="VMUser";Password="Test.123"} `
        -AllowUntrusted `
        -Verbose

若要自动测试你的应用程序，请将代码添加到 `Test-WebApplication`。请务必取消注释 **Publish-WebApplication.ps1** 中调用这些函数的行。如果不提供实现，则可以使用 Visual Studio 手动生成项目，然后运行发布脚本来发布到 Azure。

## 发布函数摘要

若要获取可在 Windows PowerShell 命令提示符处使用的函数的相关帮助，请使用 `Get-Help function-name` 命令。帮助中包含参数帮助和示例。脚本源文件 **AzureWebAppPublishModule.psm1** 和 **Publish-WebApplication.ps1** 中也提供了相同的帮助文本。脚本和帮助已使用你的 Visual Studio 语言本地化。

**AzureWebAppPublishModule**

|函数名称|说明|
|---|---|
|Add-AzureSQLDatabase|创建新的 Azure SQL 数据库。|
|Add-AzureSQLDatabases|基于 Visual Studio 生成的 JSON 配置文件中的值创建 Azure SQL 数据库。|
|Add-AzureVM|创建 Azure 虚拟机并返回所部署 VM 的 URL。该函数将设置先决条件，然后调用 **New-AzureVM** 函数（Azure 模块）来创建新的虚拟机。|
|Add-AzureVMEndpoints|将新的输入终结点添加到虚拟机，然后返回包含新终结点的虚拟机。|
|Add-AzureVMStorage|在当前订阅中创建新的 Azure 存储帐户。该帐户的名称以“devtest”开头，后接一个唯一的字母数字字符串。该函数将返回新存储帐户的名称。你必须为新的存储帐户指定位置或地缘组。|
|Add-AzureWebsite|使用特定的名称和位置创建网站。此函数将调用 Azure 模块中的 **New-AzureWebsite** 函数。如果订阅尚未包含具有指定名称的网站，此函数将创建该网站并返回一个网站对象。否则，将返回 `$null`。|
|Backup-Subscription|将当前 Azure 订阅保存在脚本作用域中的 `$Script:originalSubscription` 变量内。此函数在脚本作用域中保存当前 Azure 订阅（通过 `Get-AzureSubscription -Current` 获取）及其存储帐户，以及此脚本更改的订阅（存储在变量 `$UserSpecifiedSubscription` 中）及其存储帐户。保存这些值后，如果当前状态发生更改，你可以使用 `Restore-Subscription` 等函数将当前原始订阅和存储帐户还原到当前状态。|
|Find-AzureVM|获取指定的 Azure 虚拟机。|
|Format-DevTestMessageWithTime|在消息的前面添加日期和时间。此函数适用于写入到错误流和详细流的消息。|
|Get-AzureSQLDatabaseConnectionString|汇编一个连接字符串以连接到 Azure SQL 数据库。|
|Get-AzureVMStorage|返回指定的位置或地缘组中名称模式为“devtest”（不区分大小写）的第一个存储帐户的名称。如果“devtest”存储帐户与该位置或地缘组不匹配，该函数将忽略该帐户。你必须指定一个位置或地缘组。|
|Get-MSDeployCmd|返回一个用于运行 MsDeploy.exe 工具的命令。|
|New-AzureVMEnvironment|在订阅中查找或创建与 JSON 配置文件中的值匹配的虚拟机。|
|Publish-WebPackage|使用 MsDeploy.exe 和 Web 发布包 .Zip 文件将资源部署到网站。此函数不生成任何输出。如果调用 MSDeploy.exe 失败，该函数将引发异常。若要获取更详细的输出，请使用 **-Verbose** 选项。|
|Publish-WebPackageToVM|验证参数值，然后调用 **Publish-WebPackage** 函数。|
|Read-ConfigFile|验证 JSON 配置文件并返回选定值的哈希表。|
|Restore-Subscription|将当前订阅重置为原始订阅。|
|Test-AzureModule|如果安装的 Azure 模块版本为 0.7.4 或更高，则返回 `$true`。如果该模块未安装或者使用了更低的版本，则返回 `$false`。此函数没有参数。|
|Test-AzureModuleVersion|如果 Azure 模块的版本为 0.7.4 或更高，则返回 `$true`。如果该模块未安装或者使用了更低的版本，则返回 `$false`。此函数没有参数。|
|Test-HttpsUrl|将输入 URL 转换为 System.Uri 对象。如果 URL 是绝对的并且其方案为 https，则返回 `$True`。如果 URL 是相对的并且其方案不是 HTTPS，或者无法将输入字符串转换为 URL，则返回 `$false`。|
|Test-Member|如果某个属性或方法是对象的成员，则返回 `$true`。否则返回 `$false`。|
|Write-ErrorWithTime|写入以当前时间作为前缀的错误消息。将该消息写入错误流之前，此函数将调用 **Format-DevTestMessageWithTime** 函数来添加时间前缀。|
|Write-HostWithTime|将以当前时间作为前缀的消息写入主机程序 (**Write-Host**)。写入主机程序的效果各不相同。大多数托管 Windows PowerShell 的程序会将这些消息写入标准输出。|
|Write-VerboseWithTime|写入以当前时间作为前缀的详细消息。由于该函数调用 **Write-Verbose**，因此，仅当使用 **Verbose** 参数运行脚本，或者将 **VerbosePreference** 首选项设置为 **Continue** 时，才显示该消息。|

**Publish-WebApplication**

|函数名称|说明|
|---|---|
|New-AzureWebApplicationEnvironment|创建 Azure 资源，例如网站或虚拟机。|
|New-WebDeployPackage|未实现此函数。你可以在此函数中添加命令以生成项目。|
|Publish-AzureWebApplication|将 Web 应用程序发布到 Azure。|
|Publish-WebApplication|为 Visual Studio Web 项目创建和部署 Web Apps、虚拟机、SQL 数据库和存储帐户。|
|Test-WebApplication|未实现此函数。你可以在此函数中添加命令以测试应用程序。|

## 后续步骤

请阅读 [Scripting with Windows PowerShell（使用 Windows PowerShell 编写脚本）](https://technet.microsoft.com/zh-cn/library/bb978526.aspx)以详细了解 PowerShell 脚本功能。

<!---HONumber=Mooncake_0620_2016-->