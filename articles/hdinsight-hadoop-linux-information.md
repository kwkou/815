<properties
   pageTitle="基于 Linux 的 HDInsight 上的 Hadoop 须知信息 | Azure"
   description="基于 Linux 的 HDInsight 群集在你熟悉的 Linux 环境中提供可在 Azure 云中运行的 Hadoop。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="04/01/2015"
    wacn.date="04/15/2015"
    />


# 使用 HDInsight on Linux（预览版）

基于 Linux 的 HDInsight 群集在你熟悉的 Linux 环境中提供可在 Azure 云中运行的 Hadoop。其操作大多与 Linux 安装上的任何其他 Hadoop 相同。本文档指出了你应该注意的具体差异。

## 域名

连接到群集时要使用的完全限定域名 (FQDN) 是 **&lt;群集名称>.azurehdinsight.cn** 

## 可远程访问的服务

* **Ambari (web)** - https://&lt;群集名称>.azurehdinsight.cn

	> [AZURE.NOTE] 使用群集系统管理员用户和密码进行身份验证，然后登录 Ambari。这还要使用群集管理员用户和密码身份验证采用明文形式 - 请始终使用 HTTPS 来确保连接的安全性。

	虽然可以直接通过 Internet 访问群集的 Ambari，但若要使用某些功能，则需要根据访问群集所用的内部域名的节点来达到目的。由于这是内部域名并未公开，因此在尝试通过 Internet 访问某些功能时，你会收到找不到服务器的错误。

	若要解决此问题，请使用 SSH 隧道通过代理将 Web 流量传送到群集头节点。使用以下文章从本地计算机上的端口创建通往群集的 SSH 隧道。

	* <a href="/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/#tunnel" target="_blank">从 Linux、Unix 或 OS X 在 HDInsight 上配合使用 SSH 与基于 Linux 的 Hadoop</a> - 使用 `ssh` 命令来创建 SSH 隧道的相关步骤

	* <a href="/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/#tunnel" target="_blank">从 Windows 在 HDInsight 上配合使用 SSH 与基于 Linux 的 Hadoop</a> - 使用 Putty 来创建 SSH 隧道的相关步骤

* **Ambari (REST)** - https://&lt;群集名称>.azurehdinsight.cn/ambari

	> [WACOM.NOTE] 使用群集管理员用户和密码进行身份验证。
	> 
	> 身份验证采用明文形式 - 请始终使用 HTTPS 来确保连接的安全性。

* **WebHCat (Templeton)** - https://&lt;群集名称>.azurehdinsight.cn/templeton

	> [WACOM.NOTE] 使用群集管理员用户和密码进行身份验证。
	> 
	> 身份验证采用明文形式 - 请始终使用 HTTPS 来确保连接的安全性。

* **SSH** - &lt;群集名称> - ssh.azurehdinsight.cn，使用 22

	> [WACOM.NOTE] 你只能从客户端计算机通过 SSH 访问群集头节点。在连接后，再从头节点使用 SSH 访问辅助线程节点。

## 文件位置

Hadoop 相关文件可在群集节点上的 `/usr/hdp/current` 中找到。

示例数据和 jar 可在 HDFS (WASB) 上的"/example"或 "wasb:///example"中找到。

## HDFS、WASB 和存储最佳实践

在大多数 Hadoop 分发版中，Hadoop 分布式文件系统 (HDFS) 由群集中计算机上的本地存储提供支持。尽管这种方式很有效率，但用于基于云的解决方案时可能费用高昂，因为计算资源以小时为单位来计费。

HDInsight 使用 Azure Blob 存储作为默认存储，这提供了以下好处：

* 长期存储成本低廉

* 可从各种外部服务进行访问，例如网站、文件上载/下载实用工具、各种语言的 SDK 和 Web 浏览器

由于它是 HDInsight 的默认存储，因此你通常不需要进行任何设置就能使用。例如，以下命令会列出 **/example/data** 文件夹中的文件，这些文件存储在 Azure Blob 存储中。

	hadoop fs -ls /example/data

有些命令可能需要你指定正在使用 Blob 存储。对于这些命令，你可以在它的前面加上前缀 **WASB://**。

HDInsight 还允许你将多个 Blob 存储帐户与群集相关联。若要访问非默认 Blob 存储帐户上的数据，可以使用以下格式：**WASB://&lt;容器名称>@&lt;帐户名>.blob.core.chinacloudapi.cn/**。例如，以下命令会列出指定容器和存储帐户的 **/example/data** 目录的内容。

	hadoop fs -ls wasb://mycontainer@mystorage.blob.core.chinacloudapi.cn/example/data

### 群集使用哪个 Blob 存储？

在创建群集过程中，你可以选择了使用现有存储帐户和容器，或者新建。事后，你可能会忘记了该存储帐户和容器。可以使用以下方法查找存储帐户和容器。

**Azure 门户**

1. 在 <a href="https://manage.windowsazure.cn/" target="_blank">Azure 管理门户</a>中，选择你的 HDInsight 群集。

2. 选择页面顶部的"仪表板"。

3. 页面的"链接的资源"部分会列出存储帐户和容器。

	![linked resources](./media/hdinsight-hadoop-linux-information/storageportal.png)

**Azure 跨平台命令行界面**

*即将推出！*

### 如何访问 Blob 存储？

除了通过群集的 Hadoop 命令，还有各种不同方式可用来访问 Blob：

* <a href="/documentation/articles/xplat-cli/" target="_blank">Azure 跨平台命令行界面</a> - 安装好后，请参阅 `azure storage` 以获取有关使用存储的帮助，或参阅 `azure blob` 了解 Blob 特定的命令。

* 各种 SDK：

	* <a href="https://github.com/Azure/azure-sdk-for-java" target="_blank">Java</a>

	* <a href="https://github.com/Azure/azure-sdk-for-node" target="_blank">Node.js</a>

	* <a href="https://github.com/Azure/azure-sdk-for-php" target="_blank">PHP</a>

	* <a href="https://github.com/Azure/azure-sdk-for-python" target="_blank">Python</a>

	* <a href="https://github.com/Azure/azure-sdk-for-ruby" target="_blank">Ruby</a>

	* <a href="https://github.com/Azure/azure-sdk-for-net" target="_blank">.NET</a>

* <a href="https://msdn.microsoft.com/zh-cn/library/azure/dd135733.aspx" target="_blank">存储 REST API</a>


## 后续步骤

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce)


<!--HONumber=50-->