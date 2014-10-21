<properties linkid="manage-services-how-to-configure-a-cloud-service" urlDisplayName="How to configure" pageTitle="How to configure a cloud service - Azure" metaKeywords="Configuring cloud services" description="Learn how to configure cloud services in Azure. Learn to update the cloud service configuration and configure remote access to role instances." metaCanonical="" services="cloud-services" documentationCenter="" title="How to Configure Cloud Services" authors="davidmu" solutions="" manager="" editor="" />

# <span id="configurecloudservice"></span></a>如何配置云服务

[WACOM.INCLUDE [免责声明][免责声明]]

你可以在 Azure 管理门户中配置最常使用的云服务设置。或者，如果你希望直接更新配置文件，则可以下载要更新的服务配置文件，然后上载更新文件并使用配置更改更新云服务。无论使用哪种方法，配置更新都将应用于所有角色实例。

你还可以针对在云服务中运行的一个或所有角色启用“远程桌面”连接。通过远程桌面，你可以在应用程序正在运行时访问其桌面，并排查和诊断问题。即使你在应用程序开发过程中没有配置远程桌面服务定义文件 (.csdef)，你也可以对角色启用远程桌面连接。你无需重新部署应用程序，即可启用远程桌面连接。

如果每个角色至少具有两个角色实例（虚拟机），那么 Azure 在配置更新期间只能确保 99.95% 的服务可用性。这使得一台虚拟机可以在另一台虚拟机正更新时处理客户端请求。有关详细信息，请参阅[服务级别协议][服务级别协议]。

## 目录

-   [如何：更新云服务配置][如何：更新云服务配置]
-   [如何：配置对角色实例的远程访问][如何：配置对角色实例的远程访问]

## <span id="update"></span></a>如何：更新云服务配置

1.  在 [Azure 管理门户][Azure 管理门户]中单击“云服务”。然后单击云服务的名称以打开仪表板。

2.  单击**“配置”**。

    在“配置”页上，你可以配置监视、更新角色设置，并选择角色实例（虚拟机）的来宾操作系统和系列。

    ![配置页][配置页]

3.  在监视设置中，将监视级别设置为“详细”或“最少”，并配置详细监视所需的诊断连接字符串。有关说明，请参阅[如何监视云服务][如何监视云服务]。

4.  对于服务角色（按角色分组），你可以更新下列设置：

    > -   **设置**：修改服务配置 (.cscfg) 文件的 *ConfigurationSettings* 元素中指定的其他配置设置的值。

    > -   **证书**：更改 SSL 加密中用于角色的证书指纹。若要更改证书，你必须首先上载新证书（在“证书”页上）。然后更新角色设置中显示的证书字符串中的指纹。

5.  在“操作系统设置”中，你可以更改角色实例（虚拟机）的操作系统系列或版本，或选择“自动”以恢复当前操作系统版本的自动更新。操作系统设置适用于 Web 角色和辅助角色，但不影响已添加到以前 Azure 管理门户中的托管服务的 VM 角色。

    部署新的云服务时，你可以选择 Windows Server 2008 R2、Windows Server 2008 Service Pack 2 (SP2) 或 Windows Server 2012 操作系统。部署期间，所有角色实例都将安装最新版本的操作系统，并且默认情况下这些操作系统会自动更新。

    如果由于你代码中的兼容性要求而需要云服务在不同的操作系统版本上运行，则可选择操作系统系列和版本。当你选择一个特定的操作系统版本时，云服务的自动操作系统更新便挂起。你将需要确保操作系统接收更新。

    如果你使用最新版本的操作系统解决了应用程序中的所有兼容性问题，则可通过将操作系统版本设置成“自动”来恢复自动操作系统更新。

    ![操作系统设置][操作系统设置]

6.  若要保存你的配置设置，并将其推送至角色实例，请单击“保存”。（单击“丢弃”可取消更改。）更改设置后，命令栏中会出现“保存”和“丢弃”。

### 手动更新云服务配置文件

1.  下载包含当前配置的云服务配置文件 (.cscfg)。在云服务的“配置”页上，单击“下载”。然后单击“保存”或单击“另存为”以保存文件。

2.  更新服务配置文件后，上载并应用配置更新：

    a. 在“配置”页上，单击“上载”。

    将打开“上载新配置文件”。

    ![上载配置][上载配置]

    b. 在“配置文件”中，使用“浏览”选择已更新的 .cscfg 文件。

    c. 如果云服务包含任何只有一个实例的角色，请选中“即使一个或多个角色包含单个实例也应用配置”复选框以使这些角色的配置更新继续进行。

    除非每个角色至少定义两个实例，否则服务配置更新期间，Azure 无法保证至少 99.95% 的云服务可用性。有关详细信息，请参阅[服务级别协议][1]。

    d. 单击“确定”（复选标记）。

## <span id="remoteaccess"></span></a>如何：配置对角色实例的远程访问

你可以通过远程桌面访问在 Azure 中运行的角色的桌面。你可以使用远程桌面连接，在应用程序正在运行时排查和诊断其问题。你可以在应用程序设计期间，或者在将应用程序部署到 Azure 之后（此时角色正在运行），在角色中启用远程桌面连接。通过管理门户在运行的角色中启用远程桌面连接时，并不要求你重新部署应用程序。要对远程桌面连接进行身份验证，可以使用以前上载的证书或创建新证书。

在云服务的“配置”页上，你可以启用远程桌面，或者更改用于连接虚拟机的本地 Administrator 帐户或密码、身份验证使用的证书或到期日期。

<div class="dev-callout"> 
<b>说明</b> 
<p>如果你的云服务包含两个或更多个已连接的基于 Windows Server 的虚拟机，则你无需配置远程访问，因为会自动为这些虚拟机配置远程桌面。</p> 
</div>

### 在服务定义文件中配置远程访问

向服务定义文件 (.csdef) 中添加 **Import** 元素，以便将 RemoteAccess 和 RemoteForwarder 模块导入到服务模型中。当这些模块存在时，Azure 会将远程桌面的配置设置添加到服务配置文件中。若要完成远程桌面配置，你需要将证书导入到 Azure 中，并在服务配置文件中指定该证书。有关详细信息，请参阅[在 Azure 中为角色设置远程桌面连接][在 Azure 中为角色设置远程桌面连接]。

### 在管理门户中为角色实例启用或修改远程访问

1.  登录到[管理门户][Azure 管理门户]，然后单击“云服务”。然后单击云服务的名称以打开仪表板。

2.  打开云服务的“配置”页，然后单击“远程”。

    “配置远程桌面”将显示在部署云服务时已添加到服务配置文件的设置（如果有），如下所示。

    ![云服务远程][云服务远程]

> [WACOM.NOTE]
> **警告：**当首次启用远程桌面并单击“确定”（复选标记）时，所有角色实例会重新启动。为避免重新启动，必须对于此角色安装用于对密码进行加密的证书。如果未安装证书，你将看到此选项：
> ![CloudServices\_CreateNewCertDropDown][CloudServices\_CreateNewCertDropDown]

    To prevent a restart, install a certificate and then return to this dialog (see [Using Remote Desktop with Azure Roles][Using Remote Desktop with Azure Roles] for more information). If you choose an existing certificate, then a configuration update will be sent to all the instances in the role.

1.  在“角色”中，选择要更新的服务角色，或选择“全部”以选择所有角色。

2.  进行以下任何更改：

-   若要启用远程桌面，请选中“启用远程桌面”复选框。若要禁用远程桌面，请清除该复选框。

-   创建一个要在与角色实例的远程桌面连接中使用的帐户。

-   更新现有帐户的密码。

-   选择要用于身份验证的已上载证书（使用“证书”页上的“上载”），或者创建新证书。

-   更改远程桌面配置的到期日期。

1.  当你完成配置更新时，请单击“确认”（复选标记）。

2.  连接到角色实例：

    a. 单击“实例”打开“实例”页。

    b. 选择一个配置了远程桌面的角色实例。

    c. 单击“连接”，并按照说明打开虚拟机的桌面。

    d. 依次单击“打开”和“连接”以启动远程桌面连接。

### 在管理门户中为角色实例禁用远程访问

1.  登录到[管理门户][Azure 管理门户]，然后单击“云服务”。然后单击云服务的名称以打开仪表板。

2.  打开云服务的“配置”页，然后单击“远程”。

3.  在“角色”中，选择要更新的服务角色，或选择“全部”以选择所有角色。

4.  取消选中或清除“启用远程桌面”复选框。

5.  单击“确定”（复选标记）。

  [免责声明]: ../includes/disclaimer.md
  [服务级别协议]: https://www.windowsazure.com/zh-cn/support/legal/sla/
  [如何：更新云服务配置]: #update
  [如何：配置对角色实例的远程访问]: #remoteaccess
  [Azure 管理门户]: http://manage.windowsazure.cn/
  [配置页]: ./media/cloud-services-how-to-configure/CloudServices_ConfigurePage1.png
  [如何监视云服务]: ../how-to-monitor-a-cloud-service/
  [操作系统设置]: ./media/cloud-services-how-to-configure/CloudServices_ConfigurePage_OSSettings.png
  [上载配置]: ./media/cloud-services-how-to-configure/CloudServices_UploadConfigFile.png
  [1]: http://www.windowsazure.com/zh-cn/support/legal/sla/
  [在 Azure 中为角色设置远程桌面连接]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh124107.aspx
  [云服务远程]: ./media/cloud-services-how-to-configure/CloudServices_Remote.png
  [CloudServices\_CreateNewCertDropDown]: ./media/cloud-services-how-to-configure/CloudServices_CreateNewCertDropDown.png
