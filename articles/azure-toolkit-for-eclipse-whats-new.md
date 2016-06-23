<properties
	pageTitle="Azure Toolkit for Eclipse 的新增功能"
	description="了解 Azure Toolkit for Eclipse 中的最新功能。"
	services=""
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="multiple"
	ms.date="05/04/2016" 
	wacn.date="06/20/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh694270.aspx -->

# Azure Toolkit for Eclipse 的新增功能

## Azure Toolkit for Eclipse 版本

本文包含有关 Azure Toolkit for Eclipse 的不同版本和最新更新的信息。


### 2015 年 1 月 4 日

Azure Toolkit for Eclipse - 2016 年 1 月版包含以下增强功能：

* **支持 Zulu OpenJDK 更新**。有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。
* **更新了 Tomcat 和 Jetty 分发版**。已更新  Azure 上提供的、可配合 Azure Toolkit for Eclipse 使用的 Jetty 和 Tomcat 分发版。
* **Azure 的 Eclipse 工具包和 IntelliJ 工具包之间的功能对等性**。Azure Toolkit for Eclipse 和 [Azure Toolkit for IntelliJ] 现在支持相同的功能集。

### 2015 年 9 月 1 日

Azure Toolkit for Eclipse – 2015 年 9 月版包含以下增强功能：

* **支持 Zulu OpenJDK 更新**。有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。
* **更新了 Tomcat 和 Jetty 分发版**。已更新  Azure 上提供的、可配合 Azure Toolkit for Eclipse 使用的 Jetty 和 Tomcat 分发版。（这些分发版可让开发人员使用 Azure Toolkit for Eclipse 创建快速开发与测试项目）。
* **支持自动更新的 Tomcat 和 Jetty 引用**。除了 Azure 上所提供的 Tomcat 和 Jetty 特定版本以外，开发人员现在还可以引用名为“最新(自动更新)”的分发版，这个分发版将在下一次回收角色实例时自动更新为每个 Jetty 或 Tomcat 主要版本的最新分发版。（回收将自动发生，但开发人员也可以通过 Azure 门户手动触发回收）。 此新功能意味着开发人员不需要重新部署应用程序就能更新其服务器软件。（
*  此功能目前仅适用于开发和测试目的或非任务关键型应用程序，不建议在生产环境中使用。）
* **适用于 Azure 存储空间中 Blob、队列和表的 Azure 资源管理器视图**。可让开发人员直接从 Eclipse IDE 中对其存储项目执行一组常见的任务。例如：删除、上载或下载 Blob。

### 2015 年 8 月 1 日

Azure Toolkit for Eclipse – 2015 年 8 月版包含以下增强功能：

* **Application Insights 检测密钥管理**。此项更新可让你直接从 Eclipse IDE 获取、创建和管理 Application Insights 检测密钥。
* **Microsoft JDBC Driver 4.1 for SQL Server**。此项更新包含适用于 Microsoft SQL Server 的最新 JDBC 驱动程序支持。
* **Azure SDK 版本 2.7**。在 Windows 上安装工具包时，必须事先安装 Azure SDK 的这个最新更新版。（请注意，在非 Windows 操作系统上安装工具包时，不需要此更新。）
* **支持 Zulu OpenJDK v7 更新**。有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。

### 2015 年 5 月 1 日

Azure Toolkit for Eclipse – 2015 年 5 月版包含以下增强功能：

* **改进了服务器选择 UI**。此版本简化了非 Windows 操作系统上的工具包用法。
* **支持 Maven 项目**。此版本支持用作应用程序的 Maven 项目，该工具包可将这些项目部署到 Azure 并配置 Application Insights。
* **Azure SDK 版本 2.6**。在 Windows 上安装工具包时，必须事先安装 Azure SDK 的这个最新更新版。（请注意，在非 Windows 操作系统上安装工具包时，不需要此更新。）
* **执行部署升级而不是重新发布**。如果在已激活先前版本的情况下重新发布某个部署项目，该工具包现在将使用 Azure 的部署升级功能，而不是像过去那样关闭以前的部署，然后从头开始重新发布。因此，在可能的情况下，你的云服务可以无中断运行，即使在更新期间也能帮助实现高可用性，并加快重新发布过程。
* **支持最新的 Zulu OpenJDK v8 - Update 40**。有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。

### 2015 年 3 月 9 日

Azure Toolkit for Eclipse – 2015 年 3 月版包含以下增强功能：

* **支持 Mac、Ubuntu 和其他 Linux 风格**。此版本的 Azure Toolkit for Eclipse 添加了对 Mac OS 和多种 Unix 平台的支持，因此，开发人员可以在运行非 Windows 操作系统的 Eclipse 中，安装该工具包来创建、配置 Java 项目并将其发布到 Azure 云服务 (PaaS)。

>[AZURE.NOTE] 此功能目前以预览版提供，不建议在生产环境中使用。对于此版本，我们不提供客户支持服务级别协议 (SLA)，但鼓励并感谢大家的反馈。

* **新的 Application Insights 插件**。现在，开发人员可以在 Azure 上使用 Application Insights 配置自动服务器遥测。
* **基于 Ant 的命令行部署自动化**。此功能使开发人员能够使用 Eclipse 外部的 Ant，自动发布其较新版本的部署。首次从 Eclipse 部署某个项目后，将为该项目自动配置一个预先生成的脚本，后续的部署只需使用该脚本通过命令行完全自动化部署。
* **在 Azure 上提供了 Tomcat 与 Jetty 以简化和加速部署**。现在，开发人员可以直接引用 Azure 上提供的各个 Tomcat 和 Jetty 版本，而无需将 Java 服务器上载到他们的帐户（或通过工具包），因此，对于快速入门方案无需上载 Java 服务器。
* **将 Java Web 应用程序发布到 Azure 云服务的快捷方法**。为了减少简单开发和测试方案的学习曲线，开发人员现在可以更直接地将 Java 应用程序发布到 Azure。可以使用 Tomcat v8 和 Zulu JVM (OpenJDK) 的默认实例部署应用程序，而无需经历创建和配置 Azure 部署项目的整个过程。

### 2015 年 1 月 30 日

Azure Toolkit for Eclipse - 2015 年 1 月版包含以下增强功能：

* **支持 IBM® WebSphere® Application Server Liberty Core**。此版本在受支持应用程序服务器列表中添加了 IBM WebSphere Application Server Liberty Core，工具包可通过它来部署到 Azure。添加的这项最新功能扩展了工具包当前“现成”支持的应用程序服务器列表，而该列表中已经包含了各种版本的 Tomcat、Jetty、JBoss 和 GlassFish。
* **包含 Application Insights SDK**。此新发布的客户端 API 库 (v0.9.0) 现已成为 Package for Azure Libraries for Java 的一部分。
* **更新了 Package for Azure Libraries for Java**。此项更新包括 Azure Libraries for Java v0.7.0 和存储客户端 API v2.0.0，以及新发布的 Application Insights SDK v0.9.0。

### 2014 年 11 月 12 日

Azure Toolkit for Eclipse - 2014 年 11 月版包含以下增强功能：

* **支持 Azure SDK 2.5**。必须事先安装 Azure SDK 的这个最新更新版。
* **支持更新版本的 Zulu OpenJDK v1.8、v1.7 和 v1.6 包**。有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。
* **支持云服务的新标准 D 大小**，提供更高的性能和更多的内存资源。有关详细信息，请参阅 [Azure 的虚拟机和云服务大小]。

### 2014 年 10 月 17 日

Azure Toolkit for Eclipse - 2014 年 10 月版包含以下增强功能：

* **“发布到云”方案性能改进**。当用户有多个订阅和存储帐户时，可以更快地加载订阅信息。
* **支持更新版本的 Zulu OpenJDK v1.8 包**。有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。
* **支持即将弃用的旧版第三方 JDK**。已弃用的 JDK 包将不再显示在新部署项目的下拉菜单中。引用已弃用 JDK 包的现有项目暂时仍可继续引用，但建议升级此类项目以依赖最新的包。
* **更新了 Package for Azure Libraries for Java 客户端 API 库版本**。有关详细信息，请参阅   Azure 客户端 API]。
* **Bug 修复。** 此版本根据用户汇报和测试包含了其他大量 Bug 修复程序。

### 2014 年 8 月 5 日

Azure Toolkit for Eclipse - 2014 年 8 月版包含以下增强功能：

* **支持 Azure SDK 2.4。** 旧版 Eclipse 工具包将不适用于此新发布的 SDK。
* **更新了 Zulu OpenJDK v1.6、v1.7 和 v1.8 包的版本。** 有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。
* **更新了 Package for Azure Libraries for Java 客户端 API 库版本。** 有关详细信息，请参阅 [Azure 客户端 API]。
* **支持最新的发布设置文件格式。** 添加了对 2.0 版发布设置文件格式的支持。
* **“发布到云”功能幕后的体系结构更改。** 现在，该工具包使用新发布的适用于 Java 的 Azure 客户端 API 来提供发布到云支持。
* **Bug 修复。** 此版本包含用户请求的大量 Bug 修复程序。

### 2014 年 6 月 12 日

Azure Toolkit for Eclipse - 2014 年 6 月版是一项次要服务更新，它提供以下增强功能：

* **支持 Zulu OpenJDK 包 v1.8。** 有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。
* **更新了 Zulu OpenJDK v1.6 和 v1.7 包的版本。** 有关详细信息，请参阅 [Zulu OpenJDK 的 Azul Systems 网页]。
* **更新了 Package for Azure Libraries for Java 客户端 API 库版本。** 有关详细信息，请参阅 [Azure 客户端 API]。
* **Bug 修复。** 此版本包含用户请求的大量 Bug 修复程序。

### 2014 年 4 月 4 日

Azure Plugin for Eclipse - 2014 年 4 月版已发布。这是 Azure SDK 2.3 版随附的更新并且是一个必备组件，当你安装插件时会自动下载它。此更新包括自 2014 年 2 月预览版发布以来所推出的新功能、Bug 修复程序和一些反馈驱动的可用性增强功能：

* **支持 Azure SDK 2.3 版。** Azure Plugin for Eclipse - 2014 年 4 月版需要 Azure SDK 2.3。使用新插件时，如果你尚未安装 Azure SDK 2.3，系统将提示你允许安装该版本。请不要对早期版本的插件使用 Azure SDK 2.3。
* **无需部署整个包即可升级应用程序。** 在部署项目中包含的 Java 应用程序时，该插件现在会自动将这些应用程序上载到所选的存储帐户，因此，无需重新生成并重新部署整个包，就可以更新项目并回收角色实例，以部署最新的应用程序。
* **Tomcat 8 现在是已获认可的应用程序服务器。** 如果你在“Azure 部署项目”对话框的“服务器”选项卡中选择计算机上的 Tomcat 8 安装目录，插件将自动检测该目录，并以自动方式部署 Tomcat 8，就像部署列表中已有的旧版 Tomcat 一样。
* **Azul Zulu OpenJDK 包更新：v1.7 更新版 51 和 v1.6 更新版 47。** 从此版本开始提供了 Azul System Zulu Open JDK v7 包更新版 51。此外，从更新版 47 开始，Zulu Open JDK v6 包开始可用。这些更新是对以前提供的 Zulu Open JDK v7 包更新版 45、更新版 40 和更新版 25 的补充。
* **支持 A8 和 A9 Azure 虚拟机大小。** 现在，你可以将云服务部署到高内存 A8 和 A9 大小的虚拟机。有关 VM 大小的详细信息，请参阅 [Azure 的虚拟机和云服务大小]。
* **对于已启用 SSL 的角色自动从 HTTP 重定向到 HTTPS。** 当你的云服务只包含 HTTPS 角色时，如果用户请求指定了 HTTP，会自动重定向到 HTTPS。你不需要创建单独的角色来处理 HTTP 请求。
* **用于本地模拟的 Express Emulator。** Azure Express Emulator 现将用作本地调试应用程序时的模拟器。
* **Azure 的产品名称已更改为 Azure。** UI 屏幕现在会反映 Azure 产品名称的变化，而不再称为 Azure。

### 2014 年 2 月 6 日

Azure Plugin for Eclipse - 2014 年 2 月预览版已发布。此更新包括自 2013 年 10 月预览版发布以来所推出的新功能、Bug 修复程序和一些反馈驱动的可用性增强功能：

* **支持 SSL 卸载。** 已添加安全套接字层 (SSL) 卸载作为一项功能，使你可以轻松地在 Azure 上的 Java 部署中启用超文本传输协议 (HTTPS) 支持，而无需在 Java 应用程序服务器中配置 SSL。这特别适合用于会话相关性和/或经过身份验证的通信方案。例如，在使用工具包已支持的访问控制服务 (ACS) 筛选器时。有关详细信息，请参阅 [SSL 卸载]和[如何使用 SSL 卸载]。
* **GlassFish 4 现在是已获认可的应用程序服务器。** 如果你在“Azure 部署项目”对话框的“服务器”选项卡中选择计算机上的 GlassFish 4 安装目录，插件将自动检测该目录，并以自动方式部署 GlassFish OSE 4，就像部署列表中已有的 GlassFish OSE 3 版本一样。
* **Azul Zulu OpenJDK 包更新版 45。** 从此版本开始提供 Azul System Zulu（Open JDK v7 包）更新版 45；这是对以前提供的更新版 40 和更新版 25 的补充。
* **支持对专用终结点端口设置“auto”。** 可以将输入终结点和内部终结点的专用端口设置为自动，以允许 Azure 自动向该终结点分配端口。以前，你只能分配特定的端口号。
* **支持在自签名证书创建 UI 中自定义证书名称 (CN)。** 以前，会对所有新证书使用相同的硬编码名称；现在，你可以指定自己的证书名称，以帮助区分 Azure 门户中用于不同目的的多个证书。
* **Azure 工具栏：**Azure 工具栏已更新，它发生了以下变化。 
    * ![][ic710876] 为“新建 Azure 部署项目”按钮添加了此图标。
    * ![][ic710877] 添加了此图标作为自签名证书创建对话框的快捷方式。
* **支持 A5 Azure 虚拟机大小。** 现在，你可以将云服务部署到高内存 A5 大小的虚拟机。有关此 VM 大小的详细信息，请参阅 [Azure 的虚拟机和云服务大小]。
* **支持 Microsoft Windows Server 2012 R2。** 现在，你可以选择 Windows Server 2012 R2 作为云操作系统。

### 2013 年 10 月 22 日

Azure Plugin for Eclipse - 2013 年 10 月预览版已发布。此更新包括自 2013 年 9 月预览版发布以来所推出的新功能、Bug 修复程序和一些反馈驱动的可用性增强功能：

* **支持 Azure SDK 2.2 版。** Azure Plugin for Eclipse - 2013 年 10 月预览版支持 Azure SDK 2.2。该插件仍适用于 Azure SDK 2.1，如果你尚未安装最低版本 Azure SDK 2.1，则会自动安装 Azure SDK 2.2。
* **Azul Zulu OpenJDK 包更新版 40。** 根据 2013 年 9 月预览版的宣告，该插件会直接在 Azure 上启用第三方提供的 JDK，而无需你上载自己的 JDK。在 2013 年 10 月版中，现已提供 Azul System Zulu（Open JDK v7 包）更新版 40；这是对最初发布的更新版 25 的补充。
* **活动日志中的云部署链接。** 在 Azure 活动日志中，如果你的部署状态为“已发布”，则你可以单击“已发布”，因为它现在是部署的链接；这样便会在浏览器中打开你的部署。（“已发布”状态以前标记为“正在运行”。）
* **发布时提供目标操作系统选项。** “发布到 Azure”对话框包含一个新的字段“目标操作系统”，其中提供了更直观的方式让你设置目标操作系统。
* **自动覆盖以前的部署。** “发布到 Azure”对话框包含一个新的复选框“覆盖以前的部署”。如果选中此选项，则在发布新部署时，会自动覆盖以前的部署；在未事先取消发布以前的部署的情况下发布到相同位置不会出现“409 冲突”问题。
* **Jetty 9 现在是已获认可的应用程序服务器。** 如果你在“Azure 部署项目”对话框的“服务器”选项卡中选择计算机上的 Jetty 9 安装目录，插件将自动检测该目录，并以自动方式部署 Jetty 9，就像部署列表中已有的旧版 Jetty 一样。
* **从“项目”上下文菜单添加角色。** **Azure** 项目上下文菜单现在包含一个新的菜单项“添加角色”，它提供了更快、更直观的方式让你向 Azure 项目添加新角色。
* **对 Package for Azure Libraries for Java 库的更新。** 此项更新基于 [Azure 客户端 API] 版本 0.4.6。

### 2013 年 9 月 25 日

Azure Plugin for Eclipse - 2013 年 9 月预览版已发布。此更新包括自 2013 年 8 月预览版发布以来所推出的新功能、Bug 修复程序和一些反馈驱动的可用性增强功能：

* **允许在 Azure 上部署可用的 Azul Zulu OpenJDK 包。** 添加了一个新选项，供你在指定用于 Azure 部署的 JDK 时使用。使用此选项可以直接在 Azure 云上部署第三方 JDK 包，而无需上载自己的包。Azul Systems 即将提供基于 OpenJDK 的、名为 Zulu 的第一个此类包，现在，你可以使用此选项来部署该包。
* **对 Package for Azure Libraries for Java 库的更新。** 此项更新基于 [Azure 客户端 API] 版本 0.4.5。

### 2013 年 8 月 1 日

Azure Plugin for Eclipse - 2013 年 8 月预览版已发布。这是 Azure SDK 2.1 版随附的更新并且是一个必备组件，当你安装插件时会自动下载它。此更新包括自 2013 年 7 月预览版发布以来所推出的新功能、Bug 修复程序和一些反馈驱动的可用性增强功能：

* **删除了包含本地 JDK 和本地应用程序服务器作为部署包一部分的选项。** 在部署期间，最好是从云存储下载 JDK 和应用程序服务器以在包中包含这些组件，因为下载这些项可以减小部署包的大小、缩短部署时间，并简化维护。因此，我们已经删除了用于在部署包中包含 JDK 和应用程序服务器的选项。配置为包含本地 JDK 和本地应用程序服务器作为部署包一部分的现有项目将自动转换为向云存储自动上载 JDK 和应用程序服务器。
* **支持 Azure SDK 2.1 版。** Azure Plugin for Eclipse - 2013 年 8 月预览版需要 Azure SDK 2.1。请不要对早期版本的 Azure SDK 使用 2013 年 8 月预览版，并且不要对早期版本的 Azure Plugin for Eclipse 使用 Azure SDK 2.1。
* **支持 Eclipse Kepler 版本。** 鉴于此原因，新的最低所需 Eclipse IDE 版本为 Indigo。不再会在 Helios 上对 Azure Plugin for Eclipse 进行正式测试。

### 2013 年 7 月 3 日

Azure Plugin for Eclipse - 2013 年 7 月预览版已发布。此更新包括自 2013 年 5 月预览版发布以来所推出的新功能、Bug 修复程序和一些反馈驱动的可用性增强功能：

* **允许新建新的存储帐户。** 在“添加存储帐户”对话框中添加了一个“新建”。这样，你无需登录到 Azure 管理门户，便可以在 Eclipse 插件中创建存储帐户。（必须已有一个 Azure 订阅才能使用此功能。） 有关创建新存储帐户的详细信息，请参阅[新建存储器帐户]。
* **为存储帐户提供了新的“(自动)”选项用于自动部署 JDK 和服务器以及用于缓存。** 在对 JDK 和应用程序服务器使用“自动上载”选项时，你现在可以针对上载 JDK 和应用程序服务器或者使用 Azure Caching 时要使用的 URL 和存储帐户指定“(自动)”。然后，这些功能将自动使用你在“发布到 Azure”对话框中选择的相同存储帐户。[在 Eclipse 中为 Azure 创建 Hello World 应用程序]教程已更新为使用新的“(自动)”选项。
* **允许设置 Azure 服务终结点。** 指定服务终结点，以确定你的应用程序是在全球 Azure 平台管理、中国 21Vianet 运营的 Azure 还是私有 Azure 平台中部署和管理。有关详细信息，请参阅 [Azure 服务终结点]。
* **大型部署可以指定本地存储资源。** 如果你的部署太大，从而无法包含在默认 approot 文件夹中，则你可以指定本地存储资源作为 JDK 和应用程序服务器的部署目标。有关详细信息，请参阅[实施大型部署]。
* **支持 A6 和 A7 Azure 虚拟机大小。** 现在，你可以将云服务部署到高内存 A6 和 A7 大小的虚拟机。有关这些大小的详细信息，请参阅 [Azure 的虚拟机和云服务大小]。
* **对 Package for Azure Libraries for Java 库的更新。** 此项更新基于 [Azure 客户端 API] 版本 0.4.4。

### 2013 年 5 月 1 日

Azure Plugin for Eclipse - 2013 年 5 月预览版已发布。这是 Azure SDK 2.0 版随附的主要更新并且是一个必备组件，当你安装插件时会自动下载它。此版本包括自 2013 年 2 月预览版发布以来所推出的新功能、Bug 修复程序和一些反馈驱动的可用性增强功能：

* **将 JDK 和应用程序服务器自动上载到 Azure 存储空间以及从中进行部署。** 你可以根据需要，使用一个新选项自动将选定的 JDK 和应用程序服务器上载到指定的 Azure 存储帐户以及从中部署这些组件，而无需在部署包中嵌入这些组件，或者让用户手动上载这些组件。这项经常有人提到的功能可以极大地简化 JDK 和服务器组件的部署，尤其是对于初级用户。有关使用这些选项的演练，请参阅[在 Eclipse 中为 Azure 创建 Hello World 应用程序]。
* **集中跟踪存储帐户并更方便地引用存储帐户（通过下拉式控件）。** 这适用于依赖于存储的多个功能，例如 JDK 和服务器组件部署及缓存。有关详细信息，请参阅 [Azure 存储帐户列表]。
* **简化了“发布到云”向导中的远程访问设置。** 只需键入用户名和密码即可启用远程访问，将字段留空会保持禁用远程访问。
* **对 Package for Azure Libraries for Java 库的更新。** 此项更新基于 [Azure 客户端 API] 版本 0.4.2。
* **支持 Windows Server 2012 上的粘性会话。** 以前，粘性会话仅适用于 Windows Server 2008 R2，现在，这两种云操作系统目标都支持会话相关性。
* **包上载性能改进。** 与以前的版本相比，即使在部署包中嵌入了 JDK 和应用程序服务器，部署过程的上载部分也大能够快出两倍。

### 2013 年 2 月 8 日

Azure Plugin for Eclipse - 2013 年 2 月预览版已发布。这是一项次要更新，包括自 2012 年 11 月预览版发布以来所推出的 Bug 修复程序、反馈驱动的可用性增强功能和一些新功能：

* 支持从公有或私有 Azure blob 存储下载部署 JDK、应用程序服务器和其他任意组件，而无需在部署到云时将它们包含在部署包中。
* 在“Azure 角色属性”的“组件”部分中添加了“上移”和“下移”按钮，让用户更改角色的用户定义组件的处理顺序。
* 对 **Package for the Azure Libraries for Java** 库的更新，基于 [Azure 客户端 API] 版本 0.4.0。

### 2012 年 11 月 5 日

Azure Plugin for Eclipse - 2012 年 11 月预览版已发布。这是一项主要更新，包括自 2012 年 9 月预览版发布以来所推出的许多新功能，以及其他 Bug 修复程序和反馈驱动的可用性增强功能：

* 支持使用 Microsoft Windows Server 2012 作为云操作系统。
* 支持对 memcached 客户端提供 Azure 并置缓存支持。
* 包含 Apache Qpid JMS 客户端库以利用基于 Azure AMQP 的消息传递。
* 改进了“新建项目”向导，在向导末尾提供了一个新页面，使用户能够在其项目中快速启用多个常用的关键功能：粘性会话、缓存和远程调试。
* 在计算模拟器中运行时，将角色实例数自动减为 1，以避免服务器实例之间发生端口绑定冲突。

### 2012 年 9 月 28 日

Azure Plugin for Eclipse - 2012 年 9 月预览版已发布。此服务更新包括自 2012 年 8 月预览版发布以来所推出的许多附加 Bug 修复程序，以及对现有功能的一些反馈驱动型可用性增强：

* 支持使用 Microsoft Windows 8 和 Microsoft Windows Server 2012 作为开发操作系统，解决了以前阻止插件在这些操作系统上正常运行的问题。
* 改进了对指定终结点端口范围的支持。
* 与文件路径包含空格相关的 Bug 修复。
* 改进了角色上下文菜单，以更快地访问特定于角色的配置设置。
* 对“发布到云”向导做了轻微调整，并提供了许多附加的 Bug 修复程序。

### 2012 年 8 月 28 日

Azure Plugin for Eclipse - 2012 年 8 月预览版已发布。此服务更新包括自 2012 年 7 月预览版发布以来所推出的附加 Bug 修复程序，以及对现有功能的一些反馈驱动型可用性增强：

* 在“Azure 访问控制服务筛选器”对话框中：
    * 提供了**用于在应用程序 WAR 文件中嵌入签名证书的选项**，以简化云部署。
    * 在 ACS 筛选器 UI 中提供了**用于创建自签名证书的选项**。有关 Azure 访问控制服务筛选器的其他信息，请参阅[如何使用 Eclipse 在 Azure 访问控制服务上对 Web 用户进行身份验证]。
* 在“Azure 部署项目”向导中（同样适用于角色的“服务器配置”属性页）：
    * 在计算机上**自动发现 JDK 位置**（可视需要重写）。
    * 在选择应用程序服务器安装目录时**自动检测服务器类型**。

### 2012 年 7 月 15 日

Azure Plugin for Eclipse - 2012 年 7 月预览版，解决了 2012 年 6 月版发布后用户发现和/或汇报的许多最高优先级 Bug。这只是一项服务更新，而未包含任何新功能。

### 2012 年 6 月 7 日

Azure Plugin for Eclipse - 2012 年 6 月 CTP 已发布。新功能包括：

* **“新建 Azure 部署项目”向导：**允许你在改进的向导 UI 中直接选择 JDK、Java 应用程序服务器和 Java 应用程序。在现成的服务器配置列表中，可以选择 Tomcat 6、Tomcat 7、GlassFish OSE 3、Jetty 7、Jetty 8、JBoss 6 和 JBoss 7（单机版）。此外，你可以自定义服务器配置列表。经过这项 UI 改进，你现在不需要拖放压缩文件和通过启动脚本进行复制，而这是以前的主要方法。这种方法仍可正常使用，但似乎只可用于更高级的方案。
* **“服务器配置”角色属性页：**可让你在创建项目后轻松切换与部署关联的 JDK、Java 应用程序服务器和应用程序。有关详细信息，请参阅[服务器配置属性]。
* **“发布到云”向导：**可让你直接从 Eclipse 方便地将项目部署到 Azure，自动执行以前需要手动繁琐提取凭据、登录到 Azure 管理门户、上载包等操作。有关如何直接将项目部署到 Azure 的示例，请参阅[在 Eclipse 中为 Azure 创建 Hello World 应用程序]。
* **Azure 工具栏：**Eclipse 中现在提供了 Azure 工具栏，其中包含用于调用以下功能的按钮：
    * ![][ic710879] **在 Azure 模拟器中运行**：在模拟器中运行你的项目。
    * ![][ic710880] **重置 Azure 模拟器**：重置模拟器。
    * ![][ic710881] **为 Azure 生成云包**：编译要部署的包。
    * ![][ic710876] **新建 Azure 部署项目**：创建新的 Azure 部署项目。
    * ![][ic710882] **发布到 Azure 云**：将项目发布到 Azure。
    * ![][ic710883] **取消发布**：删除部署。
    * [在 Eclipse 中为 Azure 创建 Hello World 应用程序]中使用了其中的许多 Azure 工具栏按钮。
* **Azure Libraries for Java：**现已提供为 Eclipse 中的单个 Package for Azure Libraries for Java 库的一部分，随附了插件安装，并包含所有必要的依赖项。只需在 Java 项目中添加对该库的一个引用，而无需单独下载任何组件。有关详细信息，请参阅[安装 Azure Toolkit for Eclipse]。
* **在安装插件期间安装 Microsoft JDBC Driver 4.0 for SQL Server：**在安装新插件期间，可以安装最新版本的 Microsoft JDBC Driver for SQL Server。
* **在安装插件期间安装 Azure 访问控制服务筛选器：**在工具包中包含为 Eclipse 库的这个新组件可让你 Java Web 应用程序正常使用标识提供程序（如 Google、Live.com 和 Yahoo!）来利用 Azure 访问控制服务 (ACS) 身份验证。你不需要自行编写身份验证逻辑，只需配置几个选项，然后让筛选器执行繁琐的任务，让用户使用 ACS 登录。你可以专注于编写代码，使用户能够基于请求对象中的筛选器返回到应用程序中的标识来访问资源。有关使用 ACS 筛选器的教程，请参阅[如何使用 Eclipse 在 Azure 访问控制服务上对 Web 用户进行身份验证]。
* **自动检测 Azure SDK 1.7 必备组件：**当你创建新的 Azure 部署项目时，将会自动下载 Azure SDK 1.7（如果尚未安装）。
* **实例终结点：**允许直接端口终结点访问，以便与负载平衡角色实例通信。可以通过“[终结点属性]”页上的终结点 UI 添加实例终结点。在包含多实例部署的方案中，这有助于针对云中运行的特定计算实例启用远程调试和 JMX 诊断。 
* **组件 UI** 中：方便高级用户设置项目中各个 Azure 角色与其他外部资源（如 Java 应用程序项目）之间的项目依赖关系；此外，还可以方便描述其部署逻辑。有关详细信息，请参阅[组件属性]。
* **自动升级先前版本的项目：**如果打开的工作区包含使用先前版本的插件创建的 Azure 项目，旧项目将在 Eclipse 中显示为已关闭，因为先期版本的项目与新版本不兼容。如果你尝试打开这些旧项目之一，将会启动升级向导。如果你同意升级，将会创建一个新项目并在其名称后面附加 **\_Upgraded**，同时会自动更新它以配合新版本。你可以根据需要重命名新项目。在升级过程中，原始项目不会修改（并将保持为关闭状态）。

### 2011 年 12 月 10 日

Azure Plugin for Eclipse - 2011 年 12 月 CTP 已发布。新功能包括：新功能包括：

* **会话相关性（“粘性会话”）支持：**使用一个复选框就能启用有状态群集 Java 应用程序。有关详细信息，请参阅[会话相关性]。
* **预先制作的启动脚本示例：**对于最常见的 Java 服务器（Tomcat、Jetty、JBoss、GlassFish），只需从项目的 samples 目录复制/粘贴到启动脚本。
* **在模拟器中实时观察启动脚本的输出：**现在，你可以在专用控制台窗口中观察启动脚本的所有步骤的执行情况，当 Azure 执行脚本时，该窗口会显示执行进度和脚本中的错误。
* **自动化轻量 java.exe 监视：**当 Java.exe 停止运行时，使用自动包含在部署中的轻量预制脚本强制回收角色。
* **远程 Java 应用程序调试配置 UI：**可让你轻松启用 Eclipse 的远程调试器来访问模拟器或 Azure 云中运行的 Java 应用程序，以便可以逐步执行和调试实时 Java 代码。有关详细信息，请参阅[在 Eclipse 中调试 Azure 应用程序]。
* **本地存储资源配置 UI：**你不再需要通过直接操作 XML 来配置本地资源。在部署本地资源后，此功能使你能够通过可直接从启动脚本引用的环境变量，来访问该本地资源的有效文件路径。有关详细信息，请参阅[本地存储属性]。
* **环境变量配置 UI：**你不再需要通过手动编辑配置 XML 来设置环境变量。有关详细信息，请参阅[环境变量属性]。
* **SQL Azure 的 JDBC 驱动程序：**通过用作无缝集成 Eclipse 库的插件来安装，它可以方便你针对 SQL Azure 进行编程。 
* **通过上下文菜单快速访问角色配置 UI：**只需右键单击角色文件夹，然后单击“属性”。
* **自定义 Azure 项目和角色文件夹图标：**提供更好的可视性，方便在工作区和项目中导航。

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包]

[安装 Azure Toolkit for Eclipse]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心]。

<!-- URL List -->

[Zulu OpenJDK 的 Azul Systems 网页]: http://go.microsoft.com/fwlink/?LinkId=402457
[Azure Java 开发人员中心]: /develop/java/
[Azure 服务终结点]: /documentation/articles/azure-toolkit-for-eclipse-azure-service-endpoints/
[Azure 存储帐户列表]: /documentation/articles/azure-toolkit-for-eclipse-azure-storage-account-list/
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[Azure Toolkit for IntelliJ]: https://plugins.jetbrains.com/plugin/8053
[组件属性]: /documentation/articles/azure-toolkit-for-eclipse-azure-role-properties/#components_properties
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[在 Eclipse 中调试 Azure 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-debugging-azure-applications/
[实施大型部署]: /documentation/articles/azure-toolkit-for-eclipse-deploying-large-deployments/
[终结点属性]: /documentation/articles/azure-toolkit-for-eclipse-azure-role-properties/#endpoints_properties
[环境变量属性]: /documentation/articles/azure-toolkit-for-eclipse-azure-role-properties/#environment_variables_properties
[如何使用 Eclipse 在 Azure 访问控制服务上对 Web 用户进行身份验证]: /documentation/articles/active-directory-java-authenticate-users-access-control-eclipse/
[如何使用 SSL 卸载]: /develop/java/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/
[本地存储属性]: /documentation/articles/azure-toolkit-for-eclipse-azure-role-properties/#local_storage_properties
[Azure 客户端 API]: http://go.microsoft.com/fwlink/?LinkId=280397
[服务器配置属性]: /documentation/articles/azure-toolkit-for-eclipse-azure-role-properties/#server_configuration_properties
[会话相关性]: /documentation/articles/azure-toolkit-for-eclipse-enable-session-affinity/
[SSL 卸载]: /develop/java/
[新建存储器帐户]: /documentation/articles/azure-toolkit-for-eclipse-azure-storage-account-list/#create_new
[Azure 的虚拟机和云服务大小]: /documentation/articles/cloud-services-sizes-specs/

<!-- IMG List -->

[ic710876]: ./media/azure-toolkit-for-eclipse-whats-new/ic710876.png
[ic710877]: ./media/azure-toolkit-for-eclipse-whats-new/ic710877.png
[ic710879]: ./media/azure-toolkit-for-eclipse-whats-new/ic710879.png
[ic710880]: ./media/azure-toolkit-for-eclipse-whats-new/ic710880.png
[ic710881]: ./media/azure-toolkit-for-eclipse-whats-new/ic710881.png
[ic710876]: ./media/azure-toolkit-for-eclipse-whats-new/ic710876.png
[ic710882]: ./media/azure-toolkit-for-eclipse-whats-new/ic710882.png
[ic710883]: ./media/azure-toolkit-for-eclipse-whats-new/ic710883.png

<!---HONumber=Mooncake_0328_2016-->