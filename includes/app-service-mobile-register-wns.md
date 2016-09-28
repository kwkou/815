
1. 在“Visual Studio 解决方案资源管理器”中，右键单击 Windows 应用商店应用项目，再单击“应用商店”>“将应用与应用商店关联...”。

    ![关联应用与 Windows 应用商店](./media/app-service-mobile-register-wns/notification-hub-associate-win8-app.png)

2. 在向导中，单击“下一步”、使用 Microsoft 帐户登录，再在“保留新应用名称”处输入应用的名称，然后单击“保留”。

3. 成功创建应用注册后，选择新应用名称，再依次单击“下一步”和“关联”。这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。

7. 重复步骤 1 和 3，为使用之前为 Windows 应用商店应用创建的相同注册的 Windows Phone 应用商店应用项目进行设置。

7. 导航到 [Windows 开发人员中心](https://dev.windows.com/overview)、用 Microsoft 帐户登录，再单击“我的应用”中的新应用注册，然后展开“服务”>“推送通知”。

8. 在“推送通知”页上，单击“Windows 推送通知服务(WNS)和Azure 移动应用”下的“Live 服务网站”，并记录“程序包 SID”的值和“应用程序密钥”中的 *当前* 值。

    ![开发人员中心中的应用设置](./media/app-service-mobile-register-wns/mobile-services-win8-app-push-auth.png)

    > [AZURE.IMPORTANT] 应用程序密钥和程序包 SID 是重要的安全凭据。请勿将这些值告知任何人或随你的应用程序分发它们。

<!---HONumber=Mooncake_0919_2016-->