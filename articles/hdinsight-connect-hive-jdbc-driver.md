<properties
 pageTitle="使用 JDBC 在 Azure HDInsight 上查询 Hive"
 description="了解如何使用 JDBC 连接到 Azure HDInsight 上的 Hive，以及如何通过远程方式对存储在云中的数据运行查询。"
 services="hdinsight"
 documentationCenter=""
 authors="Blackmist"
 manager="paulettm"
 editor="cgronlun"/>

<tags
 ms.service="hdinsight"
 ms.date="07/17/2015"
 wacn.date="08/29/2015"/>

#使用 Hive JDBC 驱动程序连接到 Azure HDInsight 上的 Hive

[AZURE.INCLUDE [ODBC-JDBC-selector](../includes/hdinsight-selector-odbc-jdbc.md)]

在本文档中，你将学习如何使用 Java 应用程序中的 JDBC 将 Hive 查询远程提交到 HDInsight 群集。有关 Hive JDBC 接口的详细信息，请参阅 [HiveJDBCInterface](https://cwiki.apache.org/confluence/display/Hive/HiveJDBCInterface)。

##先决条件

若要完成本文中的步骤，你将需要：

* HDInsight 群集上的 Hadoop。可以使用基于 Windows 的群集。

* [Java 开发人员工具包 (JDK) 版本 7](https://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) 或更高版本。

* [Apache Maven](https://maven.apache.org)。Maven 是适用于 Java 项目的项目生成系统，可供与本文相关的项目使用。

##连接字符串

连接到 Azure 上的 HDInsight 群集的 JDBC 连接是通过 443 进行的，使用 SSL 来保护通信安全。公用网关（群集位于其后）会将通信重定向到 HiveServer2 在其上进行实际侦听的端口。因此，典型的连接字符串将如下所示：

    jdbc:hive2://CLUSTERNAME.azurehdinsight.cn:443/default;ssl=true?hive.server2.transport.mode=http;hive.server2.thrift.http.path=/hive2

##身份验证

建立连接时，必须指定 HDInsight 群集管理员名称和密码。这些凭据用于对提交到网关的请求进行身份验证。例如，以下 Java 代码将使用连接字符串、管理员名称和密码打开新的连接：

    DriverManager.getConnection(connectionString,clusterAdmin,clusterPassword);

##查询

一旦建立连接，即可针对 Hive 运行查询。例如，以下 Java 代码对表执行 __SELECT__ 操作，将结果限制为只包括三行，然后显示结果：

    sql = "SELECT querytime, market, deviceplatform, devicemodel, state, country from " + tableName + " LIMIT 3";
    stmt2 = conn.createStatement();
    System.out.println("\nRetrieving inserted data:");

    res2 = stmt2.executeQuery(sql);

    while (res2.next()) {
      System.out.println( res2.getString(1) + "\t" + res2.getString(2) + "\t" + res2.getString(3) + "\t" + res2.getString(4) + "\t" + res2.getString(5) + "\t" + res2.getString(6));
    }

##Java 项目示例

使用 Java 客户端在 HDInsight 上查询 Hive 的示例位于 [https://github.com/Blackmist/hdinsight-hive-jdbc](https://github.com/Blackmist/hdinsight-hive-jdbc)。按照存储库中的说明生成并运行该示例。

##后续步骤

现在，你已了解如何将 JDBC 与 Hive 配合使用，请使用以下链接来学习 Azure HDInsight 的其他用法。

* [将数据上载到 HDInsight](/documentation/articles/hdinsight-upload-data)
* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig)
* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce)

<!---HONumber=67-->