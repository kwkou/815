<properties
    pageTitle="使用 Java 的 Azure 存储示例 | Azure"
    description="查看、下载和运行 Azure 存储空间的示例代码和应用程序使用 Java 存储客户端库发现 Blob、队列、表和文件的入门示例。"
    services="storage"
    documentationcenter="na"
    author="seguler"
    manager="jahogg"
    editor="tysonn" />
<tags
    ms.service="storage"
    ms.devlang="java"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage"
    ms.date="01/12/2017"
    wacn.date="02/24/2017"
    ms.author="seguler" />  


# 使用 Java 的 Azure 存储示例

## Java 示例索引

>[AZURE.IMPORTANT] 若要使用本文中提供的示例，请将终结点 `windows.net` 替换为 `chinacloudapi.cn`（如果存在）。


下表概述了我们的示例存储库以及每个示例中介绍的方案。单击链接查看 Github 中的相应示例代码。

<table style="font-size:90%"><thead><tr><th style="font-size:110%">终结点</th><th style="font-size:110%">方案</th><th style="font-size:110%">代码示例</th></tr></thead><tbody> 
<tr> 
<td rowspan="16"><b>Blob</b></td>
<td>追加 Blob</td> 
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td> 
</tr> 
<tr> 
<td>块 blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>客户端加密</td>
<td><a href="https://github.com/Azure-Samples/storage-java-client-side-encryption">Java 中的 Azure 客户端加密入门</a></td>
</tr> 
<tr> 
<td>复制 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>创建容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>删除 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>删除容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>Blob 元数据/属性/统计信息</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobAdvanced.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>容器 ACL/元数据/属性</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobAdvanced.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>获取页面范围</td>
<td><a href="https://github.com/Azure/azure-storage-java/blob/master/microsoft-azure-storage-test/src/com/microsoft/azure/storage/blob/CloudPageBlobTests.java">页 Blob 测试示例</a></td>
</tr> 
<tr> 
<td>租赁 Blob/容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>列出 Blob/容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td>页 blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr>
<tr> 
<td>SAS</td>
<td><a href="https://github.com/Azure/azure-storage-java/blob/master/microsoft-azure-storage-test/src/com/microsoft/azure/storage/blob/SasTests.java">SAS 测试示例</a></td>
</tr> 	
<tr> 
<td>服务属性</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobAdvanced.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 			
<tr> 
<td>快照 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-java-getting-started/blob/master/src/BlobBasics.java">Java 中的 Azure Blob 服务入门</a></td>
</tr> 
<tr> 
<td rowspan="9"><b>文件</b></td>
<td>创建共享/目录/文件</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileBasics.java">Java 中的 Azure 文件服务入门</a></td> 
</tr>
<tr> 
<td>删除共享/目录/文件</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileBasics.java">Java 中的 Azure 文件服务入门</a></td> 
</tr> 
<tr> 
<td>目录属性/元数据</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileAdvanced.java">Java 中的 Azure 文件服务入门</a></td> 
</tr> 
<tr> 
<td>下载文件</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileBasics.java">Java 中的 Azure 文件服务入门</a></td> 
</tr> 
<tr> 
<td>文件属性/元数据/指标</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileAdvanced.java">Java 中的 Azure 文件服务入门</a></td> 
</tr> 
<tr> 
<td>文件服务属性</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileAdvanced.java">Java 中的 Azure 文件服务入门</a></td> 
</tr> 
<tr> 
<td>列出目录和文件</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileBasics.java">Java 中的 Azure 文件服务入门</a></td> 
</tr>
<tr> 
<td>列出共享</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileBasics.java">Java 中的 Azure 文件服务入门</a></td> 
</tr>
<tr> 
<td>共享属性/元数据/统计信息</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-java-getting-started/blob/master/src/FileAdvanced.java">Java 中的 Azure 文件服务入门</a></td> 
</tr>
<tr> 
<td rowspan="8"><b>队列</b></td>
<td>添加消息</td> 
<td><a href="https://github.com/Azure/azure-storage-java/blob/master/microsoft-azure-storage-samples/src/com/microsoft/azure/storage/queue/gettingstarted/QueueBasics.java">存储 Java 客户端库示例</a></td> 
</tr> 
<tr> 
<td>客户端加密</td> 
<td><a href="https://github.com/Azure/azure-storage-java/blob/master/microsoft-azure-storage-samples/src/com/microsoft/azure/storage/encryption/queue/gettingstarted/QueueGettingStarted.java">存储 Java 客户端库示例</a></td> 
</tr> 
<tr> 
<td>创建队列</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-java-getting-started/blob/master/src/QueueBasics.java">Java 中的 Azure 队列服务入门</a></td> 
</tr> 
<tr> 
<td>删除消息/队列</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-java-getting-started/blob/master/src/QueueBasics.java">Java 中的 Azure 队列服务入门</a></td> 
</tr> 
<tr> 
<td>速览消息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-java-getting-started/blob/master/src/QueueBasics.java">Java 中的 Azure 队列服务入门</a></td> 
</tr> 
<tr> 
<td>队列 ACL/元数据/统计信息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-java-getting-started/blob/master/src/QueueAdvanced.java">Java 中的 Azure 队列服务入门</a></td> 
</tr> 
<tr> 
<td>队列服务属性</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-java-getting-started/blob/master/src/QueueAdvanced.java">Java 中的 Azure 队列服务入门</a></td> 
</tr> 
<tr> 
<td>更新消息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-java-getting-started/blob/master/src/QueueBasics.java">Java 中的 Azure 队列服务入门</a></td> 
</tr> 
<tr> 
<td rowspan="7"><b>表</b></td>
<td>创建表</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/TableBasics.java">Java 中的 Azure 表服务入门</a></td> 
</tr> 
<tr> 
<td>删除实体/表</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/TableBasics.java">Java 中的 Azure 表服务入门</a></td> 
</tr> 
<tr> 
<td>插入/合并/替换实体</td> 
<td><a href="https://github.com/Azure/azure-storage-java/blob/master/microsoft-azure-storage-samples/src/com/microsoft/azure/storage/table/gettingtstarted/TableBasics.java">存储 Java 客户端库示例</a></td> 
</tr> 
<tr> 
<td>查询实体</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/TableBasics.java">Java 中的 Azure 表服务入门</a></td> 
</tr> 
<tr> 
<td>查询表</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/TableBasics.java">Java 中的 Azure 表服务入门</a></td> 
</tr> 
<tr> 
<td>表 ACL/属性</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/TableAdvanced.java">Java 中的 Azure 表服务入门</a></td> 
</tr> 
<tr> 
<td>更新实体</td> 
<td><a href="https://github.com/Azure/azure-storage-java/blob/master/microsoft-azure-storage-samples/src/com/microsoft/azure/storage/table/gettingtstarted/TableBasics.java">存储 Java 客户端库示例</a></td> 
</tr> 
</tbody> 
</table>
<br/>  


## Azure 代码示例库

若要查看完整的示例库，请访问 [Azure 代码示例](https://azure.microsoft.com/resources/samples/?service=storage)库，其中包括 Azure 存储的示例，可以下载并在本地运行。代码示例库提供 .zip 格式的示例代码。此外，用户也可以浏览 GitHub 存储库以获取每个示例并进行克隆。

[AZURE.INCLUDE [storage-java-samples-include](../../includes/storage-java-samples-include.md)]

## 入门指南

有关 Azure 存储客户端库的安装和入门说明，请查看以下指南。

* [Java 中的 Azure Blob 服务入门](/documentation/articles/storage-java-how-to-use-blob-storage/)
* [Java 中的 Azure 队列服务入门](/documentation/articles/storage-java-how-to-use-queue-storage/)
* [Java 中的 Azure 表服务入门](/documentation/articles/storage-java-how-to-use-table-storage/)
* [Java 中的 Azure 文件服务入门](/documentation/articles/storage-java-how-to-use-file-storage/)

## 后续步骤

了解其他语言的示例：

* .NET：[使用 .NET 的 Azure 存储示例](/documentation/articles/storage-samples-dotnet/)
* 所有其他语言：[Azure 存储示例](/documentation/articles/storage-samples/)

<!---HONumber=Mooncake_0220_2017-->