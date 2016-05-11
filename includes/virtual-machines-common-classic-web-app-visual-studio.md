

创建 Azure 的 Web 应用程序项目时，可以在 Azure 中设置虚拟机。可以然后使用其他软件来配置该虚拟机，或将虚拟机用于诊断或调试。

若要在创建 Web 应用程序时创建虚拟机，请执行以下步骤：

1. 在 Visual Studio 中，单击“文件”>“新建”>“项目”>“Web”，然后选择“ASP.NET Web 应用程序”（在“Visual C#”或“Visual Basic”节点下）。
2. 在“新建 ASP.NET 项目”对话框中，选择所需的 Web 应用程序类型，然后在对话框的“Azure”部分（位于右下角）中，确保已选中“在云中托管”复选框（在某些安装中，此复选框标记为“创建远程资源”）。

	![][0]

3. 对于此示例，在 Azure 下的下拉列表中，选择“虚拟机(v1)”，然后单击“确定”按钮。

4. 单击“视图”，打开“服务器资源管理器”，然后右键单击 **Azure**，选择“管理和筛选订阅...”。单击标记“证书”。现在，你可以为 Azure 订阅添加证书。

	>[AZURE.NOTE] 不要尝试添加帐户，因为默认情况下，所添加的帐户必须是 Azure 全局帐户。我仍在尝试找出更改该项的方法。

5. 单击[此处](https://manage.windowsazure.cn/publishsettings/index?client=vsserverexplorer&schemaversion=2.0)可下载订阅的发布配置文件。单击上一步弹出窗口中的“导入”，并选择刚下载的文件。然后，你将能够管理 Azure.cn 提供的大多数服务。

6. 右键单击“Azure”下“服务器资源管理器”中的“虚拟机”，然后选择“创建虚拟机”。选择刚添加的订阅。选择所需的服务器类型（必须是 windows server）。

7. 按照步骤创建一个 VM。对于“终结点”，必须添加“Web 部署”、“http”。如果需要，可以添加“https”。

8. 创建 VM 后，使用远程桌面连接它。为 Web 部署配置服务器。有关更多详细信息，请查看[此文](http://www.iis.net/learn/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later)。按照步骤操作，你应该能够将服务器配置为启用 Web 部署。

	![][5]

9. 在服务器的远程桌面上，在 IIS 管理器中，右键单击要部署到的站点，正确配置“Web 部署发布”。如果你有 SQL Server，请输入 Web 应用的连接字符串。按 `<your cloud service name>.chinacloudapp.cn` 重放 URL 中的主机名。该 URL 将变为如下所示内容： `https://<your cloud service name>.chinacloudapp.cn:8172/msdeploy.axd` 。单击“安装”，将为你创建发布设置文件。

10. 返回到 Visual Studio，右键单击你的解决方案，然后选择“发布”。

11. 选择“导入”，然后选择从服务器复制的文件。然后输入密码，并选中“保存密码”。现在，你便可以将 Web 应用发布到 VM 了。


[0]: ./media/virtual-machines-common-classic-web-app-visual-studio/CreateVM_NewProject.PNG
[1]: ./media/dotnet-visual-studio-create-virtual-machine/CreateVM_SignIn.PNG
[2]: ./media/virtual-machines-common-classic-web-app-visual-studio/CreateVM_CreateVM.PNG
[3]: ./media/virtual-machines-common-classic-web-app-visual-studio/CreateVM_Provisioning.png
[4]: ./media/virtual-machines-common-classic-web-app-visual-studio/CreateVM_SolutionExplorer.png
[5]: ./media/virtual-machines-common-classic-web-app-visual-studio/VS_Create_VM_Connect.png
