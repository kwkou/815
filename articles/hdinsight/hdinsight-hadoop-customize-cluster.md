<!-- not suitable for Mooncake -->

<properties
	pageTitle="使用脚本操作自定义 HDInsight 群集 | Microsoft Azure"
	description="了解如何使用脚本操作自定义 HDInsight 群集。"
	services="hdinsight"
	documentationCenter=""
	authors="nitinme"
	manager="paulettm"
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="12/30/2015"
	wacn.date="02/06/2017"/>

# 使用脚本操作自定义基于 Windows 的 HDInsight 群集

[AZURE.INCLUDE [选择器](../../includes/hdinsight-create-windows-cluster-selector.md)]

创建群集期间，可以使用**脚本操作**调用[自定义脚本](/documentation/articles/hdinsight-hadoop-script-actions/)，以便在群集上安装其他软件。

也可以使用其他各种方法自定义 HDInsight 群集，例如包括其他 Azure 存储帐户、更改 hadoop 配置文件（core-site.xml、hive-site.xml 等），或者将共享库（如 Hive、Oozie）添加到群集中的共同位置。这些自定义可以通过 Azure PowerShell、Azure HDInsight .NET SDK 或 Azure 门户预览完成。有关详细信息，请参阅[在 HDInsight 中创建 Hadoop 群集][hdinsight-provision-cluster]。

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell-cli-and-dotnet-sdk.md)]

## 群集创建期间的脚本操作

只能在创建群集期间使用脚本操作。下图说明了在创建期间执行脚本操作的时间：

![群集创建期间的 HDInsight 群集自定义和阶段][img-hdi-cluster-states]

当脚本运行时，群集进入 **ClusterCustomization** 阶段。在此阶段，脚本在系统管理员帐户下，以并行方式在群集中所有指定的节点上运行，并在节点上提供完全系统管理员权限。

> [AZURE.NOTE] 由于在 **ClusterCustomization** 阶段期间在群集节点上拥有系统管理员权限，因此可以使用脚本运行作业，例如停止和启动服务，包括 Hadoop 相关服务。因此，作为脚本的一部分，必须在脚本运行完成前，确保 Ambari 服务和其他 Hadoop 相关服务已启动且正在运行。创建时成功确定群集的运行状况和状态需要这些服务。如果更改群集上影响这些服务的任何配置，必须使用提供的帮助函数。有关帮助函数的详细信息，请参阅[为 HDInsight 开发脚本操作脚本][hdinsight-write-script]。

脚本的输出和错误日志文件存储在为群集指定的默认存储帐户中。这些日志存储在名为 **u<\\cluster-name-fragment><\\time-stamp>setuplog** 的表中。这是从群集中所有节点（头节点和辅助节点）上运行的脚本聚合的日志文件。

每个群集可接受按指定顺序调用的多个脚本操作。脚本可在头节点和/或辅助节点上运行。

HDInsight 提供了多个脚本用于在 HDInsight 群集上安装以下组件：

Name | 脚本
----- | -----
**安装 R** | https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1。请参阅[在 HDInsight 群集上安装并使用 R][hdinsight-install-r]。
**安装 Solr** | https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1。请参阅[在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install/)。
- **安装 Giraph** | https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1。请参阅[在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install/)。



## 使用 Azure 门户预览调用脚本

**通过 Azure 门户预览**

1. 使用“自定义创建”选项开始预配群集，如[使用自定义选项预配群集](/documentation/articles/hdinsight-provision-clusters/)中所述。 
2. 在向导的“脚本操作”页上，单击“添加脚本操作”，以提供有关脚本操作的详细信息，如下所示：

	![使用脚本操作自定义群集](./media/hdinsight-hadoop-customize-cluster/HDI.CreateCluster.8.png "使用脚本操作自定义群集")

	<table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>Name</td>
			<td>指定脚本操作的名称。</td></tr>
		<tr><td>脚本 URI</td>
			<td>指定调用以自定义群集的脚本的 URI。</td></tr>
		<tr><td>头节点/辅助节点</td>
			<td>指定运行自定义脚本的节点（**Head** 或 **Worker**）。</b>
		<tr><td>Parameters</td>
			<td>根据脚本的需要，请指定参数。</td></tr>
	</table>按 ENTER 可添加多个脚本操作，以便在群集上安装多个组件。

3. 单击“选择”可保存脚本操作配置，并继续创建群集。

##<a name="call_scripts_using_powershell" id="call-scripts-using-azure-powershell"></a>使用 Azure PowerShell 调用脚本

使用适用于 HDInsight 的 Azure PowerShell 命令运行单个脚本操作或多个脚本操作。可以使用 **<a href = "http://msdn.microsoft.com/zh-cn/library/dn858088.aspx" target="_blank">Add-AzureHDInsightScriptAction</a>** cmdlet 调用自定义脚本。要使用这些 cmdlet，必须已经安装并配置 Azure PowerShell。有关配置工作站以运行适用于 HDInsight 的 Azure Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

使用以下 Azure PowerShell 命令，可在部署 HDInsight 群集时运行单个脚本操作：

	$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes 4

	$config = Add-AzureHDInsightScriptAction -Config $config -Name MyScriptActionName -Uri http://uri.to/scriptaction.ps1 -Parameters MyScriptActionParameter -ClusterRoleCollection HeadNode,DataNode

	New-AzureHDInsightCluster -Config $config

使用以下 Azure PowerShell 命令，可在部署 HDInsight 群集时运行多个脚本操作：

	$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes 4

	$config = Add-AzureHDInsightScriptAction -Config $config -Name MyScriptActionName1 -Uri http://uri.to/scriptaction1.ps1 -Parameters MyScriptAction1Parameters -ClusterRoleCollection HeadNode,DataNode | Add-AzureHDInsightScriptAction -Config $config -Name MyScriptActionName2 -Uri http://uri.to/scriptaction2.ps1 -Parameters MyScriptAction2Parameters -ClusterRoleCollection HeadNode

	New-AzureHDInsightCluster -Config $config

## 使用 .NET SDK 调用脚本 

HDInsight .NET SDK 可提供 <a href="http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.management.hdinsight.clusterprovisioning.data.scriptaction.aspx" target="_blank">ScriptAction</a> 类，以调用自定义脚本。使用 HDInsight .NET SDK：

1. 创建一个 Visual Studio 应用程序，然后从 Nuget 安装 SDK。在“工具”菜单中，单击“Nuget 包管理器”，然后单击“包管理器控制台”。在控制台中运行下列命令以安装程序包：

		Install-Package Microsoft.WindowsAzure.Management.HDInsight

2. 使用 SDK 创建群集。

3. 使用 **ScriptAction** 类调用自定义脚本，如下所示：

		
		var clusterInfo = new ClusterCreateParameters()
		{
			// Provide the cluster information, like
			// name, Storage account, credentials,
			// cluster size, and version		    
			...
			...
		};

		// Add the script action to install Spark
		clusterInfo.ConfigActions.Add(new ScriptAction(
	  		"MyScriptActionName", // Name of the config action
	  		new ClusterNodeType[] { ClusterNodeType.HeadNode }, // List of nodes to install the component on
	  		new Uri("http://uri.to/scriptaction.ps1"), // Location of the script to install the component
	  		"MyScriptActionParameter" //Parameters, if any, required by the script
		));

4. 按 **F5** 运行应用程序。


## 支持 HDInsight 群集上使用的开源软件
Microsoft Azure HDInsight 服务是一个灵活的平台，可使用围绕 Hadoop 形成的开源技术生态系统，在云中生成大数据应用程序。Microsoft Azure 可为开源技术提供一般级别支持，如 <a href="/support/faq/" target="_blank">Azure 支持常见问题网站</a>上的**支持范围**节中所述。HDInsight 服务可为某些组件提供额外级别的支持，如下所述。

HDInsight 服务提供两种类型的开源组件：

- **内置组件** - 这些组件预先安装在 HDInsight 群集上，并提供群集的核心功能。例如，Yarn ResourceManager、Hive 查询语言 (HiveQL) 和 Mahout 库均属于此类别。[HDInsight 提供的 Hadoop 群集版本有哪些新增功能](/documentation/articles/hdinsight-component-versioning/)</a>中提供了群集组件的完整列表。
- **自定义组件** - 作为群集用户，可以安装，或者在工作负荷中使用由社区提供或自己创建的任何组件。

完全支持内置组件，Azure 支持部门将帮助找出并解决与这些组件相关的问题。

> [AZURE.WARNING] 完全支持通过 HDInsight 群集提供的组件，Azure 支持部门将帮助找出并解决与这些组件相关的问题。
>
> 自定义组件可获得合理范围的支持，有助于进一步解决问题。这可能会促进解决问题，或要求使用可用的开源技术渠道，在渠道中可找到该技术的深厚的专业知识。有许多可以使用的社区站点，例如：[HDInsight 的 MSDN 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=hdinsight)、[Azure CSDN](http://azure.csdn.net)。此外，Apache 项目在 [http://apache.org](http://apache.org) 上提供了项目站点，例如 [Hadoop](http://hadoop.apache.org/)。

HDInsight 服务可提供多种方法使用自定义组件。无论在群集上使用或安装组件的方式如何，均适用相同级别的支持。以下是可在 HDInsight 群集上使用自定义组件的最常见方法列表：

1. 作业提交 - 可将 Hadoop 或其他类型的作业提交到执行或使用自定义组件的群集。
2. 群集自定义 - 在群集创建期间，可指定将安装在群集节点的其他设置和自定义组件。
3. 示例 - 对于常见的自定义组件，Microsoft 和其他用户可能会提供演示如何在 HDInsight 群集上使用这些组件的示例。不会针对这些示例提供支持。

## 开发脚本操作脚本

请参阅[为 HDInsight 开发脚本操作脚本][hdinsight-write-script]。


## 另请参阅

- [在 HDInsight 中创建 Hadoop 群集][hdinsight-provision-cluster]介绍如何使用其他自定义选项创建 HDInsight 群集。
- [为 HDInsight 开发脚本操作脚本][hdinsight-write-script]
- [在 HDInsight 群集上安装并使用 R][hdinsight-install-r]
- [在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install/)
- [在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install/)

[hdinsight-install-r]: /documentation/articles/hdinsight-hadoop-r-scripts/
[hdinsight-write-script]: /documentation/articles/hdinsight-hadoop-script-actions/
[hdinsight-provision-cluster]: /documentation/articles/hdinsight-provision-clusters/
[powershell-install-configure]: /documentation/articles/powershell-install-configure/


[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster/HDI-Cluster-state.png "群集创建期间的阶段"

<!---HONumber=Mooncake_Quality_Review_1202_2016-->