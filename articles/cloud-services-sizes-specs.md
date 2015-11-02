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
 ms.date="09/14/2015"
 wacn.date="10/17/2015"/>
 
# 云服务的大小

本主题介绍云服务角色实例（Web 角色和辅助角色）的可用大小与选项。此外，还提供了在计划使用这些资源时要考虑的部署注意事项。

Azure 提供了多种类型的计算资源，其中的两种类型是 Azure 虚拟机和云服务。有关说明，请参阅 [Azure 提供的计算托管选项](/documentation/articles/fundamentals-application-models)。

> [AZURE.NOTE]若要查看相关的 Azure 限制，请参阅 <!--[-->Azure 订阅和服务限制、配额与约束<!--](/documentation/articles/azure-subscription-service-limits)-->

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
|特小型|1|768 MB|OS = 来宾 OS 大小<br/>本地资源 = 15384 MB<br/>应用 = 约 1.5 GB|
|小型|1|1\.75 GB|OS = 来宾 OS 大小<br/>本地资源 = 225304 MB<br/>应用 = 约 1.5 GB|
|中型|2|3\.5 GB|OS = 来宾 OS 大小<br/>本地资源 = 496664 MB<br/>应用 = 约 1.5 GB|
|大型|4|7 GB|OS = 来宾 OS 大小<br/>本地资源 = 1018904 MB<br/>应用 = 约 1.5 GB|
|超大型|8|14 GB|OS = 来宾 OS 大小<br/>本地资源 = 2083864 MB<br/>应用 = 约 1.5 GB|
|A5|2|14 GB|OS = 来宾 OS 大小<br/>本地资源 = 496664 MB<br/>应用 = 约 1.5 GB|
|A6|4|28 GB|OS = 来宾 OS 大小<br/>本地资源 = 1018904 MB<br/>应用 = 约 1.5 GB|
|A7|8|56 GB|OS = 来宾 OS 大小<br/>本地资源 = 2083864 MB<br/>应用 = 约 1.5 GB
|A8|8|56 GB|OS = 来宾 OS 大小<br/>本地资源 = 1856172 MB<br/>应用 = 约 1.5 GB<blockquote> 注意：有关使用此大小的信息和注意事项，请参阅<a href="http://go.microsoft.com/fwlink/p/?linkid=328042">关于 A8、A9、A10 和 A11 计算密集型实例</a>。</blockquote>|
|A9|16|112 GB|OS = 来宾 OS 大小<br/>本地资源 = 1856172 MB<br/>应用 = 约 1.5 GB<blockquote> 注意：有关使用此大小的信息和注意事项，请参阅<a href="http://go.microsoft.com/fwlink/p/?linkid=328042">关于 A8、A9、A10 和 A11 计算密集型实例</a>。</blockquote>|
|A10|8|56 GB|OS = 来宾 OS 大小<br/>本地资源 = 1856172 MB<br/>应用 = 约 1.5 GB<blockquote> 注意：有关使用此大小的信息和注意事项，请参阅<a href="http://go.microsoft.com/fwlink/p/?linkid=328042">关于 A8、A9、A10 和 A11 计算密集型实例</a>。</blockquote>|
|A11|16|112 GB|OS = 来宾 OS 大小<br/>本地资源 = 1856172 MB<br/>应用 = 约 1.5 GB<blockquote> 注意：有关使用此大小的信息和注意事项，请参阅<a href="http://go.microsoft.com/fwlink/p/?linkid=328042">关于 A8、A9、A10 和 A11 计算密集型实例</a>。</blockquote>|
|Standard\_D1|1|3\.5 GB|OS = 来宾 OS 大小<br/>本地资源 = 46104 MB<br/>应用 = 约 1.5 GB|
|Standard\_D2|2|7 GB|OS = 来宾 OS 大小<br/>本地资源 = 97304 MB<br/>应用 = 约 1.5 GB|
|Standard\_D3|4|14 GB|OS = 来宾 OS 大小<br/>本地资源 = 199704 MB<br/>应用 = 约 1.5 GB|
|Standard\_D4|8|28 GB|OS = 来宾 OS 大小<br/>本地资源 = 404504 MB<br/>应用 = 约 1.5 GB|
|Standard\_D11|2|14 GB|OS = 来宾 OS 大小<br/>本地资源 = 97304 MB<br/>应用 = 约 1.5 GB|
|Standard\_D12|4|28 GB|OS = 来宾 OS 大小<br/>本地资源 = 199704 MB<br/>应用 = 约 1.5 GB|
|Standard\_D13|8|56 GB|OS = 来宾 OS 大小<br/>本地资源 = 404504 MB<br/>应用 = 约 1.5 GB|
|Standard\_D14|16|112 GB|OS = 来宾 OS 大小<br/>本地资源 = 814104 MB<br/>应用 = 约 1.5 GB|

## 配置云服务的大小

你可以指定角色实例的虚拟机大小作为服务定义文件描述的服务模型的一部分。角色大小确定了 CPU 核心数目、内存容量，以及分配给正在运行的实例的本地文件系统大小。根据应用程序的资源要求选择角色大小。

下面是一个将 Web 角色实例的角色大小设置为“小”的示例：


    <WebRole name="WebRole1" vmsize="Small">
    ¡­
    </WebRole>

请确保指定的本地资源大小小于或等于上表中的最大本地资源大小。
## 后续步骤

[为 Azure 设置云服务](https://msdn.microsoft.com/zh-cn/library/hh124108)

<!---HONumber=74-->