

默认情况下，对移动服务资源的所有请求仅限于提供了应用程序密钥的客户端，这并不能严格保护对资源的访问。为了保护资源安全，您必须将访问权限仅限于经过身份验证的客户端。

1. 在 Visual Studio 中，打开包含您的移动服务的项目。 

2. 在"解决方案资源管理器"中，展开 Controllers 文件夹，并打开 TodoItemController.cs 项目文件。

	**TodoItemController** 类可为 TodoItem 表实现数据访问。 

3. 在代码页顶部添加以下  `using` 语句：

		using Microsoft.WindowsAzure.Mobile.Service.Security;

4. 将以下 AuthorizeLevel 属性应用于 **TodoItemController** 类：

		[AuthorizeLevel(AuthorizationLevel.User)] 

	这样可以确保对 **TodoItem** 表的所有操作都要求用户经过身份验证。 

	>[WACOM.NOTE]将 AuthorizeLevel 属性应用于单个方法，以便在控制器公开的方法上设置特定授权级别。

5. 如果要从本地调试身份验证，请展开 App_Start 文件夹、打开 WebApiConfig.cs 项目文件，然后将以下代码添加到 **Register** 方法：

		config.SetIsHosted(true);
	
	这样可以通知本地移动服务项目按照 Azure 中托管的方式运行，包括遵循 AuthorizeLevel 设置。没有此设置，发送到  *localhost* 的所有 HTTP 请求都会在未经身份验证的情况下得到允许，而不受 AuthorizeLevel 设置影响。  

6. 重新发布您的服务项目。

<!--HONumber=41-->
