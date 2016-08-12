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
   ms.date="06/01/2016"
   wacn.date="07/26/2016"/>

# 了解 Azure Active Directory (AD) 中的 OAuth2 隐式授权流

OAuth2 隐式授权是 OAuth2 规范中安全疑虑最多的授权方式，因此让人诟病。然而，这却是 ADAL JS 的实现方式，也是我们建议用于编写 SPA 应用程序的方法。这是怎么回事呢？ 不外乎是一种权衡利弊之后的结果：事实证明，对于通过 JavaScript 从浏览器使用 Web API 的应用程序而言，隐式授权是所能找到的最好方法。

## 什么是 OAuth2 隐式授权？

典型的 [OAuth2 授权代码授予](https://tools.ietf.org/html/rfc6749#section-1.3.1)是使用两个不同终结点的授权。在用户交互阶段使用授权终结点生成授权代码；然后，客户端使用令牌终结点来交换访问令牌（通常也是一个刷新令牌）的代码。Web 应用程序必须向令牌终结点提交自身的应用程序凭据，授权服务器才能对客户端进行身份验证。

[OAuth2 隐式授权](https://tools.ietf.org/html/rfc6749#section-1.3.2)定义为一种变体，其中，客户端可以直接从授权终结点获取访问令牌（若为 [OpenId Connect](http://openid.net/specs/openid-connect-core-1_0.html)，则获取 id\_token），而不必联系令牌终结点并验证客户端应用程序。此变体特别针对在 Web 浏览器中运行的基于 JavaScript 的应用程序所设计：在原始 OAuth2 规范中，令牌在 URI 片段中返回。这样，令牌位便可供客户端中的 JavaScript 代码使用，但可以保证令牌位不包含在朝向服务器的重定向中。通过浏览器重定向直接从授权终结点返回令牌还可带来一个好处：无需使用跨源调用。如果必须要有 JavaScript 应用程序才能联系令牌终结点，则需要跨源调用。

OAuth2 隐式授权的重要特征是，此类流程绝对不会将刷新令牌返回到客户端。如下一部分所述，这实际上是不必要的，反而会造成安全问题。

## OAuth2 隐式授权的适用方案

正如 OAuth2 规范本身所声明的，设计出隐式授权是为了实现用户代理应用程序，即在浏览器中执行的 JavaScript 应用。此类应用的鲜明特征是，JavaScript 代码可用于访问服务器资源（通常是 Web API）以及相应地更新应用程序 UX。以 Gmail 或 Outlook Web Access 之类的应用程序为例：当你在收件箱中选择某封邮件时，只有邮件可视化面板会更改以显示新的选择内容，页面的其余部分保持不变。这明显不同于传统的基于重定向的 Web 应用，在后者中，每个用户交互都会造成整页回传，并造成整页呈现新的服务器响应。

采用极端的基于 JavaScript 方法的应用程序称为单页应用程序 (SPA)：其思路是，这些应用程序只提供初始的 HTML 页和关联的 JavaScript，至于所有后续交互，则由通过 JavaScript 执行的 Web API 调用驱动。但是，应用大多是由回传驱动，但偶尔执行 JS 调用以实现某些体验的混合方法也并非罕见；关于隐式流用法的介绍也与这些方法有关。

基于重定向的应用程序通常通过 Cookie 来保护其请求；但是，这种方式并非也适用于 JavaScript 应用程序，因为 Cookie 只针对其生成域发生作用，但 JavaScript 调用却可能定向到其他域。事实上，情况往往是这样的：以调用 Microsoft 图形 API、Office API、Azure API 的应用程序为例 – 这些应用程序全都位于提供应用的域之外。JavaScript 应用程序的发展趋势是完全没有后端，全部依赖于第三方 Web API 来实现其业务功能。

目前，保护 Web API 调用的首选方法是使用 OAuth2 持有者令牌方法 – 每个调用都伴随一个 OAuth2 访问令牌；Web API 检查传入的访问令牌，如果它在令牌中找到所需的范围，便授予对请求操作的访问权限。隐式流提供了方便的机制供 JavaScript 应用程序获取 Web API 的访问令牌，并提供很多与 Cookie 相关的优点：

- 可以可靠地获取令牌而无需跨源调用 – 强制注册令牌要返回到的重定向 URI 可保证令牌不会转到其他位置
- JavaScript 应用可以针对任意数目的目标 Web API 获取所需数目的访问令牌 – 对域没有限制
- 会话或本地存储等 HTML5 功能可授予令牌缓存和生存期管理的完全控制权，但是 Cookie 管理对于应用而言是不透明的
- 访问令牌不容易遭受跨站点请求伪造 (CSRF) 攻击

隐式授权流不颁发刷新令牌，这主要是出于安全考虑。刷新令牌的范围不像访问令牌那么窄，前者授予更大的权力，因此万一泄露，将造成更大的损害。在隐式流中，令牌在 URL 中传递，因此遭到拦截的风险高于授权代码授予。

不过请注意，JavaScript 应用程序提供另一种可任其处置的机制，可用于续订访问令牌，且不会重复提示用户输入凭据。应用程序可以使用隐藏的 iframe 来针对 Azure AD 的授权终结点执行新的令牌请求：只要浏览器仍然针对 Azure AD 域提供活动会话（读取：有会话 Cookie），则身份验证请求就可以成功且不需要用户交互。

此模型向 JavaScript 应用授权，使其能够独立续订访问令牌，甚至获取新 API 的新访问令牌（前提是用户已同意），而不造成获取、维护和保护高值项目（例如刷新令牌）所带来的额外负担。实现无提示续订的项目（Azure AD 会话 Cookie）在应用程序外部进行管理。这种方法的另一个优点：当用户使用任何浏览器选项卡中运行的 Azure AD 从任一应用程序注销 Azure AD，从而导致 Azure AD 会话 Cookie 被删除时，JavaScript 应用程序将自动撤销已注销用户的续订令牌能力。

## 隐式授权适合我的应用吗？

隐式授权带来的风险高于其他授权。不同的文档介绍了需要注意的地方，例如，可以参阅 [Misuse of Access Token to Impersonate Resource Owner in Implicit Flow（在隐式流中误用访问令牌来模拟资源所有者）][OAuth2-Spec-Implicit-Misuse]和 [OAuth 2.0 Threat Model and Security Considerations（OAuth 2.0 威胁模型和安全注意事项）][OAuth2-Threat-Model-And-Security-Implications]。但是，风险走势之所以较高，主要是因为它要启用执行活动代码的应用程序，并由远程资源提供给浏览器。如果选择 SPA 体系结构，则不存在后端组件。一般而言，如果你想要通过 JavaScript 调用 Web API，则建议使用隐式流来获取令牌。

如果你的应用程序是本机客户端，则不适用隐式流 – 本机客户端环境中没有Azure AD 会话 Cookie，因此应用无法维持长期生存的会话，并且在获取新资源的访问令牌时，无法避免重复提示用户。

如果要开发包含后端的 Web 应用程序（意味着要从它的后端代码使用 API），则也不适合使用隐式流。其他授权提供的功能要大得多：例如，OAuth2 客户端凭据授权可让你获取令牌以反映分配给应用程序本身（而不是用户委托）的权限，即使用户不积极参与会话，你也能保持对资源的编程访问，等等。不仅如此，此类授权还能提供较高的安全保证：访问令牌永远不会通过用户浏览器来传输，它们存储在浏览器历史记录中，因此不会造成风险，等等；客户端应用程序可以在请求令牌时执行强式身份验证，等等。

## 后续步骤

- 有关开发人员资源的完整列表，包括 Azure AD 支持的整套协议和 OAuth2 授权流的参考信息，请参阅 [Azure AD Developer's Guide（Azure AD 开发人员指南）][AAD-Developers-Guide]
- 若要更深入了解应用程序集成过程，请参阅 [How to integrate an application with Azure AD（如何将应用程序与 Azure AD 集成）][ACOM-How-To-Integrate]。

<!--Image references-->
[Scenario-Topology]: ./media/active-directory-devhowto-auth-using-any-aad/multi-tenant-aad-components.png

<!--Reference style links in use-->
[AAD-Developers-Guide]: /documentation/articles/active-directory-developers-guide/
[ACOM-How-And-Why-Apps-Added-To-AAD]: /documentation/articles/active-directory-how-applications-are-added/
[ACOM-How-To-Integrate]: /documentation/articles/active-directory-how-to-integrate/
[OAuth2-Spec-Implicit-Misuse]: https://tools.ietf.org/html/rfc6749#section-10.16 
[OAuth2-Threat-Model-And-Security-Implications]: https://tools.ietf.org/html/rfc6819

<!---HONumber=Mooncake_0725_2016-->