大多时候，身份验证错误是由于不正确或不一致的配置设置。以下是有关要检查事项的一些具体建议。

* 确保在任何位置都未遗漏“保存”按钮。很容易出现这样的疏忽，结果是，你在门户网站上看到正确的值，但实际上它们并未保存到 Azure 环境或 Azure AD 应用程序中。
* 对于在 Azure 门户的“应用程序设置”边栏选项卡中配置的设置，请确保输入设置时选择正确的 API 应用或 Web 应用。还要确保设置输入为“应用设置”而不是“连接字符串”，因为这两个部分的格式相似。
* 对于 JavaScript 前端的身份验证，请再次下载清单以确认 `oauth2AllowImplicitFlow` 已成功更改为 `true`。
* 请验证在任何位置配置 URL 时都使用了 HTTPS：
  
  * 在项目代码中
  * 在 CORS 中
  * 在每个 API 应用和 Web 应用的 Azure 环境应用设置中
  * 在 Azure AD 应用程序设置中。
    
    请注意，如果从门户复制 API 应用的 URL，它通常会具有 `http://`，必须手动将其更改为 `https://`。
* 请确保已成功部署任何代码更改。例如，在多项目解决方案中，可能会更改项目代码，并且当打算部署更改时可能会不小心选择其他代码中的一个。
* 请确保你将在浏览器中前往 HTTPS URL，而不是 HTTP URL。默认情况下，Visual Studio 使用 HTTP URL 创建发布配置文件，这是部署项目后在浏览器中打开的内容。
* 对于 JavaScript 前端身份验证，确保已在 JavaScript 代码所调用的 API 应用上正确配置 CORS。如果不确定问题是否与 CORS 相关，请尝试将 "\*" 作为允许的源 URL。
* 对于 JavaScript 前端，打开浏览器的“开发人员工具控制台”选项卡，以获取有关错误的详细信息，并检查网络上的 HTTP 请求。但是，控制台的错误消息可能会产生误导。如果收到一条指示 CORS 错误的消息，真正的问题可能会是身份验证。可通过运行暂时禁用了身份验证的应用来检查是否是身份验证的问题。
* 对于 .NET API 应用，请通过将 [customErrors 模式设置为“关闭”](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio/#remoteview)，确保从错误消息中获得尽可能多的信息。
* 对于 .NET API 应用，启动[远程调试会话](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio/#remotedebug)，并检查传递给后列代码的变量的值：使用 ADAL 来获取持有者令牌的代码，或对照预期服务主体 ID 进行声明检查的代码。请注意，你的代码可以获取来自许多不同源的配置值，因此通过这种方式很可能会发现意外内容。例如，配置 Azure 应用服务环境设置时，如果将 `ida:ClientId` 错误地键入为 `ida:ClientID`，代码可能会获取其在 Web.config 文件中寻找的 `ida:ClientId` 值，并忽略 Azure App Service 设置。
* 如果一切尝试在正常的 Internet Explorer 窗口中不起作用，可能是现有登录正在造成干扰；请尝试 InPrivate，然后尝试 Chrome 或 Firefox。

<!---HONumber=Mooncake_1128_2016-->