<properties
    pageTitle="Azure 诊断故障排除 | Azure"
    description="排查在 Azure 虚拟机、Service Fabric 或云服务中使用 Azure 诊断时遇到的问题。"
    services="monitoring-and-diagnostics"
    documentationcenter=".net"
    author="rboucher"
    manager="carmonm"
    editor=""
    translationtype="Human Translation" />

<tags
    ms.assetid="66469bce-d457-4d1e-b550-a08d2be4d28c"
    ms.service="monitoring-and-diagnostics"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="02/09/2017"
    wacn.date="04/17/2017"
    ms.author="robb"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="8796853fe6c46d62b5838f9f14865688891f195d"
    ms.lasthandoff="04/07/2017" />

# <a name="azure-diagnostics-troubleshooting"></a>Azure 诊断故障排除
有关使用 Azure 诊断的故障排除信息。 有关 Azure 诊断的详细信息，请参阅 [Azure 诊断概述](/documentation/articles/azure-diagnostics/)。

## <a name="azure-diagnostics-is-not-starting"></a>Azure Diagnostics 不启动
Diagnostics 由两个组件构成：来宾代理插件和监视代理。 可以检查日志文件 **DiagnosticsPluginLauncher.log** 和 **DiagnosticsPlugin.log**，获取诊断未能启动的原因的信息。  

在云服务角色中，来宾代理插件的日志文件位于：

    C:\Logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\1.7.4.0\

在 Azure 虚拟机中，来宾代理插件的日志文件位于：

    C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\1.7.4.0\Logs\

日志文件的最后一行包含退出代码。  

    DiagnosticsPluginLauncher.exe Information: 0 : [4/16/2016 6:24:15 AM] DiagnosticPlugin exited with code 0

该插件返回以下退出代码：

| 退出代码 | 说明 |
| --- | --- |
| 0 |成功。 |
| -1 |常规错误。 |
| -2 |无法加载 rcf 文件。<p>此内部错误应仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生。 |
| -3 |无法加载诊断配置文件。<p><p>解决方案：这是配置文件未通过架构验证的结果。 解决方案是提供符合架构的配置文件。 |
| -4 |监视代理诊断的另一个实例已在使用本地资源目录。<p><p>解决方案：为 **LocalResourceDirectory** 指定不同的值。 |
| -6 |来宾代理插件启动器尝试使用无效的命令行启动诊断。<p><p>此内部错误应仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生。 |
| -10 |Diagnostics 插件退出并返回未处理的异常。 |
| -11 |来宾代理程序无法创建负责启动和监视监视代理的进程。<p><p>解决方案：验证是否有足够的系统资源用于启动新进程。<p> |
| -101 |调用诊断插件时参数无效。<p><p>此内部错误应仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生。 |
| -102 |插件进程将无法初始化自身。<p><p>解决方案：验证是否有足够的系统资源用于启动新进程。 |
| -103 |插件进程将无法初始化自身。 具体而言，它无法创建记录器对象。<p><p>解决方案：验证是否有足够的系统资源用于启动新进程。 |
| -104 |无法加载来宾代理提供的 rcf 文件。<p><p>此内部错误应仅当在 VM 上不正确地手动调用了来宾代理插件启动器时才会发生。 |
| -105 |诊断插件无法打开诊断配置文件。<p><p>此内部错误应仅当在 VM 上不正确地手动调用了诊断插件时才会发生。 |
| -106 |无法读取诊断配置文件。<p><p>解决方案：这是配置文件未通过架构验证的结果。 因此，解决方案是提供符合架构的配置文件。 对于 Azure 虚拟机，可以在 VM 上的文件夹 *%SystemDrive%/WindowsAzure/Config* 中，找到提供给诊断扩展的配置。 打开相应的 XML 文件并搜索 **Microsoft.Azure.Diagnostics**，然后搜索 **xmlCfg** 或 **WadCfg** 字段。 如果存在 WadCfg 字段，则表示配置在 JSON 中。 如果存在 xmlCfg 字段，则意味着配置在 XML 中，且已进行 base64 编码。 你需要将其解码才能查看诊断加载的 XML。 对于云服务角色，可以在 VM 上找到提供给诊断扩展的配置：C:\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\1.7.4.0\config.txt。 数据采用 base64 编码，因此需要[将其解码](http://www.bing.com/search?q=base64+decoder)才能查看诊断加载的 XML。<p> |
| -107 |传递给监视代理的资源目录无效。<p><p>此内部错误应仅当在 VM 上不正确地手动调用了监视代理时才会发生。</p> |
| -108 |无法将诊断配置文件转换为监视代理配置文件。<p><p>此内部错误应仅当使用无效的配置文件手动调用了诊断插件时才会发生。 |
| -110 |常规诊断配置错误。<p><p>此内部错误应仅当使用无效的配置文件手动调用了诊断插件时才会发生。 |
| -111 |无法启动监视代理。<p><p>解决方案：验证是否有足够的系统资源。 |
| -112 |常规错误 |

## <a name="diagnostics-data-is-not-logged-to-azure-storage"></a>未将 Diagnostics 数据记录到 Azure 存储空间
Azure 诊断会将所有数据存储在 Azure 存储空间中。

丢失事件数据的最常见原因是错误地定义了存储帐户信息。

解决方案：更正诊断配置文件，然后重新安装诊断。
如果在重新安装诊断扩展后仍有问题，则可能需要仔细检查监视代理存在的任何错误以进一步调试。 事件数据在上传到存储帐户之前，会存储在 LocalResourceDirectory 中。
此目录路径会有所不同。 

对于云服务角色：

    C:\Resources\Directory\<CloudServiceDeploymentID>.<RoleName>.DiagnosticStore\WAD<DiagnosticsMajorandMinorVersion>\Tables

对于虚拟机： 

    C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD<DiagnosticsMajorandMinorVersion>\Tables

如果 LocalResourceDirectory 文件夹中没有任何文件，监视代理将无法启动。 这种情况通常是由无效的配置文件造成的，CommandExecution.log 中应会报告相关事件。

如果监视代理已成功收集事件数据，你会看到配置文件中为每个事件定义的 .tsf 文件。 监视代理将在文件 MaEventTable.tsf 中记录它所遇到的任何错误。 若要检查此文件的内容，可以使用 tabel2csv 应用程序将 .tsf 文件转换成逗号分隔值 (.csv) 文件：

在云服务角色上：

    %SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<DiagnosticsVersion>\Monitor\x64\table2csv maeventtable.tsf

 

在虚拟机上：

    C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\Monitor\x64\table2csv maeventtable.tsf

上述命令生成日志文件 *maeventtable.csv*，可以打开该文件并检查失败消息。    

## <a name="diagnostics-data-tables-not-found"></a>找不到诊断数据表
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

这将产生 4 个表：

| 事件 | 表名称 |
| --- | --- |
| provider=”prov1” &lt;Event id=”1” /&gt; |WADEvent+MD5(“prov1”)+”1” |
| provider=”prov1” &lt;Event id=”2” eventDestination=”dest1” /&gt; |WADdest1 |
| provider=”prov1” &lt;DefaultEvents /&gt; |WADDefault+MD5(“prov1”) |
| provider=”prov2” &lt;DefaultEvents eventDestination=”dest2” /&gt; |WADdest2 |

<!-- Update_Description: wording update -->