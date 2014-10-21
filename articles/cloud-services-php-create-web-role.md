<properties linkid="develop-php-common-tasks-create-web-and-worker-roles" urlDisplayName="Create Web and Worker Roles" pageTitle="Create Web and Worker Roles" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="PHP" title="How to create PHP web and worker roles" authors="bswan" solutions="" manager="paulettm" editor="mollybos" />

# 如何创建 PHP Web 角色和辅助角色

本指南将说明如何执行以下操作：在 Windows 开发环境中创建 PHP Web 角色或辅助角色，从提供的“内置”版本中选择特定版本的 PHP，更改 PHP 配置，启用扩展以及部署到 Azure。它还介绍了如何将 Web 角色或辅助角色配置为使用你提供的 PHP 运行时（带自定义配置和扩展）。

## <a name="TableOfContents"></a>目录

-   [什么是 PHP Web 角色和辅助角色？][什么是 PHP Web 角色和辅助角色？]
-   [下载 Azure SDK for PHP][下载 Azure SDK for PHP]
-   [如何：创建云服务项目][如何：创建云服务项目]
-   [如何：添加 PHP Web 角色和辅助角色][如何：添加 PHP Web 角色和辅助角色]
-   [如何：指定内置 PHP 版本][如何：指定内置 PHP 版本]
-   [如何：自定义内置 PHP 运行时][如何：自定义内置 PHP 运行时]
-   [如何：使用你自己的 PHP 运行时][如何：使用你自己的 PHP 运行时]
-   [如何：在计算和存储模拟器中运行你的应用程序][如何：在计算和存储模拟器中运行你的应用程序]
-   [如何：发布应用程序][如何：发布应用程序]

## <a name="WhatIs"></a>什么是 PHP Web 角色和辅助角色？

Azure 提供了三种用于运行应用程序的计算模型：[Azure 网站][Azure 网站]、[Azure 虚拟机][Azure 虚拟机]和 [Azure 云服务][Azure 云服务]。这三种模型都支持 PHP。云服务（包括 Web 角色和辅助角色）提供了*平台即服务 (PaaS)*。在云服务中，Web 角色提供专用的 Internet Information Services (IIS) Web 服务器来托管前端 Web 应用程序，而辅助角色可独立于用户交互或输入运行异步任务、运行时间较长的任务或永久性任务。

有关详细信息，请参阅[什么是云服务？][什么是云服务？]。

## <a name="DownloadSdk"></a>下载 Azure SDK for PHP

[Azure SDK for PHP][Azure SDK for PHP] 由多个组件构成。本文将使用其中两个组件：Azure PowerShell 和 Azure 模拟器。可在此处通过 Microsoft Web 平台安装程序安装这两个组件：[安装 Azure PowerShell 和 Azure 模拟器][安装 Azure PowerShell 和 Azure 模拟器]。

## <a name="CreateProject"></a>如何：创建云服务项目

创建 PHP Web 角色或辅助角色的第一步是创建 Azure 服务项目。Azure 服务项目用作 Web 角色和辅助角色的逻辑容器，并包含项目的[服务定义 (.csdef)][服务定义 (.csdef)] 和[服务配置 (.cscfg)][服务配置 (.cscfg)] 文件。

若要创建新的 Azure 服务项目，请执行以下命令：

    PS C:\>New-AzureServiceProject myProject

此命令将创建可将 Web 角色和辅助角色添加到的新目录 (`myProject`)。

## <a name="AddRole"></a>如何：添加 PHP Web 角色或辅助角色

若要将 PHP Web 角色添加到项目，请从项目的根目录中运行以下命令：

    PS C:\myProject> Add-AzurePHPWebRole roleName

对于辅助角色，请使用此命令：

    PS C:\myProject> Add-AzurePHPWorkerRole roleName

<div class="dev-callout"> 
<b>说明</b> 
<p>语句 <code>roleName</code> 参数是可选的。如果省略该参数，则将自动生成角色名称。创建的第一个 Web 角色为 <code>WebRole1</code>，第二个 Web 角色为 <code>WebRole2</code>，依此类推。创建的第一个辅助角色为 <code>WorkerRole1</code>，第二个辅助角色为 <code>WorkerRole2</code>，依此类推。</p> 
</div>

## <a name="SpecifyPHPVersion"></a>如何：指定内置 PHP 版本

在将 PHP Web 角色或辅助角色添加到项目时，将修改项目的配置文件，以便在部署应用程序的每个 Web 实例或辅助进程实例时在其上安装 PHP。若要查看默认情况下安装的 PHP 的版本，请运行以下命令：

    PS C:\myProject> Get-AzureServiceProjectRoleRuntime

上述命令的输出与下图中所示类似。在本示例中，PHP 5.3.17 的 `IsDefault` 标志已设置为`true` ，这表示它将是安装的默认 PHP 版本。

    Runtime Version     PackageUri                      IsDefault
    ------- -------     ----------                      ---------
    Node 0.6.17         http://nodertncu.blob.core...   False
    Node 0.6.20         http://nodertncu.blob.core...   True
    Node 0.8.4          http://nodertncu.blob.core...   False
    IISNode 0.1.21      http://nodertncu.blob.core...   True
    Cache 1.8.0         http://nodertncu.blob.core...   True
    PHP 5.3.17          http://nodertncu.blob.core...   True
    PHP 5.4.0           http://nodertncu.blob.core...   False

可以将 PHP 运行时版本设置为列出的任意 PHP 版本。例如，若要将 PHP 版本（对于名为 `roleName`的角色）设置为 5.4.0，请使用以下命令：

    PS C:\myProject> Set-AzureServiceProjectRole roleName php 5.4.0

<div class="dev-callout"> 
<b>说明</b> 
<p>将来可能会提供更多 PHP 版本，并且可用版本可能会发生更改。</p> 
</div>

## <a name="CustomizePHP"></a>如何：自定义内置 PHP 运行时

当按上述步骤进行操作时，你可以完全控制所安装的 PHP 运行时的配置，包括修改 `php.ini` 设置和启用扩展。

若要自定义内置 PHP 运行时，请执行下列步骤：

1.  将名为 `php`的新文件夹添加到 Web 角色的 `bin` 目录。对于辅助角色，将该文件夹添加到角色的根目录。
2.  在`php` 文件夹中，创建另一个文件夹并将其命名为 `ext`. 将要启用的任何扩展名为 `.dll` 的文件（例如 `php_mongo.dll`）置于此文件夹中。
3.  将一个 `php.ini` 文件添加到 `php` 文件夹。启用任何自定义扩展，并在此文件中设置任何 PHP 指令。例如，如果打开了 `display_errors` 并启用了 `php_mongo.dll` 扩展，则 `php.ini` 文件的内容将如下所示：

        display_errors=On
        extension=php_mongo.dll

<div class="dev-callout"> 
<b>说明</b> 
<p>所提供的 <code>php.ini</code> 文件中未显式设置的所有设置都将自动设为其默认值。但请记住，你可以添加整个 <code>php.ini</code> 文件。 </p> 
</div>

## <a name="OwnPHP"></a>如何：使用你自己的 PHP 运行时

在某些情况下，可能需要提供你自己的 PHP 运行时，而不是如上所述那样选择并配置内置 PHP 运行时。例如，可以在 Web 角色或辅助角色中使用你在开发环境中使用的 PHP 运行时，以便更轻松地确保应用程序不会更改生产环境中的行为。

### <a name="OwnPHPWebRole"></a>将 Web 角色配置为使用你自己的 PHP 运行时

若要将 Web 角色配置为使用提供的 PHP 运行时，请执行下列步骤。

1.  创建 Azure 服务项目并添加 PHP Web 角色，如前面的[如何：创建云服务项目][如何：创建云服务项目]和[如何：添加 PHP Web 角色或辅助角色][如何：添加 PHP Web 角色和辅助角色]部分中所述。
2.  创建一个 `php` 文件夹（在位于 Web 角色的根目录中的 `bin` 文件夹中），然后将 PHP 运行时（所有二进制文件、配置文件、子文件夹等）添加到该 `php` 文件夹。
3.  （可选）如果 PHP 运行时使用 [Microsoft Drivers for PHP for SQL Server][Microsoft Drivers for PHP for SQL Server]，则需要将 Web 角色配置为在 [SQL Server Native Client 2012][SQL Server Native Client 2012] 可用时安装它。为此，请将 `sqlncli.msi` 安装程序添加到位于 Web 角色的根目录中的 `bin` 文件夹。你可在此处下载该安装程序：[sqlncli.msi x64 安装程序][sqlncli.msi x64 安装程序]。下一步中所述的启动脚本将在设置角色时以静默方式运行安装程序。如果你的 PHP 运行时不使用 Microsoft Drivers for PHP for SQL Server，则可从下一步所示的脚本中删除以下行：

        msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

4.  下一步是定义将 [Internet Information Services (IIS)][Internet Information Services (IIS)] 配置为使用 PHP 运行时来处理`.php` 页的请求的启动任务。为此，请在文本编辑器中打开 `setup_web.cmd` 文件（在位于 Web 角色的根目录中的 `bin` 文件中），并将其内容替换为以下脚本：

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

5.  将应用程序文件添加到 Web 角色的根目录。这将是 Web 服务器的根目录。

6.  按照下面的[如何：发布应用程序][如何：发布应用程序]部分中所述发布你的应用程序。

<div class="dev-callout"> 
<b>说明</b> 
<p>语句 <code>download.ps1</code> 脚本（位于 Web 角色的根目录的 <code>bin</code> 文件夹中）在按照上述使用你自己的 PHP 运行时的步骤进行操作后可以删除。</p> 
</div>

### <a name="OwnPHPWorkerRole"></a>将辅助角色配置为使用你自己的 PHP 运行时

若要将辅助角色配置为使用提供的 PHP 运行时，请执行下列步骤。

1.  创建 Azure 服务项目并添加 PHP 辅助角色，如前面的[如何：创建云服务项目][如何：创建云服务项目]和[如何：添加 PHP Web 角色或辅助角色][如何：添加 PHP Web 角色和辅助角色]部分中所述。
2.  在辅助角色的根目录中创建一个 `php` 文件夹，然后将 PHP 运行时（所有二进制文件、配置文件、子文件夹等）添加到该 `php` 文件夹。
3.  （可选）如果 PHP 运行时使用 [Microsoft Drivers for PHP for SQL Server][Microsoft Drivers for PHP for SQL Server]，则需要将辅助角色配置为在 [SQL Server Native Client 2012][SQL Server Native Client 2012] 可用时安装它。为此，请将 `sqlncli.msi` 安装程序添加到辅助角色的根目录。你可在此处下载该安装程序：[sqlncli.msi x64 安装程序][sqlncli.msi x64 安装程序]。下一步中所述的启动脚本将在设置角色时以静默方式运行安装程序。如果你的 PHP 运行时不使用 Microsoft Drivers for PHP for SQL Server，则可从下一步所示的脚本中删除以下行：

        msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

4.  下一步是定义在设置角色时将 `php.exe` 可执行文件添加到辅助角色的 PATH 环境变量中的启动任务。为此，请在文本编辑器中打开 `setup_worker.cmd` 文件（在辅助角色的根目录中），并将其内容替换为以下脚本：

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

5.  将应用程序文件添加到辅助角色的根目录。

6.  按照下面的[如何：发布应用程序][如何：发布应用程序]部分中所述发布你的应用程序。

## <a name="Emulators"></a>如何：在计算和存储模拟器中运行你的应用程序

Azure 计算和存储模拟器提供了一个本地环境，可在将 Azure 应用程序部署到云之前在该环境中测试此应用程序。模拟器与 Azure 环境之间存在一些差异。若要更好地理解这一点，请参阅[计算模拟器与 Azure 之间的差异][计算模拟器与 Azure 之间的差异]和[存储模拟器与 Azure 存储服务之间的差异][存储模拟器与 Azure 存储服务之间的差异]。

请注意，你必须本地安装 PHP 才能使用计算模拟器。计算模拟器将使用本地 PHP 安装来运行应用程序。

若要在模拟器中运行你的项目，请从项目的根目录中执行以下命令：

    PS C:\MyProject> Start-AzureEmulator

你将看到与下图中所示类似的输出：

    Creating local package...
    Starting Emulator...
    Role is running at http://127.0.0.1:81
    Started

通过打开 Web 浏览器并浏览到输出中所示的本地地址（上面的示例输出中的 `http://127.0.0.1:81` ），可以查看正在模拟器上运行的应用程序。

若要停止模拟器，请执行此命令：

    PS C:\MyProject> Stop-AzureEmulator

## <a name="Publish"></a>如何：发布应用程序

若要发布应用程序，你需要先使用 **Import-PublishSettingsFile** cmdlet 导入发布设置，然后使用 **Publish-AzureServiceProject** cmdlet 发布应用程序。有关使用这些 cmdlet 的详细信息，请分别参阅[如何：导入发布设置][如何：导入发布设置]和[如何：将云服务部署到 Azure][如何：将云服务部署到 Azure]。

  [什么是 PHP Web 角色和辅助角色？]: #WhatIs
  [下载 Azure SDK for PHP]: #DownloadSdk
  [如何：创建云服务项目]: #CreateProject
  [如何：添加 PHP Web 角色和辅助角色]: #AddRole
  [如何：指定内置 PHP 版本]: #SpecifyPHPVersion
  [如何：自定义内置 PHP 运行时]: #CustomizePHP
  [如何：使用你自己的 PHP 运行时]: #OwnPHP
  [如何：在计算和存储模拟器中运行你的应用程序]: #Emulators
  [如何：发布应用程序]: #Publish
  [Azure 网站]: /zh-cn/develop/net/fundamentals/compute/#WebSites
  [Azure 虚拟机]: /zh-cn/develop/net/fundamentals/compute/#VMachine
  [Azure 云服务]: /zh-cn/develop/net/fundamentals/compute/#CloudServices
  [什么是云服务？]: /zh-cn/manage/services/cloud-services/what-is-a-cloud-service/
  [Azure SDK for PHP]: /zh-cn/develop/php/common-tasks/download-php-sdk/
  [安装 Azure PowerShell 和 Azure 模拟器]: http://go.microsoft.com/fwlink/?LinkId=253447&clcid=0x409
  [服务定义 (.csdef)]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee758711.aspx
  [服务配置 (.cscfg)]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee758710.aspx
  [Microsoft Drivers for PHP for SQL Server]: http://php.net/sqlsrv
  [SQL Server Native Client 2012]: http://msdn.microsoft.com/zh-cn/sqlserver/aa937733.aspx
  [sqlncli.msi x64 安装程序]: http://go.microsoft.com/fwlink/?LinkID=239648
  [Internet Information Services (IIS)]: http://www.iis.net/
  [计算模拟器与 Azure 之间的差异]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg432960.aspx
  [存储模拟器与 Azure 存储服务之间的差异]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433135.aspx
  [如何：导入发布设置]: /zh-cn/develop/php/how-to-guides/powershell-cmdlets/#ImportPubSettings
  [如何：将云服务部署到 Azure]: /zh-cn/develop/php/how-to-guides/powershell-cmdlets/#Deploy
