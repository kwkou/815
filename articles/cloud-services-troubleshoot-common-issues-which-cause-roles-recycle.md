<properties
   pageTitle="云服务角色回收的常见原因 | Azure"
   description="突然回收的云服务角色可能会导致严重停机。以下是导致角色回收的一些常见问题，解决这些问题将有助于减少停机。"
   services="cloud-services"
   documentationCenter=""
   authors="dalechen"
   manager="felixwu"
   editor=""
   tags="top-support-issue"/>
<tags
   ms.service="cloud-services"
   ms.date="04/20/2016"
   wacn.date="05/31/2016" />

# 导致角色回收的常见问题

本文将讨论部署问题的一些常见原因，并提供故障排除技巧以帮助你解决这些问题。当角色实例无法启动，或者在“正在初始化”、“忙”和“正在停止”状态之间循环时，则指示应用程序存在问题。

## 与 Azure 客户支持联系

如果你对本文中的任何点需要更多帮助，可以联系 [MSDN Azure 和堆栈溢出论坛](/support/forums)上的 Azure 专家。

或者，你也可以提出 Azure 支持事件。请转到 [Azure 支持站点](/support/contact)并单击“获取支持”。有关使用 Azure 支持的信息，请阅读 [Azure 支持常见问题](/support/faq)。

## 缺少运行时依赖项

如果应用程序中的某个角色依赖于某个程序集，而该程序集不属 .NET Framework 或 Azure 托管库的一部分，则必须在应用程序包中显式包括该程序集。请记住，默认情况下，其他 Microsoft 框架在 Azure 上不可用。如果你的角色依赖于此类框架，则必须将这些程序集添加到应用程序包。

生成和打包应用程序之前，请验证以下各项：

- 如果使用的是 Visual Studio，请确保将项目中每个不属 Azure SDK 或 .NET Framework 的引用程序集的“复制本地”属性设置为“True”。

- 请确保 web.config 文件在 compilation 元素中未引用任何未使用的程序集。

- 每个 .cshtml 文件的“生成操作”将设置为“内容”。这可确保这些文件会正确显示在包中，并允许其他引用的文件显示在包中。

## 程序集针对的平台错误

Azure 是一个 64 位的环境。因此，针对 32 位目标编译的 .NET 程序集不会在 Azure 上运行。

## 角色在初始化或停止时引发未处理异常

任何通过 [RoleEntryPoint] 类的方法（包括 [OnStart]、[OnStop] 和 [Run] 方法）引发的异常均属未处理异常。如果这些方法中的某个方法出现未处理异常，则角色将回收。如果角色处于反复回收的状态，则每次该角色尝试启动时，都可能会引发未处理异常。

## 角色从 Run 方法返回

[Run] 方法会不限次数地运行。如果你的代码重写了 [Run] 方法，该方法会无限地休眠下去。如果 [Run] 方法返回，则角色会回收。

## 不正确的 DiagnosticsConnectionString 设置

如果应用程序使用 Azure 诊断，则你的服务配置文件必须指定 `DiagnosticsConnectionString` 配置设置。此设置应在 Azure 中指定一个连接到你的存储帐户的 HTTPS 连接。

在将应用程序包部署到 Azure 之前，若要确保你的 `DiagnosticsConnectionString` 设置正确，请验证以下内容：

- `DiagnosticsConnectionString` 设置指向 Azure 中的有效存储帐户。默认情况下，此设置指向模拟的存储帐户中，因此必须在部署应用程序包之前显式更改此设置。如果不更改此设置，则当角色实例尝试启动诊断监视器时，将引发异常。这可能导致角色实例无限期回收。

- 连接字符串是使用以下[格式](/documentation/articles/storage-configure-connection-string/)指定的。（协议必须指定为 HTTPS。） 将 *MyAccountName* 替换为你的存储帐户名称，将 *MyAccountKey* 替换为你的访问密钥：

        DefaultEndpointsProtocol=https;AccountName=MyAccountName;AccountKey=MyAccountKey;EndpointSuffix=core.chinacloudapi.cn

  如果你要使用 Azure Tools for Microsoft Visual Studio 来开发应用程序，则可使用[属性页](https://msdn.microsoft.com/zh-cn/library/ee405486)设置此值。

## 导出的证书不含私钥

若要在 SSL 下运行 Web 角色，必须确保导出的管理证书包含私钥。如果使用 Windows 证书管理器来导出证书，请务必对“导出私钥”选项选择“是”。该证书必须以 PFX 格式导出，这是当前支持的唯一格式。

## 后续步骤

查看更多针对云服务的[故障排除文章](..\?tag=top-support-issue&service=cloud-services)。

在 [Kevin Williamson 博客系列](http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.aspx)中查看更多角色回收方案。




[RoleEntryPoint]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleentrypoint.aspx
[OnStart]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleentrypoint.onstart.aspx
[OnStop]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[Run]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx

<!---HONumber=Mooncake_0523_2016-->
