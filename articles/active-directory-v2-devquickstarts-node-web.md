<properties
	pageTitle="Azure AD v2.0 NodeJS Web 应用 | Microsoft Azure"
	description="如何构建一个使用个人 Microsoft 帐户和工作或学校帐户来登录用户的 Node JS Web 应用。"
	services="active-directory"
	documentationCenter="nodejs"
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/28/2016"/>

# 将登录凭据添加到 nodeJS Web 应用


> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。


我们将在此处使用 Passport 来执行以下操作：

- 使用 Azure AD 和 v2.0 终结点将用户登录到应用。
- 显示有关用户的一些信息。
- 从应用中注销用户。

**Passport** 是 Node.js 的身份验证中间件。Passport 极其灵活并且采用模块化结构，可以在不造成干扰的情况下放入任何基于 Express 的应用程序或 Resitify Web 应用程序。一套综合性策略支持使用用户名和密码、Facebook、Twitter 等进行身份验证。我们针对 Microsoft Azure Active Directory 开发了一个策略。我们将安装此模块，然后添加 Microsoft Azure Active Directory `passport-azure-ad` 插件。

## 下载

本教程的代码[在 GitHub 上](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs)维护。若要遵照该代码，你可以[下载 .zip 格式应用骨架](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs/archive/skeleton.zip)，或克隆该骨架：

		git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs.git

本教程末尾也提供完成的应用程序。

## 1\.注册应用程序
在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 中创建新的应用程序，或遵循以下[详细步骤](/documentation/articles/active-directory-v2-app-registration/)。请确保：

- 复制分配给应用程序的**应用程序 ID**，因为稍后将要用到。
- 为应用程序添加 **Web** 平台。
- 输入正确的**重定向 URI**。重定向 URI 向 Azure AD 指示身份验证响应应定向到的位置，本教程的默认值为 `http://localhost:3000/auth/openid/return`。

## 2\.将先决条件添加到目录

在命令行中，将目录切换到根文件夹（如果尚未这样做），然后运行以下命令：

- `npm install express`
- `npm install ejs`
- `npm install ejs-locals`
- `npm install restify`
- `npm install mongoose`
- `npm install bunyan`
- `npm install assert-plus`
- `npm install passport`
- `npm install webfinger`
- `npm install body-parser`
- `npm install express-session`
- `npm install cookie-parser`

- 此外，对于快速入门中的骨架项目，我们将使用 `passport-azure-ad`。

- `npm install passport-azure-ad`


这将会安装 passport-azure-ad 所依赖的库。

## 3\.将应用设置为使用 passport-node-js 策略
在这里，我们要将 Express 中间件配置为使用 OpenID Connect 身份验证协议。Passport 将用于发出登录和注销请求、管理用户的会话、获取有关用户的信息，等等。

-	首先，打开位于项目根目录中的 `config.js` 文件，并在 `exports.creds` 节中输入应用程序的配置值。
    -	`clientID:` 是在注册门户中为应用分配的“应用程序 ID”。
    -	`returnURL` 是在门户中输入的“重定向 URI”。
    - `clientSecret` 是在门户中生成的机密。

- 接下来，打开项目根目录中的 `app.js` 文件，并添加以下调用以调用 `passport-azure-ad` 随附的 `OIDCStrategy` 策略

		
		JavaScript
		var OIDCStrategy = require('passport-azure-ad').OIDCStrategy;
		
		// Add some logging
		var log = bunyan.createLogger({
		    name: 'Microsoft OIDC Example Web Application'
		});


- 然后，使用我们刚刚提到的策略来处理登录请求

		JavaScript
		// Use the OIDCStrategy within Passport. (Section 2)
		//
		//   Strategies in passport require a `validate` function, which accept
		//   credentials (in this case, an OpenID identifier), and invoke a callback
		//   with a user object.
		passport.use(new OIDCStrategy({
		    callbackURL: config.creds.returnURL,
		    realm: config.creds.realm,
		    clientID: config.creds.clientID,
		    clientSecret: config.creds.clientSecret,
		    oidcIssuer: config.creds.issuer,
		    identityMetadata: config.creds.identityMetadata,
		    responseType: config.creds.responseType,
		    responseMode: config.creds.responseMode,
		    skipUserProfile: config.creds.skipUserProfile
		    scope: config.creds.scope
		  },
		  function(iss, sub, profile, accessToken, refreshToken, done) {
		    log.info('Example: Email address we received was: ', profile.email);
		    // asynchronous verification, for effect...
		    process.nextTick(function () {
		      findByEmail(profile.email, function(err, user) {
		        if (err) {
		          return done(err);
		        }
		        if (!user) {
		          // "Auto-registration"
		          users.push(profile);
		          return done(null, profile);
		        }
		        return done(null, user);
		      });
		    });
		  }
		));

Passport 使用适用于它的所有策略（Twitter、Facebook 等），所有策略写入器都依循类似的模式。查看该策略，你会发现，我们已将它作为 function() 来传递，其中包含一个令牌和一个用作参数的 done。策略完成所有工作之后，便尽责地返回。完成后，我们需要存储用户并隐藏令牌，因此不需要再次请求它。

> [AZURE.IMPORTANT]
上述代码使用了正好地服务器上进行身份验证的任何用户。这就是所谓的自动注册。在生产服务器中，你希望所有人都必须先经历你确定的注册过程。这通常是在使用者应用中看到的模式，可让向 Facebook 注册，但接着请求填写其他信息。如果这不是示例应用程序，我们就只能从返回的令牌对象中提取电子邮件，然后请求他们填写其他信息。由于这是测试服务器，因此，我们直接将它们加入到内存中的数据库。

- 接下来，让我们添加方法，以便根据 Passport 的要求，持续跟踪已登录的用户。这包括将用户信息序列化和反序列化：

		JavaScript
		
		// Passport session setup. (Section 2)
		
		//   To support persistent login sessions, Passport needs to be able to
		//   serialize users into and deserialize users out of the session.  Typically,
		//   this will be as simple as storing the user ID when serializing, and finding
		//   the user by ID when deserializing.
		passport.serializeUser(function(user, done) {
		  done(null, user.email);
		});
		
		passport.deserializeUser(function(id, done) {
		  findByEmail(id, function (err, user) {
		    done(err, user);
		  });
		});
		
		// array to hold logged in users
		var users = [];
		
		var findByEmail = function(email, fn) {
		  for (var i = 0, len = users.length; i < len; i++) {
		    var user = users[i];
		    log.info('we are using user: ', user);
		    if (user.email === email) {
		      return fn(null, user);
		    }
		  }
		  return fn(null, null);
		};



- 接下来，让我们添加可加载 Express 引擎的代码。在此处，你将看到我们使用了 Express 提供的默认 /views 和 /routes 模式。

		JavaScript
		
		// configure Express (Section 2)
		
		var app = express();
		
		
		app.configure(function() {
		  app.set('views', __dirname + '/views');
		  app.set('view engine', 'ejs');
		  app.use(express.logger());
		  app.use(express.methodOverride());
		  app.use(cookieParser());
		  app.use(expressSession({ secret: 'keyboard cat', resave: true, saveUninitialized: false }));
		  app.use(bodyParser.urlencoded({ extended : true }));
		  // Initialize Passport!  Also use passport.session() middleware, to support
		  // persistent login sessions (recommended).
		  app.use(passport.initialize());
		  app.use(passport.session());
		  app.use(app.router);
		  app.use(express.static(__dirname + '/../../public'));
		});



- 最后，让我们添加 POST 路由，以将实际登录请求递交到 `passport-azure-ad` 引擎：

		JavaScript
		
		// Our Auth routes (Section 3)
		
		// GET /auth/openid
		//   Use passport.authenticate() as route middleware to authenticate the
		//   request.  The first step in OpenID authentication will involve redirecting
		//   the user to their OpenID provider.  After authenticating, the OpenID
		//   provider will redirect the user back to this application at
		//   /auth/openid/return
		
		app.get('/auth/openid',
		  passport.authenticate('azuread-openidconnect', { failureRedirect: '/login' }),
		  function(req, res) {
		    log.info('Authenitcation was called in the Sample');
		    res.redirect('/');
		  });
		
		// GET /auth/openid/return
		//   Use passport.authenticate() as route middleware to authenticate the
		//   request.  If authentication fails, the user will be redirected back to the
		//   login page.  Otherwise, the primary route function function will be called,
		//   which, in this example, will redirect the user to the home page.
		app.get('/auth/openid/return',
		  passport.authenticate('azuread-openidconnect', { failureRedirect: '/login' }),
		  function(req, res) {
		
		    res.redirect('/');
		  });
		
		// POST /auth/openid/return
		//   Use passport.authenticate() as route middleware to authenticate the
		//   request.  If authentication fails, the user will be redirected back to the
		//   login page.  Otherwise, the primary route function function will be called,
		//   which, in this example, will redirect the user to the home page.
		
		app.post('/auth/openid/return',
		  passport.authenticate('azuread-openidconnect', { failureRedirect: '/login' }),
		  function(req, res) {
		
		    res.redirect('/');
		  });


## 4\.使用 Passport 向 Azure AD 发出登录和注销请求

现在，应用程序已正确配置为使用 OpenID Connect 身份验证协议来与 v2.0 终结点通信。`passport-azure-ad` 会代你处理有关创建身份验证消息、验证 Azure AD 提供的令牌以及保留用户会话的繁琐细节。你要做的一切就是提供某种方式让用户登录和注销，以及收集有关已登录用户的其他信息。

- 首先，让我们在 `app.js` 文件中添加 default、login、account 和 logout 方法：

		JavaScript
		
		//Routes (Section 4)
		
		app.get('/', function(req, res){
		  res.render('index', { user: req.user });
		});
		
		app.get('/account', ensureAuthenticated, function(req, res){
		  res.render('account', { user: req.user });
		});
		
		app.get('/login',
		  passport.authenticate('azuread-openidconnect', { failureRedirect: '/login' }),
		  function(req, res) {
		    log.info('Login was called in the Sample');
		    res.redirect('/');
		});
		
		app.get('/logout', function(req, res){
		  req.logout();
		  res.redirect('/');
		});



-	我们详细探讨一下：
    -	`/` 路由将重定向到 index.ejs 视图，并在请求中传递用户（如果存在）
    - `/account` 路由首先“确保我们已经过身份验证”（下面我们将会实现），然后在请求中传递用户，以便我们可以获取有关该用户的其他信息。
    - `/login` 路由将从 `passport-azuread` 调用 azuread-openidconnect 验证器，如果该操作不成功，则将用户重定向回到 /login
    - `/logout` 只是调用 logout.ejs（和路由），以便清除 Cookie 并将用户返回到 index.ejs


- 对于 `app.js` 的最后一个部分，让我们添加上述 `/account` 中使用的 EnsureAuthenticated 方法。

		JavaScript
		
		// Simple route middleware to ensure user is authenticated. (Section 4)
		
		//   Use this route middleware on any resource that needs to be protected.  If
		//   the request is authenticated (typically via a persistent login session),
		//   the request will proceed.  Otherwise, the user will be redirected to the
		//   login page.
		function ensureAuthenticated(req, res, next) {
		  if (req.isAuthenticated()) { return next(); }
		  res.redirect('/login')
		}
		


- 最后，在 `app.js` 中实际创建服务器本身：

JavaScript

app.listen(3000);




## 5\.在 Express 中创建视图与路由，以在网站中显示用户

我们已完成 `app.js`。现在只需添加路由和视图即可，两者将向用户显示我们获取的信息，并处理我们创建的 `/logout` 和 `/login` 路由。

- 在根目录下创建 `/routes/index.js` 路由。

		JavaScript
		
		/*
		 * GET home page.
		 */
		
		exports.index = function(req, res){
		  res.render('index', { title: 'Express' });
		};


- 在根目录下创建 `/routes/user.js` 路由

		JavaScript
		
		/*
		 * GET users listing.
		 */
		
		exports.list = function(req, res){
		  res.send("respond with a resource");
		};


这些简单路由只将请求传递到我们的视图，包括用户（如果存在）。

- 在根目录下创建 `/views/index.ejs` 视图。这是一个简单的页面，将调用我们的登录和注销方法，并允许我们捕获帐户信息。请注意，如果在请求中传递的用户证明我们拥有已登录的用户，就能使用条件性 `if (!user)`。

		JavaScript
		<% if (!user) { %>
			<h2>Welcome! Please log in.</h2>
			<a href="/login">Log In</a>
		<% } else { %>
			<h2>Hello, <%= user.displayName %>.</h2>
			<a href="/account">Account Info</a></br>
			<a href="/logout">Log Out</a>
		<% } %>


- 在根目录下创建 `/views/account.ejs` 视图，以便能够查看 `passport-azuread` 放置在用户请求中的其他信息。

		Javascript
		<% if (!user) { %>
			<h2>Welcome! Please log in.</h2>
			<a href="/login">Log In</a>
		<% } else { %>
		<p>displayName: <%= user.displayName %></p>
		<p>givenName: <%= user.name.givenName %></p>
		<p>familyName: <%= user.name.familyName %></p>
		<p>UPN: <%= user._json.upn %></p>
		<p>Profile ID: <%= user.id %></p>
		<p>Full Claimes</p>
		<%- JSON.stringify(user) %>
		<p></p>
		<a href="/logout">Log Out</a>
		<% } %>


- 最后，可以通过添加布局，使视图变得美观。在根目录下创建 '/views/layout.ejs' 视图

		HTML
		
		<!DOCTYPE html>
		<html>
			<head>
				<title>Passport-OpenID Example</title>
			</head>
			<body>
				<% if (!user) { %>
					<p>
					<a href="/">Home</a> |
					<a href="/login">Log In</a>
					</p>
				<% } else { %>
					<p>
					<a href="/">Home</a> |
					<a href="/account">Account</a> |
					<a href="/logout">Log Out</a>
					</p>
				<% } %>
				<%- body %>
			</body>
		</html>


最后，生成并运行应用程序！

运行 `node app.js` 并导航到 `http://localhost:3000`


使用个人 Microsoft 帐户或者工作或学校帐户登录，随后你会看到该用户的标识已出现在 /account 列表中。Web 应用现在使用行业标准的协议进行保护，你可以使用个人和工作/学校帐户来验证用户。

##后续步骤

[此处以 .zip 格式提供了](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs/archive/complete.zip)完整示例（不包括配置值），你也可以从 GitHub 克隆该示例：

		git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs.git

现在，可以转到更高级的主题。你可能想要尝试：

[使用 v2.0 终结点保护 node.js Web API >>](/documentation/articles/active-directory-v2-devquickstarts-node-api/)

有关更多资源，请查看：

- [v2.0 开发人员指南 >>](/documentation/articles/active-directory-appmodel-v2-overview/)
- [堆栈溢出“azure-active-directory”标记 >>](http://stackoverflow.com/questions/tagged/azure-active-directory)

<!---HONumber=Mooncake_0516_2016-->