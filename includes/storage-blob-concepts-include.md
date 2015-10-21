## 什么是 Blob 存储

Azure Blob 存储是用于存储大量非结构化数据（例如文本或二进制数据）的服务，这些数据可通过 HTTP 或 HTTPS 从世界各地进行访问。你可以使用 Blob 存储向外公开数据，或者存储应用程序的私密数据。

Blob 存储的常见用途包括：

-   直接向浏览器提供图像或文档
-   存储文件以供分布式访问
-   对视频和音频进行流式处理
-   执行安全备份和灾难恢复
-   存储数据以供本地或 Azure 托管服务执行分析

## Blob 服务概念

Blob 服务包含以下组件：

![Blob1][Blob1]

-   **存储帐户：**对 Azure 存储服务的所有访问都要通过存储帐户来完成。有关存储帐户容量的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets)。

-   **容器：**一个容器包含一组 blob 集。所有 blob 必须位于相应的容器中。一个帐户可以包含无限个容器。一个容器可以存储无限个 Blob。

-   **Blob：**任何类型和大小的文件。Azure 存储空间提供三种类型的 Blob：块 Blob、页 Blob 和追加 Blob。
    
	*块 Blob*特别适用于存储文本或二进制文件，例如文档和媒体文件。*追加 Blob* 类似于块 Blob，因为它们都是由块组成的，但针对追加操作对它们进行了优化，因此它们适用于日志记录方案。单个块 Blob 或追加 Blob 可以包含最多 50000 个块，每个块最大 4 MB，总大小稍微大于 195 GB (4 MB X 50000)。
    
	*页 Blob* 最大可达 1 TB 大小，并且对于频繁的读/写操作更加高效。Azure 虚拟机使用页 Blob 作为 OS 和数据磁盘。

	有关 Blob 的更多信息，请参阅[了解块 Blob、页 Blob 和追加 Blob](https://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)。

## 命名和引用容器和 Blob

可以使用以下 URL 格式在您的存储帐户中定位 Blob：
   
    http://<storage-account-name>.blob.core.chinacloudapi.cn/<container-name>/<blob-name>  
      
例如，以下 URL 定位上图中的一个 Blob：

    http://sally.blob.core.chinacloudapi.cn/movies/MOV1.AVI

### 容器命名规则

容器名称必须是有效的 DNS 名称，并符合以下规则：

- 容器名称必须全部都是小写
- 容器名称必须以字母或数字开头，并且只能包含字母、数字和短划线 (-) 字符。
- 每个短划线 (-) 字符的前面和后面都必须是一个字母或数字；在容器名称中不允许连续的短划线 (-)。
- 容器名称必须介于 3 到 63 个字符。

### Blob 命名规则

Blob 名称必须符合以下规则：

- Blob 名称可以包含任意字符组合。
- Blob 名称的长度必须至少为一个字符并且不能超过 1024 个字符。
- Blob 名称区分大小写。
- 必须正确转义保留的 URL 字符。
- 组成 Blob 名称的路径段的数量不能超过 254 个。路径段指相邻分隔符（例如，正斜杠 /）之间对应于一个虚拟目录名称的字符串。

Blob 服务基于一个平铺的存储结构。可以通过在 Blob 名称内指定一个用于创建虚拟层次结构的字符或字符串分隔符来创建虚拟层次结构。例如，下面的列表显示了一些有效且唯一的 Blob 名称：

	/a
	/a.txt
	/a/b
	/a/b.txt

您可以使用分隔符按层次结构列出 Blob。


[Blob1]: ./media/storage-blob-concepts-include/blob1.jpg

<!---HONumber=70-->