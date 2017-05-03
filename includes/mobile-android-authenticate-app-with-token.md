上一示例介绍了标准登录过程，此过程要求在应用每次启动时客户端同时联系标识提供者和后端 Azure 服务。 此方法效率较低，并且许多用户同时启动应用时可能产生使用率相关方面的问题。 更好的方法是缓存 Azure 服务返回的授权令牌，然后在使用基于提供者的登录之前首先尝试使用此令牌。

> [AZURE.NOTE]
> 无论使用客户端管理的还是服务管理的身份验证，都可以缓存由后端 Azure 服务颁发的令牌。 本教程使用服务管理的身份验证。
>
>

1. 打开 ToDoActivity.java 文件并添加以下 import 语句：

        import android.content.Context;
        import android.content.SharedPreferences;
        import android.content.SharedPreferences.Editor;
2. 将以下成员添加到 `ToDoActivity` 类。

        public static final String SHAREDPREFFILE = "temp";    
        public static final String USERIDPREF = "uid";    
        public static final String TOKENPREF = "tkn";    
3. 在 ToDoActivity.java 文件中，为 `cacheUserToken` 方法添加下面的定义。

        private void cacheUserToken(MobileServiceUser user)
        {
            SharedPreferences prefs = getSharedPreferences(SHAREDPREFFILE, Context.MODE_PRIVATE);
            Editor editor = prefs.edit();
            editor.putString(USERIDPREF, user.getUserId());
            editor.putString(TOKENPREF, user.getAuthenticationToken());
            editor.commit();
        }    

    此方法将用户 ID 和令牌存储在标记为私有的首选项文件中。 这可保护对缓存的访问，这样设备上的其他应用便无权访问此令牌。 因为应用的首选项已经过沙盒处理。 但是，如果有人获取了设备的访问权，则它们可能会通过其他方式获得对令牌缓存的访问权。

   > [AZURE.NOTE]
   > 如果使用令牌访问的数据非常敏感，并且有人可能会获得设备的访问权限，则可以使用加密进一步保护令牌。 但是，完全安全的解决方案超出了本教程的范围并且取决于具体的安全要求。
   >
   >
4. 在 ToDoActivity.java 文件中，为 `loadUserTokenCache` 方法添加下面的定义。

        private boolean loadUserTokenCache(MobileServiceClient client)
        {
            SharedPreferences prefs = getSharedPreferences(SHAREDPREFFILE, Context.MODE_PRIVATE);
            String userId = prefs.getString(USERIDPREF, null);
            if (userId == null)
                return false;
            String token = prefs.getString(TOKENPREF, null);
            if (token == null)
                return false;

            MobileServiceUser user = new MobileServiceUser(userId);
            user.setAuthenticationToken(token);
            client.setCurrentUser(user);

            return true;
        }
5. 在 *ToDoActivity.java* 文件中，将 `authenticate` 方法替换为下面这种使用令牌缓存的方法。 如果要使用的帐户不是 MicrosoftAccount 帐户，请更改登录提供者。

        private void authenticate() {
            // We first try to load a token cache if one exists.
            if (loadUserTokenCache(mClient))
            {
                createTable();
            }
            // If we failed to load a token cache, login and create a token cache
            else
            {
                // Login using the MicrosoftAccount.    
                ListenableFuture<MobileServiceUser> mLogin = mClient.login(MobileServiceAuthenticationProvider.MicrosoftAccount);

                Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
                    @Override
                    public void onFailure(Throwable exc) {
                        createAndShowDialog("You must log in. Login Required", "Error");
                    }           
                    @Override
                    public void onSuccess(MobileServiceUser user) {
                        createAndShowDialog(String.format(
                                "You are now logged in - %1$2s",
                                user.getUserId()), "Success");
                        cacheUserToken(mClient.getCurrentUser());
                        createTable();    
                    }
                });
            }
        }
6. 构建此应用并使用有效帐户来测试验证。 至少两次运行它。 在第一次运行期间，你应会收到要求登录并创建令牌缓存的提示。 之后，每次运行时，系统都会尝试加载用于身份验证的令牌缓存。 而你无需再次登录。