<properties
    pageTitle="了解和解决 HDInsight 上的 WebHCat 错误"
    description="了解 HDInsight 上的 WebHCat 返回的常见错误以及如何解决它们。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="1b3d94b1-207d-4550-aece-21dc45485549"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/04/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="3ec907c37bcc7f209fed4a175d3ef9073407ad0b"
    ms.lasthandoff="04/28/2017" />

# <a name="understand-and-resolve-errors-received-from-webhcat-on-hdinsight"></a>了解和解决从 HDInsight 上的 WebHCat 收到的错误

了解将 WebHCat 用于 HDInsight 时接收的错误以及如何解决错误。 WebHCat 由 Azure PowerShell 和 Data Lake Tools for Visual Studio 等客户端工具在内部使用。

## <a name="what-is-webhcat"></a>什么是 WebHCat

[WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) 是适用于 [HCatalog](https://cwiki.apache.org/confluence/display/Hive/HCatalog) 的 REST API，是针对 Hadoop 的表和存储管理层。 WebHCat 默认情况下在 HDInsight 群集上处于启用状态，可供各种工具在执行提交作业、获取作业状态等操作时使用，无需登录到群集中。

## <a name="modifying-configuration"></a>修改配置

> [AZURE.IMPORTANT]
> 本文档中列出的几大错误之所以发生，是因为超出了配置的最大值。 当解决步骤提到你可以更改一个值时，必须使用下列选项之一来执行更改：

* 对于 **Windows** 群集：使用脚本操作在群集创建过程中配置值。 有关详细信息，请参阅[开发脚本操作](/documentation/articles/hdinsight-hadoop-script-actions/)。

* 对于 **Linux** 群集：使用 Ambari（Web 或 REST API）修改值。 有关详细信息，请参阅[使用 Ambari 管理 HDInsight](/documentation/articles/hdinsight-hadoop-manage-ambari/)

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅[弃用 HDInsight 版本 3.2 和 3.3](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

### <a name="default-configuration"></a>默认配置

如果超过以下默认值，则可能降低 WebHCat 性能或导致错误：

| 设置 | 作用 | 默认值 |
| --- | --- | --- |
| [yarn.scheduler.capacity.maximum-applications][maximum-applications] |可以同时处于活动状态（挂起或运行）的最大作业数 |10,000 |
| [templeton.exec.max-procs][max-procs] |可以同时处理的最大请求数 |20 |
| [mapreduce.jobhistory.max-age-ms][max-age-ms] |作业历史记录保留的天数 |7 天 |

## <a name="too-many-requests"></a>请求过多

**HTTP 状态代码**：429

| 原因 | 解决方法 |
| --- | --- |
| 你已超过 WebHCat 每分钟能够处理的最大并发请求数（默认值为 20） |减少工作负载以确保你提交的数量没有超出最大并发请求数，或者通过修改 `templeton.exec.max-procs` 来提高并发请求限制。 有关详细信息，请参阅[修改配置](#modifying-configuration) |

## <a name="server-unavailable"></a>服务器不可用

**HTTP 状态代码**：503

| 原因 | 解决方法 |
| --- | --- |
| 此状态代码通常发生在群集的主要和辅助 HeadNode 之间进行故障转移时 |等待两分钟，然后重试该操作 |

## <a name="bad-request-content-could-not-find-job"></a>请求内容错误：找不到作业

**HTTP 状态代码**：400

| 原因 | 解决方法 |
| --- | --- |
| 作业详细信息已被作业历史记录清除器清除 |作业历史记录的默认保留期为 7 天。 通过修改 `mapreduce.jobhistory.max-age-ms` 可更改默认保留期。 有关详细信息，请参阅[修改配置](#modifying-configuration) |
| 作业因故障转移而终止 |重试提交作业，重试时间最多两分钟 |
| 使用了无效的作业 ID |检查作业 ID 是否正确 |

## <a name="bad-gateway"></a>网关错误

**HTTP 状态代码**：502

| 原因 | 解决方法 |
| --- | --- |
| WebHCat 进程内发生内部垃圾回收 |等待垃圾回收完成或重新启动 WebHCat 服务 |
| 等待 ResourceManager 服务的响应超时。 当活动应用程序的数量达到配置的最大值（默认为 10,000）时，可能会发生这种情况 |等待当前正在运行的作业完成，或者通过修改 `yarn.scheduler.capacity.maximum-applications` 来提高并发作业限制。 请参阅[修改配置](#modifying-configuration)以获取详细信息 |
| 在 `Fields` 设置为 `*` 时，尝试通过 [GET /jobs ](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference+Jobs) 调用来检索所有作业 |不检索全部作业详细信息。 而是改用 `jobid` 来仅检索作业 ID 大于特定作业 ID 的作业的详细信息。 或者，不使用 `Fields` |
| 在 HeadNode 故障转移期间 WebHCat 服务关闭 |等待两分钟，然后重试该操作 |
| 通过 WebHCat 提交的作业有超过 500 个处于挂起状态 |等到当前挂起的作业完成再提交更多作业 |

[maximum-applications]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/setting_application_limits.html
[max-procs]: https://hive.apache.org/javadocs/hcat-r0.5.0/configuration.html
[max-age-ms]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.6.0/ds_Hadoop/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml

<!--Update_Description: wording update-->