
先前示例显示标准登录，这要求在此应用每次启动时客户端同时联系标识提供商和移动服务。此方法不仅效率低下，而且如果很多客户尝试同时启动您的应用，您会遇到关于使用率的问题。更好的方法是缓存移动服务返回的授权令牌，然后在使用基于提供商的登录之前首先尝试使用此令牌。

>[AZURE.NOTE]无论你使用客户端管理的还是服务管理的身份验证，都可以缓存由移动服务颁发的令牌。本教程使用服务管理的身份验证。

1. 在 default.js 项目文件中，将现有**登录**函数替换为以下代码：

        var credential = null;
        var vault = new Windows.Security.Credentials.PasswordVault();

        // Request authentication from Mobile Services using a Facebook login.
        var login = function () {
            return new WinJS.Promise(function (complete) {
                client.login("facebook").done(function (results) {
                    // Create a credential for the returned user.
                    credential = new Windows.Security.Credentials
                        .PasswordCredential("Facebook", results.userId,
                        results.mobileServiceAuthenticationToken);
                    vault.add(credential);
                    userId = results.userId;

                    refreshTodoItems();
                    var message = "You are now logged in as: " + userId;
                    var dialog = new Windows.UI.Popups.MessageDialog(message);
                    dialog.showAsync().done(complete);
                }, function (error) {
                    var dialog = new Windows.UI.Popups
                        .MessageDialog("An error occurred during login", "Login Required");
                    dialog.showAsync().done(complete);
                });
            });
        }

2. 将现有的**身份验证**函数替换为以下代码：

        var authenticate = function () {
            // Try to get a stored credential from the PasswordVault.                
            try{
                credential = vault.findAllByResource("Facebook").getAt(0);
            }
            catch (error) {
                // This is expected when there's no stored credential.
            }
            
            if (credential) {
                // Set the user from the returned credential.   
                credential.retrievePassword();
                client.currentUser = {
                    "userId": credential.userName,
                    "mobileServiceAuthenticationToken": credential.password
                };

                // Try to return an item now to determine if the cached credential has expired.
                todoTable.take(1).read()
                            .done(function () {
                                refreshTodoItems();
                            }, function (error) {
                                if (error.request.status === 401) {
                                    login(credential, vault).then(function () {
                                        if (!credential) {

                                            // Authentication failed, try again.
                                            authenticate();
                                        }
                                    });
                                }                                   
                            });
            } else {

                login().then(function () {
                    if (!credential) {
                        // Authentication failed, try again.
                        authenticate();
                    }
                });
            }
        }

	在此版本的**身份验证**中，此应用尝试使用存储在 **PasswordVault** 中的凭证来访问移动服务。发送一个简单查询以验证存储的令牌是否未到期。返回 401 时，尝试了基于提供商的常规登录。没有存储任何凭证时，也执行常规登录。

3. 两次重新启动此应用。

	请注意，在第一次启动时，再次需要使用此提供商进行登录。但是，在第二次重新启动时，将使用缓存的凭证，而绕过登录。

<!---HONumber=74-->