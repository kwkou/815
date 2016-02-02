1. 在 [Azure 经典门户](https://manage.windowsazure.cn/)中单击目录页上的“应用程序”选项卡。
  
2. 单击集成的应用程序注册。

3. 单击应用程序页上的“配置”，然后向下滚动到页的“密钥”部分。
4. 对于新密钥，请单击“1 年”持续时间。然后单击“保存”，门户将显示新密钥值。
5. 保存后，复制所显示的“客户端 ID”和“密钥”。请注意，在您保存后，密钥值仅向您显示一次。 

    ![](./media/mobile-services-generate-aad-app-registration-access-key-rbac/client-id-and-key.png)

6. 向下滚动到集成应用程序配置页底部并为应用程序启用“读取目录数据”权限，然后单击“保存”。

    ![](./media/mobile-services-generate-aad-app-registration-access-key-rbac/app-perms.png)


7. 在 [Azure 经典门户](https://manage.windowsazure.cn/)中，浏览回到你的移动服务，然后单击“配置”选项卡。向下滚动到“应用设置”部分并添加以下应用设置，然后单击“保存”。

    <table border="1">
    <tr>
    <th>应用设置名称</th><th>说明</th>
    </tr>
    <tr>
    <td>AAD\_CLIENT\_ID</td><td>在以上步骤中从集成应用复制的客户端 ID。</td>
    </tr>
    <tr>
    <td>AAD\_CLIENT\_KEY</td><td>在以上步骤中，在 AAD 集成应用内生成的应用密钥。</td>
    </tr>
    <tr>
    <td>AAD\_TENANT\_DOMAIN</td><td>你的 AAD 域名。应类似于“mydomain.onmicrosoft.com”</td>
    </tr>
    <tr>
    <td>AAD\_GROUP\_ID</td><td>你在上一部分记下的 Sales 组的组 ID</td>
    </tr>
    </table><br/>

 
    ![](./media/mobile-services-generate-aad-app-registration-access-key-rbac/aad-app-settings.png)
  

<!---HONumber=Mooncake_0118_2016-->