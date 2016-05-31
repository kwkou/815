<properties 
	pageTitle="使用流分析构建 IOT 解决方案 | Azure" 
	description="使用收费站方案了解流分析 iot 解决方案的入门教程"
	keywords=""
	documentationCenter=""
	services="stream-analytics"
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"
/>

<tags 
	ms.service="stream-analytics" 
	ms.date="05/03/2016" 
	wacn.date="05/30/2016"
/>

# 使用流分析构建 IOT 解决方案

## 介绍

在本教程中，你将学习如何使用 Azure 流分析来实时获得数据的深入见解。Azure 流处理服务可让开发人员通过结合数据流（例如点击流、日志、设备生成的事件及历史记录或引用数据）调整动态数据的空间，以快速轻松获取深入的业务见解。由 Azure 托管的 Azure 流分析是可完全管理的实时流计算服务，它提供内置冗余、低延迟及可缩放性，可让你在几分钟之内就立刻上手。

完成本教程之后，你将能够：

-   熟悉 Azure 流分析门户。
-   配置和部署流式处理作业。
-   使用流分析查询语言来表达真实世界的问题并解决这些问题。
-   自信地使用 Azure 流分析为客户开发流解决方案。
-   使用监视和日志记录体验来排解问题。

## 先决条件

若要完成本教程，需要满足以下先决条件。

-   最新的 [Azure PowerShell](/documentation/articles/powershell-install-configure)
-   Visual Studio 2015 或免费的 [Visual Studio Community](https://www.visualstudio.com/products/visual-studio-community-vs.aspx)
-   [Azure 订阅](/pricing/1rmb-trial/)
-   计算机上的管理员权限
-   从 Microsoft 下载中心下载 [TollApp.zip](http://download.microsoft.com/download/D/4/A/D4A3C379-65E8-494F-A8C5-79303FD43B0A/TollApp.zip)。
-   可选：[GitHub](https://github.com/streamanalytics/samples/tree/master/TollApp) 中 TollApp 事件生成器的源代码

## 方案简介 -“你好，收费站！”


收费站是常见的景象，我们在世界各地的许多高速公路、桥梁及隧道上都将碰到收费站。每个收费站都有好几个收费亭，它们可能是手动的（也就是必须停车来向收费员付通行费），也可能是自动的（这代表收费亭顶部有个传感器，当车辆经过收费亭时，传感器将扫描贴在车辆挡风玻璃上的 RFID 卡）。我们可以轻松地将车辆通过这些收费站的情况想象成能够执行许多有趣操作的事件流。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image1.jpg)

## 传入的数据

我们将使用两个数据流（分别由安装在收费站入口和出口的传感器生成），以及一个拥有汽车注册数据的静态查询数据集。

### 入口数据流

入口数据流包含汽车进入收费站的相关信息。
  
  
| 收费站 ID | EntryTime | LicensePlate | 状态 | 制造商 | 型号 | 汽车类型 | 车重 | 收费站 | 标记 |
|---------|-------------------------|--------------|-------|--------|---------|--------------|----------------|------|-----------|
| 1 | 2014-09-10 12:01:00.000 | JNB 7001 | NY | Honda | CRV | 1 | 0 | 7 | |
| 1 | 2014-09-10 12:02:00.000 | YXZ 1001 | NY | Toyota | Camry | 1 | 0 | 4 | 123456789 |
| 3 | 2014-09-10 12:02:00.000 | ABC 1004 | CT | Ford | Taurus | 1 | 0 | 5 | 456789123 |
| 2 | 2014-09-10 12:03:00.000 | XYZ 1003 | CT | Toyota | Corolla | 1 | 0 | 4 | |
| 1 | 2014-09-10 12:03:00.000 | BNJ 1007 | NY | Honda | CRV | 1 | 0 | 5 | 789123456 |
| 2 | 2014-09-10 12:05:00.000 | CDE 1007 | NJ | Toyota | 4x4 | 1 | 0 | 6 | 321987654 |
  

下面是每个列的简短说明：
  
  
| TollID | 用于唯一标识收费亭的收费亭 ID |
|--------------|----------------------------------------------------------------|
| EntryTime | 汽车进入收费亭的日期和时间（世界协调时） |
| LicensePlate | 汽车的牌照号码 |
| 状态 | 美国的某个州 |
| 制造商 | 汽车制造商 |
| 型号 | 汽车型号 |
| VehicleType | 1 代表客车，2 代表商用车 |
| WeightType | 汽车的重量，单位为吨；0 代表客车 |
| 收费站 | 通行费，单位为美元 |
| 标记 | 汽车上可用于自动付费的 e-Tag，空白代表手动付费 |


### 出口数据流

出口数据流包含汽车离开收费站的相关信息。
  
  
| **TollId** | **ExitTime** | **LicensePlate** |
|------------|------------------------------|------------------|
| 1 | 2014-09-10T12:03:00.0000000Z | JNB 7001 |
| 1 | 2014-09-10T12:03:00.0000000Z | YXZ 1001 |
| 3 | 2014-09-10T12:04:00.0000000Z | ABC 1004 |
| 2 | 2014-09-10T12:07:00.0000000Z | XYZ 1003 |
| 1 | 2014-09-10T12:08:00.0000000Z | BNJ 1007 |
| 2 | 2014-09-10T12:07:00.0000000Z | CDE 1007 |

下面是每个列的简短说明：
  
  
| 列 | 说明 |
|--------------|-----------------------------------------------------------------|
| TollID | 用于唯一标识收费亭的收费亭 ID |
| ExitTime | 汽车离开收费亭的日期和时间（世界协调时） |
| LicensePlate | 汽车的牌照号码 |

### 商用车注册数据

我们将使用商用车注册数据库的静态快照。
  
  
| LicensePlate | RegistrationId | Expired |
|--------------|----------------|---------|
| SVT 6023 | 285429838 | 1 |
| XLZ 3463 | 362715656 | 0 |
| BAC 1005 | 876133137 | 1 |
| RIV 8632 | 992711956 | 0 |
| SNY 7188 | 592133890 | 0 |
| ELH 9896 | 678427724 | 1 |                      

下面是每个列的简短说明：
  
  
| 列 | 说明 |
|--------------|-----------------------------------------------------------------|
| LicensePlate | 汽车的牌照号码 |
| RegistrationId | RegistrationId |
| Expired | 0 代表汽车注册仍有效，1 代表汽车注册已过期 |


## 设置 Azure 流分析的环境

若要完成本教程，需要 Azure 订阅。Microsoft 提供免费试用版 Azure 服务，如下所述。

如果你没有 Azure 帐户，可以通过以下链接请求免费试用版：<http://azure.microsoft.com/pricing/free-trial/>。

注意：若要注册免费试用版，必须有可接收短信的移动设备和有效的信用卡。

请务必在此练习的结尾，根据“清理 Azure 帐户”部分中的步骤操作，以便充分利用 $200 美元的免费 Azure 信用额度。

## 预配本教程所需的 Azure 资源

本教程需要 2 个 Azure 事件中心来接收“入口”和“出口”数据流。我们将使用 Azure SQL 数据库来输出流分析作业的结果。我们还将使用 Azure 存储空间来存储有关汽车注册的参考数据。

GitHub 上 TollApp 文件夹中的 Setup.ps1 脚本可用于创建所有必要的资源。为了节省时间，我们建议运行此脚本。如果想要进一步了解如何在 Azure 门户中配置这些资源，请参阅附录“在 Azure 门户中配置教程资源”。

下载并保存 [TollApp](http://download.microsoft.com/download/D/4/A/D4A3C379-65E8-494F-A8C5-79303FD43B0A/TollApp.zip) 支持文件夹和文件。

**以管理员身份**打开“Azure PowerShell”窗口。如果你没有 Azure PowerShell，请根据 [Install and configure Azure PowerShell（安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure)中的说明安装 Azure PowerShell。

Windows 会自动阻止从 Internet 下载的 ps1、dll 和 exe 文件。因此我们需要在运行脚本之前设置执行策略。确保以管理员身份运行 Azure PowerShell 窗口。运行“Set-ExecutionPolicy unrestricted”。出现提示时按“Y”键。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image2.png)

运行 Get-ExecutionPolicy 以确保命令正常工作。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image3.png)

转到包含脚本和生成器应用程序的目录。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image4.png)

键入“.\\Setup.ps1”以设置 Azure 帐户，创建并配置所有所需的资源，然后开始生成事件。脚本将随机挑选一个区域来创建资源。如果想要显式指定区域，可以传递 -location 参数，如以下示例所示：

**.\\Setup.ps1 -location “Central US”**

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image5.png)

脚本将打开 Azure 的“登录”页。输入你的帐户凭据。

请注意，如果帐户有权访问多个订阅，系统将请求你输入用于本教程的订阅名称。

脚本可能需要几分钟的时间来运行。完成后，输出应类似于下面的屏幕截图。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image6.PNG)

你还将看到类似于以下屏幕截图的另一个窗口。此应用程序将事件发送到事件中心，并需要该应用程序来运行教程练习。因此，在完成本教程之前，请不要停止此应用程序，或者关闭此窗口。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image7.png)

现在，你应该能够在 Azure 管理门户中看到已创建的所有资源。转到 <https://manage.windowsazure.com>，并使用你的帐户凭据登录。

### 事件中心

单击 Azure 管理门户左侧的“服务总线”菜单项，以查看上一部分中的脚本创建的事件中心。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image8.png)

你将在自己的订阅中看到所有可用命名空间。单击开头为“TollData”的项。（在本例中为 TollData4637388511）。单击“事件中心”选项卡。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image9.png)

你将在此命名空间中看到 2 个已创建的事件中心，分别名为 *entry* 和 *exit*。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image10.png)

### Azure 存储容器

单击 Azure 管理门户左侧的“存储”菜单项，以查看在实验中使用的存储容器。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image11.png)

单击开头为“tolldata”的项。（在本例中为 tolldata4637388511）。打开“容器”选项卡以查看创建的容器。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image12.png)

单击“tolldata”容器查看已上载的、包含汽车注册数据的 JSON 文件。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image13.png)

### Azure SQL 数据库

单击 Azure 管理门户左侧的“SQL 数据库”菜单项，以查看将在实验中使用的 Azure SQL 数据库。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image14.png)

单击“TollDataDB”

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image15.png)

复制服务器名称，但不要复制端口号（例如 服务器名称.database.windows.net）

## 从 Visual Studio 连接到数据库

我们将使用 Visual Studio 来访问输出数据库中的查询结果。

从 Visual Studio 连接到 Azure 数据库（目标）：

1) 打开 Visual Studio，单击“工具”，然后单击“连接到数据库...”菜单项。

2) 出现提示时，选择“Microsoft SQL Server”作为数据源

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image16.png)

3) 在“服务器名称”字段中，粘贴在上一部分从 Azure 门户复制的 SQL Server 名称（例如，服务器名称.database.windows.net）

4) 在“身份验证”字段中，选择“SQL Server 身份验证”

5) 在“登录名”中输入“tolladmin”，在“登录密码”中输入“123toll!”

6) 选择“TollDataDB”作为数据库

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image17.jpg)
    
7) 单击“确定”。

8) 打开“服务器资源管理器”

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image18.png)
  
9) 在 TollDataDB 数据库中可以看到 4 个已创建的表。
  
![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image19.jpg)
  
## 事件生成器 - TollApp 示例项目

PowerShell 脚本自动使用 TollApp 示例应用程序来开始发送事件。你不需要执行任何附加步骤。

但是，如果你对实现的细节有兴趣，可以在 GitHub 中的 [samples/TollApp](https://github.com/streamanalytics/samples/tree/master/TollApp) 下面找到 TollApp 应用程序的源代码。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image20.png)

##创建流分析作业

请在 Azure 门户中打开“流分析”，然后单击页左下角的“新建”来创建新的分析作业。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image21.png)

单击“快速创建”。选择脚本所创建的其他资源所在的同一区域。

对于“区域监视存储帐户”设置，请选择“创建新存储帐户”，并为它指定任何唯一名称。Azure 流分析将使用此帐户来存储将来所有作业的监视信息。

单击页面底部的“创建流分析作业”。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image22.png)

## 定义输入源

在门户中单击已创建的分析作业。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image23.jpg)

打开“输入”选项卡以定义源数据。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image24.jpg)

单击“添加输入”

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image25.png)

在第一页选择“数据流”

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image26.png)

在向导的第二页选择“事件中心”。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image27.png)

输入“EntryStream”作为输入别名。

单击“事件中心”下拉列表，然后选择开头为“TollData”的项（例如 TollData9518658221）。

选择“entry”作为事件中心的名称，选择“all”作为事件中心策略名称。

设置看起来类似于：

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image28.png)

转到下一页。选择“JSON”值和“UTF8”编码。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image29.png)

单击对话框底部的“确定”完成向导。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image30.jpg)

需要按照相同的步骤顺序为包含 Exit 事件的流创建第二个事件中心输入。确保在第 3 页输入以下屏幕截图中的值。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image31.png)

现在你已定义两个输入流：

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image32.jpg)

接下来，我们将使用汽车注册数据为 Blob 文件添加“引用”数据输入。

单击“添加输入”。选择“引用数据”

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image33.png)

在下一个页选择开头为“tolldata”的存储帐户。容器名称应为“tolldata”，“模式路径”下面的 Blob 名称应为“registration.json”。此文件名区分大小写，且应该全为小写。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image34.png)

在下一页选择下图中所示的值，然后单击“确定”完成向导。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image35.png)

现在，已定义所有输入。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image36.jpg)

## 定义输出

转到“输出”选项卡，然后单击“添加输出”。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image37.jpg)

选择“SQL 数据库”。

选择在“从 Visual Studio 连接到数据库”部分中使用的服务器名称。数据库名称应为 TollDataDB。

输入“tolladmin”作为用户名，输入“123toll!”作为密码。表名称应设置为“TollDataRefJoin”。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image38.jpg)

## Azure 流分析查询

“查询”选项卡包含针对传入数据进行转换的 SQL 查询。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image39.jpg)

在本教程中，我们将尝试回答几个通行费数据相关的业务问题，并构造可在 Azure 流分析中使用的流分析查询来提供相关的答案。

在开始我们的第一个 Azure 流分析作业之前，让我们来探讨几种方案和查询语法。

## Azure 流分析查询语言简介
-----------------------------------------------------

假设我们需要统计进入某个收费亭的汽车数目。由于这是连续的事件流，我们必须定义一个“时段”。因此需要将问题修改为“每 3 分钟进入某个收费亭的汽车数目”。这通常称为轮转计数。

我们看看能回答此问题的 Azure 流分析查询：

    SELECT TollId, System.Timestamp AS WindowEnd, COUNT(*) AS Count
    FROM EntryStream TIMESTAMP BY EntryTime
    GROUP BY TUMBLINGWINDOW(minute, 3), TollId

如上所示，Azure 流分析使用类似于 SQL 的查询语言再加上其他几个扩展功能，来启用查询在时间方面的指定功能。

有关详细信息，请参阅 MSDN 中的[时间管理](https://msdn.microsoft.com/zh-cn/library/azure/mt582045.aspx)，以及查询中所用的[窗口化](https://msdn.microsoft.com/zh-cn/library/azure/dn835019.aspx)构造。

## 测试 Azure 流分析查询

编写第一个 Azure 流分析查询后，可以使用位于 TollApp 文件夹中以下路径的示例数据文件来测试查询：

**..\\TollApp\\TollApp\\Data**

此文件夹包含以下文件：

1.  Entry.json

2.  Exit.json

3.  Registration.json

## 问题 1 - 进入某个收费亭的汽车数目

打开 Azure 管理门户，并导航到创建的 Azure 流分析作业。打开“查询”选项卡，并复制/粘贴前一部分中的查询。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image40.png)

若要针对示例数据验证此查询，请单击“测试”按钮。在随后打开的对话框中导航到 Entry.json，此文件包含来自 EntryTime 事件流的示例数据。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image41.png)

验证查询的输出是否与以下预期结果相同：

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image42.jpg)

## 问题 2 - 报告每辆车通过收费亭的总时间

我们想要知道汽车经过收费站所需的平均时间，以评估效率和客户体验。

为此，需要联接包含 EntryTime 的流与包含 ExitTime 的流。我们将联接 TollId 和 LicencePlate 列中的流。JOIN 运算符要求指定一个临时弹性空间，以描述联接事件之间可接受的时间差。我们将使用 DATEDIFF 函数来指定事件之间的时间差不能超过 15 分钟。我们还将 DATEDIFF 函数应用到 Exit 及 Entry 时间，以计算汽车经过收费站的实际时间。请注意相比 JOIN 条件，在 SELECT 语句中使用 DATEDIFF 的差异。

    SELECT EntryStream.TollId, EntryStream.EntryTime, ExitStream.ExitTime, EntryStream.LicensePlate, DATEDIFF (minute , EntryStream.EntryTime, ExitStream.ExitTime) AS DurationInMinutes
    FROM EntryStream TIMESTAMP BY EntryTime
    JOIN ExitStream TIMESTAMP BY ExitTime
    ON (EntryStream.TollId= ExitStream.TollId AND EntryStream.LicensePlate = ExitStream.LicensePlate)
    AND DATEDIFF (minute, EntryStream, ExitStream ) BETWEEN 0 AND 15

如要测试此查询，请更新作业的“查询”选项卡上的查询：

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image43.jpg)

单击“测试”并指定 EntryTime 和 ExitTime 的示例输入文件。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image44.png)

单击复选框测试查询并查看输出：

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image45.png)

## 问题 3 - 报告注册已过期的所有商用车

Azure 流分析可以使用静态数据快照来与临时数据流联接。为了演示此功能，我们将使用以下示例问题。

如果某辆商用车已向收费公司注册，则可以直接通过收费亭，而不用停车接受检查。我们将使用商用车注册查找表来识别注册已过期的所有商用车。

    SELECT EntryStream.EntryTime, EntryStream.LicensePlate, EntryStream.TollId, Registration.RegistrationId
    FROM EntryStream TIMESTAMP BY EntryTime
    JOIN Registration
    ON EntryStream.LicensePlate = Registration.LicensePlate
    WHERE Registration.Expired = '1'

请注意，若要测试使用引用数据的查询，需要定义引用数据的输入源，我们已在步骤 5 中完成此操作。

若要测试此查询，请将查询粘贴到“查询”选项卡，单击“测试”，然后指定 2 个输入源：

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image46.png)

查看查询的输出：

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image47.png)

## 启动流分析作业


编写第一个 Azure 流分析查询后，便可以完成配置并启动作业。保存问题 3 中的查询，这将生成与输出表 **TollDataRefJoin** 架构匹配的输出。

导航到作业仪表板并单击“启动”。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image48.jpg)

在随后出现的对话框中，将“开始输出”时间更改为“自定义时间”。编辑“小时”，并将它设置为前 1 小时。这可以确保从教程开头生成事件的那一刻，开始处理事件中心的所有事件。现在，请单击复选标记来启动作业。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image49.png)

启动作业可能需要几分钟时间。可以在流分析的顶级页中查看状态。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image50.jpg)

## 在 Visual Studio 中检查结果

打开 Visual Studio 服务器资源管理器，然后右键单击“TollDataRefJoin”表。选择“显示表数据”以查看作业的输出。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image51.jpg)

## 扩展 Azure 流分析作业

Azure 流分析能够弹性缩放，并且能够处理较大的数据负载。Azure 流分析查询可以使用 **PARTITION BY** 子句来告诉系统此步骤将会扩展。PartitionId 是系统添加的特殊列，它与输入（事件中心）的分区 ID 匹配

    SELECT TollId, System.Timestamp AS WindowEnd, COUNT(*)AS Count
    FROM EntryStream TIMESTAMP BY EntryTime PARTITION BY PartitionId
    GROUP BY TUMBLINGWINDOW(minute,3), TollId, PartitionId    

停止当前作业，更新“查询”选项卡中的查询，然后打开“缩放”选项卡。

流式处理单元定义了作业能够接收的计算能力大小。

将滑块移到 6。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image52.jpg)

转到“输出”选项卡，然后将 SQL 表名称更改为“TollDataTumblingCountPartitioned”

现在，如果你启动作业，Azure 流分析可将工作分散到更多计算资源上，并实现更高的吞吐量。请注意，TollApp 应用程序还会发送已按 TollId 分区的事件。

## 监视

“监视”选项卡包含有关正在运行的作业的统计信息。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image53.png)

可以从“仪表板”选项卡访问操作日志。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image54.jpg)

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image55.png)

若要查看有关某个特定事件的更多信息，请选择该事件，然后单击“详细信息”按钮。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image56.png)

## 结束语

在本教程中，我们提供了 Azure 流分析服务的简介。其中演示了如何为流分析作业配置输入和输出。我们还使用了收费站数据方案来解释在数据空间将不断变化时所引发的常见问题类型，以及如何在 Azure 流分析中使用类似于 SQL 的简单查询来解决这些问题。我们介绍了用于处理临时数据的 SQL 扩展构造。我们演示了如何联接不同的数据流，以及如何使用静态引用数据来扩充数据流。我们解释了如何扩展查询以实现更高的吞吐量。

尽管本教程简介的涵盖范围相当大，但它绝对不是完整的说明。可以在[此处](/documentation/articles/stream-analytics-stream-analytics-query-patterns)找到更多使用 SAQL 语言的查询模式。
若要了解有关 Azure 流分析的详细信息，请参阅[联机文档](/documentation/services/stream-analytics/)。

## 清理 Azure 帐户

请从 Azure 门户停止流分析作业。

Setup.ps1 脚本将创建 2 个 Azure 事件中心，以及 Azure SQL 数据库服务器。以下说明将帮助你在本教程结束时清理资源。

在 PowerShell 窗口中键入“.\\Cleanup.ps1”，启动用于删除本教程所用资源的脚本。

请注意，资源按名称标识。在确认删除之前，请确保仔细检查每个项。

![](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image57.png)

<!---HONumber=Mooncake_0523_2016-->