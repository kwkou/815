<properties
    pageTitle="在基于 Linux 的 HDInsight 中使用 Hadoop Sqoop | Azure"
    description="了解如何在 HDInsight 群集上基于 Linux 的 Hadoop 和 Azure SQL 数据库之间运行 Sqoop 导入和导出。"
    editor="cgronlun"
    manager="jhubbard"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="303649a5-4be5-4933-bf1d-4b232083c354"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/14/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="455bb9637f3a2604c45404315ed737edafeeaea8"
    ms.lasthandoff="04/28/2017" />

# <a name="use-sqoop-with-hadoop-in-hdinsight-ssh"></a>将 Sqoop 与 HDInsight 中的 Hadoop 配合使用 (SSH)

[AZURE.INCLUDE [sqoop-selector](../../includes/hdinsight-selector-use-sqoop.md)]

了解如何使用 Sqoop 在 HDInsight 群集和 Azure SQL 数据库或 SQL Server 数据库之间进行导入和导出。

> [AZURE.IMPORTANT]
> 本文档中的步骤仅适用于使用 Linux 的 HDInsight 群集。 Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a name="install-freetds"></a>安装 FreeTDS

1. 使用 SSH 连接到基于 Linux 的 HDInsight 群集。 连接时要使用的地址为 `CLUSTERNAME-ssh.azurehdinsight.cn`，端口为 `22`。

    有关详细信息，请参阅[对 HDInsight 使用 SSH](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

2. 使用以下命令安装 FreeTDS：

        sudo apt-get --assume-yes install freetds-dev freetds-bin

    在多个步骤中使用 FreeTDS 连接到 SQL 数据库。

## <a name="create-the-table-in-sql-database"></a>在 SQL 数据库中创建表

> [AZURE.IMPORTANT]
> 如果使用的是在[创建群集和 SQL 数据库](/documentation/articles/hdinsight-use-sqoop/)中创建的 HDInsight 群集和 SQL 数据库，则忽略本节中的步骤。 创建数据库和表，作为[创建群集和 SQL 数据库](/documentation/articles/hdinsight-use-sqoop/)文档中的一部分步骤。

1. 在 SSH 会话中，使用以下命令连接到 SQL 数据库服务器。

        TDSVER=8.0 tsql -H <serverName>.database.chinacloudapi.cn -U <adminLogin> -P <adminPassword> -p 1433 -D sqooptest

    将收到类似于以下文本的输出：

        locale is "en_US.UTF-8"
        locale charset is "UTF-8"
        using default charset "UTF-8"
        Default database being set to sqooptest
        1>

2. 在 `1>` 提示符下，输入以下查询：

        CREATE TABLE [dbo].[mobiledata](
        [clientid] [nvarchar](50),
        [querytime] [nvarchar](50),
        [market] [nvarchar](50),
        [deviceplatform] [nvarchar](50),
        [devicemake] [nvarchar](50),
        [devicemodel] [nvarchar](50),
        [state] [nvarchar](50),
        [country] [nvarchar](50),
        [querydwelltime] [float],
        [sessionid] [bigint],
        [sessionpagevieworder] [bigint])
        GO
        CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(clientid)
        GO

    输入 `GO` 语句后，将评估前面的语句。 首先会创建 **mobiledata** 表，然后会向其添加聚集索引（根据 SQL 数据库的要求。）

    使用以下查询验证是否已创建该表：

        SELECT * FROM information_schema.tables
        GO

    将看到类似于以下文本的输出：

        TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
        sqooptest       dbo     mobiledata      BASE TABLE

3. 在 `1>` 提示符下输入 `exit` 以退出 tsql 实用工具。

## <a name="sqoop-export"></a>Sqoop 导出

1. 通过 SSH 连接到 HDInsight 后，使用以下命令验证 Sqoop 是否可以看到你的 SQL 数据库：

        sqoop list-databases --connect jdbc:sqlserver://<serverName>.database.chinacloudapi.cn:1433 --username <adminLogin> --password <adminPassword>

    此命令将返回数据库列表，其中包括之前创建的 **sqooptest** 数据库。

2. 使用以下命令将数据从 **hivesampletable** 导出到 **mobiledata** 表：

        sqoop export --connect 'jdbc:sqlserver://<serverName>.database.chinacloudapi.cn:1433;database=sqooptest' --username <adminLogin> --password <adminPassword> --table 'mobiledata' --export-dir 'wasbs:///hive/warehouse/hivesampletable' --fields-terminated-by '\t' -m 1

    此命令指示 Sqoop 连接到 **sqooptest** 数据库。 然后，Sqoop 将数据从 **wasbs:///hive/warehouse/hivesampletable** 导出到 **mobiledata** 表。

3. 该命令完成后，使用以下命令通过 TSQL 连接到数据库：

        TDSVER=8.0 tsql -H <serverName>.database.chinacloudapi.cn -U <adminLogin> -P <adminPassword> -p 1433 -D sqooptest

    连接成功以后，使用以下语句验证数据是否已导出到 **mobiledata** 表：

        SELECT * FROM mobiledata
        GO

    你会在表中看到一系列数据。 键入 `exit` 退出 tsql 实用程序。

## <a name="sqoop-import"></a>Sqoop 导入

1. 使用以下命令将数据从 SQL 数据库中的 **mobiledata** 表导入 HDInsight 上的 **wasbs:///tutorials/usesqoop/importeddata** 目录：

        sqoop import --connect 'jdbc:sqlserver://<serverName>.database.chinacloudapi.cn:1433;database=sqooptest' --username <adminLogin> --password <adminPassword> --table 'mobiledata' --target-dir 'wasbs:///tutorials/usesqoop/importeddata' --fields-terminated-by '\t' --lines-terminated-by '\n' -m 1

    数据中的字段将通过制表符分隔，并且相关行将由换行符终止。

2. 完成导入后，可使用以下命令在新目录中列出这些数据：

        hadoop fs -text wasbs:///tutorials/usesqoop/importeddata/part-m-00000

## <a name="using-sql-server"></a>使用 SQL Server

你还可以使用 Sqoop 通过 SQL Server 来导入和导出数据，不管是在数据中心进行，还是在托管在 Azure 中的虚拟机上进行。 SQL 数据库和 SQL Server 在使用方面的差异是：

* HDInsight 和 SQL Server 必须位于同一 Azure 虚拟网络上

    > [AZURE.NOTE]
    > HDInsight 仅支持基于位置的虚拟网络，并且当前不适用于基于地缘组的虚拟网络。

    在数据中心使用 SQL Server 时，必须将虚拟网络配置为“站点到站点”或“点到站点”。

    > [AZURE.NOTE]
    > 使用**点到站点**虚拟网络时，SQL Server 必须运行 VPN 客户端配置应用程序。 VPN 客户端可在 Azure 虚拟网络配置的**仪表板**中使用。

    有关 Azure 虚拟网络的详细信息，请参阅[虚拟网络概述](/documentation/articles/virtual-networks-overview/)。

* 必须将 SQL Server 配置为允许 SQL 身份验证。 有关详细信息，请参阅[选择身份验证模式](https://msdn.microsoft.com/ms144284.aspx)文档。

* 你可能需要将 SQL Server 配置为接受远程连接。 有关详细信息，请参阅[如何解决 SQL Server 数据库引擎的连接问题](http://social.technet.microsoft.com/wiki/contents/articles/2102.how-to-troubleshoot-connecting-to-the-sql-server-database-engine.aspx)文档。

* 使用诸如 **SQL Server Management Studio** 或 **tsql** 等实用工具在 SQL Server 中创建 **sqooptest** 数据库。 有关使用 Azure CLI 的步骤仅适用于 Azure SQL 数据库。

    使用以下 TSQL 语句创建 **mobiledata** 表：

        CREATE TABLE [dbo].[mobiledata](
        [clientid] [nvarchar](50),
        [querytime] [nvarchar](50),
        [market] [nvarchar](50),
        [deviceplatform] [nvarchar](50),
        [devicemake] [nvarchar](50),
        [devicemodel] [nvarchar](50),
        [state] [nvarchar](50),
        [country] [nvarchar](50),
        [querydwelltime] [float],
        [sessionid] [bigint],
        [sessionpagevieworder] [bigint])

* 在从 HDInsight 连接到 SQL Server 时，可能需要使用 SQL Server 的 IP 地址。 例如：

        sqoop import --connect 'jdbc:sqlserver://10.0.1.1:1433;database=sqooptest' --username <adminLogin> --password <adminPassword> --table 'mobiledata' --target-dir 'wasbs:///tutorials/usesqoop/importeddata' --fields-terminated-by '\t' --lines-terminated-by '\n' -m 1

## <a name="limitations"></a>限制

* 批量导出 - 在基于 Linux 的 HDInsight 上，用于将数据导出到 Microsoft SQL Server 或 Azure SQL 数据库的 Sqoop 连接器目前不支持批量插入。

* 批处理 - 在基于 Linux 的 HDInsight 上，如果在执行插入时使用 `-batch` 开关，Sqoop 将进行多次插入而不是批处理插入操作。

## <a name="next-steps"></a>后续步骤

现在你已了解如何使用 Sqoop。 若要了解更多信息，请参阅以下文章：

* [将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]：在 Oozie 工作流中使用 Sqoop 操作。
* [使用 HDInsight 分析航班延误数据][hdinsight-analyze-flight-data]：使用 Hive 分析航班延误数据，然后使用 Sqoop 将数据导出到 Azure SQL 数据库。
* [将数据上传到 HDInsight][hdinsight-upload-data]：了解将数据上传到 HDInsight/Azure Blob 存储的其他方法。

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning/
[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-analyze-flight-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/
[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[sqldatabase-get-started]: /documentation/articles/sql-database-get-started/
[sqldatabase-create-configue]: /documentation/articles/sql-database-create-configure/

[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-install]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[powershell-script]: https://msdn.microsoft.com/powershell/scripting/getting-started/fundamental/using-windows-powershell

[sqoop-user-guide-1.4.4]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html