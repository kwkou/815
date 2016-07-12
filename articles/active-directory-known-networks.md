<properties 
	pageTitle="已知网络 | Azure" 
	description="配置已知网络后，你可以避免将你组织拥有的 IP 地址包含在“从多个地理区域登录”和“从具有可疑活动的 IP 地址登录”报告中。" 
	services="active-directory" 
	documentationCenter="" 
	authors="markusvi" 
	manager="stevenpo"  
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="05/03/2016" 
	wacn.date="06/23/2016"/>

# 已知网络


你可以使用 Azure Active Directory 的访问和使用情况报告来监控你所在组织的目录的完整性和安全性。使用此信息，目录管理员可以更好地确定哪里可能存在安全风险，以便制定相应的计划来降低风险。

“从多个地理区域登录”和“从具有可疑活动的 IP 地址登录”可能会错误地报告你组织实际拥有的标志 IP 地址。

例如，在以下情况时可能会发生这种问题：

- 在波士顿办公室的用户远程登录到旧金山的数据中心，从而触发了“从多个地理区域登录”报告 

- 组织中的用户多次以错误的密码尝试登录，从而触发了“从具有可疑活动的 IP 地址登录”报告

若要防止在这种情况下生成误导性的安全报告，你应该在组织的公共 IP 地址列表中添加已知的 IP 地址范围。


###若要添加组织的公共 IP 地址范围，请执行以下步骤： 

1.	登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2.	在左窗格中，单击“Active Directory”。<br><br>![云应用程序发现的工作原理](./media/active-directory-known-networks/known-netwoks-01.png)

3.	在“目录”选项卡中，选择你的目录。

4.	在顶部菜单中，单击“配置”。<br><br>![云应用程序发现的工作原理](./media/active-directory-known-networks/known-netwoks-02.png)

5.	在“配置”选项卡上，转到“组织公共 IP 地址范围”<br><br>![云应用程序发现的工作原理](./media/active-directory-known-networks/known-netwoks-03.png)

6.	单击“添加已知的 IP 地址范围”。

7.	在出现的对话框中添加地址范围，然后在完成时单击选中按钮。<br><br>![云应用程序发现的工作原理](./media/active-directory-known-networks/known-netwoks-04.png)


**其他资源**


* [查看访问和使用情况报告](/documentation/articles/active-directory-view-access-usage-reports/)
* [从具有可疑活动的 IP 地址登录](/documentation/articles/active-directory-reporting-sign-ins-from-ip-addresses-with-suspicious-activity/)
* [来自多个地理区域的登录](/documentation/articles/active-directory-reporting-sign-ins-from-multiple-geographies/)



<!---HONumber=Mooncake_0613_2016-->