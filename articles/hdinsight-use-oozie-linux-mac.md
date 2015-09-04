<properties
	pageTitle="在 HDInsight 中使用 Hadoop Oozie | Windows Azure"
	description="在 HDInsight 中使用大数据服务 Hadoop Oozie。了解如何定义 Oozie 工作流，并提交 Oozie 作业。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="07/24/2015"
	wacn.date="08/29/2015"/>


# 将 Oozie 与 Hadoop 配合使用来在基于 Linux 的 HDInsight（预览版）上定义和运行工作流

[AZURE.INCLUDE [oozie-selector](../includes/hdinsight-oozie-selector.md)]

了解如何使用 Apache Oozie 来定义一个使用 Hive 和 Sqoop 的工作流，然后在基于 Linux 的 HDInsight 群集上运行该工作流。

Apache Oozie 是一个用于管理 Hadoop 作业的工作流/协调系统。它与 Hadoop 堆栈集成，并且它支持 Apache MapReduce、Apache Pig、Apache Hive 和 Apache Sqoop 的 Hadoop 作业。此外，它还可用于计划特定于系统的作业（例如 Java 程序或 shell 脚本）

> [AZURE.NOTE]使用 HDInsight 定义工作流的另一个选项是 Azure 数据工厂。若要了解有关 Azure 数据工厂的详细信息，请参阅[将 Pig 和 Hive 用于数据工厂][azure-data-factory-pig-hive]。

## 先决条件

在开始本教程之前，必须具备以下项：

- **Azure 订阅**

- **Azure CLI**：请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli)

- **HDInsight 群集**：请参阅 [Linux 上的 HDInsight 入门](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started)

- **Azure SQL 数据库**：将使用本文档中的步骤创建此项

## 示例工作流

将按照本文档中的说明实现的工作流包含两个操作。操作是任务的定义，例如运行 Hive、Sqoop、MapReduce 或其他进程：

![工作流关系图][img-workflow-diagram]

1. Hive 操作运行 HiveQL 脚本以从 HDInsight 附带的 **hivesampletable** 中提取记录。每一行数据描述从特定移动设备进行的访问。记录格式将显示如下：

		8       18:54:20        en-US   Android Samsung SCH-i500        California     United States    13.9204007      0       0
		23      19:19:44        en-US   Android HTC     Incredible      Pennsylvania   United States    NULL    0       0
		23      19:19:46        en-US   Android HTC     Incredible      Pennsylvania   United States    1.4757422       0       1

	本文档中使用的 Hive 脚本将计算每个平台（例如 Android 或 iPhone）的总访问次数并将这些计数存储到新的 Hive 表中。

	有关 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]。

2.  Sqoop 操作将新的 Hive 表的内容导出到 Azure SQL 数据库中的表。有关 Sqoop 的详细信息，请参阅[将 Hadoop Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]。

> [AZURE.NOTE]有关在 HDInsight 群集上支持的 Oozie 版本，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新增功能？][hdinsight-versions]。

## 创建工作目录

Oozie 要求作业所需的资源存储在同一目录中。此示例使用 ****wasb:///tutorials/useoozie**。使用以下命令创建此目录，以及将保存此工作流所创建的新 Hive 表的数据目录：

	hadoop fs -mkdir -p /tutorials/useoozie/data

> [AZURE.NOTE]`-p` 参数导致创建路径中的所有目录（如果尚未存在）。**data** 目录将用于保存 **useooziewf.hql** 脚本所使用的数据。

此外，请运行以下命令，该命令可确保 Oozie 在运行 Hive 和 Sqoop 作业时可以模拟你的用户帐户。将 **USERNAME** 替换为你的登录名：

	sudo adduser USERNAME users

如果你收到“用户已是用户成员”的错误，则可以直接忽略它。

## 添加数据库驱动程序

由于此工作流使用 Sqoop 将数据导出到 SQL 数据库，因此必须提供用于与 SQL 数据库通信的 JDBC 驱动程序的副本。使用以下命令将其复制到工作目录：

	hadoop fs -copyFromLocal /usr/share/java/sqljdbc_4.1/enu/sqljdbc4.jar /tutorials/useoozie/sqljdbc4.jar

如果你的工作流使用其他资源（如包含 MapReduce 应用程序的 jar），则也需要添加这些资源。

## 定义 Hive 查询

使用以下步骤创建定义查询的 HiveQL 脚本，稍后本文档中的 Oozie 工作流将使用该查询。

1. 使用 SSH 连接到基于 Linux 的 HDInsight 群集：

    * **Linux、Unix 或 OS X 客户端**：请参阅[在 Linux、OS X 或 Unix 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix)

    * **Windows 客户端**：请参阅[在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows)

2. 使用以下命令创建一个新文件：

        nano useooziewf.hql

1. 打开 nano 编辑器后，使用以下内容作为该文件的内容：

		DROP TABLE ${hiveTableName};
		CREATE EXTERNAL TABLE ${hiveTableName}(deviceplatform string, count string) ROW FORMAT DELIMITED
		FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
		INSERT OVERWRITE TABLE ${hiveTableName} SELECT deviceplatform, COUNT(*) as count FROM hivesampletable GROUP BY deviceplatform;

	在此脚本中使用了两个变量：

	- **${hiveTableName}**：将包含要创建的表的名称
	- **${hiveDataFolder}**：将包含用于存储表的数据文件的位置

	工作流定义文件（在本教程中为 workflow.xml）在运行时将这些值传递到此 HiveQL 脚本。

2. 按 Ctrl-X 以退出编辑器。出现提示时，选择 **Y** 以保存该文件，然后使用 **Enter** 以使用 **useooziewf.hql** 文件名。

3. 使用以下命令将 **useooziewf.hql** 复制到 ****wasb:///tutorials/useoozie/useooziewf.hql**：

		hadoop fs -copyFromLocal useooziewf.hql /tutorials/useoozie/useooziewf.hql

	这些命令将 **useooziewf.hql** 与文件存储到与此群集关联的 Azure 存储帐户中，以便即使删除了群集也保留该文件。这样可实现在群集未使用时删除群集，同时保留你的作业和工作流，从而节省了资金。

## 定义工作流

Oozie 工作流定义用 hPDL（一种 XML 流程定义语言）编写。使用以下步骤来定义工作流：

1. 使用以下语句来创建和编辑新文件：

        nano workflow.xml

1. 打开 nano 编辑器后，输入以下内容作为文件内容：

		<workflow-app name="useooziewf" xmlns="uri:oozie:workflow:0.2">
		  <start to = "RunHiveScript"/>
		  <action name="RunHiveScript">
		    <hive xmlns="uri:oozie:hive-action:0.2">
		      <job-tracker>${jobTracker}</job-tracker>
		      <name-node>${nameNode}</name-node>
		      <configuration>
		        <property>
		          <name>mapred.job.queue.name</name>
		          <value>${queueName}</value>
		        </property>
		      </configuration>
		      <script>${hiveScript}</script>
		      <param>hiveTableName=${hiveTableName}</param>
		      <param>hiveDataFolder=${hiveDataFolder}</param>
		    </hive>
		    <ok to="RunSqoopExport"/>
		    <error to="fail"/>
		  </action>
		  <action name="RunSqoopExport">
		    <sqoop xmlns="uri:oozie:sqoop-action:0.2">
		      <job-tracker>${jobTracker}</job-tracker>
		      <name-node>${nameNode}</name-node>
		      <configuration>
		        <property>
		          <name>mapred.compress.map.output</name>
		          <value>true</value>
		        </property>
		      </configuration>
		      <arg>export</arg>
		      <arg>--connect</arg>
		      <arg>${sqlDatabaseConnectionString}</arg>
		      <arg>--table</arg>
		      <arg>${sqlDatabaseTableName}</arg>
		      <arg>--export-dir</arg>
		      <arg>${hiveDataFolder}</arg>
		      <arg>-m</arg>
		      <arg>1</arg>
		      <arg>--input-fields-terminated-by</arg>
		      <arg>"\t"</arg>
		      <archive>sqljdbc4.jar</archive>
		      </sqoop>
		    <ok to="end"/>
		    <error to="fail"/>
		  </action>
		  <kill name="fail">
		    <message>Job failed, error message[${wf:errorMessage(wf:lastErrorNode())}] </message>
		  </kill>
		  <end name="end"/>
		</workflow-app>

	在此工作流中定义了两个操作：

	- **RunHiveScript**：这是启动操作中，并运行 **useooziewf.hql** Hive 脚本

	- **RunSqoopExport**：这使用 Sqoop 将从 Hive 脚本创建的数据导出到 SQL 数据库。仅当 **RunHiveScript** 操作成功时，才运行此操作。

		> [AZURE.NOTE]有关 Oozie 工作流和使用工作流操作的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（适用于 HDInsight 3.0 版）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（适用于 HDInsight 2.1 版）。

	请注意，此工作流有几个条目（如 `${jobTracker}`）将替换为本文档后面的作业定义中使用的值。

	另请注意 Sqoop 部分中的 `<archive>sqljdbc4.jar</arcive>` 条目。在此操作运行时，该条目会指示 Oozie 将该存档文件可供 Sqoop 使用。

2. 使用 Ctrl-X，然后单击 **Y** 和 **Enter** 以保存该文件。

3. 使用以下命令将 **workflow.xml** 文件复制到 ****wasb:///tutorials/useoozie/workflow.xml**：

		hadoop fs -copyFromLocal workflow.xml wasb:///tutorials/useoozie/workflow.xml

## 创建数据库

以下步骤将创建数据将导出到的 Azure SQL 数据库。

> [AZURE.IMPORTANT]在执行这些步骤之前，必须[安装并配置 Azure CLI](/documentation/articles/xplat-cli)。安装 CLI 和执行创建数据库的步骤可以从 HDInsight 群集或本地工作站进行。

1. 使用以下命令创建新的 Azure SQL 数据库服务器：

        azure sql server create <adminLogin> <adminPassword> <region>

    例如，`azure sql server create admin password "West US"`。

    该命令完成后，你将收到如下响应：

        info:    Executing command sql server create
        + Creating SQL Server
        data:    Server Name i1qwc540ts
        info:    sql server create command OK

    > [AZURE.IMPORTANT]记下此命令返回的服务器名称（在上述示例中为 **i1qwc540ts**。） 这是已创建的 SQL 数据库服务器的短名称。完全限定域名 (FQDN) 为 **&lt;shortname&gt;.database.chinacloudapi.cn**。对于上述示例，FQDN 为 **i1qwc540ts.database.chinacloudapi.cn**。

2. 使用以下命令在 SQL 数据库服务器上创建一个名为 **oozietest** 的数据库：

        azure sql db create [options] <serverName> oozietest <adminLogin> <adminPassword>

    此命令在其完成后将返回“OK”消息。

	> [AZURE.NOTE]如果你收到指示无权访问的错误，则可能需要使用以下命令将系统的 IP 地址添加到 SQL 数据库防火墙：
    >
    > `sql firewallrule create [options] <serverName> <ruleName> <startIPAddress> <endIPAddress>`

### 创建表

> [AZURE.NOTE]有多种方法可连接到 SQL 数据库以创建表。以下步骤从 HDInsight 群集使用 [FreeTDS](http://www.freetds.org/)。

3. 使用以下命令在 HDInsight 群集上安装 FreeTDS：

        sudo apt-get --assume-yes install freetds-dev freetds-bin

4. 安装 FreeTDS 后，使用以下命令连接到你先前创建的 SQL 数据库服务器：

        TDSVER=8.0 tsql -H <serverName>.database.chinacloudapi.cn -U <adminLogin> -P <adminPassword> -p 1433 -D oozietest

    你将收到如下输出：

        locale is "en_US.UTF-8"
        locale charset is "UTF-8"
        using default charset "UTF-8"
        Default database being set to oozietest
        1>

5. 在 `1>` 提示符下，输入以下行：

		CREATE TABLE [dbo].[mobiledata](
		[deviceplatform] [nvarchar](50),
		[count] [bigint])
		GO
		CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(deviceplatform)
		GO

    输入 `GO` 语句后，将评估前面的语句。这将创建一个名为**“mobiledata”**的新表，Sqoop 将写入该表。

    使用以下命令验证是否已创建该表：

        SELECT * FROM information_schema.tables
        GO

    你应看到如下输出：

        TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
        oozietest       dbo     mobiledata      BASE TABLE

8. 在 `1>` 提示符下输入 `exit` 以退出 tsql 实用工具。

## 创建作业定义

作业定义描述在何处可以找到 workflow.xml，以及工作流使用的其他文件（例如 useooziewf.hql）。 它还定义工作流及相关文件中使用的属性的值。

1. 使用以下命令获取默认存储的完整 WASB 地址。稍后将在配置文件中使用该项：

		sed -n '/<name>fs.default/,/</value>/p' /etc/hadoop/conf/core-site.xml

	这应返回如下信息：

		<name>fs.defaultFS</name>
		<value>wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn</value>

	保存 **wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn** 值，因为此值将在后续步骤中使用。

2. 使用以下命令获取群集头节点的 FQDN。这将用于群集的作业跟踪器地址。稍后将在配置文件中使用该项：

		hostname -f

	这将返回如下信息：

		headnode0.CLUSTERNAME-ssh.j7.internal.cloudapp.net

	用于作业跟踪器的端口是 8050，因此用于作业跟踪器的完整地址将为 **headnode0.CLUSTERNAME-ssh.j7.internal.cloudapp.net:8050**。

1. 使用以下命令创建 Oozie 作业定义配置：

		nano job.xml

2. 打开 nano 编辑器后，使用以下内容作为该文件的内容：

		<?xml version="1.0" encoding="UTF-8"?>
		<configuration>

		  <property>
		    <name>nameNode</name>
		    <value>wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn</value>
		  </property>

		  <property>
		    <name>jobTracker</name>
		    <value>JOBTRACKERADDRESS</value>
		  </property>

		  <property>
		    <name>queueName</name>
		    <value>default</value>
		  </property>

		  <property>
		    <name>oozie.use.system.libpath</name>
		    <value>true</value>
		  </property>

		  <property>
		    <name>hiveScript</name>
		    <value>wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/useoozie/useooziewf.hql</value>
		  </property>

		  <property>
		    <name>hiveTableName</name>
		    <value>mobilecount</value>
		  </property>

		  <property>
		    <name>hiveDataFolder</name>
		    <value>wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/useoozie/data</value>
		  </property>

		  <property>
		    <name>sqlDatabaseConnectionString</name>
		    <value>"jdbc:sqlserver://serverName.database.chinacloudapi.cn;user=adminLogin;password=adminPassword;database=oozietest"</value>
		  </property>

		  <property>
		    <name>sqlDatabaseTableName</name>
		    <value>mobiledata</value>
		  </property>

		  <property>
		    <name>user.name</name>
		    <value>YourName</value>
		  </property>

		  <property>
		    <name>oozie.wf.application.path</name>
		    <value>wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/useoozie</value>
		  </property>
		</configuration>

	* 将 **wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn** 的所有实例替换为之前收到的值。

	> [AZURE.WARNING]必须使用包含容器和存储帐户的完整 WASB 路径。使用短格式 (wasb:///) 将导致启动作业时 RunHiveScript 操作失败。

	* 将 **JOBTRACKERADDRESS** 替换为之前收到的作业跟踪器/ResourceManager 地址。

	* 将 **YourName** 替换 HDInsight 群集的登录名。

	* 将 **serverName**、**adminLogin** 和 **adminPassword** 替换为 Azure SQL 数据库的信息。

	此文件中的大部分信息用于填充在 workflow.xml 或 ooziewf.hql 文件中使用的值（如 ${nameNode}）。

	> [AZURE.NOTE]**oozie.wf.application.path** 条目定义了在哪里可以找到 workflow.xml 文件（其中包含此作业运行的工作流）。

2. 使用 Ctrl-X，然后单击 **Y** 和 **Enter** 以保存该文件。

## 提交和管理作业

以下步骤使用 Oozie 命令在群集上提交和管理 Oozie 工作流。Oozie 命令是在 [Oozie REST API](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html) 上实现的友好接口。

> [AZURE.IMPORTANT]使用 Oozie 命令时，必须使用 HDInsight 头节点的 FQDN。此 FQDN 只能从群集访问，或者如果群集位于 Azure 虚拟网络中，则可从同一网络中的其他计算机访问。

1. 使用以下命令获取 Oozie 服务的 URL：

		sed -n '/<name>oozie.base.url/,/</value>/p' /etc/oozie/conf/oozie-site.xml

	这将返回如下值：

		<name>oozie.base.url</name>
		<value>http://headnode0.CLUSTERNAME-ssh.j7.internal.cloudapp.net:11000/oozie</value>

	**http://headnode0.CLUSTERNAME-ssh.j7.internal.cloudapp.net:11000/oozie** 部分是要用于 Oozie 命令的 URL。

2. 使用以下项创建 URL 的环境变量，这样你便不必为每个命令键入它：

		export OOZIE_URL=http://headnode0.CLUSTERNAME-ssh.j7.internal.cloudapp.net:11000/oozie

	将 URL 替换为之前收到的 URL。

3. 使用以下项来提交作业：

		oozie job -config job.xml -submit

	这将从 **job.xml** 加载作业信息并将其提交给 Oozie，但不会运行它。

	命令完成后，它应返回作业的 ID。例如，`0000005-150622124850154-oozie-oozi-W`。此 ID 将用于管理作业。

4. 使用以下命令查看作业的状态。输入前一命令返回的作业 ID：

		oozie job -info <JOBID>

	这将返回如下信息。

		Job ID : 0000005-150622124850154-oozie-oozi-W
		------------------------------------------------------------------------------------------------------------------------------------
		Workflow Name : useooziewf
		App Path      : wasb:///tutorials/useoozie
		Status        : PREP
		Run           : 0
		User          : USERNAME
		Group         : -
		Created       : 2015-06-22 15:06 GMT
		Started       : -
		Last Modified : 2015-06-22 15:06 GMT
		Ended         : -
		CoordAction ID: -
		------------------------------------------------------------------------------------------------------------------------------------

	此作业的状态为 `PREP`，它指示此作业已提交但尚未启动。

4. 使用以下命令启动作业：

		oozie job -start JOBID

	如果在此命令后检查状态，作业将处于运行状态，并且将返回作业中的操作的信息。

5. 任务成功完成后，可以使用以下命令验证数据是否已生成并已导出到 SQL 数据库表：

		TDSVER=8.0 tsql -H <serverName>.database.chinacloudapi.cn -U <adminLogin> -P <adminPassword> -p 1433 -D oozietest

	在 `1>` 提示符下，输入以下命令：

		SELECT * FROM mobiledata
		GO

	你应收到如下信息：

		deviceplatform  count
		Android 31591
		iPhone OS       22731
		proprietary development 3
		RIM OS  3464
		Unknown 213
		Windows Phone   1791
		(6 rows affected)

有关 Oozie 命令的详细信息，请参阅 [Oozie 命令行工具](https://oozie.apache.org/docs/4.1.0/DG_CommandLineTool.html)。

## Oozie REST API

使用 Oozie REST API 可以构建你自己的可用于 Oozie 的工具。以下是有关使用 Oozie REST API 的 HDInsight 特定信息：

* **URI**：可以从 `https://CLUSTERNAME.azurehdinsight.cn/oozie` 处的群集外部访问 REST API

* **身份验证**：必须使用群集 HTTP 帐户 (admin) 和密码对 API 进行身份验证。例如：

		curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.cn/oozie/versions

有关使用 Oozie REST API 的详细信息，请参阅 [Oozie Web 服务 API](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html)。

## Oozie Web UI

Oozie Web UI 提供基于 Web 的视图显示群集上的 Oozie 作业的状态。它允许你查看作业状态、作业定义、配置、作业中操作的图形以及作业日志。你还可以查看作业中的操作的详细信息。

若要访问 Oozie Web UI，请使用以下步骤：

1. 创建到 HDInsight 群集的 SSH 隧道。有关如何执行此操作的信息，请参阅以下项之一：

	* [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix#tunnel)

	* [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows#tunnel)

2. 创建隧道后，请在 Web 浏览器中打开 Ambari Web UI。Ambari 站点的 URI 是 **https://CLUSTERNAME.azurehdinsight.cn**。将 **CLUSTERNAME** 替换为基于 Linux 的 HDInsight 群集的名称。

3. 在页面左侧，依次选择 **Oozie**、**“快速链接”**和**“Oozie Web UI”**。

	![菜单的图像](./media/hdinsight-use-oozie-linux-mac/ooziewebuisteps.png)

4. Oozie Web UI 默认为显示正在运行的工作流作业。若要查看所有工作流作业，请选择**“所有作业”**。

	![显示的所有作业](./media/hdinsight-use-oozie-linux-mac/ooziejobs.png)

5. 选择一个作业可查看有关该作业的详细信息。

	![作业信息](./media/hdinsight-use-oozie-linux-mac/jobinfo.png)

6. 从“作业信息”选项卡中，你可以看到基本作业信息以及作业中的各项操作。使用顶部的选项卡可以查看作业定义、作业配置，访问作业日志或查看作业的有向无环图 (DAG)。

	* **作业日志**：选择**“获取日志”**按钮可获取作业的所有日志，或者使用**“输入搜索筛选器”**字段以筛选日志

		![作业日志](./media/hdinsight-use-oozie-linux-mac/joblog.png)

	* **JobDAG**：DAG 是通过工作流获取的数据路径的图形概述

		![作业 DAG](./media/hdinsight-use-oozie-linux-mac/jobdag.png)

7. 从**“作业信息”**选项卡中选择一项操作将显示该操作的信息。例如，选择 **RunHiveScript** 操作。

	![操作信息](./media/hdinsight-use-oozie-linux-mac/action.png)

8. 你可以看到该操作的详细信息，包括指向**控制台 URL** 的链接，可使用该链接查看作业的作业跟踪器信息。

## 计划作业

协调器允许你为作业指定开始时间、结束时间和执行频率，因此可以安排作业在特定时间执行。

若要定义工作流的计划，请使用以下步骤：

1. 使用以下命令创建一个名为 **coordinator.xml**的新文件：

		nano coordinator.xml

	将以下内容用作该文件的内容：

		<coordinator-app name="my_coord_app" frequency="${coordFrequency}" start="${coordStart}" end="${coordEnd}" timezone="${coordTimezone}" xmlns="uri:oozie:coordinator:0.4">
	      <action>
	        <workflow>
	          <app-path>${workflowPath}</app-path>
	        </workflow>
	      </action>
		</coordinator-app>

	请注意，这将使用 `${...}` 变量，这些变量将替换为作业定义中的值。这些变量包括：

	* **${coordFrequency}**：作业正在运行的实例之间的时间
	* **${coordStart}**：作业开始时间
	* **${coordEnd}**：作业结束时间
	* **${coordTimezone}**：协调器作业位于没有夏时制的固定时区（通常用 UTC 表示）。此时区被称为“Oozie 处理时区”
	* **${wfPath}**：workflow.xml 的路径。

2. 使用 Ctrl-X，然后单击 **Y** 和 **Enter** 以保存该文件。

3. 使用以下命令将该文件复制到此作业的工作目录：

		hadoop fs -copyFromLocal coordinator.xml /tutorials/useoozie/coordinator.xml

4. 使用以下项修改 **job.xml** 文件：

		nano job.xml

	进行以下更改：

	* 将 `<name>oozie.wf.application.path</name>` 更改为 `<name>oozie.coord.application.path</name>`。这将指示 Oozie 运行协调器文件而不是工作流文件

	* 添加以下内容，这会将 coordinator.xml 中使用的某个变量设置为指向 workflow.xml 的位置：

		    <property>
		      <name>workflowPath</name>
		      <value>wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/useoozie</value>
		    </property>

		将 **mycontainer** 和 **mystorageaccount** 的值替换为 job.xml 文件中的其他条目使用的值。

	* 添加以下内容，这定义要用于 coordinator.xml 文件的开始时间、结束时间和频率：

		    <property>
		      <name>coordStart</name>
		      <value>2015-06-25T12:00Z</value>
		    </property>

		    <property>
		      <name>coordEnd</name>
		      <value>2015-06-27T12:00Z</value>
		    </property>

		    <property>
		      <name>coordFrequency</name>
		      <value>1440</value>
		    </property>

		    <property>
		      <name>coordTimezone</name>
		      <value>UTC</value>
		    </property>

		这些项将开始时间设为 2015 年 6 月 25 日中午 12:00 点，将结束时间设为 2015 年 6 月 27 日，将运行此作业的时间间隔设为每天（频率以分钟为单位，因此，24 小时 x 60 分钟 = 1440 分钟）。 最后，将时区设为 UTC。

5. 使用 Ctrl-X，然后单击 **Y** 和 **Enter** 以保存该文件。

6. 若要运行该作业，请使用以下命令：

		oozie job -config job.xml -run

	这将提交并启动作业。

7. 如果你访问 Oozie Web UI 并选择**“协调器作业”**选项卡，则应看到如下信息：

	![协调器作业选项卡](./media/hdinsight-use-oozie-linux-mac/coordinatorjob.png)

	请注意**下一步实现**条目；这是该作业将运行下一步的时间。

8. 与先前的工作流作业类似，在 Web UI 中选择作业条目将显示有关该作业的信息：

	![协调器作业信息](./media/hdinsight-use-oozie-linux-mac/coordinatorjobinfo.png)

	请注意，这仅显示作业是否成功运行，而不显示计划的工作流中的单个操作。若要查看该信息，请选择其中一个**“操作”**条目。这将显示的内容类似于针对先前工作流作业检索到的信息。

	![操作信息](./media/hdinsight-use-oozie-linux-mac/coordinatoractionjob.png)

## 故障排除

排查 Oozie 作业的问题时，Oozie UI 将非常有用，因为它允许你同时轻松查看这两个 Oozie 日志，并链接到 MapReduce 任务（例如 Hive 查询）的作业跟踪器日志。通常情况下，故障排除的模式应为：

1. 在 Oozie Web UI 中查看作业。

2. 如果特定操作出现错误或故障，请选择该操作以查看**“错误消息”**字段是否提供了有关故障的详细信息。

3. 如果可用，请使用该操作的 URL 来查看该操作的更多详细信息（如作业跟踪器日志）。

以下是你可能会遇到的特定错误以及解决这些错误的方法。

### JA009: 无法初始化群集

**症状**：作业状态将更改为**“已暂停”**。作业详细信息将 RunHiveScript 状态显示为 **START_MANUAL**。选择该操作将显示以下错误消息：

	JA009: Cannot initialize Cluster. Please check your configuration for map

**原因**：**job.xml** 文件中使用的 WASB 地址不包含存储容器或存储帐户名称。WASB 地址格式必须为 `wasb://containername@storageaccountname.blob.core.chinacloudapi.cn`。

**解决方案**：更改该作业所用的 WASB 地址。

### JA002: 不允许 Oozie 模拟 &lt;用户>

**症状**：作业状态将更改为**“已暂停”**。作业详细信息将 RunHiveScript 状态显示为 **START_MANUAL**。选择该操作将显示以下错误消息：

	JA002: User: oozie is not allowed to impersonate <USER>

**原因**：当前权限设置不允许 Oozie 模拟指定的用户帐户。

**解决方案**：Oozie 可以模拟**用户**组中的用户。使用 `groups USERNAME` 查看该用户帐户所属的组。如果该用户不是**用户**组的成员，则使用以下命令将该用户添加到此组：

	sudo adduser USERNAME users

> [AZURE.NOTE]可能需要几分钟时间 HDInsight 才会识别该用户已添加到此组。

### 启动器错误 (Sqoop)

**症状**：作业状态将更改为**“已终止”**。作业详细信息将 RunSqoopExport 状态显示为 **ERROR**。选择该操作将显示以下错误消息：

	Launcher ERROR, reason: Main class [org.apache.oozie.action.hadoop.SqoopMain], exit code [1]

**原因**：Sqoop 无法加载访问数据库所需的数据库驱动程序。

**解决方案**：从 Oozie 作业使用 Sqoop 时，必须在作业所用的其他资源（例如 workflow.xml）中包含数据库驱动程序。

此外，还必须在 workflow.xml 的 `<sqoop>...</sqoop>` 节中引用包含数据库驱动程序的存档文件。

例如，对于本文档中的作业，可使用以下步骤：

1. 将 sqljdbc4.jar 文件复制到 /tutorials/useoozie 目录：

		 hadoop fs -copyFromLocal /usr/share/java/sqljdbc_4.1/enu/sqljdbc4.jar /tutorials/useoozie/sqljdbc4.jar

2. 修改 workflow.xml 以在 `</sqoop>` 上面的新行上添加以下内容：

		<archive>sqljdbc4.jar</archive>

## 后续步骤
在本教程中，你已经学习了如何定义 Oozie 工作流，以及如何运行 Oozie 作业。若要了解有关使用 HDInsight 的详细信息，请参阅以下文章：

- [将基于时间的 Oozie 协调器与 HDInsight 配合使用][hdinsight-oozie-coordinator-time]
- [在 HDInsight 中上载 Hadoop 作业的数据][hdinsight-upload-data]
- [将 Sqoop 与 HDInsight 中的 Hadoop 配合使用][hdinsight-use-sqoop]
- [将 Hive 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-pig]
- [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]


[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563



[azure-data-factory-pig-hive]: /documentation/articles/data-factory-pig-hive-activities
[hdinsight-oozie-coordinator-time]: /documentation/articles/hdinsight-use-oozie-coordinator-time
[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started


[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop-mac-linux
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage
[hdinsight-get-started-emulator]: /documentation/articles/hdinsight-get-started-emulator

[hdinsight-develop-streaming-jobs]: /documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs
[hdinsight-develop-mapreduce]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce

[sqldatabase-create-configue]: /documentation/articles/sql-database-create-configure
[sqldatabase-get-started]: /documentation/articles/sql-database-get-started

[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: http://www.windowsazure.cn/downloads/
[powershell-about-profiles]: http://go.microsoft.com/fwlink/?LinkID=113729
[powershell-install-configure]: /documentation/articles/powershell-install-configure
[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-script]: https://technet.microsoft.com/zh-cn/library/ee176961.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.Preparation.Output1.png
[img-runworkflow-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.RunWF.Output.png

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx

<!---HONumber=67-->