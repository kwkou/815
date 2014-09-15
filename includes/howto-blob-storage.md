## 什么是 Blob 存储

Azure Blob 存储是一项可存储大量非结构化数据
的服务，用户可在世界任何地方通过 HTTP 或 HTTPS 访问
这些数据。一个 Blob 的大小可以为数百 GB。

一个 Azure 存储帐户可以包含大量 Blob、队列和表数据。有关存储帐户容量的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标][]。

Blob 存储的常见用途包括：

-   直接向浏览器提供图像或文档
-   存储文件以供分布式访问
-   对视频和音频进行流式处理
-   执行安全备份和灾难恢复
-   存储数据以供本地或 Azure 托管服务执行分析

你可以使用 Blob 存储向外公布数据，或者私下公布数据以用作
内部应用程序存储。

## 概念

Blob 服务包含以下组件：

![Blob1][]

-   **存储帐户：** 对 Azure 存储空间进行的所有访问都要
    通过存储帐户完成。有关存储帐户容量的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标][]。

-   **容器：** 容器可对 Blob 集进行分组。所有 Blob 都必须
    位于一个容器中。一个帐户可以包含无限个容器。
    一个容器可以存储无限个 Blob。

-   **Blob：** 任何类型和大小的文件。Azure 存储空间中可存储两种类型的 Blob：
    块 Blob 和页 Blob。
    大多数文件都是块 Blob。单个块 Blob 最大可以为 200 GB。
    本教程使用的是块 Blob。另一种 Blob 类型为页 Blob，
    其大小可以达 1 TB，在对文件中的一系列字节进行频繁修改时，
    这种 Blob 类型更加高效。有关 Blob 的详细信息，
    请参阅[了解块 Blob 和页 Blob][]。

-   **URL 格式：** Blob 是可寻址的，使用
    以下 URL 格式：
    <http://>`<storage account>`.blob.core.chinacloudapi.cn/`<container>`/`<blob>`

    可以使用以下示例 URL 表示上面的示意图中的 Blob 之一
    的地址
    ：`http://sally.blob.core.windows.net/movies/MOV1.AVI`

  [Azure 存储空间可伸缩性和性能目标]: http://msdn.microsoft.com/zh-cn/library/dn249410.aspx
  [Blob1]: ./media/howto-blob-storage/blob1.jpg
  [了解块 Blob 和页 Blob]: http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx
