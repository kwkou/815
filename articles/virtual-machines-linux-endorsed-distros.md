<properties
	pageTitle="Linux 的认可分发 | Azure"
	description="了解 Azure 认可的分发中的 Linux，包括 Ubuntu、OpenLogic、Oracle 和 SUSE 的指南。"
	services="virtual-machines"
	documentationCenter=""
	authors="szarkos"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management,azure-resource-manager"
	/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="03/25/2016"
	wacn.date="05/24/2016"/>



#Azure 认可的分发中的 Linux

Azure 库中的 Linux 映像由很多合作伙伴提供，并且我们正在与各个 Linux 社区合作，以便向“认可的分发”列表添加更多风格。在此期间，对于该库未提供的分发，你始终可以按照[本页](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)中的指南自备 Linux。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]


## 支持的分发和版本 ##

下表列出了 Azure 支持的 Linux 分发和版本。

Hyper-V 和 Azure 的 Linux 集成服务 (LIS) 驱动程序是 Microsoft 直接为上游 Linux 内核提供的内核模块。LIS 驱动程序在默认情况下内置于分发的内核中，或者作为较旧的基于 RHEL/CentOS 的分发在[此处](http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409)作为单独的下载提供。有关 LIS 驱动程序的详细信息，请参阅[此文](/documentation/articles/virtual-machines-linux-create-upload-generic/#linux-kernel-requirements)。

Azure Linux 代理已预安装在 Azure 库映像中，并通常可从分发的包存储库中获得。源代码可在 [GitHub](https://github.com/azure/walinuxagent) 上找到。

分发|版本|驱动程序|代理
---|---|---|---
CentOS by OpenLogic |CentOS 6.3+、7.0+| CentOS 6.3：[LIS 下载](http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409)<p>CentOS 6.4+：在内核中|包：在“WALinuxAgent”下的 <a href="http://olcentgbl.trafficmanager.net/openlogic/6/openlogic/x86_64/RPMS/">OpenLogic 存储库中<p><p>源代码：[GitHub](https://github.com/Azure/WALinuxAgent)
由 Credativ 提供的 Debian |Debian 7.9+、8.2+|在内核中|包：在“waagent”下的存储库中<p><p>源代码：[GitHub](https://github.com/Azure/WALinuxAgent)
SUSE Linux Enterprise |SLES 11 SP3+、SLES 12+ 和 <p><p> SLES for SAP 11.3+ |在内核中|包：在“WALinuxAgent”下的 [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools) 存储库中<p><p>源代码：[GitHub](http://go.microsoft.com/fwlink/p/?LinkID=250998)
openSUSE |openSUSE 13.1+|在内核中|包：在“WALinuxAgent”下的 [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools) 存储库中<p><p>源代码：[GitHub](https://github.com/Azure/WALinuxAgent)
Ubuntu|Ubuntu 12.04、14.04、15.10 和 16.04|在内核中|包：在“walinuxagent”下的存储库中<p><p>源代码：[GitHub](https://github.com/Azure/WALinuxAgent)
## 合作伙伴

### OpenLogic
[http://www.openlogic.com/azure](http://www.openlogic.com/azure)

OpenLogic 是针对云和数据中心的企业开放源解决方案的行业领先的提供商。OpenLogic 帮助各个行业数以百计的领先企业安全获取、支持和控制开源软件。OpenLogic 为 OpenLogic 专家社区支持的 600 个开放源包提供商业级技术支持和保护（包括针对 CentOS 的企业级支持），同时作为在 Azure 上提供基于 CentOS 的映像的产品发布合作伙伴。

### Credativ
[http://www.credativ.co.uk/credativ-blog/debian-images-microsoft-azure](http://www.credativ.co.uk/credativ-blog/debian-images-microsoft-azure)

Credativ 是一家独立的咨询和服务公司，致力于通过免费软件开发和实施专业解决方案。Credativ 是获得国际认可的开源领域专业先行者，为许多公司的 IT 部门提供支持。Credativ 与 Microsoft 合作，目前正在为 Debian 8 (Jessie) 以及 Debian 7 (Wheezy) 之前的版本准备相应的 Debian 映像。这些映像经过专门的设计，可以在 Azure 上运行并可通过该平台轻松进行管理。Credativ 还会通过其开源支持中心为 Azure 的 Debian 映像的维护和更新提供长期支持。

### SUSE
[http://www.suse.com/suse-linux-enterprise-server-on-azure](http://www.suse.com/suse-linux-enterprise-server-on-azure)

Azure 上的 SUSE Linux Enterprise Server 是一个已验证的平台，该平台为云计算提供了高级可靠性和安全性。SUSE 的通用 Linux 平台可与 Azure 云服务无缝集成，以便交付易于管理的云环境。借助 1,800 多个独立软件供应商提供的适用于 SUSE Linux Enterprise Server 的 9,200 多个认证应用程序，SUSE 可确保满怀信心地在 Azure 上部署数据中心内支持的运行负载。

### Canonical
[http://www.ubuntu.com/cloud/azure](http://www.ubuntu.com/cloud/azure)

Canonical 工程和开放社区监管对 Ubuntu 在客户端、服务器和云计算（包括用户的个人云服务）方面获得成功起到了推动作用。Canonical 期望使用 Ubuntu 开发一个统一的免费平台（从手机到云），该平台带有一系列适用于手机、平板电脑、TV 和桌面的相关接口，从而使 Ubuntu 成为各种机构（从公有云提供商到消费类电子产品制造商）的首选以及各个技术专家的最爱。

借助其遍布全球的开发人员和工程中心，Canonical 在与硬件制造商、内容提供商和软件开发人员合作以将 Ubuntu 解决方案推向市场（从 PC 到服务器和手持设备）方面拥有独特的优势。

<!---HONumber=Mooncake_0118_2016-->