<properties
	pageTitle="将应用程序部署到 Azure 应用服务"
	description="了解如何将应用程序部署到应用服务工作"
	keywords="应用服务, azure 应用服务, 正在部署, 部署"
	services="app-service"
	documentationCenter=""
	authors="dariagrigoriu"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service"
	ms.date="02/09/2016"
	wacn.date=""/>

# Azure 应用服务部署概述

Azure 应用服务提供了一组丰富的集成功能，支持创建功能强大且灵活的部署工作流。应用部署可以利用的选项包括持续集成或本地源代码管理发布、WebDeploy 和 FTP。进行生产应用部署时，建议使用部署槽交换方法。部署槽代表与生产应用关联的过渡和集成环境。可以对部署槽进行配置，将 Web 流量定向到其中进行验证，并可根据需要来交换流量，在不停机和自动预热的情况下部署到生产环境。可以通过发布管理产品（例如 Visual Studio Release Management）轻松实现部署工作流步骤的自动化。这适用于与其他解决方案资源（例如数据存储）的协调操作、循环以及跨多个部署单元的复制。

[AZURE.INCLUDE [app-service-blueprint-deployment](../../includes/app-service-blueprint-deployment.md)]

<!---HONumber=Mooncake_0919_2016-->