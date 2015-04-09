1. 单击 [Azure 管理门户](https://manage.windowsazure.cn/)中您的目录页面上的**应用程序**选项卡。
  
2. 单击集成的应用程序注册。

3. 单击应用程序页面上的**配置**，然后向下滚动到页面的**密钥**部分。 
4. 为新密钥，单击 **1 年**的持续时间。然后，单击**保存**而门户将显示新密钥值。
5. 在您保存后，复制所显示的**客户端 ID** 和**密钥**。请注意，在您保存后，密钥值仅向您显示一次。 

    ![](./media/mobile-services-generate-aad-app-registration-access-key-rbac/client-id-and-key.png)

6. 向下滚动到集成应用程序配置页底部并为应用程序启用**读取目录数据**权限，然后单击**保存**。

    ![](./media/mobile-services-generate-aad-app-registration-access-key-rbac/app-perms.png)


7. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中，浏览回到您的移动服务，然后单击**配置**选项卡。向下滚动到**应用设置**部分并添加以下应用设置，然后单击**保存**。 

    <table border="1">
    <tr>
    <th>App Setting Name</th><th>Description</th>
    </tr>
    <tr>
    <td>AAD_CLIENT_ID</td><td>The client id you copied from your integrated app in the steps above.</td>
    </tr>
    <tr>
    <td>AAD_CLIENT_KEY</td><td>The app key you generated in your AAD integrated app in the steps above.</td>
    </tr>
    <tr>
    <td>AAD_TENANT_DOMAIN</td><td>Your AAD domain name. Should be similar to "mydomain.onmicrosoft.com"</td>
    </tr>
    <tr>
    <td>AAD_GROUP_ID</td><td>The group id you noted for the Sales group in the previous section</td>
    </tr>
    </table><br/>

 
    ![](./media/mobile-services-generate-aad-app-registration-access-key-rbac/aad-app-settings.png)
  
