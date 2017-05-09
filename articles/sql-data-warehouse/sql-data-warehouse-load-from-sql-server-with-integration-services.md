<properties
    pageTitle="将数据从 SQL Server 载入 Azure SQL 数据仓库 (SSIS)| Azure"
    description="演示如何创建 SQL Server Integration Services (SSIS) 包，以便将数据从各种数据源移动到 SQL 数据仓库。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="douglaslms"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="e2c252e9-0828-47c2-a808-e3bea46c134a"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.custom="loading"
    ms.date="03/30/2017"
    wacn.date="05/08/2017"
    ms.author="douglasl;barbkess"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="b94004f56f0be0f2e4c89dccaa94f8f2d00e27e8"
    ms.lasthandoff="04/28/2017" />

# <a name="load-data-from-sql-server-into-azure-sql-data-warehouse-ssis"></a>将数据从 SQL Server 载入 Azure SQL 数据仓库 (SSIS)
> [AZURE.SELECTOR]
- [SSIS](/documentation/articles/sql-data-warehouse-load-from-sql-server-with-integration-services/)
- [PolyBase](/documentation/articles/sql-data-warehouse-load-from-sql-server-with-polybase/)
- [bcp](/documentation/articles/sql-data-warehouse-load-from-sql-server-with-bcp/)

创建 SQL Server Integration Services (SSIS) 包，以将 SQL Server 的数据加载到 Azure SQL 数据仓库。 可以在数据通过 SSIS 数据流进行传递时，选择性地对其进行重构、转换和清理。

在本教程中，您将：

* 在 Visual Studio 中创建新的 Integration Services 项目。
* 连接到数据源，包括 SQL Server（作为源）和 SQL 数据仓库（作为目标）。
* 设计一个将数据从源加载到目标的 SSIS 包。
* 运行 SSIS 包以加载数据。

本教程使用 SQL Server 作为数据源。 SQL Server 可在本地或 Azure 虚拟机中运行。

## <a name="basic-concepts"></a>基本概念
该包是 SSIS 中的工作单元。 项目中对相关的包进行了分组。 在 Visual Studio 中使用 SQL Server Data Tools 创建项目并设计包。 设计过程是一个可视的过程，在其中你可以将组件从“工具箱”拖放到设计图面，然后连接组件，并设置其属性。 完成包后，可以选择性地将其部署到 SQL Server，以进行全面管理、监视并保障安全性。

## <a name="options-for-loading-data-with-ssis"></a>使用 SSIS 加载数据的选项
SQL Server Integration Services (SSIS) 是一套灵活的工具，可提供连接到 SQL 数据仓库以及将数据加载到其中的各种选项。

1. 使用 ADO NET 目标连接到 SQL 数据仓库。 本教程使用 ADO NET 目标，因为它的配置选项最少。
2. 使用 OLE DB 目标连接到 SQL 数据仓库。 此选项可能会提供比 ADO NET 目标稍好的性能。
3. 使用 Azure Blob 上传任务将数据暂存在 Azure Blob 存储中。 然后使用 SSIS 执行 SQL 任务以启动将数据加载到 SQL 数据仓库的 Polybase 脚本。 此选项可提供此处列出的三个选项的最佳性能。 若要获取 Azure Blob 上传任务，请下载 [Microsoft SQL Server 2016 Integration Services Feature Pack for Azure][Microsoft SQL Server 2016 Integration Services Feature Pack for Azure]（用于 Azure 的 Microsoft SQL Server 2016 Integration Services 功能包）。 若要了解有关 Polybase 的详细信息，请参阅 [PolyBase 指南][PolyBase Guide]。

## <a name="before-you-start"></a>开始之前
若要逐步完成本教程，你需要：

1. **SQL Server 集成服务 (SSIS)**。 SSIS 是 SQL Server 的一个组件，且需要使用 SQL Server 的评估版或许可的版本。 若要获取 SQL Server 2016 预览版的评估版本，请参阅 [SQL Server 评估][SQL Server Evaluations]。
2. **Visual Studio**。 若要获取免费的 Visual Studio Community Edition，请参阅 [Visual Studio Community][Visual Studio Community]。
3. **适用于 Visual Studio 的 SQL Server Data Tools (SSDT)**。 若要获取 SQL Server Data Tools for Visual Studio，请参阅[下载 SQL Server Data Tools (SSDT)][Download SQL Server Data Tools (SSDT)]。
4. **示例数据**。 本教程使用 AdventureWorks 示例数据库存储在 SQL Server 中的示例数据作为要加载到 SQL 数据仓库的源数据。 若要获取 AdventureWorks 示例数据库，请参阅 [AdventureWorks 2014 示例数据库][AdventureWorks 2014 Sample Databases]。
5. **SQL 数据仓库数据库和权限**。 本教程连接到 SQL 数据仓库实例，并将数据加载到其中。 必须具有权限才能创建表并加载数据。
6. **防火墙规则**。 将数据上传到 SQL 数据仓库之前，必须使用本地计算机的 IP 地址创建用于 SQL 数据仓库的防火墙规则。

## <a name="step-1-create-a-new-integration-services-project"></a>步骤 1：创建新的 Integration Services 项目
1. 启动 Visual Studio。
2. 在“文件”菜单中，选择“新建 | 项目”。
3. 导航到“安装 | 模板 | 商业智能 | 集成服务”项目类型。
4. 选择“集成服务项目”。 提供“名称”和“位置”的值，然后选择“确定”。

Visual Studio 将打开并创建新的 Integration Services (SSIS) 项目。 然后，Visual Studio 会在项目中打开用于单个新的 SSIS 包 (Package.dtsx) 的设计器。 你会看到以下屏幕区域：

* 位于左侧的是 SSIS 组件的“工具箱”  。
* 中间是带有多个选项卡的设计图面。 通常至少会使用“控制流”和“数据流”选项卡。
* 位于右侧的是“解决方案资源管理器”和“属性”窗格。

    ![][01]
## <a name="step-2-create-the-basic-data-flow"></a>步骤 2：创建基本数据流
1. 将“数据流任务”从“工具箱”拖动到设计图面的中心（在“控制流”选项卡上）。

    ![][02]
2. 双击“数据流任务”以切换到“数据流”选项卡。
3. 从“工具箱”中的“其他源”列表，将“ADO.NET 源”拖动到设计图面上。 在源适配器仍处于选中状态时，在“属性”窗格中将其名称更改为“SQL Server 源”。
4. 从“工具箱”中的“其他目标”列表，将“ADO.NET 目标”拖动到“ADO.NET 源”下的设计图面上。 在目标适配器仍处于选中状态时，在“属性”窗格中将其名称更改为“SQL DW 目标”。

    ![][09]
## <a name="step-3-configure-the-source-adapter"></a>步骤 3：配置源适配器
1. 双击源适配器以打开“ADO.NET 源编辑器”。

    ![][03]
2. 在“ADO.NET 源编辑器”的“连接管理器”选项卡上，单击“ADO.NET 连接管理器”列表旁边的“新建”按钮，以打开“配置 ADO.NET 连接管理器”对话框，然后为本教程从中加载数据的 SQL Server 数据库创建连接设置。

    ![][04]
3. 在“配置 ADO.NET 连接管理器”对话框中，单击“新建”按钮以打开“连接管理器”对话框，并创建新的数据连接。

    ![][05]
4. 在“连接管理器”对话框中，执行以下操作。

    1. 针对“提供程序”，选择 SqlClient 数据提供程序。
    2. 针对“服务器名称”，输入 SQL Server 名称。
    3. 在“登录到服务器”部分中，选择或输入身份验证信息。
    4. 在“连接到数据库”部分中，选择 AdventureWorks 示例数据库。
    5. 单击“测试连接”。

        ![][06]
    6. 在报告连接测试结果的对话框中，单击“确定”，以返回到“连接管理器”对话框。
    7. 在“连接管理器”对话框中，单击“确定”，以返回到“配置 ADO.NET 连接管理器”对话框。
5. 在“配置 ADO.NET 连接管理器”对话框中，单击“确定”，以返回到“ADO.NET 源编辑器”。
6. 在“ADO.NET 源编辑器”中，在“表或视图名称”列表中，选择“Sales.SalesOrderDetail”表。

    ![][07]
7. 单击“预览”在“预览查询结果”对话框中查看源表中的前 200 行数据。

    ![][08]
8. 在“预览查询结果”对话框中，单击“关闭”，以返回到“ADO.NET 源编辑器”。
9. 在“ADO.NET 源编辑器”中，请单击“确定”，以完成数据源配置。

## <a name="step-4-connect-the-source-adapter-to-the-destination-adapter"></a>步骤 4 ：将源适配器连接到目标适配器
1. 选择设计图面上的源适配器。
2. 选择从源适配器扩展的蓝色箭头，并将其拖动至目标编辑器中，直到其位于合适的位置。

    ![][10]

    在典型的 SSIS 包中，在源和目标之间使用“SSIS 工具箱”的其他组件，在数据通过 SSIS 数据流进行传递时对其进行重组、转换和清理。 为使此示例尽量简单，我们将源直接连接至目标。

## <a name="step-5-configure-the-destination-adapter"></a>步骤 5：配置目标适配器
1. 双击目标适配器以打开“ADO.NET 目标编辑器”。

    ![][11]
2. 在“ADO.NET 目标编辑器”的“连接管理器”选项卡上，单击“连接管理器”列表旁边的“新建”按钮，以打开“配置 ADO.NET 连接管理器”对话框，然后为本教程将向其加载数据的 Azure SQL 数据仓库数据库创建连接设置。
3. 在“配置 ADO.NET 连接管理器”对话框中，单击“新建”按钮以打开“连接管理器”对话框，并创建新的数据连接。
4. 在“连接管理器”对话框中，执行以下操作。
    1. 针对“提供程序” ，选择 SqlClient 数据提供程序。
    2. 针对“服务器名称”，输入 SQL 数据仓库名称。
    3. 在“登录到服务器”部分中，选择“使用 SQL Server 身份验证”并输入身份验证信息。
    4. 在“连接到数据库”部分中，选择现有的 SQL 数据仓库数据库。
    5. 单击“测试连接”。
    6. 在报告连接测试结果的对话框中，单击“确定”，以返回到“连接管理器”对话框。
    7. 在“连接管理器”对话框中，单击“确定”，以返回到“配置 ADO.NET 连接管理器”对话框。
5. 在“配置 ADO.NET 连接管理器”对话框中，单击“确定”，以返回到“ADO.NET 目标编辑器”。
6. 在“ADO.NET 目标编辑器”中，单击“使用表或视图”列表旁的“新建”，以打开“创建表”对话框来创建具有与源表相匹配的列列表的新目标表。
   
    ![][12a]  
7. 在“创建列表”对话框中，执行以下操作。
   
    1. 将目标表的名称更改为 **SalesOrderDetail**。
    2. 删除 **rowguid** 列。 SQL 数据仓库不支持 **uniqueidentifier** 数据类型。
    3. 将 **LineTotal** 列的数据类型更改为 **money**。SQL 数据仓库不支持 **decimal** 数据类型。 有关受支持的数据类型，请参阅 [CREATE TABLE（Azure SQL 数据仓库，并行数据仓库）][CREATE TABLE (Azure SQL Data Warehouse, Parallel Data Warehouse)]。
      
       ![][12b]  
    4. 单击“确定”以创建表并返回到“ADO.NET 目标编辑器”。
8. 在“ADO.NET 目标编辑器”中，选择“映射”选项卡以查看如何将源中的列映射到目标中的列。

    ![][13]
9. 单击“确定”，以完成数据源配置。

## <a name="step-6-run-the-package-to-load-the-data"></a>步骤 6：运行包以加载数据。
通过单击工具栏上的“启动”按钮或选中“调试”菜单上的其中一个“运行”选项来运行包。

包开始运行时，会看到指示活动的黄色旋转轮，以及到目前为止已处理的行数。

![][14]

当包完成运行时，将看到指示成功的绿色复选标记，以及从源加载到目标的数据总行数。

![][15]

祝贺你！ 你已成功使用 SQL Server Integration Services 将数据加载到 Azure SQL 数据仓库。

## <a name="next-steps"></a>后续步骤
* 了解有关 SSIS 数据流的详细信息。 从此处开始： [数据流][Data Flow]。
* 了解如何直接在设计环境中对包进行调试和故障排除。 从此处开始： [包开发的故障排除工具][Troubleshooting Tools for Package Development]。
* 了解如何将 SSIS 项目和包部署到 Integration Services 服务器或另一个存储位置。 从此处开始： [部署项目和包][Deployment of Projects and Packages]。

<!-- Image references -->
[01]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/ssis-designer-01.png
[02]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/ssis-data-flow-task-02.png
[03]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/ado-net-source-03.png
[04]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/ado-net-connection-manager-04.png
[05]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/ado-net-connection-05.png
[06]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/test-connection-06.png
[07]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/ado-net-source-07.png
[08]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/preview-data-08.png
[09]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/source-destination-09.png
[10]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/connect-source-destination-10.png
[11]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/ado-net-destination-11.png
[12a]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/destination-query-before-12a.png
[12b]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/destination-query-after-12b.png
[13]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/column-mapping-13.png
[14]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/package-running-14.png
[15]: ./media/sql-data-warehouse-load-from-sql-server-with-integration-services/package-success-15.png

<!-- Article references -->

<!-- MSDN references -->
[PolyBase Guide]: https://msdn.microsoft.com/zh-cn/library/mt143171.aspx
[Download SQL Server Data Tools (SSDT)]: https://msdn.microsoft.com/zh-cn/library/mt204009.aspx
[CREATE TABLE (Azure SQL Data Warehouse, Parallel Data Warehouse)]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx
[Data Flow]: https://msdn.microsoft.com/zh-cn/library/ms140080.aspx
[Troubleshooting Tools for Package Development]: https://msdn.microsoft.com/zh-cn/library/ms137625.aspx
[Deployment of Projects and Packages]: https://msdn.microsoft.com/zh-cn/library/hh213290.aspx

<!--Other Web references-->

[Microsoft SQL Server 2016 Integration Services Feature Pack for Azure]: http://go.microsoft.com/fwlink/?LinkID=626967
[SQL Server Evaluations]: https://www.microsoft.com/zh-cn/evalcenter/evaluate-sql-server-2016
[Visual Studio Community]: https://www.visualstudio.com/products/visual-studio-community-vs.aspx
[AdventureWorks 2014 Sample Databases]: https://msftdbprodsamples.codeplex.com/releases/view/125550

<!--Update_Description:update meta properties;wording update-->