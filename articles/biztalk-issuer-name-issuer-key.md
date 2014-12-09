<properties linkid="manage-services-biztalk-services-issuer-name-issuer-key" urlDisplayName="Issuer name and issuer key" pageTitle="Issuer Name and Issuer Key in BizTalk Services | Azure" metaKeywords="BizTalk Services, BizTalk, issuer name, issuer key, Azure" description="Learn how to retrieve Issuer Name and Issuer Key for either Service Bus or Access Control (ACS) in BizTalk Services." metaCanonical="" services="biztalk-services" documentationCenter="" title="BizTalk Services: Issuer Name and Issuer Key" authors="mandia" solutions="" manager="paulettm" editor="susanjo" />

# BizTalk 服务：颁发者名称和颁发者密钥

Azure BizTalk 服务使用 Service Bus 颁发者名称和颁发者密钥以及 Access Control 颁发者名称和颁发者密钥。具体内容包括：

<table border="1">
<tr bgcolor="FAF9F9">
<td><strong>任务</strong></td>
<td><strong>哪个颁发者名称和颁发者密钥</strong></td>
</tr>
<tr>
<td>从 Visual Studio 部署应用程序</td>
<td>Access Control 颁发者名称和颁发者密钥</td>
</tr>
<tr>
<td>配置 Azure BizTalk 服务门户</td>
<td>Access Control 颁发者名称和颁发者密钥</td>
</tr>
<tr>
<td>在 Visual Studio 中使用 BizTalk 适配器服务创建 LOB 中继</td>
<td>Service Bus 颁发者名称和颁发者密钥</td>
</tr>
</table>

本主题列出了检索颁发者名称和颁发者密钥的步骤。

## Access Control 颁发者名称和颁发者密钥

Access Control 颁发者名称和颁发者密钥由以下各项使用：

-   在 Visual Studio 中创建的 Azure BizTalk 服务应用程序：若要在 Visual Studio 中成功将 BizTalk 服务部署到 Azure，你需要输入 Access Control 颁发者名称和颁发者密钥。
-   Azure BizTalk 服务门户：你首次登录 BizTalk 服务门户时，应输入 BizTalk 服务名称作为服务提供商，然后输入 Access Control 颁发者名称和 Access Control 颁发者密钥。

### 检索 Access Control 颁发者名称和颁发者密钥

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左侧导航窗格中，选择**“BizTalk 服务”**。
3.  选择你的 BizTalk 服务。
4.  在任务栏中，选择**“连接信息”**。Access Control 命名空间、默认颁发者（颁发者名称）和默认密钥（颁发者密钥）将列出并且可被复制和粘贴。<br/><br/>
总结：<br/>
颁发者名称 = 默认颁发者<br/>
颁发者密钥 = 默认密钥

你还可以单击**“打开 ACS 管理门户”**检索 Access Control 值：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左侧导航窗格中，选择**“BizTalk 服务”**。
3.  选择你的 BizTalk 服务。
4.  选择“连接信息”按钮，然后选择**“打开 ACS 管理门户”**。
5.  在“**服务设置”**下的“门户”中，单击**“服务标识”**。此时将显示服务标识，它是“Access Control 颁发者名称”值。单击“服务标识”链接以查看“密码”，这是“颁发者密钥”值。可以复制其值。<br/><br/>
    例如，在**“服务标识”**中，你可以看到“所有者”。“所有者”是你的 Access Control 颁发者名称。当你单击“所有者”链接时，你会看到**“密码”**。当你单击“密码”链接时，你会看到此值。此“密码”值为你的 Access Control 颁发者密钥。<br/><br/>
    总结：<br/>
    颁发者名称 = 服务标识名称<br/>
    颁发者密钥 = 密码值

在左导航窗格中，你还可以选择**“Active Directory”**检索 Access Control 值。

<div class="dev-callout"> 
<b>重要说明</b> 
<p>在使用<b>“Active Directory”</b>创建 Access Control 命名空间时，<strong>不</strong>自动创建服务标识。在你设置某一 BizTalk 服务时，将自动创建 Access Control 命名空间、名为&ldquo;所有者&rdquo;的服务标识（颁发者名称）、密码（颁发者密钥）和对称密钥。</p> 
<p><a href="http://go.microsoft.com/fwlink/p/?LinkID=303942">如何：使用 ACS 管理服务配置服务标识</a>提供了有关 Access Control 服务标识的详细信息。</p>
</div>

## Service Bus 颁发者名称和颁发者密钥

Service Bus 颁发者名称和颁发者密钥由 BizTalk 适配器服务使用。在 Visual Studio 中的 BizTalk 服务项目中，使用 BizTalk 适配器服务连接到内部部署的业务线 (LOB) 系统。若要连接，请创建 LOB 中继并输入 LOB 系统的详细信息。在执行此操作时，你还可以输入 Service Bus 颁发者名称和颁发者密钥。

### 检索 Service Bus 颁发者名称和颁发者密钥

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左侧导航窗格中，单击**“Service Bus”**。
3.  单击你的命名空间。在任务栏中，单击**“连接信息”**。这会显示**“默认颁发者”**（颁发者名称）和**“默认密钥”**（颁发者密钥）。可以复制其值。<br/><br/>
    总结：<br/>
    颁发者名称 = 默认颁发者<br/>
    颁发者密钥 = 默认密钥

## 下一步

其他 Azure BizTalk 服务主题：

-   [安装 Azure BizTalk 服务 SDK][安装 Azure BizTalk 服务 SDK]
-   [教程：Azure BizTalk 服务][教程：Azure BizTalk 服务]
-   [如何开始使用 Azure BizTalk 服务 SDK][如何开始使用 Azure BizTalk 服务 SDK]
-   [Azure BizTalk 服务][Azure BizTalk 服务]

## 另请参阅

-   [如何：使用 ACS 管理服务配置服务标识][如何：使用 ACS 管理服务配置服务标识]
-   [BizTalk 服务：开发人员版、基本版、标准版和高级版图表][BizTalk 服务：开发人员版、基本版、标准版和高级版图表]
-   [BizTalk 服务：使用 Azure 管理门户进行设置][BizTalk 服务：使用 Azure 管理门户进行设置]
-   [BizTalk 服务：设置状态图表][BizTalk 服务：设置状态图表]
-   [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡][BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]
-   [BizTalk 服务：备份和还原][BizTalk 服务：备份和还原]
-   [BizTalk 服务：限制][BizTalk 服务：限制]

  [Azure 管理门户]: http://go.microsoft.com/fwlink/p/?LinkID=213885
  [如何：使用 ACS 管理服务配置服务标识]: http://go.microsoft.com/fwlink/p/?LinkID=303942
  [安装 Azure BizTalk 服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=241589
  [教程：Azure BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=236944
  [如何开始使用 Azure BizTalk 服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=302335
  [Azure BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=303664
  [BizTalk 服务：开发人员版、基本版、标准版和高级版图表]: http://go.microsoft.com/fwlink/p/?LinkID=302279
  [BizTalk 服务：使用 Azure 管理门户进行设置]: http://go.microsoft.com/fwlink/p/?LinkID=302280
  [BizTalk 服务：设置状态图表]: http://go.microsoft.com/fwlink/p/?LinkID=329870
  [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]: http://go.microsoft.com/fwlink/p/?LinkID=302281
  [BizTalk 服务：备份和还原]: http://go.microsoft.com/fwlink/p/?LinkID=329873
  [BizTalk 服务：限制]: http://go.microsoft.com/fwlink/p/?LinkID=302282
