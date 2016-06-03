<properties 
	pageTitle="在计算模拟器中本地分析云服务 | Azure" 
	services="cloud-services"
	description="使用 Visual Studio 探查器调查云服务中的性能问题" 
	documentationCenter=""
	authors="TomArcher" 
	manager="douge" 
	editor=""
	tags="" 
	/>

<tags 
	ms.service="cloud-services" 
	ms.date="03/28/2016" 
	wacn.date="05/12/2016"/>

# 在 Azure 计算模拟器中使用 Visual Studio 探查器来本地测试云服务的性能

可通过各种工具和技术来测试云服务的性能。在将云服务发布到 Azure 后，可以让 Visual Studio 收集分析数据，然后在本地进行分析，如[分析 Azure 应用程序][1]中所述。也可以使用诊断来跟踪各种性能计数器，如[在 Azure 中使用性能计数器][2]中所述。此外，在将应用程序部署到云之前，您可能需要在计算模拟器中本地分析应用程序。

本文包含了 CPU 采样分析方法，可在模拟器中本地执行该方法。CPU 采样是一种干预性不是很强的分析方法。探查器将按照指定的采样时间间隔拍摄调用堆栈的快照。将收集一段时间内的数据并将其显示在报告中。此分析方法倾向于指示在具有大量计算的应用程序中执行大多数 CPU 工作的位置。这使你能够侧重于应用程序在其上花费最多时间的“热路径”。



## 1：配置 Visual Studio 以进行分析

首先，提供了几个 Visual Studio 配置选项，这些选项在分析时可能会有用。若要使分析报告变得有意义，你将需要应用程序的符号（.pdb 文件）与系统库的符号。你将需要确保引用可用的符号服务器。为此，请在 Visual Studio 中的“工具”菜单上，依次选择“选项”、“调试”和“符号”。确保**符号文件 (.pdb) 位置**下方列出了 Microsoft 符号服务器。你也可参考 http://referencesource.microsoft.com/symbols，其上可能提供了其他符号文件。

![][4]

如果需要，可通过设置“仅我的代码”来简化探查器生成的报告。通过启用“仅我的代码”，可简化函数调用堆栈，以便从报告中隐藏对库和 .NET Framework 的完全内部调用。在“工具”菜单上，选择“选项”。然后展开“性能工具”节点，并选择“常规”。选中“为探查器报告启用‘仅我的代码’”的复选框。

![][17]

你可以在现有项目或新项目中使用这些说明。如果你创建新的项目来尝试下面描述的技术，请选择 C# **Azure 云服务**项目，并选择“Web 角色”和“辅助角色”。

![][5]

为了进行演示，请将一些代码添加到项目中，这些代码将占用大量时间，从而演示某个明显的性能问题。例如，将以下代码添加到辅助角色项目：

	public class Concatenator
	{
	    public static string Concatenate(int number)
	    {
	        int count;
	        string s = "";
	        for (count = 0; count < number; count++)
	        {
	            s += "\n" + count.ToString();
	        }
	        return s;
	    }
	}

在辅助角色的 RoleEntryPoint-derived 类中从 RunAsync 方法调用此代码。（忽略有关以同步方式运行方法的警告。）

        private async Task RunAsync(CancellationToken cancellationToken)
        {
            // TODO: Replace the following with your own logic.
            while (!cancellationToken.IsCancellationRequested)
            {
                Trace.TraceInformation("Working");
                Concatenator.Concatenate(10000);
            }
        }

本地生成并运行云服务且不进行调试 (Ctrl+F5)，并将解决方案配置设置为 **Release**。这将确保创建的所有文件和文件夹都用于本地运行应用程序，并确保启动所有仿真程序。从任务栏启动计算模拟器 UI，以验证辅助角色是否正在运行。

## 2：附加到进程

你必须将探查器附加到正在运行的进程，而不是通过从 Visual Studio 2010 IDE 中启动应用程序来分析该应用程序。

若要将探查器附加到进程，请在“分析”菜单上选择“探查器”和“附加/分离”。

![][6]

对于辅助角色，请查找 WaWorkerHost.exe 进程。

![][7]

如果项目文件夹位于网络驱动器上，则探查器将要求你提供其他位置来保存分析报告。

 也可以通过附加到 WaIISHost.exe 来附加到 Web 角色。如果应用程序中有多个辅助角色进程，则需要使用 processID 将它们区分开来。可以通过访问 Process 对象以编程方式查询 processID。例如，如果将此代码添加到角色中 RoleEntryPoint 派生的类的 Run 方法，则可在计算模拟器 UI 中查看日志以了解要连接到的进程。

	var process = System.Diagnostics.Process.GetCurrentProcess();
	var message = String.Format("Process ID: {0}", process.Id);
	Trace.WriteLine(message, "Information");

若要查看日志，请启动计算模拟器 UI。

![][8]

通过单击控制台窗口的标题栏，在计算模拟器 UI 中打开辅助角色日志控制台窗口。可以在日志中查看进程 ID。

![][9]

完成附加后，请在应用程序的 UI 中执行这些步骤（如果需要）来复制方案。

如果想要停止分析，请选择“停止分析”链接。

![][10]

## 3：查看性能报告

这将显示应用程序的性能报告。

此时，探查器将停止执行，将数据保存到 .vsp 文件中，并显示一个展示对此数据的分析的报告。

![][11]


如果你在热路径中看到 String.wstrcpy，请单击“仅我的代码”以将视图更改为仅显示用户代码。如果你看到 String.Concat，请尝试按“显示所有代码”按钮。

您将看到占用大部分执行时间的 Concatenate 方法和 String.Concat。

![][12]

如果添加了本文中的字符串串联代码，则此代码的任务列表中将显示一个警告。此外，还可能会显示一条警告，指示存在大量垃圾回收，这是由于创建和释放了大量字符串导致的。

![][14]

## 5：进行更改并比较性能

你也可在代码更改之前或之后比较性能。停止正在运行的进程，并编辑代码以将字符串串联操作替换为使用 StringBuilder：

	public static string Concatenate(int number)
	{
	    int count;
	    System.Text.StringBuilder builder = new System.Text.StringBuilder("");
	    for (count = 0; count < number; count++)
	    {
	         builder.Append("\n" + count.ToString());
	    }
	    return builder.ToString();
	}

执行其他性能运行，然后比较性能。在性能资源管理器中，如果运行位于同一会话中，则只需选择两个报告，打开快捷菜单，并选择“比较性能报告”。若要与其他性能会话中的运行进行比较，请打开“分析”菜单，并选择“比较性能报告”。在显示的对话框中指定这两个文件。

![][15]

报告将突出显示两个运行之间的差异。

![][16]

祝贺你！ 你已开始使用探查器。

## 故障排除

- 请确保正在分析 Release 生成，并在不调试的情况下启动。

- 如果未在“探查器”菜单上启用“附加/分离”选项，请运行性能向导。

- 使用计算模拟器 UI 来查看应用程序的状态。

- 如果在模拟器中启动应用程序时或附加探查器时出现问题，请关闭并重新启动计算模拟器。如果这样做无法解决问题，请尝试重新启动。如果使用计算模拟器挂起或删除正在运行的部署，则会出现此问题。

- 如果您已从命令行使用任一分析命令（尤其是全局设置），请确保已调用 VSPerfClrEnv /globaloff 并已关闭 VsPerfMon.exe。

- 如果采样时显示了消息“PRF0025: 未收集数据”，请检查附加的进程是否有 CPU 活动。未执行任何计算工作的应用程序将无法生成任何采样数据。此外，在执行任何采样前可能会退出进程。查看以验证正在分析的角色的 Run 方法是否已终止。

## 后续步骤

Visual Studio 探查器不支持在模拟器中检测 Azure 二进制文件，但若要测试内存分配，你可以在分析时选择该选项。此外，你可以选择并发分析，这将帮助你确定线程是否正在浪费时间竞争锁；也可以选择层交互分析，这将帮助你跟踪在应用程序的各个层之间（最常见的是数据层和辅助角色之间）进行交互时的性能问题。你可以查看应用程序生成的数据库查询并使用分析数据来改进对数据库的使用。有关层交互分析的信息，请参阅博客文章[演练：在 Visual Studio Team System 2010 中使用层交互探查器][3]。



[1]: http://msdn.microsoft.com/zh-cn/library/azure/hh369930.aspx
[2]: http://msdn.microsoft.com/zh-cn/library/azure/hh411542.aspx
[3]: http://blogs.msdn.com/b/habibh/archive/2009/06/30/walkthrough-using-the-tier-interaction-profiler-in-visual-studio-team-system-2010.aspx
[4]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally09.png
[5]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally10.png
[6]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally02.png
[7]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally05.png
[8]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally010.png
[9]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally07.png
[10]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally06.png
[11]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally03.png
[12]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally011.png
[14]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally04.png
[15]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally013.png
[16]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally012.png
[17]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally08.png
 

<!---HONumber=Mooncake_0503_2016-->
