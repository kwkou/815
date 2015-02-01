<properties linkid="dev-node-remotedesktop" urlDisplayName="Enable Remote Desktop" pageTitle="对云服务启用远程桌面 (Node.js))" metaKeywords="Azure Node.js remote access, Azure Node.js remote connection, Azure Node.js VM access, Azure Node.js virtual machine access" description="了解如何对托管 Azure Node.js 应用程序的虚拟机进行远程桌面访问。 " metaCanonical="" services="cloud-services" documentationCenter="Node.js" title="Enabling Remote Desktop in Azure" authors="larryfr" solutions="" manager="" editor="" />






# 在 Azure 中启用远程桌面

你可以通过远程桌面访问在 Azure 中运行的角色实例的桌面。您可以使用远程桌面连接配置虚拟机，或者排查应用程序问题。

<div class="dev-callout">
	<b>说明</b>
	<p>本文中的步骤仅适用于托管为 Azure 云服务的 Node 应用程序。</p>
	</div>

此任务包括下列步骤：

-   [步骤 1：使用 Azure PowerShell 配置远程桌面访问服务]
-   [步骤 2：连接到角色实例]
-   [步骤 3：使用 Azure PowerShell配置禁用
    远程桌面访问的服务]

## <a name="step1"> </a>步骤 1：使用 Azure PowerShell 配置远程桌面访问服务

要使用远程桌面，你需要使用用户名、密码和证书配置服务定义和服务配置，以向云中的角色实例进行身份验证。[Azure PowerShell] 中包含 **Enable-AzureServiceProjectRemoteDesktop** cmdlet，它会为你完成此配置。

在创建服务定义的计算机上执行以下步骤。

1.  从"开始"菜单中，选择"Azure PowerShell"********。

	![Azure PowerShell start menu entry][powershell-menu]

2.  将目录切换到服务目录，键入 **Enable-AzureServiceProjectRemoteDesktop**，然后输入要向云中的角色实例进行身份验证时使用的用户名和密码。

	![enable-azureserviceprojectremotedesktop][enable-rdp]

3.  将服务配置更改发布到云。在 **Azure PowerShell** 提示符下，键入 **Publish-AzureServiceProject**。

	![publish-azureserviceproject][publish-project]

完成这些步骤后，将在云中为远程桌面访问配置服务的角色实例。

## <a name="step2"> </a>步骤 2：连接到角色实例

当你的部署在 Azure 中启动并运行后，便可以连接到角色实例。

1.  在 [Azure 管理门户]中，选择"云服务"****，然后选择在上面的步骤 1 中部署的服务

	![azure management portal][cloud-services]

2.  单击"实例"，然后单击"生产"或"过渡"查看你的服务实例************。选择一个实例，然后单击页面底部的"连接"****。

    ![The instances page][3]

2.  单击"连接"后，Web 浏览器会提示你保存 .rdp 文件****。如果你使用的是 Internet Explorer，请单击"打开"。 
    ![prompt to open or save the .rdp file][4]

3.  打开该文件时，会显示以下安全提示：

    ![Windows security prompt][5]

4.  单击"连接"，将显示一个安全提示，要求输入访问该实例的凭据****。输入在[步骤 1][Step 1: Configure the service for Remote Desktop access using Azure PowerShell] 中创建的密码，然后单击"确定"****。

    ![username/password prompt][6]

建立连接后，远程桌面连接将在 Azure 中显示该实例的桌面。你已成功获取对该实例的远程访问权限，并且可以执行任何必需任务来管理应用程序。

![Remote desktop session][7]

## <a name="step3"> </a>步骤 3：使用 Azure PowerShell 配置禁用远程桌面访问的服务

当你不再需要与云中角色实例的远程桌面连接时，可使用 [Azure PowerShell] 禁用远程桌面访问

1.  从"开始"菜单中，选择"Azure PowerShell"********。

2.  将目录切换到服务目录，并键入 **Disable-AzureServiceProjectRemoteDesktop**：

3.  将服务配置更改发布到云。在 **Azure PowerShell** 提示符下，键入 **Publish-AzureServiceProject**：

## 其他资源

- [远程访问 Azure 中的角色实例]
- [将远程桌面与 Azure 角色一起使用]

  [步骤 1：使用 Azure PowerShell 配置远程桌面访问服务]: #step1
  [步骤 2：连接到角色实例]: #step2
  [步骤 3：使用 Azure PowerShell 配置禁用远程桌面访问的服务]: #step3
  [Azure PowerShell]: http://go.microsoft.com/?linkid=9790229&clcid=0x409

[Azure 管理门户]: http://manage.windowsazure.cn
[powershell-menu]: ./media/cloud-services-nodejs-enable-remote-desktop/azure-powershell-menu.png
[publish-project]: ./media/cloud-services-nodejs-enable-remote-desktop/publish-rdp.png
[enable-rdp]: ./media/cloud-services-nodejs-enable-remote-desktop/enable-rdp.png
[cloud-services]: ./media/cloud-services-nodejs-enable-remote-desktop/cloud-services-remote.png
  [3]: ./media/cloud-services-nodejs-enable-remote-desktop/cloud-service-instance.png
  [4]: ./media/cloud-services-nodejs-enable-remote-desktop/rdp-open.png
  [5]: ./media/cloud-services-nodejs-enable-remote-desktop/remote-desktop-12.png
  [6]: ./media/cloud-services-nodejs-enable-remote-desktop/remote-desktop-13.png
  [7]: ./media/cloud-services-nodejs-enable-remote-desktop/remote-desktop-14.png
  
  [远程访问 Azure 中的角色实例]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh124107.aspx
  [将远程桌面与 Azure 角色一起使用]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg443832.aspx
<!--HONumber=39-->
