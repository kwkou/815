<properties
   pageTitle="Azure 安全和审核日志管理 | Azure"
   description="文章提供了从托管在 Azure 上的服务生成、收集和分析安全日志的介绍。它面向每天要进行信息资产管理的 IT 专业人员和安全分析人员（包括负责其组织安全和合规性工作的人员）。"
   services="virtual-machines, cloud-services, storage"
   documentationCenter="na"
   authors="nayak-mahesh"
   manager="msStevenPo"
   editor=""/>

<tags
   ms.service="azure-security"
   ms.date="12/10/2015"
   wacn.date="01/29/2016"/>

# Azure 安全和审核日志管理

Azure 使客户能够在其订阅中执行从 Azure 服务架构 (IaaS) 和平台即服务 (PaaS) 角色到中央存储的安全事件生成和收集。然后，客户就可以使用 [HDInsight](/documentation/services/hdinsight/) 来聚合和分析所收集的事件。此外，这些收集的事件可以导出到本地安全信息和事件管理 (SIEM) 系统以便进行持续监视。

Azure 安全日志记录、分析和监视生命周期包括：

- **生成**：检测应用程序和基础结构以引发事件
- **收集**：配置 Azure 以收集存储帐户中的各种安全日志
- **分析**：使用 HDInsight 和本地 SIEM 系统等 Azure 工具来分析日志并生成安全见解
- **监视和报告**：Azure 提供具有连续可见性和及时警报功能的集中式监视和分析系统

本文重点介绍生命周期的生成和收集阶段。

## 日志生成
Windows 事件日志中引发了虚拟机中有关**系统**、**安全**和**应用程序**通道的安全事件。若要确保在不出现数据丢失的情况下记录事件，请务必正确配置事件日志的大小。根据审核策略设置生成的事件数量和事件收集策略定义的事件数量来设置事件日志的大小。有关详细信息，请参阅[安全审核监视和管理规划](http://technet.microsoft.com/zh-cn/library/ee513968.aspx#BKMK_4)。

>[AZURE.NOTE]在使用 Windows 事件转发 (WEF) 或 Azure 诊断（如[日志收集](#log-collection)部分所述）从云服务或虚拟机中拉取日志时，请考虑系统中断的潜在影响。例如，如果您的 WEF 环境在一段时间内出现故障，您要么确保日志大小足以应对更长的持续时间，要么就需要做好日志数据可能丢失的准备。

对于在 Azure 中部署的云服务应用程序和从 <!--[-->Azure 虚拟机应用商店<!--](/marketplace/virtual-machines/#microsoft)-->创建的虚拟机，默认情况下启用一组操作系统安全事件。客户可以通过自定义操作系统审核策略来添加、删除或修改要审核的事件。有关详细信息，请参阅[安全策略设置参考](http://technet.microsoft.com/zh-cn/library/jj852210.aspx)。

可以使用以下方法来从操作系统（例如，审核策略更改）和 Windows 组件（例如 IIS）生成其他日志：

- 组策略，以便为加入域的 Azure 中的虚拟机执行策略设置
- Desired State Configuration (DSC)，以便推送和管理策略设置。有关详细信息，请参阅 [Azure PowerShell DSC](http://blogs.msdn.com/b/powershell/archive/2014/08/07/introducing-the-azure-powershell-dsc-desired-state-configuration-extension.aspx)。
- 服务部署角色启动代码，以便为云服务（PaaS 方案）进行设置

配置 Azure 角色启动任务可使代码在角色启动之前运行。可以通过将 **Startup** 元素添加到服务定义文件中的角色定义来定义角色的启动任务，如下面的示例中所示。有关详细信息，请参阅[在 Azure 中运行启动任务](/documentation/articles/cloud-services-startup-tasks/)。

要作为启动任务运行的任务文件（即以下示例中的 EnableLogOnAudit.cmd）需要包含在您的生成包中。如果您使用是的 Visual Studio，请将文件添加到您的云项目，右键单击该文件名，单击“属性”，然后将“复制到输出目录”设置为“始终复制”。

    <Startup>
        <Task commandLine="EnableLogOnAudit.cmd" executionContext="elevated" taskType="simple" />
    </Startup>

EnableLogOnAudit.cmd 的内容：

    @echo off
    auditpol.exe /set /category:"Logon/Logoff" /success:enable /failure:enable
    Exit /B 0

前面示例中使用的 [Auditpol.exe](https://technet.microsoft.com/library/cc731451.aspx) 是 Windows Server 操作系统中包含的命令行工具，该操作系统允许您管理审核策略设置。

除了生成 Windows 事件日志，还可以对各种 Windows 操作系统组件进行配置以生成日志，这些日志对于安全分析和监视而言非常重要。例如，自动为 Web 角色生成的 Internet Information Services (IIS) 日志和 http.err 日志，可以配置这些日志以进行收集。这些日志提供有价值的信息，可用于标识未经授权的访问或针对您的 Web 角色的攻击。有关详细信息，请参阅[在 IIS 中配置日志记录](http://technet.microsoft.com/library/hh831775.aspx)和 [ IIS 高级日志记录 – 自定义日志记录](http://www.iis.net/learn/extensions/advanced-logging-module/advanced-logging-for-iis-custom-logging)。

若要更改 Web 角色中的 IIS 日志记录，客户可以向 Web 角色服务定义文件添加启动任务。下面的示例为名为 Contoso 的网站启用 HTTP 日志记录，并指定 IIS 应记录 Contoso 网站的所有请求。

更新 IIS 配置的任务需要包含在 Web 角色的服务定义文件中。对服务定义文件的以下更改运行一个启动任务，该启动任务通过运行名为 ConfigureIISLogging.cmd 的脚本来配置 IIS 日志记录。

    <Startup>
        <Task commandLine="ConfigureIISLogging.cmd" executionContext="elevated" taskType="simple" />
    </Startup>

ConfigureIISLogging:cmd 的内容

    @echo off
    appcmd.exe set config "Contoso" -section:system.webServer/httpLogging /dontLog:"True" /commit:apphost
    appcmd.exe set config "Contoso" -section:system.webServer/httpLogging /selectiveLogging:"LogAll" /commit
    Exit /B 0


## <a name="log-collection"></a>日志收集
通过以下两种主要方法来从云服务或 Azure 中的虚拟机收集安全事件和日志：

- Azure 诊断，收集客户的 Azure 存储帐户中的事件
- Windows 事件转发 (WEF)，运行 Windows 的计算机中的一种技术

下表包括了这两种技术之间的一些主要区别。根据您的需求以及这些主要区别，需要选择适当的方法来实现日志收集。

| Azure 诊断 | Windows 事件转发 |
|-----|-----|
|支持 Azure 虚拟机和 Azure 云服务 | 仅支持已加入域的 Azure 虚拟机 |
|支持各种日志格式，如 Windows 事件日志、[Windows 事件跟踪](https://msdn.microsoft.com/zh-cn/library/windows/desktop/bb968803.aspx) (ETW) 跟踪和 IIS 日志。有关详细信息，请参阅 [支持 Azure 诊断的数据源](#diagnostics) |仅支持 Windows 事件日志 |
|将已收集的数据推送到 Azure 存储空间 |将已收集的数据移动到中央收集器服务器 |

##	使用 Windows 事件转发进行的安全事件数据收集
对于已加入域的 Azure 虚拟机，可以使用组策略设置配置 WEF（与配置本地已加入域的计算机的方法相同）。有关详细信息，请参阅[混合云](http://www.microsoft.com/server-cloud/solutions/hybrid-cloud.aspx)。

组织可以使用此方法购买 IaaS 订阅，通过使用 [ExpressRoute](/services/expressroute/) 或站点到站点 VPN 将其连接到公司网络，然后将已在 Azure 中的虚拟机加入到企业域。之后，您可以从已加入域的计算机配置 WEF。

事件转发拆分为两个部分：源和收集器。源是在其中生成安全日志的计算机。收集器是收集并合并事件日志的集中式服务器。IT 管理员可以订阅事件，以便他们可以接收和存储从远程计算机（事件源）转发的事件。有关详细信息，请参阅[配置计算机以转发和收集事件](http://technet.microsoft.com/zh-cn/library/cc748890.aspx)。

收集的 Windows 事件可以发送到 SIEM 等本地分析工具，以做进一步分析。

## 使用 Azure 诊断进行的安全数据收集
Azure 诊断使您能够从云服务辅助角色或 Web 角色，或者从 Azure 中运行的虚拟机中收集诊断数据。这是预定义的来宾代理扩展，需要启用它并加以配置以便进行数据收集。客户的订阅可以包括将数据推送到 Azure 存储空间。

数据（通过使用 HTTPS）在传输中加密。本文档中提供的示例使用的是 Azure Diagnostics 1.2。我们建议您升级到版本 1.2 或更高版本，以便进行安全数据收集。有关详细信息，请参阅[使用 Azure 诊断收集日志记录数据](http://msdn.microsoft.com/zh-cn/library/gg433048.aspx)。

下图显示了使用 Azure 诊断、进一步分析和监视的安全数据收集的高级数据流。

![][1]

Azure 诊断将日志从客户云服务应用程序和 [Azure 虚拟机](/documentation/articles/virtual-machines-linux-about/)移动到 Azure 存储空间。基于一种日志格式将一部分数据存储在 Azure 表中，另一部分存储在 blob 中。使用 Azure 存储空间客户端库将在 [Azure 存储空间](/documentation/articles/storage-introduction/)中收集的数据下载到本地 SIEM 系统，以便进行监视和分析。

此外，HDInsight 可以用于进一步分析云中的数据。以下是一些使用 Azure 诊断的安全数据收集示例。

### 使用 Azure 诊断从 Azure 虚拟机进行安全数据收集

下面的示例使用 Azure Diagnostics 1.2 和 Azure PowerShell cmdlet 从虚拟机启用安全数据收集。在计划的时间间隔（这是可配置的）里从虚拟机收集数据并将其推送到客户订阅中的 Azure 存储空间。在本部分中，我们将逐步介绍如何使用 Azure 诊断完成两个日志收集方案：

1. 在虚拟机上设置安全日志收集管道的新实例。
2. 使用虚拟机上的新配置更新现有安全日志收集管道。

#### 在虚拟机上设置安全日志收集管道的新实例
在本示例中，我们设置了使用 Azure 诊断的安全日志收集管道的新实例，并检测到虚拟机中的登录失败事件（事件 ID 为 4624 和 4625）。您可以从您的开发环境中实现以下步骤，也可以使用远程桌面会话通过远程桌面协议 (RDP) 连接到云中的节点。

##### 步骤 1：安装 Azure PowerShell SDK
Azure PowerShell SDK 提供用于配置 Azure 虚拟机上的 Azure 诊断的 cmdlet。Azure PowerShell 版本 0.8.7 或更高版本中提供这些必要的 cmdlet。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

##### 步骤 2：准备配置文件
根据您想要收集的事件准备配置文件。下面举例说明 Azure 诊断配置文件通过添加筛选器仅收集登录成功和失败事件，来从**安全**通道收集 Windows 事件。有关详细信息，请参阅 [Azure Diagnostics 1.2 配置架构](http://msdn.microsoft.com/zh-cn/library/azure/dn782207.aspx)。

可以在配置文件中指定存储帐户，也可以在运行 Azure PowerShell cmdlet 设置 Azure 诊断时将存储帐户指定为参数。

    <?xml version="1.0" encoding="utf-8" ?>
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <WadCfg>
            <DiagnosticMonitorConfiguration overallQuotaInMB="10000">
                <WindowsEventLog scheduledTransferPeriod="PT1M">
                    <DataSource name="Security!*[System[(EventID=4624 or EventID=4625)]]" />
                </WindowsEventLog>
            </DiagnosticMonitorConfiguration>
        </WadCfg>
    </PublicConfig>

在将以前的内容保存为 XML 文件时，将编码设置为 **UTF-8**。如果您使用的是记事本，将在“另存为”对话框中看到编码选项。下表列出了配置文件中一些需要注意的关键属性。

| 属性 | 说明 |
|----- |-----|
| overallQuotaInMB | 可供 Azure 诊断使用的最大本地磁盘空间量（值可配置）。 |
| scheduledTransferPeriod | 到 Azure 存储空间的计划传输之间的间隔向上舍入为最接近的分钟数。 |
| Name | 在 WindowsEventLog 中，此属性是描述要收集的 Windows 事件的 XPath 查询。您可以通过添加筛选器（如事件 ID、提供程序名称或通道）来筛选数据收集。 |

将所有 Windows 事件日志数据移动到名为 **WADWindowsEventLogsTable** 的表中。目前，Azure 诊断不支持重命名表。

##### <a name="step3"></a> 步骤 3：验证配置 XML 文件
使用以下过程来验证配置 XML 文件中是否存在错误，以及是否与 Azure 诊断架构兼容：

1. 若要下载架构文件，请运行以下命令，然后保存该文件。

    (Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File -Encoding utf8 -FilePath 'WadConfigSchema.xsd'

2. 下载架构文件后，可以针对该架构验证配置 XML 文件。若要使用 Visual Studio 验证文件，请执行以下操作：
  - 在 Visual Studio 中打开该 XML 文件
  - 按 F4 打开“属性”
  - 依次单击“架构”和“添加”，选择要下载的架构文件 (WadConfigSchema.XSD)，然后单击“确定”

3.	在“视图”菜单中，单击“错误列表”查看是否存在任何验证错误。

##### <a name="step4"></a> 步骤 4：配置 Azure 诊断
 使用以下步骤启用 Azure 诊断并启动数据收集：

 1.	若要打开 Azure PowerShell，键入 **Add-AzureAccount**，然后按 ENTER。
 2.	使用您的 Azure 帐户进行登录。
 3.	运行以下 PowerShell 脚本。请确保更新 storage\_name、key、config\_path、service\_name 和 vm\_name。

 ```PowerShell
$storage_name ="<Storage Name>"
$key = "<Storage Key>"
$config_path="<Path Of WAD Config XML>"
$service_name="<Service Name. Usually it is same as VM Name>"
$vm_name="<VM Name>"
$storageContext = New-AzureStorageContext -StorageAccountName $storage_name -StorageAccountKey $key
$VM1 = Get-AzureVM -ServiceName $service_name -Name $vm_name
$VM2 = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $config_path -Version "1.*" -VM $VM1 -StorageContext $storageContext
$VM3 = Update-AzureVM -ServiceName $service_name -Name $vm_name -VM $VM2.VM
 ```

##### 步骤 5：生成事件
出于演示目的，我们将创建一些登录事件并验证数据是否流向 Azure 存储空间。如先前在步骤 2 中所示，将 XML 文件配置为从**安全**通道收集事件 ID 4624（登录成功）和事件 ID 4625（登录失败）。

 若要生成这些事件，请执行以下操作：

1.	打开到您虚拟机的 RDP 会话。
2.	输入不正确的凭据以生成一些失败的登录事件（事件 ID 4625）。
3.	尝试几次失败登录后，输入正确的凭据以生成成功登录事件（事件 ID 4624）。

##### 步骤 6：查看数据
完成前面的步骤约五分钟后，数据应根据 XML 文件中的配置开始流向客户存储帐户。有许多工具可用于从 Azure 存储空间查看数据。有关详细信息，请参阅：

- [使用服务器资源管理器浏览存储资源](http://msdn.microsoft.com/zh-cn/library/azure/ff683677.aspx)
- [Azure Storage Explorer 6 Preview 3（2014 年 8 月）](http://azurestorageexplorer.codeplex.com/)

若要查看数据，请执行以下操作：

1.	在 Visual Studio（2013、2012 和 2010 SP1）中，单击“视图”，然后单击“服务器资源管理器”。
2.	导航到您的存储帐户。
3.	单击“表”，然后双击相应的表来查看从虚拟机中收集的安全日志。
![][2]

4.	右键单击名为 WADWindowsEventLogsTable 的表，然后单击“查看数据”以打开表视图，如下所示：

![][3]

在以前的存储表中，**PartitionKey**、**RowKey** 和 **Timestamp** 都是系统属性。

- **PartitionKey** 是一个以秒为单位的时间戳，同时也是表中此分区的唯一标识符。
- **RowKey** 是分区中实体的唯一标识符。

**PartitionKey** 和 **RowKey** 共同唯一标识表中的每个实体。

- 时间戳是在服务器上维护的日期/时间值，用来跟踪实体上次修改的时间。

>[AZURE.NOTE]Azure 存储空间表中的最大行大小被限制为 1 MB。如果帐户在 2012 年 6 月之后创建，则存储帐户可以包含多达 200 TB 的 blob、队列和表中的数据。因此，如果 blob 和队列没有任何存储空间，则表大小可以增长至 200 TB。在 2012 年 6 月之前创建的帐户具有 100 TB 的限制。

您还可以选择在存储资源管理器中编辑表数据。双击表视图中的特定行以打开“编辑实体”窗口，如下所示：

![][4]

#### 使用虚拟机上的新配置更新现有安全日志收集管道
在本部分中，我们将更新虚拟机上现有的 Azure 诊断安全日志收集管道，并检测 Windows 应用程序事件日志错误。

##### 步骤 1：更新配置文件以包括感兴趣的事件
需要更新上一示例中创建的 Azure 诊断文件以包括 Windows 应用程序事件日志错误类型。

>[AZURE.NOTE]需要将任何现有的 Azure 诊断配置设置与新的配置文件进行合并。在新文件中定义的设置将覆盖现有的配置。

若要检索现有配置设置，可以使用 **Get-AzureVMDiagnosticsExtension** cmdlet。下面是一个用于检索现有配置的示例 Azure PowerShell 脚本：

    $service_name="<VM Name>"
    $VM1 = Get-AzureVM -ServiceName $service_name
    $config = Get-AzureVMDiagnosticsExtension -VM $VM1 | Select -Expand PublicConfiguration | % {$_.substring($_.IndexOf(':"')+2,$_.LastIndexOf('",')-$_.IndexOf(':"')-2)}
    [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($config))
更新 Azure 诊断配置以收集 Windows 应用程序事件日志错误和关键事件，如下所示：

    <?xml version="1.0" encoding="utf-8" ?>
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <WadCfg>
        <DiagnosticMonitorConfiguration overallQuotaInMB="10000">
            <WindowsEventLog scheduledTransferPeriod="PT1M">
                <DataSource name="Application!*[System[(Level =2)]]" />
                <DataSource name="Security!*[System[(EventID=4624 or EventID=4625)]]" />
            </WindowsEventLog>
        </DiagnosticMonitorConfiguration>
    </WadCfg>
    </PublicConfig>

使用前面[步骤 3：验证配置 XML 文件](#step3)中所示的相同步骤来验证配置文件。

##### 步骤 2：更新 Azure 诊断以使用新配置文件
使用 **Set-AzureVMDiagnosticsExtension** 和 **Update-AzureVM** cmdlet 更新配置，如前面[步骤 4：配置 Azure 诊断](#step4)中所示。

##### 步骤 3：验证配置设置
运行以下命令来验证配置设置是否已更新：

    $service_name="<VM Name>"
    $VM1 = Get-AzureVM -ServiceName $service_name
    Get-AzureVMDiagnosticsExtension -VM $VM1

##### 步骤 4：生成事件
针对本例，请运行以下命令来生成类型为**错误**的应用程序事件日志：

    eventcreate /t error /id 100 /l application /d "Create event in application log for Demo Purpose"

![][5]

打开事件查看器来验证是否创建了该事件。

![][6]

##### 步骤 5：查看数据
在 Visual Studio 中打开服务器资源管理器来查看日志数据。您应该会看到在 **ContosoDesktop** 上创建的**事件 ID 100**，如下所示：

![][7]

## 使用 Azure 诊断从 Azure 云服务进行安全数据收集

我们现在将使用 Azure 诊断来从 Azure 云服务浏览相同的两个日志收集方案，如前面的虚拟机 (IaaS) 部分所示：

1.	设置云服务中的安全日志管道的新实例。
2.	使用云服务中的新配置更新现有的日志收集管道。

本部分中的分步演练包括：

1.	构建云服务。
2.	使用 Azure 诊断配置用于安全日志收集的云服务。
3.	说明云服务上安全事件的生成和收集：

    - 向具有提升权限的本地组添加一个管理员
    - 创建新进程
4.	更新云服务中的现有日志收集管道：

    - 使用 Auditpol 启用主机防火墙事件的审核（作为网络安全事件的一个示例）
    - 配置要收集的防火墙审核数据，并在客户存储帐户中显示所收集的事件
5.	显示 Windows 安全事件分发和峰值检测。
6.	配置 IIS 日志收集并验证数据。

所有事件和日志将收集到 Azure 中的客户存储帐户中。客户可以查看这些事件并将其导出到本地 SIEM 系统。可以使用 HDInsight 对这些事件进行聚合和分析。

### 在云服务上设置日志收集管道的新实例
在本示例中，我们设置了使用 Azure 诊断的安全日志收集管道的新实例，并且检测到向本地组添加了用户，以及云服务实例上的新进程创建事件。

#### 步骤 1：创建云服务（Web 角色）和部署

1.	在开发计算机上启动 Visual Studio 2013。
2.	创建一个新的云服务项目（我们的示例中使用的是 ContosoWebRole）。
3.	选择 **ASP.NET** Web 角色。
4.	选择 **MVC** 项目。
5.	在解决方案资源管理器中，单击“角色”，然后双击“Web 角色” (WebRole1) 以打开“属性”窗口。
6.	在“配置”选项卡上，清除“启用诊断”复选框以禁用 Visual Studio 2013 随附的 Azure 诊断版本。 ![][8]

7.	生成你的解决方案，以确认不会出错。
8.	打开 WebRole1/Controllers/HomeController.cs 文件。
9.	添加以下方法来使示例应用程序将 HTTP 状态代码 500 记录为示例 IIS 日志事件（这将在稍后的 IIS 示例中使用）：

    ```
    public ActionResult StatusCode500()
        {
            throw new InvalidOperationException("Response.StatusCode is 500");
        }
    ```

10.	 右键单击云服务项目的名称，然后单击“发布”。

#### 步骤 2：准备配置文件
我们现在将准备 Azure 诊断配置文件以添加可以帮助检测以下情况下的事件：

- 向本地组添加新用户
- 创建新进程

```
<?xml version="1.0" encoding="UTF-8"?>
<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <WadCfg>
        <DiagnosticMonitorConfiguration overallQuotaInMB="25000">
            <WindowsEventLog scheduledTransferPeriod="PT1M">
                <DataSource name="Security!*[System[(EventID=4732 or EventID=4733 or EventID=4688)]]" />
            </WindowsEventLog>
        </DiagnosticMonitorConfiguration>
    </WadCfg>
</PublicConfig>
```

#### 步骤 3：验证配置 XML 文件
使用前面[步骤 3：验证配置 XML 文件](#step3)中所示的相同步骤来验证配置文件。
#### 步骤 4：配置 Azure 诊断
运行以下 Azure PowerShell 脚本来启用 Azure 诊断（请确保更新 storage\_name、key、config\_path 和 service\_name）。

    $storage_name = "<storage account name>"
    $key = " <storage key>"
    $config_path="<path to configuration XML file>"
    $service_name="<Cloud Service Name>"
    $storageContext = New-AzureStorageContext -StorageAccountName $storage_name -StorageAccountKey $key
    Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext -DiagnosticsConfigurationPath $config_path -ServiceName $service_name

若要验证您的服务是否具有最新的诊断配置，请运行以下 Azure PowerShell 命令：

    Get-AzureServiceDiagnosticsExtension -ServiceName <ServiceName>

#### 步骤 5：生成事件
若要生成事件，请执行以下操作：

1.	若要启动到云服务实例的远程桌面会话，请在 Visual Studio 中打开服务器资源管理器，右键单击该角色实例，然后单击“使用远程桌面连接”。
2.	打开提升的命令提示符并运行以下命令以创建虚拟机上的本地管理员帐户：


    net user contosoadmin <enterpassword> /add
    net localgroup administrators contosoadmin /add

3.	打开“事件查看器”，打开“安全”通道，并注意是否已创建事件 4732，如下所示：

![][9]

#### 步骤 6：查看数据
等待大约五分钟以允许 Azure 诊断代理将事件推送到存储表。

![][10]

若要验证该进程创建事件，请打开“记事本”。如下所示，在“安全”通道中记录了进程创建事件。

![][11]

您现在可以在存储帐户中查看同一事件，如下所示：

![][12]

### 使用新配置更新云服务中现有的日志收集管道
在本部分中，我们将更新现有的 Azure 诊断安全日志收集管道，并检测云服务实例中的 Windows 防火墙更改事件。

若要检测防火墙更改，我们将更新现有配置以包括防火墙更改事件。

#### 步骤 1：获取现有配置
>[AZURE.NOTE]新的配置设置将覆盖现有配置。因此，请务必将任何现有的 Azure 诊断配置设置与新的配置文件进行合并。

若要检索现有配置设置，可以使用 **Get-AzureServiceDiagnosticsExtension** cmdlet：

    Get-AzureServiceDiagnosticsExtension -ServiceName <ServiceName>

#### 步骤 2：更新配置 XML 以包括防火墙事件

    <?xml version="1.0" encoding="UTF-8"?>
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <WadCfg>
        <DiagnosticMonitorConfiguration overallQuotaInMB="25000">
        <WindowsEventLog scheduledTransferPeriod="PT1M">
            <DataSource name="Security!*[System[(EventID=4732 or EventID=4733 or EventID=4688)]]" />
            <DataSource name="Security!*[System[(EventID &gt;= 4944 and EventID &lt;= 4958)]]" />
        </WindowsEventLog>
        </DiagnosticMonitorConfiguration>
    </WadCfg>
    </PublicConfig>

通过使用前面[步骤 3：验证配置 XML 文件](#step3)中所述的相同验证过程来验证 XML 内容。

#### 步骤 3：更新 Azure 诊断以使用新配置

运行以下 Azure PowerShell 脚本来更新 Azure 诊断以使用新配置（请确保使用您的云服务信息更新 storage\_name、key、config\_path 和 service\_name)。

    Remove-AzureServiceDiagnosticsExtension -ServiceName <ServiceName> -Role <RoleName>
    $storage_name = "<storage account name>"
    $key = " <storage key>"
    $config_path="<path to configuration XML file>"
    $service_name="<Cloud Service Name>"
    $storageContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storage_name -StorageAccountKey $key
    Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext -DiagnosticsConfigurationPath $config_path -ServiceName $service_name

若要验证您的服务是否具有最新的诊断配置，请运行以下 Azure PowerShell 命令：

    Get-AzureServiceDiagnosticsExtension -ServiceName <ServiceName>

#### 步骤 4：启用防火墙事件

1.	打开到您的云服务实例的远程桌面会话。
2.	打开提升的命令提示符并运行以下命令：

    ```
    auditpol.exe /set /category:"Policy Change" /subcategory:"MPSSVC rule-level Policy Change" /success:enable /failure:enable
    ```

#### 步骤 5：生成事件

1.	打开“Windows 防火墙”，然后单击“入站规则”。
2.	单击“添加新规则”，然后单击“端口”。
3.	在“本地端口”字段中，键入 **5000**，然后单击“下一步”三次。
4.	在“名称”字段中键入 **Test5000**，然后单击“完成”。
5.	打开“事件查看器”，打开“安全”通道，并注意是否已创建事件 ID 4946，如下所示：

![][13]

#### 步骤 6：查看数据
等待大约五分钟以允许 Azure 诊断代理将事件数据推送到存储表。

![][14]

### 安全事件分发和峰值检测
在事件都位于客户的存储帐户后，应用程序可以使用存储客户端库来访问和执行事件聚合。有关访问表数据的示例代码，请参阅[操作说明：检索表数据](/documentation/articles/storage-dotnet-how-to-use-tables/)。

下面是一个事件聚合示例。可进一步调查事件分发中的任何峰值以查找异常活动。

![][15]

### IIS 日志收集并使用 HDInsight 进行处理
在本部分中，我们从您的 Web 角色实例收集 IIS 日志，并将日志移动到客户存储帐户的 Azure blob 中。

#### 步骤 1：更新配置文件以包括 IIS 日志收集

    <?xml version="1.0" encoding="UTF-8"?>
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <WadCfg>
        <DiagnosticMonitorConfiguration overallQuotaInMB="25000">
        <Directories scheduledTransferPeriod="PT5M">
        <IISLogs containerName="iislogs" />
        </Directories>
        <WindowsEventLog scheduledTransferPeriod="PT1M">
            <DataSource name="Security!*[System[(EventID=4732 or EventID=4733 or EventID=4688)]]" />
            <DataSource name="Security!*[System[(EventID &gt;= 4944 and EventID &lt;= 4958)]]" />
        </WindowsEventLog>
        </DiagnosticMonitorConfiguration>
    </WadCfg>
    </PublicConfig>

在前面的 Azure 诊断配置中，**containerName** 是在客户的存储帐户内要将日志移动到的 blob 容器名称。

使用前面[步骤 3：验证配置 XML 文件](#step3)中所示的相同步骤来验证配置文件。


#### 步骤 2：更新 Azure 诊断以使用新配置
运行以下 Azure PowerShell 脚本来更新 Azure 诊断以使用新配置（请确保使用您的云服务信息更新 storage\_name、key、config\_path 和 service\_name)。

    Remove-AzureServiceDiagnosticsExtension -ServiceName <ServiceName> -Role <RoleName>
    $storage_name = "<storage account name>"
    $key = " <storage key>"
    $config_path="<path to configuration XML file>"
    $service_name="<Cloud Service Name>"
    $storageContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storage_name -StorageAccountKey $key
    Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext -DiagnosticsConfigurationPath $config_path -ServiceName $service_name

若要验证您的服务是否具有最新的诊断配置，请运行以下 Azure PowerShell 命令：

    Get-AzureServiceDiagnosticsExtension -ServiceName <ServiceName>

#### 步骤 3：生成 IIS 日志

1.	打开 Web 浏览器并导航到云服务 Web 角色（例如，http://contosowebrole.chinacloudapp.cn/)）。
2.	导航到“关于”和“联系人”页面以创建部分日志事件。
3.	导航到生成状态代码 500 的页面（例如，http://contosowebrole.chinacloudapp.cn/Home/StatusCode500 ）。您应该会看到一个错误，如下所示。请记住，我们在标题为“设置云服务名上日志收集管道的新实例”部分的步骤 1 中为 **StatusCode500** 添加了代码。
![][16]
4.	打开到您的云服务实例的远程桌面会话。
5.	打开 IIS 管理器。
6.	默认情况下，启用 IIS 日志记录并将其设置为每小时生成包含 W3C 格式中的所有字段的文件。单击“浏览”，至少会显示一个日志文件，如下所示：
![][17]

7.	等待大约五分钟以便 Azure 诊断代理将日志文件推送到 blob 容器。若要验证此数据，请打开“服务器资源管理器”>“存储”>“存储帐户”>“Blob”。如此处所示，创建了 blob **iislogs**：
![][18]

8.	右键单击并选中“查看 Blob 容器”以显示存储在 blob 中的 IIS 日志文件：
![][19]
9.	在 IIS 事件都位于客户的存储帐户后，利用 HDInsight 分析的应用程序可以用于执行事件聚合。下面的折线图是显示 HTTP 状态代码 500 的事件聚合任务的一个示例：
![][20]

## 安全日志收集的建议
在收集安全日志时，我们建议您：

- 收集您自己的特定于服务应用程序的审核日志事件。
- 仅配置分析和监视所需的数据。捕获过多数据会导致更加难以进行故障排除，并可能会影响您的服务或存储成本。
- 将现有的 Azure 诊断配置设置与您所做的更改进行合并。新的配置文件将覆盖现有的配置设置。
- 明智地选择“计划传输时间段”间隔。较短的传输时间将会提高数据相关性，但可能会增加存储成本和处理开销。

>[AZURE.NOTE]将会显著影响收集的数据量的另一个变量是日志记录级别。下面举例说明如何通过日志记录级别筛选日志：

    System!*[System[(Level =2)]]

日志记录级别是累积的。如果将筛选器设置为**警告**，则还会收集**错误**和**严重**事件。

- 如果不再需要诊断数据，请定期将其从 Azure 存储空间中清除。

>[AZURE.NOTE]若要了解有关诊断数据的详细信息，请参阅[在 Azure 存储空间中存储和查看诊断数据](https://msdn.microsoft.com/zh-cn/library/azure/hh411534.aspx)。容器和存储诊断数据的表就像其他容器和表一样，您可以采用对其他数据采用的相同方法来删除其中的 blob 和实体。您可以采用编程方式通过其中一个存储客户端库或采用直观的方式通过[存储资源管理器客户端](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx)来删除诊断数据。

- 它是将服务数据和安全日志数据存储在单独的存储帐户的最佳做法。这种隔离可确保安全日志数据的保存不会影响生产服务数据的存储性能。
- 基于组织的合规性策略、数据分析和监视要求选择日志保留期。

## 将安全日志导出到其他系统
可以使用 Azure 存储空间客户端库下载 blob 数据，然后将其导出到您的本地系统进行处理。有关管理 blob 数据的示例代码，请参阅[如何通过 .NET 使用 Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)。

同样，您可以使用 Azure 存储空间客户端库下载存储在 Azure 表中的安全数据。若要了解有关访问存储在表中的数据的详细信息，请参阅[如何通过 .NET 使用表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)。

## Azure Active Directory 报告
Azure Active Directory (Azure AD) 包括一组安全、使用情况和审核日志报告，让您清楚地了解 Azure AD 租户的完整性和安全性。例如，Azure AD 能够自动分析用户活动和显示异常访问，然后通过客户可见的报告提供这一功能。

通过“Active Directory”>“目录”下的“[Azure 管理门户](https://manage.windowsazure.cn/)”提供这些报告。其中一些报告是免费的，而其他报告作为 Azure AD Premium 版本的一部分功能提供。有关 Azure AD 报告的详细信息，请参阅[查看访问和使用情况报告](http://msdn.microsoft.com/zh-cn/library/azure/dn283934.aspx)。

## Azure 操作日志
与您的 Azure 订阅资源相关的操作日志还可通过管理门户中的“操作日志”功能提供。

若要查看“操作日志”，请打开“[Azure 管理门户](https://manage.windowsazure.cn/)”，依次单击“管理服务”和“操作日志”。

## <a name="diagnostics"></a>支持 Azure 诊断的数据源

| 数据源 | 说明 |
|----- | ----- |
| IIS Logs | 有关 IIS 网站的信息 |
| Azure Diagnostics基础结构日志 | 有关 Azure 诊断的信息 |
| IIS 失败请求日志 | 有关 IIS 网站或应用程序的失败请求的信息 |
| Windows 事件日志 | 发送到 Windows 事件日志记录系统的信息 |
| 性能计数器 | 操作系统和自定义性能计数器 |
| 故障转储 | 有关应用程序崩溃时进程状态的信息 |
| 自定义错误日志 | 应用程序或服务创建的日志 |
| .NET EventSource | 使用 .NET EventSource 类由代码生成的事件 |
| 基于清单的 ETW | 由任何进程生成的 Windows 事件的事件跟踪 |

## 其他资源
以下资源提供有关 Azure 和相关的 Microsoft 服务的常规信息：

- [Azure 信任中心](/support/trust-center/)

    有关如何为 Azure 开发嵌入安全和隐私的信息以及 Azure 如何满足广泛的国际和特定于行业的合规性标准的信息

- [Azure 主页](http://www.azure.cn)

    有关 Azure 常规信息和链接

- [Azure 文档中心](/documentation)

    Azure 服务和自动化脚本指南

- [Microsoft 安全响应中心 (MSRC)](https://technet.microsoft.com/zh-cn/security/dn440717)

    MSRC 与世界各地的合作伙伴和安全研究人员一起致力于防止安全事件的发生并继续提高 Microsoft 产品的安全性

- [Microsoft 安全响应中心电子邮件](mailto:secure@microsoft.com)

    通过电子邮件报告 Microsoft 安全漏洞，包括 Azure

<!--Image references-->
[1]: ./media/azure-security-audit-log-management/sec-security-data-collection-flow.png
[2]: ./media/azure-security-audit-log-management/sec-storage-table-security-log.PNG
[3]: ./media/azure-security-audit-log-management/sec-wad-windows-event-logs-table.png
[4]: ./media/azure-security-audit-log-management/sec-edit-entity.png
[5]: ./media/azure-security-audit-log-management/sec-app-event-log-cmd.png
[6]: ./media/azure-security-audit-log-management/sec-event-viewer.png
[7]: ./media/azure-security-audit-log-management/sec-event-id100.png
[8]: ./media/azure-security-audit-log-management/sec-diagnostics.png
[9]: ./media/azure-security-audit-log-management/sec-event4732.png
[10]: ./media/azure-security-audit-log-management/sec-step6.png
[11]: ./media/azure-security-audit-log-management/sec-process-creation-event.png
[12]: ./media/azure-security-audit-log-management/sec-process-creation-event-storage.png
[13]: ./media/azure-security-audit-log-management/sec-event4946.png
[14]: ./media/azure-security-audit-log-management/sec-event4946-storage.png
[15]: ./media/azure-security-audit-log-management/sec-event-aggregation.png
[16]: ./media/azure-security-audit-log-management/sec-status-code500.png
[17]: ./media/azure-security-audit-log-management/sec-w3c-format.png
[18]: ./media/azure-security-audit-log-management/sec-blob-iis-logs.png
[19]: ./media/azure-security-audit-log-management/sec-view-blob-container.png
[20]: ./media/azure-security-audit-log-management/sec-hdinsight-analysis.png

<!---HONumber=Mooncake_0118_2016-->