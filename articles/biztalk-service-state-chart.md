<properties linkid="manage-services-biztalk-state-chart" urlDisplayName="BizTalk Services: Service state chart" pageTitle="BizTalk Services: Service state chart | Azure" metaKeywords="" description="" metaCanonical="" services="biztalk-services" documentationCenter="" title="BizTalk Services: Service state chart" authors="mandia" solutions="integration" manager="paulettm" editor="cgronlun" />

# BizTalk 服务：服务状态图表

根据 BizTalk 服务的当前状态，有一些操作不一定能够在 BizTalk 服务上完成。

例如，你在 Azure 管理门户中设置新的 BizTalk 服务。在其成功完成后，BizTalk 服务处于活动状态。在活动状态下，你可以停止 BizTalk 服务。如果停止成功，BizTalk 服务将进入“已停止”状态。如果停止失败，BizTalk 服务将进入“停止失败”状态。在“停止失败”状态下，你可以重新启动 BizTalk 服务。如果你尝试执行某一不允许的操作，例如恢复 BizTalk 服务，将会发生以下错误：

**操作不允许**

若要设置 BizTalk 服务，请参阅 [BizTalk 服务：使用 Azure 管理门户进行设置][BizTalk 服务：使用 Azure 管理门户进行设置]。

下表列出了在 BizTalk 服务处于某一特定状态时可执行的操作。复选标记意味着在处于该状态时可执行的操作。空白条目表示处于该状态时不能执行的操作。

#### 启动、停止、重新启动、挂起、恢复和删除操作

<table border="1">
<tr>
<th colspan="15">
操作

</th>
</tr>
<tr>
<th rowspan="18">
BizTalk 服务状态

</th>
</tr>
<tr bgcolor="FAF9F9">
<th>
</th>
<th>
启动

</th>
<th>
停止

</th>
<th>
重新启动

</th>
<th>
挂起

</th>
<th>
恢复

</th>
<th>
删除

</th>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>活动</strong>

</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>已禁用</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>已挂起</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>已停止</strong>

</td>
<td>
<center>
x

</center>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>服务更新失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>禁用失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>启用失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>启动失败
 停止失败
 重新启动失败</strong>

</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>挂起失败
 恢复失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>创建失败
 还原失败
</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>配置更新失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>缩放失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
</table>

#### 缩放、更新配置和备份操作

<table border="1">
<tr>
<th colspan="15">
操作

</th>
</tr>
<tr>
<th rowspan="18">
BizTalk 服务状态

</th>
</tr>
<tr bgcolor="FAF9F9">
<th>
</th>
<th>
缩放

</th>
<th>
更新配置

</th>
<th>
备份

</th>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>活动</strong>

</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>已禁用</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>已挂起</strong>

</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>已停止</strong>

</td>
<td>
</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>服务更新失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>禁用失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>启用失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>启动失败
 停止失败
 重新启动失败</strong>

</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
<td>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>挂起失败
 恢复失败</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>创建失败
 还原失败
</strong>

</td>
<td>
</td>
<td>
</td>
<td>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>配置更新失败</strong>

</td>
<td>
</td>
<td>
<center>
x

</center>
</td>
<td>
</td>
</tr>
<tr>
<td bgcolor="FAF9F9">
<strong>缩放失败</strong>

</td>
<td>
<center>
x

</center>
</td>
<td>
</td>
<td>
</td>
</tr>
</table>
## 另请参阅

-   [BizTalk 服务：使用 Azure 管理门户进行设置][BizTalk 服务：使用 Azure 管理门户进行设置]
-   [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡][BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]
-   [BizTalk 服务：开发人员版、基本版、标准版和高级版图表][BizTalk 服务：开发人员版、基本版、标准版和高级版图表]
-   [BizTalk 服务：备份和还原][BizTalk 服务：备份和还原]
-   [BizTalk 服务：限制][BizTalk 服务：限制]
-   [BizTalk 服务：颁发者名称和颁发者密钥][BizTalk 服务：颁发者名称和颁发者密钥]
-   [如何开始使用 Azure BizTalk 服务 SDK][如何开始使用 Azure BizTalk 服务 SDK]

  [BizTalk 服务：使用 Azure 管理门户进行设置]: http://go.microsoft.com/fwlink/p/?LinkID=302280
  [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]: http://go.microsoft.com/fwlink/p/?LinkID=302281
  [BizTalk 服务：开发人员版、基本版、标准版和高级版图表]: http://go.microsoft.com/fwlink/p/?LinkID=302279
  [BizTalk 服务：备份和还原]: http://go.microsoft.com/fwlink/p/?LinkID=329873
  [BizTalk 服务：限制]: http://go.microsoft.com/fwlink/p/?LinkID=302282
  [BizTalk 服务：颁发者名称和颁发者密钥]: http://go.microsoft.com/fwlink/p/?LinkID=303941
  [如何开始使用 Azure BizTalk 服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=302335
