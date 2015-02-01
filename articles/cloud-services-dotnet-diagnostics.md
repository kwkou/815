<properties linkid="dev-net-commons-tasks-diagnostics" urlDisplayName="Diagnostics" pageTitle="如何使用诊断 (.NET) - Azure 功能指南" metaKeywords="Azure diagnostics monitoring,logs crash dumps C#" description="了解如何在 Azure 中使用诊断数据进行调试、度量性能、进行监视以及流量分析等等。" metaCanonical="" services="cloud-services" documentationCenter=".NET" title="Enabling Diagnostics in Azure" authors="ryanwi" solutions="" manager="" editor="" />




# 在 Azure 云服务和虚拟机中启用 Diagnostics

通过 Azure Diagnostics 1.2 和 1.3，可以从 Azure 中运行的辅助角色、Web 角色或虚拟机收集诊断数据。本指南介绍如何使用 Azure Diagnostics 1.2 和 1.3。有关创建日志记录和跟踪策略以及使用诊断和其他技术排查问题的其他深入指南，请参阅[有关开发 Azure 应用程序的故障排除最佳实践][]。

## 目录

-   [概述][]
-   [如何在辅助角色中启用 Diagnostics][]
-   [如何在虚拟机中启用 Diagnostics][]
-   [示例配置文件和架构][]
-   [故障排除][]
-   [常见问题][]
-   [比较 Azure Diagnostics 版本][]
-   [其他资源][]

##<a name="overview"></a>概述

Azure Diagnostics 1.2 和 1.3 是可让你从 Azure 中运行的辅助角色、Web 角色或虚拟机收集诊断数据的 Azure 扩展。遥测数据存储在 Azure 存储帐户中，可用于调试和故障排除、测量性能、监视资源使用情况、流量分析和容量规划以及审核。 

Azure Diagnostics 1.2 可与适用于 .NET 2.4 及更低版本的 Azure SDK 配合使用。Azure Diagnostics 1.3 可与适用于 .NET 2.5 及更低版本的 Azure SDK 配合使用。

如果你以前用过 Diagnostics 版本 1.0，应该知道，它与 Diagnostics 1.2 和 1.3 相比具有明显的差异：

1.	除了可以部署在云服务中以外，Diagnostics 1.2 和 1.3 还可以部署在虚拟机上。
2.	Diagnostics 1.0 是 Azure SDK 的一部分，在部署云服务时进行部署。诊断 1.2 和 1.3 是扩展，通过云服务部署单独进行部署。
3.	诊断 1.2 和 1.3 启用 ETW 和 .NET EventSource 事件的集合。

有关更详细的比较，请参阅本文末尾的[比较 Azure Diagnostics 版本][]。

Azure 诊断可以收集以下类型的遥测数据：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
	<tr>
			<td style="width: 100px;"><strong>数据源</strong></td>
			<td><strong>说明</strong></td>
	</tr>
	<tr>
		<td>IIS Logs</td>
		<td>有关 IIS 网站的信息。</td>            
	</tr>
	<tr>
		<td>Azure 诊断基础结构日志</td>
		<td>有关 Diagnostics 自身的信息。</td>            
	</tr>
	<tr>
		<td>IIS 失败请求日志	</td>
		<td>有关 IIS 站点或应用程序的失败请求的信息。</td>            
	</tr>
	<tr>
		<td>Windows 事件日志</td>
		<td>发送到 Windows 事件日志记录系统的信息。</td>            
	</tr>
	<tr>
		<td>性能计数器</td>
		<td>操作系统和自定义性能计数器。</td>            
	</tr>
	<tr>
		<td>故障转储</td>
		<td>有关应用程序崩溃时进程状态的信息。</td>            
	</tr>
	<tr>
		<td>自定义错误日志</td>
		<td>应用程序或服务创建的日志。</td>            
	</tr>
	<tr>
		<td>.NET EventSource</td>
		<td>使用 .NET 的代码生成的事件 <a href="http://msdn.microsoft.com/zh-cn/library/system.diagnostics.tracing.eventsource(v=vs.110).aspx">EventSource class</a>.</td>            
	</tr>
	<tr>
		<td>基于清单的 ETW</td>
		<td>任何进程生成的 ETW 事件。</td>            
	</tr>
		
</tbody>
</table>

##<a name="worker-role"></a>如何在辅助角色中启用 Diagnostics

本演练介绍如何实现使用 .NET EventSource 类发出遥测数据的 Azure 辅助角色。Azure Diagnostics 用于收集遥测数据，并将其存储在一个 Azure 存储帐户中。创建辅助角色时，Visual Studio 将在适用于 .NET 2.4 和更低版本的 Azure SDK 中，自动启用 Diagnostics 1.0 作为解决方案的一部分。以下说明介绍了创建辅助角色、从解决方案禁用 Diagnostics 1.0，以及在辅助角色中部署 Diagnostics 1.2 或 1.3 的过程。

###先决条件
本文假定你具有 Azure 订阅，并将 Visual Studio 2013 与  Azure SDK 结合使用。如果你没有 Azure 订阅，你可以注册[免费试用版][]。请确保[安装并配置 Azure PowerShell 0.8.7 或更高版本][]。

###步骤 1：创建辅助角色###
1.	启动 Visual Studio 2013。
2.	从面向 .NET Framework 4.5 的云模板创建一个新的 Microsoft Azure 云服务项目。将该项目命名为"WadExample"。
3.	选择"辅助角色"。
4.	在解决方案资源管理器，双击 WorkerRole1 properties 文件。
5.	在"配置"选项卡中，取消选中"启用 Diagnostics"以禁用 Diagnostics 1.0（Azure SDK 2.4 和更低版本）。
6.	生成你的解决方案，以确认不会出错。

###步骤 2：检测代码###
将 WorkerRole.cs 的内容替换为以下代码。继承自 [EventSource 类][]的 SampleEventSourceWriter 类实现四个日志记录方法：SendEnums、MessageMethod、SetOther 和 HighFreq。WriteEvent 方法的第一个参数定义相关事件的 ID。Run 方法实现一个无限循环，该循环每隔 10 秒调用 SampleEventSourceWriter 类中实现的每个日志记录方法。

	using Microsoft.WindowsAzure.ServiceRuntime;
	using System;
	using System.Diagnostics;
	using System.Diagnostics.Tracing;
	using System.Net;
	using System.Threading;

	namespace WorkerRole1
	{
    sealed class SampleEventSourceWriter : EventSource
    {
        public static SampleEventSourceWriter Log = new SampleEventSourceWriter();
        public void SendEnums(MyColor color, MyFlags flags) { if (IsEnabled())  WriteEvent(1, (int)color, (int)flags); }// Cast enums to int for efficient logging.
        public void MessageMethod(string Message) { if (IsEnabled())  WriteEvent(2, Message); }
        public void SetOther(bool flag, int myInt) { if (IsEnabled())  WriteEvent(3, flag, myInt); }
        public void HighFreq(int value) { if (IsEnabled()) WriteEvent(4, value); }

    }

    enum MyColor
    {
        Red,
        Blue,
        Green
    }

    [Flags]
    enum MyFlags
    {
        Flag1 = 1,
        Flag2 = 2,
        Flag3 = 4
    }

    public class WorkerRole : RoleEntryPoint
    {
        public override void Run()
        {
            // This is a sample worker implementation. Replace with your logic.
            Trace.TraceInformation("WorkerRole1 entry point called");

            int value = 0;

            while (true)
            {
                Thread.Sleep(10000);
                Trace.TraceInformation("Working");

                // Emit several events every time we go through the loop
                for (int i = 0; i < 6; i++)
                {
                    SampleEventSourceWriter.Log.SendEnums(MyColor.Blue, MyFlags.Flag2 | MyFlags.Flag3);
                }

                for (int i = 0; i < 3; i++)
                {
                    SampleEventSourceWriter.Log.MessageMethod("This is a message.");
                    SampleEventSourceWriter.Log.SetOther(true, 123456789);
                }

                if (value == int.MaxValue) value = 0;
                SampleEventSourceWriter.Log.HighFreq(value++);
            }
        }

        public override bool OnStart()
        {
            // Set the maximum number of concurrent connections 
            ServicePointManager.DefaultConnectionLimit = 12;

            // For information on handling configuration changes
            // see the MSDN topic at http://msdn.microsoft.com/zh-cn/library/hh180152.aspx.

            return base.OnStart();
        }
    }
	}


###步骤 3：部署辅助角色###
1.	从 Visual Studio 中选择 WadExample 项目，然后从"生成"菜单中选择"发布"，以将辅助角色部署到 Azure。
2.	选择你的订阅。
3.	在"Microsoft Azure 发布设置"对话框中，选择"<新建...>"。
4.	在"创建云服务和存储帐户"对话框中输入一个名称（例如"WadExample"），然后选择区域或地缘组。
5.	将"环境"设置为"过渡"。
6.	适当地修改任何其他设置，然后单击"发布"。
7.	完成部署后，在 Azure 门户中验证你的云服务是否处于"正在运行"状态。

###步骤 4：创建 Diagnostics 配置文件并安装扩展###
1.	通过执行以下 PowerShell 命令来下载公共配置文件架构定义：
2.	
		(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File -Encoding utf8 -FilePath 'WadConfig.xsd' 

2.	右键单击 WorkerRole1 项目并选择"添加"->"新建项..."->"Visual C# 项"->"数据"->"XML 文件"，将 XML 文件添加到 WorkerRole1 项目中。将该文件命名为"WadExample.xml"。

	![CloudServices_diag_add_xml](./media/cloud-services-dotnet-diagnostics/AddXmlFile.png)

3.	将 WadConfig.xsd 与配置文件相关联。确保 WadExample.xml 编辑器窗口是活动的窗口。按 F4 打开"属性"窗口。在"属性"窗口中单击"架构"属性。在"架构"属性中单击"..."。单击"添加..."按钮并导航到 XSD 文件的保存位置，然后选择文件 WadConfig.xsd。单击"确定"。
4.	将 WadExample.xml 配置文件的内容替换为以下 XML 并保存该文件。此配置文件定义两个要收集的性能计数器：一个对应于 CPU 使用率，另一个对应于内存使用率。配置将定义对应于 SampleEventSourceWriter 类中方法的四个事件。

		<?xml version="1.0" encoding="utf-8"?>
		<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  			<WadCfg>
    			<DiagnosticMonitorConfiguration overallQuotaInMB="25000">
      			<PerformanceCounters scheduledTransferPeriod="PT1M">
        			<PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />
        			<PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT1M" unit="bytes"/>
      				</PerformanceCounters>
      				<EtwProviders>
        				<EtwEventSourceProviderConfiguration provider="SampleEventSourceWriter" scheduledTransferPeriod="PT5M">
          					<Event id="1" eventDestination="EnumsTable"/>
          					<Event id="2" eventDestination="MessageTable"/>
          					<Event id="3" eventDestination="SetOtherTable"/>
          					<Event id="4" eventDestination="HighFreqTable"/>
          					<DefaultEvents eventDestination="DefaultTable" />
        				</EtwEventSourceProviderConfiguration>
      				</EtwProviders>
    			</DiagnosticMonitorConfiguration>
  			</WadCfg>
		</PublicConfig>

###步骤 5：在辅助角色上安装 Diagnostics###
用于在 Web 或辅助角色上管理 Diagnostics 的 PowerShell cmdlet 为：Set-AzureServiceDiagnosticsExtension、Get-AzureServiceDiagnosticsExtension 和 Remove-AzureServiceDiagnosticsExtension。

1.	打开 Windows Azure PowerShell。
2.	执行脚本以在辅助角色上安装 Diagnostics（将 *StorageAccountKey* 替换为 wadexample 存储帐户的存储帐户密钥）：

		$storage_name = "wadexample"
		$key = "<StorageAccountKey>"
		$config_path="c:\users\<user>\documents\visual studio 2013\Projects\WadExample\WorkerRole1\WadExample.xml"
		$service_name="wadexample"
		$storageContext = New-AzureStorageContext -StorageAccountName $storage_name -StorageAccountKey $key 
		Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext -DiagnosticsConfigurationPath $config_path -ServiceName $service_name -Slot Staging -Role WorkerRole1


###步骤 6：查看遥测数据###
在 Visual Studio 服务器资源管理器中，导航到 wadexample 存储帐户。在云服务大约运行 5 分钟后，你应该会看到表 WADEnumsTable、WADHighFreqTable、WADMessageTable、WADPerformanceCountersTable 和 WADSetOtherTable。双击其中一个表即可查看已收集的遥测数据。
	![CloudServices_diag_tables](./media/cloud-services-dotnet-diagnostics/WadExampleTables.png)

##<a name="virtual-machine"></a>如何在虚拟机中启用 Diagnostics##

本演练介绍如何从开发计算机将 Diagnostics 远程安装到 Azure 虚拟机。你还将了解如何实施在该 Azure 虚拟机上运行的应用程序，并使用 .NET [EventSource 类][]发出遥测数据。Azure Diagnostics 用于收集遥测数据，并将其存储在一个 Azure 存储帐户中。

###先决条件###
本演练假定你具有 Azure 订阅，并将 Visual Studio 2013 与  Azure SDK 结合使用。如果你没有 Azure 订阅，你可以注册[免费试用版][]。请确保[安装并配置 Azure PowerShell 0.8.7 或更高版本][]。

###步骤 1：创建虚拟机###
1.	在开发计算机上启动 Visual Studio 2013。
2.	在 Visual Studio 服务器资源管理器中，展开"Azure"，右键单击"虚拟机"然后选择"创建虚拟机"。
3.	在"选择订阅"对话框中选择你的 Azure 订阅，然后单击"下一步"。
4.	在"选择虚拟机映像"对话框中选择"Windows Server 2012 R2 Datacenter"，然后单击"下一步"。
5.	在"虚拟机基本设置"中，将虚拟机名称设置为"wadexample"。设置管理员用户名和密码，然后单击"下一步"。
6.	在"云服务设置"对话框中，创建名为"wadexampleVM"的新云服务。创建一个名为"wadexample"的新存储帐户，然后单击"下一步"。
7.	单击"创建"。

###步骤 2：创建应用程序###
1.	在开发计算机上启动 Visual Studio 2013。
2.	创建面向 .NET Framework 4.5 的新 Visual C# 控制台应用程序。将该项目命名为"WadExampleVM"。
	![CloudServices_diag_new_project](./media/cloud-services-dotnet-diagnostics/NewProject.png)
3.	将 Program.cs 的内容替换为以下代码。类 SampleEventSourceWriter 实现四个日志记录方法：SendEnums、MessageMethod、SetOther 和 HighFreq。WriteEvent 方法的第一个参数定义相关事件的 ID。Run 方法实现一个无限循环，该循环每隔 10 秒调用 SampleEventSourceWriter 类中实现的每个日志记录方法。

		using System;
		using System.Diagnostics;
		using System.Diagnostics.Tracing;
		using System.Threading;

		namespace WadExampleVM
		{
    	sealed class SampleEventSourceWriter : EventSource
    	{
        	public static SampleEventSourceWriter Log = new SampleEventSourceWriter();
        	public void SendEnums(MyColor color, MyFlags flags) { if (IsEnabled())  WriteEvent(1, (int)color, (int)flags); }// Cast enums to int for efficient logging.
        	public void MessageMethod(string Message) { if (IsEnabled())  WriteEvent(2, Message); }
        	public void SetOther(bool flag, int myInt) { if (IsEnabled())  WriteEvent(3, flag, myInt); }
        	public void HighFreq(int value) { if (IsEnabled()) WriteEvent(4, value); }

    	}

    	enum MyColor
    	{
        	Red,
        	Blue,
        	Green
    	}

    	[Flags]
    	enum MyFlags
    	{
        	Flag1 = 1,
        	Flag2 = 2,
        	Flag3 = 4
    	}

    	class Program
    	{
        static void Main(string[] args)
        {
            Trace.TraceInformation("My application entry point called");
            
            int value = 0;

            while (true)
            {
                Thread.Sleep(10000);
                Trace.TraceInformation("Working");

                // Emit several events every time we go through the loop
                for (int i = 0; i < 6; i++)
                {
                    SampleEventSourceWriter.Log.SendEnums(MyColor.Blue, MyFlags.Flag2 | MyFlags.Flag3);
                }

                for (int i = 0; i < 3; i++)
                {
                    SampleEventSourceWriter.Log.MessageMethod("This is a message.");
                    SampleEventSourceWriter.Log.SetOther(true, 123456789);
                }

                if (value == int.MaxValue) value = 0;
                SampleEventSourceWriter.Log.HighFreq(value++);
            }

        }
    	}
		}


4.	保存该文件，然后从"生成"菜单中选择"生成解决方案"以生成代码。


###步骤 3：部署应用程序###
1.	在解决方案资源管理器中右键单击"WadExampleVM"项目，然后在文件资源管理器中选择"打开文件夹"。
2.	导航到 *bin\Debug* 文件夹，并复制所有文件 (WadExampleVM.*)
3.	在服务器资源管理器中，右键单击"虚拟机"，并选择"使用远程桌面连接"。
4.	连接到 VM 后，创建名为 WadExampleVM 的文件夹，并将应用程序文件粘贴到该文件夹中。
5.	启动应用程序 WadExampleVM.exe。你应会看到一个空白控制台窗口。

###步骤 4：创建 Diagnostics 配置并安装扩展###
1.	通过执行以下 PowerShell 命令，将公共配置文件架构定义下载到开发计算机：

		(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File -Encoding utf8 -FilePath 'WadConfig.xsd' 

2.	在 Visual Studio 中打开一个新的 XML 文件：可以在已打开的项目中，或者在未打开项目的 Visual Studio 实例中执行此操作。在 Visual Studio 中，选择"添加"->"新建项..."->"Visual C# 项"->"数据"->"XML 文件"。将该文件命名为"WadExample.xml"
3.	将 WadConfig.xsd 与配置文件相关联。确保 WadExample.xml 编辑器窗口是活动的窗口。按 F4 打开"属性"窗口。在"属性"窗口中单击"架构"属性。在"架构"属性中单击"..."。单击"添加..."按钮并导航到 XSD 文件的保存位置，然后选择文件 WadConfig.xsd。单击"确定"。
4.	将 WadExample.xml 配置文件的内容替换为以下 XML 并保存该文件。此配置文件定义两个要收集的性能计数器：一个对应于 CPU 使用率，另一个对应于内存使用率。配置将定义对应于 SampleEventSourceWriter 类中方法的四个事件。

		<?xml version="1.0" encoding="utf-8"?>
		<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  			<WadCfg>
    			<DiagnosticMonitorConfiguration overallQuotaInMB="25000">
      			<PerformanceCounters scheduledTransferPeriod="PT1M">
        			<PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />
        			<PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT1M" unit="bytes"/>
      				</PerformanceCounters>
      				<EtwProviders>
        				<EtwEventSourceProviderConfiguration provider="SampleEventSourceWriter" scheduledTransferPeriod="PT5M">
          					<Event id="1" eventDestination="EnumsTable"/>
          					<Event id="2" eventDestination="MessageTable"/>
          					<Event id="3" eventDestination="SetOtherTable"/>
          					<Event id="4" eventDestination="HighFreqTable"/>
          					<DefaultEvents eventDestination="DefaultTable" />
        				</EtwEventSourceProviderConfiguration>
      				</EtwProviders>
    			</DiagnosticMonitorConfiguration>
  			</WadCfg>
		</PublicConfig>


###步骤 5：将 Diagnostics 远程安装到 Azure 虚拟机上 ###
用于在 VM 上管理 Diagnostics 的 PowerShell cmdlet 为：Set-AzureVMDiagnosticsExtension、Get-AzureVMDiagnosticsExtension 和 Remove-AzureVMDiagnosticsExtension。

1.	在开发人员计算机上，打开 Microsoft Azure PowerShell。
2.	执行脚本以在 VM 上远程安装 Diagnostics（将 *StorageAccountKey* 替换为 wadexamplevm 存储帐户的存储帐户密钥）：

		$storage_name = "wadexamplevm"
		$key = "<StorageAccountKey>"
		$config_path="c:\users\<user>\documents\visual studio 2013\Projects\WadExampleVM\WadExampleVM\WadExample.xml"
		$service_name="wadexamplevm"
		$vm_name="WadExample"
		$storageContext = New-AzureStorageContext -StorageAccountName $storage_name -StorageAccountKey $key 
		$VM1 = Get-AzureVM -ServiceName $service_name -Name $vm_name
		$VM2 = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $config_path -Version "1.*" -VM $VM1 -StorageContext $storageContext
		$VM3 = Update-AzureVM -ServiceName $service_name -Name $vm_name -VM $VM2.VM


###步骤 6：查看遥测数据###
在 Visual Studio 服务器资源管理器中，导航到 wadexample 存储帐户。在 VM 大约运行 5 分钟后，你应该会看到表 WADEnumsTable、WADHighFreqTable、WADMessageTable、WADPerformanceCountersTable 和 WADSetOtherTable。双击其中一个表即可查看已收集的遥测数据。
	![CloudServices_diag_wadexamplevm_tables](./media/cloud-services-dotnet-diagnostics/WadExampleVMTables.png)

##<a name="configuration-file-schema"></a>配置文件架构##

Diagnostics 配置文件定义启动诊断监视器时用于初始化诊断配置设置的值。以下位置提供了一个示例配置文件以及有关其架构的详细文档：[Azure Diagnostics 1.2 配置架构][]。

##<a name="troubleshooting"></a>故障排除##

###Azure Diagnostics 不启动###
Diagnostics 由两个组件构成：来宾代理插件和监视代理。来宾代理插件的日志文件位于以下文件中： 

*%SystemDrive%\ WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<DiagnosticsVersion>*\CommandExecution.log

该插件会返回以下错误代码：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
	<tr>
			<td style="width: 100px;"><strong>退出代码</strong></td>
			<td><strong>说明</strong></td>
	</tr>
    <tr>
		<td>0</td>
		<td>成功。</td>            
	</tr>
    <tr>
		<td>-1</td>
        <td>常规错误。</td>		            
	</tr>
    <tr>
		<td>-2</td>
        <td><p>无法加载 rcf 文件。</p>
<p>这是一个内部错误，仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生该错误。</p></td>		            
	</tr>
    <tr>
		<td>-3</td>
        <td><p>无法加载 Diagnostics 配置文件。</p>
<p>解决方案：这是配置文件未通过架构验证的结果。解决方案是提供符合架构的配置文件。</p></td>		            
	</tr>
    <tr>
		<td>-4</td>
        <td><p>监视代理 Diagnostics 的另一个实例已在使用本地资源目录。</p>
<p>解决方案：为 <strong>LocalResourceDirectory</strong>指定不同的值。</p></td>		            
	</tr>
    <tr>
		<td>-6</td>
        <td><p>来宾代理插件启动器尝试使用无效的命令行启动 Diagnostics。</p>
<p>这是一个内部错误，仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生该错误。</p></td>		            
	</tr>
    <tr>
		<td>-10</td>
        <td>Diagnostics 插件退出并返回未处理的异常。</td>		            
	</tr>
    <tr>
		<td>-11</td>
        <td><p>来宾代理程序无法创建负责启动和监视监视代理的进程。</p>

<p>解决方案：验证是否有足够的系统资源用于启动新进程。</p></td>		            
	</tr>
    <tr>
		<td>-101</td>
        <td><p>调用 Diagnostics 插件时参数无效。</p>

<p>这是一个内部错误，仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生该错误。</p></td>		            
	</tr>
    <tr>
		<td>-102</td>
        <td><p>插件进程将无法初始化自身。</p> 

<p>解决方案：验证是否有足够的系统资源用于启动新进程。</p></td>		            
	</tr>
    <tr>
		<td>-103</td>
        <td><p>插件进程将无法初始化自身。具体而言，它无法创建记录器对象。</p>

<p>解决方案：验证是否有足够的系统资源用于启动新进程。</p></td>		            
	</tr>
    <tr>
		<td>-104</td>
        <td><p>无法加载来宾代理提供的 rcf 文件。</p>

<p>这是一个内部错误，仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生该错误。</p></td>		            
	</tr>
    <tr>
		<td>-105</td>
        <td><p>Diagnostics 插件无法打开 Diagnostics 配置文件。</p>

<p>这是一个内部错误，仅当在 VM 上不正确地手动调用了 Diagnostics 插件时才会发生该错误。</p></td>		            
	</tr>
    <tr>
		<td>-106</td>
        <td><p>无法读取 Diagnostics 配置文件。</p>

<p>解决方案：这是配置文件未通过架构验证的结果。因此，解决方案是提供符合架构的配置文件。可以在 VM 上的文件夹 <i>%SystemDrive%\WindowsAzure\Config</i> 中，找到提供给 Diagnostics 扩展的 XML。打开相应的 XML 文件并搜索 <strong>Microsoft.Azure.Diagnostics</strong>，然后搜索 <strong>xmlCfg</strong> 字段。数据采用 base64 编码，因此你需要 <a href="http://www.bing.com/search?q=base64+decoder">将其解码，</a> 以查看 Diagnostics 加载的 XML。</p></td>		            
	</tr>
    <tr>
		<td>-107</td>
        <td><p>传递给监视代理的资源目录无效。</p>

<p>这是一个内部错误，仅当在 VM 上不正确地手动调用了监视代理时才会发生该错误。</p></td>		            
	</tr>
    <tr>
		<td>-108	</td>
        <td><p>无法将 Diagnostics 配置文件转换为监视代理配置文件。</p>

<p>这是一个内部错误，仅当使用无效的配置文件手动调用了 Diagnostics 插件时才会发生该错误。</p></td>		            
	</tr>
    <tr>
		<td>-110</td>
        <td><p>常规 Diagnostics 配置错误。</p>

<p>这是一个内部错误，仅当使用无效的配置文件手动调用了 Diagnostics 插件时才会发生该错误。</p></td>		            
	</tr>
    <tr>
		<td>-111</td>
        <td><p>无法启动监视代理。</p>

<p>解决方案：验证是否提供了足够的系统资源。</p></td>		            
	</tr>
    <tr>
		<td>-112</td>
        <td>常规错误</td>		            
	</tr>    
</tbody>
</table>

###未将 Diagnostics 数据记录到存储中###
丢失事件数据的最常见原因是错误地定义了存储帐户信息。 

解决方案：更正 Diagnostics 配置文件，然后重新安装 Diagnostics。
事件数据在上载到存储帐户之前，会存储在文件夹中。有关 LocalResourceDirectory 的详细信息，请参阅上文。

如果此文件夹中没有任何文件，监视代理将无法启动。这通常是由于配置文件无效而导致的，应该已在 CommandExecution.log 中报告。如果监视代理已成功收集事件数据，你将会看到配置文件中为每个事件定义的 .tsf 文件。

监视代理将在文件 MaEventTable.tsf 中记录它所遇到的任何错误。若要检查此文件的内容，请运行以下命令：

		%SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.[IaaS | PaaS]Diagnostics\1.3.0.0\Monitor\x64\table2csv maeventtable.tsf

该工具将生成一个名为 maeventtable.csv 的文件，你可以打开并检查日志，以确定失败的原因。


##<a name="faq"></a>常见问题##
下面是一些常见问题及其回答：

问：如何将我的 Visual Studio 解决方案从 Azure Diagnostics 1.0 升级到 Azure Diagnostics 1.1？

答：将 Visual Studio 解决方案从 Diagnostics 1.0 升级到 Diagnostics 1.1（或更高版本）是一个手动过程：
- 在 Visual Studio 解决方案中禁用 Diagnostics，以阻止随角色一起部署 Diagnostics 1.0
- 如果代码使用了跟踪侦听器，则你需要将代码修改为使用 .NET EventSource。Diagnostics 1.1 和更高版本不支持跟踪侦听器。
- 修改部署过程以安装 Diagnostics 1.1 扩展

问：如果我已在角色或 VM 上安装了 Diagnostics 1.1 扩展，如何升级到 Diagnostics 1.2 或 1.3？

答：如果你在安装 Diagnostics 1.1 时指定了"Version "1.*""，则下一次重新启动角色或 VM 时，Diagnostics 会自动更新到与正则表达式 "1.*" 匹配的最新版本。如果你在安装 Diagnostics 1.1 时指定了"-Version "1.1""，可以通过重新执行 Set- cmdlet 并指定想要安装的版本，来更新到更高的版本。

问：如何为表命名？

答：根据以下约定为表命名：

		if (String.IsNullOrEmpty(eventDestination)) {
		    if (e == "DefaultEvents")
		        tableName = "WADDefault" + MD5(provider);
		    else
		        tableName = "WADEvent" + MD5(provider) + eventId;
		}
		else
		    tableName = "WAD" + eventDestination;

下面是一个示例：

		<EtwEventSourceProviderConfiguration provider="prov1">
		  <Event id="1" />
		  <Event id="2" eventDestination="dest1" />
		  <DefaultEvents />
		</EtwEventSourceProviderConfiguration>
		<EtwEventSourceProviderConfiguration provider="prov2">
		  <DefaultEvents eventDestination="dest2" />
		</EtwEventSourceProviderConfiguration>

这会生成 4 个表：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
	<tr>
			<td style="width: 100px;"><strong>事件</strong></td>
			<td><strong>表名称</strong></td>			
	</tr>
	<tr>
			<td>provider="prov1" &lt;Event id="1" /&gt;</td>
			<td>WADEvent+MD5("prov1")+"1"</td>			
	</tr>
	<tr>
			<td>provider="prov1" &lt;Event id="2" eventDestination="dest1" /&gt;</td>
			<td>WADdest1</td>			
	</tr>
	<tr>
			<td>provider="prov1" &lt;DefaultEvents /&gt;</td>
			<td>WADDefault+MD5("prov1")</td>			
	</tr>
	<tr>
			<td>provider="prov2" &lt;DefaultEvents eventDestination="dest2" /&gt;</td>
			<td>WADdest2</td>			
	</tr>
	

</table>
</tbody>

##<a name="comparing"></a>比较 Azure Diagnostics 版本##

下表比较了 Azure Diagnostics 版本 1.0 和 1.1/1.2/1.3 支持的功能：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
	<tr>
			<td style="width: 100px;"><strong>支持的角色类型</strong></td>
			<td><strong>Diagnostics 1.0</strong></td>
			<td><strong>Diagnostics 1.1/1.2/1.3</strong></td>
	</tr>

	<tr>
			<td>Web 角色</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>辅助角色</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>IaaS</td>
			<td>否</td>
			<td>是</td>
	</tr>
</tbody>
</table>

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
	<tr>
			<td style="width: 100px;"><strong>配置和部署</strong></td>
			<td><strong>Diagnostics 1.0</strong></td>
			<td><strong>Diagnostics 1.1/1.2/1.3</strong></td>
	</tr>

	<tr>
			<td>与 Visual Studio 集成 - 集成在 Azure Web/辅助角色开发体验中。</td>
			<td>是</td>
			<td>否</td>
	</tr>
	<tr>
			<td>PowerShell 脚本 - 用于在角色上管理 Diagnostics 的安装和配置的脚本。</td>
			<td>是</td>
			<td>是</td>
	</tr>
	
</tbody>
</table>

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
	<tr>
			<td style="width: 100px;"><strong>数据源</strong></td>
			<td><strong>默认集合</strong></td>
			<td><strong>格式</strong></td>
			<td><strong>说明</strong></td>
			<td><strong>Diagnostics 1.0</strong></td>
			<td><strong>Diagnostics 1.1/1.2</strong></td>
			<td><strong>Diagnostics 1.3</strong></td>
	</tr>
	<tr>
			<td>System.Diagnostics.Trace 日志</td>
			<td>是</td>
			<td>表</td>
			<td>记录从你的代码发送到跟踪侦听器的跟踪消息（必须将跟踪侦听器添加到 web.config 或 app.config 文件）。日志数据将以 scheduledTransferPeriod 指定的传输间隔传输到存储表 WADLogsTable。</td>
			<td>是</td>
			<td>否（使用 EventSource）</td>
			<td>是</td>
	</tr>
	<tr>
			<td>IIS 日志</td>
			<td>是</td>
			<td>Blob</td>
			<td>记录有关 IIS 网站的信息。日志数据将以 scheduledTransferPeriod 指定的传输间隔传输到您指定的容器。</td>
			<td>是</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>Azure 诊断基础结构日志</td>
			<td>是</td>
			<td>表</td>
			<td>记录有关诊断基础结构、RemoteAccess 模块和 RemoteForwarder 模块的信息。日志数据将以 scheduledTransferPeriodtransfer 指定的间隔传输到存储表 WADDiagnosticInfrastructureLogsTable。</td>
			<td>是</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>IIS 失败请求日志</td>
			<td>否</td>
			<td>Blob</td>
			<td>记录有关 IIS 站点或应用程序的失败请求的信息。还必须通过在 Web.config 中的 system.WebServer 下设置跟踪选项来启用此功能。日志数据将以 scheduledTransferPeriod 指定的传输间隔传输到您指定的容器。</td>
			<td>是</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>Windows 事件日志</td>
			<td>否</td>
			<td>表</td>
			<td>记录有关操作系统、应用程序或驱动程序运行状况的信息。必须显式指定性能计数器。添加性能计数器后，性能计数器数据将以 scheduledTransferPeriod 指定的传输间隔传输到存储表 WADPerformanceCountersTable。</td>
			<td>是</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>性能计数器</td>
			<td>否</td>
			<td>表</td>
			<td>记录有关操作系统、应用程序或驱动程序运行状况的信息。必须显式指定性能计数器。添加性能计数器后，性能计数器数据将以 scheduledTransferPeriod 指定的传输间隔传输到存储表 WADPerformanceCountersTable。</td>
			<td>是</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>故障转储</td>
			<td>否</td>
			<td>Blob</td>
			<td>记录有关系统崩溃时操作系统的状态的信息。小型故障转储将在本地收集。可以启用完全转储。日志数据将以 scheduledTransferPeriod 指定的传输间隔传输到您指定的容器。由于 ASP.NET 能够处理大多数异常，因此故障转储通常仅对辅助角色或 VM 有用。</td>
			<td>是</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>自定义错误日志</td>
			<td>否</td>
			<td>Blob</td>
			<td>通过使用本地存储资源，可立即将自定义数据记录和传输到您指定的容器。</td>
			<td>是</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>EventSource</td>
			<td>否</td>
			<td>表</td>
			<td>记录你的代码使用 .NET EventSource 类生成的事件。</td>
			<td>否</td>
			<td>是</td>
			<td>是</td>
	</tr>
	<tr>
			<td>基于清单的 ETW</td>
			<td>否</td>
			<td>表</td>
			<td>任何进程生成的 ETW 事件。</td>
			<td>否</td>
			<td>是</td>
			<td>是</td>
	</tr>
</tbody>
</table>

##<a name="additional"></a>其他资源##

- [有关开发 Azure 应用程序的故障排除最佳实践][]
- [使用 Azure Diagnostics 收集日志记录数据][]
- [调试 Azure 应用程序][]
- [为 Azure 云服务和虚拟机配置 Diagnostics][]

  

[概述]: #overview
[如何在辅助角色中启用 Diagnostics]: #worker-role
[如何在虚拟机中启用 Diagnostics]: #virtual-machine
[示例配置文件和架构]: #configuration-file-schema
[故障排除]: #troubleshooting
[常见问题]: #faq
[比较 Azure Diagnostics 版本]: #comparing
[其他资源]: #additional
[EventSource 类]: http://msdn.microsoft.com/zh-cn/library/system.diagnostics.tracing.eventsource(v=vs.110).aspx
  
[为 Azure 云服务和虚拟机配置 Diagnostics]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn186185.aspx
[调试 Azure 应用程序]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee405479.aspx   
[使用 Azure Diagnostics 收集日志记录数据]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433048.aspx
[有关开发 Azure 应用程序的故障排除最佳实践]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh771389.aspx
[免费试用版]： /pricing/1rmb-trial/?trial_button=A
[安装并配置 Azure PowerShell 0.8.7 或更高版本]： /zh-cn/documentation/articles/install-configure-powershell/
[Azure Diagnostics 1.2 配置架构]: http://msdn.microsoft.com/zh-cn/library/azure/dn782207.aspx
[Set-AzureServiceDiagnosticsExtension]: http://msdn.microsoft.com/zh-cn/library/dn495270.aspx
[Get-AzureServiceDiagnosticsExtension]: http://msdn.microsoft.com/zh-cn/library/dn495145.aspx
[Remove-AzureServiceDiagnosticsExtension]: http://msdn.microsoft.com/zh-cn/library/dn495168.aspx
<!--HONumber=39-->
