<properties 
	pageTitle="使用 PowerShell 批量删除 Azure Active Directory 用户信息" 
	description="使用 PowerShell 导出、导入 .csv 文件的方式批量删除 Azure Active Directory 测试用户信息" 
	service="microsoft.activedirectory"
	resource="activedirectory"
	authors=""
	displayOrder=""
	selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Azure AD, PowerShell"​
    cloudEnvironments="MoonCake" 
/>
<tags 
	ms.service="active-directory-aog"
	ms.date="" 
	wacn.date="01/12/2017"
/>
# 使用 PowerShell 批量删除 Azure Active Directory 用户信息

## **问题描述**

在使用 Azure Active Directory (AD) 测试单点登陆时，往往会上传大量的用户信息，这就造成了在 Azure AD 中有大量的测试用户，当测试完成后如何快速删除这些测试用户是很多用户会遇到的问题。

## **问题分析**

删除用户的方法主要有以下三种 :

1.	在门户中单个删除。但是如果用户量特别大则不能满足。
2.	在本地 AD 中删除了多个用户信息, 那么再次上传时 Azure AD 中也会删除相应的用户信息。但是如果本地 AD 是生产环境, 那么该方法是不可行的。
3.	通过 PowerShell 批量导出用户信息保存到本地，在文件中批量删除用户信息后上传，那么 Azure AD 中的用户信息也会被删除。总体来说这种方式是最适合的，既方便又不会对客户的生产环境造成影响。 

## **解决方法**

首先您需要下载两个程序并安装：

1.	[Microsoft Online Services Sign-In Assistant for IT Professionals RTW](https://www.microsoft.com/zh-cn/download/details.aspx?id=41950)
2.	[Azure Active Directory Module for Windows PowerShell (64-bit version)](http://go.microsoft.com/fwlink/p/?linkid=236297)

安装完成之后按如下步骤执行：

1.	执行 Windows Azure Active Directory 模块生成的用于 Windows PowerShell 的快捷方式。

2.	执行 `$msolcred = get-credential` 命令，输入您目录中有管理权限的用户名和密码。

3.	使用 `connect-msolservice -credential $msolcred` 命令连接到 Azure AD。

4.	通过 `Get-MsolUser –All | Export-Csv C:\users.csv`  导出 AD 用户信息到本地 .csv 文件中。导出后修改此文件，移除不需要的用户信息。

5.	执行 `Import-Csv C:\users.csv | Remove-MsolUser –Force`   通过 .csv 文件删除 Azure AD 上的用户。

执行完以后可以到管理门户中查看用户信息是否已被删除。



