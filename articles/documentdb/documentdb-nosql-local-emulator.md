<properties
    pageTitle="使用 DocumentDB 模拟器在本地开发 | Azure"
    description="使用 DocumentDB 模拟器，可以在本地免费开发并测试应用程序，而无需创建 Azure 订阅。"
    services="documentdb"
    documentationcenter=""
    keywords="DocumentDB 模拟器"
    author="arramac"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
    
<tags
    ms.assetid="90b379a6-426b-4915-9635-822f1a138656"
    ms.service="documentdb"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/23/2017"
    wacn.date="04/17/2017"
    ms.author="arramac"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="6a4d1676ee6bc3a2a0999550c66e6efa224a5e6a"
    ms.lasthandoff="04/07/2017" />

# <a name="use-the-azure-documentdb-emulator-for-development-and-testing"></a>将 Azure DocumentDB 模拟器用于开发和测试

[**下载模拟器**](https://aka.ms/documentdb-emulator)

为方便进行开发，Azure DocumentDB 模拟器提供了一个模拟 Azure DocumentDB 服务的本地环境。 使用 DocumentDB 模拟器，可以在本地开发并测试应用程序，而无需创建 Azure 订阅且不会产生任何费用。 如果对应用程序在 DocumentDB 模拟器中的工作情况感到满意，则可以切换到在云中使用 Azure DocumentDB 帐户。

## <a name="documentdb-emulator-system-requirements"></a>DocumentDB 模拟器系统要求
DocumentDB 模拟器具有以下硬件和软件要求：

- 软件要求
  - Windows Server 2012 R2、Windows Server 2016 或 Windows 10
- 最低硬件要求
  - 2 GB RAM
  - 10 GB 可用硬盘空间

## <a name="installing-the-documentdb-emulator"></a>安装 DocumentDB 模拟器
可以在 [Microsoft 下载中心](https://aka.ms/documentdb-emulator)下载并安装 DocumentDB 模拟器。 

> [AZURE.NOTE]
> 若要安装、配置并运行 DocumentDB 模拟器，必须在计算机上具有管理权限。

## <a name="checking-for-documentdb-emulator-updates"></a>检查 DocumentDB 模拟器更新
DocumentDB 模拟器包括内置的 Azure DocumentDB 数据资源管理器，用于浏览 DocumentDB 中存储的数据、创建新集合，并且通知何时有新的更新可下载。 

> [AZURE.NOTE]
> 在 DocumentDB 模拟器的一个版本中创建的数据不保证在使用不同版本时可以访问。 如果需要长期保存数据，建议将数据存储在 Azure DocumentDB 帐户中，而不是存储在 DocumentDB 模拟器中。 

## <a name="how-the-documentdb-emulator-works"></a>DocumentDB 模拟器的工作方式
DocumentDB 模拟器提供对 DocumentDB 服务的高保真模拟。 它支持和 Azure DocumentDB 相同的功能，包括支持创建和查询 JSON 文档、预配集合和调整集合的规模，以及执行存储过程和触发器。 可以使用 DocumentDB 模拟器开发和测试应用程序，并且只需对 DocumentDB 的连接终结点进行单个配置的更改就可以将它们部署到全球范围内的 Azure。

虽然创建了实际 DocumentDB 服务的高保真本地模拟，但是 DocumentDB 模拟器的实现不同于该服务。 例如，DocumentDB 模拟器针对持久性使用标准 OS 组件（如本地文件系统），针对连接性使用 HTTPS 协议堆栈。 这意味着一些依赖于 Azure 基础结构的功能，如全局复制、读/写的一位数毫秒延迟和可调一致性级别，无法通过 DocumentDB 模拟器使用。

## <a name="authenticating-requests-against-the-documentdb-emulator"></a>对 DocumentDB 模拟器的请求进行身份验证
和云中的 Azure Document 一样，每个针对 DocumentDB 模拟器的请求都必须进行身份验证。 DocumentDB 模拟器使用一个固定的帐户和公开的身份验证密钥进行主密钥身份验证。 此帐户和密钥是允许用于 DocumentDB 模拟器的唯一凭据。 它们具有以下特点：

    Account name: localhost:<port>
    Account key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==

> [AZURE.NOTE]
> DocumentDB 模拟器支持的主密钥仅用于模拟器。 不能在 DocumentDB 模拟器中使用生产 DocumentDB 帐户和密钥。 

此外，和 Azure DocumentDB 服务一样，DocumentDB 模拟器仅支持使用 SSL 的安全通信。

## <a name="start-and-initialize-the-documentdb-emulator"></a>启动并初始化 DocumentDB 模拟器

若要启动 Azure DocumentDB 模拟器，请选择“开始”按钮或按 Windows 键。 开始键入**DocumentDB 模拟器**，然后从应用程序列表中选择该模拟器。 

![选择“启动”按钮或按 Windows 键，开始键入“DocumentDB 模拟器”，然后从应用程序列表中选择该模拟器](./media/documentdb-nosql-local-emulator/azure-documentdb-database-local-emulator-start.png)

运行模拟器时，在 Windows 任务栏通知区域中会显示一个图标。 默认情况下 DocumentDB 模拟器在本地计算机（“localhost”）上运行，侦听端口 8081。

![DocumentDB 本地模拟器任务栏通知](./media/documentdb-nosql-local-emulator/azure-documentdb-database-local-emulator-taskbar.png)

默认情况下，DocumentDB 模拟器安装到 `C:\Program Files\DocumentDB Emulator` 目录。 你还可以从命令行启动和停止模拟器。 有关详细信息，请参阅 [命令行工具参考](#command-line) 。

## <a name="start-the-documentdb-emulator-data-explorer"></a>启动 DocumentDB 模拟器数据资源管理器

DocumentDB 模拟器启动时，会自动在浏览器中打开 DocumentDB 数据资源管理器。 地址将显示为 https://localhost:8081/_explorer/index.html。 如果关闭浏览器并想要稍后重新打开，可在浏览器中打开 URL 或从 Windows 任务栏图标中的 DocumentDB 模拟器中启动，如下所示。

![DocumentDB 本地模拟器数据资源管理器启动器](./media/documentdb-nosql-local-emulator/azure-documentdb-database-local-emulator-data-explorer-launcher.png)

## <a name="developing-with-the-documentdb-emulator"></a>通过 DocumentDB 模拟器进行开发
在桌面上运行 DocumentDB 模拟器以后，可以使用任何支持的 [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/) 或 [DocumentDB REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx) 与模拟器进行交互。 DocumentDB 模拟器还包括内置数据资源管理器，可以利用它在不编写任何代码的情况下创建集合、查看和编辑文档。 

    // Connect to the DocumentDB Emulator running locally
    DocumentClient client = new DocumentClient(
        new Uri("https://localhost:8081"), 
        "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==");

如果使用 [DocumentDB 的 MongoDB 协议支持](/documentation/articles/documentdb-protocol-mongodb/)，请使用以下连接字符串：

    mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10250/admin?ssl=true&3t.sslSelfSignedCerts=true

可以使用现有工具如 [DocumentDB Studio](https://github.com/mingaliu/DocumentDBStudio) 连接到 DocumentDB 模拟器。 用户还可以使用 [DocumentDB 数据迁移工具](https://github.com/azure/azure-documentdb-datamigrationtool)在 DocumentDB 模拟器和 Azure DocumentDB 服务之间迁移数据。

默认情况下，使用 DocumentDB 模拟器，可创建多达 25 个单区集合或 1 个已分区集合。 有关如何更改此值的详细信息，请参阅 [设置 PartitionCount 值](#set-partitioncount)。

## <a name="export-the-documentdb-emulator-ssl-certificate"></a>导出 DocumentDB 模拟器 SSL 证书

.NET 语言和运行时使用 Windows 证书存储来安全地连接到 DocumentDB 本地模拟器。 其他语言有自己管理和使用证书方法。 Java 使用自己的[证书存储](https://docs.oracle.com/cd/E19830-01/819-4712/ablqw/index.html)，而 Python 使用[套接字包装器](https://docs.python.org/2/library/ssl.html)。

为了获得用于语言和运行时（未与 Windows 证书存储集成）的证书，需要通过 Windows 证书管理器将其导出。 可通过运行 certlm.msc 进行启动，也可按照[导出 DocumentDB 模拟器证书](/documentation/articles/documentdb-nosql-local-emulator-export-ssl-certificates/)中的分步说明进行操作。 证书管理器开始运行后，打开个人证书（如下所示），并将友好名称为“DocumentDBEmulatorCertificate”的证书导出为 BASE-64 编码的 X.509 (.cer) 文件。

![DocumentDB 本地模拟器 SSL 证书](./media/documentdb-nosql-local-emulator/azure-documentdb-database-local-emulator-ssl_certificate.png)

可按照[将证书添加到 Java CA 证书存储](/documentation/articles/java-add-certificate-ca-store/)中的说明，将 X.509 证书导入 Java 证书存储。 证书导入 cacerts 存储后，Java 和 MongoDB 应用程序即能够连接到 DocumentDB 模拟器。

从 Python 和 Node.js SDK 连接到模拟器时，会禁用 SSL 验证。

## <a id="command-line"></a>DocumentDB 模拟器命令行工具参考
从安装位置中，可以使用命令行启动和停止模拟器、配置选项，和执行其他操作。

### <a name="command-line-syntax"></a>命令行语法

    DocumentDB.Emulator.exe [/Shutdown] [/DataPath] [/Port] [/MongoPort] [/DirectPorts] [/Key] [/EnableRateLimiting] [/DisableRateLimiting] [/NoUI] [/NoExplorer] [/?]

若要查看选项列表，请在命令提示符下键入 `DocumentDB.Emulator.exe /?`。

<table>
<tr>
  <td><strong>选项</strong></td>
  <td><strong>说明</strong></td>
  <td><strong>命令</strong></td>
  <td><strong>参数</strong></td>
</tr>
<tr>
  <td>[无参数]</td>
  <td>使用默认设置启动 DocumentDB 模拟器。</td>
  <td>DocumentDB.Emulator.exe</td>
  <td></td>
</tr>
<tr>
  <td>[帮助]</td>
  <td>显示支持的命令行参数的列表。</td>
  <td>DocumentDB.Emulator.exe /?</td>
  <td></td>
</tr>
<tr>
  <td>Shutdown</td>
  <td>关闭 DocumentDB 模拟器。</td>
  <td>DocumentDB.Emulator.exe /Shutdown</td>
  <td></td>
</tr>
<tr>
  <td>DataPath</td>
  <td>指定要在其中存储数据文件的路径。 默认路径为 %LocalAppdata%\DocumentDBEmulator。</td>
  <td>DocumentDB.Emulator.exe /DataPath=&lt;datapath&gt;</td>
  <td>&lt;datapath&gt;：可访问的路径</td>
</tr>
<tr>
  <td>端口</td>
  <td>指定用于模拟器的端口号。  默认值为 8081。</td>
  <td>DocumentDB.Emulator.exe /Port=&lt;port&gt;</td>
  <td>&lt;port&gt;：单个端口号</td>
</tr>
<tr>
  <td>MongoPort</td>
  <td>指定用于 MongoDB 兼容性 API 的端口号。 默认值为 10250。</td>
  <td>DocumentDB.Emulator.exe /MongoPort=&lt;mongoport&gt;</td>
  <td>&lt;mongoport&gt;：单个端口号</td>
</tr>
<tr>
  <td>DirectPorts</td>
  <td>指定用于直接连接的端口。 默认值为 10251、10252、10253、10254。</td>
  <td>DocumentDB.Emulator.exe /DirectPorts：&lt;directports&gt;</td>
  <td>&lt;directports&gt;：以逗号分隔的 4 个端口的列表</td>
</tr>
<tr>
  <td>键</td>
  <td>模拟器的授权密钥。 密钥必须是 64 字节向量的 base 64 编码。</td>
  <td>DocumentDB.Emulator.exe /Key：&lt;key&gt;</td>
  <td>&lt;key&gt;：密钥必须是 64 字节向量的 base 64 编码</td>
</tr>
<tr>
  <td>EnableRateLimiting</td>
  <td>指定启用请求速率限制行为。</td>
  <td>DocumentDB.Emulator.exe /EnableRateLimiting</td>
  <td></td>
</tr>
<tr>
  <td>DisableRateLimiting</td>
  <td>指定禁用请求速率限制行为。</td>
  <td>DocumentDB.Emulator.exe /DisableRateLimiting</td>
  <td></td>
</tr>
<tr>
  <td>NoUI</td>
  <td>不显示模拟器用户界面。</td>
  <td>DocumentDB.Emulator.exe /NoUI</td>
  <td></td>
</tr>
<tr>
  <td>NoExplorer</td>
  <td>不在启动时显示文档资源管理器。</td>
  <td>DocumentDB.Emulator.exe /NoExplorer</td>
  <td></td>
</tr>
<tr>
  <td>PartitionCount</td>
  <td>指定已分区的集合的最大数。 有关详细信息，请参阅[更改集合数](#set-partitioncount)。</td>
  <td>DocumentDB.Emulator.exe /PartitionCount=&lt;partitioncount&gt;</td>
  <td>&lt;partitioncount&gt;：允许的单分区集合的最大数量。 默认值为 25。 允许的最大值为 250。</td>
</tr>
</table>

## <a name="differences-between-the-documentdb-emulator-and-azure-documentdb"></a>DocumentDB 模拟器与 Azure DocumentDB 之间的差异 
由于 DocumentDB 模拟器提供在本地开发人员工作站上运行的模拟环境，因此模拟器与云中的 Azure DocumentDB 帐户之间的功能存在一些差异：

- DocumentDB 模拟器只支持一个固定的帐户和公开的主密钥。  在 DocumentDB 模拟器中无法重新生成密钥。
- DocumentDB 模拟器不是可扩展的服务，并且不支持大量集合。
- DocumentDB 模拟器不模拟不同的 [DocumentDB 一致性级别](/documentation/articles/documentdb-consistency-levels/)。
- DocumentDB 模拟器不模拟[多区域复制](/documentation/articles/documentdb-distribute-data-globally/)。
- DocumentDB 模拟器不支持服务配额替代，而 Azure DocumentDB 服务支持（例如文档大小限制、增加的分区集合存储）。
- 由于 DocumentDB 模拟器副本不一定能反映 Azure DocumentDB 服务的最新更改，因此请使用 [DocumentDB 容量规划器](https://www.documentdb.com/capacityplanner)准确估计应用程序的生产吞吐量 (RU) 需求。

## <a name="set-partitioncount"></a> 更改集合数

默认情况下，使用 DocumentDB 模拟器，可创建多达 25 个单分区集合或 1 个已分区集合。 通过修改 **PartitionCount** 值，可以创建最多 250 个单分区集合或 10 个已分区集合，或两者的任意组合（不得超过 250 个单分区，其中 1 个已分区集合 = 25 个单分区集合）。

如果在已超过当前分区计数后尝试创建集合，则模拟器将引发 ServiceUnavailable 异常，并收到以下消息。

    Sorry, we are currently experiencing high demand in this region, 
    and cannot fulfill your request at this time. We work continuously 
    to bring more and more capacity online, and encourage you to try again. 
    Please do not hesitate to email docdbswat@microsoft.com at any time or 
    for any reason. ActivityId: 29da65cc-fba1-45f9-b82c-bf01d78a1f91

若要更改 DocumentDB 模拟器可用的集合数，请执行以下操作：

1. 通过在系统任务栏上右键单击“DocumentDB 模拟器”图标，然后单击“重置数据...”，删除所有本地 DocumentDB 模拟器数据。
2. 删除文件夹 C:\Users\user_name\AppData\Local\DocumentDBEmulator 中的所有模拟器数据。
3. 通过在系统任务栏上右键单击“DocumentDB 模拟器”图标，然后单击“退出”，退出所有打开的实例。 退出所有实例可能需要一分钟。
4. 安装最新版本的 [DocumentDB 模拟器](https://aka.ms/documentdb-emulator)。
5. 通过设置一个 <= 250 的值启动具有 PartitionCount 标志的模拟器。 例如：`C:\Program Files\DocumentDB Emulator>DocumentDB.Emulator.exe /PartitionCount=100`。

## <a name="troubleshooting"></a>故障排除

使用以下提示来帮助解决使用 DocumentDB 模拟器时遇到的问题：

- 如果 DocumentDB 模拟器崩溃，请从 c:\Users\user_name\AppData\Local\CrashDumps 文件夹收集转储文件、进行压缩并将其附加到电子邮件，发送至 [askdocdb@microsoft.com](mailto:askdocdb@microsoft.com)。

- 如果在 DocumentDB.StartupEntryPoint.exe 中遇到崩溃，请从管理员命令提示符运行以下命令：`lodctr /R` 

- 如果遇到连接问题，请[收集跟踪文件](#trace-files)、进行压缩并将其附加到电子邮件，发送至 [askdocdb@microsoft.com](mailto:askdocdb@microsoft.com)。

### <a id="trace-files"></a>收集跟踪文件

若要收集调试跟踪，请在管理命令提示符下运行以下命令：

1. `cd /d "%ProgramFiles%\DocumentDB Emulator"`
2. `DocumentDB.Emulator.exe /shutdown`。 监视系统任务栏，确保该程序已关闭，这可能需要一分钟时间。 也可以直接在 DocumentDB 模拟器用户界面中单击“退出”  。
3. `DocumentDB.Emulator.exe /starttraces`
4. `DocumentDB.Emulator.exe`
5. 再现问题。 如果数据资源管理器无法运行，只需等待几秒钟，待浏览器打开以捕获错误。
5. `DocumentDB.Emulator.exe /stoptraces`
6. 导航到 `%ProgramFiles%\DocumentDB Emulator`，查找 docdbemulator_000001.etl 文件。
7. 将 .etl 文件和重现步骤一起发送至 [askdocdb@microsoft.com](mailto:askdocdb@microsoft.com) 进行调试。

## <a name="next-steps"></a>后续步骤
- 若要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB 简介](/documentation/articles/documentdb-introduction/)
- 若要开始使用 DocumentDB 模拟器进行开发，请下载一个[支持的 DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/)。

<!--Update_Description: wording update-->