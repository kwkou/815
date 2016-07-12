<properties 
pageTitle="云服务的常见启动任务 |Azure" 
description="提供了一些你可能需要在云服务 Web 角色或辅助角色中执行的常见启动任务示例。" 
services="cloud-services" 
documentationCenter="" 
authors="Thraka" 
manager="timlt" 
editor=""/>
<tags 
	ms.service="cloud-services" 
	ms.date="12/07/2015" 
	wacn.date="01/15/2016"/>

# 常见的云服务启动任务

本文提供了一些你可能需要在云服务中执行的常见启动任务示例。在角色启动之前，可以使用启动任务执行操作。你可能需要执行的操作包括安装组件、注册 COM 组件、设置注册表项或启动长时间运行的进程。

参阅[本文](/documentation/articles/cloud-services-startup-tasks/)可了解启动任务的工作方式，特别是如何创建定义启动任务的条目。

此处的许多任务使用

>[AZURE.NOTE]启动任务不适用于虚拟机，只适用于云服务 Web 角色和辅助角色。


## 在角色启动之前定义环境变量

可以通过向服务定义文件中的角色定义添加 [Runtime] 元素来定义整个角色的环境变量。

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Runtime>
            <Environment>
                <Variable name="MyEnvironmentVariable" value="MyVariableValue" />
            </Environment>
        </Runtime>
    </WebRole>
</ServiceDefinition>
```

如果你需要为特定任务定义环境变量（不由其他任务共享），则可以使用 [Task] 元素内的 [Environment] 元素。

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
                <Environment>
                    <Variable name="MyEnvironmentVariable" value="MyVariableValue" />
                </Environment>
            </Task>
        </Startup>
    </WebRole>
</ServiceDefinition>
```

此外，变量还可以使用[有效的 Azure XPath 值](https://msdn.microsoft.com/zh-cn/library/azure/hh404006.aspx)引用有关部署的内容。请不要使用 `value` 属性，而是定义 [RoleInstanceValue] 子元素。

```xml
<Variable name="PathToStartupStorage">
    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
</Variable>
```


## 使用 AppCmd.exe 配置 IIS 启动

[AppCmd.exe](https://technet.microsoft.com/zh-cn/library/jj635852.aspx) 命令行工具在 Azure 上启动时可用于管理 IIS 设置。*AppCmd.exe* 提供对要在 Azure 上的启动任务中使用的配置设置的方便的命令行访问。使用 *AppCmd.exe*，可以为应用程序和站点添加、修改或删除网站设置。

但是，在使用 *AppCmd.exe* 作为启动任务时有几点需要注意：

- 启动任务在重新启动之间可以运行多次。例如，如果角色回收，则可能发生这种情况。
- 某些 *AppCmd.exe* 操作在多次执行时可能会生成错误。尝试将某个节添加到 *Web.config* 中两次会生成错误。
- 如果启动任务返回非零退出代码或 **errorlevel**，则为失败。如果 *AppCmd.exe* 生成错误，则可能会发生这种情况。

由于列出的原因，比较明智的做法通常是在调用 *AppCmd.exe* 之后检查 **errorlevel**，如果你使用 *.cmd* 文件包装对 *AppCmd.exe* 的调用，则很容易做到这一点。如果检测到已知的 **errorlevel** 响应，你可以忽略它，否则将其返回。下面的示例演示了这一点。

*AppCmd.exe* 返回的 errorlevel 在 winerror.h 文件中列出，并且还可以在 [MSDN](https://msdn.microsoft.com/zh-cn/library/windows/desktop/ms681382.aspx) 上看到。

### 示例

此示例将 JSON 的压缩节和压缩条目添加到 *Web.config* 文件，其中包含错误处理和日志记录。

此处显示了 [ServiceDefinition.csdef] 文件的相关节，其中包括将 [executionContext](https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Task) 属性设为 `elevated` 以为 *AppCmd.exe* 提供足够的权限来更改 *Web.config* 文件中的设置：

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

Startup.cmd 批处理文件使用 AppCmd.exe 将 JSON 的压缩节和压缩条目添加到 *Web.config* 文件。使用 VERIFY.EXE 命令行程序将预期的 **errorlevel** 183 设为零。意外的 errorlevel 将记录到 StartupErrorLog.txt 中。

    REM   *** Add a compression section to the Web.config file. ***
    %windir%\system32\inetsrv\appcmd set config /section:urlCompression /doDynamicCompression:True /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1
    
    REM   ERRORLEVEL 183 occurs when trying to add a section that already exists. This error is expected if this
    REM   batch file were executed twice. This can occur and must be accounted for in a Azure startup
    REM   task. To handle this situation, set the ERRORLEVEL to zero by using the Verify command. The Verify
    REM   command will safely set the ERRORLEVEL to zero.
    IF %ERRORLEVEL% EQU 183 DO VERIFY > NUL
    
    REM   If the ERRORLEVEL is not zero at this point, some other error occurred.
    IF %ERRORLEVEL% NEQ 0 (
        ECHO Error adding a compression section to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
        GOTO ErrorExit
    )
    
    REM   *** Add compression for json. ***
    %windir%\system32\inetsrv\appcmd set config  -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json; charset=utf-8',enabled='True']" /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1
    IF %ERRORLEVEL% EQU 183 VERIFY > NUL
    IF %ERRORLEVEL% NEQ 0 (
        ECHO Error adding the JSON compression type to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
        GOTO ErrorExit
    )
    
    REM   *** Exit batch file. ***
    EXIT /b 0
    
    REM   *** Log error and exit ***
    :ErrorExit
    REM   Report the date, time, and ERRORLEVEL of the error.
    DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
    TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
    ECHO An error occurred during startup. ERRORLEVEL = %ERRORLEVEL% >> "%TEMP%\StartupLog.txt" 2>&1
    EXIT %ERRORLEVEL%


## 添加防火墙规则

在 Azure 中，实际上有两个防火墙。第一个防火墙控制虚拟机与外界之间的连接。这由 [ServiceDefinition.csdef] 文件中的 [EndPoints] 元素控制。

第二个防火墙控制虚拟机与该虚拟机中的进程之间的连接。这由 `netsh advfirewall firewall` 命令行工具控制，它是本文的重点。

Azure 将为你角色中启动的进程创建防火墙规则。例如，当你启动服务或程序时，Azure 将自动创建必要的防火墙规则以允许该服务与 Internet 进行通信。但是，如果你创建的服务由你角色外部的进程（例如，COM+ 服务或使用 Windows 计划程序启动的程序）启动，则将需要手动创建防火墙规则以允许访问该服务。可以通过使用启动任务来创建这些防火墙规则。

创建防火墙规则的启动任务的 [executionContext][Task] 必须为 **elevated**。将以下启动任务添加到 [ServiceDefinition.csdef] 文件。

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="AddFirewallRules.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

若要添加防火墙规则，必须在启动批处理文件中使用相应的 `netsh advfirewall firewall` 命令。在此示例中，启动任务对 TCP 端口 80 具有安全性和加密要求。

    REM   Add a firewall rule in a startup task.
    
    REM   Add an inbound rule requiring security and encryption for TCP port 80 traffic.
    netsh advfirewall firewall add rule name="Require Encryption for Inbound TCP/80" protocol=TCP dir=in localport=80 security=authdynenc action=allow >> "%TEMP%\StartupLog.txt" 2>&1
    
    REM   If an error occurred, return the errorlevel.
    EXIT /B %errorlevel%


## 阻止特定 IP 地址

可以通过修改 IIS **web.config** 文件和创建命令文件（用于解锁 **ApplicationHost.config** 文件的 **ipSecurity** 节）来限制 Azure web 角色对指定的 IP 地址的访问。

首先，创建一个用于解锁 **ApplicationHost.config** 文件的 **ipSecurity** 节的命令文件，该文件将在你的角色启动时运行。在 web 角色的根级别创建一个名为 **startup** 的新文件夹，然后在该文件夹中创建一个名为 **startup.cmd** 的批处理文件。将此文件的属性设为“始终复制”以确保将部署它。

将以下启动任务添加到 [ServiceDefinition.csdef] 文件。

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="startup.cmd" executionContext="elevated" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

将此命令添加到 **startup.cmd** 文件：

    %windir%\system32\inetsrv\AppCmd.exe unlock config -section:system.webServer/security/ipSecurity

这将导致每次初始化 web 角色时都运行 **startup.cmd** 批处理文件，从而确保所需的 **ipSecurity** 节处于解锁状态。

最后，修改 web 角色的 **web.config** 文件的 [system.webServer 节](http://www.iis.net/configreference/system.webserver/security/ipsecurity#005)以添加被授予访问权限的 IP 地址列表，如下面的示例所示：

此示例配置**允许**所有 IP（两个已定义的 IP 除外）访问服务器

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are granted access-->
    <ipSecurity>
        <!--The following IP addresses are denied access-->
        <add allowed="false" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="false" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

此示例配置**拒绝**所有 IP（两个已定义的 IP 除外）访问服务器。

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are denied access-->
    <ipSecurity allowUnlisted="false">
        <!--The following IP addresses are granted access-->
        <add allowed="true" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="true" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

## 创建 PowerShell 启动任务

Windows PowerShell 脚本不能直接从 [ServiceDefinition.csdef] 文件调用，但它们可以从启动批处理文件中调用。

默认情况下，PowerShell 将不会运行未签名的脚本。除非你为脚本签名，否则需要配置 Windows PowerShell 以运行未签名的脚本。若要运行未签名的脚本，**ExecutionPolicy** 必须设置为 **Unrestricted**。你使用的 **ExecutionPolicy** 设置基于 Windows PowerShell 的版本。

    REM   Run an unsigned PowerShell script and log the output
    PowerShell -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1
        
    REM   If an error occurred, return the errorlevel.
    EXIT /B %errorlevel%


如果你使用的是运行 PowerShell 2.0 或 1.0 的来宾 OS，则可强制运行版本 2，如果不可用，则使用版本 1。

    REM   Attempt to set the execution policy by using PowerShell version 2.0 syntax.
    PowerShell -Version 2.0 -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

    REM   If PowerShell version 2.0 isn't available. Set the execution policy by using the PowerShell
    IF %ERRORLEVEL% EQU -393216 (

       PowerShell -Command "Set-ExecutionPolicy Unrestricted" >> "%TEMP%\StartupLog.txt" 2>&1
       PowerShell .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1
    )

    REM   If an error occurred, return the errorlevel.
    EXIT /B %errorlevel%

## 通过启动任务在本地存储中创建文件

可以使用本地存储资源来存储启动任务创建的文件，你的应用程序稍后将访问这些文件。

若要创建本地存储资源，请将 [LocalResources] 节添加到 [ServiceDefinition.csdef] 文件，然后添加 [LocalStorage] 子元素。为本地存储资源指定唯一名称，并为启动任务指定合适大小。

若要在启动任务中使用本地存储资源，需要创建一个环境变量以引用本地存储资源位置。然后启动任务和应用程序将能够读取本地存储资源并将文件写入其中。

在此处显示 **ServiceDefinition.csdef** 文件的相关节：

	<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
	    <WebRole name="WebRole1">
	        ...
	        
	        <LocalResources>
	          <LocalStorage name="StartupLocalStorage" sizeInMB="5"/>
	        </LocalResources>
	        
	        ...
	        
	        <Runtime>
	            <Environment>
	                <Variable name="PathToStartupStorage">
	                    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
	                </Variable>
	            </Environment>
	        </Runtime>
	        
	        ...
	        
	        <Startup>
	          <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" />
	        </Startup>
	    </WebRole>
	</ServiceDefinition>


例如，这个 **Startup.cmd** 批处理文件使用 **PathToStartupStorage** 环境变量在本地存储位置上创建文件 **MyTest.txt**。

    REM   Create a simple text file.

    ECHO This text will go into the MyTest.txt file which will be in the    >  "%PathToStartupStorage%\MyTest.txt"
    ECHO path pointed to by the PathToStartupStorage environment variable.  >> "%PathToStartupStorage%\MyTest.txt"
    ECHO The contents of the PathToStartupStorage environment variable is   >> "%PathToStartupStorage%\MyTest.txt"
    ECHO "%PathToStartupStorage%".                                          >> "%PathToStartupStorage%\MyTest.txt"

    REM   Exit the batch file with ERRORLEVEL 0.

    EXIT /b 0

你可以使用 [GetLocalResource](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.getlocalresource.aspx) 方法通过 Azure SDK 访问本地存储。然后，标准文件读取和写入操作将工作以读取和写入本地存储资源的内容。

```csharp
string localStoragePath = Microsoft.WindowsAzure.ServiceRuntime.RoleEnvironment.GetLocalResource("StartupLocalStorage").RootPath;

string fileContent = System.IO.File.ReadAllText(System.IO.Path.Combine(localStoragePath, "MyTest.txt"));
```


## 区分在模拟器中还是在云中运行

与在计算模拟器中运行时相比，你可以让启动任务在云中运行时执行不同的步骤。例如，仅当在模拟器中运行时，才可能需要使用 SQL 数据的新副本。或者你可能需要为云执行在模拟器中运行时不需要执行的某种性能优化。

在计算模拟器中和云中执行不同操作的功能可通过在 [ServiceDefinition.csdef] 文件中创建环境变量，然后在启动任务中测试该环境变量来实现。

若要创建环境变量，请添加 [Variable]/[RoleInstanceValue] 元素并创建 `/RoleEnvironment/Deployment/@emulated` 的 XPath 值。在计算模拟器中运行时，**%ComputeEmulatorRunning%** 环境变量的值将为 `"true"`，而在云中运行时，该值将为 `"false"`。


	<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
	    <WebRole name="WebRole1">
	
	        ...
	        
	        <Runtime>
	            <Environment>
	                <Variable name="ComputeEmulatorRunning">
	                    <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
	                </Variable>
	            </Environment>
	        </Runtime>
	
	    </WebRole>
	</ServiceDefinition>


运行的任何任务现在可以使用 **%ComputeEmulatorRunning%** 环境变量根据角色是在云中还是在模拟器中运行来执行不同的操作。下面是用于检查该环境变量的 .cmd shell 脚本。

    REM   Check if this task is running on the compute emulator.

    IF "%ComputeEmulatorRunning%" == "true" (
        REM   This task is running on the compute emulator. Perform tasks that must be run only in the compute emulator.
        
    ) ELSE (
        REM   This task is running on the cloud. Perform tasks that must be run only in the cloud.
        
    )


## 检测到你的任务已运行

该角色可能会无需重新启动即可回收，从而不会导致启动任务重新运行。有标志指示任务已在宿主 VM 上运行。你可能有一些任务它们运行多次无关紧要。但是，你也可能会遇到需要阻止任务运行多次的情况。

检测任务是否已运行的最简单方式是在任务成功时在 **%TEMP%** 文件夹中创建一个文件，然后在任务开始时查找该文件。下面是可为你执行该操作的示例 cmd shell 脚本。

    REM   If Task1_Success.txt exists, then Application 1 is already installed.
    IF EXIST "%RoleRoot%\Task1_Success.txt" (
      ECHO Application 1 is already installed. Exiting. >> "%TEMP%\StartupLog.txt" 2>&1
      GOTO Finish
    )

    REM   Run your real exe task
    ECHO Running XYZ >> "%TEMP%\StartupLog.txt" 2>&1
    "%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1

    IF %ERRORLEVEL% EQU 0 (
      REM   The application installed without error. Create a file to indicate that the task
      REM   does not need to be run again.

      ECHO This line will create a file to indicate that Application 1 installed correctly. > "%RoleRoot%\Task1_Success.txt"
      
    ) ELSE (
      REM   An error occurred. Log the error and exit with the error code.

      DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
      TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
      ECHO  An error occurred running task 1. Errorlevel = %ERRORLEVEL%. >> "%TEMP%\StartupLog.txt" 2>&1

      EXIT %ERRORLEVEL%
    )

    :Finish

    REM   Exit normally.
    EXIT /B 0


## 任务最佳做法
以下是在配置 web 角色或辅助角色的任务时应遵循的一些最佳做法。

### 始终记录启动活动

Visual Studio 未提供用于单步调试批处理文件的调试器，因此最好在批处理文件操作中尽可能多地获取数据。记录批处理文件的输出（**stdout** 和 **stderr**），可以在尝试调试和修复批处理文件时为你提供重要信息。若要记录 **%TEMP%** 环境变量指向的目录中 StartupLog.txt 文件的 **stdout** 和 **stderr**，请将文本 `>>  "%TEMP%\\StartupLog.txt" 2>&1` 添加到要记录的特定行的末尾。例如，若要在 **%PathToApp1Install%** 目录中执行 setup.exe，请执行以下操作：

    "%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1

如果要不向每行的末尾添加 `>> "%TEMP%\StartupLog.txt" 2>&1` 就记录启动任务的所有输出，则需要两个启动批处理文件。第一个批处理文件将调用第二个批处理文件并重定向以记录第二个批处理文件的所有活动。这是要进行正确的重定向所必需的。

下面演示如何重定向启动批处理文件的所有输出。在此示例中，ServerDefinition.csdef 文件将创建调用 Startup1.cmd 的启动任务。Startup1.cmd 调用 Startup2.cmd，并将所有输出都重定向到 %TEMP%\\StartupLog.txt。

ServiceDefinition.cmd：

```xml
<Startup>
    <Task commandLine="Startup1.cmd" executionContext="limited" taskType="simple" />
</Startup>
```

Startup1.cmd：

    REM   Startup1.cmd calls the main startup batch file, Startup2.cmd, redirecting all output
    REM   to the StartupLog.txt log file.

    REM   Log the startup date and time.
    ECHO Startup1.cmd: >> "%TEMP%\StartupLog.txt" 2>&1
    ECHO Current date and time: >> "%TEMP%\StartupLog.txt" 2>&1
    DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
    TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
    ECHO Starting up Startup2.cmd. >> "%TEMP%\StartupLog.txt" 2>&1

    REM   Call the Startup2.cmd batch file, redirecting all output to the StartupLog.txt log file.
    START /B /WAIT Startup2.cmd >> "%TEMP%\StartupLog.txt" 2>&1

    REM   Log the completion of Startup1.cmd.
    ECHO Returned to Startup1.cmd. >> "%TEMP%\StartupLog.txt" 2>&1

    IF ERRORLEVEL EQU 0 (
       REM   No errors occurred. Exit Startup1.cmd normally.
       EXIT /B 0
    ) ELSE (
       REM   Log the error.
       ECHO An error occurred. The ERRORLEVEL = %ERRORLEVEL%.  >> "%TEMP%\StartupLog.txt" 2>&1
       EXIT %ERRORLEVEL%
    )

Startup2.cmd：

    REM   This is the batch file where the startup steps should be performed. Because of the
    REM   way Startup2.cmd was called, all commands and their outputs will be stored in the
    REM   StartupLog.txt file in the directory pointed to by the TEMP environment variable.

    REM   If an error occurs, the following command will pass the ERRORLEVEL back to the
    REM   calling batch file.
    EXIT /B %ERRORLEVEL%

### 为启动任务适当地设置 executionContext

为启动任务适当地设置权限。有时启动任务必须以提升的权限运行，即使角色以普通权限运行，也是如此。

[executionContext][Task] 属性将设置启动任务的权限级别。使用 `executionContext="limited"` 意味着启动任务将具有与角色相同的权限级别。使用 `executionContext="elevated"` 意味着启动任务将具有管理员权限，这将允许启动任务执行管理员任务，而无需向你的角色授予管理员权限。

需要提升的权限的启动任务示例是使用 **AppCmd.exe** 配置 IIS 的启动任务。**AppCmd.exe** 需要 `executionContext="elevated"`。

### 使用适当的 taskType

[taskType][Task] 属性确定将执行启动任务的方式。有三个值：**simple**、**background** 和 **foreground**。background 和 foreground 任务以异步方式启动，然后 simple 任务以同步方式执行（一次一个）。

使用 **simple** 启动任务，可以设置顺序，让任务按照其在 ServiceDefinition.csdef 文件中列出的顺序执行。如果 **simple** 任务以非零退出代码结束，则启动过程将停止，并且角色将不会启动。

**background** 启动任务和 **foreground** 启动任务之间的区别在于 **foreground** 任务将使角色一直运行，直到 **foreground** 任务结束为止。这也意味着，如果 **foreground** 任务挂起或崩溃，角色将不会回收，直到 **foreground** 任务被强制关闭。因此，对于异步启动任务建议使用 **background** 任务，除非你需要 **foreground** 任务的功能。

### 以 EXIT /B 0 结束批处理文件

仅当每个 simple 启动任务的 **errorlevel** 均为零时，角色才会启动。并非所有程序都正确设置 **errorlevel**（退出代码），因此如果一切正常运行，批处理文件应以 `EXIT /B 0` 结束。

在启动批处理文件的末尾缺少 `EXIT /B 0` 是角色未启动的常见原因。

### 启动任务应多次运行

并非所有角色回收都包括重新启动，但所有角色回收都包括运行所有启动任务。这意味着启动任务必须能够在重新启动之间运行多次而不出现任何问题。这已在[前面](#detect-that-your-task-has-already-run)进行讨论。

### 使用本地存储来存储必须在角色中访问的文件

如果你要在启动任务期间复制或创建随后可由角色访问的文件，则该文件必须放置在本地存储中。请参阅前面的[节](#create-files-in-local-storage-from-a-startup-task)。

## 后续步骤

查看云[服务模型和包](/documentation/articles/cloud-services-model-and-package/)

详细了解[任务](/documentation/articles/cloud-services-startup-tasks/)的工作方式。


[ServiceDefinition.csdef]: /documentation/articles/cloud-services-model-and-package/#csdef
[Task]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Task
[Runtime]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Runtime
[Startup]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Startup
[Runtime]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Runtime
[Environment]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Environment
[Variable]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Variable
[RoleInstanceValue]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#RoleInstanceValue
[RoleEnvironment]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.aspx
[Endpoints]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Endpoints
[LocalStorage]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#LocalStorage
[LocalResources]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#LocalResources
[RoleInstanceValue]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#RoleInstanceValue

<!---HONumber=Mooncake_0104_2016-->
