<properties
   pageTitle="使用 Linux Azure 诊断收集日志 | Azure"
   description="本文介绍如何将 Azure 诊断设置为从 Azure 中运行的 Service Fabric Linux 群集收集日志。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/28/2016"
   wacn.date="11/28/2016"
   ms.author="subramar"/>  



# 使用 Azure 诊断收集日志

> [AZURE.SELECTOR]
- [Windows](/documentation/articles/service-fabric-diagnostics-how-to-setup-wad/)
- [Linux](/documentation/articles/service-fabric-diagnostics-how-to-setup-lad/)

当你运行 Azure Service Fabric 群集时，最好是从一个中心位置的所有节点中收集日志。只要将日志集中在一个位置，无论问题发生在服务、应用程序还是群集本身，都能轻松分析问题并进行故障排除。上载和收集日志的方式之一是使用可将日志上载到 Azure 存储空间的 Azure 诊断扩展。可以从存储中读取事件，并将它们放在其他日志分析解决方案中。

## 可以收集的日志源
- **Service Fabric 日志**：由平台通过 [LTTng](http://lttng.org) 发出，将上载到你的存储帐户。日志可能是平台发出的操作事件或运行时事件。这些日志存储在群集清单指定的位置。（若要获取存储帐户的详细信息，请搜索 **AzureTableWinFabETWQueryable** 标记，然后查看 **StoreConnectionString**）。
- **应用程序事件**：从服务代码发出。可以使用任何能够写入基于文本的日志的日志记录解决方案，例如 LTTng。有关详细信息，请参阅有关跟踪应用程序的 LTTng 文档。


## 部署诊断扩展
收集日志的第一个步骤是将诊断扩展部署在 Service Fabric 群集的每个 VM 上。诊断扩展将收集每个 VM 上的日志，并将它们上载到指定的存储帐户。根据使用的是 Azure 门户还是 Azure Resource Manager，步骤将有所不同。

若要在创建群集期间将诊断扩展部署到群集中的 VM，请将“诊断”设置为“打开”。创建群集后，无法使用门户更改此设置。

然后，配置 Linux Azure 诊断 (LAD) 来收集文件并将其放入你的存储帐户。[Using LAD to monitor and diagnose Linux VMs](/documentation/articles/virtual-machines-linux-classic-diagnostic-extension/)（使用 LAD 监视和诊断 Linux VM）一文中的方案 3（“上载自己的日志文件”）说明了此过程。遵循此过程即可访问跟踪。可以将跟踪上载到所选的可视化程序。

也可以使用 Azure Resource Manager 部署诊断扩展。Windows 和 Linux 上的部署过程类似，[如何使用 Azure 诊断收集日志](/documentation/articles/service-fabric-diagnostics-how-to-setup-wad/)中介绍了适用于 Windows 群集的过程。

也可以按 [Operations Management Suite Log Analytics with Linux](https://blogs.technet.microsoft.com/hybridcloud/2016/01/28/operations-management-suite-log-analytics-with-linux/)（在 Linux 中使用 Operations Management Suite Log Analytics）中所述使用 Operations Management Suite。

完成此配置后，LAD 代理将监视指定的日志文件。每当在文件中追加新行时，该代理将创建一个 syslog 条目并将其发送到指定的存储。


## 后续步骤
若要更详细了解在排查问题时应检查哪些事件，请参阅 [LTTng 文档](http://lttng.org/docs)和 [Using LAD](/documentation/articles/virtual-machines-linux-classic-diagnostic-extension/)（使用 LAD）。

<!---HONumber=Mooncake_1121_2016-->