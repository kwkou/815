<properties 
	pageTitle="什么是 Azure Multi-Factor Authentication？| Azure"
	description="本主题介绍什么是 Multifactor Authentication (MFA)、为何使用 MFA，以及有关 Multi-factor Authentication 客户端和不同方法和可用版本的详细信息。Azure Multi-Factor Authentication 是要求使用多种方式（而不仅仅是用户名和密码）对你的身份进行验证的一种方法。它为用户登录和事务提供了附加的安全层。"
	keywords="MFA 简介, mfa 概述, 什么是 mfa"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="billmath"
	manager="stevenpo"
	editor="curtland"/>

<tags 
	ms.service="multi-factor-authentication" 
	ms.date="05/12/2016"
	wacn.date="05/18/2016"/>

# 什么是 Azure 多重身份验证？
MFA(多重身份验证)是需要使用多个验证方法的身份验证方法，为用户登录和事务额外提供一层重要的安全保障。它需要以下验证方法中的两种或更多种来进行工作：

- 你知道的某样东西（通常为密码）
- 你具有的某样东西（无法轻易复制的可信设备，如电话）
- 你自身的特征（生物辨识系统）

<center>![Username and Password](./media/multi-factor-authentication/pword.png) &#160;&#160;&#160;&#160;&#160;![Certificates](./media/multi-factor-authentication/phone.png) &#160;&#160;&#160;&#160;&#160;![Smart Phone](./media/multi-factor-authentication/hware.png) &#160;&#160;&#160;&#160;&#160;![Smart Card](./media/multi-factor-authentication/smart.png) &#160;&#160;&#160;&#160;&#160;![Virtual Smart Card](./media/multi-factor-authentication/vsmart.png) &#160;&#160;&#160;&#160;&#160;![Username and Password](./media/multi-factor-authentication/cert.png)</center>



Azure 多重身份验证是要求使用多种方式（而不仅仅是用户名和密码）对你的身份进行验证的一种方法。它为用户登录和事务提供了附加的安全层。

Azure 多重身份验证可帮助保护对数据和应用程序的访问，同时可以满足用户对简单登录过程的需求。它通过各种简单的验证选项（例如电话、短信、移动应用通知或验证码）来提供强大的身份验证。

有关 Azure 多重身份验证工作原理的概述，请观看以下视频。

##为何使用 Azure 多重身份验证？

##为何使用 Azure Multi-Factor Authentication？

与以往相比，联网的用户越来越长。通过智能手机、平板电脑、笔记本电脑和台式个人电脑，人们可以使用各种不同的选项随时连接网络和保持联系。人们可以从任何位置访问他们的帐户与应用程序，这意味着，他们可以提高工作效率并为客户提供更好的服务。

Azure 多重身份验证是一个易于使用、可缩放且可靠的解决方案，可提供另一种身份验证方法，使你的用户永远受到保护。

![易用](./media/multi-factor-authentication/simple.png)| ![可缩放](./media/multi-factor-authentication/scalable.png)| ![始终受保护](./media/multi-factor-authentication/protected.png)|![可靠](./media/multi-factor-authentication/reliable.png)
:-------------: | :-------------: | :-------------: | :-------------: |
**易用**|**可缩放**|**始终受保护**|**可靠**

- **易用** - Azure 多重身份验证的设置和使用都很方便。Azure 多重身份验证提供额外的保护功能，可让用户使用和管理他们自己的设备，并且在许多情况下，只要单击几下鼠标就能完成设置。
- **可缩放** - Azure 多重身份验证利用云技术并与你的本地 AD 和自定义应用相集成,这种保护甚至可以延伸到高事务量的任务关键型方案。
- **始终受保护** - Azure 多重身份验证使用最高行业标准提供强大的身份验证功能。
- **可靠** - 我们保证 Azure 多重身份验证的可用性达到 99.9%。当服务无法接收或处理多重身份验证的身份验证请求时，即会将服务视为不可用。  

<!--For additional information on why use Azure Multi-Factor Authentication see the following video.

<center>[AZURE.VIDEO windows-azure-multi-factor-authentication]</center>-->

**其他资源**

* [为 Office 365 设置多重身份验证](https://support.office.com/zh-cn/article/%e8%ae%be%e7%bd%ae-Office-365-%e7%9a%84%e5%a4%9a%e5%9b%a0%e7%b4%a0%e8%ba%ab%e4%bb%bd%e9%aa%8c%e8%af%81-8f0454b2-f51a-4d9c-bcde-2c48e41621c6?omkt=zh-CN&ui=zh-CN&rs=zh-CN&ad=CN)
* [多重身份验证对我而言有什么用途？](/documentation/articles/multi-factor-authentication-end-user/)

<!---HONumber=Mooncake_0530_2016-->