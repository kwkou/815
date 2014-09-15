<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="How to Configure Traffic Manager Settings" authors="" solutions="" manager="" editor="" />

# 如何配置 Traffic Manager 设置

利用 Azure Traffic Manager，可以控制分发给 Azure 托管服务的用户流量。

Traffic Manager 的工作方式是，向针对公司主域名的 DNS 查询应用智能策略引擎。更新你公司拥有的 DNS 资源记录以指向 Traffic Manager 域。随后，附加到这些域的 Traffic Manager 策略会将针对你的公司主域名的 DNS 查询解析为 Traffic Manager 策略中包含的特定 Azure 托管服务的 IP 地址。有关详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/hh744833.aspx)。

## 目录

-   [如何：将公司 Internet 域指向 Traffic Manager 域](#howto_point_company)
-   [如何：测试策略](#howto_test)
-   [如何：临时禁用策略和托管服务](#howto_temp_disable)
-   [如何：编辑策略](#howto_edit_policy)
-   [如何：在一组托管服务中均等平衡负载流量](#howto_load_balance)
-   [如何：创建故障转移策略](#howto_create_failover)
-   [如何：基于网络性能将传入流量定向到托管服务](#howto_direct)

<a id="howto_point_company"></a>
## 如何：将公司 Internet 域指向 Traffic Manager 域

若要将公司域名指向 Traffic Manager 域，请使用 CNAME 编辑 DNS 服务器上的 DNS 资源记录。

例如，若要将公司主域 **www.contoso.com** 指向名为 **contoso.trafficmanager.net** 的 Traffic Manager 域，请更新 DNS 资源记录，如下所示：`www.contoso.com IN CNAME contoso.trafficmanager.net`

此时，转到 *www.contoso.com* 的所有流量将重定向到 *contoso.trafficmanager.net*。请确保你使用的是希望将其中的所有流量重定向到 Traffic Manager 的域。

<a id="howto_test"></a>
## 如何：测试策略

测试策略的最佳方式是设置大量客户端，然后将策略中的服务一次关闭一个。下列提示将帮助你测试 Traffic Manager 策略：

-   **将 DNS TTL 设置得非常低**，以便快速传播更改（例如 30 秒）。

-   **知道要测试的策略中的 Azure 托管服务的 IP 地址**。你可以从 Azure 管理门户中获取此信息。单击服务的“生产部署”。在右侧的属性窗格中，最后一个条目将是 VIP，它是该托管服务的虚拟 IP 地址。

	![hosted service IP location][0]

    **图 1** - 托管服务 IP 位置

-   **使用能够让你将 DNS 名称解析为 IP 地址并显示该地址的工具**。你将检查你的公司域名是否解析为策略中的托管服务的 IP 地址。它们应通过与 Traffic Manager 策略的负载平衡方法一致的方式进行解析。如果你使用的是 Windows 系统，则可在 CMD 窗口中使用 nslookup（下面将介绍）。此外，可以从 Internet 轻松获得允许你“挖掘”IP 地址的其他公用工具。

- 使用 *nslookup* **检查 Traffic Manager 策略**。

> **使用 nslookup 检查 Traffic Manager 策略**

>1. 单击“开始”-\>**运行**并键入 **CMD** 以启动 CMD 窗口。

>2.  键入 `ipconfig /flushdns` 以便刷新 DNS 解析器缓存。

>3. 键入命令 ``nslookup <your traffic manager domain>``。例如，以下命令将检查前缀为 *myapp.contoso* `nslookup myapp.contoso.trafficmanager.net` 的域

>>典型结果将显示以下内容：

>> - 为了解析此 Traffic Manager 域而要解析的 DNS 服务器的 DNS 名称和 IP 地址。

>> - 你在命令行中的“nslookup”后面键入的 Traffic Manager 域名以及 Traffic Manager 域将解析为的 IP 地址。需要重点检查第二个 IP 地址。该地址应与要测试的 Traffic Manager 策略中的某个托管服务的 VIP 匹配。

>>![nslookup command example][1]

>>**图 2** – nslookup 命令示例

- **使用下面列出的其他测试方法之一。**选择适用于要测试的负载平衡类型的方法。

### 性能策略

你将需要借助全球各个位置的客户端才能有效测试你的域。可以在 Azure 中创建将尝试通过公司域名调用服务的客户端。或者，如果你的公司是一家跨国公司，则可远程登录位于国家/地区的其他位置的客户端并从这些客户端进行测试。

可免费获得基于 Web 的 nslookup 和挖掘服务。借助其中的某些服务，你可以从不同的位置检查 DNS 名称解析。例如，对 nslookup 执行搜索。另一个选择是使用第三方解决方案（如 Gomez 或 Keynote）来确认你的策略正在按预期方式分发流量。

### 故障转移策略

1.  将所有服务保持运行。

2.  使用单一客户端。

3.  使用 ping 或类似实用工具请求适用于你的公司域的 DNS 解析。

4.  确保你获取的 IP 地址适用于你的主托管服务。

5.  关闭你的主服务或删除正在监视的文件，以使 Traffic Manager 认为其已关闭。

6.  至少等待 2 分钟。

7.  请求 DNS 解析。

8.  确保你获取的 IP 地址适用于你的辅助托管服务，并按照服务在“编辑 Traffic Manager 策略”对话框中的顺序显示。

9.  重复此过程，将关闭辅助服务，然后关闭第三位服务，依此类推。每次都请确保 DNS 解析返回列表中下一个服务的 IP 地址。关闭所有服务后，你应重新获取主托管服务的 IP 地址。

### 循环法策略

1.  将所有服务保持运行。

2.  使用单一客户端。

3.  请求适用于顶级域的 DNS 解析。

4.  确保你获取的 IP 地址是列表中的某个地址。

5.  重复此过程以让 DNS TTL 到期，这样你便能收到新的 IP 地址。你应看到为每个托管服务返回的 IP 地址。随后，此过程将重复。

<a id="howto_temp_disable"></a>
## 如何：临时禁用策略和托管服务

你可以禁用之前创建的 Traffic Manager 策略，使其不再路由流量。禁用 Traffic Manager 策略后，该策略的信息将在 Traffic Manager 界面中保持不变且可编辑。可以轻松重新启用策略，并且路由将恢复。

也可以禁用属于 Traffic Manager 策略的一部分的各个托管服务。虽然此操作可有效地将托管服务作为策略的一部分保留，但策略的工作方式就像它未包含托管服务时一样。对于临时删除处于维护模式中或要重新部署且很不稳定的托管服务，此操作很有用。在托管服务重新启动并运行后，可重新启用它。

**注意**
禁用托管服务与其在 Azure 中的部署状态无关。正常的服务将保持运行并能够接收流量（即使在 Traffic Manager 中已禁用这一行为）。此外，在一个策略中禁用某个托管服务不会影响该托管服务在其他策略中的状态。

### 禁用策略

1.  在 Traffic Manager 界面树中选择已启用的策略。

2.  单击顶部工具栏中的**禁用策略**。请注意，如果你突出显示已禁用的策略，则此按钮将灰显。

### 启用策略

1.  在 Traffic Manager 界面树中选择已禁用的策略。

2.  单击顶部工具栏中的**启用策略**。请注意，如果你突出显示已禁用的策略，则此按钮将灰显。

### 禁用托管服务

1.  在 Traffic Manager 界面中选择策略中已启用的托管服务。你必须展开策略才能看到其中包含的托管服务。

2.  单击顶部工具栏中的**禁用服务策略**。请注意，如果你突出显示已禁用的托管服务，则此按钮将灰显。

3.  流量将基于为作为策略的一部分的 Traffic Manager 域设置的 DNS TTL 时间停止流入托管服务。有关详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring)并滚动到“在 Azure Traffic Manager 中监视托管服务”一节。

### 启用托管服务

1.  在 Traffic Manager 界面中选择策略中已禁用的托管服务。你必须展开策略才能看到其中包含的托管服务。

2.  单击顶部工具栏中的**启用服务策略**。请注意，如果你突出显示已启用的托管服务，则此按钮将灰显。

3.  流量将重新开始流入策略所指定的托管服务。

<a id="howto_edit_policy"></a>
## 如何：编辑策略

如果你只是需要临时禁用策略或策略中的特定托管服务，则可在不更改策略的情况下临时禁用它们。

若要将策略更改为其他类型，请执行下列步骤：

1.  登录管理门户中的 **Traffic Manager 区域**，网址为 [http://manage.windowsazure.cn](http://manage.windowsazure.cn)。单击门户页左下角的**虚拟网络**，然后从左侧窗格中的选项中选择 **Traffic Manager**。

2.  在管理门户的 Traffic Manager 屏幕**中选择要更改的策略**。

3.  **单击顶部菜单栏上的配置**。

4.  **对策略进行所需更改**。请注意，如果你更改负载平衡方法，则**选定托管服务**列表中的托管服务的顺序可能重要，也可能不重要，具体取决于所选的选项。

5.  单击**保存**。

<a id="howto_load_balance"></a>
## 如何：在一组托管服务中均等平衡负载流量

常见负载平衡模式旨在提供一组相同的托管服务并通过循环法将流量发送到每个托管服务。本文概述了设置 Traffic Manager 域和策略以执行此类负载平衡的步骤。

有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/hh744833.aspx)并滚动到“Azure Traffic Manager 中的负载平衡方法”一节。

1.  **将托管服务部署到生产环境中**。有关开发和部署托管服务的信息，请参阅 [Azure 托管服务](http://msdn.microsoft.com/library/gg432967.aspx)。

2.  **登录管理门户中的 Traffic Manager 区域**，网址为 [http://manage.windowsazure.cn](http://manage.windowsazure.cn)。单击门户页左下角的**虚拟网络**，然后从左侧窗格中的选项中选择**Traffic Manager**。

3.  **选择“策略”并单击“创建”**。从左侧导航树中选择**策略**文件夹以启用顶部工具栏中的**创建**。选择**创建**。这将显示**创建 Traffic Manager 策略**对话框。

	![Create button for policies][2]

    **图 1** - 针对策略的“创建”按钮

4.  **选择订阅。**策略和域将与单个订阅关联。

5.  **选择“循环法”负载平衡方法。**有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/hh744833.aspx)并滚动到 “Azure Traffic Manager 中的负载平衡方法”一节。

6.  **查找托管服务并将其添加到策略中**。使用筛选器框查找包含你在该框中键入的字符串的托管服务。清除该框可显示适用于在步骤 4 中选择的订阅的生产中的所有托管服务。使用这些箭头按钮可将托管服务添加到策略中。**选定 DNS 名称**框中的顺序与此负载平衡方法无关。

7.  **设置监视。**通过监视来确保不会向处于脱机状态的托管服务发送流量。你必须已设置特定的路径和文件名。有关详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring)并滚动到“在 Azure Traffic Manager 中监视托管服务”一节。

8.  **命名 Traffic Manager 域。**为你的域指定一个唯一名称。你只能指定域的前缀。将 DNS 生存时间保留为其默认时间。有关此设置的影响的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring)并滚动到“使用 Azure Traffic Manager 时托管服务和策略的最佳实践”一节。

    **创建 Traffic Manager 策略**对话框应与以下示例类似。

	![Dialog box for Round Robin load balancing method][3]

    **图 2** –“循环法”负载平衡方法的对话框

9.  **测试 Traffic Manager 域和策略**。有关说明，请参阅[如何：测试 Azure Traffic Manager 策略](#howto_test)。

10. **将 DNS 服务器指向 Traffic Manager 域。**在设置并运行 Traffic Manager 域后，请编辑权威 DNS 服务器上的 DNS 记录以将公司域指向 Traffic Manager 域。例如，以下命令会将转到 **www.contoso.com** 的所有流量路由至 Traffic Manager 域 **contoso.trafficmanager.net**：`www.contoso.com IN CNAME contoso.trafficmanager.net` 有关详细信息，请参阅[如何：将公司 Internet 域指向 Azure Traffic Manager 域](#howto_point_company)。

<a id="howto_create_failover"></a>
## 如何：创建故障转移策略

通常，组织希望为其服务提供可靠性。如果组织的主服务已关闭，则组织可通过提供备份服务来做到这一点。服务故障转移的常见模式是提供一组相同的托管服务并向主服务发送流量，并提供一个包含 1 个或多个备份的列表。本文概述了设置 Traffic Manager 策略以执行此类故障转移备份的步骤。

有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/hh744833.aspx)并滚动到“Azure Traffic Manager 中的负载平衡方法”一节。

1.  **将托管服务部署到生产环境中**。有关开发和部署托管服务的信息，请参阅 [Azure 托管服务](http://msdn.microsoft.com/library/gg432967.aspx)。

2.  **登录管理门户中的 Traffic Manager 区域**，网址为 [http://manage.windowsazure.cn](http://manage.windowsazure.cn)。单击门户页左下角的**虚拟网络**，然后从左侧窗格中的选项中选择 **Traffic Manager**。

3.  **选择“策略”并单击“创建”**。从左侧导航树中选择**策略**文件夹以启用顶部工具栏中的**创建**。选择**创建**。这将显示**创建 Traffic Manager 策略**对话框。

	![Create button for policies][4]

    **图 1** - 针对策略的“创建”按钮

4.  **选择订阅。**策略和域将与单个订阅关联。

5.  **选择“故障转移策略”负载平衡方法。**有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/hh744833.aspx)并滚动到 “Azure Traffic Manager 中的负载平衡方法”一节。

6.  **查找托管服务并将其添加到策略中。**使用筛选器框查找包含你在该框中键入的字符串的托管服务。清除该框可显示适用于在步骤 4 中选择的订阅的生产中的所有托管服务。使用这些箭头按钮可将托管服务添加到策略中。如果选择**故障转移**负载平衡方法，则选定服务的顺序很重要。主托管服务位于顶部。根据需要使用向上和向下箭头更改顺序。

7.  通过监视来确保不会向处于脱机状态的托管服务发送流量。在不设置监视的情况下使用故障转移策略是不起作用的，因为流量将发送到主托管服务，即使该托管服务显示为脱机。若要监视托管服务，你必须指定特定的路径和文件名。有关详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring)并滚动到“在 Azure Traffic Manager 中监视托管服务”一节。

8.  **命名 Traffic Manager 域。**为你的域指定一个唯一名称。你只能指定域的前缀。将 **DNS 生存时间(TTL)**保留为其默认时间。有关此设置的影响的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring)并滚动到“使用 Azure Traffic Manager 时托管服务和策略的最佳实践”一节。

    **创建 Traffic Manager 策略**对话框应与以下示例类似。

	![Dialog box for Failover load balancing method][5]

    **图 2** –“故障转移”负载平衡方法的对话框

9.  **测试 Traffic Manager 域和策略。**有关详细信息，请参阅[如何：测试 Azure Traffic Manager 策略](#howto_test)。

10. **将 DNS 服务器指向 Traffic Manager 域。**在设置并运行 Traffic Manager 域后，请编辑权威 DNS 服务器上的 DNS 记录以将公司域指向 Traffic Manager 域。有关详细信息，请参阅[如何将公司 Internet 域指向 Traffic Manager 域](#howto_point_company)。例如，以下命令会将发往 **www.contoso.com** 的所有流量路由至 Traffic Manager 域 **contoso.trafficmanager.net** `www.contoso.com IN CNAME contoso.trafficmanager.net`

<a id="howto_direct"></a>
## 如何：基于网络性能将传入流量定向到托管服务

若要对位于全球各个数据中心的托管服务进行负载平衡，你可将传入流量定向到最近的托管服务。虽然“最近”可能直接对应于地理距离，但它还可对应于对响应请求的延迟最小的位置。利用“性能”负载平衡方法，你可根据位置和延迟进行分发，但无法考虑网络配置或负载中的实时更改。有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/hh744833.aspx)并滚动到 “Azure Traffic Manager 中的负载平衡方法”一节。

下列步骤将为你演示此过程：

1.  **将托管服务部署到生产环境中**。有关详细信息，请参阅[创建 Azure 托管服务](http://msdn.microsoft.com/zh-cn/library/azure/gg432967.aspx)。另请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring)中的“托管服务和策略的最佳实践”一节。

2.  **登录管理门户中的 Traffic Manager 区域**，网址为 [http://manage.windowsazure.cn](http://manage.windowsazure.cn)。单击门户页左下角的**虚拟网络**，然后从左侧窗格中的选项中选择**Traffic Manager**。

3.  **选择“策略”并单击“创建”**。从左侧导航树中选择**策略**文件夹以启用顶部工具栏中的**创建**。选择**创建**。这将显示**创建 Traffic Manager 策略**对话框。

	![Create button for policies][6]

    **图 1** - 针对策略的“创建”按钮

4.  **选择订阅。**策略和域将与单个订阅关联。

5.  **选取“性能”负载平衡方法。**有关 Traffic Manager 提供的各种负载平衡方法的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/hh744833.aspx)并滚动到“Azure Traffic Manager 中的负载平衡方法”一节。

6.  **查找托管服务并将其添加到策略中。**使用筛选器框查找包含你在该框中键入的字符串的托管服务。清除该框可显示适用于在步骤 4 中选择的订阅的生产中的所有托管服务。使用这些箭头按钮可将托管服务添加到策略中。**选定 DNS 名称**中的顺序与此负载平衡方法无关。

7.  **设置监视。**通过监视来确保不会向处于脱机状态的托管服务发送流量。你必须已设置特定的路径和文件名。有关详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring)并滚动到“在 Azure Traffic Manager 中监视托管服务”一节。

8.  **命名 Traffic Manager 域。**为你的域指定一个唯一名称。你只能指定域的前缀。将 **DNS 生存时间(TTL)**保留为其默认时间。有关此设置的影响的详细信息，请参阅 [Azure Traffic Manager 概述](http://msdn.microsoft.com/zh-cn/library/azure/5229dd1c-5a91-4869-8522-bed8597d9cf5#BKMK_Monitoring)并滚动到“使用 Azure Traffic Manager 时托管服务和策略的最佳实践”一节。

    **创建 Traffic Manager 策略**对话框应与以下示例类似。

	![Dialog box for Performance load balancing method][7]

    **图 2** –“性能”负载平衡方法的对话框

9.  **测试 Traffic Manager 域和策略。**有关测试的详细信息，请参阅[如何：测试 Azure Traffic Manager 策略](#howto_test)。

10. **将 DNS 服务器指向 Traffic Manager 域。**在设置并运行 Traffic Manager 域后，请编辑权威 DNS 服务器上的 DNS 记录以将公司域指向 Traffic Manager 域。例如，以下命令会将发往 **www.contoso.com** 的所有流量路由至 Traffic Manager 域 **contoso.trafficmanager.net** `www.contoso.com IN CNAME contoso.trafficmanager.net` 有关详细信息，请参阅[如何：将公司 Internet 域指向 Azure Traffic Manager 域](#howto_point_company)。

[0]: ./media/traffic-manager-configure-settings/hosted_service_IP_location.png
[1]: ./media/traffic-manager-configure-settings/nslookup_command_example.png
[2]: ./media/traffic-manager-configure-settings/Create_button_for_policies.png
[3]: ./media/traffic-manager-configure-settings/Dialog_box_for_Round_Robin_load_balancing_method.png
[4]: ./media/traffic-manager-configure-settings/Create_button_for_policies.png
[5]: ./media/traffic-manager-configure-settings/Dialog_box_for_Failover_load_balancing_method.png
[6]: ./media/traffic-manager-configure-settings/Create_button_for_policies.png
[7]: ./media/traffic-manager-configure-settings/Dialog_box_for_Performance_load_balancing_method.png

