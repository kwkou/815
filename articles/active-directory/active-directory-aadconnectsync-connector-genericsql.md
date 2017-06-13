<properties
    pageTitle="泛型 SQL 连接器 | Azure"
    description="本文介绍如何配置 Microsoft 的泛型 SQL 连接器。"
    services="active-directory"
    documentationcenter=""
    author="AndKjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid="fd8ccef3-6605-47ba-9219-e0c74ffc0ec9"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/11/2017"
    wacn.date="06/12/2017"
    ms.author="v-junlch"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="40b144e06b05745e84347f74aa453d4f7ceaa113"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="generic-sql-connector-technical-reference"></a>泛型 SQL 连接器技术参考
本指南介绍泛型 SQL 连接器。 本文适用于以下产品：

- Microsoft 标识管理器 2016 (MIM2016)
- Forefront 标识管理器 2010 R2 (FIM2010R2)
  - 必须使用修补程序 4.1.3671.0 或更高版本 [KB3092178](https://support.microsoft.com/zh-cn/kb/3092178)。

对于 MIM2016 和 FIM2010R2，可以从 [Microsoft 下载中心](http://go.microsoft.com/fwlink/?LinkId=717495)下载此连接器。

若要查看此连接器的工作原理，请参阅[泛型 SQL 连接器分步指南](/documentation/articles/active-directory-aadconnectsync-connector-genericsql-step-by-step/)一文。

## <a name="overview-of-the-generic-sql-connector"></a>泛型 SQL 连接器概述
使用泛型 SQL 连接器可以将同步服务与提供 ODBC 连接的数据库系统相集成。  

从较高层面讲，当前的连接器版本支持以下功能：

| 功能 | 支持 |
| --- | --- |
| 连接的数据源 |此连接器支持所有 64 位 ODBC 驱动程序。 此连接器已进行以下各项的测试： <li>Microsoft SQL Server 和 SQL Azure</li><li>IBM DB2 10.x</li><li>IBM DB2 9.x</li><li>Oracle 10 和 11g</li><li>MySQL 5.x</li> |
| 方案 |<li>对象生命周期管理</li><li>密码管理</li> |
| 操作 |<li>完全导入和增量导入、导出</li><li>对于导出：添加、删除、更新和替换</li><li>设置密码、更改密码</li> |
| 架构 |<li>动态发现对象和属性</li> |

### <a name="prerequisites"></a>先决条件
在使用连接器之前，请确保在同步服务器上安装以下软件：

- Microsoft .NET 4.5.2 Framework 或更高版本
- 64 位 ODBC 客户端驱动程序

### <a name="permissions-in-connected-data-source"></a>连接的数据源权限
若要在泛型 SQL 连接器中创建或执行任何支持的任务，你必须具备：

- db_datareader
- db_datawriter

### <a name="ports-and-protocols"></a>端口和协议
有关运行 ODBC 驱动程序所需的端口，请参阅数据库供应商的文档。

## <a name="create-a-new-connector"></a>创建新连接器
若要创建泛型 SQL 连接器，请在“同步服务”中选择“管理代理”和“创建”。 选择“泛型 SQL (Microsoft)”连接器。

![CreateConnector](./media/active-directory-aadconnectsync-connector-genericsql/createconnector.png)

### <a name="connectivity"></a>连接
连接器使用 ODBC DSN 文件进行连接。 使用开始菜单中“管理工具”中的“ODBC 数据源”来创建 DSN 文件。 在管理工具中创建一个**文件 DSN**，以便提供给连接器。

![CreateConnector](./media/active-directory-aadconnectsync-connector-genericsql/connectivity.png)

当你创建新的泛型 SQL 连接器时，“连接”是第一个屏幕。 首先需要提供以下信息：

- DSN 文件路径
- 身份验证
  - 用户名
  - 密码

数据库应支持以下身份验证方法之一：

- **Windows 身份验证**：身份验证数据库使用 Windows 凭据来验证用户。 指定的用户名/密码用于向数据库进行身份验证。 此帐户需要数据库的权限。
- **SQL 身份验证**：身份验证数据库使用“连接”屏幕上定义的用户名/密码连接到数据库。 如果在 DSN 文件中存储用户名/密码，则优先使用在“连接”屏幕上提供的凭据。
- **Azure SQL 数据库身份验证**：有关详细信息，请参阅[使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)。

**DN 是定位点**：如果选择此选项，DN 也用作定位点属性。 它可用于简单实现，但也有以下限制：

- 连接器仅支持一个对象类型。 因此，所有引用属性只能引用相同的对象类型。

**导出类型: 对象替换**：在导出期间，只有一些属性已更改时，包含所有属性的整个对象将会导出并替换现有对象。

### <a name="schema-1-detect-object-types"></a>架构 1（检测对象类型）
此页面上，将配置连接器要如何在数据库中查找不同的对象类型。

每个对象类型显示为一个分区，并且在“配置分区和层次结构”上进一步设置。

![schema1a](./media/active-directory-aadconnectsync-connector-genericsql/schema1a.png)

**对象类型检测方法**：连接器支持以下对象类型检测方法。

- **固定值**：以逗号分隔列表来提供对象类型列表。 例如： `User,Group,Department`。  
  ![schema1b](./media/active-directory-aadconnectsync-connector-genericsql/schema1b.png)
- **表/视图/存储过程**：提供表/视图/存储过程的名称，然后提供列名称以提供对象类型的列表。 如果使用存储过程，则还需要使用 **[名称]:[方向]:[值]** 格式提供其参数。 独行提供每个参数（使用 Ctrl+Enter 来换行）。  
  ![schema1c](./media/active-directory-aadconnectsync-connector-genericsql/schema1c.png)
- **SQL 查询**：使用此选项可以提供 SQL 查询，返回包含对象类型的单个列，例如 `SELECT [Column Name] FROM TABLENAME`。 返回的列必须是字符串类型 (varchar)。

### <a name="schema-2-detect-attribute-types"></a>架构 2（检测属性类型）
在此页面上，将配置如何检测属性名称和类型。 系统针对在前一页检测到的每个对象类型，列出配置选项。

![schema2a](./media/active-directory-aadconnectsync-connector-genericsql/schema2a.png)

**属性类型检测方法**：连接器以“架构 1”屏幕中每个检测到的对象类型，支持以下属性类型检测方法。

- **表/视图/存储过程**：提供表/视图/存储过程的名称，用于查找属性名称。 如果使用存储过程，则还需要使用 **[名称]:[方向]:[值]** 格式提供其参数。 独行提供每个参数（使用 Ctrl+Enter 来换行）。 若要检测多值属性中的属性名称，请提供以逗号分隔的表或视图列表。 如果父表和子表具有相同的列名称，则不支持多值方案。
- **SQL 查询**：使用此选项可以提供 SQL 查询，返回包含属性名称的单个列，例如 `SELECT [Column Name] FROM TABLENAME`。 返回的列必须是字符串类型 (varchar)。

### <a name="schema-3-define-anchor-and-dn"></a>架构 3（定义定位点和 DN）
此页面可让你为每个检测到的对象类型配置定位点和 DN 属性。 可以选择多个属性，让定位点变成唯一。

![schema3a](./media/active-directory-aadconnectsync-connector-genericsql/schema3a.png)

- 不列出多值属性和布尔属性。
- DN 和定位点不能使用相同的属性，除非已在“连接”页面上选择“DN 是定位点”。
- 如果已在“连接”页面上选择“DN 是定位点”，此页面只需要 DN 属性。 此属性也用作定位点属性。

  ![schema3b](./media/active-directory-aadconnectsync-connector-genericsql/schema3b.png)

### <a name="schema-4-define-attribute-type-reference-and-direction"></a>架构 4（定义属性类型、引用和方向）
使用此页面可以配置每个属性的属性类型，如整数、二进制数或布尔值和方向。 “架构 2”页面中的所有属性都列出，包括多值属性。

![schema4a](./media/active-directory-aadconnectsync-connector-genericsql/schema4a.png)

- **DataType**：用于将属性类型映射到同步引擎所知的属性类型。 默认使用在 SQL 架构中检测到的相同类型，但 DateTime 和 Reference 不容易检测。 因此，需要指定 **DateTime** 或 **Reference**。
- **方向**：可以设置 Import、Export 或 ImportExport 的属性方向。 ImportExport 是默认值。

![schema4b](./media/active-directory-aadconnectsync-connector-genericsql/schema4b.png)

说明：

- 如果连接器无法检测属性类型，则使用字符串数据类型。
- 可将嵌套表视为包含一个列的数据库表。 Oracle 不以任何特定顺序存储嵌套表的行。 但是，当你将嵌套表检索到 PL/SQL 变量时，行便有从 1 开始的连续下标。 这可让你获取单个行的类似于数组的访问。
- 连接器不支持 **VARRYS**。

### <a name="schema-5-define-partition-for-reference-attributes"></a>架构 5（定义引用属性的分区）
在此页面上，将为所有引用属性配置属性所引用的分区（对象类型）。

![schema5](./media/active-directory-aadconnectsync-connector-genericsql/schema5.png)

如果使用“DN 是定位点”，则必须使用相同的对象类型作为引用的源对象类型。 不能引用其他对象类型。

>[AZURE.NOTE]
从 2017 年 3 月更新开始，现在提供了一个“*”选项，如果选中此选项，则将导入所有可能的成员类型。

![globalparameters3](./media/active-directory-aadconnectsync-connector-genericsql/any-option.png)

>[AZURE.IMPORTANT]
 自 2017 年 5 月起，更改了 “*”（即“任何选项”），以支持导入和导出流。 如果要使用此选项，多值表/视图须具有一个包含对象类型的属性。

![](./media/active-directory-aadconnectsync-connector-genericsql/any-02.png)

 </br> 如果选择了 “*”，则还必须指定包含对象类型的列的名称。</br> ![](./media/active-directory-aadconnectsync-connector-genericsql/any-03.png)

导入后，你将看到如下图所示的内容：

  ![globalparameters3](./media/active-directory-aadconnectsync-connector-genericsql/after-import.png)



### <a name="global-parameters"></a>全局参数
“全局参数”页面用于设置增量导入、日期/时间格式，以及密码方法。

![globalparameters1](./media/active-directory-aadconnectsync-connector-genericsql/globalparameters1.png)



泛型 SQL 连接器支持使用以下增量导入方法：

- **触发器**：请参阅[使用触发器生成差异视图](https://technet.microsoft.com/zh-cn/library/cc708665.aspx)。
- **水印**：可用于任何数据库的常规方法。 水印查询根据数据库供应商预先填充。 水印列必须出现在所用的每个表/视图上。 此列必须跟踪表的插入和更新及其依赖（多值或子级）表。 同步服务与数据库服务器之间的时钟必须同步。 如果没有同步，则可能会省略增量导入中的某些项目。  
  限制：
  - 水印策略不支持已删除的对象。
- **快照**（仅适用于 Microsoft SQL Server）[使用快照生成差异视图](https://technet.microsoft.com/zh-cn/library/cc720640.aspx)
- **更改跟踪**（仅适用于 Microsoft SQL Server）[关于更改跟踪](https://msdn.microsoft.com/zh-cn/library/bb933875.aspx)  
  的限制：
  - 定位点和 DN 属性必须属于表中所选对象的主键。
  - 在使用更改跟踪的导入和导出期间，不支持 SQL 查询。

**其他参数**：指定“数据库服务器时区”，以指出数据库服务器所在的位置。 此值用于支持各种格式的日期和时间属性。

连接器始终以 UTC 格式存储日期和日期时间。 若要能够正确地转换日期和时间，必须指定数据库服务器的时区以及所用的格式。 格式应以 .Net 格式表示。

在导出期间，必须以 UTC 时间格式将每个日期时间属性提供给连接器。

![globalparameters2](./media/active-directory-aadconnectsync-connector-genericsql/globalparameters2.png)

**密码配置**：连接器提供密码同步功能并支持设置和更改密码。

连接器提供两种方法来支持密码同步：

- **存储过程**：此方法需要两个存储过程来支持设置和更改密码。 按照以下示例，分别在“设置密码 SP 参数”和“更改密码 SP 参数”中键入添加和更改密码操作的所有参数。
    
    ![globalparameters3](./media/active-directory-aadconnectsync-connector-genericsql/globalparameters3.png)

- **密码扩展**：此方法需要密码扩展 DLL（必须提供实现 [IMAExtensible2Password](https://msdn.microsoft.com/zh-cn/library/microsoft.metadirectoryservices.imaextensible2password.aspx) 接口的扩展 DLL 名称）。 密码扩展组件必须放在扩展文件夹中，连接器才可以在运行时加载 DLL。

    ![globalparameters4](./media/active-directory-aadconnectsync-connector-genericsql/globalparameters4.png)

还必须在“配置扩展”页面上启用密码管理。
![globalparameters5](./media/active-directory-aadconnectsync-connector-genericsql/globalparameters5.png)

### <a name="configure-partitions-and-hierarchies"></a>配置分区和层次结构
在分区和层次结构页面上，选择所有对象类型。 每个对象类型都在自身的分区中。

![partitions1](./media/active-directory-aadconnectsync-connector-genericsql/partitions1.png)

还可以重写在“连接”或“全局参数”页面上定义的值。

![partitions2](./media/active-directory-aadconnectsync-connector-genericsql/partitions2.png)

### <a name="configure-anchors"></a>配置定位点
此页面是只读页面，因为已经定义定位点。 选择的定位点属性始终附加对象类型，确保它在所有对象类型中保持唯一。

![anchors](./media/active-directory-aadconnectsync-connector-genericsql/anchors.png)

## <a name="configure-run-step-parameter"></a>配置运行步骤参数
对连接器的运行配置文件配置这些步骤。 这些配置将进行导入和导出数据的实际工作。

### <a name="full-and-delta-import"></a>完整和增量导入
泛型 SQL 连接器支持使用以下方法的完整和增量导入：

- 表
- 查看
- 存储过程
- SQL 查询

![runstep1](./media/active-directory-aadconnectsync-connector-genericsql/runstep1.png)

**表/视图**  
若要导入对象的多值属性，必须在“多值表/视图名称”中提供逗号分隔的表/视图名称，并在父表的“联接条件”中提供各自的联接条件。

示例：你想要导入员工对象与其所有的多值属性。 有两个表：Employee（主表）和 Department（多值）。
请执行以下操作：

- 在“表/视图/SP”中键入“Employee”。
- 在“多值表/视图名称”中键入“Department”。
- 在“联接条件”中键入 Employee 与 Department 之间的联接条件，例如 `Employee.DEPTID=Department.DepartmentID`。
  ![runstep2](./media/active-directory-aadconnectsync-connector-genericsql/runstep2.png)

**存储过程**  
![runstep3](./media/active-directory-aadconnectsync-connector-genericsql/runstep3.png)

- 如果有大量的数据，建议实现存储过程的分页。
- 若要让存储过程支持分页，需要提供起始索引和结束索引。 请参阅：[有效进行大量数据的分页](https://msdn.microsoft.com/zh-cn/library/bb445504.aspx)。
- 在执行时，将已在“配置步骤”页面上设置的各自页面大小值替换 @StartIndex 和 @EndIndex。 例如，当连接器检索第一个页面并且页面大小设置为 500 时，@StartIndex 将是 1，@EndIndex 将是 500。 当连接器检索后续页面并更改 @StartIndex 和 @EndIndex 值时，这些值将会增大。
- 若要执行参数化存储过程，请以 `[Name]:[Direction]:[Value]` 格式提供参数。 独行输入每个参数（使用 Ctrl+Enter 来换行）。
- 泛型 SQL 连接器还支持从 Microsoft SQL Server 中链接服务器的导入操作。 如果要从链接的服务器中的表检索信息，则以 `[ServerName].[Database].[Schema].[TableName]`
- 泛型 SQL 连接器仅支持在运行步骤信息和架构检测之间具有类似结构（包括别名和数据类型）的对象。 如果架构中选择的对象与在运行步骤提供的信息不同，则 SQL 连接器无法支持此类方案。

**SQL 查询**  
![runstep4](./media/active-directory-aadconnectsync-connector-genericsql/runstep4.png)

![runstep5](./media/active-directory-aadconnectsync-connector-genericsql/runstep5.png)

- 不支持多个结果集查询。
- SQL 查询支持分页并提供起始索引和结束索引作为变量，以支持分页。

### <a name="delta-import"></a>增量导入
![runstep6](./media/active-directory-aadconnectsync-connector-genericsql/runstep6.png)

相比于完整导入，增量导入的配置更详细。

- 如果选择“触发器”或“快照”方法来跟踪增量更改，可以在“历史记录表或快照数据库名称”框中提供历史记录表或快照数据库。
- 还需要提供历史记录表与父表之间的联接条件，例如 `Employee.ID=History.EmployeeID`
- 若要从历史记录表跟踪父表上的事务，必须提供包含操作信息（添加/更新/删除）的列名称。
- 如果选择“水印”来跟踪增量更改，则必须在“水印列名称”中提供包含操作信息的列名称。
- 对于更改类型，“更改类型属性”列是必需的。 此列将主表或多值表中发生的更改映射到差异视图中的更改类型。 此列可以包含适用于属性级更改的 Modify_Attribute 更改类型，或适用于对象级更改的“添加”、“修改”或“删除”更改类型。 如果更改类型不是默认值“添加”、“修改”或“删除”，则可以使用此选项来定义这些值。

### <a name="export"></a>导出
![runstep7](./media/active-directory-aadconnectsync-connector-genericsql/runstep7.png)

泛型 SQL 连接器支持使用以下四种方法的导出：

- 表
- 查看
- 存储过程
- SQL 查询

**表/视图**  
如果选择“表/视图”选项，连接器将生成相应的查询来执行导出。

**存储过程**  
![runstep8](./media/active-directory-aadconnectsync-connector-genericsql/runstep8.png)

如果选择“存储过程”选项，则导出需要三个不同的存储过程来执行插入/更新/删除操作。

- **添加 SP 名称**：如有任何对象进入连接器以便在相应表中插入，则运行此 SP。
- **更新 SP 名称**：如有任何对象进入连接器以便在相应表中更新，则运行此 SP。
- **删除 SP 名称**：如有任何对象进入连接器以便在相应表中删除，则运行此 SP。
- 从架构选择的属性作为存储过程的参数值。 示例：`EmployeeName: INPUT: @EmployeeName`（EmployeeName 已在连接器架构中选择，连接器在执行导出时替换相应的值）
- 若要运行参数化存储过程，请以 `[Name]:[Direction]:[Value]` 格式提供参数。 独行输入每个参数（使用 Ctrl+Enter 来换行）。

**SQL 查询**  
![runstep9](./media/active-directory-aadconnectsync-connector-genericsql/runstep9.png)

如果选择“SQL 查询”选项，则导出需要三个不同的查询来执行插入/更新/删除操作。

- **插入查询**：如有任何对象进入连接器以便在相应的表中插入，则运行此查询。
- **更新查询**：如有任何对象进入连接器以便在相应的表中更新，则运行此查询。
- **删除查询**：如有任何对象进入连接器以便在相应的表中删除，则运行此查询。
- 从架构中选择的用作查询参数值的属性，例如 `Insert into Employee (ID, Name) Values (@ID, @EmployeeName)`

## <a name="troubleshooting"></a>故障排除
- 有关如何启用记录来排查连接器问题的信息，请参阅 [如何启用连接器的 ETW 跟踪](http://go.microsoft.com/fwlink/?LinkId=335731)。

<!---Update_Description: wording update -->