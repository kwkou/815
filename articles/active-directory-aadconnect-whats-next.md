<properties 
	pageTitle="管理 Azure AD Connect" 
	description="了解如何扩展 Azure AD Connect 的默认配置和操作任务。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>

# 管理 Azure AD Connect 



以下主题属于高级操作主题，介绍如何根据组织的需要和要求自定义 Azure Active Directory Connect。

## 向 Azure AD Premium 或企业移动套件用户分配许可证

将用户同步到云后，你需要向他们分配许可证，以便他们可以继续使用云应用（例如 Office 365）。

### 分配 Azure AD Premium 或企业移动套件许可证
--------------------------------------------------------------------------------
1. 以管理员身份登录到 Azure 门户。
2. 在左侧选择“Active Directory”。
3. 在“Active Directory”页上，双击包含你要启用的用户的目录。
4. 在“目录”页的顶部，选择“许可证”。
5. 在“许可证”页上，选择“Active Directory Premium”或“企业移动套件”，然后单击“分配”。
6. 在对话框中，选择要向其分配许可证的用户，然后单击复选标记图标以保存更改。


## 验证计划的同步任务
如果你想要检查同步状态，可以在 Azure 门户执行此操作。

### 验证计划的同步任务
--------------------------------------------------------------------------------

1. 以管理员身份登录到 Azure 门户。
2. 在左侧选择“Active Directory”。
3. 在“Active Directory”页上，双击包含你要启用的用户的目录。
4. 在“目录”页的顶部，选择“目录集成”。
5. 在与本地 Active Directory 的集成下方，记下上次同步时间。

<center>![云](./media/active-directory-aadconnect-whats-next/verify.png)</center>

## 启动计划的同步任务
如果需要运行同步工作，你可以通过再次运行 Azure AD Connect 向导来实现此目的。需要提供 Azure AD 凭据。在向导中，选择“自定义同步选项”任务，然后在向导中一直单击“下一步”。最后，请确保已选中“初始配置完成后立即开始同步过程”框。

<center>![云](./media/active-directory-aadconnect-whats-next/startsynch.png)</center>




## Azure AD Connect 中提供的其他任务
在完成 Azure AD Connect 的初始安装后，你随时可以从 Azure AD Connect 启动页或桌面快捷方式再次启动向导。在再次运行向导的过程中，你会发现，一些新选项以“其他任务”的形式提供。

下表提供了这些任务的摘要和各个任务的简要描述。

![联接规则](./media/active-directory-aadconnect-whats-next/addtasks.png)

若要创建新规则，请选择“添加新规则”，然后配置规则。例如，假设我们要创建一个联接规则，其中，本地目录中的任何用户将与具有相同电话号码的 Metaverse 对象联接。为此，可以创建新规则，并指定连接的系统中（在本例中为 contoso.com）、连接的系统对象类型、用户、Metaverse 对象类型、人员和联接的链接类型。

其他任务 | 说明 
------------- | ------------- |
查看选定的方案 |可让你查看当前的 Azure AD Connect 解决方案。这包括常规设置、同步的目录、同步设置，等等。
自定义同步选项 | 可让你更改当前的配置，包括在配置中添加其他 Active Directory 林，或启用同步选项，例如用户、组、设备或密码回写。
启用暂存模式 | 可让你暂存稍后将要同步的信息，但不会将任何内容导出到 Azure AD 或 Active Directory。这样，你便可以在同步之前进行预览。

## 使用声明性设置  
 
声明性设置属于“无代码”设置，可以使用同步规则编辑器进行设置和配置。可以使用该编辑器来设置和创建自己的设置规则。

声明性设置的一个重要组成部分是属性流中使用的表达式语言。所用的语言是 Microsoft® Visual Basic® for Applications (VBA) 的子集。Microsoft Office 中使用了这种语言，具有 VBScript 经验的用户都认识该语言。声明性设置表达式语言只使用函数，不属于结构化语言；它不提供任何方法或语句。函数将嵌套在表达式程序流中。

有关表达式语言的详细信息，请参阅[了解声明性设置表达式](https://msdn.microsoft.com/zh-cn/library/azure/dn801048.aspx)

## 其他文档
有关使用 Azure AD Connect 的更多文档，请参阅：

<!-- - [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis) -->
- [更改 Azure AD Connect 的默认配置](/documentation/articles/active-directory-aadconnect-whats-next-change-default-config)
- [使用 Azure AD Connect 同步规则编辑器](/documentation/articles/active-directory-aadconnect-whats-next-synch-rules-editor)
- [使用声明性设置](/documentation/articles/active-directory-aadconnect-whats-next-declarative-prov)

另外，为 Azure AD Sync 创建的某些文档仍有参考价值，并适用于 Azure AD Connect。尽管我们正在努力将这些文档发布到 Azure.com，但有些文档目前仍然只能在 MSDN 范围的库中提供。有关其他文档，请参阅 [MSDN 上的 Azure AD Connect](/documentation/articles/active-directory-aadconnect) 和 [MSDN 上的 Azure AD Sync](https://msdn.microsoft.com/library/azure/dn790204.aspx)。


 

<!---HONumber=76-->