<properties 
	pageTitle="管理 Azure AD Connect" 
	description="了解如何扩展 Azure AD Connect 的默认配置和操作任务。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="terrylan" 
	editor="lisatoft"/>

<tags 
	ms.service="active-directory" 
	ms.date="04/02/2015"
	wacn.date="06/16/2015"/>

# 管理 Azure AD Connect 


<div>
<a href="/documentation/articles/active-directory-aadconnect/>简介</a> <a href="/documentation/articles/active-directory-aadconnect-how-it-works/">工作原理</a> <a href="/documentation/articles/active-directory-aadconnect-get-started/">入门</a> <a href="/documentation/articles/active-directory-aadconnect-get-manage/">管理</a></div>


主题属于高级操作主题，介绍如何根据组织的需要和要求自定义 Azure Active Directory Connect。

## 更改默认配置  

在大多数情况下，使用 Azure AD Connect 的默认配置便足以将本地目录扩展到云中。但是，在某些情况下，你可能需要修改默认配置，并根据组织的业务逻辑对其进行自定义。在这种情况下，你可以修改默认配置，不过，在这样做之前，需要注意几个事项。

如果你要升级或迁移 Azure AD Sync 或 DirSync，请注意以下事项：

- 将 Azure AD Sync 升级到较新版本之后，大多数设置会重置为默认值。
- 在应用升级后，对“现成的”同步规则所做的更改将会丢失。
- 在升级到较新版本期间，会重新创建已删除的“现成”同步规则。
- 在升级到较新版本时，你创建的自定义同步规则将会保持不变。

如果需要更改默认配置，请执行以下操作：

- 如果需要修改某个“现成”同步规则的属性流，请不要更改该规则。而是新建一个具有较高优先级（编号值较小）的、包含所需属性流的同步规则。
- 使用同步规则编辑器导出自定义同步规则。这会提供一个 PowerShell 脚本，你可以在灾难恢复方案中使用该脚本轻松重新创建同步规则。
- 如果你需要在“现成”同步规则中更改范围或联接设置，请记下这些更改，并升级到 Azure AD Connect 的较新版本之后重新应用更改。

## 使用同步规则编辑器

在 Azure AD Connect 中，你可以通过配置同步规则，来配置和优化 Azure AD 与本地目录之间的对象和属性流。可以使用同步规则编辑器配置同步规则。当你安装 Azure AD Connect 时，将会安装同步规则编辑器。若要使用该编辑器，你必须是安装 Azure AD Connect 期间指定的 ADSyncAdmins 组或 Administrator 组的成员。

在下面的屏幕截图中，你会看到使用快速安装选项安装 Azure AD Connect 时为配置创建的所有同步规则。表中的每一行代表一个同步规则。“规则类型”的左下侧列出了两种不同的类型：“入站”和“出站”。入站和出站来自 Metaverse 视图。也就是说，我们要将信息从目录发送到 Metaverse。出站表示我们要在其中向目录（例如本地 Active Directory 或 Azure AD）发送信息和属性的规则。

![同步规则编辑器](./media/active-directory-aadconnect-manage/Synch_Rule.png)


若要创建新规则，请选择“添加新规则”，然后配置规则。例如，假设我们要创建一个联接规则，其中，本地目录中的任何用户将与具有相同电话号码的 Metaverse 对象联接。为此，可以创建新规则，并指定连接的系统中（在本例中为 contoso.com）、连接的系统对象类型、用户、Metaverse 对象类型、人员和联接的链接类型。 

![创建同步规则](./media/active-directory-aadconnect-manage/synch2.png)



然后，在“联接规则”屏幕上指定 Source 属性下的 telephoneNumber 和 Target 属性下的 telephoneNumber。这就是所有的操作。现在，你已成功创建了一个联接规则。

![联接规则](./media/active-directory-aadconnect-manage/synch3.png)


你可以使用同步规则编辑器来应用默认配置外部的附加业务逻辑，并根据组织的需要对其进行自定义。有关同步规则编辑器的其他信息，请参阅[了解默认配置](https://msdn.microsoft.com/zh-cn/library/azure/dn800963.aspx)。


## 使用声明性设置 
声明性设置属于“无代码”设置，可以使用同步规则编辑器进行设置和配置。可以使用该编辑器来设置和创建自己的设置规则。

声明性设置的一个重要组成部分是属性流中使用的表达式语言。所用的语言是 Microsoft® Visual Basic® for Applications (VBA) 的子集。Microsoft Office 中使用了这种语言，具有 VBScript 经验的用户都认识该语言。声明性设置表达式语言只使用函数，不属于结构化语言；它不提供任何方法或语句。函数将嵌套在表达式程序流中。

有关表达式语言的详细信息，请参阅[了解声明性设置表达式](https://msdn.microsoft.com/zh-cn/library/azure/dn801048.aspx)

## 其他文档  

为 Azure AD Sync 创建的某些文档仍有参考价值，并适用于 Azure AD Connect。尽管我们正在努力将这些文档发布到 Azure.com，但有些文档目前仍然只能在 MSDN 范围的库中提供。有关其他文档，请参阅 [MSDN 上的 Azure AD Connect](https://msdn.microsoft.com/zh-cn/library/azure/dn832695.aspx) 和 [MSDN 上的 Azure AD Sync](https://msdn.microsoft.com/zh-cn/library/azure/dn790204.aspx)。

**其他资源**

* [在云中使用本地标识基础结构](active-directory-aadconnect-whatis)
* [Azure AD Connect 工作原理](active-directory-aadconnect-how-it-works)
* [Azure AD Connect 入门](active-directory-aadconnect-getstarted)
* [管理 Azure AD Connect](active-directory-aadconnect-manage)
* [MSDN 上的 Azure AD Connect](https://msdn.microsoft.com/zh-cn/library/azure/dn832695.aspx)

<!---HONumber=60-->