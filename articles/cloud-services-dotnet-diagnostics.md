<properties linkid="dev-net-commons-tasks-diagnostics" urlDisplayName="Diagnostics" pageTitle="How to use diagnostics (.NET) - Azure feature guide" metaKeywords="Azure diagnostics monitoring,logs crash dumps C#" description="Learn how to use diagnostic data in Azure for debugging, measuring performance, monitoring, traffic analysis, and more." metaCanonical="" services="cloud-services" documentationCenter=".NET" title="Enabling Diagnostics in Azure" authors="ryanwi" solutions="" manager="" editor="" />

# 在 Azure 中启用诊断

通过 Azure 诊断，可以从在 Azure 中运行的辅助角色或 Web 角色收集诊断数据。可以将诊断数据用于调试和故障排除、度量性能、监视资源使用状况、进行流量分析和容量规划以及进行审核。

在部署之前或者在运行时，可以使用 Azure SDK 2.0 或更高版本在 Visual Studio 2012 或 Visual Studio 2013 中配置诊断。有关详细信息，请参阅[配置 Azure 诊断][配置 Azure 诊断]。

有关创建日志记录和跟踪策略以及使用诊断和其他技术排查问题的其他深入指南，请参阅[有关开发 Azure 应用程序的问题排查最佳实践][有关开发 Azure 应用程序的问题排查最佳实践]。

有关在应用程序中手动配置诊断或者远程更改诊断监视器配置的信息，请参阅[使用 Azure 诊断收集日志记录数据][使用 Azure 诊断收集日志记录数据]。

## 后续步骤

-   [远程更改诊断监视器配置][远程更改诊断监视器配置] - 部署应用程序后，你可以修改诊断配置。
-   [如何监视云服务][如何监视云服务] - 你可以在 [Azure 管理门户][Azure 管理门户]中监视云服务的关键性能度量值。

## 其他资源

-   [使用 Azure 诊断收集日志记录数据][使用 Azure 诊断收集日志记录数据]
-   [调试 Azure 应用程序][调试 Azure 应用程序]
-   [配置 Azure 诊断][配置 Azure 诊断]

  [配置 Azure 诊断]: http://msdn.microsoft.com/en-us/library/windowsazure/dn186185.aspx
  [有关开发 Azure 应用程序的问题排查最佳实践]: http://msdn.microsoft.com/en-us/library/windowsazure/hh771389.aspx
  [使用 Azure 诊断收集日志记录数据]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433048.aspx
  [远程更改诊断监视器配置]: http://msdn.microsoft.com/en-us/library/windowsazure/gg432992.aspx
  [如何监视云服务]: ../cloud-services-how-to-monitor/
  [Azure 管理门户]: http://manage.windowsazure.cn
  [调试 Azure 应用程序]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee405479.aspx
