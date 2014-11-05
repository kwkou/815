<properties title="Active Directory 身份验证入门" pageTitle="" metaKeywords="Azure, Getting Started, Active Directory" description="" services="active-directory" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="active-directory" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb"></tags>

> [AZURE.SELECTOR]
>
> -   [入门][入门]
> -   [发生了什么情况][发生了什么情况]

## Azure Active Directory 入门（Web API 项目）

##### 访问控制器需要身份验证

您项目中的所有控制器均带有 **Authorize** 属性。此属性要求用户先进行身份验证，然后才能访问由这些控制器定义的 API。若要允许匿名访问控制器，请从控制器删除此属性。如果您想要更详细地设置这些权限，请将该属性应用到需要身份验证的每个方法，而不是将它应用到控制器类。

[详细了解 Azure Active Directory][详细了解 Azure Active Directory]

  [入门]: /documentation/articles/vs-active-directory-webapi-getting-started/
  [发生了什么情况]: /documentation/articles/vs-active-directory-webapi-what-happened/
  [详细了解 Azure Active Directory]: http://azure.microsoft.com/services/active-directory/
