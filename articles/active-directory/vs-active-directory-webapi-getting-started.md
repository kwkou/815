<properties
    pageTitle="开始使用 Azure Active Directory 和 Visual Studio 连接服务（WebApi 项目）| Azure"
    description="通过 Visual Studio 连接服务连接到或创建 Azure AD 之后，如何在 WebApi 项目中开始使用 Azure Active Directory"
    services="active-directory"
    documentationcenter=""
    author="TomArcher"
    manager="douge"
    editor="" />  

<tags
    ms.assetid="bf1eb32d-25cd-4abf-8679-2ead299fedaa"
    ms.service="active-directory"
    ms.workload="web"
    ms.tgt_pltfrm="vs-getting-started"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/18/2016"
    wacn.date="01/05/2017"
    ms.author="tarcher" />

# 开始使用 Azure Active Directory 和 Visual Studio 连接服务（WebApi 项目）

> [AZURE.SELECTOR]
> - [入门](/documentation/articles/vs-active-directory-webapi-getting-started/)
> - [发生了什么情况](/documentation/articles/vs-active-directory-webapi-what-happened/)

## 访问控制器需要身份验证
您项目中的所有控制器均带有 **Authorize** 属性。此属性要求用户先进行身份验证，然后才能访问由这些控制器定义的 API。若要允许匿名访问控制器，请从控制器删除此属性。如果您想要更详细地设置这些权限，请将该属性应用到需要身份验证的每个方法，而不是将它应用到控制器类。

[详细了解 Azure Active Directory](/home/features/identity/)

<!---HONumber=Mooncake_1226_2016-->
