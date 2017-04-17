<properties
    pageTitle="在 Azure 门户中创建服务总线命名空间 | Azure"
    description="如何使用 Azure 门户创建服务总线命名空间。"
    services="service-bus"
    documentationCenter=".net"
    authors="jtaubensee"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="service-bus"
    ms.devlang="tbd"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="11/30/2016"
    wacn.date="04/17/2017"
    ms.author="jotaub"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="d691ed3aa3885bf2c0a538dba120d8b5bf339e47"
    ms.lasthandoff="04/07/2017" />

# <a name="create-a-service-bus-namespace-using-the-azure-portal"></a>使用 Azure 门户创建服务总线命名空间。
命名空间是一个适用于所有消息传送组件的公用容器。 多个队列和主题可以位于一个命名空间中，命名空间通常用作应用程序容器。 目前有两种不同方法可用来创建服务总线命名空间。

1. Azure 门户（这篇文章）

2. [Resource Manager 模板][create-namespace-using-arm]

## <a name="create-a-namespace-in-the-azure-portal"></a>在 Azure 门户中创建命名空间

[AZURE.INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

祝贺你！ 现在已创建一个服务总线消息传送命名空间。

## <a name="next-steps"></a>后续步骤

签出 [GitHub 示例][github-samples]，它们演示了 Azure 服务总线消息传送中的某些更高级的功能。

[create-namespace-using-arm]: /documentation/articles/service-bus-resource-manager-overview/
[github-samples]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples


<!--Update_Description:update wording -->