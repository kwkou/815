<properties linkid="dev-net-common-tasks-continuous-delivery" urlDisplayName="Continuous Delivery" pageTitle="在 Azure 中使用 TFS 持续交付云服务" metaKeywords="Azure continuous delivery, continuous delivery sample code, continuous deliver PowerShell" description="了解如何设置 Azure 云应用程序的持续交付。MSBuild 命令行语句和 PowerShell 脚本的代码示例。" metaCanonical="" services="" documentationCenter="" title="在 Azure 中持续交付云服务" authors="ghogen" solutions="" manager="" editor="" />

# 在 Azure 中持续交付云服务

本文中所述过程向您演示如何设置对 Azure 云应用程序的持续
交付。此过程允许您
自动创建包，并在每个代码签入后将包部署到 Azure
。这篇
文章中描述的包生成过程等同于在 Visual Studio 中的程序包命令，
发布步骤等同于 Visual Studio 中的 Publish 命令。
该文章涵盖了将用于通过 MSBuild 命令行语句和 Windows PowerShell 脚本创建生成服务器
的方法，
它还演示了如何选择性地配置 Visual Studio Team
Foundation Server-Team Build 定义，以使用 MSBuild 命令
和 PowerShell 脚本。可针对您的生成环境和 Azure 目标环境自定义此过程。

此外，通过使用 Visual Studio Online（承载于 Azure 中的某个版本的 TFS），可以更轻松地做到这一点。有关更多信息，请参阅[使用 Visual Studio Online 向 Azure 持续交付][使用 Visual Studio Online 向 Azure 持续交付]。

在开始之前，应从 Visual Studio 发布您的应用程序。
这将确保所有资源可用，并且在您
尝试自动执行发布过程时初始化。

此任务包括下列步骤：

-   [步骤 1：配置生成服务器][步骤 1：配置生成服务器]
-   [步骤 2：使用 MSBuild 命令生成包][步骤 2：使用 MSBuild 命令生成包]
-   [步骤 3：使用 TFS Team Build 生成包（可选）][步骤 3：使用 TFS Team Build 生成包（可选）]
-   [步骤 4：使用 PowerShell 脚本发布包][步骤 4：使用 PowerShell 脚本发布包]
-   [步骤 5：使用 TFS Team Build 发布包（可选）][步骤 5：使用 TFS Team Build 发布包（可选）]

## <a name="step1"> </a><span class="short-header">配置生成服务器</span>步骤 1：配置生成服务器

您必须
先在生成服务器上安装必需的软件和工具，然后才能使用 MSBuild 创建 Azure 包。

无需在生成服务器上安装 Visual Studio。若
要使用 Team Foundation 生成服务管理生成
服务器，请按照 [Team Foundation 生成服务][Team Foundation 生成服务]
文档中的说明操作。

1.  在生成服务器上，安装包含 MSBuild 的 [.NET Framework 4][.NET Framework 4] 或 [.NET Framework 4.5][.NET Framework 4.5]。
2.  安装 [Azure 创作工具][Azure 创作工具]（根据您的生成服务器的处理器，查找
    WindowsAzureAuthoringTools-x86.msi 或 WindowsAzureAuthoringTools-x64.msi）。
3.  安装 [Windows Azure 库][Windows Azure 库]（查找 windowsazurelibsfornet-x86.msi 或 windowsazurelibsfornet-x64.msi)。
4.  
    将 Microsoft.WebApplication.targets 文件从 Visual Studio 安装复制到生成服务器中。在安装了 Visual Studio
    的计算机上，该文件位于目录 C:\\Program Files
    (x86)\\MSBuild\\Microsoft\\VisualStudio\\v11.0\\WebApplications（对于 Visual Studio 2013，
    为 v12.0）中。您
    应将该文件复制到生成服务器上的同一目录中。
5.  安装 [Azure Tools for Visual Studio][Windows Azure 库]。
    查找 WindowsAzureTools.VS120.exe 生成 Visual Studio 2012 项目，WindowsAzureTools.VS120.exe 生成 Visual Studio 2013 项目。

## <a name="step2"> </a><span class="short-header">使用 MSBuild 生成包</span>步骤 2：使用 MSBuild 命令生成包

本节介绍如何构造用于生成
 Azure 包的 MSBuild 命令。在生成服务器上执行此步骤可确认
所有内容配置正确并且 MSBuild 命令
起到预期作用。您可将此命令行添加到生成服务器上的现有
生成脚本中，也可在
TFS 生成定义中使用此命令行，如下节所述。有关命令行参数和 MSBuild 的更多
信息，请参阅 [MSBuild 命令行参考][MSBuild 命令行参考]。

1.  如果在生成服务器上安装了 Visual Studio，请单击
    **“开始”**、再单击**“所有程序”**，然后在
    **“Visual Studio 工具”**文件夹中找到并单击**“Visual Studio 命令提示符”**。

    如果未在生成服务器上安装 Visual Studio，则请打开
    命令提示符并确保可按相应的路径访问 MSBuild.exe。
    MSBuild 与 .NET Framework 一起安装在路径
    %WINDIR%\\Microsoft.NET\\Framework\\*\<版本\\\>* 中。
    例如，若要在已安装 .NET
    Framework 4 的情况下将 MSBuild.exe 添加到 PATH 环境变量，请在命令
    提示符处键入以下命令：

        set PATH=%PATH%;"C:\Windows\Microsoft.NET\Framework.0"

2.  在命令提示符处，导航到包含要生成的 Windows
    Azure 项目文件的文件夹。

3.  将 msbuild 与 /target:Publish 选项一起运行，如以下
    示例所示：

        MSBuild /target:Publish

    此选项可缩写为 /t:Publish。安装 Azure SDK
    后，MSBuild
    中的 /t:Publish 选项
    不应与 Visual Studio 中的可用 Publish 命令混淆。/t:Publish 选项仅生成 Azure
    包。其部署包的方式与
    Visual Studio 中的 Publish 命令部署包的方式不同。

    您也可以将项目名称指定为 MSBuild
    参数。如果未指定，则将使用当前目录。有关 MSBuild 命令行选项的更多
    信息，请参阅 [MSBuild 命令
    行参考][MSBuild 命令
    行参考]。

4.  查找输出。默认情况下，此命令将创建与项目的根文件夹
    相关的目录，例如
    *\<ProjectDir\>*\\bin\\*\<配置\>*\\app.publish\\。在
    构建 Azure 项目时，将生成两个文件，即包
    文件本身和附带的配置文件：

    -   Project.cspkg
    -   ServiceConfiguration.*\<TargetProfile\>*.cscfg

    默认情况下，每个 Azure 项目均包含两个
    服务配置文件（.cscfg 文件），这两个文件分别针对本地（调试）
    生成和云（过渡或生产）生成，您
    可根据需要添加或删除服务配置文件。在 Visual Studio 中
    生成包时，系统会询问您要将哪个
    服务配置文件与包一起包含。

5.  指定服务配置文件。
    使用 MSBuild 生成包时，
    默认情况下将包含本地服务配置文件。若要包含其他服务配置文件，请设置 MSBuild 命令的
    TargetProfile 属性，如以下
    示例所示：

        MSBuild /t:Publish /p:TargetProfile=Cloud

6.  指定输出的位置。使用 /p:PublishDir=
    *\<目录\\\>*\\ 选项设置路径，包含尾随
    反斜杠分隔符，如以下示例所示：

        MSBuild /target:Publish /p:PublishDir=\myserver\drops\

    构造并测试相应的 MSBuild 命令
    行以生成项目并将其并入一个 Azure 包后，
    您可将此命令行添加到生成脚本中。如果生成
    服务器使用自定义脚本，则此过程将依赖
    生成自定义过程的细节。如果您要将 TFS 用作
    生成环境，则可按照下一
    步中的说明操作来将 Azure 包生成添加到生成过程中。

## <a name="step3"> </a><span class="short-header">使用 TFS 生成包</span>步骤 3：使用 TFS Team Build 生成包（可选）

如果您已将 Team Foundation Server (TFS) 设置为生成控制器
并将生成服务器设置为 TFS 生成计算机，则可为 Azure 包设置
自动化生成。有关
如何设置 Team Foundation Server 并将其用作生成系统的信息，请参阅
[了解 Team Foundation Build 系统][了解 Team Foundation Build 系统]。具体而言，以下
过程假定您已按
[配置生成计算机][配置生成计算机]中所述配置了生成服务器。

若要将 TFS 配置为生成 Azure 包，请执行下列
步骤：

1.  
    在开发计算机上的 Visual Studio 2010 中的“视图”菜单上，选择**“团队资源管理器”**，或选择 Ctrl+\\、Ctrl+M。在
    “团队资源管理器”窗口中，展开**“生成”**节点，右键单击**“所有
    生成定义”**，然后单击**“新建生成定义”**：

    ![][0]

2.  单击**“进程”**选项卡。在“进程”选项卡上，选择默认
    模板，在“要生成的项目”下方选择项目，
    然后在网格中展开**“高级”**部分。

3.  选择**“MSBuild 参数”**，并按上面步骤 2 中所述设置相应的 MSBuild
    命令行参数。例如，
    输入 **/t:Publish /p:PublishDir=\\\\myserver\\drops\\** 以生成一个包并将包文件复制到位置
    \\\\myserver\\drops\\：

    ![][1]

    **注意：**通过将这些文件复制到公共共享，可以更轻松地
    手动从开发计算机部署包。

4.  单击**“触发器”**选项卡，
    然后为希望生成包的时间指定所需条件。例如，指定
    **“持续集成”**可在进行源
    代码管理签入时生成包。

5.  选择**“生成默认值”**选项卡，并在生成控制器下确认
    生成服务器的名称。此外，选择**“将生成
    输出复制到以下放置文件夹”**选项并指定所需的放置
    位置。

6.  通过签入对
    项目的更改来测试生成步骤是否成功或对新生成进行排队。若要对新生成进行排队，请在
    团队资源管理器中，右键单击**“所有生成定义”**，然后
    选择**“使新生成入队”**。

## <a name="step4"> </a><span class="short-header">使用 Powershell 进行发布</span>步骤 4：使用 Powershell 脚本发布包

本节介绍如何构造使用可选参数将云应用程序包输出发布到 Azure 的 Windows PowerShell 脚本。
在执行
自定义生成自动化中的生成步骤后，可以调用此脚本。也可以从 Visual Studio TFS Team Build 中的过程
模板工作流活动中调用此脚本。

1.  安装 [Azure PowerShell cmdlets][Azure PowerShell cmdlets]（v0.6.1 或更高版本）。
    在 cmdlet 设置阶段，选择作为管理单元安装。请注意
    ，此受支持的正式版本将替代通过 CodePlex 提供的旧版本，尽管早期版本已采用 2.x.x 的形式进行编号。

2.  使用“开始”菜单启动 Azure PowerShell。如果通过此方式启动，
    则将加载 Azure PowerShell cmdlet。

3.  在 PowerShell 提示符处，通过
    键入部分命令 Get-Azure，然后按 Tab 键完成
    语句，从而验证是否已加载 PowerShell cmdlet。

    您应看到各种 Azure PowerShell 命令。

4.  通过
    导入 .publishsettings 文件中的订阅信息来确认您能够连接到 Azure 订阅。

    Import-AzurePublishSettingsFile c:\\scripts\\WindowsAzure\\default.publishsettings

    然后，提供以下命令

    Get-AzureSubscription

    这将显示有关您的订阅的信息。确认所有内容正确。

5.  将[本文末尾][本文末尾]提供的脚本模板保存到
    脚本文件夹，路径为
    c:\\scripts\\WindowsAzure\\**PublishCloudService.ps1**。

6.  查看脚本的参数部分。添加或修改任何
    默认值。始终可通过传入
    显式参数来覆盖这些值。

7.  确保已在
    订阅中创建可通过发布脚本定位的有效云服务和存储帐户。
    存储帐户（Blob 存储）将用于在创建
    部署时上载和
    临时存储部署包和配置文件。

    -   若要创建新的云服务，您可调用此脚本或使用
        Azure 管理门户。云服务名称
        将用作完全限定域名中的前缀
        ，因此该名称必须是唯一的。

            New-AzureService -ServiceName "mytestcloudservice" -Location "North Central US" -Label "mytestcloudservice"

    -   若要创建新的存储帐户，您可调用此脚本或使用
        Azure 管理门户。存储帐户名称
        将用作完全限定域名中的前缀，
        因此该名称必须是唯一的。您可尝试使用与
        云服务相同的名称。

            New-AzureStorageAccount -ServiceName "mytestcloudservice" -Location "North Central US" -Label "mytestcloudservice"

8.  直接从 Azure PowerShell 调用脚本，或将此
    脚本连接到在包
    生成后进行的主机生成自动化。

    **警告：**默认情况下，此脚本将始终删除或替换现有
    部署（如果检测到这些部署）。这对于从没有用户提示
    的自动化中
    启用持续集成是必需的。

    **示例方案 1：**对服务的过渡
    环境进行持续部署：

        PowerShell c:\scripts\windowsazure\PublishCloudService.ps1 -environment Staging -serviceName mycloudservice -storageAccountName mystoragesaccount -packageLocation c:\drops\app.publish\ContactManager.Azure.cspkg -cloudConfigLocation c:\drops\app.publish\ServiceConfiguration.Cloud.cscfg -subscriptionDataFile c:\scripts\default.publishsettings

    通常，此操作后跟测试运行验证和 VIP
    交换。VIP 交换可通过 Azure 管理
    门户或使用 Move-Deployment cmdlet 执行。

    **示例方案 2：**对专用测试服务的生产
    环境进行持续部署

        PowerShell c:\scripts\windowsazure\PublishCloudService.ps1 -environment Production -enableDeploymentUpgrade 1 -serviceName mycloudservice -storageAccountName mystorageaccount -packageLocation c:\drops\app.publish\ContactManager.Azure.cspkg -cloudConfigLocation c:\drops\app.publish\ServiceConfiguration.Cloud.cscfg -subscriptionDataFile c:\scripts\default.publishsettings

    **远程桌面：**

    如果在 Azure 项目中启用远程桌面，则将
    需要执行额外的一次性步骤以确保将正确的
    云服务证书上载到通过此脚本定位的所有云服务中。

    查找角色所需的证书指纹值。这些
    指纹值在
    云配置文件（即 ServiceConfiguration.Cloud.cscfg）的“证书”部分中可见。此外，
    在显示选项并查看选定证书时，可在 Visual
    Studio 的“远程桌面配置”对话框中查看这些指纹值。

        <Certificates>
              <Certificate name="Microsoft.WindowsAzure.Plugins.RemoteAccess.PasswordEncryption" thumbprint="C33B6C432C25581601B84C80F86EC2809DC224E8" thumbprintAlgorithm="sha1" />
        </Certificates>

    使用以下 cmdlet 脚本将远程桌面证书
    作为一次性安装步骤上载：

        Add-AzureCertificate -serviceName <CLOUDSERVICENAME> -certToDeploy (get-item cert:\CurrentUser\MYAdd-AzureCertificate -serviceName <CLOUDSERVICENAME> -certToDeploy (get-item cert:\CurrentUser\MY\<THUMBPRINT>)lt;THUMBPRINT>)

    例如：

        Add-AzureCertificate -serviceName 'mytestcloudservice' -certToDeploy (get-item cert:\CurrentUser\MY\C33B6C432C25581601B84C80F86EC2809DC224E8

    或者，可以导出带私
    钥的证书文件 PFX，并使用
    Azure 管理门户将证书上载到每个目标云服务。阅读以下文章了解
    更多：
    [http://msdn.microsoft.com/zh-cn/library/windowsazure/gg443832.aspx] []。

    **升级部署与删除部署 -\> 新建部署**

    默认情况下，此脚本将在未传入参数或显式传递
    值 1 时执行升级部署
    ($enableDeploymentUpgrade = 1)。对于单一实例，此部署相对于完整部署的
    好处是，花费的时间更少。对于需要高可用性的实例，
    此部署的好处是，在升级一些实例的同时使其他实例
    保持运行（检查
    更新域）且不会删除您的 VIP。

    可使用脚本
    ($enableDeploymentUpgrade = 0) 或将 –enableDeploymentUpgrade 0 作为参数传递
    （这会将脚本行为更改为首先删除任何现有部署，然后创建新的部署）来禁用升级部署。

    **警告：**默认情况下，此脚本将始终删除或替换现有
    部署（如果检测到这些部署）。这对于从没有用户/操作员
    提示的自动化中
    启用持续集成是必需的。

## <a name="step5"> </a><span class="short-header">使用 TFS 进行发布</span>步骤 5：使用 TFS Team Build 发布包（可选）

此步骤会将 TFS Team Build 连接到步骤 4 中创建的脚本，
这将其处理将包生成发布到 Azure 的过程。这
就需要修改生成定义所使用的过程模板，使其
在工作流结束时运行 Publish 活动。Publish
活动将执行从
生成传入参数的 PowerShell 命令。MSBuild 目标和发布脚本的输出将
传送到标准生成输出中。

1.  编辑负责持续部署的生成定义。

2.  选择**“进程”**选项卡。

3.  通过单击该选项卡的
    “过程模板选择”区域中的“新建...”按钮来创建新的过程模板。这将加载“新建生成
    过程模板”对话框。在此对话框中，请确保选中“复制
    现有 XAML 文件”，并浏览到要
    复制的 ProcessTemplate（如果您需要从默认值更改基本值）。为新模板提供
    一个名称，例如 DefaultTemplateAzure。

4.  打开选定的过程模板以进行编辑。可以
    直接在工作流设计器或 XML 编辑器中打开以处理 XAML。

5.  在工作流设计器的
    参数选项卡中将以下一系列新参数作为单独的行项添加。所有参数应
    具有 direction=In 和 type=String。这两个值将用于将参数从生成定义流入
    工作流中，然后
    用于调用发布脚本。

        SubscriptionName
        StorageAccountName
        CloudConfigLocation
        PackageLocation
        Environment
        SubscriptionDataFileLocation
        PublishScriptLocation
        ServiceName

    ![][2]

    相应的 XAML 与下面类似：

        <Activity  _ />
          <x:Members>
            <x:Property Name="BuildSettings" Type="InArgument(mtbwa:BuildSettings)" />
            <x:Property Name="TestSpecs" Type="InArgument(mtbwa:TestSpecList)" />
            <x:Property Name="BuildNumberFormat" Type="InArgument(x:String)" />
            <x:Property Name="CleanWorkspace" Type="InArgument(mtbwa:CleanWorkspaceOption)" />
            <x:Property Name="RunCodeAnalysis" Type="InArgument(mtbwa:CodeAnalysisOption)" />
            <x:Property Name="SourceAndSymbolServerSettings" Type="InArgument(mtbwa:SourceAndSymbolServerSettings)" />
            <x:Property Name="AgentSettings" Type="InArgument(mtbwa:AgentSettings)" />
            <x:Property Name="AssociateChangesetsAndWorkItems" Type="InArgument(x:Boolean)" />
            <x:Property Name="CreateWorkItem" Type="InArgument(x:Boolean)" />
            <x:Property Name="DropBuild" Type="InArgument(x:Boolean)" />
            <x:Property Name="MSBuildArguments" Type="InArgument(x:String)" />
            <x:Property Name="MSBuildPlatform" Type="InArgument(mtbwa:ToolPlatform)" />
            <x:Property Name="PerformTestImpactAnalysis" Type="InArgument(x:Boolean)" />
            <x:Property Name="CreateLabel" Type="InArgument(x:Boolean)" />
            <x:Property Name="DisableTests" Type="InArgument(x:Boolean)" />
            <x:Property Name="GetVersion" Type="InArgument(x:String)" />
            <x:Property Name="PrivateDropLocation" Type="InArgument(x:String)" />
            <x:Property Name="Verbosity" Type="InArgument(mtbw:BuildVerbosity)" />
            <x:Property Name="Metadata" Type="mtbw:ProcessParameterMetadataCollection" />
            <x:Property Name="SupportedReasons" Type="mtbc:BuildReason" />
            <x:Property Name="SubscriptionName" Type="InArgument(x:String)" />
            <x:Property Name="StorageAccountName" Type="InArgument(x:String)" />
            <x:Property Name="CloudConfigLocation" Type="InArgument(x:String)" />
            <x:Property Name="PackageLocation" Type="InArgument(x:String)" />
            <x:Property Name="Environment" Type="InArgument(x:String)" />
            <x:Property Name="SubscriptionDataFileLocation" Type="InArgument(x:String)" />
            <x:Property Name="PublishScriptLocation" Type="InArgument(x:String)" />
            <x:Property Name="ServiceName" Type="InArgument(x:String)" />
          </x:Members>

          <this:Process.MSBuildArguments>

6.  在“在代理上运行”结束时添加一个新的序列：

    1.  首先，通过添加 If 语句活动来检查有效的
        脚本文件。将条件设置为此值：

            Not String.IsNullOrEmpty(PublishScriptLocation)

    2.  在 If 语句的 Then 事例中，添加一个新的 Sequence
        活动。将显示名称设置为 'Start publish'

    3.  在 Start publish 序列仍处于选定状态的情况下，在工作流设计器的变量选项卡中将
        以下一系列新
        变量作为单独的行项添加。所有变量应
        具有 Variable type =String 和 Scope=Start publish。这两个值将
        用于将参数从生成定义流入工作流中，然后用于调用发布脚本。

        -   String 类型的 SubscriptionDataFilePath

        -   String 类型的 PublishScriptFilePath

            ![][3]

    4.  在新的
        Sequence 的开头添加一个 ConvertWorkspaceItem 活动。Direction=ServerToLocal, DisplayName='Convert publish
        script filename', Input=' PublishScriptLocation',
        Result='PublishScriptFilePath', Workspace='Workspace'.此
        活动将发布脚本的路径从 TFS 服务器
        位置（如果适用）转换为标准本地磁盘路径。

    5.  在新的
         Sequence 的末尾添加另一个 ConvertWorkspaceItem 活动。Direction=ServerToLocal, DisplayName='Convert
        subscription filename', Input=' SubscriptionDataFileLocation',
        Result= 'SubscriptionDataFilePath', Workspace='Workspace'.

    6.  在新 Sequence 的末尾添加一个 InvokeProcess 活动。
        此活动使用
        生成定义传入的参数调用 PowerShell.exe。

        1.  Arguments = String.Format(" -File ""{0}"" -serviceName {1}
            -storageAccountName {2} -packageLocation ""{3}""
            -cloudConfigLocation ""{4}"" -subscriptionDataFile ""{5}""
            -selectedSubscription {6} -environment ""{7}""",
            PublishScriptFilePath, ServiceName, StorageAccountName,
            PackageLocation, CloudConfigLocation,
            SubscriptionDataFilePath, SubscriptionName, Environment)

        2.  DisplayName ='Execute publish script'

        3.  FileName = "PowerShell"

        4.  OutputEncoding=
            'System.Text.Encoding.GetEncoding(System.Globalization.CultureInfo.InstalledUICulture.TextInfo.OEMCodePage)'

    7.  在
        InvokeProcess 的“处理标准输出”部分文本框中，将文本框值设置为“data”。这是一个用于存储标准输出数据的
        变量。

    8.  在标准输出
        部分的正下方添加一个 WriteBuildMessage 活动。设置 Importance =
        'Microsoft.TeamFoundation.Build.Client.BuildMessageImportance.High'
        且 Message='data'。这可确保
        脚本的标准输出将写入到生成输出中。

    9.  在
        InvokeProcess 的“处理标准错误”部分文本框中，将文本框值设置为“data”。这是一个用于存储标准错误数据的
        变量。

    10. 在标准输出
        部分的正下方添加一个 WriteBuildError 活动。设置 Message='data'。这可确保脚本的标准
        错误将写入到生成错误输出中。

    发布工作流活动的最终结果将
    与设计器中的以下内容类似：

    ![][4]

    发布工作流活动的最终结果将与 XAML 中的
    以下内容类似：

            </TryCatch>
              <If Condition="[Not String.IsNullOrEmpty(PublishScriptLocation)]" sap:VirtualizedContainerService.HintSize="1539,552">
                <If.Then>
                  <Sequence DisplayName="Start publish" sap:VirtualizedContainerService.HintSize="297,446">
                    <Sequence.Variables>
                      <Variable x:TypeArguments="x:String" Name="PublishScriptFilePath" />
                      <Variable x:TypeArguments="x:String" Name="SubscriptionDataFilePath" />
                    </Sequence.Variables>
                    <sap:WorkflowViewStateService.ViewState>
                      <scg:Dictionary x:TypeArguments="x:String, x:Object">
                        <x:Boolean x:Key="IsExpanded">True</x:Boolean>
                      </scg:Dictionary>
                    </sap:WorkflowViewStateService.ViewState>
                    <mtbwa:ConvertWorkspaceItem DisplayName="Convert publish script filename" sap:VirtualizedContainerService.HintSize="234,22" Input="[PublishScriptLocation]" Result="[PublishScriptFilePath]" Workspace="[Workspace]" />
                    <mtbwa:ConvertWorkspaceItem DisplayName="Convert subscription data file filename" sap:VirtualizedContainerService.HintSize="234,22" Input="[SubscriptionDataFileLocation]" Result="[SubscriptionDataFilePath]" Workspace="[Workspace]" />
                    <mtbwa:InvokeProcess Arguments="[String.Format(" -File ""{0}"" -serviceName {1} -storageAccountName {2} -packageLocation ""{3}"" -cloudConfigLocation ""{4}"" -subscriptionDataFile ""{5}"" -selectedSubscription {6} -environment ""{7}""", PublishScriptFilePath, ServiceName, StorageAccountName, PackageLocation, CloudConfigLocation, SubscriptionDataFilePath, SubscriptionName, Environment)]" DisplayName="Execute publish script" FileName="PowerShell" sap:VirtualizedContainerService.HintSize="234,198">
                      <mtbwa:InvokeProcess.ErrorDataReceived>
                        <ActivityAction x:TypeArguments="x:String">
                          <ActivityAction.Argument>
                            <DelegateInArgument x:TypeArguments="x:String" Name="data" />
                          </ActivityAction.Argument>
                          <mtbwa:WriteBuildError sap:VirtualizedContainerService.HintSize="200,22" Message="[data]" />
                        </ActivityAction>
                      </mtbwa:InvokeProcess.ErrorDataReceived>
                      <mtbwa:InvokeProcess.OutputDataReceived>
                        <ActivityAction x:TypeArguments="x:String">
                          <ActivityAction.Argument>
                            <DelegateInArgument x:TypeArguments="x:String" Name="data" />
                          </ActivityAction.Argument>
                          <mtbwa:WriteBuildMessage sap:VirtualizedContainerService.HintSize="200,22" Importance="[Microsoft.TeamFoundation.Build.Client.BuildMessageImportance.High]" Message="[data]" mva:VisualBasic.Settings="Assembly references and imported namespaces serialized as XML namespaces" />
                        </ActivityAction>
                      </mtbwa:InvokeProcess.OutputDataReceived>
                    </mtbwa:InvokeProcess>
                  </Sequence>
                </If.Then>
              </If>
            </mtbwa:AgentScope>
            <mtbwa:InvokeForReason DisplayName="Check In Gated Changes for CheckInShelveset Builds" sap:VirtualizedContainerService.HintSize="1370,146" Reason="CheckInShelveset">
              <mtbwa:CheckInGatedChanges DisplayName="Check In Gated Changes" sap:VirtualizedContainerService.HintSize="200,22" />
            </mtbwa:InvokeForReason>
          </Sequence>
        </Activity>

7.  保存 DefaultTemplateAzure 工作流并签入此文件。

8.  在生成
    定义中选择 DefaultTemplateAzure 过程模板。如果您尚未在过程模板列表中
    看到此文件，请单击“刷新”或选择“新建”按钮。

9.  在“杂项”部分中设置参数属性，如下所示：

    1.  CloudConfigLocation =
        'c:\\drops\\app.publish\\ServiceConfiguration.Cloud.cscfg'
        *此值派生自：
        ($PublishDir)ServiceConfiguration.Cloud.cscfg*

    2.  PackageLocation =
        'c:\\drops\\app.publish\\ContactManager.Azure.cspkg'
        *此值派生自：($PublishDir)($ProjectName).cspkg*

    3.  PublishScriptLocation =
        'c:\\scripts\\WindowsAzure\\PublishCloudService.ps1'

    4.  ServiceName = 'mycloudservicename'
        *在此处使用适当的云服务名称*

    5.  Environment = 'Staging'

    6.  StorageAccountName = 'mystorageaccountname'
        *在此处使用适当的存储帐户名称*

    7.  SubscriptionDataFileLocation =
        'c:\\scripts\\WindowsAzure\\Subscription.xml'

    8.  SubscriptionName = 'default'

    ![][5]

10. 保存对生成定义所做的更改。

11. 对生成进行排队以便同时执行包生成和发布。如果您的
    触发器设置为“持续部署”，则将在每次签入时执行此
    行为。

### <a name="script"> </a>PublishCloudService.ps1 脚本模板

    Param(  $serviceName = "",
    $storageAccountName = "",
    $packageLocation = "",
    $cloudConfigLocation = "",
    $environment = "Staging",
    $deploymentLabel = "ContinuousDeploy to $servicename",
    $timeStampFormat = "g",
    $alwaysDeleteExistingDeployments = 1,
    $enableDeploymentUpgrade = 1,
    $selectedsubscription = "default",
    $subscriptionDataFile = ""
         )
          

    function Publish()
    {
    $deployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot -ErrorVariable a -ErrorAction silentlycontinue 
    if ($a[0] -ne $null)
        {
    Write-Output "$(Get-Date -f $timeStampFormat) - No deployment is detected.Creating a new deployment. "
        }
    #check for existing deployment and then either upgrade, delete + deploy, or cancel according to $alwaysDeleteExistingDeployments and $enableDeploymentUpgrade boolean variables
    if ($deployment.Name -ne $null)
        {
    switch ($alwaysDeleteExistingDeployments)
            {
                1 
                {
    switch ($enableDeploymentUpgrade)
                    {
    1  #Update deployment inplace (usually faster, cheaper, won't destroy VIP)
                        {
    Write-Output "$(Get-Date -f $timeStampFormat) - Deployment exists in $servicename.Upgrading deployment."
    UpgradeDeployment
                        }
    0  #Delete then create new deployment
                        {
    Write-Output "$(Get-Date -f $timeStampFormat) - Deployment exists in $servicename.Deleting deployment."
    DeleteDeployment
    CreateNewDeployment
                            
                        }
    } # switch ($enableDeploymentUpgrade)
                }
                0
                {
    Write-Output "$(Get-Date -f $timeStampFormat) - ERROR:Deployment exists in $servicename.Script execution cancelled."
    exit
                }
    } #switch ($alwaysDeleteExistingDeployments)
    } else {
    CreateNewDeployment
        }
    }

    function CreateNewDeployment()
    {
    write-progress -id 3 -activity "Creating New Deployment" -Status "In progress"
    Write-Output "$(Get-Date -f $timeStampFormat) - Creating New Deployment:In progress"

    $opstat = New-AzureDeployment -Slot $slot -Package $packageLocation -Configuration $cloudConfigLocation -label $deploymentLabel -ServiceName $serviceName
            
    $completeDeployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
    $completeDeploymentID = $completeDeployment.deploymentid

    write-progress -id 3 -activity "Creating New Deployment" -completed -Status "Complete"
    Write-Output "$(Get-Date -f $timeStampFormat) - Creating New Deployment:Complete, Deployment ID:$completeDeploymentID"
        
    StartInstances
    }

    function UpgradeDeployment()
    {
    write-progress -id 3 -activity "Upgrading Deployment" -Status "In progress"
    Write-Output "$(Get-Date -f $timeStampFormat) - Upgrading Deployment:In progress"

    # perform Update-Deployment
    $setdeployment = Set-AzureDeployment -Upgrade -Slot $slot -Package $packageLocation -Configuration $cloudConfigLocation -label $deploymentLabel -ServiceName $serviceName -Force
        
    $completeDeployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
    $completeDeploymentID = $completeDeployment.deploymentid
        
    write-progress -id 3 -activity "Upgrading Deployment" -completed -Status "Complete"
    Write-Output "$(Get-Date -f $timeStampFormat) - Upgrading Deployment:Complete, Deployment ID:$completeDeploymentID"
    }

    function DeleteDeployment()
    {

    write-progress -id 2 -activity "Deleting Deployment" -Status "In progress"
    Write-Output "$(Get-Date -f $timeStampFormat) - Deleting Deployment:In progress"

    #WARNING - always deletes with force
    $removeDeployment = Remove-AzureDeployment -Slot $slot -ServiceName $serviceName -Force

    write-progress -id 2 -activity "Deleting Deployment:Complete" -completed -Status $removeDeployment
    Write-Output "$(Get-Date -f $timeStampFormat) - Deleting Deployment:Complete"
        
    }

    function StartInstances()
    {
    write-progress -id 4 -activity "Starting Instances" -status "In progress"
    Write-Output "$(Get-Date -f $timeStampFormat) - Starting Instances:In progress"

    $deployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
    $runstatus = $deployment.Status

    if ($runstatus -ne 'Running') 
        {
    $run = Set-AzureDeployment -Slot $slot -ServiceName $serviceName -Status Running
        }
    $deployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
    $oldStatusStr = @("") * $deployment.RoleInstanceList.Count
        
    while (-not(AllInstancesRunning($deployment.RoleInstanceList)))
        {
    $i = 1
    foreach ($roleInstance in $deployment.RoleInstanceList)
            {
    $instanceName = $roleInstance.InstanceName
    $instanceStatus = $roleInstance.InstanceStatus

    if ($oldStatusStr[$i - 1] -ne $roleInstance.InstanceStatus)
                {
    $oldStatusStr[$i - 1] = $roleInstance.InstanceStatus
    Write-Output "$(Get-Date -f $timeStampFormat) - Starting Instance '$instanceName':$instanceStatus"
                }

    write-progress -id (4 + $i) -activity "Starting Instance '$instanceName'" -status "$instanceStatus"
    $i = $i + 1
            }

    sleep -Seconds 1

    $deployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
        }

    $i = 1
    foreach ($roleInstance in $deployment.RoleInstanceList)
        {
    $instanceName = $roleInstance.InstanceName
    $instanceStatus = $roleInstance.InstanceStatus

    if ($oldStatusStr[$i - 1] -ne $roleInstance.InstanceStatus)
            {
    $oldStatusStr[$i - 1] = $roleInstance.InstanceStatus
    Write-Output "$(Get-Date -f $timeStampFormat) - Starting Instance '$instanceName':$instanceStatus"
            }

    $i = $i + 1
        }
        
    $deployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
    $opstat = $deployment.Status 
        
    write-progress -id 4 -activity "Starting Instances" -completed -status $opstat
    Write-Output "$(Get-Date -f $timeStampFormat) - Starting Instances:$opstat"
    }

    function AllInstancesRunning($roleInstanceList)
    {
    foreach ($roleInstance in $roleInstanceList)
        {
    if ($roleInstance.InstanceStatus -ne "ReadyRole")
            {
    return $false
            }
        }
        
    return $true
    }

    #configure powershell with Azure 1.7 modules
    Import-Module Azure

    #configure powershell with publishsettings for your subscription
    $pubsettings = $subscriptionDataFile
    Import-AzurePublishSettingsFile $pubsettings
    Set-AzureSubscription -CurrentStorageAccount $storageAccountName -SubscriptionName $selectedsubscription

    #set remaining environment variables for Azure cmdlets
    $subscription = Get-AzureSubscription $selectedsubscription
    $subscriptionname = $subscription.subscriptionname
    $subscriptionid = $subscription.subscriptionid
    $slot = $environment

    #main driver - publish & write progress to activity log
    Write-Output "$(Get-Date -f $timeStampFormat) - Azure Cloud Service deploy script started."
    Write-Output "$(Get-Date -f $timeStampFormat) - Preparing deployment of $deploymentLabel for $subscriptionname with Subscription ID $subscriptionid."

    发布

    $deployment = Get-AzureDeployment -slot $slot -serviceName $servicename
    $deploymentUrl = $deployment.Url

    Write-Output "$(Get-Date -f $timeStampFormat) - Created Cloud Service with URL $deploymentUrl."
    Write-Output "$(Get-Date -f $timeStampFormat) - Azure Cloud Service deploy script finished."

  [使用 Visual Studio Online 向 Azure 持续交付]: ../cloud-services-continuous-delivery-use-vso/
  [步骤 1：配置生成服务器]: #step1
  [步骤 2：使用 MSBuild 命令生成包]: #step2
  [步骤 3：使用 TFS Team Build 生成包（可选）]: #step3
  [步骤 4：使用 PowerShell 脚本发布包]: #step4
  [步骤 5：使用 TFS Team Build 发布包（可选）]: #step5
  [Team Foundation 生成服务]: http://go.microsoft.com/fwlink/p/?LinkId=239963
  [.NET Framework 4]: http://go.microsoft.com/fwlink/?LinkId=239538
  [.NET Framework 4.5]: http://go.microsoft.com/fwlink/?LinkId=245484
  [Azure 创作工具]: http://go.microsoft.com/fwlink/?LinkId=239600
  [Windows Azure 库]: http://go.microsoft.com/fwlink/?LinkId=257862
  [MSBuild 命令行参考]: http://msdn.microsoft.com/zh-cn/library/ms164311(v=VS.90).aspx
  [了解 Team Foundation Build 系统]: http://go.microsoft.com/fwlink/?LinkId=238798
  [配置生成计算机]: http://go.microsoft.com/fwlink/?LinkId=238799
  [0]: ./media/cloud-services-dotnet-continuous-delivery/tfs-01.png
  [1]: ./media/cloud-services-dotnet-continuous-delivery/tfs-02.png
  [Azure PowerShell cmdlets]: http://go.microsoft.com/fwlink/?LinkId=256262
  [本文末尾]: #script
  [2]: ./media/cloud-services-dotnet-continuous-delivery/common-task-tfs-03.png
  [3]: ./media/cloud-services-dotnet-continuous-delivery/common-task-tfs-04.png
  [4]: ./media/cloud-services-dotnet-continuous-delivery/common-task-tfs-05.png
  [5]: ./media/cloud-services-dotnet-continuous-delivery/common-task-tfs-06.png
