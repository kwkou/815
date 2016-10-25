<properties
   pageTitle="了解 Azure Active Directory 中的 OAuth2 隐式授权流 | Azure"
   description="详细了解 Azure Active Directory 的 OAuth2 隐式授权流实现，以及它是否适合你的应用程序。"
   services="active-directory"
   documentationCenter="dev-center-name"
   authors="vibronet"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="08/17/2016"
   ms.author="vittorib;bryanla"
   wacn.date="10/11/2016"/>
   wacn.date="10/11/2016"/>

# 了解 Azure Active Directory (AD) 中的 OAuth2 隐式授权流

OAuth2 隐式授权是 OAuth2 规范中安全疑虑最多的授权方式，因此让人诟病。然而，这却是 ADAL JS 的实现方式，也是我们建议用于编写 SPA 应用程序的方法。这是怎么回事呢？ 不外乎是一种权衡利弊之后的结果：事实证明，对于通过 JavaScript 从浏览器使用 Web API 的应用程序而言，隐式授权是所能找到的最好方法。

## 什么是 OAuth2 隐式授权？

典型的 [OAuth2 授权代码授予](https://tools.ietf.org/html/rfc6749#section-1.3.1)是使用两个不同终结点的授权。授权终结点用于用户交互阶段，此阶段会生成授权代码。然后，客户端会通过令牌终结点用该代码交换访问令牌，通常还会交换刷新令牌。Web 应用程序必须向令牌终结点提交自身的应用程序凭据，授权服务器才能对客户端进行身份验证。

[OAuth2 隐式授权](https://tools.ietf.org/html/rfc6749#section-1.3.2)是一种变体，可以让客户端直接从授权终结点获取访问令牌（若为 [OpenId Connect](http://openid.net/specs/openid-connect-core-1_0.html)，则还会获取 id\_token），不必联系令牌终结点，也不必验证客户端应用程序。此变体特别针对在 Web 浏览器中运行的基于 JavaScript 的应用程序所设计：在原始 OAuth2 规范中，令牌在 URI 片段中返回。这样，令牌位便可供客户端中的 JavaScript 代码使用，但可以保证令牌位不包含在朝向服务器的重定向中。通过浏览器返回令牌即可直接从授权终结点重定向。另一优点是消除跨越调用的任何要求，在需要通过 JavaScript 应用程序联系令牌终结点的情况下，这些要求是必需的。

OAuth2 隐式授权的重要特征是，此类流程绝对不会将刷新令牌返回到客户端。如下一部分所述，这实际上是不必要的，反而会造成安全问题。

## OAuth2 隐式授权的适用方案

正如 OAuth2 规范本身所声明的，设计出隐式授权是为了实现用户代理应用程序，即在浏览器中执行的 JavaScript 应用程序。此类应用程序的鲜明特征是，JavaScript 代码可用于访问服务器资源（通常是 Web API）以及相应地更新应用程序 UX。以 Gmail 或 Outlook Web Access 之类的应用程序为例：当你在收件箱中选择某封邮件时，只有邮件可视化面板会更改以显示新的选择内容，页面的其余部分保持不变。这明显不同于传统的基于重定向的 Web 应用，在后者中，每个用户交互都会造成整页回传，并造成整页呈现新的服务器响应。

采用极端的基于 JavaScript 方法的应用程序称为单页应用程序 (SPA)：其思路是，这些应用程序只提供初始的 HTML 页和关联的 JavaScript，至于所有后续交互，则由通过 JavaScript 执行的 Web API 调用驱动。但是，应用程序大多是由回传驱动，但偶尔执行 JS 调用的混合方法也并非罕见；关于隐式流用法的介绍也与这些方法有关。

基于重定向的应用程序通常通过 Cookie 确保请求的安全性，但该方法不适用于 JavaScript 应用程序。Cookie 仅适用于生成 Cookie 时所针对的域，而 JavaScript 调用可能会定向到其他域。事实上，情况往往是这样的：以调用 Microsoft Graph API、Office API、Azure API 的应用程序为例 - 这些应用程序全都位于提供应用程序的域之外。JavaScript 应用程序的发展趋势是完全没有后端，全部依赖于第三方 Web API 来实现其业务功能。

当前情况下，若要保护对 Web API 的调用，首选方法是使用 OAuth2 持有者令牌方法，该方法的每个调用都伴随 OAuth2 访问令牌的使用。Web API 会检查传入的访问令牌，如果在其中发现所需的范围，则会授予对已请求操作的访问权限。隐式流提供了方便的机制供 JavaScript 应用程序获取 Web API 的访问令牌，并提供很多与 Cookie 相关的优点：

- 可以可靠地获取令牌而无需跨源调用 - 强制注册令牌要返回到的重定向 URI 可保证令牌不会转到其他位置
- JavaScript 应用程序可以针对任意数目的目标 Web API 获取所需数目的访问令牌 - 对域没有限制
- 会话或本地存储等 HTML5 功能可授予令牌缓存和生存期管理的完全控制权，但是 Cookie 管理对于应用而言是不透明的
- 访问令牌不容易遭受跨站点请求伪造 (CSRF) 攻击

隐式授权流不颁发刷新令牌，这主要是出于安全考虑。刷新令牌的范围不像访问令牌那么窄，前者授予更大的权力，因此万一泄露，将造成更大的损害。在隐式流中，令牌在 URL 中传递，因此遭到拦截的风险高于授权代码授予。

不过请注意，JavaScript 应用程序提供另一种可任其处置的机制，可用于续订访问令牌，且不会重复提示用户输入凭据。应用程序可以使用隐藏的 iframe 来针对 Azure AD 的授权终结点执行新的令牌请求：只要浏览器仍然针对 Azure AD 域提供活动会话（读取：有会话 Cookie），则身份验证请求就可以成功且不需要用户交互。

此模型授予 JavaScript 应用程序相关功能，允许其独立续订访问令牌，甚至允许其为新的 API 获取新的访问令牌，前提是用户此前已表示同意。这样可以避免因获取、维护和保护高价值项目（例如刷新令牌）而增加的负担。实现无提示续订的项目（Azure AD 会话 Cookie）在应用程序外部进行管理。此方法的另一优势是，用户可以通过登录到 Azure AD 的任意应用程序从 Azure AD（在任意浏览器标签页中运行）注销。这样会删除 Azure AD 会话 Cookie，而 JavaScript 应用程序会自动失去为已注销用户续订令牌的功能。

## 隐式授权适合我的应用吗？

隐式授权带来的风险高于其他授权。不同的文档介绍了需要注意的地方，例如，可以参阅 [Misuse of Access Token to Impersonate Resource Owner in Implicit Flow][OAuth2-Spec-Implicit-Misuse]（在隐式流中误用访问令牌来模拟资源所有者）和 [OAuth 2.0 Threat Model and Security Considerations][OAuth2-Threat-Model-And-Security-Implications]（OAuth 2.0 威胁模型和安全注意事项）。但是，风险走势之所以较高，主要是因为它要启用执行活动代码的应用程序，并由远程资源提供给浏览器。如果要规划一个 SPA 体系结构，则不要设置后端组件或尝试通过 JavaScript 调用 Web API，而应使用隐式流来获取令牌。

如果应用程序是本机客户端，则隐式流并不太适合。在使用本机客户端的情况下，如果没有 Azure AD 会话 Cookie，应用程序将无法长时间维持一个会话。这意味着，在为新资源获取访问令牌时，应用程序会反复提示用户。

如果要开发包含后端的 Web 应用程序，并需要从其后端代码使用 API，则也不适合使用隐式流。其他授权可以提供更强大的功能。例如，授予 OAuth2 客户端凭据即可获取的令牌能够反映分配给应用程序本身的权限，这不同于用户委派。这意味着，即使在用户未积极参与某个会话这样的情况下，客户端也可始终对资源进行程序性的访问。不仅如此，此类授权还提供更严格的安全保证。例如，访问令牌从不在用户浏览器中传输，因此不会有被保存在浏览器历史记录中的风险，诸如此类。在请求令牌时，客户端应用程序还可以进行严格的身份验证。

## 后续步骤

- 有关开发人员资源的完整列表，包括 Azure AD 支持的协议和 OAuth2 授权流的参考信息，请参阅 [Azure AD Developer's Guide][AAD-Developers-Guide]（Azure AD 开发人员指南）
- 若要更深入了解应用程序集成过程，请参阅 [How to integrate an application with Azure AD][ACOM-How-To-Integrate]（如何将应用程序与 Azure AD 集成）。

<!--Image references-->

<!--Reference style links in use-->
[AAD-Developers-Guide]: /documentation/articles/active-directory-developers-guide/
[ACOM-How-And-Why-Apps-Added-To-AAD]: /documentation/articles/active-directory-how-applications-are-added/
[ACOM-How-To-Integrate]: /documentation/articles/active-directory-how-to-integrate/
[OAuth2-Spec-Implicit-Misuse]: https://tools.ietf.org/html/rfc6749#section-10.16
[OAuth2-Threat-Model-And-Security-Implications]: https://tools.ietf.org/html/rfc6819

<!---HONumber=Mooncake_0926_2016-->
