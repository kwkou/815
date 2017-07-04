域名系统 (DNS) 用于在 Internet 上查找资源。例如，当你在浏览器中输入一个 Web 应用地址或单击网页上的某个链接时，它使用 DNS 将域转换为 IP 地址。IP 地址有点像街道地址，但其用户友好性并不是很好。例如，记住 **contoso.com** 这样的 DNS 名称比较容易，而记住 192.168.1.88 或 2001:0:4137:1f67:24a2:3888:9cce:fea3 这样的 IP 地址则要困难得多。

DNS 系统基于 *记录* 。记录将特定的 *名称*（例如 **contoso.com**）与一个 IP 地址或其他的 DNS 名称相关联。当某个应用程序（例如 Web 浏览器）在 DNS 中查找某个名称，它将找到该记录，并将它所指向的内容用作地址。如果它所指向的值是 IP 地址，则浏览器将使用该值。如果它指向另一个 DNS 名称，则应用程序必须再次执行解析。最终，所有名称解析都将以 IP 地址的形式结束。

在应用服务中创建 Web 应用时，将为 Web 应用自动分配 DNS 名称。此名称采用 **&lt;yourwebappname&gt;.chinacloudsites.cn** 的格式。创建 DNS 记录时，还有一个虚拟 IP 地址可供使用，因此你可以创建指向 **.chinacloudsites.cn** 的记录，也可以指向该 IP 地址。

> [AZURE.NOTE] 如果你删除或重新创建 Web 应用，或者在将 App Service 计划模式设置为“基本”、“共享”或“标准”后再将其更改为“免费”，则 Web 应用的 IP 地址将会更改。

此外还有多种类型的记录，每种类型都有其自己的功能和限制，但是对于 Web 应用，我们只关心两种，即 *A* 和 *CNAME* 记录。

###地址记录（A 记录）

A 记录将域（例如 **contoso.com** 或 **www.contoso.com**）*或通配符域*（例如 ***.contoso.com**）映射到 IP 地址。对于 App Service 中的 Web 应用，这是服务的虚拟 IP 或者你为 Web 应用购买的具体 IP 地址。

A 记录相比于 CNAME 记录的主要优势是：

* 你可以将根域（例如 **contoso.com**）映射到 IP 地址；许多注册机构仅允许使用 A 记录执行此操作

* 你可以有一个使用通配符的条目（例如 ***.contoso.com**），它将处理多个子域（例如 **mail.contoso.com**、**blogs.contoso.com** 或 **www.contso.com**）的请求。

> [AZURE.NOTE] 由于 A 记录映射到静态 IP 地址，因此它无法自动解析 Web 应用的 IP 地址的更改。用于 A 记录的 IP 地址是你为 Web 应用配置自定义域名设置时提供的；但是，如果你删除并重新创建 Web 应用或者将 App Service 计划模式改回“免费”，则该值可能会更改。

###别名记录（CNAME 记录）

CNAME 记录将*特定的* DNS 名称（例如 **mail.contoso.com** 或 **www.contoso.com**）映射到另一个（规范）域名。对于应用服务 Web 应用，规范域名是 Web 应用的 **&lt;yourwebappname>.chinacloudsites.cn** 域名。一旦创建，CNAME 即为 **&lt;你的 Web 应用名称>.chinacloudsites.cn** 域名创建一个别名。CNAME 条目将自动解析为 **&lt;你的 Web 应用名称>.chinacloudsites.cn** 域名的 IP 地址，因此，如果 Web 应用的 IP 地址发生更改，你不必采取任何操作。

> [AZURE.NOTE] 某些域注册机构只允许你在使用 CNAME 记录（例如 **www.contoso.com**）和非根名称（例如 **contoso.com**）时映射子域。有关 CNAME 记录的详细信息，请参阅由你的注册机构提供的文档、<a href="http://en.wikipedia.org/wiki/CNAME_record">CNAME 记录上的 Wikipedia 条目</a>或 <a href="http://tools.ietf.org/html/rfc1035">IETF 域名 - 实现和规范</a>文档。

###Web 应用 DNS 细节

为 Web 应用使用 A 记录需要首先创建以下 TXT 记录之一：

* **对于根域** - 将 **@** 的DNS TXT 记录映射为 **&lt;yourwebappname&gt;.chinacloudsites.cn**。

* **对于特定的子域** - 将 **&lt;子域>** 的 DNS 名称映射为 **&lt;你的 Web 应用名称&gt;.chinacloudsites.cn**。例如，如果 A 记录用于 **blogs.contoso.com**，则为 **blogs**。

* **对于通配符子域** - 将 ***** 的DNS TXT 记录映射为 **&lt;yourwebappname&gt;.chinacloudsites.cn**。

此 TXT 记录用于验证你是否拥有正在尝试使用的域。此操作是对创建指向 Web 应用的虚拟 IP 地址的 A 记录的补充。

可以按照以下步骤找到 Web 应用的 IP 地址和 **.chinacloudsites.cn** 名称：

1. 在浏览器中，打开 [Azure 门户](https://portal.azure.cn)。

2. 在“Web 应用”边栏选项卡中，单击 Web 应用的名称，然后从页面底部选择“自定义域”。

	![](./media/custom-dns-web-site/dncmntask-cname-6.png)

3. 在“自定义域”边栏选项卡上，你会看到虚拟 IP 地址。保存此信息，因为将在创建 DNS 记录时使用它

	![](./media/custom-dns-web-site/virtual-ip-address.png)

	> [AZURE.NOTE] **免费** Web 应用不可使用自定义域名，并且必须将应用服务计划升级到**共享**、**基本**、**标准**或**高级**层。若要深入了解应用服务计划的定价层，包括如何更改 Web 应用的定价层，请参阅[如何缩放 Web 应用](/documentation/articles/web-sites-scale/)。

<!---HONumber=Mooncake_0926_2016-->