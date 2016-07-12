<properties
	pageTitle="使用 CLI 管理密钥保管库 | Azure"
	description="使用本教程通过 CLI 自动执行密钥保管库中的常见任务"
	services="key-vault"
	documentationCenter=""
	authors="BrucePerlerMS"
	manager="mbaldwin"
	tags="azure-resource-manager"/>

<tags
	ms.service="key-vault"
	ms.date="04/29/2016"
	wacn.date="06/27/2016"/>

# 使用 CLI 管理密钥保管库 #
在大多数区域中提供了 Azure 密钥保管库。有关详细信息，请参阅[密钥保管库定价页](/home/features/key-vault/pricing/)。

## 介绍  
本教程将会帮助你开始使用 Azure 密钥保管库在 Azure 中创建强化容器（保管库），以存储和管理 Azure 中的加密密钥和机密。本教程将引导你完成使用 Azure 跨平台命令行接口创建包含密钥或密码（稍后可用于 Azure 应用程序）的保管库程序。然后，将会说明应用程序后续如何使用该密钥或密码。

**估计完成时间：**20 分钟。

>[AZURE.NOTE]  本教程未说明如何编写其中一个步骤所包括的 Azure 应用程序，但说明了如何授权应用程序使用密钥保管库中的密钥或机密。
>
>目前，无法在 Azure 门户中配置 Azure 密钥保管库。请改用这些跨平台命令行接口说明。或者，有关 Azure PowerShell 说明，请参阅[此对应教程](/documentation/articles/key-vault-get-started/)。

有关 Azure 密钥保管库的概述信息，请参阅[什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/)

## 先决条件

若要完成本教程，你必须准备好以下各项：

- Azure 订阅。如果你没有订阅，可以注册[试用版](/pricing/1rmb-trial)。
- 命令行接口版本 0.9.1 或更高版本。若要安装最新版本并连接到 Azure 订阅，请参阅[安装和配置 Azure 跨平台命令行接口](/documentation/articles/xplat-cli-install/)。
- 配置为使用你在本教程中所创建的密钥或密码的应用程序。你可以从 [Microsoft 下载中心](http://www.microsoft.com/en-us/download/details.aspx?id=45343)获取示例应用程序。有关说明，请参阅随附的自述文件。

## 获得 Azure 跨平台命令行接口帮助

本教程假定你熟悉命令行接口（Bash、终端、命令提示符）

可以使用 --help 或 -h 参数来查看特定命令的帮助。或者，也可以使用 azure help [命令] [选项] 格式返回相同信息。例如，以下命令都返回相同信息：

    azure account set --help

    azure account set -h

    azure help account set

当你对某一命令所需的参数有疑问时，请使用 --help、-h 或 azure help [命令] 来查看帮助。

还可阅读以下教程以熟悉如何在 Azure 跨平台命令行接口中使用 Azure 资源管理器：

- [如何安装和配置 Azure 跨平台命令行接口](/documentation/articles/xplat-cli-install/)
- [将 Azure 跨平台命令行接口用于 Azure 资源管理器](/documentation/articles/xplat-cli-azure-resource-manager/)


## 连接到订阅

要使用组织帐户登录，请使用以下命令：

    azure login -u username -p password -e azurechinacloud

或者
   
	azure login -u username -e azurechinacloud

>[AZURE.NOTE]  login 方法仅适用于组织帐户。组织帐户是指受组织管理、并在组织的 Azure Active Directory 租户中定义的用户。


如果您当前没有组织帐户，且已使用 Microsoft 帐户登录到 Azure 订阅，则您可以按照以下步骤轻松地创建一个组织帐户。

1.	登录到 [Azure 管理门户](https://manage.windowsazure.cn)，然后单击“Active Directory”。
2.	如果目录不存在，请选择“创建目录”，并提供所请求的信息。
3.	选择目录，并添加新用户。此新用户即为组织帐户。创建用户时，系统将为你提供用户电子邮件地址和临时密码。保存此信息，因为此信息还要用于另一个步骤。
4.	从门户中，选择“设置”，然后选择“管理员”。选择“添加”，并将新用户添加为共同管理员。这样组织帐户即可管理 Azure 订阅。
5.	最后，从 Azure 门户注销，然后使用新的组织帐户重新登录。如果这是使用此帐户首次登录，系统将提示更改密码。

有关在 Azure 中使用组织帐户的详细信息，请参阅[以组织身份注册 Azure](/documentation/articles/sign-up-organization/)。

如果你有多个订阅，并想要指定其中一个订阅供 Azure 密钥保管库使用，请键入以下内容以查看帐户的订阅：

    azure account list

然后，若要指定要使用的订阅，请键入：

    azure account set <subscription name>

有关配置 Azure 跨平台命令行接口的详细信息，请参阅[如何安装和配置 Azure 跨平台命令行接口](/documentation/articles/xplat-cli-install/)。


## 切换到使用 Azure 资源管理器

密钥保管库需要 Azure 资源管理器，因此请键入以下内容以切换到 Azure 资源管理器模式：

    azure config mode arm

## 创建新的资源组

使用 Azure 资源管理器时，会在资源组中创建所有相关资源。在本教程中，我们将创建新资源组“ContosoResourceGroup”。

    azure group create 'ContosoResourceGroup' 'China East'

第一个参数是资源组名称，第二个参数是位置。对于位置，请使用命令 `azure location list` 来了解如何针对本示例中的位置指定替代位置。如需更多信息，请键入：`azure help location`

## 注册密钥保管库资源提供程序
请确保在你的订阅中注册密钥保管库资源提供程序：

`azure provider register Microsoft.KeyVault`

此操作每个订阅仅需执行一次。


## 创建密钥保管库

使用 `azure keyvault create` 命令来创建密钥保管库。此脚本包含三个必需参数：资源组名称、密钥保管库名称和地理位置。

例如，如果使用的保管库名称为 ContosoKeyVault，资源组名称为 ContosoResourceGroup，位置为中国东部，请键入：

    azure keyvault create --vault-name 'ContosoKeyVault' --resource-group 'ContosoResourceGroup' --location 'China East'

此命令的输出会显示你刚刚创建的密钥保管库的属性。两个最重要的属性是：

- **名称**：在本示例中为 ContosoKeyVault。你将在其他密钥保管库 cmdlet 中使用此名称。
- **vaultUri**：在本示例中为 https://contosokeyvault.vault.chinacloudapi.cn 。通过其 REST API 使用保管库的应用程序必须使用此 URI。

你的 Azure 帐户现已获取在此密钥保管库上执行任何作业的授权。而且没有其他人有此授权。


## 将密钥或密码添加到密钥保管库

如果你希望 Azure 密钥保管库为你创建一个受软件保护的密钥，请使用 `azure key create` 命令，并键入以下内容：

    azure keyvault key create --vault-name 'ContosoKeyVault' --key-name 'ContosoFirstKey' --destination software

但是，如果你在保存为本地文件的 .pem 文件（名为 softkey.pem）中有现有密钥要上载到 Azure 密钥保管库，请键入以下命令以从 .PEM 文件（通过密钥保管库服务中的软件保护密钥）中导入该密钥：

    azure keyvault key import --vaultName 'ContosoKeyVault' --key-name 'ContosoFirstKey' --pem-file './softkey.pem' --password 'PaSSWORD' --destination software

现在，你可以通过使用密钥的 URI，引用已创建或上载到 Azure 密钥保管库的密钥。使用 **https://ContosoKeyVault.vault.chinacloudapi.cn/keys/ContosoFirstKey** 可始终获取当前版本，而使用 **https://ContosoKeyVault.vault.chinacloudapi.cn/keys/ContosoFirstKey/cgacf4f763ar42ffb0a1gca546aygd87** 可获取此特定版本。

若要将名为 SQLPassword 且其 Azure 密钥保管库的值为 Pa$$w0rd 的机密添加到保管库，请键入以下内容：

    azure keyvault secret set --vault-name 'ContosoKeyVault' --secret-name 'SQLPassword' --value 'Pa$$w0rd'

现在，你可以通过使用密码的 URI，引用已添加到 Azure 密钥保管库的此密码。使用 **https://ContosoVault.vault.chinacloudapi.cn/secrets/SQLPassword** 可始终获取当前版本，而使用 **https://ContosoVault.vault.chinacloudapi.cn/secrets/SQLPassword/90018dbb96a84117a0d2847ef8e7189d** 可获取此特定版本。

让我们查看一下刚刚创建的密钥或机密：

- 若要查看密钥，请键入：`azure keyvault key list --vault-name 'ContosoKeyVault'`
- 若要查看机密，请键入：`azure keyvault secret list --vault-name 'ContosoKeyVault'`


## 将应用程序注册到 Azure Active Directory

此步骤通常由开发人员在独立的计算机上完成。这并非 Azure 密钥保管库的特有状况，在此列出是为了让过程完整。


>[AZURE.IMPORTANT] 若要完成本教程，你的帐户、保管库以及将在本步骤中注册的应用程序全都必须位于相同的 Azure 目录中。

使用密钥保管库的应用程序必须使用 Azure Active Directory 的令牌进行身份验证。为此，应用程序的所有者首先必须在其 Azure Active Directory 中注册该应用程序。注册结束后，应用程序所有者将获得以下值：


- **应用程序 ID**（也称为客户端 ID）和**身份验证密钥**（也称为共享机密）。应用程序必须向 Azure Active Directory 提供这两个值才能获取令牌。如何将应用程序配置为执行此操作取决于应用程序。对于密钥保管库示例应用程序，应用程序所有者将在 app.config 文件中设置这些值。



要在 Azure Active Directory 中注册应用程序，请执行以下操作：

1. 登录到 Azure 门户。
2. 单击左侧的“Active Directory”，然后选择你要在其中注册应用程序的目录。<br><br>注意：你必须选择包含用于创建密钥保管库的 Azure 订阅的相同目录。如果你不知道是哪个目录，请单击“设置”，找到用于创建密钥保管库的订阅，并记下最后一列中显示的目录名称。

3. 单击“应用程序”。如果你的目录中尚未添加任何应用程序，则此页只会显示“添加应用程序”链接。单击该链接，或者单击命令栏上的“添加”。
4.	在“添加应用程序”向导的“要执行什么操作?”页面上，单击“添加我的组织正在开发的应用程序”。
5.	在“向我们说明你的应用程序”页上，指定应用程序名称，然后选择“WEB 应用程序和/或 WEB API”（默认值）。单击“下一步”图标。
6.	在“应用程序属性”页上，为你的 Web 应用程序指定“登录 URL”和“应用程序ID URI”。如果你的应用程序没有这些值，可以在此步骤中虚构这些值（例如，可以在两个框中指定 http://test1.contoso.com ）。这些网站是否存在并不重要；重要的是目录中每个应用程序的应用程序 ID URI 都不相同。目录会使用此字符串来识别你的应用程序。
7.	单击“完成”图标以保存向导中的更改。
8.	在“快速启动”页上，单击“映配置”。
9.	滚动到“密钥”部分，选择持续时间，然后单击“保存”。页面将会刷新，随后显示密钥值。你必须使用此密钥值和“客户端 ID”值来配置应用程序。（有关此配置的说明仅适用于特定的应用程序。）
10.	从此页中复制客户端 ID，后面的步骤将使用它来设置对保管库的权限。




## 授权应用程序使用密钥或密码

若要授权应用程序访问保管库中的密钥或机密，请使用 `azure keyvault set-policy` 命令。

例如，如果保管库名称是 ContosoKeyVault，要授权的应用程序的客户端 ID 为 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed，而你希望授权应用程序使用保管库中的密钥来进行解密和签名，那么，请执行以下操作：

    azure keyvault set-policy --vault-name 'ContosoKeyVault' --spn 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed --perms-to-keys '["decrypt","sign"]'

>[AZURE.NOTE] 如果你是在 Windows 命令提示符下运行，则应将单引号替换为双引号，并对内部双引号进行转义操作。例如："[\"decrypt\",\"sign\"]"。

如果要授权同一应用程序读取保管库中的机密，请运行以下命令：

	azure keyvault set-policy --vault-name 'ContosoKeyVault' --spn 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed --perms-to-secrets '["get"]'


## 删除密钥保管库以及关联的密钥和机密

如果你不再需要密钥保管库及其包含的密钥或机密，可以使用 azure keyvault delete 命令来删除密钥保管库：

    azure keyvault delete --vault-name 'ContosoKeyVault'

或者，你可以删除整个 Azure 资源组，其中包括密钥保管库和你加入该组的任何其他资源：

    azure group delete --name 'ContosoResourceGroup'


## 其他 Azure 跨平台命令行接口命令

可能有助于管理 Azure 密钥保管库的其他命令。

此命令列出以表格形式显示的所有密钥和所选属性：

    azure keyvault key list --vault-name 'ContosoKeyVault'

此命令显示特定密钥的完整属性列表：

    azure keyvault key show --vault-name 'ContosoKeyVault' --key-name 'ContosoFirstKey'

此命令列出以表格形式显示的所有机密名称和所选属性：

    azure keyvault secret list --vault-name 'ContosoKeyVault'

下面是演示如何删除特定密钥的示例：

    azure keyvault key delete --vault-name 'ContosoKeyVault' --key-name 'ContosoFirstKey'

下面是演示如何删除特定机密的示例：

    azure keyvault secret delete --vault-name 'ContosoKeyVault' --secret-name 'SQLPassword'


## 后续步骤

有关编程参考，请参阅 [Azure 密钥保管库开发人员指南](/documentation/articles/key-vault-developers-guide/)。

<!---HONumber=Mooncake_0620_2016-->