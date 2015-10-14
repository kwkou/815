
* 单击“Azure 门户”>“移动服务”> 你的移动服务 >“仪表板”，并记下“移动服务 URL”值。

* 将你的应用注册到 [Google](/documentation/articles/mobile-services-how-to-register-google-authentication)、[Facebook](/documentation/articles/mobile-services-how-to-register-facebook-authentication)、[Twitter](/documentation/articles/mobile-services-how-to-register-twitter-authentication)、[Microsoft](/documentation/articles/mobile-services-how-to-register-microsoft-authentication) 或 [Azure Active Directory](/documentation/articles/mobile-services-how-to-register-active-directory-authentication)。记下标识提供者生成的客户端标识和客户端机密值。不要分发或共享客户端机密。

* 单击“Azure 门户”>“移动服务”> 你的移动服务 >“标识” > 你的标识提供者设置。输入提供者中的应用 ID 和机密值。现在，你已将应用和移动服务配置为使用你的身份验证提供者。可以选择对你想要支持的其他每个标识提供者重复所有这些步骤。

    > [AZURE.IMPORTANT]确认已在标识提供者的开发人员站点上设置了正确的重定向 URI。如上述每个提供者的链接说明中所述，.NET 后端服务与 JavaScript 后端服务的重定向 URI 可能有所不同。配置不正确的重定向 URI 可能会导致登录屏幕显示不正常，并且应用发生意外的故障。

<!---HONumber=71-->