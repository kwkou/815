<properties 
	pageTitle="发行说明：Microsoft HDInsight Emulator for Azure | Azure" 
	description="获取有关最新版 HDInsight Hadoop Emulator 的最新信息。" 
	editor="cgronlun" 
	manager="paulettm" 
	services="hdinsight" 
	authors="mumian" 
	documentationCenter=""/>

<tags 
	ms.service="hdinsight" 
	ms.date="06/30/2015" 
	wacn.date="10/22/2015"/>


# 发行说明：Microsoft HDInsight Emulator for Hadoop 



> [AZURE.NOTE]查看版本号的最简单方法是在“添加/删除程序”中查看“Microsoft HDInsight Emulator for Azure”所对应的条目（适用于版本 1.0.0.0 或更高版本）或“Microsoft HDInsight 开发者预览版”所对应的条目（适用于 1.0.0.0 以下版本）。

## 于 2014 年 8 月 29 日发布的版本 2.0.0.0

* 此版本将 HDInsight Emulator 更新为以 3.1 版的服务中目前存在的同一组 Hadoop 项目为目标。    

* 与此产品的预览版一样，此发行版继续针对开发人员方案，因此仅支持单节点部署。

### 新增功能 
 
* [已更新 Hadoop 组件版本](/documentation/articles/hdinsight-component-versioning/)，使其对应于服务版本 3.1。这包括 Hive 0.13 和 Tez 支持。

## 于 2013 年 10 月 28 日发布的版本 1.0.0.0

* 这是 Microsoft HDInsight Emulator for Azure（以前称作 Microsoft HDInsight 开发者预览版）的公开发布 (GA) 版。 

* 与此产品的预览版一样，此发行版继续针对开发人员方案，因此仅支持单节点部署。

### 新增功能 
 
* 已添加脚本以简化将所有 Apache Hadoop 服务都设为自动启动或手动启动的操作。默认设置与以前一样仍为“自动”，但现在可以通过安装在 C:\\Hadoop 中的 set-onebox-manualstart.cmd 或 set-onebox-autostart.cmd 脚本更改所有服务。 

* 必需的安装依赖项的数量已显著减少，因此可以更快地进行安装。

* 此版本在命令包含一个 Bug 修复，用于对安装在 GettingStarted 文件夹中的 RunSamples.ps1 脚本中运行 Pig 示例。

* 此版本包含 Hortonworks 数据平台 (HDP) 版本 1.1 的更新，以匹配随 Azure HDInsight 群集版本 1.6 提供的 Hortonworks 数据平台服务。

## 于 2013 年 9 月 30 日发布的版本 0.11.0.0

### 新增功能 
		
* 已删除 HDInsight 仪表板。 

* 此版本包含 Hortonworks 数据平台开发者预览版的更新，与 Azure HDInsight 上的 Hortonworks 数据平台预览版匹配。

## 于 2013 年 8 月 9 日发布的版本 0.10.0.0

### 新增功能 
	
* 此版本包含 Hortonworks 数据平台开发者预览版的更新，与 Azure HDInsight 上的 Hortonworks 数据平台预览版匹配。

## 于 2013 年 6 月 7 日发布的版本 0.8.0.0

### 新增功能 

* 此版本包含 Hortonworks 数据平台开发者预览版的更新，与 Azure HDInsight 上的 Hortonworks 数据平台预览版匹配。

## 于 2013 年 5 月 13 日发布的版本 0.6.0.0

### 新增功能 

* 现在正在安装 Hive Server 2。这是与此更新同时发布的 0.9.2.0 版 Microsoft ODBC Driver for Hive 所必需的。 

* 所有服务均已设为自动启动，这样，在计算机重新启动后，就不需要重新启动所有内容。

## 于 2013 年 3 月 25 日发布的版本 0.4.0.0

### 新增功能 

* 新发行的 Microsoft HDInsight 开发者预览版，以及用于 Windows 的 Hortonworks 数据平台开发者预览版。 

* 包括 Apache Hadoop、Hive、Pig、Sqoop、Oozie、HCatalog 和 Templeton。

* 具有以下功能的全新 HDInsight 仪表板：
 
	* 可连接到多个群集，包括本地安装，以及通过 Azure HDInsight 服务远程运行的群集。有关 HDInsight 服务的详细信息，请参阅 [http://www.windowsazure.cn/documentation/services/hdinsight/](/documentation/services/hdinsight/)。
 
	* 在本地群集上配置 Azure Blob 存储。在下面查看详细说明。

* 在新版交互式 Hive 控制台中编写和编辑 Hive 查询。

* 查看并下载作业历史记录和结果。

### 发行说明 

* 本地 HDInsight 安装上的 REST API 终结点和 Azure HDInsight 服务通过相同服务的不同端口号进行访问： 

	本地：Oozie - http://localhost:11000/oozie/v1/admin/status Templeton - http://localhost:50111/templeton/v1/status ODBC - 在 DSN 配置或连接字符串中使用端口 10000

	HDInsight 服务：Oozie - http://ServerFQDN:563/oozie/v1/admin/status Templeton - http://ServerFQDN:563/templeton/v1/status ODBC - 在 DSN 配置或连接字符串中使用端口 563


* 在本地群集上配置 Azure Blob 存储：

	在仪表板中，你将看到名为“本地(hdfs)”的默认本地群集。如果想让 Azure Blob 存储作为本地安装的存储，请执行以下操作：

	1. 在 C:\\Hadoop\\hadoop-1.1.0-SNAPSHOT\\conf 中找到的 core-site.xml 中添加帐户标记：       

			<property>
        		<name>fs.azure.account.key.{AccountName}</name>
        		<value>{Key}</value>
     		</property>

			<property>
        		<name>fs.default.name</name>
       			<!-- cluster variant -->
       	 		<value>asv://ASVContainerName@ASVAccountName</value>
        		<description>The name of the default file system.  Either the literal string "local" or a host:port for NDFS.</description>
        		<final>true</final>
      		</property>

     		<property>
        		<name>dfs.namenode.rpc-address</name>
        		<value>hdfs://localhost:8020</value>
        		<description>A base for other temporary directories.</description>
      		</property>
      
		示例：

			<property>
    			<name>fs.azure.account.key.MyHadoopOnAzureAccountName</name>
				<value>8T45df897/a5fSox1yMjYkX66CriiGgA5zptSXpPdG4o1Qw2mnbbWQN1E9i/i7kihk6FWNhvLlqe02NXEVw6rP==</value>
    		</property>

			<property>
				<name>fs.default.name</name>
				<!-- cluster variant -->
				<value>asv://MyASVContainer@MyASVAccount</value>
				<description>The name of the default file system.  Either the literal string "local" or a host:port for NDFS.</description>
				<final>true</final>
			</property>
        

	2. 在你的桌面上以提升的模式打开 Hadoop 命令外壳，并运行以下命令：
	 
	 		%HADOOP_NODE%\stop-onebox.cmd && %HADOOP_NODE%\start-onebox.cmd

	3. 使用完整 URI 访问该帐户上的任何文件：asv://{container}@{account}/{path}（如果要使用 HTTPS 访问数据，则使用 asvs://）。示例：
	 
	 		hadoop fs -lsr 
			asvs://MyHadoopOnAzureContainerName@MyHadoopOnAzureAccountName/example/data/

	4. 删除当前已注册的本地群集，并使用新的 Azure Blob 存储凭据重新注册它。


## 于 2012 年 12 月 13 日发布的版本 0.3.0.0 

* 已将仪表板网站更改为进行匿名身份验证，而不是使用 Windows 凭据进行身份验证。这消除了前一版本的发行说明中提到的登录提示问题。 

* 修复了导出和某些类型的导入存在的一些 Sqoop Bug。

### 发行版问题 

* JavaScript 控制台无法加载。请参阅版本 0.2.0.0 的发行说明以了解详细信息。 
* Sqoop 命令行将显示警告，如下所示。这些将在以后的更新中修复，可以安全地忽略。 
	
		c:\Hadoop\sqoop-1.4.2\bin>sqoop version 
		Setting HBASE_HOME to 
		Warning: HBASE_HOME [c:\hadoop\hadoop-1.1.0-SNAPSHOT\hbase-0.94.2] does not exist HBase imports will fail. 
		Please set HBASE_HOME to the root of your HBase installation. 
		Setting ZOOKEEPER_HOME to 
		Warning: ZOOKEEPER_HOME [c:\hadoop\hadoop-1.1.0-SNAPSHOT\zookeeper-3.4.3] does not exist 
		Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation. 
		Sqoop 1.4.2 
		git commit id 3befda0a456124684768348bd652b0542b002895 
		Compiled by  on Thu 11/29/2012- 3:26:26.10

## 于 2012 年 12 月 3 日发布的版本 0.2.0.0

* 将语义版本控制引入 Windows 安装程序 

* 对 MSDN 论坛上报告的各种安装 Bug（特别是围绕 HDInsight 仪表板的安装）进行了修复

* 添加了“开始”菜单项以提高可发现性

* 对 Hive 控制台多行输入进行了修复

* 对入门内容进行了少量更新

### 发行版问题 

* JavaScript 控制台无法加载。 

	* 在进行某些安装时，JavaScript 控制台会失败，并在页面上显示 HTTP 404 错误。若要解决此问题，请直接导航到 http://localhost:8080 来使用控制台。 

* 浏览到 HDInsight 仪表板引发登录提示。

	* 我们获得一些报告说，浏览到 HDInsight 仪表板期间会出现登录对话框。在这种情况下，你可以提供当前用户的登录信息，你应该能够浏览到仪表板。 


## 于 2012 年 10 月 23 日发布的版本 1.0.0.0

* 初始版本 

### 发行版问题 

* Hive 控制台 

	* 如果提交的 Hive 命令中包含换行符，你会收到语法错误。请删除换行符，查询应该按预期执行。 



## 一般已知问题

* Hadoop 用户密码过期 

	Hadoop 用户的密码可能会过期，具体取决于推送到计算机的 Active Directory 策略。以下 Windows PowerShell 脚本将密码设置为不过期，并可以从管理命令提示符运行。

		$username = "hadoop"
		$ADS_UF_DONT_EXPIRE_PASSWD = 0x10000 # (65536, from ADS_USER_FLAG_ENUM enumeration)

		$computer = [ADSI]("WinNT://$ENV:COMPUTERNAME,computer")
		$users = $computer.psbase.children | where { $_.psbase.schemaclassname -eq "User" }

		foreach($user in $users)
		{
			if($user.Name -eq $username)
			{
				$user.UserFlags = $ADS_UF_DONT_EXPIRE_PASSWD
        		$user.SetInfo()
 
        		$user.PasswordExpired = 0
        		$user.SetInfo()
 
        		Write-Host "$username user maintenance completed. "
    		}
		}


* 临时目录
	
	hadoop.tmp.dir 文件指向错误的位置。不是指向 C:\\hadoop\\hdfs，而是指向 c:\\hdfs。此 Bug 将在下一个 HDP 位的更新中修复。

* OS 限制

	HDInsight 服务器必须安装在 64 位 OS 上。

* Web 平台安装程序 (WebPI) 搜索结果

	无法在 WebPI 搜索结果中找到 HDInsight。这通常是由于 OS 限制。HDInsight 需要最低版本为 Windows 7 Service Pack 1、Windows Server 2008 R2 Service Pack1、Windows 8 或 Windows Server 2012 的 64 位操作系统。

* 管理命令提示符文档

	* 若要运行 **hadoop mradmin** 或 **hadoop defadmin** 等命令，必须以 Hadoop 用户身份运行。 

	* 若要轻松创建以该用户身份运行的 shell，请打开 Hadoop 命令提示符并运行以下命令：
	 
	 		start-hadoopadminshell

	* 这将打开一个新的以 Hadoop 管理员特权运行的命令外壳。

##<a name="nextsteps"></a>后续步骤

- [HDInsight Emulator 入门][hdinsight-hadoop-emulator-get-started]


[hdinsight-hadoop-emulator-get-started]: /documentation/articles/hdinsight-get-started-emulator/

<!---HONumber=74-->