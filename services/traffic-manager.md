<?xml version="1.0" encoding="UTF-8"?>
<localize><TextpageLeftNav id="19269" parentID="19263" level="5" writerID="104" creatorID="94" nodeType="1137" template="1136" sortOrder="6" createDate="2013-07-11T12:12:10" updateDate="2014-09-17T14:13:34" nodeName="Traffic Manager" urlName="traffic-manager" writerName="kwkou" creatorName="xunfan" path="-1,11978,12083,18409,19263,19269" isDoc=""><bodyText><![CDATA[<div>
<h1>如何配置 Traffic Manager 设置</h1>
<p>利用 Windows Azure Traffic Manager，您可以控制分发给 Windows Azure 托管服务的用户流量。</p>
<p>Traffic Manager 的工作方式是，向针对公司主域名的 DNS 查询应用智能策略引擎。更新您公司拥有的 DNS 资源记录以指向 Traffic Manager 域。随后，附加到这些域的 Traffic Manager 策略会将针对您的公司主域名的 DNS 查询解析为 Traffic Manager 策略中包含的特定 Windows Azure 托管服务的 IP 地址。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/hh744833.aspx">Windows Azure Traffic Manager 概述（可能为英文页面）</a>。</p>
<h2>目录</h2>
<ul>
<li><a href="#howto_point_company">如何：将公司 Internet 域指向 Traffic Manager 域</a></li>
<li><a href="#howto_test">如何：测试策略</a></li>
<li><a href="#howto_temp_disable">如何：临时禁用策略和托管服务</a></li>
<li><a href="#howto_edit_policy">如何：编辑策略</a></li>
<li><a href="#howto_load_balance">如何：在一组托管服务中均等平衡负载流量</a></li>
<li><a href="#howto_create_failover">如何：创建故障转移策略</a></li>
<li><a href="#howto_direct">如何：基于网络性能将传入流量定向到托管服务</a></li>
</ul>
<h2><a id="howto_point_company"></a>如何：将公司 Internet 域指向 Traffic Manager 域</h2>
<p>若要将公司域名指向 Traffic Manager 域，请使用 CNAME 编辑 DNS 服务器上的 DNS 资源记录。</p>
<p>例如，若要将公司主域 <span class="Apple-style-span">www.contoso.com</span> 指向名为 <span class="Apple-style-span">contoso.trafficmanager.cn</span> 的 Traffic Manager 域，请更新 DNS 资源记录，如下所示：<code>www.contoso.com IN CNAME contoso.trafficmanager.cn</code></p>
<p>此时，转到 <span class="Apple-style-span">www.contoso.com</span> 的所有流量将重定向到 <span class="Apple-style-span">contoso.trafficmanager.net</span>。请确保您使用的是希望将其中的所有流量重定向到 Traffic Manager 的域。</p>
<h2><a id="howto_test"></a>如何：测试策略</h2>
<p>测试策略的最佳方式是设置大量客户端，然后将策略中的服务一次关闭一个。下列提示将帮助您测试 Traffic Manager 策略：</p>
<ul>
<li>
<p>将 DNS TTL 设置得非常低，以便快速传播更改（例如 30 秒）。</p>
</li>
<li>
<p>知道要测试的策略中的 Windows Azure 托管服务的 IP 地址。您可以从 Windows Azure 管理门户中获取此信息。单击服务的“生产部署”。在右侧的属性窗格中，最后一个条目将是 VIP，它是该托管服务的虚拟 IP 地址。</p>
</li>
</ul>
<p><img src="/media/itpro/services/hosted_service_ip_location.png" alt="托管服务 IP 位置"/></p>
<p><span class="Apple-style-span">图 1</span> - 托管服务 IP 位置</p>
<ul>
<li>
<p>使用能够让您将 DNS 名称解析为 IP 地址并显示该地址的工具。您将检查您的公司域名是否解析为策略中的托管服务的 IP 地址。它们应通过与 Traffic Manager 策略的负载平衡方法一致的方式进行解析。如果您使用的是 Windows 系统，则可在 CMD 窗口中使用 nslookup（下面将介绍）。此外，可以从 Internet 轻松获得允许您“挖掘”IP 地址的其他公用工具。</p>
</li>
<li>
<p>使用 <span class="Apple-style-span">nslookup</span> 检查 Traffic Manager 策略。</p>
</li>
</ul>
<blockquote>
<p><span class="Apple-style-span">使用 nslookup 检查 Traffic Manager 策略</span></p>
<ol>
<li>
<p>单击“开始”-&gt;“运行”并键入“CMD”以启动 CMD 窗口。</p>
</li>
<li>
<p>键入 <code>ipconfig /flushdns</code> 以便刷新 DNS 解析器缓存。</p>
</li>
<li>
<p>键入命令 <code>nslookup &lt;您的 Traffic Manager 域&gt;</code>。例如，以下命令将检查前缀为 <span class="Apple-style-span">myapp.contoso</span><code>nslookup myapp.contoso.trafficmanager.cn</code> 的域</p>
</li>
</ol>
<blockquote>
<p>典型结果将显示以下内容：</p>
<ul>
<li>
<p>要访问以解析此 Traffic Manager 域的 DNS 服务器的 DNS 名称和 IP 地址。</p>
</li>
<li>
<p>您在命令行中的“nslookup”后面键入的 Traffic Manager 域名以及 Traffic Manager 域将解析为的 IP 地址。需要重点检查第二个 IP 地址。该地址应与要测试的 Traffic Manager 策略中的某个托管服务的 VIP 匹配。</p>
</li>
</ul>
<p><img src="/media/itpro/services/nslookup_command_example.png" alt="nslookup 命令示例"/></p>
<p><span class="Apple-style-span">图 2</span> – nslookup 命令示例</p>
</blockquote>
</blockquote>
<ul>
<li><span class="Apple-style-span">使用下面列出的其他测试方法之一。</span> 选择适用于要测试的负载平衡类型的方法。</li>
</ul>
<h3>性能策略</h3>
<p>您将需要借助全球各个位置的客户端才能有效测试您的域。您可以在 Windows Azure 中创建将尝试通过您的公司域名调用您的服务的客户端。或者，如果您的公司是一家跨国公司，则可远程登录位于国家/地区的其他位置的客户端并从这些客户端进行测试。</p>
<p>可免费获得基于 Web 的 nslookup 和挖掘服务。借助其中的某些服务，您可以从不同的位置检查 DNS 名称解析。例如，对 nslookup 执行搜索。另一个选择是使用第三方解决方案（如 Gomez 或 Keynote）来确认您的策略正在按预期方式分发流量。</p>
<h3>故障转移策略</h3>
<ol>
<li>
<p>将所有服务保持运行。</p>
</li>
<li>
<p>使用单一客户端。</p>
</li>
<li>
<p>使用 ping 或类似实用工具请求适用于您的公司域的 DNS 解析。</p>
</li>
<li>
<p>确保您获取的 IP 地址适用于您的主托管服务。</p>
</li>
<li>
<p>关闭您的主服务或删除正在监视的文件，以使 Traffic Manager 认为其已关闭。</p>
</li>
<li>
<p>至少等待 2 分钟。</p>
</li>
<li>
<p>请求 DNS 解析。</p>
</li>
<li>
<p>确保您获取的 IP 地址适用于您的辅助托管服务，并按照服务在“编辑 Traffic Manager 策略”对话框中的顺序显示。</p>
</li>
<li>
<p>重复此过程，将关闭辅助服务，然后关闭第三位服务，依此类推。每次都请确保 DNS 解析返回列表中下一个服务的 IP 地址。关闭所有服务后，您应重新获取主托管服务的 IP 地址。</p>
</li>
</ol>
<h3>循环法策略</h3>
<ol>
<li>
<p>将所有服务保持运行。</p>
</li>
<li>
<p>使用单一客户端。</p>
</li>
<li>
<p>请求适用于顶级域的 DNS 解析。</p>
</li>
<li>
<p>确保您获取的 IP 地址是列表中的某个地址。</p>
</li>
<li>
<p>重复此过程以让 DNS TTL 到期，这样您便能收到新的 IP 地址。您应看到为每个托管服务返回的 IP 地址。随后，此过程将重复。</p>
</li>
</ol>
<h2><a id="howto_temp_disable"></a>如何：临时禁用策略和托管服务</h2>
<p>您可以禁用之前创建的 Traffic Manager 策略，使其不再路由流量。禁用 Traffic Manager 策略后，该策略的信息将在 Traffic Manager 界面中保持不变且可编辑。可以轻松重新启用策略，并且路由将恢复。</p>
<p>也可以禁用属于 Traffic Manager 策略的一部分的各个托管服务。虽然此操作可有效地将托管服务作为策略的一部分保留，但策略的工作方式就像它未包含托管服务时一样。对于临时删除处于维护模式中或要重新部署且很不稳定的托管服务，此操作很有用。在托管服务重新启动并运行后，可重新启用它。</p>
<p><span class="Apple-style-span">注意</span> <br /> 禁用托管服务与其在 Windows Azure 中的部署状态无关。正常的服务将保持运行并能够接收流量（即使在 Traffic Manager 中已禁用这一行为）。此外，在一个策略中禁用某个托管服务不会影响该托管服务在其他策略中的状态。</p>
<h3>禁用策略</h3>
<ol>
<li>
<p>在 Traffic Manager 界面树中选择已启用的策略。</p>
</li>
<li>
<p>单击顶部工具栏中的“禁用策略”。请注意，如果您突出显示已禁用的策略，则此按钮将灰显。</p>
</li>
</ol>
<h3>启用策略</h3>
<ol>
<li>
<p>在 Traffic Manager 界面树中选择已禁用的策略。</p>
</li>
<li>
<p>单击顶部工具栏中的“启用策略”。请注意，如果您突出显示已禁用的策略，则此按钮将灰显。</p>
</li>
</ol>
<h3>禁用托管服务</h3>
<ol>
<li>
<p>在 Traffic Manager 界面中选择策略中已启用的托管服务。您必须展开策略才能看到其中包含的托管服务。</p>
</li>
<li>
<p>单击顶部工具栏中的“禁用服务策略”。请注意，如果您突出显示已禁用的托管服务，则此按钮将灰显。</p>
</li>
<li>
<p>流量将基于为作为策略的一部分的 Traffic Manager 域设置的 DNS TTL 时间停止流入托管服务。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“在 Windows Azure Traffic Manager 中监视托管服务”一节。</p>
</li>
</ol>
<h3>启用托管服务</h3>
<ol>
<li>
<p>在 Traffic Manager 界面中选择策略中已禁用的托管服务。您必须展开策略才能看到其中包含的托管服务。</p>
</li>
<li>
<p>单击顶部工具栏中的“启用服务策略”。请注意，如果您突出显示已启用的托管服务，则此按钮将灰显。</p>
</li>
<li>
<p>流量将重新开始流入策略所指定的托管服务。</p>
</li>
</ol>
<h2><a id="howto_edit_policy"></a>如何：编辑策略</h2>
<p>如果您只是需要临时禁用策略或策略中的特定托管服务，则可在不更改策略的情况下临时禁用它们。</p>
<p>若要将策略更改为其他类型，请执行下列步骤：</p>
<ol>
<li>
<p><span class="Apple-style-span">登录管理门户中的“Traffic Manager”区域</span>，网址为 <a href="http://manage.windowsazure.cn">http://manage.windowsazure.cn</a>。单击门户页左下角的“虚拟网络”，然后从左侧窗格中的选项中选择“Traffic Manager”。</p>
</li>
<li>
<p>在管理门户的“Traffic Manager”屏幕中选择要更改的策略。</p>
</li>
<li>
<p>单击顶部菜单栏上的“配置”。</p>
</li>
<li>
<p><span class="Apple-style-span">对策略进行所需更改。</span>请注意，如果您更改负载平衡方法，则“选定托管服务”列表中的托管服务的顺序可能重要或不重要，具体取决于所选的选项。</p>
</li>
<li>
<p>单击“保存”。</p>
</li>
</ol>
<h2><a id="howto_load_balance"></a>如何：在一组托管服务中均等平衡负载流量</h2>
<p>常见负载平衡模式旨在提供一组相同的托管服务并通过循环法将流量发送到每个托管服务。本文概述了设置 Traffic Manager 域和策略以执行此类负载平衡的步骤。</p>
<p>有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/hh744833.aspx">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“Windows Azure Traffic Manager 中的负载平衡方法”一节。</p>
<ol>
<li>
<p>将托管服务部署到生产环境中。有关开发和部署托管服务的信息，请参阅 <a href="http://msdn.microsoft.com/library/gg432967.aspx">Windows Azure 托管服务（可能为英文页面）</a>。</p>
</li>
<li>
<p><span class="Apple-style-span">登录管理门户中的“Traffic Manager”区域</span>，网址为 <a href="http://manage.windowsazure.cn">http://manage.windowsazure.cn</a>。单击门户页左下角的“虚拟网络”，然后从左侧窗格中的选项中选择“Traffic Manager”。</p>
</li>
<li>
<p><span class="Apple-style-span">选择“策略”并单击“创建”。</span>从左侧导航树中选择“策略”文件夹以启用顶部工具栏中的“创建”。选择“创建”。这将显示“创建 Traffic Manager 策略”对话框。</p>
<p><img src="/media/itpro/services/create_button_for_policies.png" alt="针对策略的“创建”按钮"/></p>
<p><span class="Apple-style-span">图 1</span> - 针对策略的“创建”按钮</p>
</li>
<li>
<p><span class="Apple-style-span">选择订阅。</span> 策略和域将与单个订阅关联。</p>
</li>
<li>
<p><span class="Apple-style-span">选择“循环法”负载平衡方法。</span>有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/hh744833.aspx">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“Windows Azure Traffic Manager 中的负载平衡方法”一节。</p>
</li>
<li>
<p><span class="Apple-style-span">查找托管服务并将其添加到策略中。</span>使用筛选器框查找包含您在该框中键入的字符串的托管服务。清除该框可显示适用于在步骤 4 中选择的订阅的生产中的所有托管服务。使用这些箭头按钮可将托管服务添加到策略中。“选定 DNS 名称”框中的顺序与此负载平衡方法无关。</p>
</li>
<li>
<p><span class="Apple-style-span">设置监视。</span>通过监视来确保不会向处于脱机状态的托管服务发送流量。您必须已设置特定的路径和文件名。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“在 Windows Azure Traffic Manager 中监视托管服务”一节。</p>
</li>
<li>
<p><span class="Apple-style-span">命名 Traffic Manager 域。</span>为您的域指定一个唯一名称。您只能指定域的前缀。将 DNS 生存时间保留为其默认时间。有关此设置的影响的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“使用 Windows Azure Traffic Manager 时托管服务和策略的最佳实践”一节。</p>
</li>
</ol>
<p>“创建 Traffic Manager 策略”对话框应与以下示例类似。</p>
<p><img src="/media/itpro/services/dialog_box_for_round_robin_load_balancing_method.png" alt="“循环法”负载平衡方法的对话框"/></p>
<p><span class="Apple-style-span">图 2</span> –“循环法”负载平衡方法的对话框</p>
<ol>
<li>
<p><span class="Apple-style-span">测试 Traffic Manager 域和策略。</span>有关说明，请参阅<a href="#howto_test">如何：测试 Windows Azure Traffic Manager 策略</a>。</p>
</li>
<li>
<p><span class="Apple-style-span">将 DNS 服务器指向 Traffic Manager 域。</span>在设置并运行 Traffic Manager 域后，请编辑权威 DNS 服务器上的 DNS 记录以将公司域指向 Traffic Manager 域。例如，以下命令会将转到 <span class="Apple-style-span">www.contoso.com</span> 的所有流量路由至 Traffic Manager 域 <span class="Apple-style-span">contoso.trafficmanager.cn</span>：<code>www.contoso.com IN CNAME contoso.trafficmanager.cn</code> 有关详细信息，请参阅<a href="#howto_point_company">如何：将公司 Internet 域指向 Windows Azure Traffic Manager 域</a>。</p>
</li>
</ol>
<h2><a id="howto_create_failover"></a>如何：创建故障转移策略</h2>
<p>通常，组织希望为其服务提供可靠性。如果组织的主服务已关闭，则组织可通过提供备份服务来做到这一点。服务故障转移的常见模式是提供一组相同的托管服务并向主服务发送流量，并提供一个包含 1 个或多个备份的列表。本文概述了设置 Traffic Manager 策略以执行此类故障转移备份的步骤。</p>
<p>有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/hh744833.aspx">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“Windows Azure Traffic Manager 中的负载平衡方法”一节。</p>
<ol>
<li>
<p>将托管服务部署到生产环境中。有关开发和部署托管服务的信息，请参阅 <a href="http://msdn.microsoft.com/library/gg432967.aspx">Windows Azure 托管服务（可能为英文页面）</a>。</p>
</li>
<li>
<p><span class="Apple-style-span">登录管理门户中的“Traffic Manager”区域</span>，网址为 <a href="http://manage.windowsazure.cn">http://manage.windowsazure.cn</a>。单击门户页左下角的“虚拟网络”，然后从左侧窗格中的选项中选择“Traffic Manager”。</p>
</li>
<li>
<p><span class="Apple-style-span">选择“策略”并单击“创建”。</span>从左侧导航树中选择“策略”文件夹以启用顶部工具栏中的“创建”。选择“创建”。这将显示“创建 Traffic Manager 策略”对话框。</p>
<p><img src="/media/itpro/services/create_button_for_policies.png" alt="针对策略的“创建”按钮"/></p>
<p><span class="Apple-style-span">图 1</span> - 针对策略的“创建”按钮</p>
</li>
<li>
<p><span class="Apple-style-span">选择订阅。</span> 策略和域将与单个订阅关联。</p>
</li>
<li>
<p><span class="Apple-style-span">选择“故障转移策略”负载平衡方法。</span>有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/hh744833.aspx">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“Windows Azure Traffic Manager 中的负载平衡方法”一节。</p>
</li>
<li>
<p><span class="Apple-style-span">查找托管服务并将其添加到策略中。</span>使用筛选器框查找包含您在该框中键入的字符串的托管服务。清除该框可显示适用于在步骤 4 中选择的订阅的生产中的所有托管服务。使用这些箭头按钮可将托管服务添加到策略中。如果选择“故障转移”负载平衡方法，则选定服务的顺序很重要。主托管服务位于顶部。根据需要使用向上和向下箭头更改顺序。</p>
</li>
<li>
<p>通过监视来确保不会向处于脱机状态的托管服务发送流量。在不设置监视的情况下使用故障转移策略是不起作用的，因为流量将发送到主托管服务，即使该托管服务显示为脱机。若要监视托管服务，您必须指定特定的路径和文件名。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“在 Windows Azure Traffic Manager 中监视托管服务”一节。</p>
</li>
<li>
<p><span class="Apple-style-span">命名 Traffic Manager 域。</span>为您的域指定一个唯一名称。您只能指定域的前缀。将“DNS 生存时间(TTL)”保留为其默认时间。有关此设置的影响的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“使用 Windows Azure Traffic Manager 时托管服务和策略的最佳实践”一节。</p>
</li>
</ol>
<p>“创建 Traffic Manager 策略”对话框应与以下示例类似。</p>
<p><img src="/media/itpro/services/dialog_box_for_failover_load_balancing_method.png" alt="“故障转移”负载平衡方法的对话框"/></p>
<p><span class="Apple-style-span">图 2</span> –“故障转移”负载平衡方法的对话框</p>
<ol>
<li>
<p><span class="Apple-style-span">测试 Traffic Manager 域和策略。</span>有关详细信息，请参阅<a href="#howto_test">如何：测试 Windows Azure Traffic Manager 策略</a>。</p>
</li>
<li>
<p><span class="Apple-style-span">将 DNS 服务器指向 Traffic Manager 域。</span>在设置并运行 Traffic Manager 域后，请编辑权威 DNS 服务器上的 DNS 记录以将公司域指向 Traffic Manager 域。有关详细信息，请参阅<a href="#howto_point_company">如何将公司 Internet 域指向 Traffic Manager 域</a>。例如，以下命令会将转到 <span class="Apple-style-span">www.contoso.com</span> 的所有流量路由至 Traffic Manager 域 <span class="Apple-style-span">contoso.trafficmanager.cn</span><code>www.contoso.com IN CNAME contoso.trafficmanager.cn</code></p>
</li>
</ol>
<h2><a id="howto_direct"></a>如何：基于网络性能将传入流量定向到托管服务</h2>
<p>若要对位于全球各个数据中心的托管服务进行负载平衡，您可将传入流量定向到最近的托管服务。虽然“最近”可能直接对应于地理距离，但它还可对应于对响应请求的延迟最小的位置。利用“性能”负载平衡方法，您可根据位置和延迟进行分发，但无法考虑网络配置或负载中的实时更改。有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/hh744833.aspx">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“Windows Azure Traffic Manager 中的负载平衡方法”一节。</p>
<p>下列步骤将为您演示此过程：</p>
<ol>
<li>
<p>将托管服务部署到生产环境中。有关详细信息，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/gg432967.aspx">创建 Windows Azure 托管服务</a>。另请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring">Windows Azure Traffic Manager 概述（可能为英文页面）</a>中的“托管服务和策略的最佳实践”一节</p>
。</li>
<li>
<p><span class="Apple-style-span">登录管理门户中的“Traffic Manager”区域</span>，网址为 <a href="http://manage.windowsazure.cn">http://manage.windowsazure.cn</a>。单击门户页左下角的“虚拟网络”，然后从左侧窗格中的选项中选择“Traffic Manager”。</p>
</li>
<li>
<p><span class="Apple-style-span">选择“策略”并单击“创建”。</span>从左侧导航树中选择“策略”文件夹以启用顶部工具栏中的“创建”。选择“创建”。这将显示“创建 Traffic Manager 策略”对话框。</p>
<p><img src="/media/itpro/services/create_button_for_policies.png" alt="针对策略的“创建”按钮"/></p>
<p><span class="Apple-style-span">图 1</span> - 针对策略的“创建”按钮</p>
</li>
<li>
<p><span class="Apple-style-span">选择订阅。</span> 策略和域将与单个订阅关联。</p>
</li>
<li>
<p><span class="Apple-style-span">选取“性能”负载平衡方法。</span>有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/hh744833.aspx">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“Windows Azure Traffic Manager 中的负载平衡方法”一节。</p>
</li>
<li>
<p><span class="Apple-style-span">查找托管服务并将其添加到策略中。</span>使用筛选器框查找包含您在该框中键入的字符串的托管服务。清除该框可显示适用于在步骤 4 中选择的订阅的生产中的所有托管服务。使用这些箭头按钮可将托管服务添加到策略中。“选定 DNS 名称”中的顺序与此负载平衡方法无关。</p>
</li>
<li>
<p><span class="Apple-style-span">设置监视。</span>通过监视来确保不会向处于脱机状态的托管服务发送流量。您必须已设置特定的路径和文件名。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“在 Windows Azure Traffic Manager 中监视托管服务”一节。</p>
</li>
<li>
<p><span class="Apple-style-span">命名 Traffic Manager 域。</span>为您的域指定一个唯一名称。您只能指定域的前缀。将“DNS 生存时间(TTL)”保留为其默认时间。有关此设置的影响的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring">Windows Azure Traffic Manager 概述（可能为英文页面）</a>并滚动到“使用 Windows Azure Traffic Manager 时托管服务和策略的最佳实践”一节。</p>
</li>
</ol>
<p>“创建 Traffic Manager 策略”对话框应与以下示例类似。</p>
<p><img src="/media/itpro/services/dialog_box_for_performance_load_balancing_method.png" alt="“性能”负载平衡方法的对话框"/></p>
<p><span class="Apple-style-span">图 2</span> –“性能”负载平衡方法的对话框</p>
<ol>
<li>
<p><span class="Apple-style-span">测试 Traffic Manager 域和策略。</span>有关测试的详细信息，请参见<a href="#howto_test">如何：测试 Windows Azure Traffic Manager 策略</a>。</p>
</li>
<li>
<p><span class="Apple-style-span">将 DNS 服务器指向 Traffic Manager 域。</span>在设置并运行 Traffic Manager 域后，请编辑权威 DNS 服务器上的 DNS 记录以将公司域指向 Traffic Manager 域。例如，以下命令会将转到 <span class="Apple-style-span">www.contoso.com</span> 的所有流量路由至 Traffic Manager 域 <span class="Apple-style-span">contoso.trafficmanager.cn</span>。<code>www.contoso.com IN CNAME contoso.trafficmanager.cn</code> 有关详细信息，请参阅<a href="#howto_point_company">如何：将公司 Internet 域指向 Windows Azure Traffic Manager 域</a>。</p>
</li>
</ol></div>]]></bodyText><umbracoNaviHide>1</umbracoNaviHide><pageTitle>如何配置 Traffic Manager 设置 - Windows Azure</pageTitle><localize>1</localize><localizePartial>0</localizePartial><sitemapHide>0</sitemapHide><forbidPublish>0</forbidPublish><linkid>manage-services-traffic-manager</linkid><metaKeywords></metaKeywords><metaDescription><![CDATA[了解如何配置 Windows Azure Traffic Manager 设置以控制分发给托管服务的用户流量。]]></metaDescription><headerExpose></headerExpose><footerExpose></footerExpose><urlDisplayName>Traffic Manager</urlDisplayName><disqusComments>1</disqusComments><metaCanonical></metaCanonical><isHeader>0</isHeader><pageTemplate></pageTemplate></TextpageLeftNav></localize>