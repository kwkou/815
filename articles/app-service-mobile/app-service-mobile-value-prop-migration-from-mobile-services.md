<properties
    pageTitle="使用移动服务时，应用服务有何作用？"
    description="了解应用服务为现有的移动服务项目带来的优势。"
    services="app-service\mobile"
    documentationCenter="ios"
    authors="adrianhall"
    manager="dwrede"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-multiple"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="10/01/2016"
    wacn.date="04/24/2017"
    ms.author="adrianha"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="cccf8aaaaef7c83ef25667e801ed00037e71093e"
    ms.lasthandoff="04/14/2017" />

# <a name="getting-started"> </a>使用移动服务时，应用服务有何作用？

##<a name="overview"></a>概述
现有的移动服务很安全，将继续受到支持。 但是， *Azure 应用服务* 平台为移动应用提供了移动服务当前所不具备的多项优点：

- 针对包含 Web 和移动客户端的应用提供更简单且更符合成本效益的功能
- 新主机功能包括 Web 作业、自定义 CName 和更完善的监视
- 现成地与流量管理器集成
- 除了混合连接以外，还可使用 VPN 和 VNet 连接到本地资源
- 使用 NewRelic 或 AppInsights 对应用进行监视、发出警报和故障排除
- 更广泛的基础计算资源与定价层
- 内置自动缩放、负载均衡和性能监视。
- 内置过渡、备份、回滚和生产测试功能

## <a name="new-hosting-features"></a>新托管功能
在 *Azure 应用服务*中，*移动应用*后端代码与 Web 应用和 API 应用在相同的容器中运行。 因此，可以使用此容器中的所有功能，包括移动服务中目前不具备的功能：

- 通过 Web 作业添加连续运行的后端逻辑
- 确保后端代码始终运行
- 使用自定义 CName 为移动设备后端终结点提供友好、稳定的名称
- 使用流量管理器异地缩放应用
- 包含任何所需的库和包。
- （适用于 .NET）利用 ASP.NET 的任何功能，包括 MVC
- （适用于 Node.js）利用 Node 生态系统的任何纯 JavaScript 库，包括常用的 MVC 库。

## <a name="access-on-premises-data-using-vnet"></a>使用 VNet 访问本地数据
有了移动服务，便可以使用混合连接访问本地资源。 但是，在某些情况下，VPN 解决方案更合适。 通过 *Azure 应用服务* ，可以将 Azure VNet 用于移动应用后端代码。

##<a name="use-your-favorite-backend-language"></a>使用偏好的后端语言
*Azure 应用服务* 提供对 ASP.NET 与 Node.js 平台更广泛、更丰富的支持，包括对最新运行时的访问。

##<a name="set-up-automatic-scale"></a>设置自动缩放
使用移动服务时，后端代码的所有实例都在小型 VM 上运行。 *Azure 应用服务* 允许从更丰富的选项中选择 VM 的大小。 还可以根据各种性能指标快速纵向或横向扩展，处理任何传入的客户负载。

##<a name="be-in-the-know"></a>知晓一切
通过监视和警报自动通知用户和用户的团队，实时响应问题。根据来自 New Relic 的高级应用分析和监视功能，更全面地了解移动应用的执行方式。使用 *Azure 应用服务* ，可以根据多种性能指标，以编程方式或通过 Azure 门户预览设置警报。

##<a name="keep-your-assets-safe"></a>保护资产安全
自动备份后端和数据库。 代码和数据受到保护，即使出现灾难也不会破坏并且可以轻松还原，让用户放心开展业务。

##<a name="ready-stage-go"></a>各就位，预备，跑！
使用 *Azure 应用服务* ，可以为移动应用创建多个专用测试和过渡环境。 可以在部署之前使用这些环境执行测试。 无需停机即可切换到生产环境。 预先加载 Web 应用，确保最佳客户体验。

可以遵循此 *教程* ，开始将[应用服务](/documentation/articles/app-service-mobile-migrating-from-mobile-services/)用于现有移动服务。

<!--Update_Description:update wording-->