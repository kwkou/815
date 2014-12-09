<properties linkid="develop-net-tutorials-multi-tier-web-site-1-overview" pageTitle="Azure 云服务教程：ASP.NET MVC Web 角色、辅助角色、Azure 存储表、队列和 Blob" metaKeywords="Azure tutorial, Azure storage tutorial, Azure multi-tier tutorial, MVC Web Role tutorial, Azure worker role tutorial, Azure blobs tutorial, Azure tables tutorial, Azure queues tutorial" description="了解如何使用 ASP.NET MVC 和 Azure 创建多层应用程序。该应用程序运行在云服务中，使用 Web 角色和辅助角色，并使用 Azure 存储表、队列和 Blob。" metaCanonical="" services="cloud-services,storage" documentationCenter=".NET" title="Azure 云服务教程：ASP.NET MVC Web 角色、辅助角色、Azure 存储表、队列和 Blob" authors="tdykstra,riande" solutions="" manager="wpickett" editor="mollybos" />

# Azure 云服务教程：ASP.NET MVC Web 角色、辅助角色、Azure 存储表、队列和 Blob - 第 1 部分（共 5 部分）

本教程系列将演示如何创建和部署可在 Azure 云服务中运行并使用 Azure 存储表、队列和 Blob 的多层 ASP.NET MVC Web 应用程序。你可以从 MSDN 代码库[下载已完成的应用程序][下载已完成的应用程序]。下面是显示应用程序部件如何进行交互的关系图：

![电子邮件处理][电子邮件处理]

在本教程系列中，你将学习以下内容：

-   如何通过安装 Azure SDK 来让你的计算机可以进行 Azure 开发。
-   如何使用一个 ASP.NET MVC Web 角色和两个辅助角色来创建 Visual Studio 云项目。
-   如何将云项目发布到 Azure 云服务。
-   如何根据需要将 MVC 项目发布到 Azure 网站，并且仍使用云服务中的辅助角色。
-   如何使用 Azure 队列存储服务在层之间或辅助角色之间通信。
-   如何将 Azure 表存储服务用作高度可伸缩的数据存储，以便存储结构化非关系型数据。
-   如何使用 Azure Blob 服务在云中存储文件。
-   如何使用 Visual Studio 服务器资源管理器查看和编辑 Azure 表、队列和 Blob。
-   如何使用 SendGrid 发送电子邮件。
-   如何配置跟踪和查看跟踪数据。
-   如何通过增加辅助角色实例的数目来扩展应用程序。

## <a name="toc"></a>系列教程

下面是带内容摘要的教程的列表：

1.  **Azure 电子邮件服务应用程序简介**（本教程）。深入了解应用程序及其体系结构。如果你只是想要了解如何进行部署，或者只是想要查看代码，则可以跳过此内容。你可以回过头来查看此内容，以加深对体系结构的理解。
2.  [配置和部署 Azure 电子邮件服务应用程序][配置和部署 Azure 电子邮件服务应用程序]。如何下载示例应用程序、如何对其进行配置、如何对其进行本地测试，以及如何将其部署到云中并进行测试。
3.  [构建 Azure 电子邮件服务应用程序的 Web 角色][构建 Azure 电子邮件服务应用程序的 Web 角色]。如何生成应用程序的 MVC 组件并对其进行本地测试。
4.  [构建 Azure 电子邮件服务应用程序的辅助角色 A（电子邮件计划程序）][构建 Azure 电子邮件服务应用程序的辅助角色 A（电子邮件计划程序）]。如何生成创建队列工作项（用于发送电子邮件）所需的后端组件并对其进行本地测试。
5.  [构建 Azure 电子邮件服务应用程序的辅助角色 B（电子邮件发件人）][构建 Azure 电子邮件服务应用程序的辅助角色 B（电子邮件发件人）]。如何生成处理队列工作项（用于发送电子邮件）所需的后端组件并对其进行本地测试。

## 本教程的章节

-   [先决条件][先决条件]
-   [为什么使用电子邮件列表][为什么使用电子邮件列表]
-   [前端概述][前端概述]
-   [后端概述][后端概述]
-   [Azure 表][Azure 表]
-   [Azure 队列][Azure 队列]
-   [数据图][数据图]
-   [Azure Blob][Azure Blob]
-   [Azure 云服务与 Azure 网站][Azure 云服务与 Azure 网站]
-   [成本][成本]
-   [身份验证和授权][身份验证和授权]
-   [后续步骤][后续步骤]

## 先决条件

这些教程中的说明适用于以下产品：

-   Visual Studio 2012
-   Visual Studio 2012 Express for Web
-   Visual Studio 2010
-   Visual Web Developer Express 2010。

> [WACOM.NOTE] 本教程编写完以后，Visual Studio 2013 发布，而 Azure 管理门户和 SDK 则进行了更新。如果你使用的是 Visual Studio 2013 和最新版 SDK，则必须执行不同的操作，在这种情况下，我们添加了类似这样的注释来提醒你。这些注释是在 2014 年 3 月编写的，经过修改的操作过程已使用 SDK 版本 2.3 进行测试。本教程的主要文本和屏幕快照将在以后进行更新。
> [TechNet 电子书库][TechNet 电子书库]提供免费的电子书，其中包含本教程的内容，不含 Visual Studio 2013 的最新更新。

## <a name="whyanemaillistapp"></a><span class="short-header">为什么使用本应用程序</span>为什么使用电子邮件列表服务应用程序

我们之所以为本示例应用程序选择电子邮件列表服务，是因为它是那种需要弹性和可伸缩性的应用程序，这两种特性使得它尤其适合 Azure。

### 弹性

如果某个服务器在向大型列表发送电子邮件时发生故障，你需要能够快速方便地启动新的服务器，而且需要应用程序从中断的地方开始，既不漏发电子邮件，也不重复发送电子邮件。如果某个 Azure 云服务 Web 角色实例或辅助角色实例（虚拟机）发生故障，则会自动将其替换。此外，还可以使用 Azure 存储队列和表来实现服务器之间的通信，因此即使发生故障也不会造成所工作的项目丢失。

### 可缩放

电子邮件服务还必须能够应对工作负载峰值，因为你有时候是向小型列表发送电子邮件，有时候是向特大型列表发送电子邮件。在许多托管环境中，你必须购买并维护足够的硬件来应对工作负载峰值，而且要支付在全部时间使用所有这些容量的费用，虽然你可能只需要在 5% 的时间用到它。而有了 Azure，你就只需为实际需要使用的那部分计算能力付费，想使用多长时间就使用多长时间。在需要扩容来发送大量邮件时，你只需更改配置设置来增加用于处理工作负载的服务器的数量即可，而更改配置设置可通过编程方式来进行。例如，你可以对应用程序进行相应的配置，使得 Azure 能够在队列中的等待工作项目数超过某个数目时，自动启动额外的辅助角色实例来处理这些工作项目。

## <a name="frontend"></a>前端概述

你将构建的应用程序是电子邮件列表服务。应用程序的前端包括服务管理员用于管理电子邮件列表的网页。

![邮件列表索引页][邮件列表索引页]

![订户索引页][订户索引页]

此外还有一组页面，可供管理员用来创建发送到电子邮件列表的邮件。

![邮件索引页][邮件索引页]

![邮件创建页][邮件创建页]

服务的客户端是公司，这些公司可以让其客户注册客户端网站上的邮件列表。例如，管理员可以设置一个列表，用于 Contoso 大学历史系的公告。当某个对历史系公告感兴趣的学生单击 Contoso 大学网站上的链接时，Contoso 大学就会向 Azure 电子邮件服务应用程序发出 Web 服务调用。服务方法会将电子邮件发送给客户。该电子邮件包含一个超链接，当收件人单击该链接时，会显示一个欢迎客户访问历史系公告列表的页面。

![确认电子邮件][确认电子邮件]

![欢迎访问列表页面][欢迎访问列表页面]

由该服务发送的每封电子邮件（订阅确认电子邮件除外）都包含一个可用于取消订阅的超链接。如果收件人单击该链接，将会显示一个要求确认取消订阅意愿的网页。

![确认取消订阅页][确认取消订阅页]

如果收件人单击“确认”按钮，则会显示一个页面，确认已将收件人从列表中删除。

![取消订阅已确认页][取消订阅已确认页]

## <a name="backend"></a><span class="short-header">后端概述</span>后端概述

前端存储电子邮件列表以及要在 Azure 表中发送给这些列表的邮件。当管理员计划发送某封邮件时，就会向 `message` 表添加一个表行，其中包含计划的日期和其他数据，例如主题行。一个辅助角色会定期扫描 `message` 表，看其中是否存在需要发送的邮件（我们将该角色称为辅助角色 A）。

当辅助角色 A 找到一封需要发送的邮件时，它会执行以下任务：

-   获取目标电子邮件列表中的所有电子邮件地址。
-   将发送每封电子邮件所需的信息置于 `message` 表中。
-   为每封需要发送的电子邮件创建一个队列工作项。

另一个辅助角色（辅助角色 B）会轮询队列中是否存在工作项。当辅助角色 B 找到一个工作项时，它会通过发送电子邮件对该项进行处理，然后从队列中将其删除。下图显示了这些关系。

![电子邮件处理][电子邮件处理]

即使辅助角色 B 发生故障且必须重新启动，也不会出现电子邮件漏发的情况，因为电子邮件的队列工作项在该电子邮件真正发送出去之前不会被删除。后端也会进行表处理，防止在辅助角色 A 发生故障且必须重新启动的情况下发送多封电子邮件。在上面所说的情况下，一个给定的目标电子邮件地址可能会生成多个队列工作项。不过对于每个目标电子邮件地址，`message` 表中都会有一行用于跟踪该电子邮件是否已发送。可能是辅助角色 A 使用该行来避免创建第二个队列工作项，也可能是辅助角色 B 使用该行来避免发送第二封电子邮件，具体取决于重新启动和电子邮件处理的时间。

![阻止重复的电子邮件][阻止重复的电子邮件]

辅助角色 B 也会轮询订阅队列中是否存在工作项，这些工作项是由 Web API 服务方法放置在那里的，用于新的订阅。当辅助角色 B 找到一个工作项时，就会发送确认电子邮件。

![订阅队列邮件处理][订阅队列邮件处理]

## <a name="tables"></a><span class="short-header">表</span>Azure 表

Azure 电子邮件服务应用程序在 Azure 存储表中存储数据。Azure 表是一种 NoSQL 数据存储形式，不同于 [Azure SQL Database][Azure SQL Database] 这样的关系数据库。因此，当效率和可伸缩性比数据规范化和关系完整性更重要时，不妨选择这种存储形式。例如，在本应用程序中，一个辅助角色会在每次创建一个队列工作项时创建一个行，而另一个辅助角色会在每次发送一封电子邮件时检索并更新一个行，这在使用关系数据库的情况下，可能会造成性能瓶颈。此外，Azure 表比 Azure SQL 更廉价。有关 Azure 表的详细信息，请参阅[本系列最后一个教程][构建 Azure 电子邮件服务应用程序的辅助角色 B（电子邮件发件人）]结尾处列出的资源。

以下各节介绍 Azure 电子邮件服务应用程序所使用的 Azure 表的内容。如需一张显示这些表及其关系的图，请参阅本页后面的 [Azure 电子邮件服务数据图][数据图]。

### mailinglist 表

`mailinglist` 表存储有关邮件列表及邮件列表订户的信息。（Azure 表命名约定的最佳做法是全部使用小写字母。）管理员使用网页来创建和编辑邮件列表，而客户端和订户则使用一组网页和服务方法来订阅和取消订阅。

在 NoSQL 表中，不同的行可以有不同的架构，通常可以利用这种灵活性来使用一个表存储在关系数据库中需要多个表才能存储的数据。例如，在 SQL Database 中存储邮件列表数据时，你可能会使用三个表：一个 `mailinglist` 表，用于存储列表信息；一个 `subscriber` 表，用于存储订户信息；一个 `mailinglistsubscriber` 表，用于将邮件列表与订户关联起来（反过来也一样）。在本应用程序的 NoSQL 表中，所有这些功能都集中在一个名为 `mailinglist` 的表中。

在 Azure 表中，每一行都有一个*分区键* 和一个*行键*，用于唯一标识该行。分区键通过逻辑方式将表划分为各个分区。在分区中，行键用于唯一标识某个行。没有辅助索引，因此若要确保应用程序能够进行伸缩，必须将表设计为允许你在查询的 Where 子句中指定分区键和行键的值。

`mailinglist` 表的分区键是邮件列表的名称。

`mailinglist` 表的行键可能为：常量“mailinglist”或订户的电子邮件地址。行键为“mailinglist”的行包含邮件列表的相关信息。使用电子邮件地址作为行键的行包含邮件列表订户的相关信息。

换句话说，行键为“mailinglist”的行相当于关系数据库中的 `mailinglist` 表。行键等于电子邮件地址的行相当于关系数据库中的 `subscriber` 表和 `mailinglistsubscriber` 关联表。

通过这种方式实现一表多用可以提高性能。在关系数据库中，必须读取三个表，必须对三个行组排序并互相进行匹配，这都需要时间。而在这里，只需读取一个表，其行就会按分区键和行键顺序自动返回。

以下表格显示了包含邮件列表信息的行的属性（行键为“MailingList”）。

| 属性             | 数据类型 | 说明                                                                                                                                       |
|------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------|
| PartitionKey     | String   | ListName：邮件列表的唯一标识符，例如：contoso1。表的典型用途是检索特定邮件列表的所有信息，因此使用列表名称是一种有效的对表进行分区的方式。 |
| RowKey           | String   | 常量“mailinglist”。                                                                                                                        |
| 说明             | String   | 描述邮件列表，例如：“Contoso 大学历史系公告”。                                                                                             |
| FromEmailAddress | String   | 发送到该列表的电子邮件中的“发件人”电子邮件地址，例如：donotreply@contoso.edu。                                                             |

以下网格显示包含列表订户信息的行的属性（行键为电子邮件地址）。

| 属性           | 数据类型 | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|----------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PartitionKey   | String   | ListName：邮件列表的名称（唯一标识符），例如：contoso1。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RowKey         | String   | EmailAddress：订户电子邮件地址，例如：student1@contoso.edu。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| SubscriberGUID | String   | 将电子邮件地址添加到列表时生成。用于订阅和取消订阅链接中，因此难以使用他人的电子邮件地址进行订阅或取消订阅。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
                              针对订阅和取消订阅网页的某些查询仅指定 PartitionKey 和本属性。不使用 RowKey 查询某个分区会限制应用程序的可伸缩性，因为邮件列表越大，查询所耗用的时间也越长。若要改进可伸缩性，可以选择添加查找行，以便在 RowKey 属性中设置 SubscriberGUID。例如，对于每个电子邮件地址，可以使用一行将 RowKey 设置为“email:student1@domain.com”，并使用另一行将这同一个订户的 RowKey 设置为“guid:6f32b03b-90ed-41a9-b8ac-c1310c67b66a”。这实施起来很简单，因为在一个分区中对行执行原子批处理事务很容易进行编码。我们希望在下一版的示例应用程序中实现这一功能。有关详细信息，请参阅[实际应用：为 Azure 表存储设计可扩展分区策略][实际应用：为 Azure 表存储设计可扩展分区策略]  |
| 已验证         | Boolean  | 在一开始为新的订户创建该行时，值为 false。仅当新订户单击欢迎电子邮件中的“确认”超链接后，或者管理员将该值设置为 true 后，该值才会更改为 true。如果向列表发送了一封邮件，而其订户之一的“已验证”值为 false，则不会向该订户发送电子邮件。                                                                                                                                                                                                                                                                                                                                                                             |

以下列表显示了一个示例，说明了表中的数据看起来会是什么样子。

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
contoso1

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
mailinglist

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
说明

</th>
<td>
Contoso 大学历史系公告

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
FromEmailAddress

</th>
<td>
donotreply@contoso.edu

</td>
</tr>
</table>

------------------------------------------------------------------------

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
contoso1

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
student1@domain.com

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubscriberGUID

</th>
<td>
6f32b03b-90ed-41a9-b8ac-c1310c67b66a

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
已验证

</th>
<td>
是

</td>
</tr>
</table>

------------------------------------------------------------------------

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
contoso1

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
student2@domain.com

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubscriberGUID

</th>
<td>
01234567-90ed-41a9-b8ac-c1310c67b66a

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
已验证

</th>
<td>
false

</td>
</tr>
</table>

------------------------------------------------------------------------

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
fabrikam1

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
mailinglist

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
说明

</th>
<td>
Fabrikam 工程招聘广告

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
FromEmailAddress

</th>
<td>
donotreply@fabrikam.com

</td>
</tr>
</table>

------------------------------------------------------------------------

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
fabrikam1

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
applicant1@domain.com

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubscriberGUID

</th>
<td>
76543210-90ed-41a9-b8ac-c1310c67b66a

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
已验证

</th>
<td>
是

</td>
</tr>
</table>
### 邮件表

`message` 表存储计划要发送到邮件列表中的邮件的相关信息。管理员通过网页创建和编辑此表中的行，而辅助角色则通过网页将每封电子邮件的相关信息从辅助角色 A 传递给辅助角色 B。

邮件表的分区键是计划发送电子邮件的日期，以 yyyy-mm-dd 格式表示。这样可以针对查询对表进行优化，而查询通常也是针对该表执行的，用于选择 `ScheduledDate` 为当天或更早的行。不过，这不会造成潜在的性能瓶颈，因为 Azure 存储表的最大吞吐量为每个分区每秒 500 个实体。对于每封要发送的电子邮件，应用程序会写入一个 `message` 表行，读取一个行，然后删除一个行。因此，处理 1,000,000 封计划在一天内发送的电子邮件的最短可能时间为差不多两小时，不管添加多少辅助角色来处理增加的负载。

`message` 表的行键可能为：常量“message”加上邮件的唯一键，称为 `MessageRef`；或者 `MessageRef` 值加上订户的电子邮件地址。行键以“message”开头的行包含邮件的相关信息，例如要向其发送邮件的邮件列表，以及应何时发送邮件。以 `MessageRef` 和电子邮件地址作为行键的行包含向该电子邮件地址发送电子邮件所需的所有信息。

在关系数据库术语中，行键以“message”开头的行相当于 `message` 表。行键为 `MessageRef` 加电子邮件地址的行相当于一个联接查询视图，其中包含 `mailinglist`、`message` 和 `subscriber` 信息。

以下网格显示了包含邮件自身信息的 `message` 表行的属性。

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">属性</th>
<th align="left">数据类型</th>
<th align="left">说明</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">PartitionKey</td>
<td align="left">String</td>
<td align="left">计划发送邮件的日期，采用 yyyy-mm-dd 格式。</td>
</tr>
<tr class="even">
<td align="left">RowKey</td>
<td align="left">String</td>
<td align="left">与 <code data-inline="1">MessageRef</code> 值连接的常量“message”。<code data-inline="1">MessageRef</code> 是一个唯一值，是通过在创建行时从 <code data-inline="1">DateTime.Now</code> 获取 <code data-inline="1">Ticks</code> 值创建的。<br /><br />注意：使用 Ticks 时，应让高容量多线程多实例应用程序准备好处理重复的 RowKey 异常。Ticks 不一定是唯一的。</td>
</tr>
<tr class="odd">
<td align="left">ScheduledDate</td>
<td align="left">Date</td>
<td align="left">计划发送邮件的日期。（与 <code data-inline="1">PartitionKey</code> 相同，但是采用 Date 格式。）</td>
</tr>
<tr class="even">
<td align="left">SubjectLine</td>
<td align="left">String</td>
<td align="left">电子邮件的主题行。</td>
</tr>
<tr class="odd">
<td align="left">ListName</td>
<td align="left">String</td>
<td align="left">要向其发送此邮件的列表。</td>
</tr>
<tr class="even">
<td align="left">状态</td>
<td align="left">String</td>
<td align="left"><ul>
<li>“挂起”-- 辅助角色 A 尚未开始创建用于计划电子邮件的队列消息。</li>
<li>“正在排队”-- 辅助角色 A 已开始创建用于计划电子邮件的队列消息。</li>
<li>“正在处理”-- 辅助角色 A 已为列表中的所有电子邮件创建队列工作项，但电子邮件尚未全部发送。</li>
<li>“已完成”-- 辅助角色 B 已处理完所有队列工作项（所有电子邮件均已发送）。已完成的行在 <code data-inline="1">messagearchive</code> 表中存档，详见后面的叙述。我们希望在下一版中让这个属性成为 <code data-inline="1">enum</code>。</li>
</ul></td>
</tr>
</tbody>
</table>

当辅助角色 A 为要发送到列表的电子邮件创建队列消息时，它会在 `message` 表中创建一个电子邮件行。当辅助角色 B 发送电子邮件时，它会将电子邮件行移至 `messagearchive` 表，并将 `EmailSent` 属性更新为 `true`。将“正在处理”状态的消息的所有电子邮件行存档以后，辅助角色 A 会将状态设置为“已完成”，然后将 `message` 行移至 `messagearchive` 表。

以下网格显示 `message` 表中电子邮件行的属性。

| 属性              | 数据类型 | 说明                                                                                    |
|-------------------|----------|-----------------------------------------------------------------------------------------|
| PartitionKey      | String   | 计划发送邮件的日期，采用 yyyy-mm-dd 格式。                                              |
| RowKey            | String   | `mailinglist` 表中 `subscriber` 行的 `MessageRef` 值和目标电子邮件地址。                |
| MessageRef        | Long     | 与 `RowKey` 的 `MessageRef` 组件相同。                                                  |
| ScheduledDate     | Date     | `message` 表中 `message` 行的计划日期。（与 `PartitionKey` 相同，但是采用 Date 格式。） |
| SubjectLine       | String   | `message` 表中 `message` 行的电子邮件主题行。                                           |
| ListName          | String   | `mailinglist` 表的邮件列表名称。                                                        |
| From EmailAddress | String   | `mailinglist` 表中 `mailinglist` 行的“发件人”电子邮件地址。                             |
| EmailAddress      | String   | `mailinglist` 表中 `subscriber` 行的电子邮件地址。                                      |
| SubscriberGUID    | String   | `mailinglist` 表中 `subscriber` 行的订户 GUID。                                         |
| EmailSent         | Boolean  | False 表示电子邮件尚未发送；true 表示电子邮件已发送。                                   |

这些行中存在冗余数据，而在关系数据库中则通常需要避免冗余数据。而在本示例中，虽然存在冗余数据的不利之处，但却可以获得提高处理效率和可伸缩性的优势。由于电子邮件所需的所有数据存在于这些行中的某一个行中，辅助角色 B 只需读取一个行即可发送电子邮件，同时会将一个工作项从队列中移除。

你可能想要知道电子邮件的正文来自何处。这些行并没有包含电子邮件正文的文件的 Blob 引用，因为该值是从 `MessageRef` 值派生的。例如，如果 `MessageRef` 为 634852858215726983，则会将 Blob 命名为 634852858215726983.htm 和 634852858215726983.txt。

以下列表显示了一个示例，说明了表中的数据看起来会是什么样子。

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
2012-10-15

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
message634852858215726983

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
MessageRef

</th>
<td>
634852858215726983

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
ScheduledDate

</th>
<td>
2012-10-15

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubjectLine

</th>
<td>
新的讲座系列

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
ListName

</th>
<td>
contoso1

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
状态

</th>
<td>
正在处理

</td>
</tr>
</table>

------------------------------------------------------------------------

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
2012-10-15

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
634852858215726983student1@contoso.edu

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
MessageRef

</th>
<td>
634852858215726983

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
ScheduledDate

</th>
<td>
2012-10-15

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubjectLine

</th>
<td>
新的讲座系列

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
ListName

</th>
<td>
contoso1

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
FromEmailAddress

</th>
<td>
donotreply@contoso.edu

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
EmailAddress

</th>
<td>
student1@contoso.edu

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubscriberGUID

</th>
<td>
76543210-90ed-41a9-b8ac-c1310c67b66a

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
EmailSent

</th>
<td>
是

</td>
</tr>
</table>

------------------------------------------------------------------------

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
2012-10-15

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
634852858215726983student2@contoso.edu

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
MessageRef

</th>
<td>
634852858215726983

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
ScheduledDate

</th>
<td>
2012-10-15

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubjectLine

</th>
<td>
新的讲座系列

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
ListName

</th>
<td>
contoso1

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
FromEmailAddress

</th>
<td>
donotreply@contoso.edu

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
EmailAddress

</th>
<td>
student2@contoso.edu

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubscriberGUID

</th>
<td>
12345678-90ed-41a9-b8ac-c1310c679876

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
EmailSent

</th>
<td>
是

</td>
</tr>
</table>

------------------------------------------------------------------------

<table border="1">
<tr>
<th width="200" align="right" bgcolor="lightgray">
分区键

</th>
<td>
2012-11-15

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
行键

</th>
<td>
message124852858215726999

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
MessageRef

</th>
<td>
124852858215726999

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
ScheduledDate

</th>
<td>
2012-11-15

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
SubjectLine

</th>
<td>
新的招聘广告

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
ListName

</th>
<td>
fabrikam

</td>
</tr>
<tr>
<th align="right" bgcolor="lightgray">
状态

</th>
<td>
挂起

</td>
</tr>
</table>

### messagearchive 表

若要确保查询的执行效率，尤其是在必须在除 `PartitionKey` 和 `RowKey` 之外的字段中进行搜索的情况下的执行效率，一种策略是限制表的大小。辅助角色 A（负责查看消息的所有电子邮件是否已发送）中的查询需要查找 `message` 表中 `EmailSent` 为 false 的电子邮件行。`EmailSent` 值不在 PartitionKey 或 RowKey 中，因此对于有许多电子邮件行的消息来说，这不是一种有效的查询。所以，应用程序会在发送电子邮件时，将电子邮件行移至 `messagearchive` 表。结果就是，一个查询在查看某个消息的所有电子邮件是否已发送时，只需查询邮件表的 `PartitionKey` 和 `RowKey`，因为如果它发现某个消息的电子邮件行，则意味着存在未发送的邮件，因此不能将该消息标记为 `Complete`。

`messagearchive` 表的行架构与 `message` 表相同。通过减少为每个行存储的属性数以及删除存在时间长于某个特定值的行，你可以限制此存档数据的大小和开销，具体取决于你想要如何利用这些存档数据。

## <a name="queues"></a><span class="short-header">队列</span>Azure 队列

Azure 队列可用于加强此多层应用程序各层之间以及后端层中辅助角色之间的通信。
可以使用队列在辅助角色 A 和辅助角色 B 之间进行通信，使应用程序具有可伸缩性。辅助角色 A 可以在邮件表中为每封电子邮件创建一个行，而辅助角色 B 则可以扫描表中是否存在代表未发送电子邮件的行，不过你无法添加额外的辅助角色 B 实例来对工作进行划分。使用表行来协调辅助角色 A 与辅助角色 B 的工作的一个问题是，你无法确保只有一个辅助角色实例会选取任何给定的表行来进行处理。队列为你提供这种保障。当某个辅助角色实例从队列中拉取一个工作项时，队列服务会确保其他辅助角色实例无法拉取同一个工作项。Azure 队列的这种排他性的租约功能有助于在一个辅助角色的多个实例之间共享工作负载。

Azure 还提供 Service Bus 队列服务。有关 Azure 存储队列和 Service Bus 队列的详细信息，请参阅[本系列最后一个教程][构建 Azure 电子邮件服务应用程序的辅助角色 B（电子邮件发件人）]结尾处列出的资源。

Azure 电子邮件服务应用程序使用名为 `AzureMailQueue` 和 `AzureMailSubscribeQueue` 的两个队列。

### AzureMailQueue

`AzureMailQueue` 队列用于协调将电子邮件发送到电子邮件列表这一过程。辅助角色 A 在队列中为每封要发送的电子邮件放置一个工作项，而辅助角色 B 则从队列中拉取一个工作项并发送该电子邮件。

队列工作项包含逗号分隔的字符串，其中包含邮件的计划发送日期（`message` 表的分区键）以及 `MessageRef` 和 `EmailAddress`（`message` 表的行键）值，此外还有一个标记，用于指示该项是否是在辅助角色发生故障并重新启动之后创建的，例如：

      2012-10-15,634852858215726983,student1@contoso.edu,0

辅助角色 B 使用这些值来查找 `message` 表（包含发送电子邮件所需的全部信息）中的行。如果重新启动标记指示存在重新启动操作，则辅助角色 B 会在验证该电子邮件尚未发送之后，才会发送邮件。

处于流量高峰时，可以对云服务进行重新配置，以便实例化辅助角色 B 的多个实例，使得每一个实例都能独立拉取队列中的工作项。

### AzureMailSubscribeQueue

`AzureMailSubscribeQueue` 队列用于协调订阅确认电子邮件的发送过程。调用服务方法时，服务方法会将一个工作项放置在队列中。辅助角色 B 从队列中拉取该工作项，然后发送订阅确认电子邮件。

队列工作项包含订户 GUID。此值用于唯一标识电子邮件地址以及通过电子邮件地址订阅的列表，辅助角色 B 据此即可发送确认电子邮件。如前所述，这需要对既不在 `PartitionKey` 中，也不在 `RowKey` 中的字段进行查询，仅有这两个条件是不够的。若要使应用程序更具可伸缩性，必须重新构建 `mailinglist` 表，使 `RowKey` 中包含订户 GUID。

## <a name="datadiagram"></a><span class="short-header">数据图</span>Azure 电子邮件服务数据图

下图显示表和队列及其关系。

![Azure 电子邮件服务应用程序的数据图][Azure 电子邮件服务应用程序的数据图]

## <a name="blobs"></a><span class="short-header">Blob</span>Azure Blob

Blob 是指“二进制大型对象”。Azure Blob 服务是一种上载文件到云中以及在云中存储文件的方法。有关 Azure Blob 的详细信息，请参阅[本系列最后一个教程][构建 Azure 电子邮件服务应用程序的辅助角色 B（电子邮件发件人）]结尾处列出的资源。

Azure 邮件服务管理员将电子邮件的正文以 HTML 形式保存在 *.htm* 文件中，同时以纯文本形式保存在 *.txt* 文件中。计划发送某封电子邮件时，他们会将这些文件上载到“创建邮件”网页，该网页的 ASP.NET MVC 控制器会将上载的文件存储在 Azure Blob 中。

Blob 存储在 Blob 容器中，就像文件存储在文件夹中一样。Azure 邮件服务应用程序使用名为 **azuremailblobcontainer** 的单个 Blob 容器。容器中 Blob 的名称是通过将 MessageRef 值与文件扩展名连接到一起来派生的，例如：
634852858215726983.htm 和 634852858215726983.txt。

由于 HTML 和纯文本邮件实质上都是字符串，因此我们本来也可以将应用程序设计为将电子邮件正文存储在 `Message` 表的字符串属性中而非 Blob 中。不过，表行中属性的大小存在 64K 的限制，因此使用 Blob 可以避免对电子邮件正文大小设置的该限制。（64K 是该属性的最大总大小，而考虑到编码开销，你能够存储在属性中的最大字符串的大小实际上约为 48K。）

## <a name="wawsvswacs"></a><span class="short-header">云服务与网站</span>Azure 云服务与 Azure 网站

但你下载 Azure 电子邮件服务时，该服务已配置为让前端和后端都在单个 Azure 云服务中运行。

![应用程序体系结构概述][应用程序体系结构概述]

可以使用替代的体系结构，即在 Azure 网站中运行前端。

![替代的应用程序体系结构][替代的应用程序体系结构]

将所有组件都保留在一个云服务中可以简化配置和部署。如果你在创建应用程序时让 ASP.NET MVC 前端保留在 Azure 网站中，你就会有两个部署，一个部署是部署到 Azure 网站，另一个部署是部署到 Azure 云服务。此外，Azure 云服务 Web 角色还提供以下功能，这些功能在 Azure 网站中不可用：

-   支持自定义证书和通配符证书。
-   完全控制 IIS 的配置方式。许多 IIS 功能无法在 Azure 网站上启用。使用 Azure Web 角色，你可以定义一个启动命令，以便运行 [AppCmd][AppCmd] 程序来修改 IIS 设置，该设置无法在 *Web.config* 文件中配置。有关详细信息，请参阅[如何在 Azure 中配置 IIS 组件][如何在 Azure 中配置 IIS 组件]和[如何阻止特定 IP 地址访问 Web 角色][如何阻止特定 IP 地址访问 Web 角色]。
-   支持使用[自动缩放应用程序块][自动缩放应用程序块]来自动缩放 Web 应用程序。
-   能够运行提升的启动脚本，以便安装应用程序、修改注册表设置、安装性能计数器，等等。
-   网络隔离，适用于 [Azure Connect][Azure Connect] 和 [Azure 虚拟网络][Azure 虚拟网络]。
-   远程桌面访问，用于调试和高级诊断。
-   通过 [虚拟 IP 交换][虚拟 IP 交换]进行滚动升级。此功能用于交换过渡部署和生产部署的内容。

此替代体系结构可能会有一些成本优势，因为与在云服务中运行的 Web 角色相比，虽然提供的容量类似，但使用 Azure 网站的成本可能更低。本系列后面的教程介绍了这两种体系结构的不同实施细节。

有关如何在 Azure 网站和 Azure 云服务之间进行抉择的详细信息，请参阅 [Azure 执行模型][Azure 执行模型]。

## <a name="cost"></a><span class="short-header">成本</span>成本

本部分概述了在 Azure 中运行此示例应用程序的成本，采用的是本教程在 2012 年 12 月发布时的实际费率。在基于成本进行业务决策时，请确保查看以下网页上提供的最新费率：

-   [Azure 价格计算器][Azure 价格计算器]
-   [SendGrid Azure][SendGrid Azure]

成本受你决定要维护的 Web 角色实例和辅助角色实例数目的影响。若要获得 [Azure 云服务 99.95% 服务级别协议 (SLA)][Azure 云服务 99.95% 服务级别协议 (SLA)] 的资格，你必须每个角色部署两个或两个以上的实例。之所以必须运行至少两个角色实例，其中一个原因是运行应用程序的虚拟机大约每月要重新启动两次进行操作系统升级。（有关 OS 更新的详细信息，请参阅[重新启动角色实例进行 OS 升级][重新启动角色实例进行 OS 升级]。）

本示例中这两个辅助角色所做的工作对时间要求并不严格，因此不需要 99.5% SLA。因此，每个辅助角色只运行一个实例是可行的，但前提是该实例能够及时处理工作负载。Web 角色实例是时间敏感型的，换句话说，用户不希望网站出现停机的情况，因此生产型应用程序应该至少有两个 Web 角色实例。

下表显示了使用最小工作负载的 Azure 电子邮件服务示例应用程序的默认体系结构的成本。所示成本基于这样一个事实，即用户会使用特小型（共享）虚拟机大小。当你创建 Visual Studio 云项目时，默认的虚拟机大小是小型，其价格大约是特小型的六倍。

<table border="1">
<tr bgcolor="lightgray">
<th>
组件或服务

</th>
<th>
费率

</th>
<th>
每月成本

</th>
</tr>
<tr>
<td>
Web 角色

</td>
<td>
2 个实例的价格为 $.02/小时，适用于特小型实例

</td>
<td>
$29.00

</td>
</tr>
<tr>
<td>
辅助角色 A（计划要发送的电子邮件）

</td>
<td>
1 个实例的价格为 $.02/小时，适用于特小型实例

</td>
<td>
$14.50

</td>
</tr>
<tr>
<td>
辅助角色 B（发送电子邮件）

</td>
<td>
1 个实例的价格为 $.02/小时，适用于特小型实例

</td>
<td>
$14.50

</td>
</tr>
<tr>
<td>
Azure 存储事务

</td>
<td>
每月 1 百万个事务的价格为 $0.10/百万（每个查询都计为一个事务；辅助角色 A 会持续查询表中是否存在需要发送的邮件。此外，应用程序也会配置为将诊断数据写入 Azure 存储空间，每写入一次计为一个事务。）

</td>
<td>
$0.10

</td>
</tr>
<tr>
<td>
Azure 的本地冗余存储

</td>
<td>
每 25 GB（包括应用程序表和诊断数据的存储空间）$2.33

</td>
<td>
$2.33

</td>
</tr>
<tr>
<td>
带宽

</td>
<td>
免费提供 5 GB 出口带宽

</td>
<td>
免费

</td>
</tr>
<tr>
<td>
SendGrid

</td>
<td>
Azure 客户每月可免费发送 25,000 封电子邮件

</td>
<td>
免费

</td>
</tr>
<tr>
<td colspan="2">
总计

</td>
<td>
$60.43

</td>
</tr>
</table>
你可以看到，角色实例的成本占总成本的很大一部分。角色实例即使在已停止的情况下也会产生成本；你必须删除角色实例才不需要支付任何费用。一个节省成本的方法是将辅助角色 A 和辅助角色 B 的所有代码都移到一个辅助角色中去。在本系列教程中，我们特意选择实施两个辅助角色实例，目的是简化横向扩展过程。辅助角色 B 所做的工作由 Azure 队列服务进行协调，这意味着，你只需增加角色实例的数目即可横向扩展辅助角色 B。（辅助角色 B 是针对高负载情况的限制因素。）辅助角色 A 所做的工作不通过队列进行协调，因此无法运行辅助角色 A 的多个实例。如果将两个辅助角色结合起来，并且需要启用横向扩展，则需实施一个机制来确保辅助角色 A 的任务仅在一个实例中运行。（[CloudFx][CloudFx] 提供这样一种机制。请参阅 [WorkerRole.cs 示例][WorkerRole.cs 示例]。）

也可以将两个辅助角色的所有代码移到 Web 角色中，使一切都在 Web 角色中运行。但是，在 ASP.NET 中执行后台任务是不受支持的，也并不可靠，而且这种体系结构会使可伸缩性的实现变得复杂化。有关详细信息，请参阅[在 ASP.NET 中实施重复的后台任务的危险性][在 ASP.NET 中实施重复的后台任务的危险性]。另请参阅[如何在 Azure 中组合使用辅助角色和 Web 角色][如何在 Azure 中组合使用辅助角色和 Web 角色]和[将多个 Azure 辅助角色组合成一个 Azure Web 角色][将多个 Azure 辅助角色组合成一个 Azure Web 角色]。

另一个可以减少成本的替代体系结构是仅在计划的期间使用[自动缩放应用程序块][自动缩放应用程序块]来自动部署辅助角色，一旦工作完成即删除这些角色。有关自动缩放的详细信息，请参阅[本系列最后一个教程][构建 Azure 电子邮件服务应用程序的辅助角色 B（电子邮件发件人）]结尾处的链接。

未来版本的 Azure 可能会提供一个针对计划的重启的通知机制，让你只需启动一个额外的针对重启时间窗口的 Web 角色实例。虽然你没有资格获得 99.95% SLA，但你可以减少大约一半的成本，并且可以确保 Web 应用程序在重启过程中始终可用。

## <a name="auth"></a><span class="short-header">身份验证和授权</span>身份验证和授权

在生产应用程序中，你会实施一个身份验证或授权机制，例如一个针对 ASP.NET MVC Web 前端的 ASP.NET 成员身份系统，其中包括 ASP.NET Web API 服务方法。此外还可以使用其他选项（例如使用共享机密）来确保 Web API 服务方法的安全性。本示例应用程序去除了身份验证和授权功能，目的是简化安装和部署过程。（本系列的第二个教程讲述如何实施 IP 限制，使得未经授权的人员无法使用你部署到云中的应用程序。）

有关如何在 ASP.NET MVC Web 项目中实施身份验证和授权的详细信息，请参阅以下资源：

-   [ASP.NET Web API 中的身份验证和授权][ASP.NET Web API 中的身份验证和授权]
-   [音乐商店第 7 部分：成员身份和授权][音乐商店第 7 部分：成员身份和授权]

**注意**：我们曾计划提供一个使用共享机密来确保 Web API 服务方法安全性的机制，但未能赶在初始发布之前及时完成任务。因此，第三个教程不讲述如何生成针对订阅过程的 Web API 控制器。我们希望在本教程的下一版本中，能够提供有关如何实施安全订阅过程的说明。而现在，你可以使用管理员网页将电子邮件地址订阅到列表中，通过这种方式测试应用程序。

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

在[下一教程][配置和部署 Azure 电子邮件服务应用程序]中，你需要下载示例项目、配置开发环境、针对环境来配置项目，以及在本地和云中对项目进行测试。在以下教程中，你将了解如何从头开始生成项目。

有关如何使用 Azure 存储表、队列和 Blob 的其他资源的链接，请参阅[本系列最后一个教程][本系列最后一个教程]。

<div><a href="/zh-cn/develop/net/tutorials/multi-tier-web-site/2-download-and-run/" class="site-arrowboxcta download-cta">教程 2</a></div>

  [下载已完成的应用程序]: http://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36
  [电子邮件处理]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-worker-roles-a-and-b.png
  [配置和部署 Azure 电子邮件服务应用程序]: /zh-cn/develop/net/tutorials/multi-tier-web-site/2-download-and-run/
  [构建 Azure 电子邮件服务应用程序的 Web 角色]: /zh-cn/develop/net/tutorials/multi-tier-web-site/3-web-role/
  [构建 Azure 电子邮件服务应用程序的辅助角色 A（电子邮件计划程序）]: /zh-cn/develop/net/tutorials/multi-tier-web-site/4-worker-role-a/
  [构建 Azure 电子邮件服务应用程序的辅助角色 B（电子邮件发件人）]: /zh-cn/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/
  [先决条件]: #prerequisites
  [为什么使用电子邮件列表]: #whyanemaillistapp
  [前端概述]: #frontend
  [后端概述]: #backend
  [Azure 表]: #tables
  [Azure 队列]: #queues
  [数据图]: #datadiagram
  [Azure Blob]: #blobs
  [Azure 云服务与 Azure 网站]: #wawsvswacs
  [成本]: #cost
  [身份验证和授权]: #auth
  [后续步骤]: #nextsteps
  [TechNet 电子书库]: http://social.technet.microsoft.com/wiki/contents/articles/11608.e-book-gallery-for-microsoft-technologies.aspx#ASPNETMultiTierWindowsAzureApplicationUsingStorageTablesQueuesandBlobs
  [邮件列表索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-mailing-list-index-page.png
  [订户索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-subscribers-index-page.png
  [邮件索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-message-index-page.png
  [邮件创建页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-message-create-page.png
  [确认电子邮件]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-subscribe-email.png
  [欢迎访问列表页面]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-subscribe-confirmation-page.png
  [确认取消订阅页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-unsubscribe-query-page.png
  [取消订阅已确认页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-unsubscribe-confirmation-page.png
  [阻止重复的电子邮件]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-message-processing.png
  [订阅队列邮件处理]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-subscribe-diagram.png
  [Azure SQL Database]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee336279.aspx
  [实际应用：为 Azure 表存储设计可扩展分区策略]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh508997.aspx
  [Azure 电子邮件服务应用程序的数据图]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-datadiagram.png
  [应用程序体系结构概述]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-architecture-overview.png
  [替代的应用程序体系结构]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-overview/mtas-alternative-architecture.png
  [AppCmd]: http://www.iis.net/learn/get-started/getting-started-with-iis/getting-started-with-appcmdexe "appCmd"
  [如何在 Azure 中配置 IIS 组件]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433059.aspx
  [如何阻止特定 IP 地址访问 Web 角色]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj154098.aspx
  [自动缩放应用程序块]: /zh-cn/develop/net/how-to-guides/autoscaling/
  [Azure Connect]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433122.aspx
  [Azure 虚拟网络]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156007.aspx
  [虚拟 IP 交换]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee517253.aspx "VIP 交换"
  [Azure 执行模型]: http://www.windowsazure.com/zh-cn/manage/windows/fundamentals/compute/
  [Azure 价格计算器]: http://www.windowsazure.com/zh-cn/pricing/calculator/
  [SendGrid Azure]: http://sendgrid.com/windowsazure.html
  [Azure 云服务 99.95% 服务级别协议 (SLA)]: https://www.windowsazure.com/zh-cn/support/legal/sla/ "SLA"
  [重新启动角色实例进行 OS 升级]: http://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx
  [CloudFx]: http://nuget.org/packages/Microsoft.Experience.CloudFx "CloudFX"
  [WorkerRole.cs 示例]: http://code.msdn.microsoft.com/windowsazure/CloudFx-Samples-60c3a852/sourcecode?fileId=57087&pathId=528472169
  [在 ASP.NET 中实施重复的后台任务的危险性]: http://haacked.com/archive/2011/10/16/the-dangers-of-implementing-recurring-background-tasks-in-asp-net.aspx
  [如何在 Azure 中组合使用辅助角色和 Web 角色]: http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2010/12/how-to-combine-worker-and-web-role-in.html
  [将多个 Azure 辅助角色组合成一个 Azure Web 角色]: http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html
  [ASP.NET Web API 中的身份验证和授权]: http://www.asp.net/web-api/overview/security/authentication-and-authorization/authentication-and-authorization-in-aspnet-web-api
  [音乐商店第 7 部分：成员身份和授权]: http://www.asp.net/mvc/tutorials/mvc-music-store/mvc-music-store-part-7
  [本系列最后一个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/#nextsteps
