<properties 
	pageTitle="为 Blob 存储终结点配置域名 | Azure"
	description="了解如何将自定义用户域映射到 Azure 存储帐户的 Blob 存储终结点。"
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="02/14/2016"
	wacn.date="04/11/2016"/>


# 为 Blob 存储终结点配置自定义域名

## 概述

你可以配置自定义域以便访问 Azure 存储帐户中的 Blob 数据。Blob 存储的默认终结点为 https://<*mystorageaccount*>.blob.core.chinacloudapi.cn。如果你将自定义域和子域（例如 **www.contoso.com**）映射到你的存储帐户的 Blob 终结点，则你的用户也可以使用该域访问你的存储帐户中的 Blob 数据。 


> [AZURE.NOTE]	此任务中的过程适用于 Azure 存储帐户。对于云服务，请参阅<a href = "/documentation/articles/cloud-services-custom-domain-name/">为 Azure 云服务配置自定义域名</a>；对于 Web 应用，请参阅<a href="/documentation/articles/web-sites-custom-domain-name/">为 Azure Web 应用配置自定义域名</a>。 

有两种方法可用于将你的自定义域指向你的存储帐户的 Blob 终结点。最简单方法是创建一个 CNAME 记录，将你的自定义域和子域映射到 Blob 终结点。CNAME 记录是一种 DNS 功能，用于将源域映射到目标域。在此情况下，源域是你的自定义域和子域 -- 请注意，始终需要子域。目标域是你的 Blob 服务终结点。

但是，将你的自定义域映射到 Blob 终结点的过程会导致域在你在 [Azure 管理门户](https://manage.windowsazure.cn)中注册域时出现短暂的停机时间。如果你的自定义域目前所支持的应用程序的服务级别协议 (SLA) 要求不能有停机时间，则可以使用 Azure **asverify** 子域提供中间注册步骤，以便用户在 DNS 映射进行时能够访问你的域。

下表显示了用于访问名为 **mystorageaccount** 的存储帐户中的 Blob 数据的示例 URL。为存储帐户注册的自定义域是 **www.contoso.com**：

资源类型|URL 格式
---|---
存储帐户|**默认 URL：** http://mystorageaccount.blob.core.chinacloudapi.cn<p>**自定义域 URL：** http://www.contoso.com</td>
Blob|**默认 URL：** http://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myblob<p>**自定义域 URL：** http://www.contoso.com/mycontainer/myblob
根容器|**默认 URL：** http://mystorageaccount.blob.core.chinacloudapi.cn/myblob 或 http://mystorageaccount.blob.core.chinacloudapi.cn/$root/myblob<p>**自定义域：** http://www.contoso.com/myblob 或 http://www.contoso.com/$root/myblob

## 为你的存储帐户注册自定义域

如果你不担心域暂时对用户不可用，或者你的自定义域当前未托管应用程序，则使用本过程可注册你的自定义域。

如果你的自定义域目前在支持不能有任何停机时间的应用程序，则使用在<a href="#register-a-custom-domain-for-your-storage-account-using-the-intermediary-asverify-subdomain">使用中间 asverify 子域为你的存储帐户注册自定义域</a>中介绍的过程。

若要配置自定义域名，必须通过你的域注册机构创建一个新的 CNAME 记录。该 CNAME 记录为域名指定别名；在这个例子中，它将自定义域的地址映射到你的存储帐户的 Blob 存储终结点。

每个注册机构指定 CNAME 记录的方法类似但略有不同，但概念是相同的。请注意，许多基本域注册程序包不提供 DNS 配置，因此，您可能需要首先升级您的域注册程序包，然后才能创建 CNAME 记录。

1.  在 [Azure 管理门户](https://manage.windowsazure.cn)中，导航到“存储”选项卡。

2.  在“存储”选项卡中，单击要为其映射自定义域的存储帐户的名称。

3.  单击“配置”选项卡。

4.  在屏幕的底部，单击“管理域”以显示“管理自定义域”对话框。在该对话框顶部的文本中，你将看到有关如何创建 CNAME 记录的信息。对于此过程，请忽略引用 **asverify** 子域的文本。

5.  登录到您的 DNS 注册机构的网站，然后转至用于管理 DNS 的页面。可能会在“域名”、“DNS”或“名称服务器管理”等部分中找到此页。

6.  找到用于管理 CNAME 的部分。你可能需要转至高级设置页面，并查找“CNAME”、“别名”或“子域”字样。

7.  创建一个新的 CNAME 记录，并且提供子域别名，例如 **www** 或 **photos**。然后以 **mystorageaccount.blob.core.chinacloudapi.cn** 格式（其中，**mystorageaccount** 是你的存储帐户的名称）提供主机名，这是你的 Blob 服务终结点。在“管理自定义域”对话框的文本中为你提供了要使用的主机名。

8.  在创建该 CNAME 记录后，返回到“管理自定义域”对话框，并且在“自定义域名”字段中输入你的自定义域（包括子域）的名称。例如，如果你的域是 **contoso.com** 并且你的子域是 **www**，则输入 **www.contoso.com**；如果你的子域是 **photos**，则输入 **photos.contoso.com**。请注意，子域是必需的。

9. 单击“注册”按钮以注册你的自定义域。

	如果注册成功，你将看到消息**你的自定义域处于活动状态**。用户现在可以查看你的自定义域上的 Blob 数据，只要用户具有适当的权限。

## 使用中间 asverify 子域为你的存储帐户注册自定义域

如果你的自定义域当前所支持的应用程序的 SLA 要求没有停机时间，则使用此过程注册你的自定义域。通过创建从 asverify.&lt;subdomain&gt;.&lt;customdomain&gt; 指向 asverify.&lt;storageaccount&gt;.blob.core.chinacloudapi.cn 的 CNAME，你可以在 Azure 预先注册你的域。然后，你可以创建第二个从 &lt;subdomain&gt;.&lt;customdomain&gt; 指向 &lt;storageaccount&gt;.blob.core.chinacloudapi.cn 的 CNAME，此时到你的自定义域的流量将被定向到你的 Blob 终结点。

该 asverify 子域是 Azure 能够识别的一个特殊子域。通过将 **asverify** 追加到你自己的子域，可以使 Azure 能够识别你的自定义域且不需要修改针对该域的 DNS 记录。一旦您修改该域的 DNS 记录，它将映射到 Blob 终结点且没有停机时间。

1.  在 [Azure 管理门户](https://manage.windowsazure.cn)中，导航到“存储”选项卡。

2.  在“存储”选项卡中，单击要为其映射自定义域的存储帐户的名称。

3.  单击“配置”选项卡。

4.  在屏幕的底部，单击“管理域”以显示“管理自定义域”对话框。在对话框顶部的文本中，你将看到有关如何使用 **asverify** 子域创建 CNAME 记录的信息。

5.  登录到您的 DNS 注册机构的网站，然后转至用于管理 DNS 的页面。可能会在“域名”、“DNS”或“名称服务器管理”等部分中找到此页。

6.  找到用于管理 CNAME 的部分。你可能需要转至高级设置页面，并查找“CNAME”、“别名”或“子域”字样。

7.  创建一个新的 CNAME 记录，并且提供包括 asverify 子域的子域别名。例如，你指定的子域将采用 **asverify.www** 或 **asverify.photos** 格式。然后提供主机名，这是你的 Blob 服务终结点，格式为 **asverify.mystorageaccount.blob.core.chinacloudapi.cn**（其中 **mystorageaccount** 是你的存储帐户的名称）。在“管理自定义域”对话框的文本中为你提供了要使用的主机名。

8.  在创建该 CNAME 记录后，返回到“管理自定义域”对话框，并且在“自定义域名”字段中输入你的自定义域的名称。例如，如果你的域是 **contoso.com** 并且你的子域是 **www**，则输入 **www.contoso.com**；如果你的子域是 **photos**，则输入 **photos.contoso.com**。请注意，子域是必需的。

9.	单击“高级：使用 'asverify' 子域预先注册自定义域”复选框。

10. 单击“注册”按钮以预先注册你的自定义域。

	如果预先注册成功，你将会看到一条消息“你的自定义域处于活动状态”。

11. 此时，你的自定义域已由 Azure 进行了验证，但传输到你的域的流量尚未路由到你的存储帐户。若要完成此过程，请返回到你的 DNS 注册机构的网站，创建将你的子域映射到你的 Blob 终结点的另一条 CNAME 记录。例如，将该子域指定为 **www** 或 **photos**，将主机名指定为 **mystorageaccount.blob.core.chinacloudapi.cn**（其中，**mystorageaccount** 是你的存储帐户的名称）。完成此步骤后，也就完成了你的自定义域的注册。

12. 最后，你可以使用 **asverify** 删除你创建的 CNAME 记录，因为只需要将其作为中间步骤使用。

用户现在可以查看你的自定义域上的 Blob 数据，只要用户具有适当的权限。

## 验证该自定义域引用你的 Blob 服务终结点

若要验证你的自定义域是否确实已映射到你的 Blob 服务终结点，请在你的存储帐户内的公共容器中创建一个 Blob。然后在 Web 浏览器中，使用以下格式的 URI 来访问该 Blob：

-   http://<*subdomain.customdomain*>/<*mycontainer*>/<*myblob*>

例如，你可以使用以下 URI 来通过映射到 **myforms** 容器中的 Blob 的自定义子域 **photos.contoso.com** 访问 Web 窗体：

-   http://photos.contoso.com/myforms/applicationform.htm

## 其他资源

-   <a href="http://msdn.microsoft.com/zh-cn/library/azure/gg680307.aspx">如何将 CDN 内容映射到自定义域</a>
 

<!---HONumber=Mooncake_0405_2016-->