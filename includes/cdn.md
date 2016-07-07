> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-overview/)
- [English](/documentation/articles/cdn-overview/) 
# 使用 Azure CDN

Azure 内容传送网络 (CDN) 通过遍布在中国大陆的众多物理节点上缓存Azure平台上的Storage Blob，Cloud Service和WebSites的静态内容，为开发人员提供一个传送高带宽内容的解决方案。目前本CDN服务也同时支持没有部署在Azure平台上的源站使用。

此任务包括下列步骤：

+ [步骤 1:创建存储帐户，云服务, Web 应用或媒体服务](#step1)
+ [步骤 2:创建新的 CDN 终结点](#step2)
+ [步骤 3:访问 CDN 内容](#step3)
+ [步骤 4:删除 CDN 中的内容](#step4)
+ [步骤 5:使用高级管理功能](#step5)

使用 CDN 缓存 Azure 数据的优点包括：

- 远离内容源并使用需要进行多次“互联网旅行”才能加载内容的应用程序的最终用户可获得更好的性能和用户体验
- 大型分布式规模可更好地处理瞬时高负载（例如在像产品发布这样的活动开始时）

现有 Azure 中国客户现在可使用[Azure 管理门户](https://manage.windowsazure.cn/)中的 Azure CDN。 如需开通HTTPS加速类型，请参考[Azure CDN HTTPS 加速服务](https://www.azure.cn/documentation/articles/cdn-https-how-to/)。

## 步骤 1:创建存储帐户，云服务， Web 应用或媒体服务<a id="step1"></a>
您可以为现有的Azure订阅中的存储账户，云服务， Web 应用或媒体服务创建CDN终结点。您也可以按以下过程创建新的存储帐户，云服务或者 Web 应用用于 Azure 订阅。

### 为 Azure 订阅创建存储帐户
请参阅 [如何创建存储帐户](/zh-cn/documentation/articles/storage-create-storage-account/)

### 为 Azure 订阅创建云服务
请参阅 [如何创建和部署云服务](/zh-cn/documentation/articles/cloud-services-how-to-create-deploy/) 

### 为 Azure 订阅创建 Web 应用
请参阅 [如何创建和部署 Web 应用](/zh-cn/documentation/articles/web-sites-create-deploy/) 

### 为 Azure 订阅创建媒体服务
请参阅 [如何创建和部署媒体服务](/documentation/articles/media-services-create-account/) 

## 步骤 2:创建新的 CDN 终结点<a id="step2"></a>
一旦启用对存储帐户，云服务或者 Web 应用的 CDN 访问，所有公开可用的对象将有资格获得 CDN 边缘高速缓存。如果您修改一个当前在 CDN 中缓存的对象，则只有 CDN 在缓存内容生存时间到期时刷新了对象的内容后（或通过高级管理功能进行手动刷新），才能通过 CDN 访问新内容。

### 创建新的 CDN 终结点
1. 在 [Azure 管理门户](https://manage.windowsazure.cn/)的导航窗格中，单击“CDN”。
2. 在功能区上，单击“新建”。在“新建”对话框上，依次选择“应用服务”、“CDN”和“快速创建”。

    ![CDN quick create][1]
3. 在“订阅”下拉列表中选择所要使用的Azure 订阅（如果有多个订阅的话）。
4. 在“加速类型”下拉列表中选择加速类型。目前支持“WEB加速”，“下载加速”，“HTTP VOD（视频点播）加速”和“Live Streaming（视频直播）加速”。
5. 在“原始域类型”下拉列表中，选择云服务，存储账户，WEB应用，媒体服务（media service）或者自定义原始域。
6. 在“原始域”下拉列表中，从可用的云服务，存储帐户，WEB应用或者媒体服务列表中选择一个用于创建CDN终结点。如果“原始域类型”选择的是“自定义原始域”，那么请在“原始域”里输入您自己的原始域地址。
7. 在“自定义域”中输入要使用的自定义域名如：cdn.yourcompany.com
8. 在“原点主机标头（origin host header）”中输入您的源站所接受的回源访问host header。当您输入完“自定义域”之后，系统会根据您所选择的“原始域类型”来自动填充一个默认值。具体的规则是，如果您的源站是在Azure上的话，默认值就是相应的源站地址。如果您的源站不在Azure上，默认值是您输入的“自定义域名”，当然您也可以根据自己源站的实际配置情况来修改。
9. 在“ICP编号”中输入和上一步中所输入的自定义域名相对应的**ICP备案号**（如：京ICP备XXXXXXXX号-X）。
10. 单击“创建”按钮以创建新的终结点。
11. 终结点创建后将出现在订阅的终结点的列表中。列表视图显示了用于访问缓存内容的自定义域以及原始域。

原始域是 CDN 所缓存内容的原始位置。自定义域是用于访问CDN缓存内容的URL。
> **注意** 为终结点创建的配置将不能立即可用：

> 1. 首先需要审核所提供的自定义域名和ICP编号是否匹配、有效。这个过程需要最多**一个工作日**的时间来完成。
2. 如果ICP审核没有通过，您需要删除之前创建的这个CDN终结点，然后使用正确的自定义域名和ICP编号重新创建。
3. 如果ICP审核通过，CDN服务最多需要 **60 分钟**时间进行注册以便通过 CDN 网络传播。与此同时，您还需要按照界面上的提示信息配置CNAME映射信息，这样才可以最终通过自定义域名访问CDN缓存内容。

## 步骤 3:访问 CDN 内容<a id="step3"></a>
若要访问 CDN 上的缓存内容，请使用您在步骤2中所提供的自定义域名来访问CDN缓存内容。缓存 Blob 的地址类似于下面的地址（以步骤2中的例子为例）：

`http://cdn.yourcompany.com/<myPublicContainer>/<BlobName>`

## 步骤 4:删除 CDN 中的内容<a id="step4"></a>
如果您不想继续在 Azure 内容交付网络 (CDN) 中缓存对象，则可执行下列步骤之一：

- 对于 Azure Blob，可从公共容器中删除该 Blob。
- 生成专用容器代替公用容器。有关更多信息，请参见[限制对容器和 Blob 的访问](http://msdn.microsoft.com/zh-cn/library/dd179354.aspx)。
- 您可使用管理门户禁用或删除 CDN 终结点。
- 您可将云服务修改为不再响应此对象的请求。

已在 CDN 中缓存的对象将保持缓存状态，直到该对象的生存时间到期为止。当生存时间到期时，CDN 将查看 CDN 终结点是否仍有效，且是否仍可对该对象进行匿名访问。如果不能访问，则不再对该对象进行缓存。


## 步骤 5:使用高级管理功能<a id="step5"></a>
当您创建好CDN终结点之后，除了可以在管理门户中查看基本的配置信息和对CDN终结点做“禁用/启用”和“删除”等基本操作外，您还可以通过点击“管理”按钮，跳转到另外的管理页面进行高级管理功能：

![Manage Button][2]
> **注意：您将被引导至另外的CDN管理页面，它不属于Azure管理门户的一部分。（请注意允许浏览器打开新的窗口）**

![Adv Portal][3]

这个高级管理界面提供了“概览”，“域名管理”，“流量报表”，“带宽报表”，“缓存刷新”，“内容预取”，“日志下载”以及“服务检查”等功能。具体的使用和配置方式，可以点击进入相应的功能模块，然后查看对应的帮助文件。或者通过左侧导航栏的“帮助文档”，直接查看。




[步骤 1:创建存储帐户，云服务, Web 应用或媒体服务]: https://github.com/mccdn/cdndoc/blob/master/wacn/azurecdn.md#步骤-1创建存储帐户云服务 Web 应用或媒体服务
[步骤 2:创建新的 CDN 终结点]: #%E6%AD%A5%E9%AA%A4-2%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84-cdn-%E7%BB%88%E7%BB%93%E7%82%B9
[步骤 3:访问 CDN 内容]: #%E6%AD%A5%E9%AA%A4-3%E8%AE%BF%E9%97%AE-cdn-%E5%86%85%E5%AE%B9
[步骤 4:删除 CDN 中的内容]: #%E6%AD%A5%E9%AA%A4-4%E5%88%A0%E9%99%A4-cdn-%E4%B8%AD%E7%9A%84%E5%86%85%E5%AE%B9
[步骤 5:使用高级管理功能]: #%E6%AD%A5%E9%AA%A4-5%E4%BD%BF%E7%94%A8%E9%AB%98%E7%BA%A7%E7%AE%A1%E7%90%86%E5%8A%9F%E8%83%BD


<!--Image references-->
[1]: ./media/cdn/image005.png
[2]: ./media/cdn/image002.png
[3]: ./media/cdn/how_to_001.png