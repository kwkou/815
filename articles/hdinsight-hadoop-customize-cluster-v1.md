<properties 
	pageTitle="使用脚本操作自定义 HDInsight 群集 | Azure" 
	description="了解如何使用脚本操作自定义 HDInsight 群集。" 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="02/26/2016"
	wacn.date="04/26/2016"/> 

# 使用脚本操作自定义 HDInsight 群集

HDInsight 提供了一个称为**脚本操作**的配置选项，该脚本操作可调用自定义脚本，以定义设置过程中要在群集上执行自定义。这些脚本可用于在群集上安装额外的软件或更改群集上的应用程序配置。


> [AZURE.NOTE]只有在随附 Windows 操作系统的 HDInsight 群集 3.1 或更高版本上才支持脚本操作。有关 HDInsight 群集版本的详细信息，请参阅 [HDInsight 群集版本](/documentation/articles/hdinsight-component-versioning-v1/)。

你也可以使用多种其他方法来自定义 HDInsight 群集，例如包含其他存储帐户、更改 hadoop 配置文件（core-site.xml、hive-site.xml 等），或者将共享库（例如 Hive、Oozie）添加到群集中的共同位置。这些自定义可以通过使用 Azure PowerShell、Azure HDInsight .NET SDK 或 Azure 经典管理门户来完成。有关详细信息，请参阅[使用自定义选项在 HDInsight 中设置 Hadoop 群集][hdinsight-provision-cluster]。

## 群集设置过程中的脚本操作

只能在创建群集过程中使用脚本操作。下图演示了在设置过程中执行脚本操作的时间：

![群集设置过程中的 HDInsight 群集自定义和阶段][img-hdi-cluster-states]

当脚本运行时，群集进入 **ClusterCustomization** 阶段。在此阶段，脚本在系统管理员帐户下，以并行方式在群集中所有指定的节点上运行，而在节点上提供完全的系统管理员权限。

> [AZURE.NOTE]因为你在 **ClusterCustomization** 阶段中于群集节点上拥有系统管理员权限，所以你可以使用脚本来运行作业，例如停止和启动服务，包括 Hadoop 相关服务。因此，在脚本中，你必须在脚本完成运行之前，确定 Ambari 服务及其他 Hadoop 相关服务已启动并且正在运行。这些服务必须在群集创建时，成功地确定群集的运行状况和状态。如果你更改群集上的任何影响这些服务的配置，必须使用所提供的帮助器函数。有关帮助器函数的详细信息，请参阅[为 HDInsight 开发脚本操作脚本][hdinsight-write-script]。

脚本的输出以及错误日志文件存储在你为群集指定的默认存储帐户中。这些日志存储在名为 **u<\\cluster-name-fragment><\\time-stamp>setuplog** 的表中。这是从群集中所有节点上（头节点和辅助节点）运行的脚本聚合的日志文件。


每个群集可接受多个脚本操作，这些脚本依其指定顺序被调用。脚本可在头节点和/或工作线程节点上运行。

## 调用脚本操作脚本

可以从 Azure 经典管理门户、Azure PowerShell 或 HDInsight.NET SDK 使用脚本操作脚本。

HDInsight 提供了多个脚本用于在 HDInsight 群集上安装以下组件：

Name | 脚本
----- | -----
**安装 R** | https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1 。请参阅[在 HDInsight 群集上安装并使用 R][hdinsight-install-r]。
**安装 Solr** | https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1 。请参阅[在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install-v1/)。
- **安装 Giraph** | https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1 。请参阅[在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-v1/)。



**从 Azure 经典管理门户**

1. 根据[使用自定义选项预配群集](/documentation/articles/hdinsight-provision-clusters-v1/#portal)中的说明，使用“自定义创建”选项开始预配群集。 
2. 在向导的“脚本操作”页上，单击“添加脚本操作”，以提供有关脚本操作的详细信息，如下所示：

	![使用脚本操作自定义群集](./media/hdinsight-hadoop-customize-cluster-v1/HDI.CustomProvision.Page6.png "使用脚本操作自定义群集")
	
	<table border='1'>
	<tr><th>属性</th><th>值</th></tr>
	<tr><td>Name</td>
		<td>指定脚本操作的名称。</td></tr>
	<tr><td>脚本 URI</td>
		<td>指定要调用以自定义群集的脚本的 URI。</td></tr>
	<tr><td>节点类型</td>
		<td>指定在其上运行自定义脚本的节点。你可以选择“所有节点”、“仅限头节点”或“仅限从节点”<b></b><b></b><b></b>。
	<tr><td>Parameters</td>
		<td>根据脚本的需要，指定参数。</td></tr>
	</table>

	你可以添加多个脚本操作，以在群集上安装多个组件。

3. 单击复选标记以开始设置群集。
  
<a name="call-scripts-using-azure-powershell" id="call_scripts_using_azure_powershell"></a>
**从 Azure PowerShell cmdlet**

使用 Azure PowerShell 命令来运行单个脚本操作或多个脚本操作。你可以使用 **<a href = "http://msdn.microsoft.com/zh-cn/library/dn858088.aspx" target="_blank">Add-AzureHDInsightScriptAction</a>** cmdlet 调用自定义脚本。若要使用这些 cmdlet，你必须已安装并设置 Azure PowerShell。有关配置工作站以运行适用于 HDInsight 的 Azure Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

使用以下 Azure PowerShell 命令可以在部署 HDInsight 群集时运行单个脚本操作：

	$config = New-AzureHDInsightClusterConfig –ClusterSizeInNodes 4

	$config = Add-AzureHDInsightScriptAction -Config $config –Name MyScriptActionName –Uri http://uri.to/scriptaction.ps1 –Parameters MyScriptActionParameter -ClusterRoleCollection HeadNode,DataNode

	New-AzureHDInsightCluster -Config $config

使用以下 Azure PowerShell 命令可以在部署 HDInsight 群集时运行多个脚本操作：

	$config = New-AzureHDInsightClusterConfig –ClusterSizeInNodes 4

	$config = Add-AzureHDInsightScriptAction -Config $config –Name MyScriptActionName1 –Uri http://uri.to/scriptaction1.ps1 –Parameters MyScriptAction1Parameters -ClusterRoleCollection HeadNode,DataNode | Add-AzureHDInsightScriptAction -Config $config –Name MyScriptActionName2 –Uri http://uri.to/scriptaction2.ps1 -Parameters MyScriptAction2Parameters -ClusterRoleCollection HeadNode

	New-AzureHDInsightCluster -Config $config

<a name="call-scripts-using-net-sdk"></a>
**从 HDInsight .NET SDK**

HDInsight .NET SDK 提供了 <a href="http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.management.hdinsight.clusterprovisioning.data.scriptaction.aspx" target="_blank">ScriptAction</a> 类用于调用自定义脚本。使用 HDInsight .NET SDK：

1. 创建一个 Visual Studio 应用程序，然后从 Nuget 安装 SDK。在“工具”菜单中，单击“Nuget Package Manager”，然后单击“Package Manager Console”。在控制台中运行下列命令以安装程序包：

		Install-Package Microsoft.WindowsAzure.Management.HDInsight

2. 使用 SDK 创建群集。有关说明，请参阅[使用 .NET SDK 设置 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters-v1/#sdk)。

3. 使用 **ScriptAction** 类调用自定义脚本，如下所示：

		
		var clusterInfo = new ClusterCreateParameters()
		{
			// Provide the cluster information, like
			// name, Storage account, credentials,
			// cluster size, and version		    
			...
			...
		};



## 支持 HDInsight 群集上使用的开放源代码软件
Azure HDInsight 服务是一个弹性平台，可让你使用围绕着 Hadoop 形成的开放源代码技术生态系统，在云中生成大数据应用程序。Azure 为开放源代码技术提供一般级别的支持，如 <a href="/support/faq/" target="_blank">Azure 支持常见问题 Web 应用</a>上的**支持范围**部分中所述。HDInsight 服务为如下所述的某些组件提供附加的支持级别。

HDInsight 服务中有两种类型的开放源代码组件：

- **内置组件** - 这些组件预先安装在 HDInsight 群集上，并提供在群集的核心功能。例如，Yarn ResourceManager、Hive 查询语言 (HiveQL) 及 Mahout 库均属于此类别。<a href="/documentation/articles/hdinsight-component-versioning-v1/" target="_blank">HDInsight 提供的 Hadoop 群集版本有哪些新功能</a>中提供了群集组件的完整列表。
- **自定义组件** - 作为群集用户，你可以安装，或者在工作负荷中使用由社区提供的或你自己创建的任何组件。

完全支持内置组件，Microsoft 支持部门将帮助你找出并解决与这些组件相关的问题。

自定义组件可获得合理范围的支持，以帮助你进一步排查问题。这可能导致问题解决，或要求你参与可用的开放源代码技术渠道，在该处可找到该技术的深入专业知识。有许多可以使用的社区站点，例如：<a href ="https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hdinsight" target="_blank">HDInsight 的 MSDN 论坛</a>和<a href="http://azure.csdn.net/" target="_blank">CSDN Azure</a>。此外，Apache 项目在 <a href="http://apache.org" target="_blank">Apache.org</a> 上提供了项目站点，例如 <a href="http://hadoop.apache.org/" target="_blank">Hadoop</a>。

HDInsight 服务提供多种方式来使用自定义组件。不论在群集上使用组件或安装组件的方式为何，均适用相同级别的支持。以下是可以在 HDInsight 群集上使用的自定义组件最常见方式的列表：

1. 作业提交 - Hadoop 或其他类型的作业可以提交到执行或使用自定义组件的群集。
2. 群集自定义 - 在群集创建期间，你可以指定将安装在群集节点的其他设置和自定义组件。
3. 示例 - 对于常见的自定义组件，Microsoft 和其他人可能会提供演示如何在 HDInsight 群集上使用这些组件的示例。我们不针对这些示例提供支持。

## 开发脚本操作脚本

请参阅[为 HDInsight 开发脚本操作脚本][hdinsight-write-script]。


## 另请参阅

- [使用自定义选项在 HDInsight 中设置 Hadoop 群集][hdinsight-provision-cluster]说明了如何使用其他自定义选项来设置 HDInsight 群集。
- [为 HDInsight 开发脚本操作脚本][hdinsight-write-script]
- [在 HDInsight 群集上安装并使用 R][hdinsight-install-r]
- [在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install-v1/)
- [在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-v1/)

[hdinsight-install-r]: /documentation/articles/hdinsight-hadoop-r-scripts/
[hdinsight-write-script]: /documentation/articles/hdinsight-hadoop-script-actions/
[hdinsight-provision-cluster]: /documentation/articles/hdinsight-provision-clusters-v1/
[powershell-install-configure]: /documentation/articles/powershell-install-configure/
[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster-v1/HDI-Cluster-state.png "群集设置过程中的阶段"
 

<!---HONumber=74-->