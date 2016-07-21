<properties
	pageTitle="Azure 仪表板共享和访问控制 | Azure"
	description="了解如何管理 Azure 仪表板的共享和访问控制。"
	services="azure-resource-manager,azure-portal"
    	keywords="门户,共享,访问"
	documentationCenter=""
	authors="adamabdelhamed"
	manager="dend"
	editor="dend"/>

<tags
	ms.service="azure-resource-manager"
	ms.date="03/28/2016"
	wacn.date="07/18/2016"/>
    
# Azure 仪表板共享和访问控制

对门户中大多数磁贴所显示信息的访问均由 Azure [基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)管理。为了将仪表板无缝集成到生态系统中，所有已发布的仪表板均应以 Azure 资源的形式实现。从访问控制角度来看，仪表板与虚拟机或存储帐户没有什么不同。

下面是一个示例。假设你有一个 Azure 订阅，并且你团队中的各个成员都分配了订阅的**所有者**、**参与者**或**读取者**角色。作为所有者或参与者的用户将能够列出、查看、创建、修改或删除订阅中的仪表板。作为读者的用户将能够列出并查看仪表板，但不能修改或删除它们。具有读者访问权限的用户将能够对已发布的仪表板进行本地编辑（例如排查问题时），但不能将这些更改发布回服务器。他们可以为自己制作仪表板的私有副本。

请注意，仪表板中的各个磁贴将依据显示其数据的资源强制执行自己的访问控制要求。这就意味着，你可以设计一个仪表板，该仪表板可供更广泛地共享，同时保护各个磁贴上的数据。

<!---HONumber=Mooncake_0711_2016-->