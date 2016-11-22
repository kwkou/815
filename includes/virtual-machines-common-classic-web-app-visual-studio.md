
>[AZURE.NOTE] 若要在 Visual Studio 2015 中使用 Azure 中国区，需要配置 Visual Studio 环境。有关详细信息，请参阅[开发人员差异](/documentation/articles/developerdifferences/)。

创建 Azure 的 Web 应用程序项目时，可以在 Azure 中设置虚拟机。可以然后使用其他软件来配置该虚拟机，或将虚拟机用于诊断或调试。

若要在创建 Web 应用程序时创建虚拟机，请执行以下步骤：

1. 在 Visual Studio 中，单击“文件”>“新建”>“项目”>“Web”，然后选择“ASP.NET Web 应用程序”（在“Visual C#”或“Visual Basic”节点下）。
2. 在“新建 ASP.NET 项目”对话框中，选择所需的 Web 应用程序类型，然后在对话框的“Azure”部分（位于右下角）中，确保已选中“在云中托管”复选框（在某些安装中，此复选框标记为“创建远程资源”）。

	![][0]  

3. 对于此示例，在 Azure 下的下拉列表中，选择“虚拟机(v1)”，然后单击“确定”按钮。
4. 根据系统提示登录到 Azure。将显示“创建虚拟机”对话框。

	![][2]  

5. 在“DNS 名称”框中，输入虚拟机的名称。DNS 名称在 Azure 中必须唯一。如果输入的名称不可用，则会出现红色感叹号。
6. 在“映像”列表中，选择要用作虚拟机基础的映像。你可以选择任何标准 Azure 虚拟机映像，或者已上载到 Azure 的自有映像。
7. 除非你打算安装其他 Web 服务器，否则请保留选中“启用 IIS 和 Web 部署”复选框。如果禁用“Web 部署”，将无法从 Visual Studio 发布。可以将 IIS 和 Web 部署添加到任何打包的 Windows Server 映像，包括你自己的自定义映像。
8. 在“大小”列表中，选择虚拟机的大小。
9. 指定此虚拟机的登录凭据。请记下这些信息，因为在通过远程桌面访问计算机时需要这些信息。
10. 在“位置”列表中，选择虚拟机的托管区域。
11. 单击“确定”按钮开始创建虚拟机。可以在“输出”窗口中查看操作的进度。

	![][3]  

12. 预配虚拟机时，系统会在解决方案的 **PublishScripts** 节点中创建发布脚本。发布脚本在 Azure 中运行并设置虚拟机。“输出”窗口将显示状态。脚本将执行以下操作来设置虚拟机：

	* 如果虚拟机尚不存在，则创建虚拟机。
	* 仅当指定的区域中没有名称以 `devtest` 开头的存储帐户时，才创建具有此名称的存储帐户。
	* 为虚拟机创建用作容器的云服务，并为 Web 应用程序创建 Web 角色。
	* 在虚拟机上配置 Web 部署。
	* 在虚拟机上配置 IIS 和 ASP.NET。

	![][4]  

13. （可选）你可以连接到新的虚拟机。在“服务器资源管理器”中，展开“虚拟机”节点，选择你创建的虚拟机节点，然后在其快捷方式菜单中，选择“使用远程桌面连接”。或者，在“云资源管理器”中，你可以在快捷菜单中选择“在门户中打开”并连接到那里的虚拟机。

 ![][5]  


[0]: ./media/virtual-machines-common-classic-web-app-visual-studio/CreateVM_NewProject.PNG
[1]: ./media/dotnet-visual-studio-create-virtual-machine/CreateVM_SignIn.PNG
[2]: ./media/virtual-machines-common-classic-web-app-visual-studio/CreateVM_CreateVM.PNG
[3]: ./media/virtual-machines-common-classic-web-app-visual-studio/CreateVM_Provisioning.png
[4]: ./media/virtual-machines-common-classic-web-app-visual-studio/CreateVM_SolutionExplorer.png
[5]: ./media/virtual-machines-common-classic-web-app-visual-studio/VS_Create_VM_Connect.png

<!---HONumber=Mooncake_1114_2016-->