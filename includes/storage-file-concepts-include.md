## 什么是 Azure 文件存储？

文件存储使用标准 SMB 2.1 或 SMB 3.0 协议为应用程序提供共享存储。Azure 虚拟机和云服务可通过装载的共享在应用程序组件之间共享文件数据，本地应用程序可通过文件存储 API 来访问共享中的文件数据。

在 Azure 虚拟机或云服务中运行的应用程序可以装载文件存储共享以访问文件数据，就像桌面应用程序可以装载典型 SMB 共享一样。任意数量的 Azure 虚拟机或角色可以同时装载并访问文件存储共享。

由于文件存储共享是 Azure 中使用 SMB 协议的标准文件共享，在 Azure 中运行的应用程序可通过文件 I/O API 来访问共享中的数据。因此，开发人员可以利用其现有代码和技术迁移现有应用程序。IT 专业人员在管理 Azure 应用程序的过程中，可以使用 PowerShell cmdlet 来创建、装载和管理文件存储共享。本指南将演示这两方面的示例。

文件存储的常见用途包括：

- 迁移依赖文件共享在 Azure 虚拟机或云服务中运行的本地应用程序，而无需进行昂贵的重写操作
- 存储共享的应用程序设置，例如在配置文件中进行存储
- 在共享位置存储诊断数据，如日志、指标和故障转储 
- 存储开发或管理 Azure 虚拟机或云服务所需的工具和实用工具

## 文件存储概念

文件存储包含以下组件：

![files-concepts][files-concepts]

-   **存储帐户：**对 Azure 存储服务的所有访问都要通过存储帐户来完成。有关存储帐户容量的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

-   **共享：**文件存储共享是 Azure 中的 SMB 文件共享。所有目录和文件都必须在父共享中创建。一个帐户可以包含无限数量的共享，一个共享可以存储无限数量的文件，直到达到文件共享的 5TB 总容量限制为止。

-   **目录：**可选的目录层次结构。

-	**文件：**共享中的文件。文件大小最大可以为 1 TB。

-   **URL 格式：**可使用以下 URL 格式对文件寻址：  
	https://<storage account>.file.core.chinacloudapi.cn/<share>/<目录/目录>/<file>
    
    可使用以下示例 URL 寻址上图中的文件：`http://samples.file.core.chinacloudapi.cn/logs/CustomLogs/Log1.txt`

有关如何命名共享、目录和文件的详细信息，请参阅[命名和引用共享、目录、文件和元数据](http://msdn.microsoft.com/zh-cn/library/azure/dn167011.aspx)。

[files-concepts]: ./media/storage-file-concepts-include/files-concepts.png

<!---HONumber=Mooncake_0307_2016-->