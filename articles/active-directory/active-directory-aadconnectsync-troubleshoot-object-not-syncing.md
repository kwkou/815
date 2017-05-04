<properties
    pageTitle="针对对象无法同步到 Azure AD 进行故障排除 | Azure"
    description="针对对象无法同步到 Azure AD 进行故障排除。"
    services="active-directory"
    documentationcenter=""
    author="andkjell"
    manager="femila"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/02/2017"
    wacn.date="05/02/2017"
    ms.author="billmath"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="d588cd858411fa7b09cd334d38196960977a096e"
    ms.lasthandoff="04/22/2017" />

# <a name="troubleshoot-an-object-that-is-not-synchronizing-to-azure-ad"></a>对未同步到 Azure AD 的对象进行故障排除

如果某个对象没有按预期同步到 Azure AD，则可能有多种原因。 如果你收到来自 Azure AD 的有关错误的电子邮件，或者在 Azure AD Connect Health 中看到错误，请参阅[解决导出错误](/documentation/articles/active-directory-aadconnect-troubleshoot-sync-errors/)。 但如果你是要排查不在 Azure AD 中的对象的问题，则本主题适用于你。 它介绍了如何在本地组件 Azure AD Connect 同步中查找错误。

若要查找错误，可以按以下顺序查看几个不同的位置：

1. [操作日志](#operations) ，查找由同步引擎在导入和同步期间识别出的错误。
2. [连接器空间](#connector-space-object-properties) ，查找缺少的对象和同步错误。
3. [metaverse](#metaverse-object-properties) ，查找与数据相关的问题。

在开始这些步骤之前，启动 [Synchronization Service Manager](/documentation/articles/active-directory-aadconnectsync-service-manager-ui/)。

## <a name="operations"></a>操作
Synchronization Service Manager 中的“操作”选项卡是你应当从中开始故障排除的位置。 “操作”选项卡显示最新操作的结果。  
![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/operations.png)  

上半部分按时间顺序显示所有运行。 默认情况下，操作日志保留最后七天的相关信息，但可以使用[计划程序](/documentation/articles/active-directory-aadconnectsync-feature-scheduler/)来更改此设置。 想要查找所有未显示成功状态的运行。 可以单击标题来更改排序。

“状态”列是最重要的信息，并显示最严重的运行问题。 下面是按调查优先级顺序显示的最常见状态的快速摘要（其中 * 表示多个可能的错误字符串）。

| 状态 | 注释 |
| --- | --- |
| stopped-* |运行无法完成。 例如，如果远程系统已关闭且无法访问。 |
| stopped-error-limit |有 5,000 个以上的错误。 运行因错误数量过多而自动停止。 |
| completed-\*-errors |运行已完成，但发生应调查的错误（数量少于 5,000 个）。 |
| completed-\*-warnings |运行已完成，但某些数据并未处于预期的状态。 如果遇到错误，则此消息通常只是一种征兆。 在解决错误之前，不应该调查警告。 |
| success |没有问题。 |

选择某一行时，底部将更新以显示该运行的详细信息。 在底部的最左边，可能会有一份显示“步骤编号”的列表。 仅当林中有多个域，而且每个域都以一个步骤表示时，才会显示此列表。 可以在“分区”标题下方找到域名。 在“同步统计信息”下方，可以找到有关已处理更改次数的详细信息。 可以单击链接获取已更改对象的列表。 如果有对象发生错误，这些错误将显示在“同步错误”下方。

### <a name="troubleshoot-errors-in-operations-tab"></a>排查“操作”选项卡中的错误
![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/errorsync.png)  
遇到错误时，发生错误的对象与错误本身都是链接，这些链接提供更多信息。

首先单击错误字符串（图中的 **sync-rule-error-function-triggered**）。 随后会先看到对象概述。 若要查看实际的错误，可单击“堆栈跟踪”按钮。 此跟踪提供错误的调试级别信息。

可以在“调用堆栈信息”框中单击右键，然后选择“全选”“复制”。 接着可以复制堆栈，并在最喜爱的编辑器（例如记事本）中查看此错误。

- 如果错误来自 **SyncRulesEngine**，则调用堆栈信息首先会列出对象上的所有属性。 向下滚动，直到你看到 **InnerException =>** 标题为止。  
  ![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/errorinnerexception.png)  
  后面的行显示错误。 在上图中，错误来自所创建的自定义同步规则 Fabrikam。

如果错误本身未提供足够的信息，则请查看数据本身。 你可以单击具有对象标识符的链接并继续对 [连接器空间导入的对象](#cs-import)进行故障排除。

## <a name="connector-space-object-properties"></a>连接器空间对象属性
如果没有在[“操作”](#operations)选项卡中找到任何错误，则下一步是从 Active Directory 到 metaverse 然后到 Azure AD 查找连接器空间对象的问题。 在此路径中，你应能找到问题所在。

### <a name="search-for-an-object-in-the-cs"></a>搜索 CS 中的对象

在“Synchronization Service Manager”中，单击“连接器”，选择“Active Directory 连接器”和“搜索连接器空间”。

在“作用域”中，选择“RDN”（如果想要搜索 CN 属性）或“DN 或定位点”（如果想要搜索 distinguishedName 属性）。 输入一个值，然后单击“搜索”。  
![连接器空间搜索](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/cssearch.png)  

如果找不到要查找的对象，则可能已通过[基于域的筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering/#domain-based-filtering/)或[基于 OU 的筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering/#organizational-unitbased-filtering/)对其进行筛选。 阅读[配置筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering/)主题，以验证已按所需方式配置筛选。

另一种有用的搜索是选择 Azure AD 连接器，在“作用域”中，选择“挂起的导入”，然后选择“添加”复选框。 此搜索提供 Azure AD 中不能与本地对象相关联的所有已同步对象。  
![连接器空间搜索孤立](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/cssearchorphan.png)  
这些对象已由其他同步引擎或具有不同筛选配置的同步引擎创建。 此视图是不再托管的**孤立**对象的列表。 应查看此列表并考虑使用 [Azure AD PowerShell](http://aka.ms/aadposh) cmdlet 删除这些对象。

### <a name="cs-import"></a>CS 导入
打开 cs 对象时，顶端会出现数个选项卡。 “导入”选项卡显示导入后暂存的数据。  
![CS 对象](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/csobject.png)    
“旧值”显示当前存储在 Connect 中的数据，而“新值”显示从源系统收到但尚未应用的数据。 如果对象出现错误，则不会处理更改。

**错误**  
![CS 对象](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/cssyncerror.png)  
对象出现问题时才会显示“同步错误”选项卡。 有关详细信息，请参阅[解决同步错误](#troubleshoot-errors-in-operations-tab)。

### <a name="cs-lineage"></a>CS 沿袭
“沿袭”选项卡显示连接器空间对象与 Metaverse 对象有何关联。 可以看到连接器上次从连接的系统导入更改的时间，以及应用哪些规则以便在 Metaverse 中填充数据。  
![CS 沿袭](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/cslineage.png)  
在“操作”列中，可以看到有一个操作为“预配”的“入站”同步规则。 这表示只要此连接器空间对象存在，就会保留 Metaverse 对象。 如果同步规则列表显示的同步规则方向为“出站”和“预配”，则表示删除 Metaverse 对象时，也将删除此对象。  
![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/cslineageout.png)  
在“PasswordSync”列中，还会发现入站连接器空间可进行密码更改，因为有一个同步规则的值为 **True**。 此密码接着会通过出站规则发送到 Azure AD。

从“沿袭”选项卡中，可以单击“Metaverse 对象属性”转到 [Metaverse](#mv-attributes)。

所有选项卡的底部都有两个按钮：“预览”和“日志”。

### <a name="preview"></a>预览
“预览”页用来同步单个对象。 如果你正在对某些自定义同步规则进行故障排除，并且想要在单个对象上查看更改的效果，则此页面非常有用。 可以在“完全同步”和“增量同步”之间选择。 还可以在“生成预览”（仅在内存中保留更改）和“提交预览”（将更新 metaverse 并暂存对目标连接器空间的所有更改）之间选择。  
![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/preview.png)  
可以检查对象，以及哪一个规则适用于特定的属性流。  
![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/previewresult.png)

### <a name="log"></a>日志
“日志”页用来查看密码同步状态和历史记录。 有关详细信息，请参阅[排查密码同步问题](/documentation/articles/active-directory-aadconnectsync-troubleshoot-password-synchronization/)。

## <a name="metaverse-object-properties"></a>Metaverse 对象属性
通常，最好从源 Active Directory [连接器空间](#connector-space)开始搜索。 但是也可以从 metaverse 开始搜索。

### <a name="search-for-an-object-in-the-mv"></a>搜索 MV 中的对象
在“Synchronization Service Manager”中，单击“Metaverse 搜索”。 创建一个查找用户的查询。 可以搜索公共属性，例如 accountName (sAMAccountName) 和 userPrincipalName。 有关详细信息，请参阅 [Metaverse 搜索](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-mvsearch/)。
![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/mvsearch.png)  

在“搜索结果”窗口中，单击对象。

如果未找到对象，则它尚未到达 metaverse。 继续在 Active Directory [连接器空间](#connector-space-object-properties)中搜索对象。 可能是同步期间发生的错误阻止了对象来到 metaverse，也有可能是因为应用了筛选器。

### <a name="mv-attributes"></a>Metaverse 属性
在“属性”选项卡中，可以看到值，以及是哪个连接器提供了值。  
![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/mvobject.png)  

如果未能同步对象，可查看 metaverse 中的以下属性：
- 属性“cloudFiltered”是否存在并设置为“true”？ 如果是，则已根据[基于属性的筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering/#attribute-based-filtering/)中的步骤对其进行筛选。
- 属性 **sourceAnchor** 是否存在？ 如果不存在，你是否拥有帐户资源林拓扑？ 如果对象被标识为链接的邮箱（属性“msExchRecipientTypeDetails”的值为 2），则由具有已启用的 Active Directory 帐户的林提供 sourceAnchor。 请确保已正确导入和同步主帐户。 主帐户必须列在对象的 [连接器](#mv-connectors) 中。

### <a name="mv-connectors"></a>Metaverse 连接器
“连接器”选项卡显示具有对象表示形式的所有连接器空间。  
![Sync Service Manager](./media/active-directory-aadconnectsync-troubleshoot-object-not-syncing/mvconnectors.png)  
你应当具有连接到以下项的连接器：

- 其中表示了用户的每个 Active Directory 林。 此表示形式可能包括 foreignSecurityPrincipals 和 Contact 对象。
- Azure AD 中的某个连接器。

如果缺少连接到 Azure AD 的连接器，则阅读 [MV 属性](#MV-attributes)以验证有关预配到 Azure AD 的条件。

此选项卡也可让你导航到[连接器空间对象](#connector-space-object-properties)。 选择一行，然后单击“属性”。

## <a name="next-steps"></a>后续步骤
了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。

了解有关 [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!--Update_Description: wording properties -->