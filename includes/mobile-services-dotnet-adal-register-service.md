## <a name="register-mobile-service-aad"></a>向 Azure Active Directory 注册你的移动服务


在本节中，您将向 Azure Active Directory 注册您的移动服务，并配置权限以允许单点登录模拟。

1. 遵循[如何向 Azure Active Directory 注册]主题向 Azure Active Directory 注册你的应用程序。

2. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中，返回到 Azure Active Directory 扩展，并单击你的活动目录

3. 单击“应用程序”选项卡，然后单击你的应用程序。

4. 单击“管理清单”。然后单击“下载清单”，并将应用程序清单保存到本地目录中。

       ![](./media/mobile-services-dotnet-adal-register-service/mobile-services-aad-app-manage-manifest.png)

5. 使用 Visual Studio 打开应用程序清单文件。在文件顶部找到应用程序权限行，该行如下所示：

        "oauth2Permissions": [],

    将该行替换为以下应用程序权限，并保存该文件。

        "oauth2Permissions": [
            {
                "adminConsentDescription": "Allow the application access to the mobile service",
                "adminConsentDisplayName": "Have full access to the mobile service",
                "id": "b69ee3c9-c40d-4f2a-ac80-961cd1534e40",
                "isEnabled": true,
                "origin": "Application",
                "type": "User",
                "userConsentDescription": "Allow the application full access to the mobile service on your behalf",
                "userConsentDisplayName": "Have full access to the mobile service",
                "value": "user_impersonation"
            }
        ],

6. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中，再次针对应用程序单击“管理清单”，然后单击“上载清单”。浏览到你刚更新的应用程序清单的位置，然后上载该清单。

<!-- URLs. -->
[如何向 Azure Active Directory 注册]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/

<!---HONumber=Mooncake_0118_2016-->