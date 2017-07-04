<properties
    pageTitle="对 Azure RBAC 进行故障排除 | Azure"
    description="获取有关基于角色的访问控制资源问题或疑问的帮助。"
    services="azure-portal"
    documentationcenter="na"
    author="kgremban"
    manager="femila"
    editor="" />
<tags
    ms.assetid="df42cca2-02d6-4f3c-9d56-260e1eb7dc44"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/02/2017"
    wacn.date="04/05/2017"
    ms.author="kgremban" />  


# 基于角色的访问控制故障排除

本文回答有关使用角色授予的特定访问权限的常见问题，以便你了解在 Azure 门户中使用角色时可以预期什么，并可以排查访问权限问题。以下三种角色涵盖所有资源类型：

- 所有者
- 参与者
- 读取器

所有者和参与者对管理体验具有完全访问权限，但是参与者无法向其他用户授予访问权限。具有读者角色事情会变得更加有趣，因此，我们将着重介绍读者角色。有关如何授予访问权限的详细信息，请参阅[基于角色的访问控制入门文章](/documentation/articles/role-based-access-control-configure/)。

## App Service 工作负荷
### 写访问功能
如果你为用户授予单个 Web 应用的只读访问权限，某些功能可能会被禁用，这可能不是你所期望的。以下管理功能需要对 Web 应用具有**写**访问权限（参与者或所有者），并且在任何只读方案中不可用。

- 命令（例如启动、停止等。）
- 更改设置（如常规配置、缩放设置、备份设置和监视设置）
- 访问发布凭据和其他机密（如应用设置和连接字符串）
- 流式传输日志
- 诊断日志配置
- 控制台（命令提示符）
- 活动和最新部署（适用于本地 Git 连续部署）
- 估计费用
- Web 测试
- 虚拟网络（如果虚拟网络是以前具有写访问权限的用户所配置的，则仅对读者可见）。

如果你无法访问以上任何磁贴，则需要让管理员为你提供对 Web 应用的“参与者”访问权限。

### 处理相关资源
由于存在几个相互作用的不同资源，Web 应用程序是复杂的。下面是包含几个网站的典型资源组：

![Web 应用程序资源组](./media/role-based-access-control-troubleshooting/website-resource-model.png)

因此，如果你只授予某人对 Web 应用的访问权限，则 Azure 门户中的网站边栏选项卡上的很多功能将被禁用。

这些项需要对与网站对应的 **App Service 计划**具有**写**访问权限：

- 查看 Web 应用的定价层（“免费”或“标准”）
- 规模配置（实例数、虚拟机大小、自动缩放设置）
- 配额（存储空间、带宽、CPU）

这些项需要对包含网站的整个**资源组**具有**写**访问权限：

- SSL 证书和绑定（这是因为 SSL 证书可以在同一资源组和地理位置中的站点之间共享）
- 警报规则
- 自动缩放设置
- Application Insights 组件
- Web 测试

## 虚拟机工作负荷
与 Web 应用程序很类似，虚拟机边栏选项卡上的某些功能需要对虚拟机或资源组中的其他资源具有写访问权限。

虚拟机与域名、虚拟网络、存储帐户和警报规则相关。

这些项需要对**虚拟机**具有**写**访问权限：

- 终结点
- IP 地址
- 磁盘
- 扩展

这些项需要对**虚拟机**和其所在的**资源组**（以及域名）具有**写**访问权限：

- 可用性集
- 负载均衡集
- 警报规则

如果你不能访问以上任何磁贴，则需要让管理员为你提供对资源组的“参与者”访问权限。

## 另请参阅
- [基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)：Azure 门户中的 RBAC 入门。
- [内置角色](/documentation/articles/role-based-access-built-in-roles/)：获取有关 RBAC 中的标准角色的详细信息。
- [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)：了解如何创建自定义角色，以满足访问需要。
- [创建访问权限更改历史记录报告](/documentation/articles/role-based-access-control-access-change-history-report/)：记录 RBAC 中的角色分配更改。

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->