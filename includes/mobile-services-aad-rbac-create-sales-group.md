在本节中，将两个新用户添加到您的目录以及新销售组。其中一个用户将被授予销售组成员身份。另一个用户将不会被授予该组成员身份。 

### 创建用户


1. 在 [Azure 管理门户](https://manage.windowsazure.cn)中，导航到你前面在完成有关向应用添加身份验证的教程时为身份验证配置的目录。
2. 单击页面顶部的“用户”。然后单击底部的“添加用户”按钮。 
3. 完成“新建用户”对话框，该对话框创建来创建名为 **Bob** 的用户。请注意用户的临时密码。 
4. 创建名为 **Dave** 的另一个用户。请注意用户的临时密码。
5. 新用户应类似于以下显示的内容。

    ![](./media/mobile-services-aad-rbac-create-sales-group/users.png)    


### 创建销售组


1. 在目录页上，单击页面顶部的“组”。然后单击底部的“添加组”按钮。 
2. 输入**销售**作为组名称，然后按对话框上的“完成”按钮以创建组。 

    ![](./media/mobile-services-aad-rbac-create-sales-group/sales-group.png)

### 将用户成员身份添加到销售组。


1. 单击目录页面顶部的“组”。然后单击“销售”组以转到销售组页面。 
2. 在“销售”组页上，单击“添加成员”。将名为 **Bob** 的用户添加到销售组。名为 **Dave** 的用户不应是该组的成员。

    ![](./media/mobile-services-aad-rbac-create-sales-group/group-membership.png)

2. 在“销售”组页上单击“属性”，然后复制页面底部的销售组“对象 ID”。

   
    ![](./media/mobile-services-aad-rbac-create-sales-group/sales-group-id.png)

3. 导航回到移动服务配置页，并添加该对象 ID 作为名为 **AAD\_SALES\_GROUP\_ID** 的应用设置。本教程使用组的对象 ID 作为应用设置，而不是基于组名称查找 ID。这是因为组名称可能会更改，而 ID 会保持相同。

    ![](./media/mobile-services-aad-rbac-create-sales-group/sales-group-id-app-setting.png)

<!---HONumber=Mooncake_0118_2016-->