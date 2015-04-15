## 什么是 Blob 存储

Azure Blob 存储是一项可存储大量非结构化数据（如文本或二进制数据）的服务，用户可在世界任何地方通过 HTTP 或 HTTPS 访问这些数据。你可以使用 Blob 存储向外公开数据，或者私下存储应用程序数据。

Blob 存储的常见用途包括：

-   直接为浏览器提供图像或文档
-   存储文件以供分布式访问
-   流式传输视频和音频
-   执行安全备份和灾难恢复
-   存储数据以供本地或 Azure 托管服务执行分析

## Blob 服务概念

Blob 服务包含以下组件：

![Blob1][Blob1]

-   **存储帐户：**对 Azure 存储的所有访问都通过存储帐户完成。有关存储帐户容量的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](http://msdn.microsoft.com/zh-cn/library/azure/dn249410.aspx)。

-   **容器：**容器提供一组 blob 的分组。所有 blob 都必须在一个容器中。一个帐户可包含无限数量的容器。一个容器可存储无限数量的 blob。

-   **Blob：**任何类型和大小的文件。有两种类型的 blob 可存储在 Azure 存储中：块和页 blob。大部分文件是块 blobs。单个块 blob 最大可以为 200 GB。本教程使用块 blob。页 blob、其他 blob 类型最大可以为 1 TB，并且当文件中字节的范围频繁修改时更高效。有关 blob 的详细信息，请参阅[了解块 Blob 和页 Blob][]。

-   **URL 格式：**可使用以下 URL 格式对 Blob 寻址： http://`<storage account>`.blob.core.chinacloudapi.cn/`<container>`/`<blob>`

    可以使用以下示例 URL 表示上面的示意图中的 Blob 之一的地址： `http://sally.blob.core.chinacloudapi.cn/movies/MOV1.AVI`


[Blob1]: ./media/storage-blob-concepts-include/blob1.jpg


<!--HONumber=50-->