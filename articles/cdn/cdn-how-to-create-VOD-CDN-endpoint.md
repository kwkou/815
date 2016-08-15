<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="How to create VOD acceleration type CDN - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN加速,CDN服务,主流CDN, VOD, 视频点播加速, VOD加速, 缓存规则, 媒体服务, Azure Media Service, CDN技术文档, CDN帮助文档" description="Learn How to create VOD acceleration type CDN on Azure Management Portal and default caching rules for VOD CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="cn"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-how-to-create-VOD-CDN-endpoint/)
- [English](/documentation/articles/cdn-enus-how-to-create-VOD-CDN-endpoint/) 
#VOD视频点播加速CDN节点创建


VOD视频点播加速服务主要针对在线音视频点播提供加速服务。随着网路视频媒体服务的兴起，越来越多的用户选择使用网络平台收听观看各种音视频。加之国内网络环境的限制，对音视频内容的最终分发提出了非常高的要求。Azure CDN将音频、视频等流媒体内容分发缓存到CDN边缘节点，将用户请求指向最优节点，减少源站服务器的负载，节省带宽资源，给用户提供高速、流畅、高质量的在线视频体验。

Azure CDN VOD视频点播加速支持Azure内置的[媒体服务](/home/features/media-services/)。

VOD视频点播加速适用于各类在线音视频点播网站，如媒体类视频网站，在线教育网站，移动端APP客户端等。

本文是针对VOD视频点播加速域名创建，您也可以参考[使用Azure CDN](/documentation/articles/cdn-how-to-use/)了解基本的Azure CDN加速节点创建信息。

###**视频点播加速默认缓存规则**
Azure CDN针对VOD视频点播加速设置了默认缓存规则（见下文）。您也可以根据需求自定义设置缓存规则，具体请参考Azure CDN管理门户高级管理的帮助文档“域名管理”。如果源站内容更改或者更新，同时设置的缓存生存时间未到期，可以通过手动刷新CDN缓存文件实时同步源站更新的内容，具体请参考Azure CDN管理门户高级管理的帮助文档“缓存刷新”。

**VOD视频点播加速系统默认缓存规则**

1. 对php、aspx、asp、jsp、do等动态文件不缓存
2. 对MP3、wma等缓存1天
3. 对mwv、html、htm、shtml、hml、gif、swf、png、bmp、js等缓存1小时
4. 对7z、apk、 wdf、 cab、 dhp、exe、flv、gz、ipa、iso、mpk、MPQ、pbcv、pxl、qnp、r00、rar、xy、xy2、zip、CAB等文件缓存一个月
      
###**创建VOD视频点播加速域名**

1. 在 Azure 管理门户的导航窗格中，单击“CDN”。
2. 在功能区上，单击“新建”。在“新建”对话框上，依次选择“应用服务”、“CDN”和“快速创建”。
3. 在“加速类型”下拉列表中选择“VOD加速”。

    ![016](./media/cdn-doc/016.png)

4. 在“原始域类型”下拉列表中，选择云服务，存储账户，Web应用，媒体服务或者自定义原始域。
5. 在“原始域”下拉列表中，从可用的存储帐户，媒体服务列表中选择一个用于创建CDN终结点。
  
    ![017](./media/cdn-doc/017.png)

    如果“原始域类型”选择的是“自定义原始域”，那么请在“原始域”里输入您自己的原始域地址。您可以填写一个或者多个原始域ip地址，多个请以“;”分隔，如“126.1.1.1;172.1.1.1），或者原始域名，如origin.chazuretest.com。

    ![018](./media/cdn-doc/018.png)

6. 在“自定义域”中输入要使用的自定义域名如：cdn.chazuretest.com。自定义域支持泛域名加速。
7. 在“原点主机标头（origin host header）”中输入您的源站所接受的回源访问host header。当您输入完“自定义域”之后，系统会根据您所选择的“原始域类型”来自动填充一个默认值。具体的规则是，如果您的源站是在Azure上的话，默认值就是相应的源站地址。如果您的源站不在Azure上，默认值是您输入的“自定义域名”，当然您也可以根据自己源站的实际配置情况来修改。

    原始域类型是媒体服务，对应的回源主机标头：

    ![020](./media/cdn-doc/020.png)  
    
    原始域类型是自定义原始域对应的回源主机标头：

    ![019](./media/cdn-doc/019.png)
          
8. 在“ICP编号”中输入和上一步中所输入的自定义域名相对应的ICP备案号（如：京ICP备XXXXXXXX号-X）。
     
    ![021](./media/cdn-doc/021.png) 

9. 单击“创建”按钮以创建新的终结点。

终结点创建后将出现在订阅的终结点的列表中。列表视图显示了用于访问缓存内容的自定义域以及原始域。
原始域是 CDN 所缓存内容的原始位置。自定义域是用于访问CDN缓存内容的URL。

   ![022](./media/cdn-doc/022.png)

>**注意** 为终结点创建的配置将不能立即可用，需要审核所提供的ICP自定义域名和ICP编号是否匹配，详情请参考[使用Azure CDN](/documentation/articles/cdn-how-to-use/)中步骤2：创建新的CDN终结点的后半部分。

