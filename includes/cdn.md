<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="使用 Azure CDN" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons" description="Learn how to use the Azure Content Delivery Network (CDN) to deliver high-bandwidth content by caching blobs and static content." metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />

# 使用 Azure CDN



Microsoft Azure 内容传送网络 (CDN) 通过遍布在中国大陆的众多物理节点上缓存Azure平台上的Storage Blob，Cloud Service和WebSites的静态内容和动态内容，为开发人员提供一个传送高带宽内容的解决方案。目前本CDN服务也同时支持没有部署在Azure平台上的源站。

此任务包括下列步骤：

+ [步骤 1:创建存储帐户，云服务或者网站](#1)
+ [步骤 2:创建新的 CDN 终结点](#2)
+ [步骤 3:访问 CDN 内容](#3)
+ [步骤 4:删除 CDN 中的内容](#4)
+ [步骤 5:使用高级管理功能](#5)

使用 CDN 缓存 Microsft Azure 数据的优点包括：

- 远离内容源并使用需要进行多次“互联网旅行”才能加载内容的应用程序的最终用户可获得更好的性能和用户体验
- 大型分布式规模可更好地处理瞬时高负载（例如在像产品发布这样的活动开始时）

现有 Microsoft Azure 中国客户现在可使用[Microsoft Azure 管理门户](https://manage.windowsazure.cn/)中的 Microsoft Azure CDN。 

## [步骤 1:创建存储帐户，云服务或者网站](id:1)
您可以为现有的Microsoft Azure订阅中的存储账户，云服务或者网站创建CDN终结点。您也可以按以下过程创建新的存储帐户，云服务或者网站用于 Microsft Azure 订阅。

### 为 Microsoft Azure 订阅创建存储帐户
请参阅 [如何创建存储帐户](/zh-cn/documentation/articles/storage-create-storage-account/)

### 为 Microsft Azure 订阅创建云服务
请参阅 [如何创建和部署云服务](/zh-cn/documentation/articles/cloud-services-how-to-create-deploy/) 

### 为 Microsoft Azure 订阅创建网站
请参阅 [如何创建和部署网站](/zh-cn/documentation/articles/web-sites-create-deploy/) 

## [步骤 2:创建新的 CDN 终结点](id:2)
一旦启用对存储帐户，云服务或者网站的 CDN 访问，所有公开可用的对象将有资格获得 CDN 边缘高速缓存。如果您修改一个当前在 CDN 中缓存的对象，则只有 CDN 在缓存内容生存时间到期时刷新了对象的内容后（或通过高级管理功能进行手动刷新），才能通过 CDN 访问新内容。

### 创建新的 CDN 终结点
1. 在 [Microsoft Azure 管理门户](https://manage.windowsazure.cn/)的导航窗格中，单击“CDN”。
2. 在功能区上，单击“新建”。在“新建”对话框上，依次选择“应用服务”、“CDN”和“快速创建”。

    ![CDN quick create][1]
3. 在“订阅”下拉列表中选择所要使用的Azure 订阅（如果有多个订阅的话）。
4. 在“原始域类型”下拉列表中，选择云服务，存储账户或者网站。
5. 在“原始域”下拉列表中，从可用的云服务，存储帐户或者网站列表中选择一个用于创建CDN终结点。
6. 在“自定义域”中输入要使用的自定义域名如：cdn.yourcompany.com
7. 在“ICP编号”中输入和上一步中所输入的自定义域名相对应的**ICP备案号**（如：京ICP备XXXXXXXX号-X）。
8. 单击“创建”按钮以创建新的终结点。
9. 终结点创建后将出现在订阅的终结点的列表中。列表视图显示了用于访问缓存内容的自定义域以及原始域。

原始域是 CDN 所缓存内容的原始位置。自定义域是用于访问CDN缓存内容的URL。
> **注意** 为终结点创建的配置将不能立即可用：

> 1. 首先需要审核所提供的自定义域名和ICP编号是否匹配、有效。这个过程需要最多一个工作日的时间来完成。
2. 如果ICP审核没有通过，您需要删除之前创建的这个CDN终结点，然后使用正确的自定义域名和ICP编号重新创建。
3. 如果ICP审核通过，CDN服务最多需要 60 分钟时间进行注册以便通过 CDN 网络传播。与此同时，您还需要按照界面上的提示信息配置CNAME映射信息，这样才可以最终通过自定义域名访问CDN缓存内容。**建议您在修改CNAME映射之前，先通过步骤5中所提供的“服务检查”功能，确定CDN服务已经生效，之后再进行CNAME配置，以免发生服务中断。**

## [步骤 3:访问 CDN 内容](id:3)
若要访问 CDN 上的缓存内容，请使用您在步骤2中所提供的自定义域名来访问CDN缓存内容。缓存 Blob 的地址将类似下面这样（以步骤2中的例子为例）：

`http://cdn.yourcompany.com/<myPublicContainer>/<BlobName>`

## [步骤 4:删除 CDN 中的内容](id:4)
如果您不再想在 Microsoft Azure 内容交付网络 (CDN) 中缓存对象，则可执行下列步骤之一：

- 对于 Microsoft Azure Blob，可从公共容器中删除该 Blob。
- 生成专用容器代替公用容器。有关更多信息，请参见[限制对容器和 Blob 的访问](http://msdn.microsoft.com/zh-cn/library/dd179354.aspx)。
- 您可使用管理门户禁用或删除 CDN 终结点。
- 您可将云服务修改为不再响应此对象的请求。

已在 CDN 中缓存的对象将保持缓存状态，直到该对象的生存时间到期为止。当生存时间到期时，CDN 将查看 CDN 终结点是否仍有效，且是否仍可对该对象进行匿名访问。如果不能访问，则不再对该对象进行缓存。

## [步骤 5:使用高级管理功能](id:5)
当您创建好CDN终结点之后，除了可以在管理门户中查看基本的配置信息和对CDN终结点做“禁用/启用”和“删除”等基本操作外，您还可以通过点击“管理”按钮，从而跳转到另外的管理页面进行高级管理功能：

![Manage Button][2]
> **注意：您将被引导至另外的管理页面，它不属于管理门户的一部分。（请注意允许浏览器打开新的窗口）**

![Adv Portal][3]

这个高级管理界面提供了“服务配置”、“统计报表”、“缓存刷新”以及“服务检查”四大类功能。具体的使用和配置方式，可以点击进入相应的功能模块，然后查看对应的帮助文件。

下面着重介绍一下“服务检查”这个功能，您可以在切换CNAME映射之前使用这个功能来检验一下CDN服务是否生效。在确定CDN服务生效后，再进行CNAME映射切换，这样可以避免出现服务中断的情况。

下图所示是一个实际的“服务检查”例子：

![service check][4]

首先在输入框中输入一个用于服务检查的测试URL（使用用于创建该CDN终结点的自定义域名），如：`http://cdn.yourcompany.com/images/logo.png`。您需要先确定在使用源站地址的情况下，这个URL可被访问。

### 接下来详细解释一下各检测值所表达的含义：

1. 源站检查情况：到源站服务器请求这个URL，看返回结果是否是200。如果返回非200，则认为源站异常。如果直接检查域名，而有的网站域名本身是不可访问的，会返回302、400等结果，那么可以忽略该项。
2. 是否CNAME：如果没有到域名托管商那里修改DNS解析，那么这里是“否”。
3. 是否被CDN缓存：该项主要是检查所测试的URL是否能被CDN缓存（比如：响应header里有cookie、no-cache等字段，那么是不能缓存的）。该项不会影响CNAME后，网站的正常访问。 
4. 源站与CDN一致性：到各个CDN节点请求该URL，然后将返回结果与去源站请求返回的结果做对比，看是否一致。如果不一致，则说明有CDN节点有问题，或者没有拿到任务。此时，如果CNAME，当解析到那个有问题的节点时，就会无法访问。**所以建议您只有当这个测试值显示“一致”之后，再进行CNAME解析切换，以防产生服务中断。**

### 一致性检查 主要检查三项：
1. Status是否一致：到CDN节点请求和到源站请求返回的状态码是否一致；
2. ContentLength：到CDN节点请求和到源站请求得到的内容大小是否一致；
3. LastModified：到CDN节点请求和到源站请求返回的内容最近更改时间是否一致；

如果1 在ICP审核通过60分钟后“不一致”个数仍大于零， 则说明CDN节点或者源站有问题。这是您需要及时联系客服。

如果2、3不一致，说明源站内容有更新了，CDN节点上缓存还没有更新。此时可以等待原缓存内容的生存时间到期后自动更新，或者手动提交缓存刷新请求即可。


[步骤 1:创建存储帐户，云服务或者网站]: #步骤-1创建存储帐户云服务或者网站
[步骤 2:创建新的 CDN 终结点]: #步骤-2创建新的-cdn-终结点
[步骤 3:访问 CDN 内容]: #步骤-3访问-cdn-内容
[步骤 4:删除 CDN 中的内容]: #步骤-4删除-cdn-中的内容
[步骤 5:使用高级管理功能]: #步骤-5使用高级管理功能


<!--Image references-->
[1]: ./media/cdn/image001.png
[2]: ./media/cdn/image002.png
[3]: ./media/cdn/image003.png
[4]: ./media/cdn/image004.png