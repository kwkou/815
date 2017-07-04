<properties
    pageTitle="Azure AD Connect 同步：配置筛选 | Azure"
    description="介绍如何在 Azure AD Connect 同步中配置筛选。"
    services="active-directory"
    documentationcenter=""
    author="andkjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid="880facf6-1192-40e9-8181-544c0759d506"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/21/2017"
    wacn.date="04/05/2017"
    ms.author="billmath" />  


# Azure AD Connect 同步：配置筛选
使用筛选功能可以控制本地目录中的哪些对象应该出现在 Azure Active Directory (Azure AD) 中。默认配置将采用配置的林中所有域内的所有对象。我们一般建议使用这种配置。使用 Exchange Online 和 Skype for Business 等 Office 365 工作负荷的用户将受益于完整的全局地址列表，因为这样可以发送电子邮件和呼叫每个联系人。使用默认配置时，用户获得的体验与使用 Exchange 或 Lync 的本地实现获得的相同。

但在某些情况下，需要对默认配置进行一些更改。下面是一些示例：

- 你打算使用[多重 Azure AD 目录拓扑](/documentation/articles/active-directory-aadconnect-topologies/#each-object-only-once-in-an-azure-ad-tenant/)。然后，需要应用筛选器以控制要将哪些对象同步到特定的 Azure AD 目录。
- 要试用 Azure 或 Office 365，因此只想在 Azure AD 中创建少量的用户。在进行小规模试用时，无需使用完整全局地址列表即可演示功能。
- Azure AD 中有很多不需要的服务帐户和其他非个人帐户。
- 出于合规性的原因，不能删除任何本地用户帐户，而只能禁用它们。但是，在 Azure AD 中，你只希望存在活动的帐户。

本文介绍如何配置不同的筛选方法。

> [AZURE.IMPORTANT]
Microsoft 不支持在正式记录的这些操作之外修改或操作 Azure AD Connect 同步。其中的任何操作都可能会导致 Azure AD Connect 同步出现不一致或不受支持状态。因此，Microsoft 无法提供这种部署的技术支持。

## 基础知识和重要说明
在 Azure AD Connect 同步中，你可以随时启用筛选。如果你从目录同步的默认配置开始并在此后配置了筛选，则筛选出的对象不再同步到 Azure AD。由于这种变化，Azure AD 中前面已同步，但之后进行筛选的任何对象会在 Azure AD 中进行删除。

在开始更改筛选之前，请确保[禁用计划的任务](#disable-scheduled-task)，以免意外导出尚未确认是否正确的更改。

由于筛选操作可能会同时删除很多的对象，因此请先确保新的筛选器正确无误，然后再开始将更改导出到 Azure AD。在完成配置步骤后，强烈建议先执行[验证步骤](#apply-and-verify-changes)，然后才对 Azure AD 进行导出和更改操作。

为了防止意外删除许多对象，默认情况下已打开[防止意外删除](/documentation/articles/active-directory-aadconnectsync-feature-prevent-accidental-deletes/)功能。如果由于筛选而删除了许多对象（默认为 500 个），则需要遵循本文中的步骤来允许将删除结果传播到 Azure AD。

如果使用 2015 年 11 月 ([1\.0.9125](/documentation/articles/active-directory-aadconnect-version-history/#1091250/)) 之前的内部版本、更改筛选器配置或使用密码同步，则在完成配置之后，需要触发所有密码的完全同步。有关如何触发密码完全同步的步骤，请参阅[触发所有密码的完全同步](/documentation/articles/active-directory-aadconnectsync-troubleshoot-password-synchronization/#trigger-a-full-sync-of-all-passwords/)。如果使用内部版本 1.0.9125 或更高版本，则常规的**完全同步**操作也会计算是否应同步密码，因此不再需要执行这个额外的步骤。

如果在 Azure AD 中由于筛选错误导致**用户**对象被无意中删除，可以通过删除筛选配置，在 Azure AD 中重新创建用户对象，然后再次同步目录。此操作可以从 Azure AD 的回收站中还原用户。但是，无法取消删除其他对象类型。例如，如果意外删除了某个安全组，而该组用于将资源加入 ACL，则无法恢复该组及其 ACL。

Azure AD Connect 只删除其曾经认为在范围中的对象。如果 Azure AD 中的对象由另一个同步引擎创建且不在范围内，则添加筛选并不会删除这些对象。例如，如果一开始使用 DirSync 服务器，而该服务器在 Azure AD 中创建了整个目录的完整副本，然后在从一开始便启用筛选的情况下并行安装了新的 Azure AD Connect 同步服务器，则 Azure AD Connect 不会删除 DirSync 所创建的额外对象。

安装或升级到较新版本的 Azure AD Connect 时，筛选配置将会保留。运行第一个同步周期之前，在升级到较新版本之后验证配置没有被无意中更改，这始终是最佳做法。

如果有多个林，必须将本主题中所述的筛选配置应用到每个林（假设要让所有林使用相同的配置）。

### 禁用计划任务 <a name="disable-scheduled-task"></a>
若要禁用每隔 30 分钟触发同步周期一次的内置计划程序，请遵循以下步骤：

1. 转到 PowerShell 提示符。
2. 运行 `Set-ADSyncScheduler -SyncCycleEnabled $False` 以禁用计划程序。
3. 如本文中所述进行更改。
4. 运行 `Set-ADSyncScheduler -SyncCycleEnabled $True` 以再次启用计划程序。

**如果使用低于 1.1.105.0 的 Azure AD Connect 内部版本**若要禁用每三小时触发一次同步循环的已计划任务，请执行以下步骤：

1. 从开始菜单启动“任务计划程序”。
2. 在“任务计划程序库”正下方找到名为“Azure AD 同步计划程序”的任务，单击右键，然后选择“禁用”。![任务计划程序](./media/active-directory-aadconnectsync-configure-filtering/taskscheduler.png)
3. 现在可以进行配置更改，并从“同步服务管理器”控制台手动运行同步引擎。

完成所有筛选更改之后，别忘了回来重新**启用**任务。

## 筛选选项
可将以下筛选配置类型应用到目录同步工具：

- [**基于组**](/documentation/articles/active-directory-aadconnect-get-started-custom/#sync-filtering-based-on-groups/)：初始安装时只能使用安装向导配置基于单个组的筛选。本文不会进一步介绍此操作。
- [**基于域**](#domain-based-filtering)：使用此选项可以选择要将哪些域同步到 Azure AD。在安装 Azure AD Connect 同步之后对本地基础结构进行更改时，还可以在同步引擎配置中添加和删除域。
- [**基于组织单位 (OU)**](#organizational-unitbased-filtering)：使用此选项可以选择要将哪些 OU 同步到 Azure AD。将对所选 OU 中的所有对象类型应用此选项。
- [**基于属性**](#attribute-based-filtering)：使用此选项可根据对象属性值筛选对象。你也可以对不同的对象类型使用不同的筛选器。

可以同时使用多个筛选选项。例如，可以使用基于 OU 的筛选以便只包含某个 OU 中的对象。同时，可以使用基于属性的筛选进一步筛选这些对象。使用多个筛选方法时，筛选器之间使用逻辑“AND”。

## 基于域的筛选 <a name="domain-based-filtering"></a>
本部分提供配置域筛选器时需要执行的步骤。如果安装 Azure AD Connect 之后在林中添加或删除了域，则也必须更新筛选配置。

更改基于域的筛选的首选方法是运行安装向导并更改[域和 OU 筛选](/documentation/articles/active-directory-aadconnect-get-started-custom/#domain-and-ou-filtering/)。使用安装向导可以自动完成本主题中所述的所有任务。

仅当出于某种原因而无法运行安装向导时，才遵循以下步骤。

基于域的筛选配置包括以下步骤：

1. [选择想要包含在同步操作中的域](#select-domains-to-be-synchronized)。
2. 针对添加和删除的每个域，请调整[运行配置文件](#update-run-profiles)。
3. [应用并验证更改](#apply-and-verify-changes)。

### 选择要同步的域 <a name="select-domains-to-be-synchronized"></a>
若要设置域筛选器，请执行以下步骤：

1. 通过使用属于 **ADSyncAdmins** 安全组的成员的帐户，登录到正在运行 Azure AD Connect 同步的服务器。
2. 从“开始”菜单启动“同步服务”。
3. 选择“连接器”，然后在“连接器”列表中选择类型为“Active Directory 域服务”的连接器。从“操作”中选择“属性”。
![连接器属性](./media/active-directory-aadconnectsync-configure-filtering/connectorproperties.png)
4. 单击“配置目录分区”。
5. 在“选择目录分区”列表中，根据需要选择和取消选择域。确认只选择了想要同步的分区。
![分区](./media/active-directory-aadconnectsync-configure-filtering/connectorpartitions.png) 如果更改了本地 Active Directory 基础结构并在林中添加或删除了域，请单击“刷新”按钮以获取更新的列表。刷新时，系统将要求提供凭据。请提供具有 Windows Server Active Directory 读取权限的任何凭据。不一定要使用对话框中预先填充的用户。
![需要刷新](./media/active-directory-aadconnectsync-configure-filtering/refreshneeded.png)
6. 完成后，请单击“确定”关闭“属性”对话框。如果在林中删除了域，屏幕上会弹出消息，指出已删除域且将清除配置。
7. 继续调整[运行配置文件](#update-run-profiles)。

### 更新运行配置文件 <a name="update-run-profiles"></a>
如果已修改域筛选器，则还需要更新运行配置文件。

1. 在“连接器”列表中，确保已选择在上一个步骤中更改的连接器。在“操作”中选择“配置运行配置文件”。![连接器运行配置文件 1](./media/active-directory-aadconnectsync-configure-filtering/connectorrunprofiles1.png)
2. 找到以下配置文件：
    - 完全导入
    - 完全同步
    - 增量导入
    - 增量同步
    - 导出
3. 对于每个配置文件，调整**已添加**和**已删除**的域。
    1. 针对上述五个配置文件，请对每个**已添加**的域执行以下步骤：
        1. 选择运行配置文件，然后单击“新步骤”。
        2. 在“配置步骤”页上的“类型”下拉菜单中，选择与要配置的配置文件同名的步骤类型。然后，单击“下一步”。
	![连接器运行配置文件 2](./media/active-directory-aadconnectsync-configure-filtering/runprofilesnewstep1.png)
        3. 在“连接器配置”页上的“分区”下拉菜单中，选择已添加到域筛选器的域名。
	![连接器运行配置文件 3](./media/active-directory-aadconnectsync-configure-filtering/runprofilesnewstep2.png)
        4. 若要关闭“配置运行配置文件”对话框，请单击“完成”。
    2. 针对上述五个配置文件，请对每个**已删除**的域执行以下步骤：
        1. 选择运行配置文件。
        2. 如果“分区”属性的“值”为 GUID，请选择运行步骤并单击“删除步骤”。
	![连接器运行配置文件 4](./media/active-directory-aadconnectsync-configure-filtering/runprofilesdeletestep.png)
    3. 检查更改。想要同步的每个域都已列为每个运行配置文件中的步骤。
4. 若要关闭“配置运行配置文件”对话框，请单击“确定”。
5.  若要完成配置，需要运行**完全导入**和**增量同步**。请继续阅读[应用并检查更改](#apply-and-verify-changes)部分。

## 基于组织单位的筛选 <a name="organizational-unitbased-filtering"></a>
更改基于 OU 的筛选的首选方法是运行安装向导并更改[域和 OU 筛选](/documentation/articles/active-directory-aadconnect-get-started-custom/#domain-and-ou-filtering/)。使用安装向导可以自动完成本主题中所述的所有任务。

仅当出于某种原因而无法运行安装向导时，才遵循以下步骤。

若要配置基于组织单位的筛选，请执行以下步骤：

1. 通过使用属于 **ADSyncAdmins** 安全组的成员的帐户，登录到正在运行 Azure AD Connect 同步的服务器。
2. 从“开始”菜单启动“同步服务”。
3. 选择“连接器”，然后在“连接器”列表中选择类型为“Active Directory 域服务”的连接器。从“操作”中选择“属性”。
![连接器属性](./media/active-directory-aadconnectsync-configure-filtering/connectorproperties.png)
4. 单击“配置目录分区”，选择要配置的域，然后单击“容器”。
5. 出现提示时，请提供具有本地 Active Directory 读取权限的任何凭据。不一定要使用对话框中预先填充的用户。
6. 在“选择容器”对话框中，清除不想与云目录同步的 OU，然后单击“确定”。
![“选择容器”对话框中的 OU](./media/active-directory-aadconnectsync-configure-filtering/ou.png)
   - 应选择“计算机”容器，这样 Windows 10 计算机才能成功同步到 Azure AD。如果已加入域的计算机位于其他 OU，请确保已选择这些计算机。
   - 如果有多个信任的林，则应选择 **ForeignSecurityPrincipals** 容器。使用此容器可以解析跨林安全组成员身份。
   - 如果已启用设备写回功能，则应选择 **RegisteredDevices** OU。如果你使用另一种写回功能，例如组写回，请确保已选择这些位置。
   - 选择用户、iNetOrgPersons、组、联系人和计算机所在位置的其他 OU。在上图中，这些 OU 全都位于 ManagedObjects OU 中。
   - 如果使用基于组的筛选，必须包含该组所在的 OU。
   - 请注意，可以配置在完成筛选配置后添加的新 OU 是否应该同步。有关详细信息，请参阅下一节。
7. 完成后，请单击“确定”关闭“属性”对话框。
8. 若要完成配置，需要运行**完全导入**和**增量同步**。请继续阅读[应用并检查更改](#apply-and-verify-changes)部分。

### 同步新 OU
默认情况下，将同步配置完筛选之后创建的新 OU。选中的复选框指示了此状态。可以取消选中某些子 OU。为此，请单击该框，直到该框变为白色，复选标记为蓝色（这是其默认状态）。然后，取消选中不需要同步的子 OU。

如果已同步所有子 OU，该框将变为白色，复选标记为蓝色。![OU，已选中所有框](./media/active-directory-aadconnectsync-configure-filtering/ousyncnewall.png)

如果已取消选中某些子 OU，则框变为灰色，复选标记为白色。![OU，已取消选中某些子 OU](./media/active-directory-aadconnectsync-configure-filtering/ousyncnew.png)

使用此配置时，将会同步在 ManagedObjects 下创建的新 OU。

Azure AD Connect 安装向导始终创建此配置。

### 不要同步新 OU
可将同步引擎配置为在完成筛选配置后不同步新 OU。如果 UI 中纯灰色框内不包含复选标记，则表示状态为不同步。若要实施此行为，请单击该框，使其变为白色且不包含复选标记。然后，选择要同步的子 OU。

![OU，已取消选中根](./media/active-directory-aadconnectsync-configure-filtering/oudonotsyncnew.png)  


使用此配置时，在 ManagedObjects 下创建的新 OU 不会同步。

## 基于属性的筛选 <a name="attribute-based-filtering"></a>
为了正常执行这些步骤，请确保使用 2015 年 11 月 ([1\.0.9125](/documentation/articles/active-directory-aadconnect-version-history/#1091250/)) 或更高版本。

基于属性的筛选是最灵活的对象筛选方式。将对象同步到 Azure AD 时，可以使用[声明性预配](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning/)功能来控制几乎每个方面的问题。

可以应用从 Active Directory 到 Metaverse 的[入站](#inbound-filtering)筛选，以及从 Metaverse 到 Azure AD 的[出站](#outbound-filtering)筛选。建议应用入站筛选，因为这样做最容易进行维护。仅当需要先要从多个林加入对象再进行评估时，才使用出站筛选。

### 入站筛选 <a name="inbound-filtering"></a>
入站筛选使用默认配置，其中，传入 Azure AD 的对象必须未将 Metaverse 属性 cloudFiltered 设置为要同步的值。如果这个属性的值设置为 **True**，则不会同步对象。根据设计，此值不应设为 **False**。若要确保其他规则能够提供值，这个属性只应具有 **True** 或 **NULL**（不存在）值。

在入站筛选中，使用**范围**功能来决定哪些对象要同步或者不同步。可以在此处根据组织的要求进行调整。范围模块包含**组**和**子句**，决定何时在范围内包含同步规则。一个组包含一个或多个子句。多个子句之间使用逻辑“AND”，多个组之间使用逻辑“OR”。

让我们看看以下示例：
![作用域](./media/active-directory-aadconnectsync-configure-filtering/scope.png) 
这应该写为 **(department = IT) OR (department = Sales AND c = US)**。

以下示例和步骤以用户对象为例，但可以将此例子应用到所有对象类型。

在以下示例中，优先顺序值从 50 开始。这可以是未使用的任何数值，但应小于 100。

#### 负筛选：“不同步这些项目”
在以下示例中，将筛选出（不同步）其中 **extensionAttribute15** 具有值 **NoSync** 的所有用户。

1. 通过使用属于 **ADSyncAdmins** 安全组的成员的帐户，登录到正在运行 Azure AD Connect 同步的服务器。
2. 从“开始”菜单启动“同步规则编辑器”。
3. 确保选择了“入站”，然后单击“添加新规则”。
4. 为规则指定一个说明性的名称，如 *In from AD - User DoNotSyncFilter*。选择正确的林，选择“用户”作为“CS 对象类型”，选择“人员”作为“MV 对象类型”。在“链接类型”中选择“联接”。在“优先顺序”中，键入当前未由其他同步规则使用的值（例如 50），然后单击“下一步”。
![入站 1 说明](./media/active-directory-aadconnectsync-configure-filtering/inbound1.png)
5. 在“范围筛选器”中，单击“添加组”，然后单击“添加子句”。在“属性”中选择“ExtensionAttribute15”。确保“运算符”设置为“等于”，在“值”框中键入值 **NoSync**。单击“下一步”。
![入站 2 范围](./media/active-directory-aadconnectsync-configure-filtering/inbound2.png)
6. 将“联接”规则留空，然后单击“下一步”。
7. 单击“添加转换”，为“FlowType”选择“Constant”，为“目标属性”选择“cloudFiltered”。在“源”文本框中键入 **True**。单击“添加”保存规则。
![入站 3 转换](./media/active-directory-aadconnectsync-configure-filtering/inbound3.png)
8. 若要完成配置，需要运行**完全同步**。请继续阅读[应用并检查更改](#apply-and-verify-changes)部分。

#### 正筛选：“只同步这些项目”
表达正筛选更加复杂，因为必须同时考虑不是明显需要同步的对象，例如会议室。你还要重写现成规则 **In from AD - User Join** 中的默认筛选器。创建自定义筛选器时，请确保不包括 Azure AD Connect 的关键系统对象、复制冲突对象、特殊邮箱和服务帐户。

正筛选选项需要两个同步规则。需要一个或多个包含要同步对象的正确范围的规则。还需要另一个全方位同步规则，用于筛选出尚未标识为属于应同步对象的所有对象。

在以下示例中，只同步部门属性值为 **Sales** 的用户对象。

1. 通过使用属于 **ADSyncAdmins** 安全组的成员的帐户，登录到正在运行 Azure AD Connect 同步的服务器。
2. 从“开始”菜单启动“同步规则编辑器”。
3. 确保选择了“入站”，然后单击“添加新规则”。
4. 为规则指定一个说明性的名称，如 *In from AD - User Sales sync*。选择正确的林，选择“用户”作为“CS 对象类型”，选择“人员”作为“MV 对象类型”。在“链接类型”中选择“联接”。在“优先顺序”中，键入当前未由其他同步规则使用的值（例如 51），然后单击“下一步”。
![入站 4 说明](./media/active-directory-aadconnectsync-configure-filtering/inbound4.png)
5. 在“范围筛选器”中，单击“添加组”，然后单击“添加子句”。在“属性”中选择“department”。确保“运算符”设置为“等于”，在“值”框中键入值 **Sales**。单击“下一步”。
![入站 5 范围](./media/active-directory-aadconnectsync-configure-filtering/inbound5.png)
6. 将“联接”规则留空，然后单击“下一步”。
7. 单击“添加转换”，为“FlowType”选择“Constant”，为“目标属性”选择“cloudFiltered”。在“源”框中键入 **False**。单击“添加”保存规则。
![入站 6 转换](./media/active-directory-aadconnectsync-configure-filtering/inbound6.png) 这是一种特殊情况，在此将 cloudFiltered 显式设置为 **False**。
8. 我们现在必须创建全方位同步规则。为规则指定一个说明性的名称，如 *In from AD - User Catch-all filter*。选择正确的林，选择“用户”作为“CS 对象类型”，选择“人员”作为“MV 对象类型”。在“链接类型”中选择“联接”。在“优先顺序”中，键入当前未由其他同步规则使用的值（例如 99）。现已选择高于先前同步规则的优先顺序值（较低优先顺序）。但同时也预留了一些空间，以便可以在稍后想要开始同步其他部门时添加其他筛选同步规则。单击“下一步”。
![入站 7 说明](./media/active-directory-aadconnectsync-configure-filtering/inbound7.png)
9. 让“范围筛选器”保留空白，然后单击“下一步”。空白筛选器表示规则将应用到所有对象。
10. 将“联接”规则留空，然后单击“下一步”。
11. 单击“添加转换”，为“FlowType”选择“Constant”，为“目标属性”选择“cloudFiltered”。在“源”框中键入 **True**。单击“添加”保存规则。
![入站 3 转换](./media/active-directory-aadconnectsync-configure-filtering/inbound3.png)
12. 若要完成配置，需要运行**完全同步**。请继续阅读[应用并检查更改](#apply-and-verify-changes)部分。

如果需要，可以创建更多第一种类型的规则，以便在同步中包含更多的对象。

### 出站筛选 <a name="outbound-filtering"></a>
在某些情况下，仅在对象已联接到 metaverse 中之后执行筛选是必要的。例如，可能需要从资源林中查看邮件属性，并需要从帐户林中查看 userPrincipalName 属性，确定某个对象是否应同步。在这些情况下，可以基于出站规则创建筛选。

本示例将更改筛选，以便只同步 mail 和 userPrincipalName 均以 @contoso.com 结尾的用户：

1. 通过使用属于 **ADSyncAdmins** 安全组的成员的帐户，登录到正在运行 Azure AD Connect 同步的服务器。
2. 从“开始”菜单启动“同步规则编辑器”。
3. 在“规则类型”下，单击“出站”。
4. 查找名为 **Out to AAD - User Join SOAInAD** 的规则，然后单击“编辑”。
5. 在弹出窗口中，回答“是”创建规则的副本。
6. 在“说明”页上，将“优先顺序”更改为某个尚未使用的值，例如 50。
7. 单击左侧导航栏中的“范围筛选器”，然后单击“添加子句”。在“属性”中选择“mail”。在“运算符”中选择“ENDSWITH”。在“值”中键入 **@contoso.com**，然后单击“添加子句”。在“属性”中选择“userPrincipalName”。在“运算符”中选择“ENDSWITH”。在“值”中键入 **@contoso.com**。
8. 单击“保存”。
9. 若要完成配置，需要运行**完全同步**。请继续阅读[应用并检查更改](#apply-and-verify-changes)部分。

## 应用并验证更改 <a name="apply-and-verify-changes"></a>
更改配置后，必须将这些更改应用到系统中现有的对象。也可能需要处理同步引擎中当前不存在的对象，因此同步引擎需要再次读取源系统来验证其内容。

如果使用**域**或**组织单位**筛选更改了配置，则需要执行**完全导入**，接着执行**增量同步**。

如果使用**属性**筛选更改了配置，则需要执行**完全同步**。

执行以下步骤：

1. 从“开始”菜单启动“同步服务”。
2. 选择“连接器”。在“连接器”列表中选择前面进行了配置更改的连接器。在“操作”中选择“运行”。
![连接器运行](./media/active-directory-aadconnectsync-configure-filtering/connectorrun.png)
3. 在“运行配置文件”中，选择上一部分中所述的操作。如果需要运行两项操作，请在完成第一项操作后再运行第二项操作。（所选连接器的“状态”列为“空闲”）。

同步后，将暂存所有更改以便导出。在 Azure AD 中实际进行更改之前，请验证所有更改是否正确。

1. 启动命令提示符并转到 `%Program Files%\Azure AD Sync\bin`。
2. 运行 `csexport "Name of Connector" %temp%\export.xml /f:x`。在同步服务中可以找到连接器名称。它的名称类似于“contoso.com - AAD”（表示 Azure AD）。
3. 运行 `CSExportAnalyzer %temp%\export.xml > %temp%\export.csv`。
4. 现在在 %temp% 中已经有名为 export.csv 的文件，可在 Microsoft Excel 中检查。此文件包含要导出的所有更改。
5. 对数据或配置进行必要的更改并再次运行这些步骤（导入、同步和验证），直到要导出的更改都按预期进行。

感到满意后，将更改导出到 Azure AD。

1. 选择“连接器”。在“连接器”列表中选择“Azure AD 连接器”。在“操作”中选择“运行”。
2. 在“运行配置文件”中，选择“导出”。
3. 如果配置更改会删除许多对象，且数目超过配置的阈值（默认为 500），则在导出时会出现错误。如果看到此错误，需要暂时禁用[防止意外删除](/documentation/articles/active-directory-aadconnectsync-feature-prevent-accidental-deletes/)功能。

现在，需要再次启用计划程序。

1. 从开始菜单启动“任务计划程序”。
2. 在“任务计划程序库”正下方找到名为“Azure AD 同步计划程序”的任务，单击右键，然后选择“启用”。

## 后续步骤
- 了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。
- 了解有关[将本地标识与 Azure AD 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->