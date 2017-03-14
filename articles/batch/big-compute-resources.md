<properties
    pageTitle="批处理和 HPC 工作负荷在云中的资源 | Azure"
    description="列出了旨在帮助你在 Azure 中运行大规模并行批处理和高性能计算 (HPC) 工作负荷的技术资源。"
    services="batch, cloud-services, virtual-machines"
    documentationcenter=""
    author="dlepow"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="6f8be911-c841-41ae-88d3-3bcfc029eb7f"
    ms.service="multiple"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="big-compute"
    ms.date="01/23/2017"
    wacn.date="03/14/2017"
    ms.author="danlep" />  


# Azure 中的大型计算：用于批处理和高性能计算的技术资源
此为技术资源指南，旨在帮助用户在 Azure 中运行大规模并行批处理和高性能计算 (HPC) 工作负荷。可以使用各种 Azure 服务将现有的批处理或 HPC 工作负荷扩展到 Azure 云，或者生成新的大型计算解决方案。

## 解决方案选项
了解 Azure 中的大型计算选项，并根据工作负荷和业务需要选择适当的方法。

- [Batch 和 HPC 解决方案](/documentation/articles/batch-hpc-solutions/)

## Azure Batch
[Batch](/home/features/batch/) 是一种平台服务，可让你轻松地在 Linux 和 Windows 应用程序中启用云功能并运行作业，而无需设置和管理群集与作业计划程序。使用 SDK 可将不同语言的客户端应用与 Azure 批处理集成，将数据迁移到 Azure，以及生成作业运行管道。

- [文档](/documentation/services/batch/)
- [.NET](https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx)、Python、[Node.js](http://azure.github.io/azure-sdk-for-node/azure-batch/latest/)、[Java](http://azure.github.io/azure-sdk-for-java/) 和 [REST](https://msdn.microsoft.com/zh-cn/library/azure/dn820158.aspx) API 参考
- [Batch 管理 .NET 库](https://msdn.microsoft.com/zh-cn/library/mt463120.aspx)参考
- 教程：[用于 .NET 的 Azure Batch 库入门](/documentation/articles/batch-dotnet-get-started/)和 [Batch Python 客户端入门](/documentation/articles/batch-python-tutorial/)
- [Batch 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=azurebatch)

## HPC 群集解决方案
将现有的 Windows HPC 群集部署或扩展到 Azure，以运行计算密集型工作负荷。

### Microsoft HPC Pack
HPC Pack 是在 Azure 和 Windows Server 技术基础之上构建的 Microsoft 免费 HPC 解决方案，它能够运行 Windows 和 Linux HPC 工作负荷。

- [下载 HPC Pack 2016](https://www.microsoft.com/zh-cn/download/details.aspx?id=54507)
- [下载 HPC Pack 2012 R2 Update 3](https://www.microsoft.com/zh-cn/download/details.aspx?id=49922)
- [文档](https://technet.microsoft.com/zh-cn/library/jj899572.aspx)
- [使用 HPC Pack 迸发到 Azure 辅助角色实例](https://technet.microsoft.com/zh-cn/library/gg481749.aspx)
- [使用 HPC Pack 迸发到 Azure Batch](https://technet.microsoft.com/zh-cn/library/mt612877.aspx)
- [Windows HPC 论坛](https://social.microsoft.com/Forums/home?category=windowshpc)

### Linux 和 OSS 群集解决方案
使用这些 Azure 模板来部署 Linux HPC 群集。

- [博客文章](http://blogs.technet.com/b/windowshpc/archive/2015/06/06/deploy-a-slurm-cluster-on-azure.aspx)

## Microsoft MPI
[Microsoft MPI](https://msdn.microsoft.com/zh-cn/library/bb524831.aspx) (MS-MPI) 是 Microsoft 实现的消息传递接口标准，用于在 Windows 平台上开发和运行并行应用程序。

- [下载 MS-MPI](http://go.microsoft.com/FWLink/p/?LinkID=389556)
- [MS-MPI 参考](https://msdn.microsoft.com/zh-cn/library/dn473458.aspx)
- [MPI 论坛](https://social.microsoft.com/Forums/zh-cn/home?forum=windowshpcmpi)

## 计算密集型实例
Azure 为后端 RDMA 网络提供 [VM 大小范围](/documentation/articles/virtual-machines-windows-sizes?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json/)，以便运行 Linux 和 Windows HPC 工作负荷。


## 示例和演示
- [Azure Batch C# and Python code samples（Azure Batch C# 和 Python 代码示例）](https://github.com/Azure/azure-batch-samples)
- [批处理 Shipyard](https://azure.github.io/batch-shipyard/) 工具包，可以轻松地将批处理样式的 Dockerized 工作负荷部署到 Azure 批处理
- [Test drive SUSE Linux Enterprise Server for HPC（试用 SUSE Linux Enterprise Server for HPC）](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)

## 相关的 Azure 服务


- [HDInsight](/documentation/services/hdinsight/)
- [虚拟机](/documentation/services/virtual-machines/)
- [虚拟机规模集](/documentation/services/virtual-machine-scale-sets/)
- [云服务](/documentation/services/cloud-services/)
- [App Service](/documentation/services/app-service/)
- [媒体服务](/documentation/services/media-services/)


## 行业解决方案
- [银行和资本市场](https://finance.azure.com/)
- [工程模拟](https://simulation.azure.com/)

## 客户案例
- [ANEO](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=4168)
- [d3View](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=22088)
- [Milliman](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=14967)
- [Mitsubishi UFJ Securities International](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=26266)
- [Schlumberger](http://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [Towers Watson](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=18222)
- [UberCloud](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

## 后续步骤
- 有关最新通告，请参阅 [Microsoft HPC 和批处理团队博客](http://blogs.technet.com/b/windowshpc/)与 [Azure 博客](https://azure.microsoft.com/blog/tag/hpc/)。
- 另请参阅 [Batch 中的新增功能](https://azure.microsoft.com/updates/?service=batch)或订阅 [RSS 源](https://azure.microsoft.com/updates/feed/?service=batch)。

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->