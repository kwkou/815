
   * 单击“登录”，然后输入你的 Azure 帐户的凭据。

     此方法虽然又快又简单，但如果你使用此方法，将无法在“服务器资源管理器”窗口中看到 Azure SQL 数据库或移动服务。

   * 单击“管理订阅”以便安装允许访问你帐户的管理证书。

     在“管理 Azure 订阅”对话框中，单击“证书”选项卡，然后单击“导入”。按照说明为你的 Azure 帐户下载并导入一个订阅文件（也称为 .publishsettings 文件）。

     
     > [AZURE.NOTE] 将此订阅文件下载到源代码目录之外的文件夹中（例如，在 Downloads 文件夹中），然后在导入完成后将其删除。获得了此订阅文件访问权的恶意用户可以编辑、创建和删除你的 Azure 服务。

   有关详细信息，请参阅[如何通过 Visual Studio 连接到 Azure](/documentation/articles/role-based-access-control-configure/)。

<!---HONumber=Mooncake_0815_2016-->