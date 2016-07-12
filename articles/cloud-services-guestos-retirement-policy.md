<properties 
   pageTitle="Azure 来宾 OS 可支持性和停用策略指南 | Azure" 
   description="介绍有关 Microsoft 对云服务使用的 Azure 来宾 OS 提供的支持的信息。" 
   services="cloud-services" 
   documentationCenter="na" 
   authors="yuemlu" 
   manager="timlt" 
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="04/19/2016"
   wacn.date="05/31/2016"/>

# Azure 来宾 OS 可支持性和停用策略
本页面上的信息与 Azure 来宾操作系统（[来宾 OS](/documentation/articles/cloud-services-guestos-update-matrix/)）相关。来宾 OS 仅适用于云服务辅助角色和 Web 角色 (PaaS)，而不适用于虚拟机 (IaaS)。

Microsoft 已发布[来宾 OS 的支持策略](http://support.microsoft.com/zh-cn/gp/azure-cloud-lifecycle-faq)。你目前正在阅读的页面描述了如何实施该策略。

该策略规定，

1. Microsoft 支持**至少两个最新的来宾 OS 系列**。在某个系列停用后，客户可以在从正式停用之日起的 12 个月内更新为受支持的较新来宾 OS 系列。
2. Microsoft 支持**至少两个最新的支持来宾 OS 系列版本**。 
3. Microsoft 支持**至少两个最新的 Azure SDK 版本**。在某个 SDK 版本停用后，客户可以在从正式停用之日起的 12 个月内更新为较新的版本。 

有时，可能会支持两个以上的系列或发行版。将在 [Azure 来宾 OS 版本和 SDK 兼容性对照表](/documentation/articles/cloud-services-guestos-update-matrix/)中显示正式的来宾 OS 支持信息。


## 何时停用来宾 OS 系列或版本 


在发布新的正式 Windows Server 操作系统版本后，将在某个时间推出新的来宾 OS **系列**。每次推出新的来宾 OS 系列时，Microsoft 将停用最早的来宾 OS 系列。

大约每个月都会推出新来宾 OS **版本**，以合并最新 MSRC 更新。由于定期每月更新，来宾 OS 版本正常情况下将在其发布的 60 天后禁用。这样，对于每个可供使用的系列，至少都保留两个来宾 OS 版本。

### 来宾 OS 停用的过程 


宣布停用之后，客户在较旧系列正式从服务中移除之前有 12 个月的“过渡”期。过渡时间可能延长，这由 Microsoft 决定。将在 [Azure 来宾 OS 版本和 SDK 兼容性对照表](/documentation/articles/cloud-services-guestos-update-matrix/)中发布更新。

在过渡期开始的 6 个月后，将逐步执行停用过程。在此期间：

1. Microsoft 会通知客户即将发生停用。 
2. 更新的 Azure SDK 版本不支持停用的来宾 OS 系列。
3. 不允许在停用的系列上对云服务进行全新部署和重新部署

Microsoft 将继续推出合并了最新 MSRC 更新的新来宾 OS 版本，直至过渡期的最后一天（称为“到期日期”）。此时，仍在运行的任何云服务都不受 Azure SLA 的支持。在该日期后，Microsoft 有权自行强制要求升级、删除或停止这些服务。



### 来宾 OS 版本停用期间的过程 
如果客户将其来宾 OS 设置为自动更新，则他们不必担心如何处理有关来宾 OS 版本的问题。他们将始终使用最新来宾 OS 版本。

来宾 OS 版本每个月发布一次。由于常规发布的速率，每个版本都具有固定生存期。

60 天使用期后，版本会“停用”。“停用”表示该版本将从 Azure 管理门户中删除。也无法再通过 CSCFG 配置文件进行设置。现有部署仍保持运行，但是不允许进行新部署以及针对现有部署的代码和配置更新。

之后，来宾 OS 版本“到期”，仍在运行该版本的任何安装都会强制升级并设置为在将来自动更新来宾 OS。过期是分批过期的，因此从禁用到过期的时间段可能各不相同。

这些期间可能会延长，这由 Microsoft 决定，以便于客户过渡。将在 [Azure 来宾 OS 版本和 SDK 兼容性对照表](/documentation/articles/cloud-services-guestos-update-matrix/)中通告所有更改。



### 停用期间的通知 

* **系列停用** <br>Microsoft 将在博客文章和 Azure 管理门户中发布通知。将通过与指定的服务管理员进行直接通信（电子邮件、门户消息、电话）以通知仍使用停用的来宾 OS 系列的客户。将在此页面以及此页面开头列出的 RSS 源中发布所有更改。 


* **版本停用** <br>将在此页面以及此页面开头列出的 RSS 源中发布所有更改，包括发布、禁用和到期日期。如果服务管理员有运行在禁用的来宾 OS 版本或系列上的部署，则将收到电子邮件。这些电子邮件的时间可能各不相同。通常，在禁用前至少有一个月的时间，但是此时间安排并不是官方的 SLA。


## 常见问题

**我如何消除迁移的影响？**

应使用最新的来宾 OS 系列设计你的云服务。

1. 及早开始计划迁移到较新的系列。 
2. 设置临时测试部署以测试在新系列上运行的云服务。 
3. **将来宾 OS 版本设置**为“自动”（在 [.cscfg](/documentation/articles/cloud-services-model-and-package/#cscfg) 文件中设置 osVersion=*），以便自动迁移到新的来宾 OS 版本。

**如果我的 Web 应用程序需要更深入地与 OS 集成，我该怎么办？**

如果你的 Web 应用程序体系结构需要更深入地依赖于基本操作系统，请使用平台支持的功能（例如[启动任务](/documentation/articles/cloud-services-startup-tasks/)）或将来可能存在的其他扩展性机制。此外，你还可以使用 [Azure 虚拟机](/documentation/services/virtual-machines)（IaaS – 基础结构即服务），你可以在其中负责维护基本操作系统。
 
## 后续步骤
查看最新的[来宾 OS 版本](/documentation/articles/cloud-services-guestos-update-matrix/)。

<!---HONumber=Mooncake_0523_2016-->
