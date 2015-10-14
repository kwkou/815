<properties 
	pageTitle="Azure Government 开发人员指南" 
	description="这里提供了针对 Azure Government 的应用程序开发使用到的功能和指南对比" 
	services="" 
	documentationCenter="" 
	authors="Joharve2" 
	manager="carolz" 
	editor=""/>

<tags 
	ms.service="multiple" 
	ms.date="01/21/2014" 
	wa.date="10/3/2015"/>


#  Microsoft Azure Government 开发人员指南 

<p>Microsoft Azure 政府版是物理和网络方面均独立的 Microsoft Azure 实例。  
此开发人员指南将为应用程序开发人员和管理员可能需要进行交互并处理 Azure 的这些单独区域这一差异提供详细信息。

<!--Table of contents for topic, the words in brackets must match the heading wording exactly-->


## 本主题内容


+ [概述](#Overview)
+ [开发人员指南](#Guidance)
+ [Microsoft Azure 政府版中的当前可用功能](#Features)
+ [终结点映射](#Endpoint)
+ [后续步骤](#next)


## <a name="Overview"></a>概述

Microsoft Azure Government 是 Microsoft Azure 服务的单独实例，满足了美国联邦机构、州和地方政府，以及他们解决方案供应商的安全和合规需求。Azure Government 提供来自非美国政府部署的物理和网络隔离，并提供筛选过的美国人员。

Microsoft 提供多种工具来创建和部署云应用程序到 Microsoft 的全局 Microsoft Azure 服务（简称“全局服务”）和 Microsoft Azure Government 服务。

当创建和部署应用程序到 Azure Government 服务，而不是全局服务时，开发人员需要了解这两个服务的主要区别。特别是围绕设置和配置其编程环境、配置终结点、编写应用程序以及将它们作为服务部署到 Azure Government。

本文档中的信息总结了这些区别，并补充了 [Azure Government](http://www.azure.com/gov "Azure Government") 站点和 MSDN 上 [Microsoft Azure 技术库](http://msdn.microsoft.com/cloud-app-development-msdn "MSDN")的可用信息。官方信息也可在许多其他位置提供，如 [Microsoft Azure 信任中心](http://www.windowsazure.cn/support/trust-center/ "Microsoft Azure 信任中心")、[Azure 文档中心](http://www.windowsazure.cn/documentation/)和 [Azure 博客](http://www.windowsazure.cn/blog/ "Azure 博客")。

此内容适用于部署到 Microsoft Azure Government 的合作伙伴和开发人员。



## <a name="Guidance"></a>开发人员指南
由于大部分目前可用的技术内容假定应用程序正在为全局服务而开发，而不是 Microsoft Azure Government，务必要确保开发人员意识到开发应用程序并将其托管在 Azure Government 中的主要区别。

- 首先，存在服务和功能上的差异，这意味着全局服务的特定区域中的某些功能可能在 Azure Government 中不可用。

- 其次，Azure Government 中提供的功能，与全局服务的配置不同。因此，应该检查您的示例代码、配置和步骤，以确保您是在 Azure Government 云服务环境中构建并执行。


## <a name="Features"></a>Microsoft Azure 政府版中的当前可用功能
目前，Azure 政府版在 US GOV IOWA 和 US GOV VIRGINIA 区域提供以下服务：

- 虚拟机
- 云服务
- 存储
- Active Directory
- 计划程序
- 虚拟网络
- SQL 数据库

Azure 政府版还提供其他服务，并将继续添加更多服务。有关服务的最新列表，请参阅[“区域”页](http://www.windowsazure.cn/regions/#services)，其中突出显示了每个可用区域及其服务。  


目前，US GOV IOWA 和 US GOV VIRGINIA 是支持 Azure 政府版的数据中心。有关当前可用的数据中心和服务，请参阅上述“区域”页。

## <a name="Endpoint"></a>终结点映射

当映射公共 Microsoft Azure 和 SQL 数据库终结点到 Azure Government 特定终结点时，请参考下表。


服务类型|Azure Public|Azure Government
---|---|---
Azure Government 主页|windowsazure.cn|microsoftazure.us
管理门户|manage.windowsazure.cn|manage.windowsazure.us
常规|*.chinacloudapi.cn|*.usgovcloudapi.net
核心|*.core.chinacloudapi.cn|*.core.usgovcloudapi.net
计算|*.chinacloudapp.cn|*.usgovchinacloudapp.cn
Blob 存储|*.blob.core.chinacloudapi.cn| *.blob.core.usgovcloudapi.net 
Queue Storage|*.queue.core.chinacloudapi.cn|*.queue.core.usgovcloudapi.net
表存储|*.table.core.chinacloudapi.cn|*.table.core.usgovcloudapi.net
服务管理|management.core.chinacloudapi.cn|management.core.usgovcloudapi.net
SQL 数据库|*.database.chinacloudapi.cn|*.database.usgovcloudapi.net

## <a name="next"></a>后续步骤
如果有兴趣了解更多关于 Azure Government 的信息，以及您的组织如何取得访问的资格，请访问 <A href="http://azure.com/gov">http://www.azure.com/gov</a>

<!--Anchors-->



<!-- Images. -->

[1]: ./media/azure-government-developer-guide/publisherguide.png


<!--Link references-->
[Link 1 to another www.windowsazure.cn documentation topic]: /documentation/articles/virtual-machines-windows-tutorial
[Link 2 to another www.windowsazure.cn documentation topic]: /documentation/articles/web-sites-custom-domain-name
[Link 3 to another www.windowsazure.cn documentation topic]: /documentation/articles/storage-whatis-account

<!---HONumber=71-->