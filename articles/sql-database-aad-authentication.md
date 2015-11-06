<properties
   pageTitle="通过使用 Azure Active Directory 身份验证连接到 SQL 数据库 | Windows Azure"
   description="了解如何通过使用 Azure Active Directory 身份验证连接到 SQL 数据库"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jeffreyg"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.date="09/14/2015"
   wacn.date="10/17/2015"/>

# 通过使用 Azure Active Directory 身份验证连接到 SQL 数据库 

Azure Active Directory 身份验证是使用 Azure Active Directory (Azure AD) 中的标识连接到 Windows Azure SQL 数据库的一种机制。通过 Azure Active Directory 身份验证，可以在一个中心位置中集中管理数据库用户和其他 Microsoft 服务的标识。集中 ID 管理提供一个单一位置来管理 SQL 数据库用户，并简化权限管理。包括如下优点：

- 提供一个 SQL Server 身份验证的替代方法。
- 帮助阻止用户标识在数据库服务器之间激增。
- 允许在单一位置中轮换密码
- 客户可以使用外部 (AAD) 组管理数据库权限。
- 它可以通过启用集成的 Windows 身份验证和 Azure Active Directory 支持的其他形式的身份验证来消除存储密码。
- Azure Active Directory 身份验证使用包含的数据库用户以数据库级别对标识进行身份验证。

> [AZURE.IMPORTANT]Azure Active Directory 身份验证是一个预览功能，并遵守许可协议（例如，企业协议、Windows Azure 协议或 Microsoft 在线订阅协议）中的预览条款以及任何适用的[ Windows Azure 预览版补充使用条款](http://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

配置步骤包括配置和使用 Azure Active Directory 身份验证的以下过程。

1. 创建并填充 Azure Active Directory
2. 确保你的数据库位于 Azure SQL 数据库 V12 中
3. 可选：关联或更改当前与你的 Azure 订阅关联的活动目录
4. 为 Azure SQL Server 创建 Azure Active Directory 管理员
5. 配置客户端计算机
6. 在映射到 Azure AD 标识的数据库中创建包含的数据库用户
7. 通过使用 Azure AD 标识连接到你的数据库


## 信任体系结构
 
以下高级别关系图概述了将 Azure AD 身份验证用于 Azure SQL 数据库的解决方案体系结构。箭头表示通信路径。

![aad 身份验证关系图][1]

下图表明允许客户端通过提交已由 Azure AD 身份验证且受数据库信任的一个令牌来连接到数据库的联合、信任和托管关系。请务必了解使用 Azure AD 身份验证访问数据库需要托管订阅与 Azure Active Directory 相关联。

![订阅关系][2]

## 管理员结构
 
使用 Azure AD 身份验证时，SQL 数据库服务器会有两个管理员帐户：原始的 SQL Server 管理员和 Azure AD 管理员。只有基于 Azure AD 帐户的管理员可以在用户数据库中创建第一个 Azure AD 包含的数据库用户。Azure AD 管理员登录名可以是 Azure AD 用户，也可以是 Azure AD 组。当管理员为组帐户时，可以由任何组成员使用，为 SQL Server 实例启用多个 Azure AD 管理员。以管理员身份使用组帐户时，允许你无需更改 SQL 数据库中的用户或权限，便可集中添加和删除 Azure AD 中的组成员，从而提高可管理性。无论何时都仅可配置一个 Azure AD 管理员（一个用户或组）。

![管理结构][3]

## 权限
 
若要创建新用户，必须具有数据库中的**更改任意用户**权限。**更改任意用户**权限可以授予任何数据库用户。**更改任意用户**权限还由服务器管理员帐户、具有该数据库的**数据库控制**或**更改任意数据库**权限的数据库用户以及 **db\_owner** 数据库角色的成员拥有。

若要在 Azure SQL 数据库中创建一个包含数据库用户，必须使用 Azure AD 标识连接到数据库。若要创建第一个包含数据库用户，必须通过使用 Azure AD 管理员（其是数据库的所有者）连接到数据库。在步骤 4 和 5 中演示了此操作。

## Azure AD 功能和限制 

可以在 Azure SQL Server 中设置以下 Azure Active Directory 成员：
- 本机成员：在托管域或客户域中的 Azure AD 中创建的成员。有关详细信息，请参阅[将自己的域名添加到 Azure AD](/documentation/articles/active-directory-add-domain)。
- 联合域成员：在联合域的 Azure AD 中创建的成员。有关详细信息，请参阅 [Windows Azure 现在支持与 Windows Server Active Directory 联合](http://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory/)。
- 作为本机或联合域成员从其他 Azure Active Directory 导入的成员。
- 以安全组形式创建的 Active Directory 组。

不支持 Microsoft 帐户（例如 outlook.com、hotmail.com、live.com）或其他来宾帐户（例如 gmail.com、yahoo.com)。如果可以使用帐户和密码登录到 [https://login.live.com](https://login.live.com)，则你使用的是 Microsoft 帐户，Azure SQL 数据库的 Azure AD 身份验证不支持此类帐户。

### 其他注意事项 

- 为了增强可管理性，我们建议你将一个专用 Azure Active Directory 组设置为管理员。
- 无论何时都仅可为 Azure SQL Server 配置一个 Azure AD 管理员（一个用户或组）。
- 只有 Azure Active Directory 管理员最初可以使用 Azure Active Directory 帐户连接到 Azure SQL Server。Active Directory 管理员可以配置后续的 Azure Active Directory 数据库用户。
- 我们建议将连接超时值设置为 30 秒。
- 不支持 BI 和 Excel 这样的工具。
- Azure Active Directory 身份验证只支持用于 **SqlServer 的 .NET Framework 数据提供程序**（至少是 .NET Framework 4.6 版本）。因此，Management Studio（适用于 SQL Server 2016）和数据层应用程序（DAC 和 .bacpac）可以连接，但 **sqlcmd.exe** 则无法连接，因为 **sqlcmd** 使用 ODBC 提供程序。
- 不支持双因素身份验证或其他形式的交互式身份验证。


## 1\.创建并填充 Azure Active Directory
 
创建 Azure Active Directory 并对其填充用户和组。这包括：

- 创建初始域 Azure AD 托管的域。
- 将本地 Active Directory 域服务与 Azure Active Directory 联合。

有关详细信息，请参阅[将自己的域名添加到 Azure AD](/documentation/articles/active-directory-add-domain)、[Windows Azure 现在支持与 Windows Server Active Directory 联合](http://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory/)、[管理 Azure AD 目录](https://msdn.microsoft.com/zh-cn/library/azure/hh967611.aspx)和[使用 Windows PowerShell 管理 Azure AD](https://msdn.microsoft.com/zh-cn/library/azure/jj151815.aspx)。

## 2\.确保你的数据库位于 Azure SQL 数据库 V12 中
 
在最新的 SQL 数据库 V12 中支持 Azure Active Directory 身份验证。有关 SQL 数据库 V12 以及了解其是否已在你所在区域上市的信息，请参阅[最新 SQL 数据库更新 V12 中的新增功能](/documentation/articles/sql-database-v12-whats-new)。

如果你已经拥有一个数据库，通过连接到该数据库（例如使用 SQL Server Management Studio）并执行 `SELECT @@VERSION;` 来验证其是否托管在 SQL 数据库 V12 中。SQL 数据库 V12 中数据库的预期输出至少是 **Microsoft SQL Azure (RTM) - 12.0**。

如果你的数据库未在 SQL 数据库 V12 中托管，请参阅[计划并准备升级到 SQL 数据库 V12](/documentation/articles/sql-database-v12-plan-prepare-upgrade)，然后访问 Azure 门户将数据库迁移到 SQL 数据库 V12。

或者，可以按照[在 SQL 数据库更新中创建数据库](/documentation/articles/sql-database-create)中列出的步骤，在 SQL 数据库 V12 中创建一个新数据库。**提示**：请先阅读下一步，然后再为新的数据库选择订阅。

## 3\.可选：关联或更改当前与你的 Azure 订阅关联的活动目录

若要将你的数据库与你组织的 Azure AD 目录相关联，请允许目录成为托管数据库的 Azure 订阅的一个受信目录。有关详细信息，请参阅 [Azure 订阅与 Azure AD 的关联方式](https://msdn.microsoft.com/zh-cn/library/azure/dn629581.aspx)。

**其他信息：**每个 Azure 订阅都与某个 Azure AD 实例存在信任关系。这意味着，此订阅信任该目录对用户、服务和设备执行身份验证。多个订阅可以信任同一个目录，但一个订阅只能信任一个目录。可以访问 [https://manage.windowsazure.cn/](https://manage.windowsazure.cn/)，在“设置”选项卡下查看你的订阅信任的目录。订阅与目录之间的这种信任关系不同于订阅与 Azure 中所有其他资源（网站、数据库等）之间的信任关系，在后一种关系中，这些资源更像是订阅的子资源。如果某个订阅过期，则对该订阅关联的其他那些资源的访问权限也将被终止。但是，目录将保留在 Azure 中，并且你可以将另一个订阅与该目录相关联，然后继续管理目录用户。有关资源的详细信息，请参阅[了解 Azure 中的资源访问](https://msdn.microsoft.com/zh-cn/library/azure/dn584083.aspx)。

以下过程提供分步说明，介绍如何更改给定订阅的关联目录。

1. 通过使用 Azure 订阅管理员，连接到你的 [Azure 门户](https://manage.windowsazure.cn/)。
2. 在左侧标题中，选择“设置”。 
3. 在设置屏幕中显示你的订阅。如果未显示所需订阅，请单击顶部的“订阅”，下拉“按目录筛选”框，并选择包含你的订阅的目录，然后单击“应用”。

	![选择订阅][4]
4. 在“设置”区域中，单击你的订阅，然后单击页面底部的“编辑目录”。

	![ad-设置-门户][5]
5. 在“编辑目录”框中，选择与你的 SQL Server 相关联的 Azure Active Directory，然后单击箭头转到下一步。

	![编辑-目录-选择][6]
6. 在“确认”目录“映射”对话框中，确认“所有协同管理员都将被删除”。

	![编辑-目录-确认][7]
7. 单击复选标记以重新加载门户。

> [AZURE.NOTE]更改目录时，将删除对所有协同管理员、Azure AD 用户和组以及目录支持的资源用户的访问权限，而它们也将不再有权访问此订阅或其资源。只有你作为服务管理员时，才能基于新的目录配置主体的访问权限。此更改可能需要大量时间来传播到所有资源。更改目录时还会更改 SQL 数据库的 Azure AD 管理员，并且不允许任何现有 Azure AD 用户访问 SQL 数据库。Azure AD 管理员必须重置（如下所述），并且必须创建新的 Azure AD 用户。

## 4\.为 Azure SQL Server 创建 Azure Active Directory 管理员
 
每个 Azure SQL Server 开始时只使用单个服务器管理员帐户，它是整个 Azure SQL Server 的管理员。必须创建第二个服务器管理员，这是一个 Azure AD 帐户。此主体在 master 数据库中作为包含的数据库用户创建。作为管理员，服务器管理员帐户是每个用户数据库中 **db\_owner** 角色的成员，并且以 **dbo** 用户身份输入每个用户数据库。有关服务器管理员帐户的详细信息，请参阅[管理 Azure SQL 数据库中的数据库和登录名](/documentation/articles/sql-database-manage-logins)以及 [Azure SQL 数据库安全指导原则和限制](/documentation/articles/sql-database-security-guidelines)中的**登录名和用户**部分。

> [AZURE.NOTE]如果用户使用的不是基于 Azure AD 的帐户（包括 Azure SQL Server 管理员帐户），则无法创建基于 Azure AD 的用户，这是因为他们没有权限来使用 Azure AD 验证建议的数据库用户。

### 通过使用 Azure 门户为 Azure SQL Server 设置 Azure Active Directory 管理员 

1. 在 [Azure 门户](https://manage.windowsazure.cn)的右上角中，单击你的连接以下拉一个列表，列出可能存在的 Active Directory。选择正确的 Active Directory 作为默认的 Azure AD。此步骤将与 Active Directory 关联的订阅链接到 Azure SQL 数据库，确保为 Azure AD 和 SQL Server 使用相同的订阅。

	![选择-ad][8]
2. 在左侧标题中，选择“SQL 服务器”，选择你的“SQL 服务器”，然后在“SQL Server”边栏选项卡的顶部，单击“设置”。

	![ad 设置][9]
3. 在“设置”边栏选项卡中，单击“Active Directory 管理员（预览）”，并接受预览子句。
4. 在“Active Directory 管理员（预览）”边栏选项卡中，单击以查看，然后单击“确定”接受预览条款。
5. 在“Active Directory 管理员（预览）”边栏选项卡中，单击“Active Directory 管理员”，然后在顶部，单击“设置管理员”。
6. 在“添加管理员”边栏选项卡中，搜索某个用户，选择该用户或组为管理员，然后单击“选择”。（“Active Directory 管理员”边栏选项卡将显示你的 Active Directory 的所有成员和组。若用户或组为灰显，则无法选择，因为不支持它们作为 Azure AD 管理员。（请参阅上述 **Azure AD 功能和限制**中受支持的管理员列表。）
7. 在“Active Directory 管理员”边栏选项卡的顶部，单击“保存”。![选择管理员][10]

	更改管理员的过程可能需要几分钟时间。然后，新管理员将出现在“Active Directory 管理员”框中。

之后若要删除管理员，请在“Active Directory 管理员”边栏选项卡的顶部，单击“删除管理员”。

### 使用 PowerShell 为 Azure SQL Server 设置 Azure AD 管理员 

若要继续，必须安装和配置最低为 0.9.8 版本的 Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/#Install)。

若要设置 Azure AD 管理员，必须执行以下 Azure PowerShell 命令：

- Add-AzureAccount -Environment AzureChinaCloud
- Select-AzureSubscription
- Switch-AzureMode -Name AzureResourceManager

用于设置和管理 Azure AD 管理员的 Cmdlet：

| Cmdlet 名称 | 说明 |
|---------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| Set-AzureSqlServerActiveDirectoryAdministrator | 为 Azure SQL Server 设置 Azure Active Directory 管理员。（必须来自当前订阅。） |
| Remove-AzureSqlServerActiveDirectoryAdministrator | 为 Azure SQL Server 删除 Azure Active Directory 管理员。 |
| Get-AzureSqlServerActiveDirectoryAdministrator | 针对为 Azure SQL Server 当前配置的 Azure Active Directory 管理员返回相关信息。 |

使用 PowerShell 命令 get-help 查看每个这些命令的更多详细信息，例如 ``get-help Set-AzureSqlServerActiveDirectoryAdministrator``。

以下脚本为一个名为 **Group-23** 的资源组中的 **demo\_server** 服务器设置一个 Azure AD 管理员组，其名为 **DBA\_Group**（对象 ID `40b79501-b343-44ed-9ce7-da4c8cc7353f`）：

```
Set-AzureSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23" 
–ServerName "demo_server" -DisplayName "DBA_Group"
```

**DisplayName** 输入参数接受 Azure AD 显示名称或用户主体名称。例如，``DisplayName="John Smith"`` 和 ``DisplayName="johns@contoso.com"``。对于 Azure AD 组，只支持 Azure AD 显示名称。

> [AZURE.NOTE]Azure PowerShell 命令 ```Set-AzureSqlServerActiveDirectoryAdministrator``` 不会阻止你为不受支持的用户设置 Azure AD 管理员。可以设置不受支持的用户，但其无法连接到数据库。（请参阅上述 **Azure AD 功能和限制**中受支持的管理员列表。）

以下示例使用可选的 **ObjectID**：

```
Set-AzureSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23" 
–ServerName "demo_server" -DisplayName "DBA_Group" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353f"
```

> [AZURE.NOTE]**DisplayName** 并不唯一时需要使用 Azure AD **ObjectID**。若要检索 **ObjectID** 和 **DisplayName** 值，请使用 Azure 门户的 Active Directory 部分，并查看用户或组的属性。

下面的示例针对 Azure SQL Server 的当前 Azure AD 管理员返回相关信息：

```
Get-AzureSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23" –ServerName "demo_server" | Format-List
```

下面的示例删除一个 Azure AD 管理员：
```
Remove-AzureSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" –ServerName "demo_server"
```

## 5\.配置客户端计算机
 
在所有客户端计算机上，如果你的应用程序或用户从中使用 Azure AD 标识连接到 Azure SQL 数据库，则必须安装以下软件：

- 若要下载 .NET Framework 4.6 或更高版本，请访问 [https://msdn.microsoft.com/zh-cn/library/5a4x27ek.aspx](https://msdn.microsoft.com/zh-cn/library/5a4x27ek.aspx)。
- 用于 SQL Server 的 Azure Active Directory 身份验证库 (**ADALSQL.DLL**) 提供多个语言版本（x86 和 amd64），请访问[用于 Microsoft SQL Server 的 Microsoft Active Directory 身份验证库](http://www.microsoft.com/download/details.aspx?id=48742)的下载中心下载。

### 工具

- 安装 [SQL Server 2016 Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 或[用于 Visual Studio 2015 的 SQL Server Data Tools](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx) 以符合 .NET Framework 4.6 要求。 
- SSMS 安装 **ADALSQL.DLL** 的 x86 版本。 
- SSMS 安装 **ADALSQL.DLL** 的 amd64 版本。SSDT 仅部分支持 Azure AD 身份验证。
- [Visual Studio 下载](https://www.visualstudio.com/downloads/download-visual-studio-vs)提供的最新 Visual Studio 符合 .NET Framework 4.6 要求，但并未安装所需的 amd64 版本 **ADALSQL.DLL**。

## 6\.在映射到 Azure AD 标识的数据库中创建包含的数据库用户 

### 关于包含数据库用户

Azure Active Directory 身份验证要求以包含数据库用户的身份创建数据库用户。基于 Azure AD 标识的包含数据库用户是在 master 数据库中不具有登录名的数据库用户，它映射到与数据库相关联的 Azure AD 目录中的一个标识。Azure AD 标识可以是单独的用户帐户，也可以是组。有关包含数据库用户的详细信息，请参阅[包含数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)。

### 通过使用 SQL Server Management Studio 连接到用户数据库
 
若要确认 Azure AD 管理员已正确设置，请使用 Azure AD 管理员帐户连接到 **master** 数据库。若要设置基于 Azure AD 的包含数据库用户（而不是拥有数据库的服务器管理员），请使用有数据库访问权限的 Azure AD 标识连接到数据库。

> [AZURE.IMPORTANT][SQL Server 2016 Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 对 Azure Active Directory 身份验证提供支持。

#### 使用 Active Directory 集成的身份验证进行连接 

如果你从联合域使用 Azure Active Directory 凭据登录到 Windows，则使用此方法。

1. 在“连接到数据库引擎”（或“连接到服务器”）对话框中启动 Management Studio，在“身份验证”框中，选择“Active Directory 集成的身份验证”。由于会为连接提供你的现有凭据，因此无需密码，也无法输入密码。
2. 单击“选项”按钮，然后在“连接属性”页面上的“连接到数据库”框中，键入你所要连接用户数据库的名称。

#### 使用 Active Directory 密码身份验证进行连接 

在使用 Azure AD 托管域与 Azure AD 主体名称连接时使用此方法。你还可以将其用于无需访问该域的联合帐户，例如远程工作时。

如果从未与 Azure 联合的域中使用凭证登录到 Windows，或者基于初始域或客户端域通过 Azure AD 使用 Azure AD 身份验证时，请使用此方法。

1. 在“连接到数据库引擎”（或“连接到服务器”）对话框中启动 Management Studio，在“身份验证”框中，选择“Active Directory 密码身份验证”。
2. 在“用户名”框中，以格式 **username@domain.com** 键入你的 Azure Active Directory 用户名称。这必须是来自 Azure Active Directory 的帐户或来自与 Azure Active Directory 联合的域的帐户。
3. 在“密码”框中，为 Azure Active Directory 帐户或联合域帐户键入你的用户密码。
4. 单击“选项”按钮，然后在“连接属性”页面上的“连接到数据库”框中，键入你所要连接用户数据库的名称。


### 在用户数据库中创建 Azure AD 包含数据库用户

若要创建基于 Azure AD 的包含数据库用户（而不是拥有数据库的服务器管理员），请以至少拥有**更改任意用户**权限的用户身份使用 Azure AD 标识（如前面过程中所述）连接到数据库。然后，使用以下 Transact-SQL 语法：

	CREATE USER Azure_AD_principal_name 
	FROM EXTERNAL PROVIDER;


*Azure\_AD\_principal\_name* 可以是 Azure AD 用户的用户主体名称，也可以是 Azure AD 组的显示名称。

**示例：**若要创建代表 Azure AD 联合或托管域用户的包含数据库用户：

	CREATE USER [bob@contoso.com] FROM EXTERNAL PROVIDER;
	CREATE USER [alice@fabrikam.partner.onmschina.cn] FROM EXTERNAL PROVIDER;

若要创建代表 Azure AD 联合域组的包含数据库用户：

	CREATE USER [Nurses] FROM EXTERNAL PROVIDER;


有关基于 Azure Active Directory 标识创建包含数据库用户的详细信息，请参阅[创建用户 (TRANSACT-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173463.aspx)。

创建数据库用户时，该用户会收到**连接**权限，并能够以**公共**角色的成员身份连接到该数据库。最初，仅供用户使用的权限是授予**公共**角色的任何权限，或者授予其所属任何 Windows 组的任何权限。设置基于 Azure AD 的包含数据库用户后，便可以授予用户其他权限，方法与向任何其他类型用户授予权限相同。通常，将权限授予数据库角色，并将用户添加到角色。有关详细信息，请参阅[数据库引擎权限基础知识](http://social.technet.microsoft.com/wiki/contents/articles/4433.database-engine-permission-basics.aspx)。有关特殊 SQL 数据库角色的详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins)。如果将联合域用户导入到管理域，则此用户必须使用托管域标识。

> [AZURE.NOTE]Azure AD 用户在数据库元数据中均标记为类型 E (EXTERNAL\_USER)，而组则标记为类型 X (EXTERNAL\_GROUPS)。有关详细信息，请参阅 [sys.database\_principals](https://msdn.microsoft.com/zh-cn/library/ms187328.aspx)。


## 7\.通过使用 Azure Active Directory 标识连接到你的数据库
 
Azure Active Directory 身份验证支持使用 Azure AD 标识连接到数据库的以下方法：

- 使用集成的 Windows 身份验证
- 使用 Azure AD 主体名称和密码

### 7\.1.使用集成的 (Windows) 身份验证进行连接
 
若要使用集成的 Windows 身份验证，域的 Active Directory 必须与 Azure Active Directory 联合，而且必须在使用用户的域凭据已加入域的计算机上运行与数据库连接的客户端应用程序（或服务）。

若要使用集成的身份验证和 Azure AD 标识连接到数据库，数据库连接字符串中的身份验证关键字必须设置为 Active Directory Integrated。下面的 C# 代码示例使用 ADO.NET。

	string ConnectionString = 
	@"Data Source=n9lxnyuzhv.database.chinacloudapi.cn; Authentication=Active Directory Integrated;";
	SqlConnection conn = new SqlConnection(ConnectionString);
	conn.Open();

请注意，对连接字符串关键字 ``Integrated Security=True`` 不支持连接到 Azure SQL 数据库。

### 7\.2.使用 Azure AD 主体名称和密码进行连接 
若要使用集成的身份验证和 Azure AD 标识连接到数据库，身份验证关键字必须设置为 Active Directory Password，而连接字符串必须包含 User ID/UID 和 Password/PWD 关键字和值。下面的 C# 代码示例使用 ADO.NET。

	string ConnectionString =
	  @"Data Source=n9lxnyuzhv.database.chinacloudapi.cn; Authentication=Active Directory Password; UID=bob@contoso.partner.onmschina.cn; PWD=MyPassWord!";
	SqlConnection conn = new SqlConnection(ConnectionString);
	conn.Open();

有关与 Azure AD 身份验证相关的特定代码示例，请参阅 MSDN 上的 [SQL Server 安全博客](http://blogs.msdn.com/b/sqlsecurity/)。

## 另请参阅

[管理 Azure SQL 数据库的数据库和登录名](/documentation/articles/sql-database-manage-logins)

[关于包含数据库用户](https://msdn.microsoft.com/zh-cn/library/ff929071.aspx)

[创建用户 (TRANSACT-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173463.aspx)

<!--Image references-->

[1]: ./media/sql-database-aad-authentication/1aad-auth-diagram.png
[2]: ./media/sql-database-aad-authentication/2subscription-relationship.png
[3]: ./media/sql-database-aad-authentication/3admin-structure.png
[4]: ./media/sql-database-aad-authentication/4select-subscription.png
[5]: ./media/sql-database-aad-authentication/5ad-settings-portal.png
[6]: ./media/sql-database-aad-authentication/6edit-directory-select.png
[7]: ./media/sql-database-aad-authentication/7edit-directory-confirm.png
[8]: ./media/sql-database-aad-authentication/8choose-ad.png
[9]: ./media/sql-database-aad-authentication/9ad-settings.png
[10]: ./media/sql-database-aad-authentication/10choose-admin.png

<!---HONumber=74-->