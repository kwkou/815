<properties
    pageTitle="Azure AD Connect 同步：处理 userCertificate 属性导致的 LargeObject 错误 | Azure"
    description="本主题提供针对 userCertificate 属性导致的 LargeObject 错误的补救步骤。"
    services="active-directory"
    documentationcenter=""
    author="cychua"
    manager="femila"
    editor="" />
<tags
    ms.assetid="146ad5b3-74d9-4a83-b9e8-0973a19828d9"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/27/2017"
    ms.author="billmath"
    wacn.date="06/12/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="7e11cb67eafee13c95fb6a76c08261f4ea4fcbb7"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="azure-ad-connect-sync-handling-largeobject-errors-caused-by-usercertificate-attribute"></a>Azure AD Connect 同步：处理 userCertificate 属性导致的 LargeObject 错误

Azure AD 针对 **userCertificate** 属性的最大证书值数目限制为 **15** 个。 如果 Azure AD Connect 将包含超过 15 个值的对象导出到 Azure AD，Azure AD 将返回 **LargeObject** 错误和以下消息：

>*“预配的对象太大。请减少此对象上属性值的数目。在下一同步周期内将重试该操作...”*

LargeObject 错误可能由其他 AD 属性导致。 若要确认该错误是否确实由 userCertificate 属性导致，需要在本地 AD 中或[同步服务管理器 Metaverse 搜索](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-mvsearch/)中验证该对象。

若要获取租户中出现 LargeObject 错误的对象列表，请使用以下方法之一：

 - 每个同步周期结束时发送的有关目录同步错误的通知电子邮件包含出现 LargeObject 错误的对象列表。 
 - 如果单击最新的“导出到 Azure AD”操作，“同步服务管理器操作”选项卡将显示出现 LargeObject 错误的对象列表。[](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-operations/)
 
## <a name="mitigation-options"></a>缓解选项
在解决 LargeObject 错误之前，对同一对象所做的其他属性更改将无法导出到 Azure AD 中。 若要解决该错误，可以考虑以下选项：

 - 在 Azure AD Connect 中实现一个**出站同步规则**，以便导出**对包含 15 个以上证书值的对象导出 null 值而不是实际值**。 如果你不需要将包含超过 15 个证书值的对象的任何证书值导出到 Azure AD，则此选项不适用。 有关如何实现此同步规则的详细信息，请参阅下一部分[实现同步规则以限制 userCertificate 属性的导出](#implementing-sync-rule-to-limit-export-of-usercertificate-attribute)。

 - 通过删除你的组织不再使用的值来减少本地 AD 对象中的证书值数（减到 15 个或更少）。 如果已过期或未使用的证书导致属性膨胀，则适合使用此方法。 可以使用[此处提供的 PowerShell 脚本](https://gallery.technet.microsoft.com/Remove-Expired-Certificates-0517e34f)来帮助查找、备份和删除本地 AD 中已过期的证书。 在删除证书之前，我们建议你与组织中的关键基础结构管理员确认。

 - 配置 Azure AD Connect，以避免将 userCertificate 属性导出到 Azure AD。 一般情况下，我们不建议使用此选项，因为 Microsoft Online Services 可能会使用该属性来实现特定的方案。 具体而言：
    - Exchange Online 和 Outlook 客户端使用 User 对象中的 userCertificate 属性进行消息签名和加密。 若要详细了解此功能，请参阅 [S/MIME for message signing and encryption](https://technet.microsoft.com/zh-cn/library/dn626158(v=exchg.150).aspx)（用于消息签名和加密的 S/MIME）一文。

    - Azure AD 使用 Computer 对象中的 userCertificate 属性来让已加入本地域的 Windows 10 设备连接到 Azure AD。

## <a name="implementing-sync-rule-to-limit-export-of-usercertificate-attribute"></a>实现同步规则以限制 userCertificate 属性的导出
若要解决 userCertificate 属性导致的 LargeObject 错误，可在 Azure AD Connect 中实现一个出站同步规则，以便**对包含 15 个以上证书值的对象导出 null 值而不是实际值**。 本部分介绍需要执行哪些步骤来为 **User** 对象实现同步规则。 可以针对 **Contact** 和 **Computer** 对象改写这些步骤。

> [AZURE.IMPORTANT]
> 导出 null 值会删除以前已成功导出到 Azure AD 的证书值。

步骤归纳如下：

1. 禁用同步计划程序，并验证是否没有正在进行的同步操作。
3. 查找 userCertificate 属性的现有出站同步规则。
4. 创建所需的出站同步规则。
5. 验证针对出现 LargeObject 错误的现有对象实施的新同步规则。
6. 将新的同步规则应用到出现 LargeObject 错误的其余对象。
7. 验证是否没有意外的更改正在等待导出到 Azure AD。
8. 将更改导出到 Azure AD。
9. 重新启用同步计划程序。

### <a name="step-1----disable-sync-scheduler-and-verify-there-is-no-synchronization-in-progress"></a>步骤 1 - 禁用同步计划程序，并验证是否没有正在进行的同步操作
确保实现新同步规则的中途不会发生同步，以免将意外的更改导出到 Azure AD。 若要禁用内置的同步计划程序，请执行以下操作：

1. 在 Azure AD Connect 服务器上启动 PowerShell 会话。

2. 通过运行以下 cmdlet 来禁用计划的同步：`Set-ADSyncScheduler -SyncCycleEnabled $false`

    > [AZURE.NOTE]
    > 前面的步骤仅适用于使用内置计划程序的较新 Azure AD Connect 版本 (1.1.xxx.x)。 如果你操作的是使用 Windows 任务计划程序的较旧 Azure AD Connect 版本 (1.0.xxx.x)，或者使用你自己的自定义计划程序（不常见）来触发定期同步，则需要相应地禁用这种同步。

3. 转到“开始”→“同步服务”，启动“Synchronization Service Manager”。

4. 转到“操作”选项卡，确认是否不存在状态为“正在进行”的操作。

### <a name="step-2----find-the-existing-outbound-sync-rule-for-usercertificate-attribute"></a>步骤 2 - 查找 userCertificate 属性的现有出站同步规则
应已启用并配置一个现有的同步规则用于将 User 对象的 userCertificate 属性导出到 Azure AD。 请找到此同步规则，确定其**优先顺序**和**范围筛选器**配置：

1. 转到“开始”→“同步规则编辑器”，启动“同步规则编辑器”。

2. 使用以下值配置搜索筛选器：

    | 属性 | 值 |
    | --- | --- |
    | 方向 |**Outbound** |
    | MV 对象类型 |**Person** |
    | 连接器 |*Azure AD 连接器的名称* |
    | 连接器对象类型 |**user** |
    | MV 属性 |**userCertificate** |

3. 如果使用 Azure AD 连接器的 OOB（现成）同步规则来导出 User 对象的 userCertficiate 属性，应取回“Out to AAD - User ExchangeOnline”规则。
4. 记下此同步规则的**优先顺序**值。
5. 选择该同步规则，然后单击“编辑”。
6. 在“编辑保留的规则确认”弹出对话框中单击“否”。 （不要担心，我们不会对此同步规则进行任何更改）。
7. 在编辑屏幕中选择“范围筛选器”选项卡。
8. 记下范围筛选器配置。 如果使用的是 OOB 同步规则，应该正好有**一个包含两个子句的范围筛选器组**，其中包括：

    | 属性 | 运算符 | 值 |
    | --- | --- | --- |
    | sourceObjectType | EQUAL | User |
    | cloudMastered | NOTEQUAL | True |

### <a name="step-3-create-the-outbound-sync-rule-required"></a>步骤 3 - 创建所需的出站同步规则
新同步规则的**范围筛选器**必须与现有同步规则相同，其**优先顺序**必须高于现有同步规则。 这可以确保将新同步规则应用到与现有同步规则相同的一组对象，并重写 userCertificate 属性的现有同步规则。 若要创建同步规则，请执行以下操作：
1. 在同步规则编辑器中，单击“添加新规则”按钮。
2. 在“说明”选项卡下面提供以下配置：

    | 属性 | 值 | 详细信息 |
    | --- | --- | --- |
    | 名称 | *提供名称* | 例如“Out to AAD - Custom override for userCertificate” |
    | 说明 | *提供说明* | 例如“If userCertificate attribute has more than 15 values, export NULL” |
    | 连接的系统 | *选择 Azure AD 连接器* |
    | 连接的系统对象类型 | **user** | |
    | Metaverse 对象类型 | **person** | |
    | 链接类型 | **Join** | |
    | 优先级 | *选择一个介于 1 和 99 之间的数字* | 选择的数字不能由任何现有同步规则使用，并且值必须小于现有的同步规则（因此优先顺序更高）。 |

3. 转到“范围筛选器”选项卡，并实现现有同步规则所用的相同范围筛选器。
4. 跳过“联接规则”选项卡。
5. 转到“转换”选项卡，使用以下配置添加一个新的转换：

    | 属性 | 值 |
    | --- | --- |
    | 流类型 |**表达式** |
    | 目标属性 |**userCertificate** |
    | 源属性 |使用以下表达式：`IIF(IsNullOrEmpty([userCertificate]), NULL, IIF((Count([userCertificate])> 15),AuthoritativeNull,[userCertificate]))` |
    
6. 单击“添加”按钮来创建同步规则。

### <a name="step-4----verify-the-new-sync-rule-on-an-existing-object-with-largeobject-error"></a>步骤 4 - 验证针对出现 LargeObject 错误的现有对象实施的新同步规则
在将创建的同步规则应用到其他对象之前，可以执行此步骤来验证是否可对出现 LargeObject 错误的现有 AD 对象正常运行该规则：

1. 在 Synchronization Service Manager 中转到“操作”选项卡。
2. 选择最近的“导出到 Azure AD”操作，然后单击出现 LargeObject 错误的对象之一。
3. 在“连接器空间对象属性”弹出屏幕中，单击“预览”按钮。
4. 在“预览”弹出屏幕中选择“完全同步”，然后单击“提交预览”。
5. 关闭“预览”屏幕和“连接器空间对象属性”屏幕。
6. 在 Synchronization Service Manager 中转到“连接器”选项卡。
7. 右键单击“Azure AD”连接器，然后选择“运行...”
8. 在“运行连接器”弹出窗口中选择“导出”步骤，然后单击“确定”。
9. 等待完成导出到 Azure AD，然后确认此特定对象未出现其他 LargeObject 错误。

### <a name="step-5----apply-the-new-sync-rule-to-remaining-objects-with-largeobject-error"></a>步骤 5 - 将新的同步规则应用到出现 LargeObject 错误的其余对象
添加同步规则后，需要在 AD 连接器上运行完全同步步骤：

1. 在 Synchronization Service Manager 中转到“连接器”选项卡。
2. 右键单击“AD”连接器，然后选择“运行...”
3. 在“运行连接器”弹出窗口中选择“完全同步”步骤，然后单击“确定”。
4. 等待完全同步步骤完成。
5. 如果有多个 AD 连接器，请针对剩余的 AD 连接器重复上述步骤。 通常，如果有多个本地目录，则需要多个连接器。

### <a name="step-6----verify-there-are-no-unexpected-changes-waiting-to-be-exported-to-azure-ad"></a>步骤 6 - 验证是否没有意外的更改正在等待导出到 Azure AD
1. 在 Synchronization Service Manager 中转到“连接器”选项卡。
2. 右键单击“Azure AD”连接器，然后选择“搜索连接器空间”。
3. 在“搜索连接器空间”弹出窗口中：
    1. 将“范围”设置为“挂起的导出”。
    2. 选中所有 3 个复选框，包括“添加”、“修改”和“删除”。
    3. 单击“搜索”按钮，返回等待将其更改导出到 Azure AD 的所有对象。
    4. 验证是否没有意外的更改。 若要检查给定对象的更改，请双击该对象。

### <a name="step-7----export-the-changes-to-azure-ad"></a>步骤 7 - 将更改导出到 Azure AD
若要将更改导出到 Azure AD，请执行以下操作：

1. 在 Synchronization Service Manager 中转到“连接器”选项卡。
2. 右键单击“Azure AD”连接器，然后选择“运行...”
4. 在“运行连接器”弹出窗口中选择“导出”步骤，然后单击“确定”。
5. 等待完成导出到 Azure AD，然后确认不存在其他 LargeObject 错误。

### <a name="step-8----re-enable-sync-scheduler"></a>步骤 8 - 重新启用同步计划程序
解决问题后，请重新启用内置的同步计划程序：

1. 启动 PowerShell 会话。
2. 通过运行以下 cmdlet 来重新启用计划的同步：`Set-ADSyncScheduler -SyncCycleEnabled $true`

> [AZURE.NOTE]
> 前面的步骤仅适用于使用内置计划程序的较新 Azure AD Connect 版本 (1.1.xxx.x)。 如果你操作的是使用 Windows 任务计划程序的较旧 Azure AD Connect 版本 (1.0.xxx.x)，或者使用你自己的自定义计划程序（不常见）来触发定期同步，则需要相应地禁用这种同步。

## <a name="next-steps"></a>后续步骤
了解有关 [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

