<properties
 pageTitle="了解和解决 HDInsight 上的 WebHCat 错误"
 description="了解 HDInsight 上的 WebHCat 返回的常见错误以及如何解决它们。"
 services="hdinsight"
 documentationCenter=""
 authors="Blackmist"
 manager="paulettm"
 editor="cgronlun"
 tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/22/2016"
	wacn.date="06/29/2016"/>

#了解和解决从 HDInsight 上的 WebHCat (Templeton) 收到的错误

将 WebHCat（以前称为 Templeton）与 HDInsight 配合使用时，你可能会收到错误。本文档提供常见错误指导 – 为什么会发生这些错误？如何解决这些错误？

##什么是 WebHCat？

[WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat)是适用于 [HCatalog](https://cwiki.apache.org/confluence/display/Hive/HCatalog) 的 REST API，是针对 Hadoop 的表和存储管理层。WebHCat 默认情况下在 HDInsight 群集上处于启用状态，可供各种工具在执行提交作业、获取作业状态等操作时使用，无需登录到群集中。

##<a name="modifying-configuration"></a> 修改配置

> [AZURE.IMPORTANT]本文档中列出的几大错误之所以发生，是因为超出了配置的最大值。当解决步骤提到你可以更改一个值时，必须使用下列选项之一来执行更改：

* 对于 **Windows** 群集：使用脚本操作在群集创建过程中配置值。有关详细信息，请参阅[开发脚本操作](/documentation/articles/hdinsight-hadoop-script-actions/)。

###默认配置

以下是在超过时可能影响 WebHCat 性能或导致错误的默认配置值：

| 设置 | 作用 | 默认值 |
| ------- | ------------ | ------------- |
| [yarn.scheduler.capacity.maximum-applications][maximum-applications] | 可以同时处于活动状态（挂起或运行）的最大作业数 | 10,000 |
| [templeton.exec.max-procs][max-procs] | 可以同时处理的最大请求数 | 20 |
| [mapreduce.jobhistory.max-age-ms][max-age-ms] | 保留作业历史记录的天数 | 7 天 |

##请求过多

**HTTP 状态代码**：429

| 原因 | 解决方法 |
| ----- | ---------- |
| 你已超过 WebHCat 每分钟能够处理的最大并发请求数（默认值为 20） | 减少工作负载以确保你提交的数量没有超出最大并发请求数，或者通过修改 `templeton.exec.max-procs` 来提高并发请求限制。请参阅[修改配置](#modifying-configuration)以获取更多信息 |

##服务器不可用

**HTTP 状态代码**：503

| 原因 | 解决方法 |
| ---------------- | ------------------- |
| 这通常发生在群集的主要和辅助 HeadNode 之间进行故障转移时。 | 等待两分钟，然后重试该操作 |

##请求内容错误：找不到作业

**HTTP 状态代码**：400

| 原因 | 解决方法 |
| ---------------- | ------------------- |
| 作业详细信息已被作业历史记录清除器清除 | 作业历史记录的默认保留期为 7 天。这可以通过修改 `mapreduce.jobhistory.max-age-ms` 进行更改。请参阅[修改配置](#modifying-configuration)以获取更多信息 |
| 作业因故障转移而终止 | 重试提交作业，重试时间最多两分钟 |
| 使用了无效的作业 ID | 检查作业 ID 是否正确 |

##网关错误

**HTTP 状态代码**：502

| 原因 | 解决方法 |
| ---------------- | ------------------- |
| WebHCat 进程内发生内部垃圾回收 | 等待垃圾回收完成或重新启动 WebHCat 服务 |
| 等待 ResourceManager 服务的响应超时。当活动应用程序的数量达到配置的最大值（默认为 10,000）时，可能会发生这种情况 | 等待当前正在运行的作业完成，或者通过修改 `yarn.scheduler.capacity.maximum-applications` 来提高并发作业限制。请参阅[修改配置](#modifying-configuration)以获取更多信息 |
| 尝试通过 [GET /jobs](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference+Jobs) 调用检索所有作业而 `Fields` 设置为 `*` 时 | 不检索*所有*作业详细信息，而是改用 `jobid` 来检索作业 ID 大于特定作业 ID 的作业的详细信息。或者，不使用 `Fields` |
| 在 HeadNode 故障转移期间 WebHCat 服务关闭 | 等待两分钟，然后重试该操作 |
| 通过 WebHCat 提交的作业有超过 500 个处于挂起状态 | 等到当前挂起的作业完成再提交更多作业 |

[maximum-applications]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/setting_application_limits.html
[max-procs]: https://hive.apache.org/javadocs/hcat-r0.5.0/configuration.html
[max-age-ms]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.6.0/ds_Hadoop/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml
 

<!---HONumber=Mooncake_1207_2015-->