<properties
   pageTitle="设置命名的身份验证凭据 | Azure"
   description="了解如何提供 Visual Studio 可用于验证对 Azure 的请求的凭据，以便从 Visual Studio 将应用程序发布到 Azure 或者监视现有云服务。"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />  

<tags
    ms.assetid="61570907-42a1-40e8-bcd6-952b21a55786"
    ms.service="multiple"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="multiple"
    ms.date="11/18/2016"
    wacn.date="02/04/2017"
    ms.author="tarcher" />

# 设置命名的身份验证凭据

若要从 Visual Studio 中将应用程序发布到 Azure，或是要监视现有云服务，必须提供 Visual Studio 可用于验证对 Azure 的请求的凭据。Visual Studio 中有几个位置可以供你登录来提供这些凭据。例如，从服务器资源管理器，你可以打开“Azure”节点的快捷菜单，然后选择“连接到 Azure”。当你登录时，在 Visual Studio 中提供与你的 Azure 帐户关联的订阅信息，你什么也不需要做。

Azure Tools 还支持以前提供凭据的方式，即使用订阅文件（.publishsettings 文件）。本主题介绍了此方法，它在 Azure SDK 2.2 中仍受支持。

在 Azure 中进行身份验证需要做好以下准备：

- 你的订阅 ID

- 有效的 X.509 v3 证书

>[AZURE.NOTE] X.509 v3 证书的密钥长度必须至少为 2048 位。Azure 将拒绝所有不符合此要求或无效的证书。

Visual Studio 使用订阅 ID 和证书数据作为凭据。订阅文件（.publishsettings 文件）中引用了适当的凭据，该文件包含证书的公共密钥。该订阅文件可以包含多个订阅的凭据。

从“新建/编辑订阅”对话框中，可以编辑订阅信息，如本主题后面所述。

如果要自行创建证书，可以参阅 [创建并上传 Azure 管理证书](/documentation/articles/cloud-services-certs-create/)中的说明，然后将证书手动上传到[Azure 经典管理门户](http://manage.windowsazure.cn/)。

>[AZURE.NOTE] Visual Studio 管理云服务所需的凭据不同于针对 Azure 存储空间服务验证请求所需的凭据。

## 在 Visual Studio 中修改或导出身份验证凭据

在“新建订阅”对话框中，还可以设置、修改或导出身份验证凭据。该对话框在执行下列任一操作时出现：

- 在“服务器资源管理器”中，打开“Azure”节点的快捷菜单，选择“管理订阅”，选择“证书”选项卡，然后选择“添加”或“编辑”按钮。

- 从“发布 Azure 应用程序”向导发布 Azure 云服务时，请在“选择订阅”列表中选择“管理”，选择“证书”选项卡，然后选择“添加”或“编辑”按钮。

以下过程假设“新建订阅”对话框已打开。

### 在 Visual Studio 中设置身份验证凭据

1. 在身份验证列表的“选择现有证书”中选择一个证书。

1. 选择“复制完整路径”按钮。证书（.cer 文件）的路径将复制到剪贴簿。

    >[AZURE.IMPORTANT] 若要从 Visual Studio 中发布 Azure 应用程序，必须将此证书上载到 [Azure 经典管理门户](http://manage.windowsazure.cn/)。

1. 将证书上载到 [Azure 经典门户](http://manage.windowsazure.cn/)，请执行以下操作：

    1. 选择“Azure 经典管理门户”链接。

         [Azure 经典管理门户](http://manage.windowsazure.cn/)随即打开。

    1. 登录到 [Azure 经典管理门户](http://manage.windowsazure.cn/)，然后选择“云服务”按钮。

    1. 选择所需的云服务。

        此时将打开该服务的页面。

    1. 在“证书”选项卡上选择“上载”按钮。

    1. 粘贴刚刚创建 .cer 文件的完整路径，然后输入指定的密码。

<!---HONumber=Mooncake_0815_2016-->