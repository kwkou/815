<properties linkid="dev-net-storage" urlDisplayName="Windows Azure 存储" pageTitle="Windows Azure 服务管理：存储" metaKeywords="Azure Storage" description="" metaCanonical="" services="Storage" documentationCenter="Services" title="Configure, monitor, and scale your storage account in Azure" authors="" solutions="" manager="" editor="" />

<h1>存储</h1>
<div class="wa-spacer wa-spacer-6down">
<h4>在 Windows Azure 中配置、监视和缩放存储帐户</h4>
<p>使用 Azure 存储服务存储和访问数据。使用 Blob 存储非结构化的二进制和文本数据。使用队列存储客户端可访问的消息。将非关系结构化数据存储在表中。</p>
<h4>快速链接</h4>
<ul class="wa-linkList">
<li><a href="/zh-cn/manage/services/storage/" class="wa-arrowLink-light">服务概述</a></li>
<li><a href="/zh-cn/solutions/data-management/" class="wa-arrowLink-light">可交付的解决方案</a></li>
<li><a href="/zh-cn/pricing/details/storage/" class="wa-arrowLink-light">定价详细信息</a></li>
</ul>
</div>
<div class="wa-spacer wa-spacer-asideLight wa-spacer-4down">
<h4>特色</h4>
<ul class="wa-iconList">
<li><a href="/zh-cn/documentation/articles/storage-introduction/"> Azure 存储简介</a></li>
  <li style="display:none"><a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-files/"> 通过文件存储创建 SMB 文件共享</a></li>
<li><a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/"> 将文件存储在 Blob 存储中</a></li>
  <li style="display:none"><a href="/zh-cn/documentation/articles/storage-monitoring-diagnosing-troubleshooting/"> 对存储进行监视、诊断和故障排除</a></li></ul>
</div>
<div class="section s2 tutorials">
<h2>教程和指南</h2>

  <div style="display:none" class="selector-wrap">
<div class="horizontal-option-selector tutorial-lang-selector">
<ul class="wa-tabs wa-tabsBlock" data-tab-panel="tab-panel" data-control="tabs">
<li><a class="wa-tab active" data-id="1" data-slug="net">.NET</a></li>
<li><a class="wa-tab" data-id="2" data-slug="node">Node.js</a></li>
<li><a class="wa-tab" data-id="3" data-slug="java">Java</a></li>
<li><a class="wa-tab" data-id="4" data-slug="php">PHP</a></li>
<li><a class="wa-tab" data-id="6" data-slug="ruby">Ruby</a></li>
<li><a class="wa-tab" data-id="5" data-slug="python">Python</a></li>
</ul>
<div class="paragraph-toggle"><span class="less selected">更少</span> <span class="more">更多</span></div>
</div>

<p> </p>
<h3 class="light-font">探究</h3>
<div data-tab-panel-id="tab-panel" class="wa-tabs-container">
<div class="article-group dotnet active">
<h4><a href="/zh-cn/documentation/articles/storage-introduction/">Azure 存储简介</a></h4>
<p>详细了解 Azure 存储的优势。</p>
<h4><a href="/zh-cn/documentation/articles/storage-whatis-account/">什么是存储帐户？</a></h4>
<p>了解 Azure 存储帐户的功能，包括复制、身份验证、度量和日志记录。</p>
<h4><a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/">将文件存储在 Blob 存储中</a></h4>
<p>以编程方式使用 Azure Blob 服务上载和下载 Blob、在容器中列出 Blob 和执行其他常见方案。</p>
<h4><a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/">使用表存储来存储结构化数据</a></h4>
<p>以编程方式使用 Azure 表服务创建表、建立分区、添加数据和查询数据。</p>
<h4><a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-queues/">使用队列存储来可靠存储和访问消息</a></h4>
<p>使用 Azure 队列服务创建队列、将消息插入队列中以及读取和处理消息。</p>
  <h4 style="display:none"><a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-files/">通过文件存储在 Azure 中创建一个 SMB 文件</a></h4>
<p>就像桌面应用程序会安装一个典型的 SMB 共享一样，在 Azure 虚拟机或云服务中运行的应用程序可安装一个文件存储共享来访问文件数据。在本教程中了解使用文件存储的基础知识。</p>
  <div style="display:none" class="horz-rule"></div>
  <h3 style="display:none" class="light-font">计划</h3>
  <h4 style="display:none"><a href="/zh-cn/documentation/articles/cloud-services-dotnet-multi-tier-app-storage-1-overview/">使用表、队列和 Blob 创建 .NET 多层应用程序</a></h4>
  <p style="display:none">创建使用表、队列和 Blob 的多层 ASP.NET MVC 4 Web 应用程序。将应用程序部署到 Azure 云服务。</p>
<h4><a href="/zh-cn/documentation/articles/azure-subscription-service-limits/">Azure 订阅和服务限制、配额和约束条件</a></h4>
<p>了解订阅、Web Workers、虚拟机、网络、存储以及 SQL 数据库最常见的 Microsoft Azure 限制。</p>
<div class="horz-rule"></div>
<h3 class="light-font">开发</h3>
<h4><a href="/zh-cn/documentation/articles/storage-use-store-apps/">如何在 Windows 应用商店应用程序中使用 Azure 存储</a></h4>
<p>使用 Azure Blob、队列和表可为 Windows 应用商店应用程序存储数据。</p>
<h4><a href="/zh-cn/documentation/articles/storage-dotnet-shared-access-signature-part-1/">如何使用共享访问签名授予对存储资源的访问权限</a></h4>
<p>了解如何使用共享访问签名来授予对您的存储帐户中的 Blob、队列和表的受限访问权限。本教程的<a href="/zh-cn/documentation/articles/storage-dotnet-shared-access-signature-part-1/">第 1 部分</a>说明了共享访问签名的重要概念并提供了最佳实践。<a href="/zh-cn/manage/services/storage/net/shared-access-signature-part-2/">第 2 部分</a>将逐步引导您完成为 Blob 资源生成共享访问签名的过程，然后演示如何从客户端应用程序使用这些签名。</p>
<div class="horz-rule"></div>
<h3 class="light-font">管理</h3>
<h4><a href="/zh-cn/documentation/articles/storage-create-storage-account/">创建存储帐户</a></h4>
<p>创建存储帐户，以便您能使用 Azure Blob、表和队列服务。</p>
<h4><a href="/zh-cn/documentation/articles/storage-manage-storage-account/">管理存储帐户</a></h4>
<p>管理如何复制存储帐户，查看、复制和重新生成存储访问密钥以及删除存储帐户。</p>
<h4><a href="/zh-cn/documentation/articles/storage-monitor-storage-account/">监视存储帐户</a></h4>
<p>在管理门户中自定义对存储帐户的监视，并配置日志记录以进行深入的故障排除。</p>
  <h4 style="display:none"><a href="/zh-cn/documentation/articles/storage-monitoring-diagnosing-troubleshooting/">对 Azure 存储进行监视、诊断和故障排除</a></h4>
  <p style="display:none">了解如何使用诸如存储分析、客户端日志记录之类的功能以及其他第三方工具来查明、诊断并解决您的解决方案中与 Azure 存储相关的问题。</p>
  <h4 style="display:none"><a href="/zh-cn/documentation/articles/storage-use-azcopy/">AZCopy 实用程序入门</a></h4>
  <p style="display:none">了解如何使用 AzCopy，AzCopy 是一个便捷的命令行实用程序，用于将操作上载、下载和复制到 Azure 存储中。</p>
<h4><a href="/zh-cn/documentation/articles/storage-custom-domain-name/">配置自定义域名</a></h4>
<p>配置自定义域以便访问 Azure 存储帐户中的 Blob 数据。</p>

  <h4 style="display:none"><a href="/zh-cn/documentation/articles/storage-import-export-service/">使用 Azure 导入/导出服务可将数据传输到 Blob 存储中</a></h4>
  <p style="display:none">Azure 导入/导出服务可高效地将大量文件数据传输到 Azure Blob 存储中，以及从 Blob 存储传输到您的本地安装中。</p>
</div>
<div class="article-group nodejs">
<h4><a href="/zh-cn/documentation/articles/storage-introduction/">Azure 存储简介</a></h4>
<p>详细了解 Azure 存储的优势。</p>
<h4><a href="/zh-cn/documentation/articles/storage-whatis-account/">什么是存储帐户？</a></h4>
<p>了解 Azure 存储帐户的功能，包括复制、身份验证、度量和日志记录。</p>
<h4><a href="/zh-cn/documentation/articles/storage-nodejs-how-to-use-blob-storage/">使用 Blob 服务管理文件数据</a></h4>
<p>以编程方式使用 Azure Blob 服务上载和下载 Blob、在容器中列出 Blob 和执行其他常见方案。</p>
<h4><a href="/zh-cn/documentation/articles/storage-nodejs-how-to-use-table-storage/">使用表存储来存储结构化数据</a></h4>
<p>以编程方式使用 Azure 表服务创建表、建立分区、添加数据和查询数据。</p>
<h4><a href="/zh-cn/documentation/articles/storage-nodejs-how-to-use-queues/">使用队列存储来可靠存储和访问消息</a></h4>
<p>使用 Azure 队列服务创建队列、将消息插入队列中以及读取和处理消息。</p>
<h4><a href="/zh-cn/documentation/articles/storage-nodejs-use-table-storage-web-site/">使用表服务创建任务管理应用程序</a></h4>
<p>创建允许创建、检索和完成任务的 Node.js 应用程序。使用表服务来存储和访问 Azure 上托管的应用程序中的数据。</p>

<div class="horz-rule"></div>
<h3 class="light-font">计划</h3>
<h4><a href="/zh-cn/documentation/articles/azure-subscription-service-limits/">Azure 订阅和服务限制、配额和约束条件</a></h4>
<p>了解订阅、Web Workers、虚拟机、网络、存储以及 SQL 数据库最常见的 Microsoft Azure 限制。</p>

<div class="horz-rule"></div>
</div>
</div>
</div>
<div  style="display:none" class="section s3 light-grey video-gallery">
<h2>视频</h2>
<div class="video-wrapper"></div>
<div class="video-blocks">
<div class="vid-block vb1"><a href="http://channel9.msdn.com/Events/Build/2012/4-004/" class="fix-vid"> <span class="thumb"> <span class="triangle"> </span> <span class="play-btn"> </span> </span> <span class="desc">Azure Storage: Building applications that scale</span> </a></div>
<div class="vid-block vb2"><a href="http://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/WAD-B406/" class="fix-vid"> <span class="thumb"> <span class="triangle"> </span> <span class="play-btn"> </span> </span> <span class="desc">Get the most out of Azure Storage</span> </a></div>
<div class="vid-block vb3"><a href="http://channel9.msdn.com/Events/windowsazure/meet2012sf/Windows-Azure-Storage-Introduction/" class="fix-vid"> <span class="thumb"> <span class="triangle"> </span> <span class="play-btn"> </span> </span> <span class="desc">Introduction to Azure Storage</span> </a></div>
<div class="vid-block vb4"><a href="http://channel9.msdn.com/Shows/Visual-Studio-Toolbox/New-Tools-for-Azure-Storage-and-Diagnostics/"> <span class="thumb"> <span class="triangle"> </span> <span class="play-btn"> </span> </span> <span class="desc">New tools for Azure Storage and diagnostics</span> </a></div>
</div>
<a href="/en-us/documentation/videos/index/?services=storage" class="wa-arrowLinkLarge-dark">View more 存储 videos</a></div>
<div class="wa-content">
<h2>寻找更多资源？</h2>
<div class="wa-resourceBlockRow"><a href="http://social.msdn.microsoft.com/Forums/windowsazure/en-US/home?forum=windowsazuredata" class="wa-resourceBlock"><span class="wa-resourceBlock-header">论坛</span>询问问题、分享见解和讨论平台</a> <a href="http://msdn.microsoft.com/en-us/library/dd179355.aspx" class="wa-resourceBlock"><span class="wa-resourceBlock-header">参考</span>Azure 存储服务 REST API 参考</a> <a href="http://code.msdn.microsoft.com/windowsazure" class="wa-resourceBlock"><span class="wa-resourceBlock-header">示例</span>浏览可下载的示例应用程序</a> <a href="/zh-cn/downloads/?sdk=net" class="wa-resourceBlock"><span class="wa-resourceBlock-header">下载</span>下载用于配置 Azure 的脚本</a><a href="http://msdn.microsoft.com/zh-cn/library/gg433040.aspx" class="wa-resourceBlock"><span class="wa-resourceBlock-header">MSDN参考文档</span>从MSDN上获取有帮助的资源</a></div>
</div>

  <div style="display:none" class="wa-content">
<div class="footer-map">
<ul class="social">
<li class="header">社会化</li>
<li class="facebook"><a href="http://go.microsoft.com/fwlink/?LinkId=306390">Facebook</a></li>
<li class="twitter"><a href="http://go.microsoft.com/fwlink/?LinkID=306391">Twitter</a></li>
<li class="rss"><a href="http://azure.microsoft.com/blog/feed/">Rss</a></li>
<li class="newsletter"><a href="http://go.microsoft.com/fwlink/?LinkId=306393">新闻稿</a></li>
</ul>
<ul>
<li class="header"><a href="/zh-cn/">Microsoft Azure</a></li>
<li><a href="/zh-cn/solutions/">功能</a></li>
<li><a href="/zh-cn/services/">服务</a></li>
<li><a href="/zh-cn/regions/">区域</a></li>
<li><a href="/zh-cn/case-studies/">案例研究</a></li>
<li><a href="/zh-cn/pricing/overview/">价格</a></li>
<li><a href="/zh-cn/pricing/calculator/">计算器</a></li>
<li><a href="/zh-cn/documentation/">文档</a></li>
<li><a href="/zh-cn/downloads/?sdk=net">下载</a></li>
<li><a href="/zh-cn/gallery/">库</a></li>
<li><a href="http://windowsazure.cn/zh-cn/">Microsoft Azure (中国)</a></li>
</ul>
<ul>
<li class="header"><a href="http://go.microsoft.com/fwlink/?LinkId=394285">社区</a></li>
<li><a href="/blog/">博客</a></li>
<li><a href="/zh-cn/updates/">服务更新</a></li>
<li><a href="/zh-cn/support/forums/">论坛</a></li>
<li><a href="/zh-cn/community/events/">活动</a><br /><br /></li>
<li class="header"><a href="/zh-cn/support/options/">支持</a></li>
<li><a href="/zh-cn/support/forums/">论坛</a></li>
<li><a href="http://status.azure.com">服务仪表板</a></li>
<li><a href="/zh-cn/support/options/">支持</a></li>
</ul>
<ul>
<li class="header"><a href="https://account.windowsazure.com">帐户</a></li>
<li><a href="https://account.windowsazure.com/subscriptions/">订阅</a></li>
<li><a href="https://account.windowsazure.com/profile/">配置文件</a></li>
<li><a href="/zh-cn/services/preview/">预览功能</a></li>
<li><a href="https://manage.windowsazure.com/">管理门户</a></li>
<li class="header trust-center"><a href="/zh-cn/support/trust-center/">信任中心</a></li>
<li><a href="/zh-cn/support/trust-center/security/">安全性</a></li>
<li><a href="/zh-cn/support/trust-center/privacy/">隐私</a></li>
<li><a href="/zh-cn/support/trust-center/compliance/">合规</a></li>
</ul>
</div>
<div class="footer-bottom">
<ul>
<li class="hello-note">来自西雅图的问候。</li>
<li>
<div class="languages-selector">
<div class="locale-selector"><a class="current-locale">中文(简体) </a>
<div class="locale-selection-panel site-flag site-flag-lang">
<div class="content">
<table border="0" cellspacing="0" class="all-locales" style="border-collapse: collapse;">
<tbody>
<tr>
<td><a href="#" title="English" class="locale-link " data-loc="en-us">English</a></td>
<td><a href="#" title="Čeština" class="locale-link " data-loc="cs-cz">Čeština</a></td>
<td><a href="#" title="Dansk" class="locale-link " data-loc="da-dk">Dansk</a></td>
<td><a href="#" title="Deutsch" class="locale-link " data-loc="de-de">Deutsch</a></td>
</tr>
<tr>
<td><a href="#" title="English (India)" class="locale-link " data-loc="en-in">English (India)</a></td>
<td><a href="#" title="English (UK)" class="locale-link " data-loc="en-gb">English (UK)</a></td>
<td><a href="#" title="Español" class="locale-link " data-loc="es-es">Español</a></td>
<td><a href="#" title="Finnish" class="locale-link " data-loc="fi-fi">Finnish</a></td>
</tr>
<tr>
<td><a href="#" title="Français" class="locale-link " data-loc="fr-fr">Français</a></td>
<td><a href="#" title="Greek" class="locale-link " data-loc="el-gr">Greek</a></td>
<td><a href="#" title="Italiano" class="locale-link " data-loc="it-it">Italiano</a></td>
<td><a href="#" title="Magyar" class="locale-link " data-loc="hu-hu">Magyar</a></td>
</tr>
<tr>
<td><a href="#" title="Nederlands" class="locale-link " data-loc="nl-nl">Nederlands</a></td>
<td><a href="#" title="Norwegian" class="locale-link " data-loc="nb-no">Norwegian</a></td>
<td><a href="#" title="Polski" class="locale-link " data-loc="pl-pl">Polski</a></td>
<td><a href="#" title="Português (BR)" class="locale-link " data-loc="pt-br">Português (BR)</a></td>
</tr>
<tr>
<td><a href="#" title="Português (PT)" class="locale-link " data-loc="pt-pt">Português (PT)</a></td>
<td><a href="#" title="Svenska" class="locale-link " data-loc="sv-se">Svenska</a></td>
<td><a href="#" title="Romanian" class="locale-link " data-loc="ro-ro">Romanian</a></td>
<td><a href="#" title="Türkçe" class="locale-link " data-loc="tr-tr">Türkçe</a></td>
</tr>
<tr>
<td><a href="#" title="Ukrainian" class="locale-link " data-loc="uk-ua">Ukrainian</a></td>
<td><a href="#" title="русский" class="locale-link " data-loc="ru-ru">русский</a></td>
<td><a href="#" title="日本語" class="locale-link " data-loc="ja-jp">日本語</a></td>
<td><a href="#" title="한국어" class="locale-link " data-loc="ko-kr">한국어</a></td>
</tr>
<tr>
<td><a href="#" title="中文(简体)" class="locale-link selected" data-loc="zh-cn">中文(简体)</a></td>
<td><a href="#" title="中文(繁體)" class="locale-link " data-loc="zh-tw">中文(繁體)</a></td>
</tr>
</tbody>
</table>
<div class="arrow"> </div>
</div>
</div>
</div>
</div>
</li>
<li>
<div class="currencies-selector">
<div class="default-currency-selector"><a class="current-default-currency">USD </a>
<div class="currency-selection-panel site-flag site-flag-lang">
<div class="content">
<table border="0" cellspacing="0" class="all-currencies" style="border-collapse: collapse;">
<tbody>
<tr>
<td><a href="#" title="美元 ($)" class="default-currency-link selected" data-curr="USD">美元 ($)</a></td>
<td><a href="#" title="加拿大元 ($)" class="default-currency-link " data-curr="CAD">加拿大元 ($)</a></td>
<td><a href="#" title="英镑 (£)" class="default-currency-link " data-curr="GBP">英镑 (£)</a></td>
<td><a href="#" title="丹麦克朗 (kr)" class="default-currency-link " data-curr="DKK">丹麦克朗 (kr)</a></td>
</tr>
<tr>
<td><a href="#" title="欧元 (€)" class="default-currency-link " data-curr="EUR">欧元 (€)</a></td>
<td><a href="#" title="挪威克朗 (kr)" class="default-currency-link " data-curr="NOK">挪威克朗 (kr)</a></td>
<td><a href="#" title="瑞典克朗 (kr)" class="default-currency-link " data-curr="SEK">瑞典克朗 (kr)</a></td>
<td><a href="#" title="瑞士法郎 (chf)" class="default-currency-link " data-curr="CHF">瑞士法郎 (chf)</a></td>
</tr>
<tr>
<td><a href="#" title="日元 (¥)" class="default-currency-link " data-curr="JPY">日元 (¥)</a></td>
<td><a href="#" title="澳大利亚元 ($)" class="default-currency-link " data-curr="AUD">澳大利亚元 ($)</a></td>
<td><a href="#" title="新西兰元 ($)" class="default-currency-link " data-curr="NZD">新西兰元 ($)</a></td>
<td><a href="#" title="韩元 (₩)" class="default-currency-link " data-curr="KRW">韩元 (₩)</a></td>
</tr>
<tr>
<td><a href="#" title="俄罗斯卢布 (руб)" class="default-currency-link " data-curr="RUB">俄罗斯卢布 (руб)</a></td>
<td><a href="#" title="南非兰特 (R)" class="default-currency-link " data-curr="ZAR">南非兰特 (R)</a></td>
<td><a href="#" title="土耳其里拉 (TL)" class="default-currency-link " data-curr="TRY">土耳其里拉 (TL)</a></td>
<td><a href="#" title="沙特里亚尔 (R)" class="default-currency-link " data-curr="SAR">沙特里亚尔 (R)</a></td>
</tr>
<tr>
<td><a href="#" title="阿根廷比索 ($)" class="default-currency-link " data-curr="ARS">阿根廷比索 ($)</a></td>
<td><a href="#" title="巴西雷亚尔 (R$)" class="default-currency-link " data-curr="BRL">巴西雷亚尔 (R$)</a></td>
<td><a href="#" title="印度卢比 (₹)" class="default-currency-link " data-curr="INR">印度卢比 (₹)</a></td>
<td><a href="#" title="港元 (HK$)" class="default-currency-link " data-curr="HKD">港元 (HK$)</a></td>
</tr>
<tr>
<td><a href="#" title="墨西哥比索 (MXN$)" class="default-currency-link " data-curr="MXN">墨西哥比索 (MXN$)</a></td>
<td><a href="#" title="马来西亚林吉特 ($)" class="default-currency-link " data-curr="MYR">马来西亚林吉特 ($)</a></td>
<td><a href="#" title="印度尼西亚卢比 (Rp)" class="default-currency-link " data-curr="IDR">印度尼西亚卢比 (Rp)</a></td>
<td><a href="#" title="台币 ($)" class="default-currency-link " data-curr="TWD">台币 ($)</a></td>
</tr>
</tbody>
</table>
<div class="arrow"> </div>
</div>
</div>
</div>
</div>
</li>
</ul>
<ul class="legal">
<li class="german-legal" style="display: none;"><a href="http://www.microsoft.com/germany/siteservices/impressum/copyright.mspx?CM=Foot">Nutzungsbedingungen</a></li>
<li class="german-legal" style="display: none;"><a href="http://www.microsoft.com/germany/siteservices/impressum/default.mspx">Impressum</a></li>
<li><a href="http://support.microsoft.com/contactus/?ws=mscom">与我们联系</a></li>
<li><a href="http://go.microsoft.com/fwlink/p/?linkid=222682">商标</a></li>
<li><a href="http://go.microsoft.com/fwlink/p/?linkid=131004">隐私和 Cookie</a></li>
<li class="loc-hide"><a href="http://feedback.azure.com/">反馈</a></li>
</ul>
<div class="footer-copyright"><a href="http://www.microsoft.com" class="logo-microsoft">Microsoft</a> <span>© 2014 Microsoft</span></div>
</div>
</div>
