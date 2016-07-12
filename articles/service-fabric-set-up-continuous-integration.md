<properties
   pageTitle="Service Fabric 的持续集成 | Azure"
   description="大致了解如何使用 Visual Studio Team Services (VSTS) 为 Service Fabric 应用程序设置持续集成。"
   services="service-fabric"
   documentationCenter="na"
   authors="cawams"
   manager="timlt"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="03/29/2016"
   wacn.date="07/07/2016" />

# 使用 Visual Studio Team Services 为 Service Fabric 应用程序设置持续集成

本文介绍使用 Visual Studio Team Services (VSTS) 为 Azure Service Fabric 应用程序设置持续集成，从而确保以自动方式生成、打包和部署应用程序的步骤。请注意，这些说明每次都将从零开始重新创建群集。

本文档反映当前过程，应随时间推移而更改。

## 先决条件

首先，请在 Visual Studio Team Services 上设置项目：

1. 如果你尚未创建 Team Services 帐户，请使用你的 [Microsoft 帐户](http://www.microsoft.com/account)设置它。

2. 使用 Microsoft 帐户在 Team Services 上创建新项目。

3. 将新的或现有 Service Fabric 应用的源推送到此项目。

有关处理 Team Services 项目的详细信息，请参阅[连接到 Visual Studio](https://www.visualstudio.com/get-started/setup/connect-to-visual-studio-online)。

## 设置服务主体

### 为自动化设置身份验证

你需要先创建生成代理将用于对 Azure 进行身份验证的[服务主体](/documentation/articles/resource-group-create-service-principal-portal/)，才能设置生成计算机。你还需要创建证书并将其上载到 Azure 密钥保管库，因为密钥保管库不支持服务主体身份验证。可以从任意计算机进行这些步骤。你的开发计算机是一个不错的选择。

### 安装 Azure PowerShell 并登录

1.  安装 PowerShellGet。

    a.如果正在运行具有最新更新的 Windows 10，则可以跳过此步骤（已安装 PowerShellGet）。

    b.如果未运行，则安装 [Windows Management Framework 5.0](http://www.microsoft.com/download/details.aspx?id=48729)，它包括 PowerShellGet。

2.	安装和更新 AzureRM 模块。
如果你安装了 Azure PowerShell 的任何以前版本，请将其删除：

    a.右键单击“开始”按钮，然后选择“添加/删除程序”。

    b.搜索“Azure PowerShell”并将其卸载。

    c.打开 PowerShell 命令提示符。

    d.使用命令 `Install-Module AzureRM` 安装 AzureRM 模块。

    e.使用命令 `Update-AzureRM` 更新 AzureRM 模块。

3.	禁用（或启用）Azure 数据收集。

    Azure cmdlet 将提示你选择加入或退出数据收集，直到你做出选择为止。这些提示将阻止自动化，同时等待用户输入。若要取消这些提示，请通过运行下列命令之一提前进行选择：

    - Enable-AzureRmDataCollection

    - 禁用 AzureRmDataCollection

4.	登录到 Azure PowerShell：

    a.运行命令 `Login-AzureRmAccount -EnvironmentName AzureChinaCloud`。

    b.在出现的对话框中，输入你的 Azure 凭据。

    c.运行命令 `Get-AzureRmSubscription`。

    d.找到你想要使用的订阅。

    e.运行命令 `Select-AzureRmSubscription -SubscriptionId <ID for your subscription>`。

### 创建服务主体

1. 请按照[这些说明](https://blogs.msdn.microsoft.com/visualstudioalm/2015/10/04/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/)，为你的项目创建服务主体和服务终结点。

2. 请注意，值会打印在脚本输出的末尾。你需要这些值才能设置生成定义。

### 创建证书并将其上传到新的 Azure 密钥保管库

>[AZURE.NOTE] 此示例脚本生成自签名证书，这不是安全的做法，仅可接受将其用于试验。按照组织的指导原则改为获取合法证书。这些说明也会针对服务器和客户端使用单一证书。在生产环境中，你应该使用不同的服务器和客户端证书。

1. 下载 [ServiceFabricContinuousIntegrationScripts.zip](https://gallery.technet.microsoft.com/Set-up-continuous-f8b251f6) 并将其解压缩到此计算机上的文件夹。

2. 在管理员 PowerShell 提示符下，更改为目录 `<extracted zip>/Manual`。

3. 使用以下参数运行 PowerShell 脚本 `CreateAndUpload-Certificate.ps1`：

| 参数 | 值 |
| --- | --- |
| KeyVaultLocation | 任何值。此参数必须与你打算在其中创建群集的位置匹配。 |
| CertificateSecretName | 任何值。 |
| CertificateDnsName | 必须匹配群集的 DNS 名称。 |
| SecureCertificatePassword | 任何值。在生成计算机上导入证书时，使用此参数。 |
| KeyVaultResourceGroupName | 任何值。但是，请勿使用计划用于群集的资源组名称。 |
| KeyVaultName | 任何值。 |
| PfxFileOutputPath| 任何值。使用此文件将证书导入到生成计算机上。 |

在脚本结束后，它将输出下列三个值。请注意这些值，因为它们被用作生成变量。

 - `ServiceFabricCertificateThumbprint`
 - `ServiceFabricKeyVaultId`
 - `ServiceFabricCertificateSecretId`

## 设置生成计算机

### 安装 Visual Studio 2015

如果你已经预配计算机（或打算提供你自己的计算机），请在所选计算机上安装 [Visual Studio 2015](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx)。

如果还没有计算机，可以使用预安装的 Visual Studio 2015 快速预配 Azure 虚拟机 (VM)。为此，请按以下步骤操作：

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。

2. 选择屏幕左上角的“新建”命令。

3. 选择“应用商店”。

4. 搜索 **Visual Studio 2015**。

5. 选择“计算”>“虚拟机”>“从库中”。

6. 选择映像 **Visual Studio Enterprise 2015 Update 1 With Azure SDK 2.8 on Windows Server 2012 R2**。

    >[AZURE.NOTE] Azure SDK 不是必需组件，但是当前没有任何可用的且仅安装有 Visual Studio 2015 的映像。

7.	按照对话框中的说明创建 VM。

### 安装 Service Fabric SDK

在计算机上安装 [Service Fabric SDK](/home/features/service-fabric)。

### 安装 Azure PowerShell

若要安装 Azure PowerShell，请遵循上一节“安装 Azure PowerShell 并登录”中的步骤。跳过“登录到 Azure PowerShell”步骤。

### 使用网络服务帐户注册 Azure PowerShell 模块

>[AZURE.NOTE] 执行这项操作*之后*再启动生成代理。否则，它不会接受新的环境变量。

1. 按 Windows 徽标键 + R，键入 **regedit**，再按 Enter。

2. 右键单击节点 `HKEY_Users\.Default\Environment`，然后选择“新建”>“可扩展字符串值”。

3. 输入 `PSModulePath` 作为名称，输入 `%PROGRAMFILES%\WindowsPowerShell\Modules` 作为值。将 `%PROGRAMFILES%` 替换为 `PROGRAMFILES` 环境变量的值。

### 导入你的自动化证书

1.	将证书导入到你的生成计算机上。为此，请按以下步骤操作：

    a.将由脚本 CreateAndUpload-Certificate.ps1 创建的 PFX 文件复制到生成计算机。

    b.使用你之前传递给 `CreateAndUpload-Certificate.ps1` 的密码，打开管理员 PowerShell 提示符并运行以下命令。

        ```
        $password = Read-Host -AsSecureString
        Import-PfxCertificate -FilePath <path/to/cert.pfx> -CertStoreLocation Cert:\LocalMachine\My -Password $password -Exportable
        ```

2.	运行证书管理器：

    a.在 Windows 中打开控制面板。右键单击“开始”按钮，然后选择“控制面板”。

    b.搜索**证书**。

    c.选择“管理工具”>“管理计算机证书”。

3.	授权网络服务帐户使用自动化证书：

    a.在“证书 - 本地计算机”下，展开“个人”，然后选择“证书”。

    b.在列表中查找你的证书。

    c.右键单击你的证书，然后选择“所有任务”>“管理私钥”。

    d.选择“添加”按钮，输入“网络服务”，然后选择“检查名称”。

    e.选择“确定”，然后关闭证书管理器。

    ![屏幕截图：授予本地服务帐户权限的步骤](./media/service-fabric-set-up-continuous-integration/windows-certificate-manager.png)

4.  将证书复制到 `Trusted People` 文件夹。

    a.你的证书已导入到“个人/证书”，但我们需要将其添加到“受信任人”。右键单击证书并选择“复制”。然后，右键单击“受信任人”文件夹，并选择“粘贴”。

### 注册你的生成代理

1.	下载 agent.zip。为此，请按以下步骤操作：

    a.登录到你的团队项目，如 **https://[your-VSTS-account-name].visualstudio.com**。

    b.选择屏幕右上角的齿轮图标。

    c.选择“代理池”选项卡。

    d.选择“下载代理”下载 agent.zip 文件。

    >[AZURE.NOTE] 如果未开始下载，请检查你的弹出窗口阻止程序。

    e.将 agent.zip 复制到先前创建的生成计算机。

    f.将 agent.zip 解压缩到生成计算机上的 `C:\agent`（或任何具有短路径的位置）。

    >[AZURE.NOTE] 如果你打算使用 ASP.NET 5 Web 服务，建议为此文件夹选择可能的最短名称，以免在部署期间遇到 **PathTooLongExceptions** 错误。当 ASP.NET Core 发行时，即可缓解此问题。

2.	在管理员命令提示符下，运行 `C:\agent\ConfigureAgent.cmd`。该脚本会提示你输入以下参数：

|参数|值|
|---|---|
|代理名称|接受默认值 `Agent-[machine name]`。|
|TFS Url|输入团队项目的 URL，如 `https://[your-VSTS-account-name].visualstudio.com`。|
|代理池|输入代理池的名称。（如果尚未创建代理池，则接受默认值。）|
|工作文件夹|接受默认值。生成代理将在此文件夹中实际生成你的应用程序。如果你打算使用 ASP.NET 5 Web 服务，建议为此文件夹选择可能的最短名称，以免在部署期间遇到 PathTooLongExceptions 错误。|
|安装为 Windows 服务？|默认值是 N。将该值更改为 **Y**。|
|用于运行服务的用户帐户|接受默认值 `NT AUTHORITY\NetworkService`。|
|`NT AUTHORITY\Network Service` 的密码|网络服务帐户没有密码，但会拒绝空白密码。输入任何非空字符串作为密码（将忽略你所输入的内容）。|
|是否取消配置现有代理？|接受默认值 **N**。|

3.  当系统提示你输入凭据时，输入有权访问你的团队项目的 Microsoft 帐户的凭据。

4.  请确认你的生成代理已注册并配置其功能。为此，请按以下步骤操作：

    a.返回到 Web 浏览器 (`https://[your-VSTS-account-name].visualstudio.com/_admin/_AgentPool`)，并刷新页面。

    b.选择你之前在运行 ConfigureAgent.ps1 时选择的代理池。

    c.验证你的生成代理显示于列表中且其状态突出显示为绿色。如果突出显示为红色，则表示生成代理连接到 Team Services 时遇到问题。

    ![屏幕截图：显示生成代理的状态](./media/service-fabric-set-up-continuous-integration/vso-configured-agent.png)

    d.选择生成代理，然后选择“功能”选项卡。

    e.添加名为 **azureps** 的功能以及任何值。对 VSTS 而言，这表示此计算机上已安装 Azure PowerShell，而需有该模块才能使用 VSTS 提供的某些生成任务。


## 创建生成定义

>[AZURE.NOTE] 即使在单独的计算机上，按照这些说明创建的生成定义也不支持多个并发生成。这是因为每个生成会争夺同一个资源组/群集。如果想要运行多个生成代理，你将需要修改以下说明/脚本，以防止这种干扰。

### 将 Service Fabric Azure Resource Manager 模板添加到应用程序

1. 从[此示例](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype-wad)下载 `azuredeploy.json` 和 `azuredeploy.parameters.json`。

2. 打开 `azuredeploy.parameters.json` 并编辑以下参数：

    |参数|值|
    |---|---|
    |clusterLocation|必须匹配密钥保管库的位置。示例：`westus`|
    |clusterName|必须匹配证书的 DNS 名称。例如，如果证书的 DNS 名称为 `mycluster.westus.cloudapp.net`，则 `clusterName` 必须为 `mycluster`。|
    |adminPassword|8-123 个字符，包含以下字符类型中的至少 3 种类型：大写、小写、数字、特殊字符。|
    |certificateThumbprint|来自 `CreateAndUpload-Certificate.ps1` 的输出|
    |sourceVaultValue|来自 `CreateAndUpload-Certificate.ps1` 的输出|
    |certificateUrlvalue|来自 `CreateAndUpload-Certificate.ps1` 的输出|

3. 将新文件添加到源控件，并推送到 VSTS。

### 创建生成定义

1.	创建空的生成定义。为此，请按以下步骤操作：

    a.在 Visual Studio Team Services 中打开你的项目。

    b.选择“生成”选项卡。

    c.选择绿色 **+** 号以创建新的生成定义。

    d.选择“空”，然后选择“下一步”。

    e.验证是否选择了正确的存储库和分支。

    f.选择向其注册生成代理的代理队列，然后选中“持续集成”复选框。

2.	在“变量”选项卡上，使用这些值创建以下变量。

    |变量|值|密钥|允许在队列时|
    |---|---|---|---|
    |BuildConfiguration|版本||X|
    |BuildPlatform|x64||||

3.  保存生成定义并为其命名。稍后可根据需要更改此名称。

### 添加“还原 NuGet 包”步骤

1. 在“生成”选项卡上，选择“添加生成步骤...”命令。

2. 选择“包”>“NuGet 安装程序”

3. 选择生成步骤名称旁边的铅笔图标，并将其重命名为“还原 NuGet 包”。

4. 选择“解决方案”字段旁边的“...”按钮，然后选择你的 .sln 文件。

5. 保存生成定义。

### 添加“生成”步骤。

1.	在“生成”选项卡上，选择“添加生成步骤...”命令。

2.	选择“生成”>“MSBuild”。

3.	选择生成步骤名称旁边的铅笔图标，然后将其重命名为“生成”。

4. 选择以下值：

    |设置名称|值|
    |---|---|
    |解决方案|单击“...”按钮，然后选择解决方案的 `.sln` 文件。|
    |平台|`$(BuildPlatform)`|
    |配置|`$(BuildConfiguration)`|

5.	保存生成定义。

### 添加“包”步骤

1.	在“生成”选项卡上，选择“添加生成步骤...”命令。

2.	选择“生成”>“MSBuild”。

3.	选择生成步骤名称旁边的铅笔图标，然后将其重命名为“包”。

4. 选择以下值：

    |设置名称|值|
    |---|---|
    |解决方案|单击“...”按钮，然后选择应用程序项目的 `.sfproj` 文件。|
    |平台|`$(BuildPlatform)`|
    |配置|`$(BuildConfiguration)`|
    |MSBuild 参数|`/t:Package`|

5.	保存生成定义。

### 添加“删除群集资源组”步骤

如果上一个生成在其自身后未清理（例如，如果由于取消了生成而未完成清理），可能存在与新资源组冲突的现有资源组。为了避免冲突，在创建新资源组前，请清理任何剩余的资源组（及其关联资源）。

1.	在“生成”选项卡上，选择“添加生成步骤...”命令。

2.	选择“部署”>“Azure 资源组部署”。

3.	选择生成步骤名称旁边的铅笔图标，然后将其重命名为“删除群集资源组”。

4. 选择以下值：

    |设置名称|值|
    |---|---|
    |AzureConnectionType|**Azure 资源管理器**|
    |Azure RM 订阅|选择你在“创建服务主体”部分中创建的连接终结点。|
    |操作|**删除资源组**|
    |资源组|输入任何未使用的名称。在下一步中，必须使用相同的名称。|

5.	保存生成定义。

### 添加“预配安全群集”步骤

1.	在“生成”选项卡上，选择“添加生成步骤...”命令。

2.	选择“部署”>“Azure 资源组部署”。

3.	选择生成步骤名称旁边的铅笔图标，然后将其重命名为“预配安全群集”。

4. 选择以下值：

    |设置名称|值|
    |---|---|
    |AzureConnectionType|**Azure 资源管理器**|
    |Azure RM 订阅|选择你在“创建服务主体”部分中创建的连接终结点。|
    |操作|**创建或更新资源组**|
    |资源组|必须匹配你在上一步中使用的名称。|
    |位置|必须匹配密钥保管库的位置。|
    |模板|单击“...”按钮，然后选择 `azuredeploy.json`|
    |模板参数|单击“...”按钮，然后选择 `azuredeploy.parameters.json`|

5.	保存生成定义。

### 添加“部署”步骤

1.	在“生成”选项卡上，选择“添加生成步骤...”命令。

2.	选择“实用工具”>“PowerShell”。

3.	选择生成步骤名称旁边的铅笔图标，然后将其重命名为“部署”。

4. 选择以下值：

    |设置名称|值|
    |---|---|
    |类型|**文件路径**|
    |脚本文件名|单击“...”按钮，并导航到应用程序项目内的 **Scripts** 目录。选择 `Deploy-FabricApplication.ps1`。|
    |参数|`-PublishProfileFile path/to/MySolution/MyApplicationProject/PublishProfiles/MyPublishProfile.xml -ApplicationPackagePath path/to/MySolution/MyApplicationProject/pkg/$(BuildConfiguration)`|

5.	保存生成定义。

### 试用

选择“为生成排队”以启动生成。在推送或签入时也将触发生成。

## 备选解决方案

前面的说明为每个生成创建新的群集，并在生成的末尾删除它。如果你希望改为让每个生成执行应用程序升级（到现有群集），请使用以下步骤：

1.	按照[这些说明](/documentation/articles/service-fabric-cluster-creation-via-arm/)，通过 Azure PowerShell 手动创建测试群集。

2.	按照[这些说明](/documentation/articles/service-fabric-visualstudio-configure-upgrade/)配置发布配置文件，以支持应用程序升级。

4.	从你的生成定义中删除“删除群集资源组”和“预配群集”生成步骤。

## 后续步骤

若要了解有关与 Service Fabric 应用程序持续集成的详细信息，请阅读以下文章：

- [生成文档主页](https://msdn.microsoft.com/zh-cn/Library/vs/alm/Build/overview)
- [部署生成代理](https://msdn.microsoft.com/zh-cn/Library/vs/alm/Build/agents/windows)
- [创建和配置生成定义](https://msdn.microsoft.com/zh-cn/Library/vs/alm/Build/vs/define-build)

<!---HONumber=Mooncake_0503_2016-->