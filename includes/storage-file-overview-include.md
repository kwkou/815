## 概述

Azure 文件存储是一种使用标准[服务器消息块 (SMB) 协议](https://msdn.microsoft.com/zh-cn/library/windows/desktop/aa365233.aspx)在云中提供文件共享的服务。支持 SMB 2.1 和 SMB 3.0。通过 Azure 文件存储，你可以将依赖文件共享的旧版应用程序快速迁移到 Azure 且无成本高昂的重写。在 Azure 虚拟机或云服务中或者从本地客户端运行的应用程序可以在云中装载文件共享，就像桌面应用程序装载典型的 SMB 共享一样。之后，任意数量的应用程序组件可以装载并同时访问文件存储共享。

由于文件存储共享是标准的 SMB 文件共享，在 Azure 中运行的应用程序可以通过文件系统 I/O API 访问共享中的数据。因此，开发人员可以利用其现有代码和技术迁移现有应用程序。IT 专业人员在管理 Azure 应用程序的过程中，可以使用 PowerShell cmdlet 来创建、装载和管理文件存储共享。

可以使用 [Azure 门户预览](https://portal.azure.cn)、Azure 存储空间 PowerShell cmdlet、Azure 存储空间客户端库或 Azure 存储空间 REST API 来创建 Azure 文件共享。此外，由于这些文件共享是 SMB 共享，因此你还可以通过标准的和熟悉的文件系统 API 来访问它们。

<!---HONumber=Mooncake_1031_2016-->