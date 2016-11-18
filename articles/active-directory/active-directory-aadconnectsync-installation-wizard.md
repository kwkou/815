<properties
	pageTitle="Azure AD Connect 同步：第二次运行安装向导 | Azure"
	description="介绍安装向导第二次运行时的工作原理。"
	keywords="Azure AD Connect 安装向导允许你在第二次运行它时配置维护设置"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/31/2016"
	wacn.date="10/11/2016"
	ms.author="billmath"/>


# Azure AD Connect 同步：第二次运行安装向导
首次运行 Azure AD Connect 安装向导时，该向导将逐步引导用户配置安装。如果再次运行安装向导，它将提供维护选项。

你可以在"开始"菜单中找到名为 **Azure AD Connect** 的安装向导。

!["开始"菜单](./media/active-directory-aadconnectsync-installation-wizard/startmenu.png)

启动安装向导时，将看到包含以下选项的页：

![列出其他任务的页面](./media/active-directory-aadconnectsync-installation-wizard/additionaltasks.png)

如果已使用 Azure AD Connect 安装 ADFS，则会有更多选项。[ADFS management](/documentation/articles/active-directory-aadconnect-federation-management/)（ADFS 管理）中介绍了其他 ADFS 选项。

请选择其中一项任务，然后单击"下一步"继续。

> [AZURE.IMPORTANT] 当你打开安装向导时，同步引擎中的所有操作都会挂起。请确保在完成配置更改后，立即关闭安装向导。

## 查看当前配置
此选项可让你快速查看当前配置的选项。

![列出所有选项及其状态的页面](./media/active-directory-aadconnectsync-installation-wizard/viewconfig.png)

单击"上一步"可以后退。如果选择"退出"，将关闭安装向导。

## 自定义同步选项
此选项可用于更改同步配置。将在自定义配置安装路径中看到一部分选项。即使一开始是使用快速安装也会看到此选项。

- [添加更多目录](/documentation/articles/active-directory-aadconnect-get-started-custom/)。若要删除目录，请参阅[删除连接器](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-connectors/)。
- [更改域和 OU 筛选](/documentation/articles/active-directory-aadconnect-get-started-custom/)。
- 删除组筛选。
- [更改可选功能](/documentation/articles/active-directory-aadconnect-get-started-custom/)。

初始安装中的其他选项既无法更改，也不能使用。这些选项包括：

- 更改用于 userPrincipalName 和 sourceAnchor 的属性。
- 更改不同林中对象的联接方法。
- 启用基于组的筛选。

## 刷新目录架构
如果已更改其中一个本地 AD DS 林中的架构，则应使用此选项。例如，可能已安装 Exchange，或升级到包含设备对象的 Windows Server 2012 架构。在这种情况下，需指示 Azure AD Connect 从 AD DS 再次读取架构并更新其缓存。此操作还会重新生成同步规则。举例来说，如果你添加 Exchange 架构，配置中就会添加 Exchange 的同步规则。

选择此选项时，将列出配置中的所有目录。可以保留默认设置，并刷新所有林或取消选择其中某些林。

![列出环境中所有目录的页面](./media/active-directory-aadconnectsync-installation-wizard/refreshschema.png)

## 配置过渡模式
此选项可让你启用和禁用服务器上的过渡模式。过渡模式及其使用方式的详细信息可在[操作](/documentation/articles/active-directory-aadconnectsync-operations/)中找到。

此选项显示过渡模式当前是已启用还是已禁用：  
![同时显示过渡模式当前状态的选项](./media/active-directory-aadconnectsync-installation-wizard/stagingmodecurrentstate.png)

若要更改状态，请选择此选项，并选中或取消选中复选框。  
![同时显示过渡模式当前状态的选项](./media/active-directory-aadconnectsync-installation-wizard/stagingmodeenable.png)

## 更改用户登录
此选项可让你从密码同步更改为联盟，反之亦然。但不能更改为"不配置"。

有关此选项的详细信息，请参阅[用户登录](/documentation/articles/active-directory-aadconnect-user-signin/)。

## 后续步骤

- 在 [Understanding Declarative Provisioning](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning/)（了解声明性预配）中了解 Azure AD Connect 同步使用的配置模型。

**概述主题**

- [Azure AD Connect 同步：理解和自定义同步](/documentation/articles/active-directory-aadconnectsync-whatis/)
- [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

<!---HONumber=Mooncake_0926_2016-->
