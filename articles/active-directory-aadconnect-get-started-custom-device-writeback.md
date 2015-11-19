<properties 
	pageTitle="在 Azure AD Connect 中启用设备写回" 
	description="本文档详细说明如何使用 Azure AD Connect 启用设备写回功能" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="msStevenPo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="09/15/2015"
	wacn.date="11/02/2015"/>

# 在 Azure AD Connect 中启用设备写回

以下文档提供有关如何在 Azure AD Connect 中启用设备写回功能的信息。设备写回用于以下方案：

对 ADFS（2012 R2 或更高版本）保护的应用程序（信赖方信任），启用基于设备的条件性访问。

这可以提供额外的安全性，确保只有受信任的设备才能访问应用程序。有关条件性访问的详细信息，请参阅[使用条件性访问管理风险](/documentation/articles/active-directory-conditional-access)和[使用 Azure Active Directory Device Registration 设置本地的条件性访问](https://msdn.microsoft.com/library/azure/dn788908.aspx)。

>[AZURE.Note]使用在 Azure Active Directory Device Registration 服务条件性访问策略中注册的设备时，需要 Office 365 或 Azure AD Premium 的订阅。这包括由 Active Directory 联合身份验证服务 (AD FS) 对本地资源强制实施的策略。

## 第 1 部分：准备 Azure AD Connect
使用以下步骤来准备使用设备写回。

1.	从已安装 Azure AD Connect 的计算机上，以权限提升模式启动 PowerShell。

2.	如果未安装 Active Directory PowerShell 模块。使用以下命令安装它：

	Install-WindowsFeature –Name AD-DOMAIN-Services –IncludeManagementTools

3.	使用企业管理员凭据运行以下命令，然后退出 PowerShell。

	Import-Module ‘C:\Program Files\Microsoft Azure Active Directory Connect\AdPrep\AdSyncAdPrep.psm1’

	Initialize-ADSyncDeviceWriteback –DomainName <name> -AdConnectorAccount <account>

说明:



- 该命令将在 CN=Device Registration Configuration,CN=Services,CN=Configureation,<forest-dn> 下创建并配置新的容器和对象（如果不存在）。



- 该命令将在 CN=RegisteredDevices,<domain-dn> 下创建并配置新的容器和对象（如果不存在）。将在此容器中创建设备对象。



- 在 Azure AD 连接器帐户中设置必要的权限，以便管理 Active Directory 上的设备。



- 即使 Azure AD Connect 安装在多个林中，也只需要在一个林中运行。

参数：


- DomainName：将在其中创建设备对象的 Active Directory 域。注意：给定的 Active Directory 林的所有设备都在单个域中创建。 


- AdConnectorAccount：Azure AD Connect 使用此 Active Directory 帐户来管理目录中的对象。

## 第 2 部分：启用设备写回
使用以下过程在 Azure AD Connect 中启用设备写回。

1.	运行 AAD Connect 向导。如果这是你第一次使用该向导，请从“快速设置”屏幕中选择“自定义”来执行自定义安装 
![自定义安装](./media/active-directory-aadconnect-get-started-custom-device-writeback/devicewriteback1.png)
2.	如果不是第一次使用该向导，请从“其他任务”页中选择自定义同步选项，然后单击“下一步”。
![自定义安装](./media/active-directory-aadconnect-get-started-custom-device-writeback/devicewriteback2.png)
3.	在“可选功能”页中，设备写回不再灰显。请注意，如果 Azure AD Connect 准备步骤未完成，“可选功能”页中的设备写回将会灰显。选中设备写回对应的框并单击“下一步”。
![设备写回](./media/active-directory-aadconnect-get-started-custom-device-writeback/devicewriteback3.png)
4.	在写回页中，你会看到提供的域是默认的设备写回林。
![自定义安装](./media/active-directory-aadconnect-get-started-custom-device-writeback/devicewriteback4.png)
5.	在向导中完成安装，不需要更改其他配置。如果需要，请参阅 [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom)。



## 启用条件性访问
[使用 Azure Active Directory Device Registration 设置本地条件性访问](https://msdn.microsoft.com/library/azure/dn788908.aspx)中提供了有关启用此方案的详细说明。

## 验证设备是否已同步到 Active Directory
设备写回现在应在正常运行。请注意，将设备对象写回到 Active Directory 最长可能需要 3 个小时。若要验证设备是否已正确同步，请在同步规则完成之后执行以下操作：
 
1.	启动 Active Directory 管理中心。 
2.	在要联合的域中展开 RegisteredDevices。
![自定义安装](./media/active-directory-aadconnect-get-started-custom-device-writeback/devicewriteback5.png)
3.	其中将会列出当前已注册的设备。 

![自定义安装](./media/active-directory-aadconnect-get-started-custom-device-writeback/devicewriteback6.png)

## 其他信息 


- [使用条件性访问管理风险](/documentation/articles/active-directory-conditional-access)
<!-- 
- [使用 Azure Active Directory Device Registration 设置本地条件性访问](/documentation/articles/active-directory-conditional-access-on-premises-setup)
-->
<!---HONumber=76-->