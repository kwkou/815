<?xml version="1.0" encoding="UTF-8"?>
<localize><umbTextpage id="19382" parentID="19381" level="3" writerID="52" creatorID="94" nodeType="1059" template="1052" sortOrder="1" createDate="2014-04-23T13:42:34" updateDate="2014-08-18T11:34:13" nodeName="scheduler" urlName="scheduler" writerName="uRest" creatorName="xunfan" path="-1,11978,19381,19382" isDoc=""><bodyText><![CDATA[<div style="margin-top: 40px;">
<h1>计划程序</h1>
<h2>在简单或复杂的定期计划程序上运行作业</h2>
<p>利用 Windows Azure 计划程序，您可按照任何计划调用操作，例如调用 HTTP/S 终结点或将消息发布到存储队列。利用计划程序，可在云中创建在 Windows Azure 内部和外部可靠调用服务的作业，并可按需或按定复计划运行这些作业或者为将来日期指定作业。该服务当前可用作独立 API。</p>
<p>利用计划程序，可以：</p>
<h3>通过 HTTP/s 调用 Web 服务</h3>
<p>利用计划程序，可以通过 HTTP/s 调用一次或按定期计划调用任何 Web 服务终结点。现在，有多个内部服务使用计划程序来支持各种方案，包括：</p>
<ul>
<li>全球可用的使用者 SaaS 应用程序依靠计划程序来调用按设定计划执行数据管理的 Web 服务</li>
<li>Windows Azure 移动服务利用计划程序推动其计划的脚本功能</li>
<li>另一项 Windows Azure 服务使用计划程序来定期调用执行诊断日志清理和数据聚合的 Web 服务</li>
</ul>
<h3>将消息发布到 Windows Azure 存储队列中</h3>
<p>还可使用计划程序将消息发布到存储队列中，并支持对定期请求的异步处理而无需支持 Web 服务。当您的作业可从队列中做好准备时，您可以解锁几个附加方案，即：</p>
<ul>
<li><strong>处理长时间运行的请求</strong> - 在特定时段内未接收到响应时，Http/s 请求将超时。对于复杂请求（例如，针对大型数据库的一系列 SQL 查询），通过将消息发布到存储队列中，可使您完成工作而无需将其他异步逻辑内置于 Web 服务中。</li>
<li><strong>允许在脱机状态下调用服务</strong> - 通常，当计划程序调用终结点时，Web 服务需要处于联机状态。不过，利用发布到存储队列中的消息，该服务可在计划程序发送消息时处于脱机状态，并在稍后联机时对请求做出响应。</li>
</ul>
<div class="section next-steps">
<h2>后续步骤</h2>
<p>查看定价详细信息。浏览文档中心以获取相关资源。了解灵活的购买选项。</p>
</div>
</div>]]></bodyText><umbracoNaviHide>0</umbracoNaviHide><pageTitle>Windows Azure 计划程序 | Windows Azure</pageTitle><metaKeywords></metaKeywords><metaDescription><![CDATA[]]></metaDescription><linkid>services-scheduler</linkid><urlDisplayName>计划程序</urlDisplayName><headerExpose></headerExpose><footerExpose></footerExpose><disqusComments>0</disqusComments><metaCanonical></metaCanonical><isHeader>0</isHeader><pageTemplate>services-template</pageTemplate><localize>0</localize><localizePartial>0</localizePartial><sitemapHide>0</sitemapHide><headerText><![CDATA[]]></headerText></umbTextpage></localize>