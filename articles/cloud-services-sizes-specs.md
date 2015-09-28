<properties 
 pageTitle="云服务的大小" 
 description="列出 Azure 云服务 Web 角色和辅助角色的不同大小。" 
 services="cloud-services" 
 documentationCenter="" 
 authors="Thraka" 
 manager="timlt" 
 editor=""/>
<tags 
 ms.service="cloud-services" 
 ms.date="06/04/2015" 
 wacn.date="09/15/2015"/>
 
# 云服务的大小

本主题介绍云服务角色实例（Web 角色和辅助角色）的可用大小与选项。此外，还提供了在计划使用这些资源时要考虑的部署注意事项。

Azure 提供了多种类型的计算资源，其中的两种类型是 Azure 虚拟机和云服务。有关说明，请参阅[计算 Azure 提供的托管选项](/documentation/articles/fundamentals-application-models)。

> [AZURE.NOTE]若要查看相关的 Azure 限制，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits)

## Web 角色实例和辅助角色实例的大小

以下注意事项可能会帮助你决定大小：

* 现在，可将实例配置为使用 D 系列 VM。这些 VM 旨在运行需要更高计算能力和临时磁盘性能的应用程序。D 系列 VM 为临时磁盘提供更快的处理器、更高的内存内核比和固态驱动器 (SSD)。有关详细信息，请参阅 Azure 博客[新的 D 系列虚拟机大小](http://azure.microsoft.com/blog/2014/09/22/new-d-series-virtual-machine-sizes/)上的公告。  

* 由于系统要求，Web 角色和辅助角色需要比 Azure 虚拟机更多的临时磁盘空间。系统文件为 Windows 页面文件保留 4 GB 的空间，并为 Windows 转储文件保留 2 GB 的空间。

* 操作系统磁盘包含 Windows 来宾操作系统并包含 Program Files 文件夹（除非你指定另一个磁盘，否则包括通过启动任务完成的安装）、注册表更改、System32 文件夹和 .NET Framework。

* 本地资源磁盘包含 Azure 日志和配置文件、Azure 诊断（包括 IIS 日志）和你定义的任何本地存储资源。

* 应用（应用程序）磁盘是提取你的 .cspkg 的位置，它包含你的网站、二进制文件、角色主机进程、启动任务、web.config，等等。

* A8/A10 和 A9/A11 虚拟机大小具有相同的容量。A8 和 A9 虚拟机实例包括一个连接到远程直接内存访问 (RDMA) 网络的附加网络适配器，从而实现两台虚拟机之间的快速通信。A8 和 A9 实例专门用于在执行期间需要在两个节点之间保持恒定和低延迟通信的高性能计算应用程序，例如使用消息传递接口 (MPI) 的应用程序。A10 和 A11 虚拟机实例不包含其他网络适配器。A10 和 A11 实例专门用于不需要在两个节点之间保持恒定和低延迟通信的高性能计算应用程序，也称为参数化或高度并行应用程序。

|大小|CPU<br>核心数|内存|磁盘大小|
|---|---|---|---|
|特小型|1|768 MB|OS = 来宾操作系统大小<br/>本地资源 = 19 GB<br/>应用 = 约 1.5 GB|
|小型|1|1.75 GB|OS = 来宾操作系统大小<br/>本地资源 = 224 GB<br/>应用 = 约 1.5 GB|
|中型|2|3.5 GB|OS = 来宾操作系统大小<br/>本地资源 = 489 GB<br/>应用 = 约 1.5 GB|
|大型|4|7 GB|OS = 来宾操作系统大小<br/>本地资源 = 999 GB<br/>应用 = 约 1.5 GB|
|超大型|8|14 GB|OS = 来宾操作系统大小<br/>本地资源 = 2,039 GB<br/>应用 = 约 1.5 GB|
|A5|2|14 GB|OS = 来宾操作系统大小<br/>本地资源 = 489 GB<br/>应用 = 约 1.5 GB|
|A6|4|28 GB|OS = 来宾操作系统大小<br/>本地资源 = 999 GB<br/>应用 = 约 1.5 GB|
|A7|8|56 GB|OS = 来宾操作系统大小<br/>本地资源 = 2,039 GB<br/>应用 = 约 1.5 GB
|A8|8|56 GB|OS = 来宾操作系统大小<br/>本地资源 = 1.77 TB<br/>应用 = 约 1.5 GB<!--blockquote> Note: For information and considerations about using this size, see <a href="http://go.microsoft.com/fwlink/p/?linkid=328042">About the A8, A9, A10, and A11 Compute Intensive Instances</a>.</blockquote-->|
|A9|16|112 GB|OS = 来宾操作系统大小<br/>本地资源 = 1.77 TB<br/>应用 = 约 1.5 GB<!--blockquote> Note: For information and considerations about using this size, see <a href="http://go.microsoft.com/fwlink/p/?linkid=328042">About the A8, A9, A10, and A11 Compute Intensive Instances</a>.</blockquote-->|
|A10|8|56 GB|OS = 来宾操作系统大小<br/>本地资源 = 1.77 TB<br/>应用 = 约 1.5 GB<!--blockquote> Note: For information and considerations about using this size, see <a href="http://go.microsoft.com/fwlink/p/?linkid=328042">About the A8, A9, A10, and A11 Compute Intensive Instances</a>.</blockquote-->|
|A11|16|112 GB|OS = 来宾操作系统大小<br/>本地资源 = 1.77 TB<br/>应用 = 约 1.5 GB<!--blockquote> Note: For information and considerations about using this size, see <a href="http://go.microsoft.com/fwlink/p/?linkid=328042">About the A8, A9, A10, and A11 Compute Intensive Instances</a>.</blockquote-->|
|Standard_D1|1|3.5 GB|OS = 来宾操作系统大小<br/>本地资源 = 50 GB<br/>应用 = 约 1.5 GB|
|Standard_D2|2|7 GB|OS = 来宾操作系统大小<br/>本地资源 = 100 GB<br/>应用 = 约 1.5 GB|
|Standard_D3|4|14 GB|OS = 来宾操作系统大小<br/>本地资源 = 200 GB<br/>应用 = 约 1.5 GB|
|Standard_D4|8|28 GB|OS = 来宾操作系统大小<br/>本地资源 = 400 GB<br/>应用 = 约 1.5 GB|
|Standard_D11|2|14 GB|OS = 来宾操作系统大小<br/>本地资源 = 100 GB<br/>应用 = 约 1.5 GB|
|Standard_D12|4|28 GB|OS = 来宾操作系统大小<br/>本地资源 = 200 GB<br/>应用 = 约 1.5 GB|
|Standard_D13|8|56 GB|OS = 来宾操作系统大小<br/>本地资源 = 400 GB<br/>应用 = 约 1.5 GB|
|Standard_D14|16|112 GB|OS = 来宾操作系统大小<br/>本地资源 = 800 GB<br/>应用 = 约 1.5 GB|

## 后续步骤

[为 Azure 设置云服务](https://msdn.microsoft.com/zh-cn/library/hh124108)[配置云服务大小](https://msdn.microsoft.com/zh-cn/library/ee814754)

<!---HONumber=69-->