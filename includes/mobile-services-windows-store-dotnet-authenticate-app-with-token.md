
先前示例显示标准登录，这要求在此应用每次启动时客户端同时联系标识提供商和移动服务。此方法不仅效率低下，而且如果很多客户尝试同时启动您的应用，您会遇到关于使用率的问题。更好的方法是缓存移动服务返回的授权令牌，然后在使用基于提供商的登录之前首先尝试使用此令牌。 

>[AZURE.NOTE]无论您使用客户端管理的还是服务管理的身份验证，都可以缓存由移动服务颁发的令牌。本教程使用服务管理的身份验证。

1. 在 MainPage.xaml.cs 项目文件中，添加以下 **using** 语句：

		using System.Linq;		
		using Windows.Security.Credentials;		

2. 将 **AuthenticateAsync** 方法替换为以下代码：

        private async System.Threading.Tasks.Task AuthenticateAsync()
        {
            string message;
            // This sample uses the Facebook provider.
            var provider = "Facebook";
              
            // Use the PasswordVault to securely store and access credentials.
            PasswordVault vault = new PasswordVault();
            PasswordCredential credential = null;

            while (credential == null)
            {
                try
                {
                    // Try to get an existing credential from the vault.
                    credential = vault.FindAllByResource(provider).FirstOrDefault();
                }
                catch (Exception)
                {
                    // When there is no matching resource an error occurs, which we ignore.
                }

                if (credential != null)
                {
                    // Create a user from the stored credentials.
                    user = new MobileServiceUser(credential.UserName);
                    credential.RetrievePassword();
                    user.MobileServiceAuthenticationToken = credential.Password;
                    
                    // Set the user from the stored credentials.
                    App.MobileService.CurrentUser = user;

                    try
                    {
                        // Try to return an item now to determine if the cached credential has expired.
                        await App.MobileService.GetTable<TodoItem>().Take(1).ToListAsync();
                    }
                    catch (MobileServiceInvalidOperationException ex)
                    {                        
                        if (ex.Response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
                        {
                            // Remove the credential with the expired token.
                            vault.Remove(credential);
                            credential = null;
                            continue;
                        }
                    }
                }
                else
                {
                    try
                    {
                        // Login with the identity provider.
                        user = await App.MobileService
                            .LoginAsync(provider);                        

                        // Create and store the user credentials.
                        credential = new PasswordCredential(provider,
                            user.UserId, user.MobileServiceAuthenticationToken);
                        vault.Add(credential);
                    }
                    catch (MobileServiceInvalidOperationException ex)
                    {
                        message = "You must log in. Login Required";
                    }
                }
                message = string.Format("You are now logged in - {0}", user.UserId);
                var dialog = new MessageDialog(message);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
            }
        }

	在此版本的 **AuthenticateAsync** 中，应用将尝试使用存储在 **PasswordVault** 中的凭证来访问移动服务。发送一个简单查询以验证存储的令牌是否未到期。返回 401 时，尝试了基于提供商的常规登录。没有存储任何凭证时，也执行常规登录。

	>[AZURE.NOTE]此应用在登录过程中测试令牌是否到期，但正在使用此应用时，可能会在身份验证之后发生令牌到期。有关用于处理到期令牌相关的授权错误的解决方案，请参阅文章[在 Azure 移动服务托管 SDK 中缓存和处理到期令牌](http://blogs.msdn.com/b/carlosfigueira/archive/2014/03/13/caching-and-handling-expired-tokens-in-azure-mobile-services-managed-sdk.aspx)。

3. 两次重新启动此应用。

	请注意，在第一次启动时，再次需要使用此提供商进行登录。但是，在第二次重新启动时，将使用缓存的凭证，而绕过登录。

<!---HONumber=71-->