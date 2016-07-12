<properties 
 pageTitle="开始在管理门户中使用 Azure 计划程序 | Azure"
 description="" 
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags
 ms.service="scheduler"
 ms.date="03/09/2016"
 wacn.date="04/11/2016"/>

# Azure 管理门户中的 Azure 计划程序入门

在 Azure 计划程序中创建计划的作业很简单。在本教程中，你将了解如何创建作业。还将学习计划程序的监视和管理功能。

## 创建作业

1.  登录到[管理门户](https://manage.windowsazure.cn)。  

2.  单击“应用程序服务”>“新建”>“计划程序”，然后单击“自定义创建”。<br /><br />![][2]

3.  在“作业集合”中的“作业集合”下拉列表下，选择现有作业集合的名称。如果你要将作业添加到的作业集合不存在，请选择“新建”并输入一个名称来标识新的作业集合。<br /><br /> ![][3]

4.  在“区域”中，为作业集合选择地理区域。

5.  单击箭头键以创建作业集合并转到下一阶段 – 创建作业。

6.  让我们来创建一个作业，这只需要使用 GET 请求访问 http://www.microsoft.com/。在“作业操作”屏幕中，为请求的表单字段定义以下值：

    1.  **名称：**` getmicrosoft`  

    2.  **操作类型：**` HTTP`

    3.  **方法：**` GET`

    4.  **URI：**` http://www.microsoft.com`

   	![][4]

7.  创建作业后，定义计划。该作业可定义为一次性作业，但是我们选择了重复执行的计划。本教程中的一些屏幕快照显示一分钟重复一次（仅用于演示目的），但是我们选择了 12 小时重复一次。

    1.  **执行间隔：**` 12 Hours`  

    2.  **开始时间：**` Now`

    3.  **结束时间：**` Select date 2 days after current day and any time`

   	![][5]

8.  单击**“确定”**。  
    创建作业和作业集合可能需要一段时间。若要检查状态，可以监视门户底部的通知。

   	![][6]

   	在创建作业和作业集合后，将显示一条消息，告知你已成功创建作业或作业集合。该作业将在“计划程序”部分的“作业”部分中列出，作业集合将在“作业集合”部分中列出。若要在作业中配置其他高级设置，请参阅以下“配置作业”部分。

   	![][7]

## 管理和监视作业集合和作业

创建作业集合后，该作业集合将显示在主计划程序管理屏幕中。

![][8]

单击某个作业集合将打开具有以下选项的新窗口：

1.  仪表板  

2.  缩放

3.  历史记录

4.  作业

以下主题更为详细地介绍了这些选项卡。

### 仪表板

单击作业集合名称时，显示“仪表板”选项卡。“仪表板”将显示以下信息：

![][9]

#### 作业使用概览和执行使用概览

一个显示固定度量值列表的表和图表系列。这些度量值提供有关你的作业集合的运行状况的实时值，其中包括：

1.  当前作业  

2.  已完成的作业

3.  出错的作业

4.  已启用的作业

5.  已禁用的作业

6.  作业执行次数

#### 速览

一个显示状态和度量值设置的固定列表的表。这些度量值提供有关你的作业集合的状态和相关设置的实时值，其中包括：

1.  状态  

2.  区域

3.  错误数

4.  错误发生次数

5.  URI

### 缩放

在“缩放”选项卡中，你可以更改设置和计划程序使用的服务层。

![][10]

#### 常规

这将会显示你执行的是“免费”还是“标准”计划。

#### 配额

Azure 计划程序基于几个条件实施配额。本节列出了配额阈值，你可以更改它们。默认情况下，配置了一组配额。这些配额设置的限制值由你的计划决定，更改计划可能影响定价。可以更改配额以缩放计划程序。选项包括：

1.  最大作业数  

2.  最大频率

3.  最大间隔

### 历史记录

“历史记录”选项卡显示所选作业的以下信息：

![][11]

#### “历史记录”表

一个表，它显示在所选作业的系统中每次作业执行的所选度量值。这些度量值提供有关你的计划程序的运行状况的实时值。

#### 可用度量值

提供以下性能计数器/度量值：

1.  状态  

2.  详细信息

3.  重试次数

4.  执行次数（第一次、第二次、第三次等）

5.  执行的时间戳

你可以单击“查看历史记录详细信息”以查看每次执行的响应情况。此对话框还允许你将响应复制到剪贴板。

![][12]

### 作业

“作业”选项卡显示以下信息来监视作业的执行历史记录：

![][13]

#### “作业”表

一个表，它显示在系统中每个作业的所选度量值。这些度量值提供有关你的计划程序的运行状况的实时值。

#### 禁用、启用或删除作业

单击某个作业名称可提供启用、禁用或删除作业的选项。删除的作业可能无法恢复。

#### 可用度量值

提供以下计数器和度量值：

1.  名称  

2.  上次运行时间

3.  下次运行时间

4.  状态

5.  频率

6.  失败数

7.  错误数

8.  执行次数

9.  操作类型

### 配置作业

在“作业”屏幕中单击某个作业可以配置该作业。这样，便可以配置快速创建向导中未提供的其他高级设置。若要配置某个作业，请在“作业”屏幕中单击该作业名称旁边的右箭头。

你可以在作业配置页中更新作业设置。下面显示了 HTTP 和 HTTPS 作业的作业配置页。对于 HTTP 和 HTTPS 作业操作类型，可以将方法更改为允许的任何 HTTP 谓词。你还可以添加、删除或更改标头及基本身份验证信息。

![][14]

下面显示了存储队列作业的作业配置页。对于存储队列操作类型，可以更改存储帐户、队列名称、SAS 令牌和正文。“计划”部分（下图中未显示）与 HTTP/HTTPS 作业操作类型的“计划”部分相同。

![][15]

最后，对于所有操作类型，你可以更改计划本身及其重复行为。可以更改开始日期与时间、重复计划以及结束日期与时间（如果该作业是重复进行的。） 进行任何更改后，可单击“保存”以保存更改，或单击“放弃”以放弃更改。

## 另请参阅

 [计划程序是什么？](/documentation/articles/scheduler-intro/)

 [计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms/)

 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing/)

 [如何使用 Azure 计划程序生成复杂的计划和高级重复执行](/documentation/articles/scheduler-advanced-complexity/)

 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)

 [计划程序 PowerShell Cmdlet 参考](/documentation/articles/scheduler-powershell-reference/)

 [计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability/)

 [计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors/)

 [计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication/)



[1]: ./media/scheduler-get-started-portal/scheduler-get-started-portal001.png
[2]: ./media/scheduler-get-started-portal/scheduler-get-started-portal002.png
[3]: ./media/scheduler-get-started-portal/scheduler-get-started-portal003.png
[4]: ./media/scheduler-get-started-portal/scheduler-get-started-portal004.png
[5]: ./media/scheduler-get-started-portal/scheduler-get-started-portal005.png
[6]: ./media/scheduler-get-started-portal/scheduler-get-started-portal006.png
[7]: ./media/scheduler-get-started-portal/scheduler-get-started-portal007.png
[8]: ./media/scheduler-get-started-portal/scheduler-get-started-portal008.png
[9]: ./media/scheduler-get-started-portal/scheduler-get-started-portal009.png
[10]: ./media/scheduler-get-started-portal/scheduler-get-started-portal010.png
[11]: ./media/scheduler-get-started-portal/scheduler-get-started-portal011.png
[12]: ./media/scheduler-get-started-portal/scheduler-get-started-portal012.png
[13]: ./media/scheduler-get-started-portal/scheduler-get-started-portal013.png
[14]: ./media/scheduler-get-started-portal/scheduler-get-started-portal014.png
[15]: ./media/scheduler-get-started-portal/scheduler-get-started-portal015.png
<!---HONumber=Mooncake_0405_2016-->