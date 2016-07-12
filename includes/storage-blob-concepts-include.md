## 什么是 Blob 存储

Azure Blob 存储是用于存储大量非结构化数据（例如文本或二进制数据）的服务，这些数据可通过 HTTP 或 HTTPS 从世界各地进行访问。你可以使用 Blob 存储向外公开数据，或者私下存储应用程序数据。

Blob 存储的常见用途包括：

-   直接向浏览器提供图像或文档
-   存储文件以供分布式访问
-   对视频和音频进行流式处理
- 存储数据以用于备份和还原、灾难恢复及存档
-   存储数据以供本地或 Azure 托管服务执行分析

## Blob 服务概念

Blob 服务包含以下组件：

![Blob1][Blob1]

- **存储帐户：**对 Azure 存储服务的所有访问都要通过存储帐户来完成。此存储帐户可以是**常规用途存储帐户**，也可以是专用于存储对象/Blob 的 **Blob 存储帐户**。有关存储帐户的详细信息，请参阅 [Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。

-   **容器：**一个容器包含一组 blob 集。所有 blob 必须位于相应的容器中。一个帐户可以包含无限个容器。一个容器可以存储无限个 Blob。请注意，容器名称必须小写。

-   **Blob：**任何类型和大小的文件。Azure 存储空间提供三种类型的 Blob：块 Blob、页 Blob 和追加 Blob。
    
    *块 Blob*特别适用于存储短的文本或二进制文件，例如文档和媒体文件。*追加 Blob* 类似于块 Blob，因为它们是由块组成的，但针对追加操作对它们进行了优化，因此它们适用于日志记录方案。单个块 Blob 或追加 Blob 可以包含最多 50000 个块，每个块最大 4 MB，总大小稍微大于 195 GB (4 MB X 50000)。
    
	*页 Blob* 最大可达 1 TB 大小，并且对于频繁的读/写操作更加高效。Azure 虚拟机使用页 Blob 作为 OS 和数据磁盘。

	有关命名容器和 Blob 的详细信息，请参阅[命名和引用容器、Blob 和元数据](https://msdn.microsoft.com/zh-cn/library/azure/dd135715.aspx)。


[Blob1]: ./media/storage-blob-concepts-include/blob1.jpg

<!---HONumber=Mooncake_0530_2016-->