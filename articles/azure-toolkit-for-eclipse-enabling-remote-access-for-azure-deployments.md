<properties
    pageTitle="在 Eclipse 中为 Azure 部署启用远程访问"
    description="了解如何使用 Azure Toolkit for Eclipse 为 Azure 部署启用远程访问。"
    services=""
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor=""/>

<tags
    ms.service="multiple"
    ms.date="05/04/2016" 
    wacn.date="06/20/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690951.aspx -->

# 在 Eclipse 中为 Azure 部署启用远程访问 #

为了帮助排查部署问题，你可以启用并使用远程访问连接到托管部署的虚拟机。远程访问功能依赖于远程桌面协议 (RDP)。你可以在将部署发布到 Azure 后为部署配置远程访问。或者，如果你在 Windows 操作系统中使用 Eclipse，可以在发布到 Azure 之前配置远程访问。请注意，若要连接到 Azure 中的部署的虚拟机，需要使用与操作系统兼容的远程桌面客户端。

## 如何在部署到 Azure 之前启用远程访问 ##

>[AZURE.NOTE] 若要在将应用程序部署到 Azure 之前启用远程访问，你需要在 Windows 上运行 Eclipse。

下图显示了用于启用远程访问的“远程访问”属性对话框。

![][ic719494]

可通过两种方法显示“远程访问”属性对话框：

* 在“发布到 Azure”对话框的“远程访问”部分中单击“高级”链接。
* 打开 Azure 项目的“属性”对话框。

当你创建新的 Azure 部署项目时，默认情况下不会为该项目启用远程访问。但是，你可以通过在“发布到 Azure”对话框中指定用户名和密码轻松启用远程访问。远程访问密码已使用 X.509 证书加密。如果你未提供自己的证书，加密将依赖于 Azure Plugin for Eclipse 随附的自签名证书。此自签名证书位于 Azure 项目的 **cert** 文件夹中，它存储为公共证书文件 (SampleRemoteAccessPublic.cer) 和个人信息交换 (PFX) 证书文件 (SampleRemoteAccessPrivate.pfx)。后者包含证书的私钥，并具有默认密码 **Password1**。但是，由于此密码是公开的，因此默认证书只应该用于学习目的，而不能用于生产部署。当你出于学习目的以外的其他目的想要为部署启用远程会话时，应该在“发布到 Azure”对话框中单击“高级”链接，以指定你自己的证书。请注意，你需要将该证书的 PFX 版本上载到 Azure 管理门户中的托管服务，使 Azure 能够解密用户密码。

本教程的余下部分将说明如何为最初创建的已禁用远程访问的 Azure 部署项目启用远程访问。在本教程中，我们将要创建一个新的自签名证书，其 .pfx 文件包含你选择的密码。你也可以选择使用证书颁发机构颁发的证书。

## 如何在部署到 Azure 之后启用远程访问 ##

若要在部署到 Azure 之后启用远程访问，请使用以下步骤：

1. 使用你的 Azure 帐户登录到 Azure 管理门户
1. 在“云服务”列表中，选择你的已部署的云服务
1. 在云服务网页中，单击“配置”链接
1. 在配置页的底部，单击“远程”链接
1. 当出现弹出式对话框时：
    1. 指定你要为其启用远程访问的角色
    1. 单击“启用远程桌面”复选框以将其选中
    1. 指定要用于远程访问的用户名和密码
    1. 选择要使用的证书
1. 单击**“确定”** 

此时，你将看到一条消息，指出正在更改你的配置，这可能需要几分钟才能完成。在完成配置更改后，遵照本文后面的**远程登录**部分中的步骤。
	
## 如何在包中启用远程访问 ##

* 在 Eclipse 的“项目资源管理器”窗格中，右键单击你的 Azure 项目并单击“属性”。
* 在“属性”对话框的左窗格中展开“Azure”，然后单击“远程访问”。
* 在“远程访问”对话框中，确保已选中“允许所有角色使用这些登录凭据接受远程桌面连接”。
* 指定用于远程桌面连接的用户名。
* 指定并确认用户的密码。当你建立远程桌面连接时，将使用此对话框中设置的用户名和密码值。（请注意，此密码不同于你的 PFX 密码）。
* 指定用户帐户的过期日期。
* 单击“新建”创建新的自签名证书。（或者，你可以通过“工作区”或“文件系统”按钮来分别从工作区或文件系统中选择一个证书，但对于本教程，我们将创建一个新证书。）
* 在“新建证书”对话框中，指定并确认你要用于 PFX 文件的密码。
* 接受提供给“名称(CN)”的值，或使用自定义名称。
* 指定新证书的保存路径和文件名（使用 .cer 格式）。对于此步骤和下一步骤，你可以使用 Azure 项目的 **cert** 文件夹，但你可以自由选择其他位置。对于此教程，我们将使用 **c:\\mycert\\mycert.cer**。（在继续下一步之前创建 **c:\\mycert** 文件夹，或者视需要使用现有文件夹。）
* 指定新证书及其私钥的保存路径和文件名（使用 .pfx 格式）。对于此教程，我们将使用 **c:\\mycert\\mycert.pfx**。“新建证书”对话框应与下图类似（如果你未使用 **c:\\mycert**，文件夹路径将会更新）：![][ic712275]
* 单击“确定”关闭“新建证书”对话框。
* “远程访问”对话框应与下图类似：</p>![][ic719495]
* 单击“确定”关闭“远程访问”对话框。
	
使用部署到云的生成设置重新生成你的应用程序。

## 远程登录 ##

准备好角色实例后，你可以远程登录到托管应用程序的虚拟机。

* 如果在 Windows 上使用 Eclipse 并且在部署到 Azure 期间选择了“部署时启动远程桌面”选项，则在启动部署时，系统会显示“远程桌面连接”登录屏幕。当系统提示你输入用户名和密码时，请输入你为远程用户指定的值，然后即可登录。
* 远程登录的另一种方法是通过 <a href="http://go.microsoft.com/fwlink/?LinkID=512959">Azure 管理门户</a>。
    * 在 Azure 管理门户的“云服务”视图中，依次单击你的云服务、“实例”、特定的实例、“连接”按钮。命令栏中会出现如下所示的“连接”按钮：
    ![][ic659273]  
    >[AZURE.NOTE] 如果你在非 Windows 操作系统上操作，则需要使用与你的操作系统兼容的远程桌面客户端，并遵照相应的步骤以使用已下载 RDP 文件中的设置配置该客户端。
    * 单击“连接”按钮后，系统会提示你打开 RDP 文件。打开该文件并按照提示操作。（你也可以将此文件保存到本地计算机，然后通过双击来运行该文件，这样即可远程登录到虚拟机，而无需首先进入管理门户。）
    * 当系统提示你输入用户名和密码时，请输入你为远程用户指定的值，然后即可登录。

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包][]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序][]

[安装 Azure Toolkit for Eclipse][]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心][]。

<!-- URL List -->

[Azure Java 开发人员中心]: /develop/java/
[Azure Management Portal]: http://manage.windowsazure.cn
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/

<!-- IMG List -->

[ic712275]: ./media/azure-toolkit-for-eclipse-enabling-remote-access-for-azure-deployments/ic712275.png
[ic719495]: ./media/azure-toolkit-for-eclipse-enabling-remote-access-for-azure-deployments/ic719495.png
[ic719494]: ./media/azure-toolkit-for-eclipse-enabling-remote-access-for-azure-deployments/ic719494.png
[ic659273]: ./media/azure-toolkit-for-eclipse-enabling-remote-access-for-azure-deployments/ic659273.png

<!---HONumber=Mooncake_0215_2016-->