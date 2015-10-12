## 设置存储连接字符串

用于 .NET 的 Azure 存储空间客户端库支持使用存储连接字符串来配置终结点和用于访问存储服务的凭据。我们建议您将连接字符串保存到配置文件中，而不是将其硬编码到应用程序中。您有两个选项来保存您的连接字符串：

- 如果您的应用程序在 Azure 云服务中运行，使用 Azure 服务配置系统（`*.csdef` 和 `*.cscfg` 文件）保存您的连接字符串。请参阅[如何创建和部署云服务](/documentation/articles/cloud-services-how-to-create-deploy)以了解有关 Azure 云服务配置的详细信息。
- 如果您的应用程序在 Azure 虚拟机中运行，或者如果您构建 .NET 应用程序将在 Azure 外运行，使用 .NET 配置系统（例如 `web.config` 或 `app.config` 文件）保存您的连接字符串。

在本指南的后面，我们将说明如何从您的代码检索您的连接字符串。

### 从 Azure 云服务配置连接字符串

Azure 云服务有一个特有的服务配置机制，让您能够从 Azure 管理门户动态地更改配置设置，而无需重新部署您的应用程序。

在 Azure 服务配置中配置连接字符串：

1.  在 Visual Studio 解决方案资源管理器内 Azure 部署项目的“角色”文件夹中，右键单击您的 Web 角色或辅助角色，然后单击“属性”。![在 Visual Studio 中选择云服务角色的属性][connection-string1]

2.  单击“设置”选项卡并按“添加设置”按钮。![在 Visual Studio 中添加云服务设置][connection-string2]

    之后，新的 **Setting1** 条目将显示在设置网格中。

3.  在新的 **Setting1** 条目的“类型”下拉列表中，选择“连接字符串”。![设置连接字符串类型][connection-string3]

4.  单击 **Setting1** 条目最右侧的 **...** 按钮。此时将打开“存储帐户连接字符串”对话框。

5.  选择是要定位到存储仿真程序（在本地计算机上模拟的 Windows Azure 存储），还是云中的存储帐户。本指南中的代码使用其中任一方式。

	> [AZURE.NOTE]您可以指向存储模拟器以避免引发与 Azure 存储空间有关的任何费用。但是，如果您确实选择指向云中的 Azure 存储帐户，则执行此教程的费用将会忽略不计。

	如果您指向云中的存储帐户，则输入该存储帐户的主访问密钥。若要了解如何通过 Azure 管理门户复制您的主访问密钥，请参阅[查看、复制和重新生成存储访问密钥](/documentation/articles/storage-create-storage-account#view-copy-and-regenerate-storage-access-keys)。

	> [AZURE.NOTE]您的存储帐户密钥类似于您的存储帐户的根密码。请务必保护好您的密钥。避免将其分发给其他用户或将其保存在其他人可以访问的纯文本文件中。如果您认为密钥可能已泄漏，请使用管理门户重新生成您的密钥。
	
    ![选择目标环境][connection-string4]

6.  将条目“名称”从 **Setting1** 更改为更友好的名称，例如 **StorageConnectionString**。稍后将在本指南的代码中引用此连接字符串。![更改连接字符串名称][connection-string5]
	
### 使用 .NET 配置来配置连接字符串

如果您编写的不是 Azure 云服务的应用程序（参见上一部分），则建议您使用 .NET 配置系统（例如 `web.config` 或 `app.config`）。这包括 Azure 网站或 Azure 虚拟机，以及设计为在 Azure 外部运行的应用程序。您可以使用 `<appSettings>` 元素存储连接字符串，如下所示。将 `account-name` 替换为您的存储帐户名称，将 `account-key` 替换为您的存储帐户密钥：

	<configuration>
  		<appSettings>
    		<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key" />
  		</appSettings>
	</configuration>

例如，您的配置文件中的配置设置可能类似于：

	<configuration>
    	<appSettings>
      		<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=nYV0gln9fT7bvY+rxu2iWAEyzPNITGkhM88J8HUoyofpK7C8fHcZc2kIZp6cKgYRUM74lHI84L50Iau1+9hPjB==" />
    	</appSettings>
	</configuration>

你现在即可准备执行本指南中的操作任务。

[connection-string1]: ./media/storage-configure-connection-string-include/connection-string1.png
[connection-string2]: ./media/storage-configure-connection-string-include/connection-string2.png
[connection-string3]: ./media/storage-configure-connection-string-include/connection-string3.png
[connection-string4]: ./media/storage-configure-connection-string-include/connection-string4.png
[connection-string5]: ./media/storage-configure-connection-string-include/connection-string5.png

[Configuring Connection Strings]: /documentation/articles/storage-configure-connection-string

<!---HONumber=70-->