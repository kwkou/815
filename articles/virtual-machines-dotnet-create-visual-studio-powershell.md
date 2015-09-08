<properties urlDisplayName="Create a virtual machine for a website" pageTitle="使用 Visual Studio 创建网站的虚拟机" metaKeywords="Visual Studio, ASP.NET, web project, virtual machine" description="为网站创建虚拟机" metaCanonical="" services="" documentationCenter="" title="Creating a virtual machine for a website with Visual Studio" authors="ghogen" solutions="" manager="douge" editor="" />
<tags
	ms.service="virtual-machines"
	ms.date="06/10/2015"
    wacn.date="08/29/2015"/>

# 使用 Visual Studio 创建用于网站的虚拟机

创建 Azure 网站的 Web 项目时，可以在 Azure 中设置虚拟机。可以然后使用其他软件来配置该虚拟机，或将虚拟机用于诊断或调试。

若要在创建网站时创建虚拟机，请执行以下步骤：

1. 在 Visual Studio 中，选择“文件”、“新建项目”，选择“Web”，然后选择“ASP.NET Web 应用程序”（在“Visual C#”或“Visual Basic”节点下）。
2. 在“新建 ASP.NET 项目”对话框中，选择所需的 Web 应用程序类型，然后在对话框的“Azure”部分（位于右下角）中，确保已选中“在云中托管”复选框（在某些安装中，此复选框标记为“创建远程资源”）。

	![][0]

3. 在 Windows Azure 下的下拉列表框中，选择“虚拟机”，然后选择“确定”按钮。
4. 根据系统提示登录到 Azure。将显示“创建虚拟机”对话框。

	![][2]

5. 在“DNS 名称”框中，输入虚拟机的名称。DNS 名称在 Azure 中必须唯一。如果输入的名称不可用，则会出现红色感叹号。
6. 在“映像”列表中，选择要用作虚拟机基础的 VM 映像。你可以选择任何标准 Azure VM 映像，或者已上载到 Azure 的自有映像。
7. 除非你打算安装其他 Web 服务器，否则请保留选中“启用 IIS 和 Web 部署”复选框。如果禁用“Web 部署”，将无法从 Visual Studio 发布。可以将 IIS 和 Web 部署添加到任何打包的 Windows Server 映像，包括你自己的自定义映像。
8. 在“大小”列表中，选择虚拟机的大小。
9. 指定此虚拟机的登录凭据。请记下这些信息，因为在通过远程桌面访问计算机时需要这些信息。
10. 在“位置”列表中，选择虚拟机的托管区域。
11. 选择“确定”按钮开始创建虚拟机。可以在“输出”窗口中查看操作的进度。

	![][3]

12. 设置虚拟机时，系统会在解决方案的 **PublishScripts** 节点中创建发布脚本。发布脚本在 Azure 中运行并设置虚拟机。“输出”窗口将显示状态。脚本将执行以下操作来设置虚拟机：

	* 如果虚拟机尚不存在，则创建虚拟机。
	* 仅当指定的区域中没有名称以 `devtest` 开头的存储帐户时，才创建具有此名称的存储帐户。
	* 为虚拟机创建用作容器的云服务，并为网站创建 Web 角色。
	* 在虚拟机上配置 Web 部署。
	* 在虚拟机上配置 IIS 和 ASP.NET。

	![][4]

<br/> 13.（可选）你可以连接到新的虚拟机。在“服务器资源管理器”中，展开“虚拟机”节点，选择你创建的虚拟机节点，然后在其快捷方式菜单中，选择“使用远程桌面连接”。

 ![][5]


## 后续步骤

如果你要自定义所创建的发布脚本，请参阅[此处](http://msdn.microsoft.com/zh-cn/library/dn642480.aspx)了解更深入的信息。

[0]: ./media/virtual-machines-dotnet-create-visual-studio-powershell/CreateVM_NewProject.PNG
[1]: ./media/dotnet-visual-studio-create-virtual-machine/CreateVM_SignIn.PNG
[2]: ./media/virtual-machines-dotnet-create-visual-studio-powershell/CreateVM_CreateVM.PNG
[3]: ./media/virtual-machines-dotnet-create-visual-studio-powershell/CreateVM_Provisioning.png
[4]: ./media/virtual-machines-dotnet-create-visual-studio-powershell/CreateVM_SolutionExplorer.png
[5]: ./media/virtual-machines-dotnet-create-visual-studio-powershell/VS_Create_VM_Connect.png

<!---HONumber=67-->