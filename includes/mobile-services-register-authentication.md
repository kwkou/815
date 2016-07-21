
1. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中，单击“移动服务”> 你的移动服务 >“仪表板”，并记下“移动服务 URL”值。

2. 使用一个或多个以下身份验证提供者注册你的应用程序：
   * [Microsoft](/documentation/articles/mobile-services-how-to-register-microsoft-authentication/)
   * [Azure Active Directory](/documentation/articles/mobile-services-how-to-register-active-directory-authentication/)。 
   
	记下标识提供者生成的客户端标识和客户端机密值。不要分发或共享客户端机密。

3. 返回 [Azure 管理门户](https://manage.windowsazure.cn/)，单击“移动服务”> 你的移动服务 >“身份”> 你的标识提供者设置，然后输入你的提供者的客户端 ID 和密钥值。 
 
现在，你已将应用和移动服务配置为使用你的身份验证提供者。可以选择对你想要支持的其他每个标识提供者重复所有这些步骤。

> [AZURE.IMPORTANT]确认已在标识提供者的开发人员站点上设置了正确的重定向 URI。如上述每个提供者的链接说明中所述，.NET 后端服务与 JavaScript 后端服务的重定向 URI 可能有所不同。配置不正确的重定向 URI 可能会导致登录屏幕显示不正常，并且应用发生意外的故障。

<!---HONumber=Mooncake_0118_2016-->