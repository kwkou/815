<properties
	pageTitle="创建 Web 角色和辅助角色"
	description="如何在 Azure 云服务中创建 PHP Web 和辅助角色以及配置 PHP 运行时的指南。"
	services=""
	documentationCenter="php"
	authors="tfitzmac"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="cloud-services"
	ms.date="06/09/2015"
	wacn.date="10/3/2015"/>

#如何创建 PHP Web 角色和辅助角色

## 概述

本指南将说明如何执行以下操作：在 Windows 开发环境中创建 PHP Web 角色或辅助角色，从提供的“内置”版本中选择特定版本的 PHP，更改 PHP 配置，启用扩展以及部署到 Azure。它还介绍了如何将 Web 角色或辅助角色配置为使用你提供的 PHP 运行时（带自定义配置和扩展）。

## 什么是 PHP Web 角色和辅助角色？
Azure 提供了三种计算模型以运行应用程序：[Azure 网站] [执行模型-网站]、[Azure 虚拟机][execution model-vms]和 [Azure 云服务][execution model-cloud services]。这三种模型都支持 PHP。云服务（包括 Web 角色和辅助角色）提供了*平台即服务 (PaaS)*。在云服务中，Web 角色提供专用的 Internet Information Services (IIS) Web 服务器来托管前端 Web 应用程序，而辅助角色可独立于用户交互或输入运行异步任务、运行时间较长的任务或永久性任务。

有关详细信息，请参阅[什么是云服务？]。

## 下载 Azure SDK for PHP

[Azure SDK for PHP] 由多个组件构成。本文将使用其中两个：Azure PowerShell 和 Azure 仿真程序。可在此处通过 Microsoft Web 平台安装程序安装这两个组件：[安装 Azure PowerShell 和 Azure 仿真程序][install ps and emulators]

## 如何：创建云服务项目

创建 PHP Web 角色或辅助角色的第一步是创建 Azure 服务项目。Azure 服务项目用作 Web 角色和辅助角色的逻辑容器，并包含项目的[服务定义 (.csdef)] 和[服务配置 (.cscfg)] 文件。

若要创建新的 Azure 服务项目，请以管理员身份运行 Azure PowerShell 并执行以下命令：

	PS C:\>New-AzureServiceProject myProject

此命令将创建可将 Web 角色和辅助角色添加到的新目录 (`myProject`)。

## 如何：添加 PHP Web 角色或辅助角色

若要将 PHP Web 角色添加到项目，请从项目的根目录中运行以下命令：

	PS C:\myProject> Add-AzurePHPWebRole roleName

对于辅助角色，请使用此命令：

	PS C:\myProject> Add-AzurePHPWorkerRole roleName

> [AZURE.NOTE]`roleName` 参数是可选的。如果省略该参数，则将自动生成角色名称。创建的第一个 Web 角色将为 `WebRole1`，第二个 Web 角色为 `WebRole2`，依此类推。创建的第一个辅助角色将为 `WorkerRole1`，第二个辅助角色为 `WorkerRole2`，依此类推。

## 如何：指定内置 PHP 版本

在将 PHP Web 角色或辅助角色添加到项目时，将修改项目的配置文件，以便在部署应用程序的每个 Web 实例或辅助进程实例时在其上安装 PHP。若要查看默认情况下安装的 PHP 的版本，请运行以下命令：

	PS C:\myProject> Get-AzureServiceProjectRoleRuntime

上述命令的输出与下图中所示类似。在此示例中，将 PHP 5.3.17 的 `IsDefault` 标志设置为 `true`，这指示它将是安装的默认 PHP 版本。

	Runtime Version		PackageUri						IsDefault
	------- ------- 	----------  					---------
   	Node 0.6.17      	http://nodertncu.blob.core...   False
   	Node 0.6.20         http://nodertncu.blob.core...   True
   	Node 0.8.4          http://nodertncu.blob.core...   False
	IISNode 0.1.21      http://nodertncu.blob.core...   True
  	Cache 1.8.0         http://nodertncu.blob.core...   True
    PHP 5.3.17          http://nodertncu.blob.core...   True
    PHP 5.4.0           http://nodertncu.blob.core...   False

可以将 PHP 运行时版本设置为列出的任意 PHP 版本。例如，若要将 PHP 版本（对于名为 `roleName` 的角色）设置为 5.4.0，请使用以下命令：

	PS C:\myProject> Set-AzureServiceProjectRole roleName php 5.4.0

> [AZURE.NOTE]将来可能会提供更多 PHP 版本，并且可用版本可能会发生更改。

## 如何：自定义内置 PHP 运行时

当按上述步骤进行操作时，您可以完全控制所安装的 PHP 运行时的配置，包括修改 `php.ini` 设置和启用扩展。

若要自定义内置 PHP 运行时，请执行下列步骤：

1. 将一个名为 `php` 的新文件夹添加到 Web 角色的 `bin` 目录。对于辅助角色，将该文件夹添加到角色的根目录。
2. 在 `php` 文件夹中，创建另一个名为 `ext` 的文件夹。将要启用的任何扩展名为 `.dll` 的文件（例如，`php_mongo.dll`）置于此文件夹中。
3. 将 `php.ini` 文件添加到 `php` 文件夹中。启用任何自定义扩展，并在此文件中设置任何 PHP 指令。例如，若要打开 `display_errors` 并启用 `php_mongo.dll` 扩展，则 `php.ini` 文件的内容将如下所示：

		display_errors=On
		extension=php_mongo.dll

> [AZURE.NOTE]所提供的 `php.ini` 文件中未显式设置的所有设置都将自动设为其默认值。但请记住，您可以添加整个 `php.ini` 文件。

## 如何：使用您自己的 PHP 运行时
在某些情况下，可能需要提供你自己的 PHP 运行时，而不是如上所述那样选择并配置内置 PHP 运行时。例如，可以在 Web 角色或辅助角色中使用你在开发环境中使用的 PHP 运行时，以便更轻松地确保应用程序不会更改生产环境中的行为。

### 将 Web 角色配置为使用你自己的 PHP 运行时

若要将 Web 角色配置为使用提供的 PHP 运行时，请执行下列步骤。

1. 按照以上[如何：创建云服务项目](#how-to-create-a-cloud-services-project)和[如何：添加 PHP Web 角色或辅助角色](#how-to-add-php-web-or-worker-roles)章节中所述创建 Azure 服务项目并添加 PHP web 角色。
2. 在位于 Web 角色的根目录中的 `bin` 文件夹中创建一个 `php` 文件夹，然后将 PHP 运行时（所有二进制文件、配置文件、子文件夹等）添加到该 `php` 文件夹中。
3. （可选）如果 PHP 运行时使用 [Microsoft Drivers for PHP for SQL Server][sqlsrv drivers]，则需要将 Web 角色配置为在 [SQL Server Native Client 2012][sql native client] 可用时安装它。为此，将 `sqlncli.msi` 安装程序添加到 Web 角色的根目录中的 `bin` 文件夹。您可以在此处下载该安装程序：[sqlncli.msi x64 安装程序]。下一步中所述的启动脚本将在设置角色时以静默方式运行安装程序。如果你的 PHP 运行时不使用 Microsoft Drivers for PHP for SQL Server，则可从下一步所示的脚本中删除以下行：

		msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

4. 下一步是定义将 [Internet Information Services (IIS)][iis.net] 配置为使用 PHP 运行时来处理 `.php` 页的请求的启动任务。为此，请在文本编辑器中打开 `setup_web.cmd` 文件（位于 Web 角色的根目录的 `bin` 文件夹中），并将其内容替换为以下脚本：

		@ECHO ON
		cd "%~dp0"

		if "%EMULATED%"=="true" exit /b 0

		msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

		SET PHP_FULL_PATH=%~dp0php\php-cgi.exe
		SET NEW_PATH=%PATH%;%RoleRoot%\base\x86

		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%',maxInstances='12',idleTimeout='60000',activityTimeout='3600',requestTimeout='60000',instanceMaxRequests='10000',protocol='NamedPipe',flushNamedPipe='False']" /commit:apphost
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%'].environmentVariables.[name='PATH',value='%NEW_PATH%']" /commit:apphost
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%'].environmentVariables.[name='PHP_FCGI_MAX_REQUESTS',value='10000']" /commit:apphost
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/handlers /+"[name='PHP',path='*.php',verb='GET,HEAD,POST',modules='FastCgiModule',scriptProcessor='%PHP_FULL_PATH%',resourceType='Either',requireAccess='Script']" /commit:apphost
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /"[fullPath='%PHP_FULL_PATH%'].queueLength:50000"

5. 将应用程序文件添加到 Web 角色的根目录。这将是 Web 服务器的根目录。

6. 按照以下[如何：发布应用程序](#how-to-publish-your-application)一节中所述发布您的应用程序。

> [AZURE.NOTE]在按照上述使用您自己的 PHP 运行时的步骤进行操作后，可以删除 `download.ps1` 脚本（位于 Web 角色的根目录的 `bin` 文件夹中）。

### 将辅助角色配置为使用你自己的 PHP 运行时

若要将辅助角色配置为使用提供的 PHP 运行时，请执行下列步骤。

1. 按照以上[如何：创建云服务项目](#how-to-create-a-cloud-services-project)和[如何：添加 PHP Web 角色或辅助角色](#how-to-add-php-web-or-worker-roles)章节中所述创建 Azure 服务项目并添加 PHP 辅助角色。
2. 在辅助角色的根目录中创建一个 `php` 文件夹，然后将 PHP 运行时（所有二进制文件、配置文件、子文件夹等）添加到该 `php` 文件夹中。
3. （可选）如果 PHP 运行时使用 [Microsoft Drivers for PHP for SQL Server][sqlsrv drivers]，则需要将辅助角色配置为在 [SQL Server Native Client 2012][sql native client] 可用时安装它。为此，将 `sqlncli.msi` 安装程序添加到辅助角色的根目录。您可以下载该安装程序：[sqlncli.msi x64 安装程序]。下一步中所述的启动脚本将在设置角色时以静默方式运行安装程序。如果你的 PHP 运行时不使用 Microsoft Drivers for PHP for SQL Server，则可从下一步所示的脚本中删除以下行：

		msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

4. 下一步是定义在设置角色时将 `php.exe` 可执行文件添加到辅助角色的 PATH 环境变量中的启动任务。为此，请在文本编辑器中打开 `setup_worker.cmd` 文件（位于辅助角色的根目录中），并将其内容替换为以下脚本：

		@echo on

		cd "%~dp0"

		echo Granting permissions for Network Service to the web root directory...
		icacls ..\ /grant "Network Service":(OI)(CI)W
		if %ERRORLEVEL% neq 0 goto error
		echo OK

		if "%EMULATED%"=="true" exit /b 0

		msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

		setx Path "%PATH%;%~dp0php" /M

		if %ERRORLEVEL% neq 0 goto error

		echo SUCCESS
		exit /b 0

		:error

		echo FAILED
		exit /b -1

5. 将应用程序文件添加到辅助角色的根目录。

6. 按照以下[如何：发布应用程序](#how-to-publish-your-application)一节中所述发布您的应用程序。

## 如何：在计算和存储模拟器中运行您的应用程序

Azure 计算和存储模拟器提供了一个本地环境，可在将 Azure 应用程序部署到云之前在该本地环境中测试此应用程序。模拟器与 Azure 环境之间存在一些差异。若要更好地理解这一点，请参阅[计算模拟器与 Azure 之间的差异](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg432960.aspx)和[存储模拟器与 Azure 存储服务之间的差异](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433135.aspx)。

请注意，你必须本地安装 PHP 才能使用计算模拟器。计算模拟器将使用本地 PHP 安装来运行应用程序。

若要在模拟器中运行你的项目，请从项目的根目录中执行以下命令：

	PS C:\MyProject> Start-AzureEmulator

你将看到与下图中所示类似的输出：

	Creating local package...
	Starting Emulator...
	Role is running at http://127.0.0.1:81
	Started

通过打开 Web 浏览器并浏览到输出中所示的本地地址（上面的示例输出中的 `http://127.0.0.1:81`），可以查看正在仿真程序上运行的应用程序。

若要停止模拟器，请执行此命令：

	PS C:\MyProject> Stop-AzureEmulator

## 如何：发布应用程序

若要发布应用程序，您需要先使用 **Import-PublishSettingsFile** 导入发布设置，然后使用 **Publish-AzureServiceProject** cmdlet 发布应用程序。有关使用这些 cmdlet 的详细信息，请参阅[如何：导入发布设置]和[如何：将云服务部署到 Azure]。

[execution model- web sites]: /documentation/articles/fundamentals-application-models#WebSites
[execution model-vms]: 
documentation/articles/fundamentals-application-models#VMachine
[execution model-cloud services]: /documentation/articles/fundamentals-application-models#CloudServices
[Azure SDK for PHP]: 
documentation/articles/php-download-sdk
[install ps and emulators]: http://go.microsoft.com/fwlink/?LinkId=253447&clcid=0x409
[什么是云服务？]: /documentation/articles/cloud-services-what-is
[服务定义 (.csdef)]: http://msdn.microsoft.com/zh-cn/library/azure/ee758711.aspx
[服务配置 (.cscfg)]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee758710.aspx
[iis.net]: http://www.iis.net/
[sql native client]: http://msdn.microsoft.com/zh-cn/sqlserver/aa937733.aspx
[sqlsrv drivers]: http://php.net/sqlsrv
[sqlncli.msi x64 安装程序]: http://go.microsoft.com/fwlink/?LinkID=239648
[如何：导入发布设置]: /documentation/articles/install-configure-powershell#ImportPubSettings
[如何：将云服务部署到 Azure]: 
documentation/articles/install-configure-powershell#Deploy

<!---HONumber=71-->