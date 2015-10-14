

默认情况下，对移动服务资源的所有请求仅限于提供了应用程序密钥的客户端，这并不能严格保护对资源的访问。为了保护资源安全，您必须将访问权限仅限于经过身份验证的客户端。

1. 在 Visual Studio 中打开移动服务项目，展开 Controllers 文件夹，然后打开 **TodoItemController.cs**。**TodoItemController** 类可为 TodoItem 表实现数据访问。添加以下 `using` 语句：



		using Microsoft.WindowsAzure.Mobile.Service.Security;

2. 将以下 AuthorizeLevel 属性应用于 **TodoItemController** 类：

		[AuthorizeLevel(AuthorizationLevel.User)] 

	这样可以确保对 _TodoItem_ 表的所有操作都要求用户经过身份验证。你也可以在方法级别应用 *AuthorizeLevel* 属性。

3. （可选）如果要从本地调试身份验证，请展开 `App_Start` 文件夹，打开 **WebApiConfig.cs**，然后将以下代码添加到 **Register** 方法。

		config.SetIsHosted(true);

	这样可以通知本地移动服务项目按照 Azure 中托管的方式运行，包括遵循 *AuthorizeLevel* 设置。没有此设置，发送到 localhost 的所有 HTTP 请求都会在未经身份验证的情况下得到允许，而不受 *AuthorizeLevel* 设置影响。当你启用自我托管模式时，还需要为本地应用程序密钥设置值。

4. （可选）在 web.config 项目文件中，设置 `MS_ApplicationKey` 应用设置的字符串值。

	这是当你在本地运行服务时用来测试 API 帮助页的密码（不结合用户名）。Azure 中的实时站点不使用此字符串值，并且你不需要使用实际的应用程序密钥；任何有效的字符串值都是可行的。
 
5. 重新发布你的项目。

<!---HONumber=71-->