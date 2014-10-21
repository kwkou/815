<properties linkid="manage-services-how-to-monitor-a-cloud-service" urlDisplayName="How to monitor" pageTitle="How to monitor a cloud service - Azure" metaKeywords="Azure monitoring cloud services, Azure Management Portal cloud services" description="Learn how to monitor cloud services by using the Azure Management Portal." metaCanonical="" services="cloud-services" documentationCenter="" title="How to Monitor Cloud Services" authors="ryanwi" solutions="" manager="" editor="" />

# 如何监视云服务

[WACOM.INCLUDE [免责声明][免责声明]]

你可以在 Azure 管理门户中监视你的云服务的关键性能度量值。你可以针对每个服务角色将监视级别设置为“最少监视”和“详细监视”，并且可以自定义监视显示信息。详细监视数据存储在存储帐户中，你可以在门户外部访问该存储帐户。

管理门户中的监视显示信息是高度可配置的。你可以在“监视”页上的度量值列表中选择要监视的度量值，并且可以选择要在“监视”页和仪表板上的度量值图表中显示的度量值。

## 目录

-   [概念][概念]
-   [如何：为云服务配置监视][如何：为云服务配置监视]
-   [如何：接收云服务度量值的警报][如何：接收云服务度量值的警报]
-   [如何：向度量值表中添加度量值][如何：向度量值表中添加度量值]
-   [如何：自定义度量值图表][如何：自定义度量值图表]
-   [如何：在管理门户外部访问详细监视数据][如何：在管理门户外部访问详细监视数据]

## <span id="concepts"></span></a>概念

默认情况下，将使用从角色实例（虚拟机）的主机操作系统收集到的性能计数器数据对新的云服务进行最少监视。最少监视度量值限于“CPU 百分比”、“输入数据量”、“输出数据量”、“磁盘读取吞吐量”以及“磁盘写入吞吐量”。通过配置详细监视，你可以根据虚拟机（角色实例）中的性能数据设定其他度量值。详细监视度量值可对应用程序运行期间出现的问题进行进一步分析。

> [WACOM.NOTE]
> 如果你使用详细监视，则可通过诊断配置文件，或者远程使用 Azure 诊断 API，在角色实例启动时添加更多性能计数器。为了能够在管理门户中监视这些度量值，你必须在配置详细监视之前添加性能计数器。有关详细信息，请参阅 [使用 Azure 诊断收集日志记录数据][使用 Azure 诊断收集日志记录数据]和[在 Azure 应用程序中创建和使用性能计数器][在 Azure 应用程序中创建和使用性能计数器]。

默认情况下，将每隔 3 分钟从角色实例中收集和传输一次性能计数器数据。当你启用详细监视时，将每隔 5 分钟、1 小时和 12 小时为每个角色实例，以及每个角色的所有角色实例汇总一次原始性能计数器数据。该汇总数据将在 10 天之后清除。

在启用详细监视后，汇总后的监视数据存储在你的存储帐户中的相应表中。若要为某个角色启用详细监视，必须配置链接到该存储帐户的诊断连接字符串。你可以对不同的角色使用不同的存储帐户。

请注意，启用详细监视将增加与数据存储、数据传输和存储事务相关的存储成本。最少监视不需要存储帐户。即使你将监视级别设置为“详细监视”，在最少监视级别公开的度量值数据也不会存储在你的存储帐户中。

## <span id="verbose"></span></a>如何：为云服务配置监视

可使用以下过程在管理门户中配置详细监视或最少监视。在启用 Azure 诊断，以及配置支持 Azure 诊断访问用于存储详细监视数据的存储帐户的诊断连接字符串之前，你无法启用详细监视。

### 开始之前

-   创建用于存储监视数据的存储帐户。你可以对不同的角色使用不同的存储帐户。有关详细信息，请参阅“存储帐户”帮助，或者参阅[如何创建存储帐户][如何创建存储帐户]。

-   为你的云服务角色启用 Azure 诊断。 <br /><br />你必须更新云服务定义文件 (.csdef) 和云服务配置文件 (.cscfg)。有关详细信息，请参阅[配置 Azure 诊断][配置 Azure 诊断]。

在管理门户中，你可以添加或修改 Azure 诊断用于访问存储详细监视数据的存储帐户的诊断连接字符串，你还可以将监视级别设置为“详细监视”或“最少监视”。由于详细监视将数据存储在存储帐户中，因此你必须首先配置诊断连接字符串，然后才能将监视级别设置为“详细监视”。

### 为详细监视配置诊断连接字符串

1.  复制你将用于存储详细监视数据的存储帐户的存储访问密钥。在 [Azure 管理门户][Azure 管理门户]中，你可以使用“存储帐户”页上的“管理密钥”。有关详细信息，请参阅[如何管理云服务][如何管理云服务]或者参阅“存储帐户”页上的帮助。

2.  打开“云服务”。随后，若要打开仪表板，请单击要配置的云服务的名称。

3.  单击“生产”或“过渡”以显示要配置的部署。

4.  单击**“配置”**。

    你将在“配置”页的顶部编辑“监视”设置，如下所示。如果你没有为云服务启用 Azure 诊断，则“级别”选项不可用。你无法更改数据保留策略。云服务的详细监视数据将存储 10 天。

    ![监视选项][监视选项]

5.  在“诊断连接字符串”中，为你要进行详细监视的每个角色完成诊断连接字符串。

    连接字符串具有以下格式。（该示例针对使用默认端点的云服务。）若要更新连接字符串，请输入要使用的存储帐户的有效存储帐户名称和存储访问密钥。

    DefaultEndpointsProtocol=https;AccountName=StorageAccountName;AccountKey=StorageAccountKey

6.  单击**“保存”**。

如果你要启用详细监视，请在为服务角色配置诊断连接字符串之后执行下一步。

### 将监视级别更改为详细监视或最少监视

1.  在[管理门户][Azure 管理门户]中，打开云服务部署的“配置”页。

2.  在“级别”中，单击“详细”或“最少”。

3.  单击**“保存”**。

在打开详细监视后，你应开始在管理门户中看到一小时内的监视数据。

原始性能计数器数据和汇总监视数据存储在存储帐户中由角色的部署 ID 限定的表中。

## <span id="receivealerts"></span></a>如何：接收云服务度量值的警报

你可能会基于云服务监视度量值收到警告。在 Azure 管理门户上的“管理服务”页上，你可以创建一条规则，以便在你选择的度量值达到指定的值时触发警报。你还可以选择在触发警报时发送电子邮件。有关更多信息，请参见[如何：在 Azure 中接收警报通知和管理警报规则][如何：在 Azure 中接收警报通知和管理警报规则]。

## <span id="addmetrics"></span></a>如何：向度量值表中添加度量值

1.  在[管理门户][管理门户]中，打开云服务的“监视”页。

    默认情况下，度量值表显示可用度量值的子集。下图中显示云服务的默认详细监视度量值，该度量值仅由 Memory\\Available MBytes 性能计数器提供，并且数据在角色级别汇总。使用“添加度量值”可选择要在管理门户中监视的其他汇总和角色级别的度量值。

    ![详细监视屏幕][详细监视屏幕]

2.  向度量值表中添加度量值：

    a. 单击“添加度量值”打开“选择度量值”，如下所示。
    将展开第一个可用度量值以显示可用选项。对于每个度量值，最上面的选项显示所有角色的汇总监视数据。此外，你还可以选择要为其显示数据的各个角色。

    ![添加度量值][添加度量值]

    b. 选择要显示的度量值：

    -   单击度量值旁边的向下箭头，以展开监视选项。
    -   选中要显示的每个监视选项对应的复选框。

    可以在度量值表中显示多达 50 个度量值。

    <div class="dev-callout"> 
<b>提示</b> 
<p>在详细监视中，度量值列表可以包含几十个度量值。要显示滚动条，请将鼠标指针悬停于对话框右侧。若要筛选列表，请单击搜索图标，并在搜索框中输入文本，如下所示。</p> 
</div>

    ![添加度量值搜索][添加度量值搜索]

3.  在选择度量值后，请单击“确定”（复选标记）。

    所选度量值将添加到度量值表中，如下所示。

    ![监视度量值][监视度量值]

4.  若要从度量值表中删除某个度量值，请单击该度量值以选中它，然后单击“删除度量值”。（在选择度量值后，你只会看到“删除度量值”。）

## <span id="customizechart"></span></a>如何：自定义度量值图表

1.  在度量值表中，最多可选择 6 个要显示在度量值图表上的度量值。要选择度量值，请单击其左侧的复选框。若要从度量值图表中删除度量值，请在度量值表中清除其复选框。

    当你选择度量值表中的度量值时，这些度量值将添加到度量值图表中。在收缩视图中，“更多(n 个)”下拉列表包含该视图中无法显示的度量值标题。

2.  若要切换显示相对值（仅显示每个度量值的最终值）和绝对值（显示 Y 轴），请在图表顶部选择“相对”或“绝对”。

    ![相对或绝对][相对或绝对]

3.  若要更改度量值图表显示的时间范围，请在图表顶部选择 1 小时、24 小时或者 7 天。

    ![监视器显示期间][监视器显示期间]

    在仪表板度量值图表中，显示度量值的方式是不同的。将显示一组标准度量值，并且可通过选择度量值标题来添加或删除度量值。

### 在仪表板上自定义度量值图表

1.  打开云服务的仪表板。

2.  从图表中添加或删除度量值：

    -   若要显示新的度量值，请在图表标题中选择该度量值对应的复选框。在收缩视图中，单击“*n* 个度量值”旁的向下箭头将显示图表标题区域无法显示的度量值。

    -   要删除显示在图表上的某个度量值，请清除其标题旁的复选框。

3.  在“相对”和“绝对”视图之间切换。

4.  选择要显示 1 小时、24 小时还是 7 天的数据。

## <span id="accessverbose"></span></a>如何：在管理门户外部访问详细监视数据

详细监视数据存储在你为每个角色指定的存储帐户中的相应表中。对于每个云服务部署，将为角色创建 6 个表。即，将为每 5 分钟、1 小时和 12 小时的数据创建 2 个表。其中一个表存储角色级别的汇总数据；其他表存储角色实例的汇总数据。

表名称采用以下格式：

    WAD*deploymentID*PT*aggregation_interval*[R|RI]Table

其中：

-   *DeploymentID* 是分配给云服务部署的 GUID

-   *aggregation\_interval* = 5 分钟、1 小时或 12 小时

-   R = 角色级别的汇总

-   RI = 角色实例的汇总

例如，下表将存储在 1 小时时间间隔内汇总的详细监视数据。

    WAD8b7c4233802442b494d0cc9eb9d8dd9fPT1HRTable (hourly aggregations for the role)

    WAD8b7c4233802442b494d0cc9eb9d8dd9fPT1HRITable (hourly aggregations for role instances)

  [免责声明]: ../includes/disclaimer.md
  [概念]: #concepts
  [如何：为云服务配置监视]: #verbose
  [如何：接收云服务度量值的警报]: #receivealerts
  [如何：向度量值表中添加度量值]: #addmetrics
  [如何：自定义度量值图表]: #customizechart
  [如何：在管理门户外部访问详细监视数据]: #accessverbose
  [使用 Azure 诊断收集日志记录数据]: http://msdn.microsoft.com/zh-cn/library/gg433048.aspx
  [在 Azure 应用程序中创建和使用性能计数器]: http://msdn.microsoft.com/zh-cn/library/hh411542.aspx
  [如何创建存储帐户]: /zh-cn/manage/services/storage/how-to-create-a-storage-account/
  [配置 Azure 诊断]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn186185.aspx
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [如何管理云服务]: http://www.windowsazure.com/zh-cn/manage/services/cloud-services/how-to-manage-a-cloud-service/
  [监视选项]: ./media/cloud-services-how-to-monitor/CloudServices_MonitoringOptions.png
  [如何：在 Azure 中接收警报通知和管理警报规则]: http://go.microsoft.com/fwlink/?LinkId=309356
  [管理门户]: http://manage.windowsazure.cn/
  [详细监视屏幕]: ./media/cloud-services-how-to-monitor/CloudServices_DefaultVerboseDisplay.png
  [添加度量值]: ./media/cloud-services-how-to-monitor/CloudServices_AddMetrics.png
  [添加度量值搜索]: ./media/cloud-services-how-to-monitor/CloudServices_AddMetrics_Search.png
  [监视度量值]: ./media/cloud-services-how-to-monitor/CloudServices_Monitor_UpdatedMetrics.png
  [相对或绝对]: ./media/cloud-services-how-to-monitor/CloudServices_Monitor_RelativeAbsolute.png
  [监视器显示期间]: ./media/cloud-services-how-to-monitor/CloudServices_Monitor_DisplayPeriod.png
