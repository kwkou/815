<properties
    pageTitle="在云服务角色上安装 .NET"
    description="本文介绍如何在云服务 Web 角色和辅助角色上手动安装 .NET Framework"
    services="cloud-services"
    documentationCenter=".net"
    authors="thraka"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="cloud-services"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/24/2016"
    ms.author="adegeo"
    wacn.date="04/24/2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="c4d5a3d1b85f5a264000b02466a1ad9355a15125"
    ms.lasthandoff="04/14/2017" />

# <a name="install-net-on-a-cloud-service-role"></a>在云服务角色上安装 .NET 
本文介绍如何在云服务 Web 角色和辅助角色上安装来宾 OS 随附版本以外的 .NET Framework 版本。 例如，可以使用这些步骤在 Azure 来宾 OS 系列 4 上安装不随任何版本的 .NET 4.6 提供的 .NET 4.6.1。 有关最新的来宾 OS 版本信息，请参阅 [Azure 来宾 OS 版本发行动态](/documentation/articles/cloud-services-guestos-update-matrix/)。

>[AZURE.NOTE]
>来宾 OS 5 包含 .NET 4.6

>[AZURE.IMPORTANT]
>Azure SDK 2.9 包含一个限制，限制将 .NET 4.6 部署到来宾 OS 4 或更低版本。 [此处](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Azure%20Targets%20SDK%202.9)提供了一个解决方法。

在 Web 角色和辅助角色上安装 .NET 的过程涉及到在云项目中添加 .NET 安装包，并在执行角色的启动任务过程中启动安装程序。  

## <a name="add-the-net-installer-to-your-project"></a>将 .NET 安装程序添加到项目
- 下载你要安装的.NET Framework 的 Web 安装程序
    - [.NET 4.6.1 Web 安装程序](http://go.microsoft.com/fwlink/?LinkId=671729)
- 对于 Web 角色
  1. 在“**解决方案资源管理器**”中，于云服务项目中的“**角色**”下，右键单击你的角色，然后选择“**添加 > 新文件夹**”。 创建一个名为 *bin* 的文件夹
  2. 右键单击 **bin** 文件夹，然后选择“**添加 > 现有项**”。 选择 .NET 安装程序，并将它添加到 bin 文件夹。
- 对于辅助角色
  1. 右键单击你的角色，然后选择“**添加 > 现有项**”。 选择 .NET 安装程序，并将它添加到角色。 

以此方式添加到角色内容文件夹的文件会自动添加到云服务包，并部署到虚拟机上的一致位置。对云服务中的所有 Web 和辅助角色重复此过程，使所有角色都有安装程序的副本。

> [AZURE.NOTE]
> 即使你的应用程序面向 .NET 4.6，你也应该在云服务角色上安装 .NET 4.6.1。 Azure 来宾 OS 包含更新 [3098779](https://support.microsoft.com/zh-cn/kb/3098779) 和 [3097997](https://support.microsoft.com/zh-cn/kb/3097997)。 在这些更新的顶层安装 .NET 4.6 可能会在运行 .NET 应用程序时造成问题，因此，你应该直接安装 .NET 4.6.1 而不是 .NET 4.6。 有关详细信息，请参阅 [KB 3118750](https://support.microsoft.com/zh-cn/kb/3118750)。

![包含安装程序文件的角色内容][1]  

## <a name="define-startup-tasks-for-your-roles"></a>为角色定义启动任务
启动任务可让你在启动角色之前执行操作。 将 .NET Framework 作为启动任务的一部分安装，可确保在运行任何应用程序代码之前已安装好 Framework。 有关启动任务的详细信息，请参阅[在 Azure 中运行启动任务](/documentation/articles/cloud-services-startup-tasks/)。 

1. 将以下内容添加到所有角色的 **WebRole** 或 **WorkerRole** 节点下的 *ServiceDefinition.csdef* 文件中：

    	<LocalResources>
          <LocalStorage name="NETFXInstall" sizeInMB="1024" cleanOnRoleRecycle="false" />
        </LocalResources>    
    	<Startup>
          <Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
            <Environment>
              <Variable name="PathToNETFXInstall">
                <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='NETFXInstall']/@path" />
              </Variable>
              <Variable name="ComputeEmulatorRunning">
                <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
              </Variable>
            </Environment>
          </Task>
        </Startup>

    上述配置将使用管理员特权来运行控制台命令*install.cmd*，以安装 .NET Framework。 该配置还会创建名为 *NETFXInstall*的 LocalStorage。 启动脚本会将临时文件夹设置为使用此本地存储资源，以便从此资源下载并安装 .NET Framework 安装程序。 必须将此资源的大小设置为至少 1024MB，以确保能够正确安装 Framework。 有关启动任务的详细信息，请参阅：[常见的云服务启动任务](/documentation/articles/cloud-services-startup-tasks-common/) 

2. 创建文件 **install.cmd**，然后右键单击角色并选择“**添加 > 现有项...**”将此文件添加到所有角色。 因此，所有角色现在应该都有 .NET 安装程序文件，以及 install.cmd 文件。

    ![包含所有文件的角色内容][2]

    > [AZURE.NOTE]
    > 使用记事本之类的简单文字编辑器来创建这个文件。 如果使用 Visual Studio 创建文本文件，然后将其重命名为“.cmd”，则此文件可能仍包含 UTF-8 字节顺序标记，并在运行第一行脚本时出现错误。 如果你要使用 Visual Studio 来创建文件，请在文件的第一行保留添加 REM（备注），以便在运行时将它忽略。 

3. 将以下脚本添加到 **install.cmd** 文件：


		REM Set the value of netfx to install appropriate .NET Framework. 
		REM ***** To install .NET 4.5.2 set the variable netfx to "NDP452" *****
		REM ***** To install .NET 4.6 set the variable netfx to "NDP46" *****
		REM ***** To install .NET 4.6.1 set the variable netfx to "NDP461" *****
		REM ***** To install .NET 4.6.2 set the variable netfx to "NDP462" *****
		set netfx="NDP461"
		
		
		REM ***** Set script start timestamp *****
		set timehour=%time:~0,2%
		set timestamp=%date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2%
		set "log=install.cmd started %timestamp%."
		
		REM ***** Exit script if running in Emulator *****
		if %ComputeEmulatorRunning%=="true" goto exit
		
		REM ***** Needed to correctly install .NET 4.6.1, otherwise you may see an out of disk space error *****
		set TMP=%PathToNETFXInstall%
		set TEMP=%PathToNETFXInstall%
		
		REM ***** Setup .NET filenames and registry keys *****
		if %netfx%=="NDP462" goto NDP462
		if %netfx%=="NDP461" goto NDP461
		if %netfx%=="NDP46" goto NDP46
		    set "netfxinstallfile=NDP452-KB2901954-Web.exe"
		    set netfxregkey="0x5cbf5"
		    goto logtimestamp
		
		:NDP46
		set "netfxinstallfile=NDP46-KB3045560-Web.exe"
		set netfxregkey="0x6004f"
		goto logtimestamp
		
		:NDP461
		set "netfxinstallfile=NDP461-KB3102438-Web.exe"
		set netfxregkey="0x6040e"
		goto logtimestamp
	
		:NDP462
		set "netfxinstallfile=NDP462-KB3151802-Web.exe"
		set netfxregkey="0x60632"
		:logtimestamp
		REM ***** Setup LogFile with timestamp *****
		md "%PathToNETFXInstall%\log"
		set startuptasklog="%PathToNETFXInstall%log\startuptasklog-%timestamp%.txt"
		set netfxinstallerlog="%PathToNETFXInstall%log\NetFXInstallerLog-%timestamp%"
		echo %log% >> %startuptasklog%
		echo Logfile generated at: %startuptasklog% >> %startuptasklog%
		echo TMP set to: %TMP% >> %startuptasklog%
		echo TEMP set to: %TEMP% >> %startuptasklog%
		
		REM ***** Check if .NET is installed *****
		echo Checking if .NET (%netfx%) is installed >> %startuptasklog%
		set /A netfxregkeydecimal=%netfxregkey%
		set foundkey=0
		FOR /F "usebackq skip=2 tokens=1,2*" %%A in (`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full" /v Release 2^>nul`) do @set /A foundkey=%%C
		echo Minimum required key: %netfxregkeydecimal% -- found key: %foundkey% >> %startuptasklog%
		if %foundkey% GEQ %netfxregkeydecimal% goto installed
		
		REM ***** Installing .NET *****
		echo Installing .NET with commandline: start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog%  /chainingpackage "CloudService Startup Task" >> %startuptasklog%
		start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog% /chainingpackage "CloudService Startup Task" >> %startuptasklog% 2>>&1
		if %ERRORLEVEL%== 0 goto installed
			echo .NET installer exited with code %ERRORLEVEL% >> %startuptasklog%	
			if %ERRORLEVEL%== 3010 goto restart
			if %ERRORLEVEL%== 1641 goto restart
			echo .NET (%netfx%) install failed with Error Code %ERRORLEVEL%. Further logs can be found in %netfxinstallerlog% >> %startuptasklog%
		
		:restart
		echo Restarting to complete .NET (%netfx%) installation >> %startuptasklog%
		EXIT /B %ERRORLEVEL%
		
		:installed
		echo .NET (%netfx%) is installed >> %startuptasklog%
		
		:end
		echo install.cmd completed: %date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2% >> %startuptasklog%
		
		:exit
		EXIT /B 0

    安装脚本将通过查询注册表来检查指定的 .NET Framework 版本是否已安装在计算机上。 如果未安装该 .NET 版本，.Net Web 安装程序将会启动。 为帮助排查任何问题，该脚本会将所有活动记录到名为 *startuptasklog-(currentdatetime).txt* 的文件（存储在 *InstallLogs* 本地存储中）。

    > [AZURE.NOTE]
    > 为了保持内容连贯，该脚本仍会演示如何安装 .NET 4.5.2 或 .NET 4.6。 你不需要手动安装 .NET 4.5.2，因为 Azure 来宾 OS 上已提供该组件。 由于 [KB 3118750](https://support.microsoft.com/zh-cn/kb/3118750)中所述的原因，应该直接安装 .NET 4.6.1，而不要安装 .NET 4.6。

## <a name="configure-diagnostics-to-transfer-the-startup-task-logs-to-blob-storage"></a>配置诊断以将启动任务日志传输到 Blob 存储 
为了方便排查任何安装问题，你可以配置 Azure 诊断，将启动脚本或 .NET 安装程序生成的任何日志文件传输到 Blob 存储。 使用这种方法可以从 Blob 存储直接下载日志文件，而无需通过远程桌面访问角色，即可查看日志。

若要配置诊断，请打开 *diagnostics.wadcfgx*，并在 **Directories** 节点下添加以下内容：


        <DataSources>
         <DirectoryConfiguration containerName="netfx-install">
          <LocalResource name="NETFXInstall" relativePath="log"/>
         </DirectoryConfiguration>
        </DataSources>

这会将 Azure 诊断配置为将 *NETFXInstall* 资源下的 *log* 目录中的所有文件传输到 *netfx-install* Blob 容器中的诊断存储帐户。

## <a name="deploying-your-service"></a>部署服务 
部署服务时，启动任务将会运行并安装 .NET Framework（如果尚未安装）。 安装 Framework 时，角色将处于忙碌状态，而且甚到可能会重新启动（如果 Framework 安装有此要求）。 

## <a name="additional-resources"></a>其他资源

- [安装 .NET Framework][]
- [如何：确定安装的 .NET Framework 版本][]
- [.NET Framework 安装疑难解答][]

[如何：确定安装的 .NET Framework 版本]: https://msdn.microsoft.com/zh-cn/library/hh925568.aspx
[安装 .NET Framework]: https://msdn.microsoft.com/zh-cn/library/5a4x27ek.aspx
[.NET Framework 安装疑难解答]: https://msdn.microsoft.com/zh-cn/library/hh925569.aspx

<!--Image references-->
[1]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithinstallerfiles.png
[2]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithallfiles.png

 

<!--Update_Description:update wording-->