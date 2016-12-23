<properties
	pageTitle="Azure 诊断故障排除"
	description="排查在 Azure 云服务和虚拟机中使用 Azure 诊断时遇到的问题 "
	services="multiple"
	documentationCenter=".net"
	authors="rboucher"
	manager="jwhit"
	editor=""/>  


<tags
	ms.service="multiple"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="10/04/2016"
	ms.author="robb"
	wacn.date="12/23/2016"/>


# Azure 诊断故障排除
有关使用 Azure 诊断的故障排除信息。有关 Azure 诊断的详细信息，请参阅 [Azure 诊断概述](/documentation/articles/azure-diagnostics/)。

## Azure Diagnostics 不启动
Diagnostics 由两个组件构成：来宾代理插件和监视代理。可以检查日志文件 **DiagnosticsPluginLauncher.log** 和 **DiagnosticsPlugin.log** 来获取有关诊断工具无法启动的信息。

在云服务角色中，来宾代理插件的日志文件位于：

	C:\Logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\1.6.3.0\

在 Azure 虚拟机中，来宾代理插件的日志文件位于：

	C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\1.6.3.0\Logs\

日志文件的最后一行包含退出代码。

	DiagnosticsPluginLauncher.exe Information: 0 : [4/16/2016 6:24:15 AM] DiagnosticPlugin exited with code 0

该插件返回以下退出代码：

| 退出代码 | 说明 |
| --- | --- |
| 0 |成功。 |
| -1 |常规错误。 |
| -2 |无法加载 rcf 文件。<p>这是一个内部错误，仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生该错误。 |
| -3 |无法加载 Diagnostics 配置文件。<p><p>解决方案：这是配置文件未通过架构验证的结果。解决方案是提供符合架构的配置文件。 |
| -4 |监视代理 Diagnostics 的另一个实例已在使用本地资源目录。<p><p>解决方案：为 **LocalResourceDirectory** 指定不同的值。 |
| -6 |来宾代理插件启动器尝试使用无效的命令行启动 Diagnostics。<p><p>这是一个内部错误，仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生该错误。 |
| -10 |Diagnostics 插件退出并返回未处理的异常。 |
| -11 |来宾代理程序无法创建负责启动和监视监视代理的进程。<p><p>解决方案：验证是否有足够的系统资源用于启动新进程。<p> |
| -101 |调用 Diagnostics 插件时参数无效。<p><p>这是一个内部错误，仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生该错误。 |
| -102 |插件进程将无法初始化自身。<p><p>解决方案：验证是否有足够的系统资源用于启动新进程。 |
| -103 |插件进程将无法初始化自身。具体而言，它无法创建记录器对象。<p><p>解决方案：验证是否有足够的系统资源用于启动新进程。 |
| -104 |无法加载来宾代理提供的 rcf 文件。<p><p>这是一个内部错误，仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生该错误。 |
| -105 |Diagnostics 插件无法打开 Diagnostics 配置文件。<p><p>这是一个内部错误，仅当在 VM 上不正确地手动调用了 Diagnostics 插件时才会发生该错误。 |
| -106 |无法读取 Diagnostics 配置文件。<p><p>解决方案：这是配置文件未通过架构验证的结果。因此，解决方案是提供符合架构的配置文件。可以在 VM 上的文件夹 *%SystemDrive%/WindowsAzure/Config* 中，找到提供给 Diagnostics 扩展的 XML。打开相应的 XML 文件并搜索 **Microsoft.Azure.Diagnostics**，然后搜索 **xmlCfg** 字段。数据采用 base64 编码，因此你需要[将其解码](http://www.bing.com/search?q=base64+decoder)，以查看 Diagnostics 加载的 XML。<p> |
| -107 |传递给监视代理的资源目录无效。<p><p>这是一个内部错误，仅当在 VM 上不正确地手动调用了监视代理时才会发生该错误。</p> |
| -108 |无法将 Diagnostics 配置文件转换为监视代理配置文件。<p><p>这是一个内部错误，仅当使用无效的配置文件手动调用了 Diagnostics 插件时才会发生该错误。 |
| -110 |常规 Diagnostics 配置错误。<p><p>这是一个内部错误，仅当使用无效的配置文件手动调用了 Diagnostics 插件时才会发生该错误。 |
| -111 |无法启动监视代理。<p><p>解决方案：验证是否提供了足够的系统资源。 |
| -112 |常规错误 |

## 未将 Diagnostics 数据记录到 Azure 存储空间
Azure 诊断会将所有数据存储在 Azure 存储空间中。

丢失事件数据的最常见原因是错误地定义了存储帐户信息。

解决方案：更正 Diagnostics 配置文件，然后重新安装 Diagnostics。
如果在重新安装诊断扩展后仍有问题，则可能需要仔细检查监视代理存在的任何错误以进一步调试。事件数据在上载到存储帐户之前，会存储在 LocalResourceDirectory 中。

对于云服务角色，LocalResourceDirectory 为：

    C:\Resources\Directory<CloudServiceDeploymentID>.<RoleName>.DiagnosticStore\WAD<DiagnosticsMajorandMinorVersion>\Tables

对于虚拟机，LocalResourceDirectory 为：

    C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics<DiagnosticsVersion>\WAD<DiagnosticsMajorandMinorVersion>\Tables

如果 LocalResourceDirectory 文件夹中没有任何文件，监视代理将无法启动。这种情况通常是由无效的配置文件造成的，CommandExecution.log 中应会报告相关事件。

如果监视代理已成功收集事件数据，你将会看到配置文件中为每个事件定义的 .tsf 文件。监视代理将在文件 MaEventTable.tsf 中记录它所遇到的任何错误。若要检查此文件的内容，可以使用 tabel2csv 应用程序将 .tsf 文件转换成逗号分隔值 (.csv) 文件：

在云服务角色上：

    %SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics<DiagnosticsVersion>\Monitor\x64\table2csv maeventtable.tsf

云服务角色上的 %SystemDrive% 通常为是 D:

在虚拟机上：

    C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics<DiagnosticsVersion>\Monitor\x64\table2csv maeventtable.tsf

上述命令将生成日志文件 maeventtable.csv，你可以打开该文件并检查失败消息。

## 找不到诊断数据表
Azure 存储空间中保存 Azure 诊断数据的表是使用以下代码命名的：

        if (String.IsNullOrEmpty(eventDestination)) {
            if (e == "DefaultEvents")
                tableName = "WADDefault" + MD5(provider);
            else
                tableName = "WADEvent" + MD5(provider) + eventId;
        }
        else
            tableName = "WAD" + eventDestination;

下面是一个示例：

        <EtwEventSourceProviderConfiguration provider=”prov1”>
          <Event id=”1” />
          <Event id=”2” eventDestination=”dest1” />
          <DefaultEvents />
        </EtwEventSourceProviderConfiguration>
        <EtwEventSourceProviderConfiguration provider=”prov2”>
          <DefaultEvents eventDestination=”dest2” />
        </EtwEventSourceProviderConfiguration>

这会生成 4 个表：

| 事件 | 表名称 |
| --- | --- |
| provider=”prov1” &lt;Event id=”1” /&gt; |WADEvent+MD5(“prov1”)+”1” |
| provider=”prov1” &lt;Event id=”2” eventDestination=”dest1” /&gt; |WADdest1 |
| provider=”prov1” &lt;DefaultEvents /&gt; |WADDefault+MD5(“prov1”) |
| provider=”prov2” &lt;DefaultEvents eventDestination=”dest2” /&gt; |WADdest2 |

<!---HONumber=Mooncake_1212_2016-->
