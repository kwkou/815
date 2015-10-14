<properties 
   pageTitle="Azure 政府版概述" 
   description="本文概述了 Azure 政府版云功能以及可靠的设计和安全性，其可满足美国联邦、州和地方政府组织及其合作伙伴的合规性要求。" 
   services="Azure-Government" 
   documentationCenter="" 
   authors="joharve2" 
   manager="chrisnie" 
   editor=""/>

<tags
   ms.service="multiple"
   ms.date="07/22/2015"
   wa.date="10/3/2015"/>

#  Microsoft Azure 政府版概述 

<p>Microsoft Azure 政府版是物理和网络方面均独立的 Microsoft Azure 实例。本文概述了环境和服务，并提供了其他信息的链接。


## <a name="Overview"></a>概述

Azure 政府版是政府社区云 (GCC)，旨在为对速度、规模、安全性、合规性和经济性均有要求的美国政府组织战略政府方案提供支持。在其他 Microsoft 云产品/服务（如 Azure 公共、Office 365、O365 GCC、Microsoft CRM Online 等）中，Microsoft 对提供软件、安全性、合规性和控制已经有了非常丰富的经验，而 Azure 政府版正是基于此经验开发而成。

此外，Azure 政府版旨在满足机密的专用美国公共部门工作负荷对安全性和合规性的更高级要求。这些工作负荷来自于法律法规，如美国联邦风险和授权管理计划 (FedRAMP)、美国国防部企业云服务中转站 (ECSB)、刑事司法信息系统 (CJIS)、安全政策和健康保险可移植性和责任法案 (HIPAA)。

下面概述了 Azure 政府版云基础架构、结构、服务和框架，有助于政府组织构建混合云解决方案来实现自己的目标。由于下面的一些服务仅处于预览阶段，因此，请参阅[“区域”页](http://www.windowsazure.cn/regions/#services)，其中列出了已全面推出的最新服务。

![][2]

Azure 政府版包括服务架构 (IaaS) 和平台即服务 (PaaS) 的核心组件。这包括基础架构、网络、存储、数据管理、身份管理和其他许多服务。Azure 政府版支持公共 Azure 客户利用的相同强大功能，如地区同步数据复制和自动调整。业界领先的行业分析师已将 Microsoft 确认为 <a href="https://www.gartner.com/doc/2575715/magic-quadrant-cloud-infrastructure-service" target="_new">IaaS</a> 和 <a href="https://www.gartner.com/doc/2645317/magic-quadrant-enterprise-application-platform" target="_new">PaaS<a/></a> 领导者。

除了提供公共 Azure 的可靠服务和功能之外，Azure 政府版还提供了很多功能来确保美国政府实体的数据安全性，具体是通过提供：

- **物理和网络方面均独立的实例** - Azure 政府版环境是完全独立于 Microsoft Azure 公共的实例，仅供限定的美国政府组织和解决方案提供商使用。

- **安全性、隐私和合规性** - Microsoft 已实现可靠的安全性、隐私和合规性控制框架，以及其他严格的控制，以满足更高级的 ECSB 影响级别和 CJIS 要求。

- **数据存储** - Azure 政府版环境负责维护相距超过 500 英里的 2 个数据中心。客户托管的所有数据都存储在美国本土 (CONUS) 数据中心

- **美国人员** - 所有 Azure 政府版操作人员和管理员都是经过筛选的美国公民。

- **身份管理** - Azure 政府版环境中的身份管理是独立的 Azure Active Directory 实例。

- **合规性** - Microsoft 持续投资以满足和维护严格且不断变化的联邦、州和地方法规的合规性要求，如面向美国政府云解决方案的 FedRAMP、CJIS、ECSB 和 HIPAA。

- **云集成** - Azure 政府版提供与 O365 政府版集成的环境，以便您可以跨云服务和增强服务（如 1 TB 的 OneDrive 存储空间）单一登录。

借助 Azure 政府版，组织还能保持其现有技术投资，并享受云服务的好处。由于 Azure 政府版是一个交互云平台，因此组织可以通过产品和技术从头开始构建更加开放的应用程序。机构可以为云解决方案选择工具、服务、操作系统、体系结构和框架，包括 Windows、Linux、Oracle、SharePoint、.NET、Java、PHP 和 Node.js。Azure 政府版平台十分灵活，可方便您以新的形式跨机构展开协作、开发应用程序和集成。

希望使用云服务的美国政府组织可以确信的是，Azure 政府版能严格执行大规模安全操作，满足其不断发展变化的需求。







## <a name="Features"></a>Microsoft Azure 政府版中的当前可用功能
目前，Azure 政府版在 US GOV IOWA 和 US GOV VIRGINIA 区域提供以下服务：

- 虚拟机
- 云服务
- 存储
- Active Directory
- 计划程序
- 虚拟网络
- SQL 数据库
- Azure 文件
- 媒体服务
- 流量管理器
- 服务总线
- StorSimple

Azure 政府版还提供其他服务，并将继续添加更多服务。有关服务的最新列表，请参阅[“区域”页](http://www.windowsazure.cn/regions/#services)，其中突出显示了每个可用区域及其服务。

目前，US GOV IOWA 和 US GOV VIRGINIA 是支持 Azure 政府版的数据中心。有关当前可用的数据中心和服务，请参阅上述“区域”页。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged -->

## <a name="next"></a>后续步骤

如果您想详细了解 Azure 政府版，请单击下面的一些链接。

- **<A href="http://azure.com/gov">获取和使用 Azure 政府版</a>**

- **<A href="/azure-government-developer-guide">Azure 政府版开发人员指南</a>**

<!--- **<A href="/azure-government-service-description">Azure Government Service Descriptions</a>**-->




<!-- Images. -->

[1]: ./media/azure-government-developer-guide/publisherguide.png
[2]: ./media/azure-government-overview/azure-gov-overview.jpg

<!--Link references-->
[Link 1 to another www.windowsazure.cn documentation topic]: /documentation/articles/virtual-machines-windows-tutorial
[Link 2 to another www.windowsazure.cn documentation topic]: /documentation/articles/web-sites-custom-domain-name
[Link 3 to another www.windowsazure.cn documentation topic]: /documentation/articles/storage-whatis-account

<!---HONumber=71-->