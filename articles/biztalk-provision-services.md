<properties linkid="provisioning-biztalk-service" urlDisplayName="Provision BizTalk Services in management portal" pageTitle="Create BizTalk Services in management portal | Azure" metaKeywords="Get started Azure biztalk services, provision, Azure unstructured data" description="Learn how to provision a BizTalk service in the Azure Management Portal, as well as create an optional SQL database server and Storage account." metaCanonical="http://www.windowsazure.com/en-us/manage/services/biztalk-services/provisioning-biztalk-service" services="biztalk-services" documentationCenter="" title="BizTalk Services: Provisioning Using Azure Management Portal" authors="mandia" solutions="" manager="paulettm" editor="cgronlun" />

# 使用 Azure 管理门户创建 BizTalk 服务

本主题列出了在 Azure 管理门户中创建 Azure BizTalk 服务的步骤。具体内容包括：

-   [创建 BizTalk 服务][创建 BizTalk 服务]
-   [设置后的步骤][设置后的步骤]
-   [要求说明][要求说明]
-   [混合连接 - 新增功能！][混合连接 - 新增功能！]

<div class="dev-callout"> 
<b>提示</b> 
<p>若要登录到 Azure 管理门户，你需要使用 Azure 帐户和 Azure 订阅。如果你没有帐户，则可以创建一个免费的试用帐户，只需几分钟即可完成。请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkID=239738">Azure 免费试用</a>。</p> 
</div>

## <a name="BizTalk"></a>创建 BizTalk 服务

你不一定能够使用所有的 BizTalk 服务设置，这取决于你选择的版本。

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在导航窗格的底部，选择**“新建”**：

    ![选择“新建”按钮][选择“新建”按钮]

3.  选择**“应用程序服务”**\>**“BIZTALK 服务”**\>**“自定义创建”**：

    ![依次选择“BizTalk 服务”和“自定义创建”][依次选择“BizTalk 服务”和“自定义创建”]

4.  输入 BizTalk 服务设置。

	<table border="1">
	<tr>
	<td><strong>BizTalk 服务名称</strong></td>
	<td>你可以输入任何名称，但请尽量具体。示例包括：<br/><br/>
	<em>mycompany</em>.biztalk.windows.net<br/>
	<em>mycompanymyapplication</em>.biztalk.windows.net<br/>
	<em>myapplication</em>.biztalk.windows.net<br/><br/>
“.biztalk.windows.net”将自动添加到你输入的名称上。此时将会创建一个用于访问 BizTalk 服务的 URL，例如<strong>https://<em>myapplication</em>.biztalk.windows.net。</strong>.
	</td>
	</tr>
	<tr>
	<td><strong>版本</strong></td>
	<td>如果你处在开发/测试阶段，则选择<strong>“开发人员版”</strong>。如果你正处于生产阶段，请选择<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=302279">BizTalk 服务:版本图表</a>以确定<strong>“高级”</strong>、<strong>“标准”</strong>或<strong>“基本”</strong>选项中哪一项适合你的业务方案。	 
	</td>
	</tr>
	<tr>
	<td><strong>区域 </strong></td>
	<td>选择托管 BizTalk 服务的地理区域。</td>
	</tr>
	<tr>
	<td><strong>域 URL</strong></td>
	<td><strong>可选</strong>。 默认情况下，域 URL 为 <em>YourBizTalkServiceName</em>.biztalk.windows.net。也可以输入自定义域。例如，如果你的域是contoso，则可输入：<br/><br/>
	<em>MyCompany</em>.contoso.com<br/>
	<em>MyCompanyMyApplication</em>.contoso.com<br/>
	<em>MyApplication</em>.contoso.com<br/>
	<em>YourBizTalkServiceName</em>.contoso.com<br/>
	</td>
	</tr>
	</table>


    选择“下一步”箭头。

5.  输入存储和数据库设置：

    <table>
    <colgroup>
    <col width="50%" />
    <col width="50%" />
    </colgroup>
    <tbody>
    <tr class="odd">
    <td align="left"><strong>监视/存档存储帐户</strong></td>
    <td align="left">选择一个现有存储帐户或者选择创建新的存储帐户。<br /><br /> 如果要创建新的存储帐户，请输入<b>“存储帐户名称”</b>。</td>
    </tr>
    <tr class="even">
    <td align="left"><strong>跟踪数据库</strong></td>
    <td align="left">如果你要使用现有的 Azure SQL Database，需保证它没有被其他 BizTalk 服务使用。你需要使用创建该 Azure SQL Database 服务器时输入的登录名和密码。<br /><br />
    <div class="dev-callout"> 
<b>提示</b> 
<p>在与 BizTalk 服务相同的区域中创建跟踪数据库和监视/存档存储帐户。</p> 
</div></td>
    </tr>
    </tbody>
    </table>

    选择“下一步”箭头。

6.  输入数据库设置：


	<table border="1">
	<tr>
	<td><strong>名称</strong></td>
	<td>如果在上一屏幕中选择了<b>“创建新的 SQL Database 实例”</b>，将会提供此设置。
	<br/><br/>
	输入 BizTalk 服务要使用的 SQL Database 名称。</td>
	</tr>
	<tr>
	<td><strong>服务器</strong></td>
	<td>如果在上一屏幕中选择了<b>“创建新的 SQL Database 实例”</b>，将会提供此设置。
	<br/><br/>
	 选择现有的 SQL Database 服务器或创建新的 SQL Database 服务器。</td>
	</tr>
	<tr>
	<td><strong>服务器登录名</strong></td>
	<td>输入登录用户名。</td>
	</tr>
	<tr>
	<td><strong>服务器登录密码</strong></td>
	<td>输入登录密码。</td>
	</tr>
	<tr>
	<td><strong>区域</strong></td>
	<td>如果选择了<b>“创建新的 SQL Database 实例”</b>，将会提供此设置。选择用于托管你的 SQL Database 的地理区域。</td>
	</tr>
	</table>

单击复选标记以完成向导操作。此时将显示进度图标：

![完成时显示进度图标][完成时显示进度图标]

完成向导操作后，即会创建该 Azure BizTalk 服务，它随时可用于你的应用程序。默认设置足以满足你的需要。如果你想要更改默认设置，请在左侧导航窗格中选择**“BIZTALK 服务”**，然后选择你的 BizTalk 服务。其他设置显示在顶部的[“仪表板”、“监视”和“缩放”选项卡][“仪表板”、“监视”和“缩放”选项卡]中。

根据 BizTalk 服务的状态，某些操作可能无法完成。有关这些操作的列表，请转到 [BizTalk 服务状态图表][BizTalk 服务状态图表]。

## <a name="PostProv"></a>设置后的步骤

-   [添加生产就绪证书][添加生产就绪证书]
-   [获取 Access Control 命名空间][获取 Access Control 命名空间]

#### <a name="AddCert"></a>添加生产就绪证书

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左侧导航窗格中选择**“BIZTALK 服务”**，然后选择你的 BizTalk 服务。
3.  选择**“仪表板”**选项卡。
4.  选择**“更新 SSL 证书”**：

    ![修改 SSL 证书][修改 SSL 证书]

5.  浏览到包含 BizTalk 服务名称的私有 SSL 证书 (*CertificateName*.pfx)，输入密码并单击复选标记。

当你创建 Azure BizTalk 服务时，系统将自动为该 BizTalk 服务创建一个自签名证书。自签名证书仅在开发环境中使用。可以下载自签名证书，或者将它替换为生产就绪证书。

#### <a name="ACS"></a>获取 Access Control 命名空间

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左侧导航窗格中选择**“BIZTALK 服务”**，然后选择你的 BizTalk 服务。
3.  在任务栏中，选择**“连接信息”**：

    ![选择连接信息][选择连接信息]

4.  复制 Access Control 值。

当你从 Visual Studio 中部署 BizTalk 服务项目时，你进入此 Access Control 命名空间。系统将自动为 BizTalk 服务创建 Access Control 命名空间。

Access Control 值可用于任何应用程序。当你创建 Azure BizTalk 服务时，此 Access Control 命名空间将控制向 BizTalk 服务部署进行的身份验证。如果你要更改或管理命名空间，则在左侧导航窗格中选择**“ACTIVE DIRECTORY”**，然后选择你的命名空间。任务栏列出了你的选项。

单击**“管理”**会打开 Access Control 管理门户。在 Access Control 管理门户中，BizTalk 服务使用**“服务标识”**：

![Access Control 管理门户中的 ACS 服务标识][Access Control 管理门户中的 ACS 服务标识]

Access Control 服务标识是一组凭据，这些凭据允许应用程序或客户端使用 Access Control 直接进行身份验证并接收令牌。

**重要说明**
BizTalk 服务使用**“所有者”**（用于默认服务标识）以及**“密码”**值。如果你使用“对称密钥”值而不是“密码”值，则可能会发生以下错误：

*无法使用指定的凭据连接到 Access Control 管理服务帐户*

[管理 ACS 命名空间][管理 ACS 命名空间]列出了一些指导和建议。

## <a name="Requirements"></a>要求说明

下列要求不适用于免费版。

<table border="1">

<tr bgcolor="FAF9F9">

<td>
<b>需要什么</b>

</td>

<td>
<b>为何需要</b>

</td>

</tr>

<tr>

<td>
Azure 订阅

</td>

<td>
订阅决定了谁可以登录 Azure 管理门户，由帐户持有人在 <a HREF="https://account.windowsazure.com/Subscriptions"> Azure 订阅</a>中创建。
<br/><br/>
Azure 帐户可以有多个订阅，并且可由获得许可的任何用户管理。例如，你的 Azure 帐户持有人创建一个名为 <em>BizTalkServiceSubscription</em> 的订阅，并向贵公司内的 BizTalk 管理员（如 <ContosoBTSAdmins@live.com>）授予对此订阅的访问权限。在这种情况下，BizTalk 管理员登录到 Azure 管理门户，并对订阅中的所有托管服务（包括 Azure BizTalk 服务）拥有完全管理员权限。BizTalk 管理员不是 Azure 帐户持有人，因此无法访问任何结算信息。
<br/><br/>
<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=267577"> 在 Azure 管理门户中管理订阅和存储帐户</a>提供了详细信息。

</td>

</tr>

<tr>

<td>
Azure SQL Database

</td>

<td>
存储 Azure BizTalk 服务使用的表、视图和存储过程，包括跟踪数据。
<br/><br/>
创建 BizTalk 服务时，你可以使用现有的 Azure SQL Server、Azure SQL Database，或者自动创建新的服务器或数据库。
<br/><br/>
SQL Database 的规模是自动配置的。通常，默认的规模就足以满足 BizTalk 服务的需要。更改规模会影响定价。请参阅 <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=234930"> Azure SQL Database 中的帐户和计费</a>
<br/><br/>
<b>注意</b>
<br/>
<ul>
<li>当你创建新的 Azure SQL Server 和 Database 时，将自动启用 Azure 服务。BizTalk 服务要求启用 Azure 服务。</li>
<li>如果你在现有 Azure SQL Server 的基础上创建新的 Azure SQL Database，将不会更改服务器的防火墙规则。因此，可能不允许其他 Azure 服务访问服务器的数据库。</li>
</ul>
</td>

</tr>

<tr>

<td>
Azure Access Control 命名空间

</td>

<td>
向 Azure BizTalk 服务进行身份验证。当你从 Visual Studio 中部署 BizTalk 服务项目时，你进入此 Access Control 命名空间。当你创建 BizTalk 服务时，将自动创建 Access Control 命名空间。

</td>

</tr>
</p>
<tr>
<td>
Azure 存储帐户

</td>
<td>
提供对 BizTalk 服务使用的表、Blob 和队列的访问权限，以执行以下操作：

<ul>
<li>保存用于监视 BizTalk 服务的日志文件。监视输出也会显示在 Azure 管理门户的“监视”选项卡中。</li>
<li>在合作伙伴之间创建 X12 或 AS2 协议时，你可以启用“存档”功能来存储消息属性。此数据保存在该存储帐户中。</li>
</ul>
<br/>
 当你创建 BizTalk 服务时，可以使用现有的存储帐户，或者自动创建新的存储帐户。
<br/><br/>
 默认的存储设置就足以满足 BizTalk 服务的需要。
<br/><br/>
 当你创建存储帐户时，将自动创建主密钥和辅助密钥。这些密钥控制对存储帐户的访问。BizTalk 服务自动使用主密钥。
<br/><br/>
 有关详细信息，请参阅<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=285671">存储</a>。

</td>
</tr>
<tr>
<td>
SSL 专用证书

</td>
<td>
当你创建 Azure BizTalk 服务时，会同时创建包含你的 BizTalk 服务名称的 HTTPS URL。此 URL 已自动配置为使用仅供开发的自签名证书。对于生产目的，你需要使用私有 SSL 证书。
<br/><br/>
<b>重要的 SSL 证书信息</b>

<ul>
<li>证书到期日期必须低于 5 年。</li>
<li>所有私有证书都需要密码。你应该知道此密码，并且最好与管理员共享此密码。</li>
<li>自签名证书需在测试/开发环境中使用。使用自签名证书时，请将证书导入到你的“个人”证书存储和“受信任的根证书颁发机构”证书存储中。</li>
</ul>
<br/>将生产证书请求发送到证书颁发机构时，请提供以下证书属性：
<br/>

<ul>
<li><b>增强型密钥用法</b>：Azure BizTalk 服务要求至少进行服务器身份验证。</li>
<li><b>公用名</b>：请输入你的 Azure BizTalk 服务 URL 的完全限定域名 (FQDN)。请参阅本主题中的<a HREF="#BizTalk">创建 BizTalk 服务</a>。</li>
</ul>
<br/>
 创建 BizTalk 服务后，便可以添加新的或不同的证书。

</td>
</tr>
</table>
## <a name="HC"></a>混合连接

当你创建 Azure BizTalk 服务时，将会出现**“混合连接”**选项卡：

![混合连接选项卡][混合连接选项卡]

使用混合连接可将 Azure 网站连接到使用静态 TCP 端口的任何本地资源，例如 SQL Server、MySQL、HTTP Web API、移动服务和大多数自定义 Web 服务。混合连接不同于 BizTalk 适配器服务。BizTalk 适配器服务用于将 Azure BizTalk 服务连接到本地业务线 (LOB) 系统。

有关详细信息，包括如何创建和管理混合连接，请参阅[混合连接][混合连接]。

## 下一步

现在已创建 BizTalk 服务，可以在 [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡][“仪表板”、“监视”和“缩放”选项卡]中熟悉各个不同的选项卡了。BizTalk 服务已准备就绪，可用于你的应用程序了。若要开始创建应用程序，请转到 [Azure BizTalk 服务][Azure BizTalk 服务]。

## 另请参阅

-   [BizTalk 服务：版本图表][BizTalk 服务：版本图表]
-   [BizTalk 服务：状态图表][BizTalk 服务状态图表]
-   [BizTalk 服务：备份和还原][BizTalk 服务：备份和还原]
-   [BizTalk 服务：限制][BizTalk 服务：限制]
-   [BizTalk 服务：颁发者名称和颁发者密钥][BizTalk 服务：颁发者名称和颁发者密钥]
-   [如何开始使用 Azure BizTalk 服务 SDK][如何开始使用 Azure BizTalk 服务 SDK]
-   [混合连接][混合连接]

  [创建 BizTalk 服务]: #BizTalk
  [设置后的步骤]: #PostProv
  [要求说明]: #Requirements
  [混合连接 - 新增功能！]: #HC
  [Azure 免费试用]: http://go.microsoft.com/fwlink/p/?LinkID=239738
  [Azure 管理门户]: http://go.microsoft.com/fwlink/p/?LinkID=213885
  [选择“新建”按钮]: ./media/biztalk-provision-services/WABS_New.png
  [依次选择“BizTalk 服务”和“自定义创建”]: ./media/biztalk-provision-services/WABS_NewBizTalkService.png
  []: http://go.microsoft.com/fwlink/p/?LinkID=302279
  [完成时显示进度图标]: ./media/biztalk-provision-services/WABS_ProgressComplete.png
  [“仪表板”、“监视”和“缩放”选项卡]: http://go.microsoft.com/fwlink/p/?LinkID=302281
  [BizTalk 服务状态图表]: http://go.microsoft.com/fwlink/p/?LinkID=329870
  [添加生产就绪证书]: #AddCert
  [获取 Access Control 命名空间]: #ACS
  [修改 SSL 证书]: ./media/biztalk-provision-services/WABS_QuickGlance.png
  [选择连接信息]: ./media/biztalk-provision-services/WABS_ACSConnectInformation.png
  [Access Control 管理门户中的 ACS 服务标识]: ./media/biztalk-provision-services/WABS_ACSServiceIdentities.png
  [管理 ACS 命名空间]: http://go.microsoft.com/fwlink/p/?LinkID=285670
  [Azure 订阅]: https://account.windowsazure.com/Subscriptions
  [在 Azure 管理门户中管理订阅和存储帐户]: http://go.microsoft.com/fwlink/p/?LinkID=267577
  [Azure SQL Database 中的帐户和计费]: http://go.microsoft.com/fwlink/p/?LinkID=234930
  [存储]: http://go.microsoft.com/fwlink/p/?LinkID=285671
  [混合连接选项卡]: ./media/biztalk-provision-services/WABS_HybridConnectionTab.png
  [混合连接]: http://go.microsoft.com/fwlink/p/?LinkID=397274
  [Azure BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=235197
  [BizTalk 服务：备份和还原]: http://go.microsoft.com/fwlink/p/?LinkID=329873
  [BizTalk 服务：限制]: http://go.microsoft.com/fwlink/p/?LinkID=302282
  [BizTalk 服务：颁发者名称和颁发者密钥]: http://go.microsoft.com/fwlink/p/?LinkID=303941
  [如何开始使用 Azure BizTalk 服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=302335
  [BizTalk 服务：版本图表]: http://go.microsoft.com/fwlink/p/?LinkID=302279
