### <a name="server-auth"></a>如何：使用提供程序（服务器流）进行身份验证
若要让移动应用管理应用中的身份验证过程，必须向标识提供者注册应用。然后，需要在 Azure App Service 中配置提供者提供的应用程序 ID 和机密。有关详细信息，请参阅[向应用程序添加身份验证](/documentation/articles/app-service-mobile-cordova-get-started-users/)教程。

注册标识提供者后，只需使用提供者的名称调用 .login() 方法即可。例如，若要使用 microsoftaccount 登录，请使用以下代码。


	client.login("microsoftaccount").done(function (results) {
	     alert("You are now logged in as: " + results.userId);
	}, function (err) {
	     alert("Error: " + err);
	});


如果使用的标识提供程序不是 microsoftaccount，请将传递给上述 login 方法的值更改为 `aad`。

在这种情况下，由 Azure 应用服务管理 OAuth 2.0 身份验证流程。它显示所选提供者的登录页，并在使用标识提供者成功登录后生成应用服务身份验证令牌。login 函数在完成时返回一个 JSON 对象，该对象分别在 userId 和 authenticationToken 字段中公开用户 ID 和应用服务身份验证令牌。可以缓存此令牌，并在它过期之前重复使用。

###<a name="client-auth"></a>如何：使用提供程序（客户端流）进行身份验证

你的应用还能够独立联系标识提供者，然后将返回的令牌提供给应用服务以进行身份验证。使用此客户端流可为用户提供单一登录体验，或者从标识提供者中检索其他用户数据。


#### Microsoft 帐户示例

以下示例使用 Live SDK，该 SDK 使用 Microsoft 帐户来支持 Windows 应用商店应用程序的单一登录：


	WL.login({ scope: "wl.basic"}).then(function (result) {
	      client.login(
	            "microsoftaccount",
	            {"authenticationToken": result.session.authentication_token})
	      .done(function(results){
	            alert("You are now logged in as: " + results.userId);
	      },
	      function(error){
	            alert("Error: " + err);
	      });
	});


这个示例将从 Live Connect 获取一个令牌，并通过调用 login 函数将该令牌提供给你的应用服务。

###<a name="auth-getinfo"></a>如何：获取已经过身份验证的用户相关信息

可以使用具有任何 AJAX 库的 HTTP 调用从 `/.auth/me` 终结点检索身份验证信息。确保将 `X-ZUMO-AUTH` 标头设置为身份验证令牌。身份验证令牌存储在 `client.currentUser.mobileServiceAuthenticationToken` 中。例如，若要使用提取 API：


	var url = client.applicationUrl + '/.auth/me';
	var headers = new Headers();
	headers.append('X-ZUMO-AUTH', client.currentUser.mobileServiceAuthenticationToken);
	fetch(url, { headers: headers })
	    .then(function (data) {
	        return data.json()
	    }).then(function (user) {
	        // The user object contains the claims for the authenticated user
	    });


提取可作为 [npm 包](https://www.npmjs.com/package/whatwg-fetch)，或供浏览器从 [CDNJS](https://cdnjs.com/libraries/fetch) 进行下载。也可以使用 jQuery 或其他 AJAX API 提取信息。数据作为 JSON 对象接收。

<!---HONumber=Mooncake_0116_2017-->