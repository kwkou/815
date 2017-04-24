<properties
    pageTitle="Azure AD Synchronization Service Manager UI 中的连接器 | Azure"
    description="了解用于 Azure AD Connect 的 Synchronization Service Manager 中的“连接器”选项卡。"
    services="active-directory"
    documentationcenter=""
    author="andkjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid="60f1d979-8e6d-4460-aaab-747fffedfc1e"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/02/2017"
    wacn.date="04/05/2017"
    ms.author="billmath"
    ms.custom="H1Hack27Feb2017" />  


# 将连接器与 Azure AD Connect Sync Service Manager 配合使用

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/connectors.png)  


“连接器”选项卡可用于管理同步引擎连接的所有系统。

## 连接器操作
| 操作 | 注释 |
| --- | --- |
| 创建 |请勿使用。若要连接到其他 AD 林，请使用安装向导。 |
| 属性 |用于域和 OU 筛选。 |
| [删除](#delete) |用于删除连接器空间中的数据或删除与林的连接。 |
| [配置运行配置文件](#configure-run-profiles) |除了对于域筛选，无需在此进行任何配置。可以通过此操作来查看已配置的运行配置文件。 |
| 运行 |用于启动配置文件的一次性运行。 |
| 停止 |停止当前运行配置文件的连接器。 |
| 导出连接器 |请勿使用。 |
| 导入连接器 |请勿使用。 |
| 更新连接器 |请勿使用。 |
| 刷新架构 |刷新缓存架构。最好改为在安装向导中使用此选项，因为它也会更新同步规则。 |
| [搜索连接器空间](#search-connector-space) |用于查找对象，以及在整个系统中跟踪对象及其数据。 |

### 删除 <a name="delete"></a>
删除操作适用于两种不同的用途。
![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/connectordelete.png)

“仅删除连接器空间”选项会删除所有数据，但保留所有配置。

“删除连接器和连接器空间”选项会删除数据以及所有配置。不想再连接到林时可以使用此选项。

这两个选项都会同步所有对象，并更新 Metaverse 对象。这是一个长时间运行的操作。

### 配置运行配置文件 <a name="configure-run-profiles"></a>
此选项可让你查看为连接器配置的运行配置文件。

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/configurerunprofiles.png)

### 搜索连接器空间 <a name="search-connector-space"></a>
查找对象和排查数据问题时，搜索连接器空间操作非常有用。

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearch.png)

先选择一个“范围”。你可以基于数据（RDN、DN、定位点、子树）或对象状态（所有其他选项）进行搜索。

![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearchscope.png) 

例如，如果进行子树搜索，将获取某个 OU 中的所有对象。
![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/cssearchsubtree.png) 
可以从此处选择对象，选择“属性”，从源连接器空间到 Metaverse 再到目标连接器空间一直[跟踪对象](/documentation/articles/active-directory-aadconnectsync-troubleshoot-object-not-syncing/)。

## 后续步骤
了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: wording update-->