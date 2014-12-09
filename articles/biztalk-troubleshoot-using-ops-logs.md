<properties linkid="biztalk-troubleshoot-using-ops-logs" urlDisplayName="BizTalk Services: Troubleshoot using operation logs" pageTitle="BizTalk Services: Troubleshoot using ops logs | Azure" metaKeywords="" description="BizTalk Services: Troubleshoot using ops logs" metaCanonical="" services="" documentationCenter="" title="BizTalk Services: Troubleshoot using ops logs" authors="mandia"  solutions="" writer="nitinme" manager="paulettm" editor="cgronlun"  />

# BizTalk 服务：使用操作日志进行故障排除

操作日志是 Azure 管理门户中提供的一项管理服务功能，可让你查看针对 Azure 服务（包括 BizTalk 服务）执行的操作的历史日志。可以查看过去最多 180 天内对 BizTalk 服务订阅执行的管理操作的相关历史数据。

<div class="dev-callout"><b>说明</b>
<p>此功能只会捕获对 BizTalk 服务执行的管理操作日志，例如，服务的启动时间、备份时间，等等。不管此类操作是从 Azure 管理门户执行的，还是使用 <a href="http://msdn.microsoft.com/en-us/library/windowsazure/dn232347.aspx">BizTalk 服务 REST API</a> 执行的，都会受到跟踪。有关使用管理服务跟踪的操作的完整列表，请参阅<a href="#bizops">使用 Azure 管理服务跟踪的操作</a>。</p>
<p>此功能不会捕获与 BizTalk 服务运行时相关的活动（例如桥处理的消息等）的日志。若要查看此类日志，必须使用 BizTalk 服务门户中的&ldquo;跟踪&rdquo;视图。有关详细信息，请参阅<a HREF="http://msdn.microsoft.com/library/windowsazure/hh949805.aspx">跟踪消息</a>。</p>
</div>

## <a name="viewlogs"></a>查看 BizTalk 服务操作日志

1.  在 Azure 管理门户中单击“管理服务”，然后单击“操作日志”选项卡。
2.  你可以基于订阅、日期范围、服务类型（例如 BizTalk 服务）、服务名称或状态（操作的状态，例如“成功”、“失败”）等不同的参数来筛选日志。
3.  单击复选标记可以查看筛选后的列表。下图显示了与 testbiztalkservice 相关的活动。

	![查看操作日志][查看操作日志] 
4.  若要查看有关特定操作的详细信息，请选择相应的行，然后在页面底部单击**“详细资料”**。

## <a name="bizops"></a>使用 Azure 管理服务跟踪的操作

下表列出了使用 Azure 管理服务跟踪的操作。



<table border="1" cellpadding="5">
<tr>
<td>CreateBizTalkService</td> 
<td align="left">用于创建新 BizTalk 服务的操作/td> 
</tr> 
<tr>
<td>DeleteBizTalkService</td> 
<td align="left">用于删除 BizTalk 服务的操作</td>  
</tr> 
<tr>
<td>RestartBizTalkService</td> 
<td align="left">用于重新启动 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>StartBizTalkService</td> 
<td align="left">用于启动 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>StopBizTalkService</td> 
<td align="left"> 用于停止 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>DisableBizTalkService</td> 
<td align="left">用于禁用 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>EnableBizTalkService</td> 
<td align="left">用于启用 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>BackupBizTalkService</td> 
<td align="left">用于备份 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>RestoreBizTalkService</td> 
<td align="left">用于从指定的备份还原 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>SuspendBizTalkService</td> 
<td align="left">用于挂起 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>ResumeBizTalkService</td> 
<td align="left">用于恢复 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>ScaleBizTalkService</td> 
<td align="left">用于扩展或缩减 BizTalk 服务的操作</td> 
</tr>
<tr>
<td>ConfigUpdateBizTalkService</td> 
<td align="left">用于更新 BizTalk 服务配置的操作</td> 
</tr>
<tr>
<td>ServiceUpdateBizTalkService</td> 
<td align="left">用于将 BizTalk 服务升级或降级为不同版本的操作</td> 
</tr>
<tr>
<td>PurgeBackupBizTalkService</td> 
<td align="left">用于清除超过保留期的 BizTalk 服务备份的操作</td> 
</tr>
</table>

## 另请参阅

-   [备份 BizTalk 服务][备份 BizTalk 服务]
-   [从备份还原 BizTalk 服务][从备份还原 BizTalk 服务]
-   [BizTalk 服务：开发人员版、基本版、标准版和高级版图表][BizTalk 服务：开发人员版、基本版、标准版和高级版图表]
-   [BizTalk 服务：使用 Azure 管理门户进行设置][BizTalk 服务：使用 Azure 管理门户进行设置]
-   [BizTalk 服务：设置状态图表][BizTalk 服务：设置状态图表]
-   [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡][BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]
-   [BizTalk 服务：限制][BizTalk 服务：限制]
-   [BizTalk 服务：颁发者名称和颁发者密钥][BizTalk 服务：颁发者名称和颁发者密钥]
-   [如何开始使用 Azure BizTalk 服务 SDK][如何开始使用 Azure BizTalk 服务 SDK]

  [BizTalk 服务 REST API]: http://msdn.microsoft.com/en-us/library/windowsazure/dn232347.aspx
  [使用 Azure 管理服务跟踪的操作]: #bizops
  [跟踪消息]: http://msdn.microsoft.com/library/windowsazure/hh949805.aspx
  [查看操作日志]: ./media/biztalk-troubleshoot-using-ops-logs/Operation-Logs.png
  [备份 BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=325584
  [从备份还原 BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=325582
  [BizTalk 服务：开发人员版、基本版、标准版和高级版图表]: http://go.microsoft.com/fwlink/p/?LinkID=302279
  [BizTalk 服务：使用 Azure 管理门户进行设置]: http://go.microsoft.com/fwlink/p/?LinkID=302280
  [BizTalk 服务：设置状态图表]: http://go.microsoft.com/fwlink/p/?LinkID=329870
  [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]: http://go.microsoft.com/fwlink/p/?LinkID=302281
  [BizTalk 服务：限制]: http://go.microsoft.com/fwlink/p/?LinkID=302282
  [BizTalk 服务：颁发者名称和颁发者密钥]: http://go.microsoft.com/fwlink/p/?LinkID=303941
  [如何开始使用 Azure BizTalk 服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=302335
