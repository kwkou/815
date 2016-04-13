<properties
	pageTitle="Azure 诊断版本历史记录"
	description="介绍不同的 Azure SDK 版本随附的不同 Azure 诊断版本之间的差异。"
	services="multiple"
	documentationCenter=".net"
	authors="rboucher"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="multiple"
	ms.date="02/12/2016"
	wacn.date="04/11/2016"/>


# Azure 诊断版本历史记录

你是 Azure 诊断的新手？ 请参阅 [Azure 诊断概述](/documentation/articles/azure-diagnostics)。

每个 Azure SDK 版本通常都随附了新版的 Azure 诊断。下表描述了与 SDK 版本相关的 Azure SDK 和 Azure 诊断版本。



Azure SDK 版本 | Azure 诊断版本 | 型号
--- | --- | ---
1\.x | 1\.0 | 插件
2\.0 到 2.4| 1\.0 | "
2\.5 | 1\.2 | 扩展
2\.6 | 1\.3 | "
2\.7 | 1\.4 | "
2\.8 | 1\.5 | "


最新版本 1.5 随附于 Azure SDK 2.8。SDK 随附的 Azure 诊断扩展版本仅用于本地模拟器方案。任何已部署的应用程序在 Azure 中运行时，都自动使用最新的版本，不管应用程序是使用哪个 SDK 版本构建的。但是，除非你安装最新的 Azure SDK，否则仍无法获得所有帮助你使用新功能的工具。

各种功能和更改如下所述。

## Azure SDK 2.8
Azure SDK 2.8 添加了将诊断数据发送到 [Application Insights](/documentation/articles/app-insights-cloudservices) 的功能，因此可让你更轻松地在应用程序以及系统和基础结构级别诊断问题。

## Azure 2.6 诊断更改

对 Visual Studio 中的 Azure SDK 2.6 云服务项目进行了以下更改。（这些更改同样适用于更高版本的 Azure SDK。）

- 本地模拟器现在支持诊断。这意味着，当你在 Visual Studio 中开发和测试时，你可以收集诊断数据并确保应用程序正在创建相应的跟踪。当你使用 Azure 存储模拟器在 Visual Studio 中运行云服务项目时，连接字符串 `UseDevelopmentStorage=true` 可启用诊断数据收集。所有诊断数据都在“(开发存储)”存储帐户中收集。

- 诊断存储帐户连接字符串 (Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString) 将再次存储在服务配置 (.cscfg) 文件中。在 Azure SDK 2.5 中，在 diagnostics.wadcfgx 文件中指定诊断存储帐户。

连接字符串在 Azure SDK 2.4 及更早版本中的工作方式与它在 Azure SDK 2.6 中的工作方式之间有一些明显的差异。

- 在 Azure SDK 2.4 及更早版本中，连接字符串由诊断插件用作运行时以获取用于传输诊断日志的存储帐户信息。

- 在 Azure SDK 2.6 中，Visual Studio 在发布过程中通过诊断连接字符串使用相应的存储帐户信息配置诊断扩展。连接字符串让你为 Visual Studio 将在发布时使用的不同服务配置定义不同的存储帐户。但是，因为诊断插件已不再可用（在 Azure SDK 2.5 之后），.cscfg 文件本身不能启用诊断扩展。你必须通过工具（如 Visual Studio 或 PowerShell）单独启用扩展。

- 为了简化使用 PowerShell 配置诊断扩展的过程，Visual Studio 的程序包输出还包含每个角色的诊断扩展的公共配置 XML。Visual Studio 使用诊断连接字符串来填充公共配置中存在的存储帐户信息。公共配置文件在“扩展”文件夹中创建，并遵循模式 PaaSDiagnostics.<RoleName>.PubConfig.xml。任何基于 PowerShell 的部署都可以使用此模式将每个配置映射到角色。

- .cscfg 文件中的连接字符串还由 Azure 门户用于访问诊断数据，使这些数据可以显示在“监视”选项卡中。需要连接字符串才能配置服务以在门户中显示详细监视数据。

### 将项目迁移到 Azure SDK 2.6 和更高版本

从 Azure SDK 2.5 迁移到 Azure SDK 2.6 或更高版本时，如果你在 .wadcfgx 文件中指定了诊断存储帐户，则该帐户将继续保留在那里。若要针对不同存储配置充分使用不同存储帐户的灵活性，必须手动将连接字符串添加到项目。如果你将项目从 Azure SDK 2.4 或更低版本迁移到 Azure SDK 2.6，系统将保留诊断连接字符串。但是，请注意 Azure SDK 2.6 中处理连接字符串的方式的更改，如上一部分中所述。

### Visual Studio 如何确定诊断存储帐户

- 如果在 .cscfg 文件中指定了诊断连接字符串，则 Visual Studio 在发布时以及在打包过程中生成公共配置 xml 文件时将使用它来配置诊断扩展。

- 如果未在 .cscfg 文件中指定任何诊断连接字符串，则 Visual Studio 在发布时以及在打包过程中生成公共配置 xml 文件时将回退到使用 .wadcfgx 文件中指定的存储帐户来配置诊断扩展。

- .cscfg 文件中的诊断连接字符串将优先于 .wadcfgx 文件中的存储帐户。如果在 .cscfg 文件中指定了诊断连接字符串，则 Visual Studio 将使用该字符串，而忽略 .wadcfgx 中的存储帐户。

### “更新开发存储连接字符串...”复选框的作用是什么？

“在发布到 Azure 时使用 Azure 存储帐户凭据更新诊断和缓存的开发存储连接字符串”复选框为你提供了使用发布过程中指定的 Azure 存储帐户更新任何开发存储帐户连接字符串的简便方法。

例如，假设你选中此复选框，并且诊断连接字符串指定 `UseDevelopmentStorage=true`。将项目发布到 Azure 时，Visual Studio 将自动使用发布向导中指定的存储帐户更新诊断连接字符串。但是，如果已将实际的存储帐户指定为诊断连接字符串，则将改用该帐户。

## Azure SDK 2.4 及更早版本与 Azure SDK 2.5 及更高版本之间的诊断功能差异

如果要将项目从 Azure SDK 2.4 升级到 Azure SDK 2.5 或更高版本，则应考虑到以下诊断功能差异。

- **配置 API 已弃用** – 诊断的编程配置在 Azure SDK 2.4 或更早版本中可用，但在 Azure SDK 2.5 及更高版本中已弃用。如果你目前在代码中定义了诊断配置，则需在已迁移的项目中从头开始重新配置这些设置，这样才能让诊断正常工作。Azure SDK 2.4 的诊断配置文件是 diagnostics.wadcfg，而 Azure SDK 2.5 及更高版本的诊断配置文件是 diagnostics.wadcfgx。

- **云服务应用程序的诊断只能在角色级别配置，而不是在实例级别配置。**

- **每次部署应用程序时，都会更新诊断配置** – 如果你从服务器资源管理器更改诊断配置并重新部署你的应用，这可能会导致奇偶校验问题。

- **在 Azure SDK 2.5 及更高版本中，故障转储是在诊断配置文件而非代码中配置的** – 如果你在代码中配置了故障转储，则必须手动将配置从代码传输至配置文件，因为故障转储并未在迁移至 Azure SDK 2.6 的过程中传输。

<!---HONumber=Mooncake_0405_2016-->