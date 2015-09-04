<properties 
	pageTitle="我如何发现我的组织内使用的未经认可的云应用" 
	description="本主题介绍 Cloud App Discvery 的定义和用途。" 
	services="active-directory" 
	documentationCenter="" 
	authors="markusvi" 
	manager="swadhwa" 
	editor="lisatoft"/>

<tags 
	ms.service="active-directory" 
	ms.date="07/23/2015" 
	wacn.date="08/29/2015"/>

# 我如何发现我的组织内使用的未经认可的云应用

在现代企业中，IT 部门通常不了解用户执行其工作所使用的所有云应用程序。因此，管理员经常担忧公司数据的未授权访问、潜在的数据泄漏和应用程序固有的其他安全风险。因为他们不知道使用了多少或哪些应用，甚至连开始制定应对这些风险的计划似乎都是困难的。

你可通过使用 Cloud App Discovery 解决这些问题。Cloud App Discovery 是 Azure Active Directory 的高级功能，借此可发现组织中的员工所使用的云应用程序。


**通过 Cloud App Discovery，你可以：**

* 发现正在使用的应用程序，并通过用户数量、流量大小或对应用程序的 Web 请求数量来测量使用情况。 
* 确定正在使用应用程序的用户。 
* 导出数据用于其他脱机分析。 
* 确定要纳入 IT 控制的应用程序的优先顺序，轻松整合应用程序，以实现单一登录和用户管理。 

通过 Cloud App Discovery，数据检索部分由你环境中的计算机上运行的代理完成。由代理捕获的应用使用情况信息经安全加密通道传送到 Cloud App Discovery 服务。Cloud App Discovery 服务评估数据，并生成可用于分析你的环境的报告。


<center>![Cloud App Discovery 工作原理](./media/active-directory-cloudappdiscovery/cad01.png)</center>

##后续步骤


* 要深入了解 Cloud App Discovery 工作原理，请参见 [Cloud App Discovery 入门](http://social.technet.microsoft.com/wiki/contents/articles/30962.getting-started-with-cloud-app-discovery.aspx) 
* 有关安全和隐私注意事项，请参阅 [Cloud App Discovery 安全和隐私注意事项](/documentation/articles/active-directory-cloudappdiscovery-security-and-privacy-considerations) 
* 有关在以下企业环境部署 Cloud App Discovery 代理的信息： 
 * 对于使用 Active Directory 组策略管理的企业环境，请参阅 [Active Directory 组策略管理部署指南](http://social.technet.microsoft.com/wiki/contents/articles/30965.cloud-app-discovery-group-policy-deployment-guide.aspx) 
 * 对于使用 Microsoft System Center Configuration Manager 的企业环境，请参阅 [Cloud App Discovery System Center 部署指南](http://social.technet.microsoft.com/wiki/contents/articles/30968.cloud-app-discovery-system-center-deployment-guide.aspx) 
 * 对于使用带自定义端口的代理服务器的企业环境，请参阅[针对带自定义端口的代理服务器的 Cloud App Discovery 注册表设置](/documentation/articles/active-directory-cloudappdiscovery-registry-settings-for-proxy-services) 





**其他资源**


* [Cloud App Discovery - 代理更改日志](http://social.technet.microsoft.com/wiki/contents/articles/24616.cloud-app-discovery-agent-changelog.aspx)
* [Cloud App Discovery - 常见问题](http://social.technet.microsoft.com/wiki/contents/articles/24037.cloud-app-discovery-frequently-asked-questions.aspx)

<!---HONumber=67-->