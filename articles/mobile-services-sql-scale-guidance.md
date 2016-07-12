<properties 
	pageTitle="缩放 Azure SQL 数据库支持的移动服务 | Azure" 
	description="了解如何诊断和修复 SQL 数据库支持的移动服务中的可扩展性问题" 
	services="mobile-services" 
	documentationCenter="" 
	authors="lindydonna" 
	manager="dwrede" 
	editor="mollybos"/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/23/2016" 
	wacn.date="05/23/2016"/>

# 扩展 Azure SQL 数据库支持的移动服务


Azure 移动服务可轻松启动和构建连接云托管后端的应用，从而将数据存储在 SQL 数据库中。随着应用的增长，服务示例的扩展与在门户中的调整扩展设置一样简单，可轻松提高计算和网络容量。然而，扩展支持服务的 SQL 数据库要求在服务接收更多负载的同时进行主动规划和监控。本文档将指导您实行一组最佳实践，以确保 SQL 支持的移动服务能够持续提供最佳性能。

本主题将指导你完成以下基本步骤：

1. [诊断问题](#Diagnosing)
2. [索引](#Indexing)
3. [架构设计](#Schema)
4. [查询设计](#Query)
5. [服务体系结构](#Architecture)
6. [高级故障排除](#Advanced)

<a name="Diagnosing"></a>
##  诊断问题

如果你怀疑移动服务出现欠载问题，首先需要在 [Azure 管理门户]中查看服务的“仪表板”选项卡。以下几点需要验证：

- 用量计量表（包括“API 调用”和“活动设备”计量表）未超出配额
- “终结点监视”状态指示服务处于上升阶段（仅支持服务正使用标准层以及终结点监视已启用的情况） 

如与上述任一情况不符，请考虑在“缩放”选项卡上调整缩放设置。如果问题未得以解决，您可以继续操作并调查问题的根源是否为 Azure SQL 数据库。下文介绍了几种不同的问题诊断方法。

### 选择合适的 SQL 数据库层 

请务必了解，您可以在不同的数据库层进行选择，以确保选择合适的数据库层满足应用需求。Azure SQL 数据库提供三个不同的服务层：

- 基本
- 标准
- 高级


关于如何为数据库选择合适的层级，下面提供几点建议：

- **基本** - 在开发期间使用，或者用于仅预计每次进行单次数据库查询的小型生产服务
- **标准** - 用于预计进行多次并发数据库查询的生产服务
- **高级** - 用于并发查询数量较多、高峰值负载，以及每次请求的预计延迟较低的大型生产服务。

有关各层使用时机的详细信息，请参阅[使用新服务层的原因]。

###  分析数据库指标

如果您已对不同数据库层有所了解，我们将探讨数据库性能指标，以帮助我们探寻在各层内部以及各层之间进行扩展的原因。

1. 启动 [Azure 管理门户]。
2. 在移动服务 (Mobile Services) 选项卡中选择您希望使用的服务。
3. 选择“配置”选项卡。
4. 在“数据库设置”部分中选择“SQL 数据库”名称。这样可导航到门户中的 Azure SQL 数据库选项卡。
5. 导航到“监视”选项卡
6. 确保使用“添加度量值”按钮显示相关度量值。待显示指标包含以下内容
    - *CPU 百分比*（仅在基本/标准/高级层中显示）

    - *数据 IO 百分比*（仅在基本/标准/高级层中显示）
    - *日志 IO 百分比*（仅在基本/标准/高级层中显示）
    - *存储* 
7. 当服务遇到问题时，检查高于时窗的指标。 

    ![Azure 经典门户 - SQL 数据库度量值][PortalSqlMetrics]

如果指标超出了时间延长期 80% 的利用率，说明存在性能问题。有关数据库利用率的详细信息，请参阅[了解资源用量](http://msdn.microsoft.com/zh-cn/library/azure/dn369873.aspx#Resource)。

如果指标显示数据库的利用率较高，请考虑**将数据库纵向扩展至更高的服务层**，这是缓解问题的第一步。为尽快解决问题，请考虑使用数据库的“缩放”选项卡，对数据库进行缩放。这会增加你的费用。
![Azure 经典门户 - SQL 数据库缩放][PortalSqlScale]

请尽早考虑以下其他缓解步骤：

- **优化数据库。**
优化数据库通常可以降低数据库利用率，并避免扩展到更高的服务层。 
- **考虑服务体系结构。**
随着时间的推移，您的服务负载通常分布不均，但包含高需求“峰值”。如果不纵向扩展数据库应对峰值，并让数据库在低需求期间保持低利用率，通常可以调整服务体系结构，以避免出现此类峰值，或在不对数据库造成干扰的情况下处理峰值情况。

本文余下部分将介绍自定义指南，以帮助实施这些缓解措施。


###  配置警报

通常，最好主动为关键数据库指标配置警报，以确保您有充足的时间对资源耗尽情况做出反应。

1. 如果你希望为某数据库设置警报，导航到该数据库的“监视”选项卡
2. 确保如上节所述显示相关指标。
3. 选择你希望为其设置警报的指标，然后选择“添加规则”
    ![Azure 管理门户 - SQL 警报][PortalSqlAddAlert]
4. 提供警报名称及描述
    ![Azure 管理门户-SQL 警报名称和说明][PortalSqlAddAlert2]
5. 指定用于警报阈值的值。请考虑使用 **80%**，以便有时间做出反应。此外，请务必指定你主动监控的电子邮箱地址。
    ![Azure 管理门户 - SQL 警报阈值和电子邮件][PortalSqlAddAlert3]

有关诊断 SQL 问题的详细信息，请参阅本文末尾的[高级诊断](#AdvancedDiagnosing)。

<a name="Indexing"></a>
##  索引

如果您希望查看查询性能问题，首先您应该调查索引设计。索引非常重要，因为它们直接影响 SQL 引擎执行查询的方式。

例如，如果您经常需要根据字段查询某个要素，您应考虑添加该列的索引。否则，SQL 引擎将强制执行表扫描，并读取每个物理记录（或者至少查询列）和实质上已分散在磁盘上的记录。

因此，如果您经常在特定列中执行 WHERE 或 JOIN 语句，您应确保已为其编制索引。有关详细信息，请参阅[创建索引](#CreatingIndexes)部分。

如果索引编制得非常出色，但表扫描极其糟糕，这是否表示你应该为表中的每个数据行编制索引，以求保险？ 答案很简短：“或许不是。” 索引占用空间并产生开销：每次只要在表中插入内容，均需要更新每个索引列的索引结构。有关列索引选择指南，请参阅下文。

###  索引设计指南

如上所述，最好不要在一个表中添加过多索引，因为索引本身在性能和存储开销两方面代价高昂。

####  查询注意事项

- 请考虑将索引添加到通常以谓词（例如，WHERE 子句）和联接条件句使用的列中，同时平衡下列数据库注意事项。
- 编写在单个语句中插入或修改尽可能多个行的查询，而不要使用多个查询更新相同的行。当只有一条语句时，数据库引擎可以更好地优化索引维护方式。
	
####  数据库注意事项

一个表中含有大量索引会影响 INSERT、UPDATE、DELETE 和 MERGE 等语句的性能，因为所有索引必须随表格中数据的更改进行适当调整。

- 对于**经常更新**的表格，应避免对经常更新的列编制索引。
- 对于**不经常更新**但含有大量数据的表格，应使用多个索引。这样可以提升不修改数据（例如，SELECT 语句）的查询的性能，因为查询优化器具备更多选项，可查找最佳的访问方法。

为小表格编制索引也许不是最佳方法，因为查询优化器遍历用于搜索数据的索引所耗费的时间比执行简单表扫描的时间更长。因此，小表格的索引可能从来不用，但仍然须随表格中数据的更改而进行维护。


<a name="CreatingIndexes"></a>
###  创建索引

####  JavaScript 后端

若要设置 JavaScript 后端中某一列的索引，请执行以下操作：

1. 在 [Azure 管理门户]中打开你的移动服务。
2. 单击“数据”选项卡。
3. 选择你想要修改的表。
4. 单击“列”选项卡。
5. 选择该列。在命令栏中单击“设置索引”：

	![移动服务门户 - 设置索引][SetIndexJavaScriptPortal]

您还可以删除该视图中的索引。

####  .NET 后端

若要定义实体框架中的索引，请在你希望创建索引的字段中使用 `[Index]` 索引。例如：

    public class TodoItem : EntityData
    {
        public string Text { get; set; }

		[Index]
        public bool Complete { get; set; }
    }
		 
更多有关索引的详细信息，请参阅[实体框架中的索引批注][]。有关优化索引的更多提示，请参阅本文末尾的“[高级索引](#AdvancedIndexing)”。

<a name="Schema"></a>
##  架构设计

为对象选取数据类型，然后将其转换为 SQL 数据库架构时，需注意以下几个问题。由于 SQL 具备自定义的优化方式处理不同数据类型的索引和存储，因此优化架构通常可显著提高性能：

- **使用所提供的 ID 列**。每个移动服务表均带有以主关键配置的默认 ID 列，并具有索引设置。因此无需创建其他 ID 列。
- **在模型中使用正确的数据类型。** 如果你知道模型的特定属性将是数字或布尔值，请务必在模型中将其定义为该形式，而不要定义为字符串。在 JavaScript 后端中，请使用文本，例如，使用 `true` 而不是 `"true"`，使用 `5` 而不是 `"5"`。在 .NET 后端中，当你声明模型的属性时，请使用 `int` 和 `bool` 类型。这会让 SQL 为这些类型创建正确的架构，因而提高查询效率。  

<a name="Query"></a>
##  查询设计

查询数据库时要考虑的以下指南：

- **始终在数据库中执行联接操作。** 你经常需要合并来自两个或更多表的记录，且这些要合并的记录共享相同的字段（称为联接）。此操作涉及到同时从两个表中提取所有实体，然后循环访问所有实体，因此，如果未正确执行此操作，可能会降低效率。此类操作最好在数据库中执行，但有时却很容易误由客户端执行，或者在移动服务代码中执行。
    - 请不要在应用程序代码中执行联接
    - 请不要在移动服务代码中执行联接。在使用 JavaScript 后端时，请注意，[table 对象](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554210.aspx)不处理联接。请务必直接使用 [mssql 对象](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554212.aspx)，以确保在数据库中执行联接。有关详细信息，请参阅[联接关系表](/documentation/articles/mobile-services-how-to-use-server-scripts/#joins)。如果使用 .NET 后端，并且通过 LINQ 查询，实体框架将在数据库级别自动处理联接。
- **实现分页。** 查询数据库有时可能会导致大量记录返回到客户端。为了尽可能减少操作的大小和延迟，请考虑实现分页。
    - 默认情况下，你的移动服务将所有传入的查询限制在大小为 50 的页面中，但您可以手动请求多达 1000 条记录。有关详细信息，请参阅适用于 [Windows 应用商店](/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/#paging)、[iOS](/documentation/articles/mobile-services-ios-how-to-use-client-library/#paging)、[Android](/documentation/articles/mobile-services-android-how-to-use-client-library/#paging)、[HTML/JavaScript](/documentation/articles/mobile-services-html-how-to-use-client-library/#paging) 和 [Xamarin](/documentation/articles/partner-xamarin-mobile-services-how-to-use-client-library/#paging) 的“在页中返回数据”。
    - 通过移动服务代码进行的查询没有默认页面大小。如果您的应用不实现分页，也不用作防御措施，请考虑将默认限制应用于您的查询。在 JavaScript 后端是，对 [query 对象](http://msdn.microsoft.com/zh-cn/library/azure/jj613353.aspx)使用 **take** 运算符。如果你使用 .NET 后端，请考虑以 [Take 方法] (http://msdn.microsoft.com/zh-cn/library/vstudio/bb503062(v=vs.110).aspx) 作为 LINQ 查询的一部分。  

有关改进查询设计的详细信息，请参阅本文末尾的[高级查询设计](#AdvancedQuery)。

<a name="Architecture"></a>
##  服务体系结构

假设您要向所有客户发送推送通知，提醒他们查看应用中的新内容。他们点击该通知时，该应用将启动，这样可能会触发调用您的移动服务，并根据 SQL 数据库执行查询。由于可能会有数百万客户在仅仅几分钟的跨度内执行该操作，将形成 SQL 负载高峰，该峰值大大高于您应用的稳定状态负载。通过在峰值期间将应用扩展到更高版本的 SQL 层，然后再回缩可解决这种问题，但这种解决方法需要手动干预，并且会导致成本上升。通常，细微调整移动服务体系结构可显著平衡访问 SQL 数据库的负载客户端，并消除问题需求峰值。这些调整通常可以轻松执行，而对客户体验的影响可降至最低。下面是一些示例：

- **将负载分散到不同时间。** 如果你对特定事件（例如广播推送通知）的执行时间进行控制，并预期这些事件会产生需求上的高峰，且这些事件的执行时间并不重要，请考虑将其分散到不同时间。在上述示例中，或许你的应用程序客户可以在一天的不同时间分批获取新应用程序内容的通知，而无需在几乎相同的时间获取。请考虑将客户分成允许交错传送到每个批的组。使用通知中心时，应用附加标记以跟踪批，然后将推送通知传送到该标记，这样便可提供实现此策略的简单途径。有关标记的详细信息，请参阅[使用通知中心发送突发新闻](/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/)。
- **在可能的情况下使用 Blob 和表存储。** 客户在高峰期所查看的内容经常是较为静态的，且不需要存储在 SQL 数据库中，因为你不可能需要对该内容的关系查询功能。在此情况下，请考虑将内容存储在 Blob 或表存储中。你可以直接从设备访问 Blob 存储中的公共 Blob。若要以安全方式访问 Blob 或使用表存储，必须通过移动服务自定义 API 保护存储访问密钥。有关详细信息，请参阅[使用移动服务将图像上载到 Azure 存储空间](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-upload-data-blob-storage/)。
- **使用内存中缓存**。另一种方法是将流量峰值期间通常访问的数据存储于内存中缓存，比如 [Azure 缓存](/documentaiton/services/cache/)。这意味着传入的请求能够从内存中提取所需的信息，而不是重复查询数据库。

<a name="Advanced"></a>
##  高级故障排除
本部分介绍一些更高级的诊断任务，如果上述步骤未完全解决此问题，这些高级任务可能会有所帮助。

### 先决条件
若要执行本部分的诊断任务，你需要访问 SQL 数据库的管理工具，比如 **SQL Server Management Studio** 或内置于 **Azure 经典门户**的管理功能。

SQL Server Management Studio 是一个免费 Windows 应用，可提供最先进的功能。如果你无法访问 Windows 计算机（例如，你使用的是 Mac），请考虑按照[创建运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-windows-tutorial/)中的说明在 Azure 中设置虚拟机，然后远程连接到该虚拟机。如果你使用 VM 的主要目的是运行 SQL Server Management Studio，则一个**基本 A0**（以前称为“超小型”）实例应该够用。

Azure 经典门户提供内置管理体验，虽然限制更多，但无需本地安装即可提供。

下列步骤向您介绍如何获取关于支持移动服务的 SQL 数据库的连接信息，以及如何使用以下两种工具进行连接。您可以挑选任意一种您喜欢的工具。

#### 获取 SQL 连接信息
1. 启动 [Azure 经典门户]。
2. 在移动服务 (Mobile Services) 选项卡中选择您希望使用的服务。
3. 选择“配置”选项卡。
4. 在“数据库设置”部分中选择“SQL 数据库”名称。这样可导航到门户中的 Azure SQL 数据库选项卡。
5. 选择“为此 IP 地址设置 Azure 防火墙规则”。
6. 记下“连接到数据库”部分中的服务器地址，例如：*mcml4otbb9.database.chinacloudapi.cn*。

#### SQL Server Management Studio
1. 导航到“[SQL Server 版本 - Express](http://www.microsoft.com/zh-cn/server-cloud/products/sql-server-editions/sql-server-express.aspx)”
2. 找到“SQL Server Management Studio”部分，然后选择下方的“下载”按钮。
3. 完成安装步骤，直到成功运行该应用：

    ![SQL Server Management Studio][SSMS]

4. 在“连接到服务器”对话框中输入以下值
    - 服务器名称：*前面获取的服务器地址*
    - 身份验证：*SQL Server 身份验证*
    - 登录名：*创建服务器时选择的登录名*
    - 密码：*创建服务器时选择的密码*
5. 立即连接。

#### SQL 数据库管理门户
1. 在数据库的“Azure SQL 数据库”选项卡上，选择“管理”按钮 
2. 输入下列值对连接进行配置
    - 服务器：*应预设为正确值*
    - 数据库：*保留空白*
    - 用户名：*创建服务器时选择的登录名*
    - 密码：*创建服务器时选择的密码*
3. 立即连接。

    ![Azure 经典门户 - SQL 数据库][PortalSqlManagement]

<a name="AdvancedDiagnosing" /></a>
###  高级诊断

许多诊断任务都可直接在 **Azure 经典门户**中轻松完成，但有些高级诊断任务只能通过 **SQL Server Management Studio** 或 **SQL 数据库管理门户**来完成。我们将充分利用动态管理视图，它是一组已自动填充数据库相关诊断信息的视图。本部分将提供一组我们根据这些视图所运行的查询，以检查各种指标。有关详细信息，请参阅[使用动态管理视图监视 SQL 数据库][]。

完成上一部分中的步骤以连接到 SQL Server Management Studio 中的数据库后，请在“对象资源管理器”中选择你的数据库。依次展开“视图”，“系统视图”将显示管理视图列表。若要执行以下查询，请选择“新建查询”（前面已在“对象资源管理器”中选择了数据库），然后粘贴查询并选择“执行”。

![SQL Server Management Studio - 动态管理视图][SSMSDMVs]

如果你使用的是 SQL 数据库管理门户，请先选择你的数据库，然后选择“新建查询”。

![SQL 数据库管理门户 - 新建查询][PortalSqlManagementNewQuery]

若要执行以下任一查询，请将其粘贴到窗口中，然后选择“运行”。

![SQL 数据库管理门户 - 运行查询][PortalSqlManagementRunQuery]

####  高级指标


如果使用基础层、标准层和高级层，管理门户可随时提供部分指标。无论你使用哪种层，都可以通过 **[sys.resource\_stats](http://msdn.microsoft.com/zh-cn/library/dn269979.aspx)** 管理视图轻松获取所有度量值。请考虑下列查询：

    SELECT TOP 10 * 
    FROM sys.resource_stats 
    WHERE database_name = 'todoitem_db' 
    ORDER BY start_time DESC

> [AZURE.NOTE] 
请在你服务器的 **master** 数据库上执行此查询，因为只有该数据库显示 **sys.resource\_stats** 视图。

结果将会包含以下有用的度量值：CPU（层限制百分比）、存储 (MB)、物理数据读取（层限制百分比）、日志写入（层限制百分比）、内存（层限制百分比）、工作线程计数、会话计数等。

####  SQL 连接事件

**[sys.event\_log](http://msdn.microsoft.com/zh-cn/library/azure/jj819229.aspx)** 视图包含连接相关事件的详细信息。

    select * from sys.event_log 
    where database_name = 'todoitem_db'
    and event_type like 'throttling%'
    order by start_time desc

> [AZURE.NOTE] 
请在服务器的 **master** 数据库上执行此查询，**sys.event\_log** 视图只会出现在该数据库上。

<a name="AdvancedIndexing" ></a>
### 高级索引

表或视图可能包含以下类型的索引：

- **聚集**。聚集索引规定记录在磁盘上的物理存储方式。每个表只能有一个聚集索引，因为数据行本身可以按一个顺序进行排列。

- **非聚集**。非聚集索引与数据行分开存储，并用于执行基于索引值的查找。表中的所有非聚集索引均可将聚集索引中的键值用作查找键。

若要提供真实类比：请考虑使用书本或技术手册。每页的内容为一条记录，页码为聚集索引，书背后的主题索引为非聚集索引。主题索引的每个条目指向聚集索引，页码。

> [AZURE.NOTE] 
默认情况下，Azure 移动服务的 JavaScript 后端将 **\_createdAt** 设置为聚集索引。如果你要删除这列，或想要不同的聚集索引，请务必遵循以下[聚集索引设计指南](#ClusteredIndexes)。在.NET 后端，类 `EntityData` 会使用批注 `[Index(IsClustered = true)]` 将 `CreatedAt` 定义为聚集索引。

<a name="ClusteredIndexes"></a>
####  聚集索引设计指南

每个表都有关于具备以下属性的列（或数列（在复合键的情况下））的聚集索引：

- Narrow - 使用小型数据类型，或属于少量窄数据行的[复合键][Primary and Foreign Key Constraints]
- 唯一，或通常唯一
- 静态 - 值不会经常更改
- 不断增加 
- （可选）固定宽度
- （可选） nonnull

**narrow** 属性的原因是表中的所有其他索引都将聚集索引中的键值作为查找键。以书背面的主题索引为例，聚集索引是页码，小数。如果聚集索引中包含章节标题，主题索引本身则要长得多，因为键值将为（章节名称、页码）。

此键应为 **static** 和**不断增长**的状态，以避免维护记录的物理位置（这意味要物理移动记录，或可能需要通过拆分存储地记录页数来分段存储）。

聚集索引对执行下列操作的查询最有价值：

- 使用诸如 BETWEEN、>、>=、< 和 <= 等运算符返回一定范围的值。 
	- 使用聚集索引查找到第一个值的行后，才能保证包含后续索引值的行物理相邻。 
- 使用 JOIN 子句；通常为外键列。
- 使用 ORDER BY 或 GROUP BY 子句。
	- ORDER BY 或 GROUP BY 子句中指定列的索引可能无需采用数据库引擎对数据进行排序，因为行已经进行了排序。这将有助于提升查询性能。

####  在实体框架中创建聚集索引

若要使用实体框架在 .NET 后端设置 `IsClustered` 索引，请设置批注的属性。例如，这是在 `Microsoft.WindowsAzure.Mobile.Service.EntityData` 中的 `CreatedAt` 定义：

	[Index(IsClustered = true)]
	[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
	[TableColumnAttribute(TableColumnType.CreatedAt)]
	public DateTimeOffset? CreatedAt { get; set; }

####  在数据库架构中创建索引

就 JavaScript 后端而言，您只能通过 SQL Server Management Studio 或 Azure SQL 数据库门户直接更改数据库架构，然后修改表的聚集索引。

以下指南介绍了如何通过直接修改数据库架构设置聚集或非聚集索引：

- [创建和修改主键约束][]
- [创建非聚集索引][]
- [创建聚集索引][]
- [创建唯一索引][]

####  查找前 n 个缺失索引 
你可以在动态管理视图上编写 SQL 查询，以告知你与单个查询的资源使用情况有关的详细信息，或引导你找出所要添加的索引。以下查询将确定哪 10 个缺失索引会为用户查询生成最高的预期累积改进（采用降序）。

    SELECT TOP 10 *
    FROM sys.dm_db_missing_index_group_stats
    ORDER BY avg_total_user_cost * avg_user_impact * (user_seeks + user_scans)
    DESC;

下列示例查询在这些表内运行了一个 join，以获取应为各缺失索引一部分的列，并计算“索引优势”，以确定是否需要考虑给定的索引：

    SELECT * from 
    (
        SELECT 
        (user_seeks+user_scans) * avg_total_user_cost * (avg_user_impact * 0.01) AS index_advantage, migs.*
        FROM sys.dm_db_missing_index_group_stats migs
    ) AS migs_adv,
      sys.dm_db_missing_index_groups mig,
      sys.dm_db_missing_index_details mid
    WHERE
      migs_adv.group_handle = mig.index_group_handle and
      mig.index_handle = mid.index_handle
      AND migs_adv.index_advantage > 10
    ORDER BY migs_adv.index_advantage DESC;

有关详细信息，请参阅[使用动态管理视图监视 SQL 数据库][] 和[缺失索引动态管理视图][]。

<a name="AdvancedQuery" ></a>
### 高级查询设计 

通常难以诊断数据库中开销最大的查询。


####  查找前 n 个查询

下列示例返回了按平均 CPU 时间排名的前五个查询的信息。该示例根据查询散列收集了查询，以便逻辑上等值的查询能够根据累积资源消耗分组。

	SELECT TOP 5 query_stats.query_hash AS "Query Hash", 
	    SUM(query_stats.total_worker_time) / SUM(query_stats.execution_count) AS "Avg CPU Time",
	    MIN(query_stats.statement_text) AS "Statement Text"
	FROM 
	    (SELECT QS.*, 
	    SUBSTRING(ST.text, (QS.statement_start_offset/2) + 1,
	    ((CASE statement_end_offset 
	        WHEN -1 THEN DATALENGTH(st.text)
	        ELSE QS.statement_end_offset END 
	            - QS.statement_start_offset)/2) + 1) AS statement_text
	     FROM sys.dm_exec_query_stats AS QS
	     CROSS APPLY sys.dm_exec_sql_text(QS.sql_handle) as ST) as query_stats
	GROUP BY query_stats.query_hash
	ORDER BY 2 DESC;

有关详细信息，请参阅[使用动态管理视图监视 SQL 数据库][]。除执行查询之外，**SQL 数据库管理门户**还可为你提供有效的捷径查看数据：选择数据库“摘要”，然后选择“查询性能”：

![SQL 数据库管理门户 - 查询性能][PortalSqlManagementQueryPerformance]

####  分析查询计划

一旦你已确定了昂贵的查询，或如果你希望使用新查询部署代码，并希望了解其性能，此工具可全力支持你分析**查询计划**。借助查询计划，您可以查看给定 SQL 查询运行时，哪些操作占用了大量 CPU 时间和 IO 资源。若要在 **SQL Server Management Studio** 中分析查询计划，可使用突出显示的工具栏按钮。

![SQL Server Management Studio - 查询计划][SSMSQueryPlan]

若要在 **SQL 数据库管理门户**中分析查询计划，可使用突出显示的工具栏按钮。

![SQL 数据库管理门户 - 查询计划][PortalSqlManagementQueryPlan]

## 另请参阅

- [Azure SQL 数据库文档][]
- [Azure SQL 数据库性能和缩放][]
- [Azure SQL 数据库故障排除][]

### 索引

- [索引基础知识][]
- [常规索引设计指南][]
- [唯一索引设计指南][]
- [聚集索引设计指南][]
- [主键和外键约束][]
- [键的开销][]

### 实体框架
- [实体框架 5 性能注意事项][]
- [Code First 数据批注][]

<!-- IMAGES -->
[SSMS]: ./media/mobile-services-sql-scale-guidance/1.png
[PortalSqlManagement]: ./media/mobile-services-sql-scale-guidance/2.png
[PortalSqlMetrics]: ./media/mobile-services-sql-scale-guidance/3.png
[PortalSqlScale]: ./media/mobile-services-sql-scale-guidance/4.png
[PortalSqlAddAlert]: ./media/mobile-services-sql-scale-guidance/5.png
[PortalSqlAddAlert2]: ./media/mobile-services-sql-scale-guidance/6.png
[PortalSqlAddAlert3]: ./media/mobile-services-sql-scale-guidance/7.png
[SetIndexJavaScriptPortal]: ./media/mobile-services-sql-scale-guidance/set-index-portal-ui.png
[SSMSDMVs]: ./media/mobile-services-sql-scale-guidance/8.png
[PortalSqlManagementNewQuery]: ./media/mobile-services-sql-scale-guidance/9.png
[PortalSqlManagementRunQuery]: ./media/mobile-services-sql-scale-guidance/10.png
[PortalSqlManagementQueryPerformance]: ./media/mobile-services-sql-scale-guidance/11.png
[SSMSQueryPlan]: ./media/mobile-services-sql-scale-guidance/12.png
[PortalSqlManagementQueryPlan]: ./media/mobile-services-sql-scale-guidance/13.png

<!-- LINKS -->

[Azure 管理门户]: http://manage.windowsazure.cn

[Azure SQL 数据库文档]: /documentation/services/sql-database/
[Managing SQL Database using SQL Server Management Studio]: http://go.microsoft.com/fwlink/p/?linkid=309723&clcid=0x409
[使用动态管理视图监视 SQL 数据库]: http://go.microsoft.com/fwlink/p/?linkid=309725&clcid=0x409
[Azure SQL 数据库性能和缩放]: http://go.microsoft.com/fwlink/p/?linkid=397217&clcid=0x409
[Azure SQL 数据库故障排除]: http://msdn.microsoft.com/zh-cn/library/azure/ee730906.aspx
[使用新服务层的原因]: http://msdn.microsoft.com/zh-cn/library/azure/dn369873.aspx#Reasons

[Take 方法]: http://msdn.microsoft.com/zh-cn/library/vstudio/bb503062(v=vs.110).aspx
<!-- MSDN -->
[创建和修改主键约束]: http://technet.microsoft.com/zh-cn/library/ms181043(v=sql.105).aspx
[创建聚集索引]: http://technet.microsoft.com/zh-cn/library/ms186342(v=sql.120).aspx
[创建唯一索引]: http://technet.microsoft.com/zh-cn/library/ms187019.aspx
[创建非聚集索引]: http://technet.microsoft.com/zh-cn/library/ms189280.aspx

[Primary and Foreign Key Constraints]: http://msdn.microsoft.com/zh-cn/library/ms179610(v=sql.120).aspx
[主键和外键约束]: http://msdn.microsoft.com/zh-cn/library/ms179610(v=sql.120).aspx
[索引基础知识]: http://technet.microsoft.com/zh-cn/library/ms190457(v=sql.105).aspx
[常规索引设计指南]: http://technet.microsoft.com/zh-cn/library/ms191195(v=sql.105).aspx
[唯一索引设计指南]: http://technet.microsoft.com/zh-cn/library/ms187019(v=sql.105).aspx
[聚集索引设计指南]: http://technet.microsoft.com/zh-cn/library/ms190639(v=sql.105).aspx

[缺失索引动态管理视图]: http://technet.microsoft.com/zh-cn/library/ms345421.aspx

<!-- EF -->
[实体框架 5 性能注意事项]: http://msdn.microsoft.com/zh-cn/data/hh949853
[Code First 数据批注]: http://msdn.microsoft.com/zh-cn/data/jj591583.aspx
[实体框架中的索引批注]: http://msdn.microsoft.com/zh-cn/data/jj591583.aspx#Index

<!-- BLOG LINKS -->
[键的开销]: http://www.sqlskills.com/blogs/kimberly/how-much-does-that-key-cost-plus-sp_helpindex9/

<!---HONumber=Mooncake_0118_2016-->