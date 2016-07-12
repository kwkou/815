<properties 
pageTitle="在 Azure 云服务中运行启动任务 | Azure" 
description="启动任务可帮助为你的应用准备云服务环境。这将讲授启动任务的工作方式以及如何生成启动任务" 
services="cloud-services" 
documentationCenter="" 
authors="Thraka" 
manager="timlt" 
editor=""/>
<tags 
	ms.service="cloud-services" 
	ms.date="12/07/2015" 
	wacn.date="01/15/2016"/>



# 如何配置和运行云服务的启动任务

在角色启动之前，可以使用启动任务执行操作。你可能需要执行的操作包括安装组件、注册 COM 组件、设置注册表项或启动长时间运行的进程。

>[AZURE.NOTE]启动任务不适用于虚拟机，只适用于云服务 Web 角色和辅助角色。

## 启动任务的工作方式

启动任务是在角色开始之前执行的操作，并在 [ServiceDefinition.csdef] 文件中定义（通过使用 [Startup] 元素内的 [Task] 元素）。启动任务通常是批处理文件，但它们也可以是控制台应用程序或启动 PowerShell 脚本的批处理文件。

环境变量将信息传递给启动任务，而本地存储可用于从启动任务中传出信息。例如，环境变量可以指定你要安装的程序的路径，并可以将文件写入到本地存储，然后你的角色可以稍后读取这些文件。

启动任务可以将信息和错误记录到 **TEMP** 环境变量指定的目录。在云中运行时，在启动任务期间，**TEMP** 环境变量将解析为 *C:\\Resources\\temp\\[guid].[rolename]\\RoleTemp* 目录。

此外，启动任务还可以在重新启动之间执行多次。例如，每次角色回收时都会运行启动任务，但角色回收可能并非始终包括重新启动。应以这样的方式编写启动任务：使其能够多次运行而不会出现问题。

启动任务必须以为零的 **errorlevel**（或退出代码）结束，才能完成启动过程。如果启动任务以非零的 **errorlevel** 结束，则角色将不会启动。


## 角色启动顺序

下面列出了 Azure 中的角色启动过程：

1. 实例将标记为“正在启动”并且不接收流量。

2. 所有启动任务均根据其 **taskType** 属性执行。
    - **simple** 任务以同步方式执行（一次一个任务）。
    - **background** 和 **foreground** 任务与启动任务并行，以异步方式启动。  
       
    > [AZURE.WARNING]在启动过程中的启动任务阶段，IIS 可能未完全配置，因此角色特定的数据可能不可用。需要角色特定的数据的启动任务应使用 [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.OnStart](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx)。

3. 将启动角色主机进程并在 IIS 中创建站点。

4. 将调用 [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.OnStart](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx) 方法。

5. 实例将标记为“准备就绪”，并且流量将路由到实例。

6. 将调用 [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.Run](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法。


## 启动任务的示例

启动任务在 [ServiceDefinition.csdef] 文件的 **Task** 元素中定义。**commandLine** 属性指定启动批处理文件或控制台命令的名称和参数，**executionContext** 属性指定启动任务的权限级别，**taskType** 属性指定将如何执行该任务。

在此示例中，将为启动任务创建环境变量 **MyVersionNumber**，并将该变量设为值**“1.0.0.0”**。

**ServiceDefinition.csdef**：

```xml
<Startup>
    <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" >
        <Environment>
            <Variable name="MyVersionNumber" value="1.0.0.0" />
        </Environment>
    </Task>
</Startup>
```

在下面的示例中，**Startup.cmd** 批处理文件会将行“当前版本是 1.0.0.0”写入到由 TEMP 环境变量指定的目录下的 StartupLog.txt 文件中。`EXIT /B 0` 行确保启动任务以为零的 **errorlevel** 结尾。

```cmd
ECHO The current version is %MyVersionNumber% >> "%TEMP%\StartupLog.txt" 2>&1
EXIT /B 0
```

> [AZURE.NOTE]在 Visual Studio 中，启动批处理文件的“复制到输出目录”属性应设为“始终复制”，以确保将启动批处理文件正确地部署到 Azure 上你的项目（对于 Web 角色，为 **approot\\bin**；对于辅助角色，为 **approot**）。

## 任务属性的说明

下面介绍 [ServiceDefinition.csdef] 文件中的 **Task** 元素的属性：

**commandLine** - 为启动任务指定命令行：

- 该命令具有可选的命令行参数，用于开始启动任务。
- 通常它是 .cmd 或 .bat 批处理文件的文件名。
- 该任务相对于部署的 AppRoot\\Bin 文件夹。在确定任务的路径和文件时不扩展环境变量。如果需要环境扩展，则可以创建用于调用启动任务的小型 .cmd 脚本。
- 可以是一个启动 [PowerShell 脚本](/documentation/articles/cloud-services-startup-tasks-common/#create-a-powershell-startup-task)的控制台应用程序或批处理文件。

**executionContext** - 为启动任务指定权限级别。权限级别可以为 limited 或 elevated：

- **limited**  
启动任务以与角色相同的权限运行。当 [Runtime] 元素的 **executionContext** 属性也是 **limited** 时，则使用用户权限。

- **elevated**  
启动任务以管理员特权运行。这将允许启动任务安装程序、更改 IIS 配置、执行注册表更改和其他管理员级别任务，而不会提高角色本身的权限级别。

> [AZURE.NOTE]启动任务的权限级别不需要与角色本身相同。

**taskType** - 指定启动任务的执行方式。

- **simple**  
任务按照 [ServiceDefinition.csdef] 文件中指定的顺序一次一个地以同步方式执行。当一个 **simple** 启动任务以为零的 **errorlevel** 结束时，将执行下一个 **simple** 启动任务。如果没有更多 **simple** 启动任务要执行，则将启动角色本身。   

    > [AZURE.NOTE]如果 **simple** 任务以非零 **errorlevel** 结束，则将阻止该实例。后续 **simple** 启动任务和角色本身将不会启动。

    若要确保你的批处理文件以为零的 **errorlevel** 结束，请在在批处理文件进程结束时执行命令 `EXIT /B 0`。

- **background**  
任务与角色同时启动，并以异步方式执行。

- **foreground**  
任务与角色同时启动，并以异步方式执行。**foreground** 任务与 **background** 任务之间的主要区别在于 **foreground** 任务阻止角色回收或关闭，直到任务结束。**background** 任务没有此限制。

## 环境变量

环境变量是一种将信息传递给启动任务的方法。例如，可以放置一个 blob 的路径，该 blob 包含要安装的程序或你的角色将使用的端口号或用于控制启动任务的功能的设置。

启动任务有两种类型的环境变量；静态环境变量和基于 [RoleEnvironment] 类的成员的环境变量。这两种环境变量都在 [ServiceDefinition.csdef] 文件的 [Environment] 节中，并且都使用 [Variable] 元素和 **name** 属性。

静态环境变量使用 [Variable] 元素的 **value** 属性。上面的示例创建了环境变量 **MyVersionNumber**，该变量具有静态值 **1.0.0.0**。另一个示例就是创建 **StagingOrProduction** 环境变量，你可以手动将该变量设置为值 **staging** 或 **production**，以根据 **StagingOrProduction** 环境变量的值执行不同的启动操作。

基于 RoleEnvironment 类的成员的环境变量不使用 [Variable] 元素的 **value** 属性。而是使用具有相应 **xPath** 属性值的 [RoleInstanceValue] 子元素基于 [RoleEnvironment] 类的特定成员创建环境变量。用于访问各种 [RoleEnvironment] 值的 **xPath** 属性的值可以在 [Azure 中的 xPath 值](https://msdn.microsoft.com/zh-cn/library/azure/hh404006.aspx)中找到。



例如，若要创建这样一个环境变量（当实例在计算模拟器中运行时为**“true”**，在云中运行时为**“false”**），请使用以下 [Variable] 和 [RoleInstanceValue] 元素：


	<Startup>
	    <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
	        <Environment>
	    
	            <!-- Create the environment variable that informs the startup task whether it is running
	                in the Compute Emulator or in the cloud. "%ComputeEmulatorRunning%"=="true" when
	                running in the Compute Emulator, "%ComputeEmulatorRunning%"=="false" when running
	                in the cloud. -->
	    
	            <Variable name="ComputeEmulatorRunning">
	                <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
	            </Variable>
	    
	        </Environment>
	    </Task>
	</Startup>


## 后续步骤
了解如何使用云服务执行一些[常见的启动任务](/documentation/articles/cloud-services-startup-tasks-common/)。

[打包](/documentation/articles/cloud-services-model-and-package/)你的云服务。


[ServiceDefinition.csdef]: /documentation/articles/cloud-services-model-and-package/#csdef
[Task]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Task
[Startup]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Startup
[Runtime]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Runtime
[Environment]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Environment
[Variable]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#Variable
[RoleInstanceValue]: https://msdn.microsoft.com/zh-cn/library/azure/gg557552.aspx#RoleInstanceValue
[RoleEnvironment]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.aspx

<!---HONumber=Mooncake_0104_2016-->
