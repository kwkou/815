<properties
    pageTitle="Azure AD Android 入门 | Azure"
    description="如何生成一个与 Azure AD 集成以方便登录，并使用 OAuth 调用 Azure AD 保护 API 的 Android 应用程序。"
    services="active-directory"
    documentationcenter="android"
    author="xerners"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="da1ee39f-89d3-4d36-96f1-4eabbc662343"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="mobile-android"
    ms.devlang="java"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/07/2017"
    ms.author="brandwe" />

# 将 Azure AD 集成到 Android 应用中
[AZURE.INCLUDE [active-directory-devquickstarts-switcher](../../includes/active-directory-devquickstarts-switcher.md)]

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

如果你要开发桌面应用程序，Azure AD 可让你简单直接地使用用户的 Active Directory 帐户对其进行身份验证。它还可以让应用程序安全地使用 Azure AD 保护的任何 Web API，例如 Office 365 API 或 Azure API。

对于需要访问受保护资源的 Android 客户端，Azure AD 提供 Active Directory 身份验证库 (ADAL)。在本质上，ADAL 的唯一用途就是方便应用获取访问令牌。为了演示操作的简单性，下面我们要生成一个 Android 待办事项列表应用程序，该应用程序可以：

- 使用 [OAuth 2.0 身份验证协议](/documentation/articles/active-directory-protocols-oauth-code/)获取调用待办事项列表 API 的访问令牌。
- 获取用户的待办事项列表
- 将用户注销。

若要开始，你需要一个可在其中创建用户和注册应用程序的 Azure AD 租户。如果你还没有租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

> [AZURE.TIP]
欢迎试用新的[开发人员门户](https://identity.microsoft.com/Docs/Android)预览版，只需花费几分钟时间，它就能帮助你开始使用 Azure Active Directory！ 在开发人员门户中，可以逐步完成注册应用并将 Azure AD 集成到代码的整个过程。完成上述过程后，将会创建一个可对租户中的用户进行身份验证的简单应用程序，以及一个可以接受令牌和执行验证的后端。
> 
> 

## 步骤 1：下载并运行 Node.js REST API TODO 示例服务器
专门编写这个示例是为了与用于生成 Azure Active Directory 的单租户待办事项 REST API 的现有示例配合使用。这是本快速入门教程的先决条件。

有关如何设置的信息，请访问我们的现有示例：

- [适用于 Node.js 的 Azure Active Directory 示例 REST API 服务](/documentation/articles/active-directory-devquickstarts-webapi-nodejs/)

## 步骤 2：向 Azure AD 租户注册 Web API
**我正在执行什么操作？**

*Microsoft Active Directory 支持添加两种类型的应用程序。Web API，用于向访问这些 Web API 的用户和应用程序（在 Web 上或者设备中运行的应用程序上）提供服务。在此步骤中，你将注册你在本地运行的用于测试此示例的 Web API。通常，此 Web API 是一种 REST 服务，提供应用需要访问的功能。Azure Active Directory 可以保护任何终结点！*

*此处我们假设你要注册上面引用的 TODO REST API，但这也适用于你希望 Azure Active Directory 保护的任何 Web API。*

向 Azure AD 注册 Web API 的步骤

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 在左侧的导航栏中单击“Active Directory”。
3. 单击你要在其中注册示例应用程序的目录租户。
4. 单击“应用程序”选项卡。
5. 在抽屉中，单击“添加”。
6. 单击“添加我的组织正在开发的应用程序”。
7. 为应用程序输入一个友好的名称，例如“TodoListService”，选择“Web 应用程序和/或 Web API”，然后单击“下一步”。
8. 对于登录 URL，请输入示例的基 URL，默认情况下为 `https://localhost:8080`。
9. 对于应用程序 ID URI，请输入 `https://<your_tenant_name>/TodoListService`，并将 `<your_tenant_name>` 替换为你的 Azure AD 租户的名称。单击“确定”完成注册。
10. 仍然在 Azure 门户预览中，单击你的应用程序的“配置”选项卡。
11. **查找“客户端 ID”值并将它复制到某个位置**，稍后配置应用程序时需要用到它。

## 步骤 3：注册示例 Android 本机客户端应用程序
注册 Web 应用程序是第一步。接下来，你还需要告诉 Azure Active Directory 有关应用程序的情况。这样，应用程序便能够与刚刚注册的 Web API 通信

**我正在执行什么操作？**

*如前所述，Azure Active Directory 支持添加两种类型的应用程序。Web API，用于向访问这些 Web API 的用户和应用程序（在 Web 上或者设备中运行的应用程序上）提供服务。在此步骤中，你要将应用程序注册到此示例。只有执行了此操作，此应用程序才能请求访问你刚刚注册的 Web API。除非注册了应用程序，否则 Azure Active Directory 甚至可能会拒绝应用程序的登录请求！ 这是模型安全功能的一部分。*

*此处我们假设你要注册上面引用的这个示例应用程序，但这也适用于你正在开发的任何应用程序。*

**为什么要将应用程序和 Web API 放在一个租户中？**

*如你可能猜到的那样，你可以生成一个从其他租户访问 Azure Active Directory 中注册的外部 API 的应用程序。如果你这样做，系统将提示你的客户许可使用该应用程序中的 API。令人欣慰的是，适用于 iOS 的 Active Directory 身份验证库将负责为你处理此许可！ 在了解更高级的功能后，你将发现，这是从 Azure 和 Office 以及任何其他服务提供程序访问 Microsoft API 套件所需工作的重要部分。现在，由于你已将 Web API 和应用程序注册到同一个租户下，因此，你不会看到任何许可提示。如果你只是在为自己的公司开发要使用的应用程序，则通常会出现这种情况。*

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 在左侧的导航栏中单击“Active Directory”。
3. 单击你要在其中注册示例应用程序的目录租户。
4. 单击“应用程序”选项卡。
5. 在抽屉中，单击“添加”。
6. 单击“添加我的组织正在开发的应用程序”。
7. 为应用程序输入一个友好的名称，例如“TodoListClient-Android”，选择“本机客户端应用程序”，然后单击“下一步”。
8. 对于“重定向 URI”，请输入 `http://TodoListClient`。单击“完成”。
9. 单击应用程序的“配置”选项卡。
10. 查找“客户端 ID”值并将它复制到某个位置，稍后配置应用程序时需要用到它。
11. 在“针对其他应用程序的权限”中，单击“添加应用程序”。 在“显示”下拉列表中选择“其他”，然后单击上方的复选标记。找到并单击“TodoListService”，然后单击底部的复选标记以添加该应用程序。从“委派的权限”下拉列表中选择“访问 TodoListService”，然后保存配置。

若要使用 Maven 生成，你可以在顶层使用 pom.xml

- 将此存储库克隆到所选的目录：
  
		$ git clone git@github.com:AzureADSamples/NativeClient-Android.git 

- 根据[“先决条件”部分中的步骤针对 Android 设置 Maven](https://github.com/MSOpenTech/azure-activedirectory-library-for-android/wiki/Setting-up-maven-environment-for-Android)
- 使用 SDK 19 设置模拟器
- 转到存储库克隆到的根文件夹
- 运行命令：mvn clean install
- 将目录切换到快速入门项目示例：cd samples\hello
- 运行命令：mvn android:deploy android:run
- 你应会看到应用正在启动
- 输入测试用户凭据以尝试启动！

除了 aar 包以外，还将提交 Jar 包。

### 步骤 4：下载 Android ADAL 并将其添加到 Eclipse 工作区
我们提供了多个选项以方便你在 Android 项目中使用此库：

- 你可以使用源代码将此库导入到 Eclipse 并链接到应用程序。
- 如果使用 Android Studio，则可以使用 *aar* 包格式并引用二进制文件。

#### 选项 1：源 Zip
若要下载源代码副本，请在页面右侧单击"下载 ZIP"，或单击[此处](https://github.com/AzureAD/azure-activedirectory-library-for-android/archive/v1.0.9.tar.gz)。

#### 选项 2：通过 Git 获取源代码
若要通过 git 获取 SDK 的源代码，只需键入：

    git clone git@github.com:AzureAD/azure-activedirectory-library-for-android.git
    cd ./azure-activedirectory-library-for-android/src

#### 选项 3：通过 Gradle 获取二进制文件
你可以从 Maven 中心存储库获取二进制文件。AndroidStudio 的项目中可能包含了如下所示的 AAR 包：

gradle

		repositories {
		    mavenCentral()
		    flatDir {
		        dirs 'libs'
		    }
		    maven {
		        url "YourLocalMavenRepoPath\\.m2\\repository"
		    }
		}
		dependencies {
		    compile fileTree(dir: 'libs', include: ['*.jar'])
		    compile('com.microsoft.aad:adal:1.1.1') {
		        exclude group: 'com.android.support'
		    } // Recent version is 1.1.1
		}


#### 选项 4：通过 Maven 获取 aar
如果使用 Eclipse 中的 m2e 插件，可以在 pom.xml 文件中指定依赖关系：

xml

		<dependency>
		    <groupId>com.microsoft.aad</groupId>
		    <artifactId>adal</artifactId>
		    <version>1.1.1</version>
		    <type>aar</type>
		</dependency>
		


#### 选项 5：libs 文件夹中的 jar 包
可以从 maven 存储库获取 jar 文件并将其放入项目中的 *libs* 文件夹。你还需要将所需的资源复制到项目，因为 jar 包不包括这些项目。

### 步骤 5：在项目中添加对 Android ADAL 的引用
1. 添加对项目的引用，并将其指定为 Android 库。如果你不确定如何执行此操作，请[单击此处了解详细信息](http://developer.android.com/tools/projects/projects-eclipse.html)
2. 在项目设置中添加用于调试的项目依赖关系
3. 更新项目的 AndroidManifest.xml 文件以包括：
   
		<uses-permission android:name="android.permission.INTERNET" />
		<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
		<application
            android:allowBackup="true"
            android:debuggable="true"
            android:icon="@drawable/ic_launcher"
            android:label="@string/app_name"
            android:theme="@style/AppTheme" >
   
            <activity
                android:name="com.microsoft.aad.adal.AuthenticationActivity"
                android:label="@string/title_login_hello_app" >
            </activity>
		....
		<application/>

4. 在主要活动中创建 AuthenticationContext 的实例。有关此调用的详细信息超出了本自述文件的范畴，但你可以通过查看 [Android 本机客户端示例](https://github.com/AzureADSamples/NativeClient-Android)来获得一个良好的起点。下面是一个示例，它使用 SharedPreferences 作为默认缓存，其中颁发机构采用 `https://login.chinacloudapi.cn/yourtenant.partner.onmschina.cn` 的形式：

	Java

		mContext = new AuthenticationContext(MainActivity.this, authority, true);// mContext is a field in your activity

5. 复制此代码块，以便在用户输入凭据并收到授权代码后处理 AuthenticationActivity 的结束工作：
   
	Java

		@Override
         protected void onActivityResult(int requestCode, int resultCode, Intent data) {
             super.onActivityResult(requestCode, resultCode, data);
             if (mContext != null) {
                mContext.onActivityResult(requestCode, resultCode, data);
             }
         }

6. 若要请求令牌，你可以定义一个回调
   
	Java

	    private AuthenticationCallback<AuthenticationResult> callback = new AuthenticationCallback<AuthenticationResult>() {
	   
	            @Override
	            public void onError(Exception exc) {
	                if (exc instanceof AuthenticationException) {
	                    textViewStatus.setText("Cancelled");
	                    Log.d(TAG, "Cancelled");
	                } else {
	                    textViewStatus.setText("Authentication error:" + exc.getMessage());
	                    Log.d(TAG, "Authentication error:" + exc.getMessage());
	                }
	            }
	   
	            @Override
	            public void onSuccess(AuthenticationResult result) {
	                mResult = result;
	   
	                if (result == null || result.getAccessToken() == null
	                        || result.getAccessToken().isEmpty()) {
	                    textViewStatus.setText("Token is empty");
	                    Log.d(TAG, "Token is empty");
	                } else {
	                    // request is successful
	                    Log.d(TAG, "Status:" + result.getStatus() + " Expired:"
	                            + result.getExpiresOn().toString());
	                    textViewStatus.setText(PASSED);
	                }
	            }
	        };
 
7. 最后，使用该回调请求令牌：

	Java

	     mContext.acquireToken(MainActivity.this, resource, clientId, redirect, user_loginhint, PromptBehavior.Auto, "",
	                    callback);
	    

	参数说明：
	
	- Resource 是必需的，它是你尝试访问的资源。
	- Clientid 是必需的，它来自 AzureAD 门户。
	- 你可以将 redirectUri 设置为包名称。对于 acquireToken 调用，不需要提供此参数。
	- PromptBehavior 可帮助请求凭据以跳过缓存和 Cookie。
	- 在交换令牌的授权代码后，将调用 Callback。
	  
	  Callback 具有一个包含 accesstoken、过期日期和 idtoken 信息的 AuthenticationResult 对象。
	
	可选：**acquireTokenSilent**
	
	可以调用 **acquireTokenSilent** 来处理缓存和令牌刷新。它也提供了同步版本。它接受使用 userid 作为参数。
	
	Java
	
	     mContext.acquireTokenSilent(resource, clientid, userId, callback );
	    

8. **Broker**：
   Microsoft Intune 的公司门户应用程序将提供代理组件。如果在验证器中创建了一个用户帐户并且开发人员选择不跳过中转站帐户，ADAL 将使用中转站帐户。开发人员可以使用以下操作跳过中转站用户：
   
    Java

		AuthenticationSettings.Instance.setSkipBroker(true);
   
	   开发人员需要注册特殊的 redirectUri 供中转站使用。RedirectUri 的格式为 `msauth://packagename/Base64UrlencodedSignature`。你可以使用脚本“brokerRedirectPrint.ps1”获取应用的 redirecturi，也可以使用 API 调用 mContext.getBrokerRedirectUri。签名与签名证书相关。
	   
	   当前中转站模型针对一个用户。AuthenticationContext 提供用于获取中转站用户的 API 方法。
	   
	   Java
	
		String brokerAccount =  mContext.getBrokerUser(); //Broker user will be returned if account is valid.`
	   
	   应用程序清单应有权使用 AccountManager 帐户：http://developer.android.com/reference/android/accounts/AccountManager.html
	   
	   - GET\_ACCOUNTS
	   - USE\_CREDENTIALS
	   - MANAGE\_ACCOUNTS

使用本演练时，你应会获得与 Azure Active Directory 成功集成所需的项目。有关此工作的更多示例，请访问 GitHub 上的 AzureADSamples/ 存储库。

## 重要信息
### 自定义
应用程序资源可能会覆盖库项目资源。在应用生成时将发生这种情况。为此，你可以使用所需的方式自定义身份验证活动布局。需要确保保留 ADAL 使用的控件 ID (Webview)。

### 中转站
中转站组件将随 Microsoft Intune 的公司门户应用一起提供。帐户将在帐户管理器中创建。帐户类型为“com.microsoft.workaccount”。它只允许单个 SSO 帐户。在完成一个应用的设备质询后，它将为此用户创建 SSO Cookie。

### 机构 URL 和 ADFS
ADFS 不会识别为生产 STS，因此，你需要关闭实例发现，并在 AuthenticationContext 构造函数中传递 false。

机构 URL 需要 STS 实例和租户名称：https://login.chinacloudapi.cn/yourtenant.partner.onmschina.cn

### 查询缓存项
ADAL 在 SharedPreferences 中提供默认缓存，以及一些简单的缓存查询函数。你可以使用以下命令从 AuthenticationContext 中获取当前缓存：

Java

	ITokenCacheStore cache = mContext.getCache();

你还可以提供缓存实现（如果你想要对其进行自定义）。
Java

	mContext = new AuthenticationContext(MainActivity.this, authority, true, yourCache);


### PromptBehavior

ADAL 提供用于指定提示行为的选项。如果刷新令牌无效并且需要用户凭据，PromptBehavior.Auto 会弹出 UI。PromptBehavior.Always 会跳过缓存使用并始终显示 UI。

### 来自缓存和刷新的无提示令牌请求

此方法不使用 UI 弹出，并且不需要活动。它将从缓存返回令牌（如果可用）。如果令牌已过期，它将尝试刷新令牌。如果刷新令牌已过期或失败，它将返回 AuthenticationException。

Java

	Future<AuthenticationResult> result = mContext.acquireTokenSilent(resource, clientid, userId, callback );

你也可以使用此方法执行同步调用。可以将回调设置为 null，或使用 acquireTokenSilentSync。

### 诊断
下面是有关诊断问题的主要信息来源：

- 异常
- 日志
- 网络跟踪

另请注意，相关性 ID 是在库中进行诊断的关键所在。如果你想要将某个 ADAL 请求关联到代码中的其他操作，可以根据请求设置相关性 ID。如果未设置相关性 ID，ADAL 将生成一个随机 ID，并且所有日志消息和网络调用将戳记该相关性 ID。每发出一个请求，自我生成的 ID 都会更改。

#### 异常
显然这是第一项诊断。我们将尝试提供有用的错误消息。如果你发现某个错误消息没有作用，请记录相应的问题并告诉我们。此外，请提供设备信息，例如模型和 SDK 编号。

#### 日志
你可以将库配置为生成有助于诊断问题的日志消息。若要配置日志记录，你可以执行以下调用以配置一个回调，ADAL 将使用该回调来移交它所生成的每条日志消息。

Java

	Logger.getInstance().setExternalLogger(new ILogger() {
		 @Override
		public void Log(String tag, String message, String additionalMessage, LogLevel level, ADALError errorCode) {
		 ...
		// You can write this to logfile depending on level or errorcode.
		 writeToLogFile(getApplicationContext(), tag +":" + message + "-" + additionalMessage);
		}
	}
		 
消息可写入自定义日志文件，如下所示。遗憾的是，没有标准的方法可从设备中获取日志。有些服务可帮助你实现此目的。你也可以还创造自己的方法，例如，将文件发送到服务器。

Java

	private syncronized void writeToLogFile(Context ctx, String msg) {
		File directory = ctx.getDir(ctx.getPackageName(), Context.MODE_PRIVATE);
		File logFile = new File(directory, "logfile");
		FileOutputStream outputStream = new FileOutputStream(logFile, true);
		OutputStreamWriter osw = new OutputStreamWriter(outputStream);
		osw.write(msg);
		osw.flush();
		osw.close();
	}

##### 日志记录级别
- 错误（异常）
- 警告（警告）
- 信息（供参考）
- 详细（更多详细信息）

可按如下所述设置日志级别：

Java

	Logger.getInstance().setLogLevel(Logger.LogLevel.Verbose);

 除了将所有日志消息发送到任何自定义日志回调以外，还将其发送到 logcat。
 可以将日志从 logcat 提取到文件，如下所示：

	adb logcat > "C:\logmsg\logfile.txt"
 有关 adb 命令的更多示例：https://developer.android.com/tools/debugging/debugging-log.html#startingLogcat

#### 网络跟踪
可以使用各种工具来捕获 ADAL 生成的 HTTP 流量。如果你熟悉 OAuth 协议或者需要向 Microsoft 或其他支持渠道提供诊断信息，这将十分有用。

Fiddler 是最方便的 HTTP 跟踪工具。使用以下链接设置该工具以正确记录 ADAL 网络流量。若要发挥作用，你必须配置 fiddler 或任何其他工具（如 Charles）以记录未加密的 SSL 流量。注意：以这种方式生成的跟踪可能包含高特权信息，例如访问令牌、用户名和密码。如果你使用的是生产帐户，请不要与第三方共享这些跟踪。如果你需要向某人提供跟踪以便获得支持，请使用一个临时帐户再现问题，该帐户包含你不介意共享的用户名和密码。

- [针对 Android 设置 Fiddler](http://docs.telerik.com/fiddler/configure-fiddler/tasks/ConfigureForAndroid)
- [为 ADAL 配置 Fiddler 规则](https://github.com/AzureAD/azure-activedirectory-library-for-android/wiki/How-to-listen-to-httpUrlConnection-in-Android-app-from-Fiddler)

### 对话模式
没有活动的 acquireToken 方法支持对话提示。

### 加密
默认情况下，ADAL 会加密令牌并将其存储在 SharedPreferences 中。你可以查看 StorageHelper 类以了解详细信息。Android 引入了 AndroidKeyStore 4.3(API18) 来安全存储私钥。ADAL 将针对 API18 及更高版本使用该组件。如果你想要对较低的 SDK 版本使用 ADAL，需要在 AuthenticationSettings.INSTANCE.setSecretKey 中提供密钥

### Oauth2 持有者质询
AuthenticationParameters 类提供通过 Oauth2 持有者质询获取 authorization\_uri 的功能。

### Webview 中的会话 Cookie
在关闭应用程序后，Android Webview 不会清除会话 Cookie。你可以使用以下示例代码来处理此问题：

Java
		
	CookieSyncManager.createInstance(getApplicationContext());
	CookieManager cookieManager = CookieManager.getInstance();
	cookieManager.removeSessionCookie();
	CookieSyncManager.getInstance().sync();

有关 Cookie 的详细信息：http://developer.android.com/reference/android/webkit/CookieSyncManager.html

### 资源重写
ADAL 库包含以下两条 ProgressDialog 消息的英文字符串。

如果需要本地化的字符串，你的应用程序应覆盖这些英文字符串。

     <string name="app_loading">Loading...</string>
     <string name="broker_processing">Broker is processing</string>
     <string name="http_auth_dialog_username">Username</string>
     <string name="http_auth_dialog_password">Password</string>
     <string name="http_auth_dialog_title">Sign In</string>
     <string name="http_auth_dialog_login">Login</string>
     <string name="http_auth_dialog_cancel">Cancel</string>

### NTLM 对话
ADAL 版本 1.1.0 支持通过 WebViewClient 中的 onReceivedHttpAuthRequest 事件处理的 NTLM 对话。你可以自定义对话布局和字符串。

### 跨应用 SSO
了解[如何使用 ADAL 在 Android 上启用跨应用 SSO](/documentation/articles/active-directory-sso-android/)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_0120_2017-->
<!---Update_Description: wording update -->