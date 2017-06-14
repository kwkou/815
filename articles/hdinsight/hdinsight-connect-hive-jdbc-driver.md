<properties
    pageTitle="使用 JDBC 在 Azure HDInsight 上查询 Hive"
    description="了解如何使用 JDBC 连接到 Azure HDInsight 上的 Hive，以及如何通过远程方式对存储在云中的数据运行查询。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="928f8d2a-684d-48cb-894c-11c59a5599ae"
    ms.service="hdinsight"
    ms.devlang="java"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="02/13/2017"
    wacn.date="03/24/2017"
    ms.author="larryfr" />

# 使用 Hive JDBC 驱动程序连接到 Azure HDInsight 上的 Hive

[AZURE.INCLUDE [ODBC-JDBC-selector](../../includes/hdinsight-selector-odbc-jdbc.md)]

在本文档中，学习如何使用 Java 应用程序中的 JDBC 将 Hive 查询远程提交到 HDInsight 群集。学习如何从 SQuirreL SQL 客户端进行连接，以及如何通过 Java 以编程方式进行连接。

有关 Hive JDBC 接口的详细信息，请参阅 [HiveJDBCInterface](https://cwiki.apache.org/confluence/display/Hive/HiveJDBCInterface)。

## 先决条件

要完成本文中的步骤，需要：

* HDInsight 群集上的 Hadoop。可以使用基于 Linux 或基于 Windows 的群集。

    > [AZURE.IMPORTANT]
    Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* [SQuirreL SQL](http://squirrel-sql.sourceforge.net/)。SQuirreL 是 JDBC 客户端应用程序。

要生成和运行本文中链接的 Java 应用程序示例，需要以下软件。

* [Java 开发人员工具包 (JDK) 版本 7](https://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) 或更高版本。
* [Apache Maven](https://maven.apache.org)。Maven 是适用于 Java 项目的项目生成系统，可供与本文相关的项目使用。

## 连接字符串

JDBC 通过 443 连接到 Azure 上的 HDInsight 群集，并使用 SSL 保护通信安全。公用网关（群集位于其后）会将通信重定向到 HiveServer2 实际进行侦听的端口。因此，典型连接字符串将如下所示：

    jdbc:hive2://CLUSTERNAME.azurehdinsight.cn:443/default;ssl=true?hive.server2.transport.mode=http;hive.server2.thrift.http.path=/hive2

将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

## 身份验证

建立连接时，必须使用 HDInsight 群集管理员名称和密码对群集网关进行身份验证。从 JDBC 客户端（如 SQuirreL SQL）进行连接时，必须在客户端设置中输入管理员名称和密码。

从 Java 应用程序建立连接时，必须使用该名称和密码。例如，以下 Java 代码使用连接字符串、管理员名称和密码打开新连接：

    DriverManager.getConnection(connectionString,clusterAdmin,clusterPassword);

## 使用 SQuirreL SQL 客户端进行连接

SQuirreL SQL 是一个 JDBC 客户端，可用于通过 HDInsight 群集远程运行 Hive 查询。以下步骤假设已安装 SQuirreL SQL，并引导下载和配置 Hive 的驱动程序。

1. 从 HDInsight 群集复制 Hive JDBC 驱动程序。
   
    * 对于**基于 Linux 的 HDInsight**，请使用以下步骤下载所需的 jar 文件。
     
        1. 创建一个新的目录以包含这些文件。例如，`mkdir hivedriver`。
        2. 从命令提示符、Bash、PowerShell 或其他命令行提示符，将目录切换到新目录并使用以下命令从 HDInsight 群集复制文件。
        
                scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/hive-jdbc*standalone.jar .
                scp USERNAME@CLUSTERNAME:/usr/hdp/current/hadoop-client/hadoop-common.jar .
                scp USERNAME@CLUSTERNAME:/usr/hdp/current/hadoop-client/hadoop-auth.jar .
        
            将 **USERNAME** 替换为群集的 SSH 用户帐户名称。将 **CLUSTERNAME** 替换为 HDInsight 群集名称。
        
        > [AZURE.NOTE]
        在 Windows 环境中，可能没有 `scp` 命令。如果是这样，请改用 PSCP 实用程序。可以从 [http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 安装该程序。

    * 对于**基于 Windows 的 HDInsight**，请使用以下步骤下载 jar 文件。
     
        1. 在 Azure 门户中，选择 HDInsight 群集，然后选择“远程桌面”图标。
        
            ![“远程桌面”图标](./media/hdinsight-connect-hive-jdbc-driver/remotedesktopicon.png)  

        2. 在“远程桌面”边栏选项卡上，使用“连接”按钮连接到群集。如果未启用远程桌面，请使用表单提供用户名和密码，然后选择“启用”为群集启用远程桌面。
        
            ![“远程桌面”边栏选项卡](./media/hdinsight-connect-hive-jdbc-driver/remotedesktopblade.png)  

            选择“连接”后，将会下载 .rdp 文件。使用此文件启动远程桌面客户端。出现提示时，使用输入的用户名和密码进行远程桌面访问。

        3. 连接后，将以下文件从远程桌面会话复制到本地计算机。将其放置在名为 `hivedriver` 的本地目录中。
        
            * C:\\apps\\dist\\hive-0.14.0.2.2.9.1-7\\lib\\hive-jdbc-0.14.0.2.2.9.1-7-standalone.jar
            * C:\\apps\\dist\\hadoop-2.6.0.2.2.9.1-7\\share\\hadoop\\common\\hadoop-common-2.6.0.2.2.9.1-7.jar
            * C:\\apps\\dist\\hadoop-2.6.0.2.2.9.1-7\\share\\hadoop\\common\\lib\\hadoop-auth-2.6.0.2.2.9.1-7.jar
          
            > [AZURE.NOTE]
            群集的路径和文件名中包含的版本号可能会有所不同。

        4. 文件复制完成后，断开远程桌面会话的连接。

2. 启动 SQuirreL SQL 应用程序。在窗口左侧，选择“驱动程序”。
   
    ![窗口左侧的“驱动程序”选项卡](./media/hdinsight-connect-hive-jdbc-driver/squirreldrivers.png)

3. 从“驱动程序”对话框顶部的图标中，选择“+”图标创建新驱动程序。
   
    ![驱动程序图标](./media/hdinsight-connect-hive-jdbc-driver/driversicons.png)

4. 在“添加驱动程序”对话框中，添加以下信息。
   
    * **名称**：Hive
    * **示例 URL**：`jdbc:hive2://localhost:443/default;ssl=true?hive.server2.transport.mode=http;hive.server2.thrift.http.path=/hive2`
    * **额外类路径**：使用“添加”按钮添加之前下载的 jar 文件
    * **类名**：org.apache.hive.jdbc.HiveDriver
    
    ![添加驱动程序对话框](./media/hdinsight-connect-hive-jdbc-driver/adddriver.png)
    
    单击“确定”以保存这些设置。

5. 在 SQuirreL SQL 窗口左侧，选择“别名”。然后，单击“+”图标创建新连接别名。
   
    ![添加新的别名](./media/hdinsight-connect-hive-jdbc-driver/aliases.png)

6. 对“添加别名”对话框使用以下值。
   
    * **名称**：Hive on HDInsight

    * **驱动程序**：使用下拉列表选择 **Hive** 驱动程序

    * **URL**：jdbc:hive2://CLUSTERNAME.azurehdinsight.cn:443/default;ssl=true?hive.server2.transport.mode=http;hive.server2.thrift.http.path=/hive2
     
        将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

    * **用户名**：HDInsight 群集的群集登录帐户名。默认为 `admin`。

    * **密码**：群集登录帐户的密码。该密码是在创建 HDInsight 群集时提供的密码。
     
    ![添加别名对话框](./media/hdinsight-connect-hive-jdbc-driver/addalias.png)
     
    使用“测试”按钮验证连接是否有效。出现“连接到: Hive on HDInsight”对话框时，选择“连接”进行测试。如果测试成功，将显示“连接成功”对话框。
     
    使用“添加别名”对话框底部的“确定”按钮保存连接别名。

7. 在 SQuirreL SQL 顶部的“连接到”下拉列表中，选择“Hive on HDInsight”。出现提示时，选择“连接”。
   
    ![连接对话框](./media/hdinsight-connect-hive-jdbc-driver/connect.png)

8. 连接后，在 SQL 查询对话框中输入以下查询，然后选择“运行”图标。结果区域会显示查询的结果。
   
        select * from hivesampletable limit 10;
   
    ![sql 查询对话框，其中包括结果](./media/hdinsight-connect-hive-jdbc-driver/sqlquery.png)

## 从 Java 应用程序示例进行连接

使用 Java 客户端查询 Hive on HDInsight 的示例位于 [https://github.com/Azure-Samples/hdinsight-java-hive-jdbc](https://github.com/Azure-Samples/hdinsight-java-hive-jdbc)。按照存储库中的说明生成并运行该示例。

>[AZURE.NOTE] 此示例是针对全球 Azure 编写。对于 Azure 中国区，需要在连接字符串中将“azurehdinsight.net”替换为“azurehdinsight.cn”。

## 故障排除

### 尝试打开 SQL 连接时发生意外错误

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

**症状**：连接到 HDInsight 群集 3.3 版或 3.4 版时，可能会遇到意外错误。此错误的堆栈跟踪将以下列行为开头：

    java.util.concurrent.ExecutionException: java.lang.RuntimeException: java.lang.NoSuchMethodError: org.apache.commons.codec.binary.Base64.<init>(I)V
    at java.util.concurrent.FutureTas...(FutureTask.java:122)
    at java.util.concurrent.FutureTask.get(FutureTask.java:206)

**原因**：之所以出现此错误，是因为 SQuirreL 使用的 commons-codec.jar 文件版本与 Hive JDBC 组件所需的、从 HDInsight 群集下载的文件版本不匹配。

**解决方法**：要解决此错误，请使用以下步骤。

1. 从 HDInsight 群集下载 commons-codec jar 文件。
   
        scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/commons-codec*.jar ./commons-codec.jar

2. 退出 SQuirreL，然后转到系统上安装 SQuirreL 的目录。在 SquirreL 目录的 `lib` 目录下，将现有的 commons-codec.jar 替换为从 HDInsight 群集下载的文件。

3. 重新启动 SQuirreL。连接到 HDInsight 上的 Hive 时，应不再会出现该错误。

## 后续步骤

现在，已了解如何将 JDBC 与 Hive 配合使用，请使用以下链接学习 Azure HDInsight 的其他用法。

* [将数据上载到 HDInsight](/documentation/articles/hdinsight-upload-data/)
* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: add note about windows cluster being abandoned-->