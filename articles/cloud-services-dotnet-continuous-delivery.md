<properties
	pageTitle="在 Azure 中使用 TFS 持续交付云服务 | Azure"
	description="了解如何设置 Azure 云应用程序的持续交付。MSBuild 命令行语句和 PowerShell 脚本的代码示例。"
	services="cloud-services"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="cloud-services"
	ms.date="05/08/2016"
	wacn.date="05/31/2016"/>

# 在 Azure 中持续交付云服务

本文中所述过程向你演示如何设置对 Azure 云应用程序的持续交付。此过程使你能够在签入每个代码后，自动创建服务包并将其部署到 Azure。本文中介绍的包生成过程与 Visual Studio 中的 **Package** 命令等效，而发布步骤与 Visual Studio 中的 **Publish** 命令等效。本文包含用于创建生成服务器的方法以及 MSBuild 命令行语句和 Windows PowerShell 脚本，并演示了如何选择性地配置 Visual Studio Team Foundation Server - Team Build 定义以使用 MSBuild 命令和 PowerShell 脚本。可针对你的生成环境和 Azure 目标环境自定义此过程。

你也可以使用 Visual Studio Team Services（Azure 中托管的 TFS 版本）更轻松地实现此目的。有关详细信息，请参阅 [使用 Visual Studio Team Services 向 Azure 持续交付][]。

开始之前，您应从 Visual Studio 中发布应用程序。这将确保所有资源在您尝试实现发布过程的自动化时可用并进行初始化。

## 1：配置生成服务器

你必须先在生成服务器上安装必需的软件和工具，然后才能使用 MSBuild 创建 Azure 包。

无需在生成服务器上安装 Visual Studio。若要使用 Team Foundation 生成服务管理生成服务器，请按照 [Team Foundation 生成服务][]文档中的说明操作。

1.  在生成服务器上，安装包含 MSBuild 的 [.NET Framework 4.5.2][]。
2.  安装最新的 [Azure Authoring Tools for .NET](/develop/net)。
3.	安装 [Azure Libraries for .NET](http://go.microsoft.com/fwlink/?LinkId=623519)。
4.  将 Microsoft.WebApplication.targets 文件从 Visual Studio 安装复制到生成服务器。

	在已安装 Visual Studio 的计算机上，此文件位于目录 C:\\Program Files(x86)\\MSBuild\\Microsoft\\VisualStudio\\v14.0\\WebApplications。您应将该文件复制到生成服务器上的同一目录中。
5.  安装 [Azure Tools for Visual Studio](https://www.visualstudio.com/features/azure-tools-vs.aspx)。

## 2：使用 MSBuild 命令生成包

本部分介绍如何构造用于生成 Azure 包的 MSBuild 命令。在生成服务器上执行此步骤可确认所有内容配置正确并且 MSBuild 命令起到预期作用。你可将此命令行添加到生成服务器上的现有生成脚本中，也可在 TFS 生成定义中使用此命令行，如下一部分所述。有关命令行参数和 MSBuild 的详细信息，请参阅 [MSBuild 命令行参考](https://msdn.microsoft.com/zh-cn/library/ms164311%28v=vs.140%29.aspx)。

1.  如果在生成服务器上安装了 Visual Studio，请在 Windows 上的“Visual Studio 工具”文件夹中找到并选择“Visual Studio 命令提示符”。

    如果未在生成服务器上安装 Visual Studio，则请打开命令提示符并确保可按相应的路径访问 MSBuild.exe。MSBuild 与 .NET Framework 一起安装在路径 %WINDIR%\\Microsoft.NET\\Framework\*版本* 中。例如，若要在已安装 .NET Framework 4 的情况下将 MSBuild.exe 添加到 PATH 环境变量，请在命令提示符处键入以下命令：

        set PATH=%PATH%;"C:\Windows\Microsoft.NET\Framework\v4.0.30319"

2.  在命令提示符处，导航到包含要生成的 Azure 项目文件的文件夹。

3.  将 MSBuild 与 /target:Publish 选项一起运行，如以下示例所示：

        MSBuild /target:Publish

    此选项可缩写为 /t:Publish。安装 Azure SDK 后，MSBuild 中的 /t:Publish 选项不应与 Visual Studio 中的可用 Publish 命令混淆。/t:Publish 选项仅生成 Azure 包。其部署包的方式与 Visual Studio 中的 Publish 命令部署包的方式不同。

    您也可以将项目名称指定为 MSBuild 参数。如果未指定，则将使用当前目录。有关 MSBuild 命令行选项的详细信息，请参阅 [MSBuild 命令行参考](1)。

4.  查找输出。默认情况下，此命令将创建与项目的根文件夹相关的目录，例如 *ProjectDir*\\bin\\*Configuration*\\app.publish\\。在生成 Azure 项目时，将生成两个文件，即包文件本身和附带的配置文件：

    -   Project.cspkg
    -   ServiceConfiguration.*TargetProfile*.cscfg

    默认情况下，每个 Azure 项目均包含两个服务配置文件（.cscfg 文件），这两个文件分别针对本地（调试）生成和云（过渡或生产）生成，你可根据需要添加或删除服务配置文件。在 Visual Studio 中生成包时，系统会询问您要将哪个服务配置文件与包一起包含。

5.  指定服务配置文件。使用 MSBuild 生成包时，默认情况下将包含本地服务配置文件。若要包含其他服务配置文件，请设置 MSBuild 命令的 TargetProfile 属性，如以下示例所示：

        MSBuild /t:Publish /p:TargetProfile=Cloud

6.  指定输出的位置。使用 /p:PublishDir=*Directory*\\ 选项设置路径，包含尾随反斜杠分隔符，如以下示例所示：

        MSBuild /target:Publish /p:PublishDir=\\myserver\drops\

    构造并测试相应的 MSBuild 命令行以生成项目并将其并入一个 Azure 包后，你可将此命令行添加到生成脚本中。如果生成服务器使用自定义脚本，则此过程将依赖自定义生成过程的细节。如果你要将 TFS 用作生成环境，则可按照下一步中的说明操作来将 Azure 包生成添加到生成过程中。

## 3：使用 TFS Team Build 生成包

如果你已将 Team Foundation Server (TFS) 设置为生成控制器并将生成服务器设置为 TFS 生成计算机，则可以选择为 Azure 包设置自动化生成。有关如何设置 Team Foundation Server 并将其用作生成系统的信息，请参阅[扩大生成系统][]。具体而言，以下过程假设你已根据[部署和配置生成服务器][]中所述配置了生成服务器，此外，你已创建了一个团队项目并在该团队项目中创建了一个云服务项目。

若要将 TFS 配置为生成 Azure 包，请执行下列步骤：

1.  在开发计算机上的 Visual Studio 中，从“视图”菜单中选择“团队资源管理器”，或选择 Ctrl+\\、Ctrl+M。在“团队资源管理器”窗口中，展开“生成”节点，或者选择“生成” 页，然后选择“新建生成定义”。

    ![][0]

2.  选择“触发器”选项卡，然后为希望生成包的时间指定所需条件。例如，指定“持续集成”可在进行源代码管理签入时生成包。

3.	选择“源设置”选项卡，并确保你的项目文件夹已列在“源代码管理文件夹”列中，并且状态为“活动”。

4.  选择“生成默认值”选项卡，并在生成控制器下确认生成服务器的名称。此外，选择“将生成输出复制到以下放置文件夹”选项并指定所需的放置位置。

5.  选择“进程”选项卡。在“进程”选项卡上选择默认模板，在“生成”下选择项目（如果尚未选择），然后展开网格“生成”部分中的“高级”部分。

6.  选择“MSBuild 参数”，并按上面步骤 2 中所述设置相应的 MSBuild 命令行参数。例如，输入 **/t:Publish /p:PublishDir=\\\myserver\\drops\** 可以生成一个包并将包文件复制到位置 \\\myserver\\drops\\：

    ![][2]

    **注意：**通过将这些文件复制到公共共享，可以更轻松地手动从开发计算机部署包。

5.  通过签入对项目的更改来测试生成步骤是否成功或对新生成进行排队。若要对新生成进行排队，请在团队资源管理器中，右键单击“所有生成定义”，然后选择“使新生成入队”。

## 4：使用 PowerShell 脚本发布包

本节介绍如何构造使用可选参数将云应用程序包输出发布到 Azure 的 Windows PowerShell 脚本。在执行自定义生成自动化中的生成步骤后，可以调用此脚本。也可以从 Visual Studio TFS Team Build 中的过程模板工作流活动中调用此脚本。

1.  安装 [Azure PowerShell cmdlet][]（v0.6.1 或更高版本）。在 cmdlet 设置阶段，选择作为管理单元安装。请注意，此受支持的正式版本将替代通过 CodePlex 提供的旧版本，尽管早期版本已采用 2.x.x 的形式进行编号。

2.  使用“开始”菜单或“开始”页启动 Azure PowerShell。如果通过此方式启动，则将加载 Azure PowerShell cmdlet。

3.  在 PowerShell 提示符下，通过输入部分命令 `Get-Azure`，然后按 Tab 键完成语句，从而验证是否已加载 PowerShell cmdlet。

    重复按 Tab 键应会看到各个 Azure PowerShell 命令。

4.  通过导入 .publishsettings 文件中的订阅信息来确认你能够连接到 Azure 订阅。

    `Import-AzurePublishSettingsFile -Environment AzureChinaCloud c:\scripts\WindowsAzure\default.publishsettings`

    然后输入该命令

    `Get-AzureSubscription`

    这会显示有关你的订阅的信息。确认所有内容正确。

4.  将本文末尾提供的脚本模板保存到脚本文件夹，路径为 c:\\scripts\\WindowsAzure\**PublishCloudService.ps1**。

5.  查看脚本的参数部分。添加或修改任何默认值。始终可通过传入显式参数来覆盖这些值。

6.  确保已在订阅中创建可通过发布脚本定位的有效云服务和存储帐户。存储帐户（Blob 存储）将用于在创建部署时上载和临时存储部署包和配置文件。

    -   若要创建新的云服务，你可调用此脚本或使用 [Azure 管理门户](https://manage.windowsazure.cn)。云服务名称将用作完全限定域名中的前缀，因此该名称必须是唯一的。

            New-AzureService -ServiceName "mytestcloudservice" -Location "China North" -Label "mytestcloudservice"

    -   若要创建新的存储帐户，你可调用此脚本或使用 [Azure 管理门户](https://manage.windowsazure.cn)。存储帐户名称将用作完全限定域名中的前缀，因此该名称必须是唯一的。您可尝试使用与云服务相同的名称。

            New-AzureStorageAccount -ServiceName "mytestcloudservice" -Location "China North" -Label "mytestcloudservice"

7.  直接从 Azure PowerShell 调用脚本，或将此脚本连接到在包生成后进行的主机生成自动化。

    >[AZURE.IMPORTANT] 默认情况下，此脚本将始终删除或替换现有部署（如果检测到这些部署）。这对于从没有用户提示的自动化中启用持续集成是必需的。

    **示例方案 1：**对服务的过渡环境进行持续部署：

        PowerShell c:\scripts\windowsazure\PublishCloudService.ps1 -environment Staging -serviceName mycloudservice -storageAccountName mystoragesaccount -packageLocation c:\drops\app.publish\ContactManager.Azure.cspkg -cloudConfigLocation c:\drops\app.publish\ServiceConfiguration.Cloud.cscfg -subscriptionDataFile c:\scripts\default.publishsettings

    通常，此操作后跟测试运行验证和 VIP 交换。VIP 交换可通过 [Azure 管理门户](https://manage.windowsazure.cn)或使用 Move-Deployment cmdlet 执行。

    **示例方案 2：**对专用测试服务的生产环境进行持续部署

        PowerShell c:\scripts\windowsazure\PublishCloudService.ps1 -environment Production -enableDeploymentUpgrade 1 -serviceName mycloudservice -storageAccountName mystorageaccount -packageLocation c:\drops\app.publish\ContactManager.Azure.cspkg -cloudConfigLocation c:\drops\app.publish\ServiceConfiguration.Cloud.cscfg -subscriptionDataFile c:\scripts\default.publishsettings

    **远程桌面：**

    如果在 Azure 项目中启用远程桌面，则将需要执行额外的一次性步骤以确保将正确的云服务证书上载到通过此脚本定位的所有云服务中。

    查找角色所需的证书指纹值。这些指纹值在云配置文件（即 ServiceConfiguration.Cloud.cscfg）的“证书”部分中可见。此外，在显示选项并查看选定证书时，可在 Visual Studio 的“远程桌面配置”对话框中查看这些指纹值。

        <Certificates>
              <Certificate name="Microsoft.WindowsAzure.Plugins.RemoteAccess.PasswordEncryption" thumbprint="C33B6C432C25581601B84C80F86EC2809DC224E8" thumbprintAlgorithm="sha1" />
        </Certificates>

    使用以下 cmdlet 脚本将远程桌面证书作为一次性安装步骤上载：

        Add-AzureCertificate -serviceName <CLOUDSERVICENAME> -certToDeploy (get-item cert:\CurrentUser\MY<THUMBPRINT>)

    例如：

        Add-AzureCertificate -serviceName 'mytestcloudservice' -certToDeploy (get-item cert:\CurrentUser\MY\C33B6C432C25581601B84C80F86EC2809DC224E8

    或者，可以导出带私钥的证书文件 PFX，并使用 [Azure 管理门户](https://manage.windowsazure.cn)将证书上载到每个目标云服务。阅读以下文章以了解详细信息：[http://msdn.microsoft.com/zh-cn/library/windowsazure/gg443832.aspx][]。

    **升级部署与删除部署 -> 新建部署**

    默认情况下，此脚本将在未传入参数或显式传递值 1 时执行升级部署 ($enableDeploymentUpgrade = 1)。对于单一实例，此部署相对于完整部署的好处是，花费的时间更少。对于需要高可用性的实例，此部署的好处是，在升级一些实例的同时使其他实例保持运行（检查更新域）且不会删除您的 VIP。

    可使用脚本 ($enableDeploymentUpgrade = 0) 或将 *-enableDeploymentUpgrade 0* 作为参数传递（这会将脚本行为更改为首先删除任何现有部署，然后创建新的部署）来禁用升级部署。

    >[AZURE.IMPORTANT] 默认情况下，此脚本将始终删除或替换现有部署（如果检测到这些部署）。这对于从没有用户/操作员提示的自动化中启用持续集成是必需的。

## 5：使用 TFS Team Build 发布包

此可选步骤会将 TFS Team Build 连接到步骤 4 中创建的脚本，这将其处理将包生成发布到 Azure 的过程。这就需要修改生成定义所使用的过程模板，使其在工作流结束时运行 Publish 活动。Publish 活动将执行从生成传入参数的 PowerShell 命令。MSBuild 目标和发布脚本的输出将传送到标准生成输出中。

1.  编辑负责持续部署的生成定义。

2.  选择“进程”选项卡。

3.	按照[这些说明](http://msdn.microsoft.com/zh-cn/library/dd647551.aspx)添加生成过程模板的活动项目，下载默认模板，将其添加到项目并将其签入。为生成过程模板指定新名称，如 AzureBuildProcessTemplate。

3.  返回到“进程”选项卡，然后使用“显示详细信息”显示可用生成过程模板的列表。选择“新建...”按钮，然后导航到你刚刚添加并签入的项目。找到刚刚创建的模板，然后选择“确定”。

4.  打开选定的过程模板以进行编辑。可以直接在工作流设计器或 XML 编辑器中打开以处理 XAML。

5.  在工作流设计器的参数选项卡中将以下一系列新参数作为单独的行项添加。所有参数应具有 direction=In 和 type=String。这两个值将用于将参数从生成定义流入工作流中，然后用于调用发布脚本。

        SubscriptionName
        StorageAccountName
        CloudConfigLocation
        PackageLocation
        Environment
        SubscriptionDataFileLocation
        PublishScriptLocation
        ServiceName

    ![][3]

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

    1.  首先，通过添加 If 语句活动来检查有效的脚本文件。将条件设置为此值：

            Not String.IsNullOrEmpty(PublishScriptLocation)

    2.  在 If 语句的 Then 事例中，添加一个新的 Sequence 活动。将显示名称设置为 'Start publish'

    3.  在 Start publish 序列仍处于选定状态的情况下，在工作流设计器的变量选项卡中将以下一系列新变量作为单独的行项添加。所有变量应具有 Variable type =String 和 Scope=Start publish。这两个值将用于将参数从生成定义流入工作流中，然后用于调用发布脚本。

        -   String 类型的 SubscriptionDataFilePath

        -   String 类型的 PublishScriptFilePath

            ![][4]

    4.  如果你使用的是 TFS 2012 或更低版本，请在新序列的开头添加一个 ConvertWorkspaceItem 活动。如果你使用的是 TFS 2013 或更高版本，请在新序列的开头添加一个 GetLocalPath 活动。对于 ConvertWorkspaceItem，请按如下所示设置属性：Direction=ServerToLocal, DisplayName='Convert publish script filename', Input=' PublishScriptLocation', Result='PublishScriptFilePath', Workspace='Workspace'。对于 GetLocalPath 活动，请将属性 IncomingPath 设置为“PublishScriptLocation”，将 Result 设置为“PublishScriptFilePath”。此活动将发布脚本的路径从 TFS 服务器位置（如果适用）转换为标准本地磁盘路径。

    5.  如果你使用的是 TFS 2012 或更低版本，请在新序列的末尾添加另一个 ConvertWorkspaceItem 活动。Direction=ServerToLocal, DisplayName='Convert subscription filename', Input=' SubscriptionDataFileLocation', Result= 'SubscriptionDataFilePath', Workspace='Workspace'。如果你使用的是 TFS 2013 或更高版本，请添加另一个 GetLocalPath。IncomingPath='SubscriptionDataFileLocation' 和 Result='SubscriptionDataFilePath'。

    6.  在新的 Sequence 的末尾添加一个 InvokeProcess 活动。此活动使用生成定义传入的参数调用 PowerShell.exe。

        1.  Arguments = String.Format(" -File ""{0}"" -serviceName {1} -storageAccountName {2} -packageLocation ""{3}"" -cloudConfigLocation ""{4}"" -subscriptionDataFile ""{5}"" -selectedSubscription {6} -environment ""{7}""", PublishScriptFilePath, ServiceName, StorageAccountName, PackageLocation, CloudConfigLocation, SubscriptionDataFilePath, SubscriptionName, Environment)

        2.  DisplayName = Execute publish script

        3.  FileName = "PowerShell" (include the quotes)

        4.  OutputEncoding= System.Text.Encoding.GetEncoding(System.Globalization.CultureInfo.InstalledUICulture.TextInfo.OEMCodePage)

    7.  在 InvokeProcess 的“处理标准输出”部分文本框中，将文本框值设置为“data”。这是一个用于存储标准输出数据的变量。

    8.  在“处理错误输出”部分的正下方添加一个 WriteBuildError 活动。设置 Importance = 'Microsoft.TeamFoundation.Build.Client.BuildMessageImportance.High' 和 Message='data'。这可确保脚本的标准输出将写入到生成输出中。

    9.  在 InvokeProcess 的“处理错误输出”部分文本框中，将文本框值设置为“data”。这是一个用于存储标准错误数据的变量。

    10. 在“处理错误输出”部分的正下方添加一个 WriteBuildError 活动。设置 Message='data'。这可确保脚本的标准错误将写入到生成错误输出中。

	11. 更正蓝色感叹号指示的任何错误。将鼠标悬停在感叹号上可以获取有关错误的提示。保存工作流以清除错误。

    发布工作流活动的最终结果将与设计器中的以下内容类似：

    ![][5]

    发布工作流活动的最终结果将与 XAML 中的以下内容类似：

		<If Condition="[Not String.IsNullOrEmpty(PublishScriptLocation)]" sap2010:WorkflowViewState.IdRef="If_1">
	        <If.Then>
	          <Sequence DisplayName="Start Publish" sap2010:WorkflowViewState.IdRef="Sequence_4">
	            <Sequence.Variables>
	              <Variable x:TypeArguments="x:String" Name="SubscriptionDataFilePath" />
	              <Variable x:TypeArguments="x:String" Name="PublishScriptFilePath" />
	            </Sequence.Variables>
	            <mtbwa:ConvertWorkspaceItem DisplayName="Convert publish script filename" sap2010:WorkflowViewState.IdRef="ConvertWorkspaceItem_1" Input="[PublishScriptLocation]" Result="[PublishScriptFilePath]" Workspace="[Workspace]" />
	            <mtbwa:ConvertWorkspaceItem DisplayName="Convert subscription filename" sap2010:WorkflowViewState.IdRef="ConvertWorkspaceItem_2" Input="[SubscriptionDataFileLocation]" Result="[SubscriptionDataFilePath]" Workspace="[Workspace]" />
	            <mtbwa:InvokeProcess Arguments="[String.Format("; -File ";";{0}";"; -serviceName {1}&#xD;&#xA;            -storageAccountName {2} -packageLocation ";";{3}";";&#xD;&#xA;            -cloudConfigLocation ";";{4}";"; -subscriptionDataFile ";";{5}";";&#xD;&#xA;            -selectedSubscription {6} -environment ";";{7}";";";,&#xD;&#xA;            PublishScriptFilePath, ServiceName, StorageAccountName,&#xD;&#xA;            PackageLocation, CloudConfigLocation,&#xD;&#xA;            SubscriptionDataFilePath, SubscriptionName, Environment)]" DisplayName="'Execute Publish Script'" FileName="[PowerShell]" sap2010:WorkflowViewState.IdRef="InvokeProcess_1">
	              <mtbwa:InvokeProcess.ErrorDataReceived>
	                <ActivityAction x:TypeArguments="x:String">
	                  <ActivityAction.Argument>
	                    <DelegateInArgument x:TypeArguments="x:String" Name="data" />
	                  </ActivityAction.Argument>
	                  <mtbwa:WriteBuildError Message="{x:Null}" sap2010:WorkflowViewState.IdRef="WriteBuildError_1" />
	                </ActivityAction>
	              </mtbwa:InvokeProcess.ErrorDataReceived>
	              <mtbwa:InvokeProcess.OutputDataReceived>
	                <ActivityAction x:TypeArguments="x:String">
	                  <ActivityAction.Argument>
	                    <DelegateInArgument x:TypeArguments="x:String" Name="data" />
	                  </ActivityAction.Argument>
	                  <mtbwa:WriteBuildMessage sap2010:WorkflowViewState.IdRef="WriteBuildMessage_2" Importance="[Microsoft.TeamFoundation.Build.Client.BuildMessageImportance.High]" Message="[data]" mva:VisualBasic.Settings="Assembly references and imported namespaces serialized as XML namespaces" />
	                </ActivityAction>
	              </mtbwa:InvokeProcess.OutputDataReceived>
	            </mtbwa:InvokeProcess>
	          </Sequence>
	        </If.Then>
	      </If>
	    </Sequence>


7.  保存生成过程模板工作流并签入此文件。

8.  编辑生成定义（如果已打开，请将它关闭）。如果在“过程模板”列表中看不到新模板，请选择“新建”按钮。

9.  在“杂项”部分中设置参数属性，如下所示：

    1.  CloudConfigLocation ='c:\\drops\\app.publish\\ServiceConfiguration.Cloud.cscfg' *此值派生自：($PublishDir)ServiceConfiguration.Cloud.cscfg*

    2.  PackageLocation = 'c:\\drops\\app.publish\\ContactManager.Azure.cspkg' *此值派生自：($PublishDir)($ProjectName).cspkg*

    3.  PublishScriptLocation = 'c:\\scripts\\WindowsAzure\\PublishCloudService.ps1'

    4.  ServiceName = 'mycloudservicename' *在此处使用适当的云服务名称*

    5.  Environment = 'Staging'

    6.  StorageAccountName = 'mystorageaccountname' *在此处使用适当的存储帐户名称*

    7.  SubscriptionDataFileLocation = 'c:\\scripts\\WindowsAzure\\Subscription.xml'

    8.  SubscriptionName = 'default'

    ![][6]

10. 保存对生成定义所做的更改。

11. 对生成进行排队以便同时执行包生成和发布。如果你的触发器设置为“持续集成”，则将在每次签入时执行此行为。

### PublishCloudService.ps1 脚本模板

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
	        Write-Output "$(Get-Date -f $timeStampFormat) - No deployment is detected. Creating a new deployment. "
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
	                        Write-Output "$(Get-Date -f $timeStampFormat) - Deployment exists in $servicename.  Upgrading deployment."
					        UpgradeDeployment
	                    }
	                    0  #Delete then create new deployment
	                    {
	                        Write-Output "$(Get-Date -f $timeStampFormat) - Deployment exists in $servicename.  Deleting deployment."
					        DeleteDeployment
	                        CreateNewDeployment
	
	                    }
	                } # switch ($enableDeploymentUpgrade)
				}
		        0
				{
					Write-Output "$(Get-Date -f $timeStampFormat) - ERROR: Deployment exists in $servicename.  Script execution cancelled."
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
		Write-Output "$(Get-Date -f $timeStampFormat) - Creating New Deployment: In progress"
	
		$opstat = New-AzureDeployment -Slot $slot -Package $packageLocation -Configuration $cloudConfigLocation -label $deploymentLabel -ServiceName $serviceName
	
	    $completeDeployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
	    $completeDeploymentID = $completeDeployment.deploymentid
	
	    write-progress -id 3 -activity "Creating New Deployment" -completed -Status "Complete"
		Write-Output "$(Get-Date -f $timeStampFormat) - Creating New Deployment: Complete, Deployment ID: $completeDeploymentID"
	
		StartInstances
	}
	
	function UpgradeDeployment()
	{
		write-progress -id 3 -activity "Upgrading Deployment" -Status "In progress"
		Write-Output "$(Get-Date -f $timeStampFormat) - Upgrading Deployment: In progress"
	
	    # perform Update-Deployment
		$setdeployment = Set-AzureDeployment -Upgrade -Slot $slot -Package $packageLocation -Configuration $cloudConfigLocation -label $deploymentLabel -ServiceName $serviceName -Force
	
	    $completeDeployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
	    $completeDeploymentID = $completeDeployment.deploymentid
	
	    write-progress -id 3 -activity "Upgrading Deployment" -completed -Status "Complete"
		Write-Output "$(Get-Date -f $timeStampFormat) - Upgrading Deployment: Complete, Deployment ID: $completeDeploymentID"
	}
	
	function DeleteDeployment()
	{
	
		write-progress -id 2 -activity "Deleting Deployment" -Status "In progress"
		Write-Output "$(Get-Date -f $timeStampFormat) - Deleting Deployment: In progress"
	
	    #WARNING - always deletes with force
		$removeDeployment = Remove-AzureDeployment -Slot $slot -ServiceName $serviceName -Force
	
		write-progress -id 2 -activity "Deleting Deployment: Complete" -completed -Status $removeDeployment
		Write-Output "$(Get-Date -f $timeStampFormat) - Deleting Deployment: Complete"
	
	}
	
	function StartInstances()
	{
		write-progress -id 4 -activity "Starting Instances" -status "In progress"
		Write-Output "$(Get-Date -f $timeStampFormat) - Starting Instances: In progress"
	
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
					Write-Output "$(Get-Date -f $timeStampFormat) - Starting Instance '$instanceName': $instanceStatus"
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
				Write-Output "$(Get-Date -f $timeStampFormat) - Starting Instance '$instanceName': $instanceStatus"
			}
	
			$i = $i + 1
		}
	
	    $deployment = Get-AzureDeployment -ServiceName $serviceName -Slot $slot
		$opstat = $deployment.Status
	
		write-progress -id 4 -activity "Starting Instances" -completed -status $opstat
		Write-Output "$(Get-Date -f $timeStampFormat) - Starting Instances: $opstat"
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
	Set-AzureSubscription -CurrentStorageAccountName $storageAccountName -SubscriptionName $selectedsubscription
	Select-AzureSubscription $selectedsubscription
	
	#set remaining environment variables for Azure cmdlets
	$subscription = Get-AzureSubscription $selectedsubscription
	$subscriptionname = $subscription.subscriptionname
	$subscriptionid = $subscription.subscriptionid
	$slot = $environment
	
	#main driver - publish & write progress to activity log
	Write-Output "$(Get-Date -f $timeStampFormat) - Azure Cloud Service deploy script started."
	Write-Output "$(Get-Date -f $timeStampFormat) - Preparing deployment of $deploymentLabel for $subscriptionname with Subscription ID $subscriptionid."
	
	Publish
	
	$deployment = Get-AzureDeployment -slot $slot -serviceName $servicename
	$deploymentUrl = $deployment.Url
	
	Write-Output "$(Get-Date -f $timeStampFormat) - Created Cloud Service with URL $deploymentUrl."
	Write-Output "$(Get-Date -f $timeStampFormat) - Azure Cloud Service deploy script finished."

## 后续步骤

若要在使用持续交付时启用远程调试，请参阅[使用连续交付功能发布到 Azure 时如何启用远程调试](/documentation/articles/cloud-services-virtual-machines-dotnet-continuous-delivery-remote-debugging/)。

  [Team Foundation 生成服务]: https://msdn.microsoft.com/zh-cn/library/ee259687.aspx
  [.NET Framework 4]: https://www.microsoft.com/zh-cn/download/details.aspx?id=17851
  [.NET Framework 4.5]: https://www.microsoft.com/zh-cn/download/details.aspx?id=30653
  [.NET Framework 4.5.2]: https://www.microsoft.com/zh-cn/download/details.aspx?id=42643
  [扩大生成系统]: https://msdn.microsoft.com/zh-cn/library/dd793166.aspx
  [部署和配置生成服务器]: https://msdn.microsoft.com/zh-cn/library/ms181712.aspx
  [Azure PowerShell cmdlet]: /documentation/articles/powershell-install-configure/
  [the .publishsettings file]: https://manage.windowsazure.cn/download/publishprofile.aspx?wa=wsignin1.0
  [0]: ./media/cloud-services-dotnet-continuous-delivery/tfs-01bc.png
  [2]: ./media/cloud-services-dotnet-continuous-delivery/tfs-02.png
  [3]: ./media/cloud-services-dotnet-continuous-delivery/common-task-tfs-03.png
  [4]: ./media/cloud-services-dotnet-continuous-delivery/common-task-tfs-04.png
  [5]: ./media/cloud-services-dotnet-continuous-delivery/common-task-tfs-05.png
  [6]: ./media/cloud-services-dotnet-continuous-delivery/common-task-tfs-06.png

<!---HONumber=Mooncake_0523_2016-->
