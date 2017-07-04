<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="How to create Download acceleration type CDN - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN加速, CDN服务, 云加速, 下载加速, Download, 缓存规则, ICP, ICP备案号, ICP编号, 技术文档, 帮助文档, 带宽, 大文件下载, 软件升级安装包, 游戏下载加速, 应用程序下载加速, 手机app更新, 系统固件升级" description="Learn How to create Download acceleration type CDN on Azure Management Portal and default caching rules for Download CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="7/72016"
    wacn.date="7/7/2016"
    wacn.lang="cn"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-how-to-create-Download-CDN-endpoint/)
- [English](/documentation/articles/cdn-enus-how-to-create-Download-CDN-endpoint/) 
#下载加速CDN节点创建


下载加速主要针对20MB以上的大文件下载，例如软件安装包、游戏客户端、应用程序、影音等大文件的下载分发。Azure CDN将文件缓存到CDN边缘节点，缓解源站下载的带宽压力，提高用户下载体验。

下载加速适用于操作系统固件升级，游戏客户端，手机app更新，应用程序下载等用户场景。

本文是针对下载类型加速域名创建，您也可以参考[使用Azure CDN](/documentation/articles/cdn-how-to-use/)了解基本的Azure CDN加速节点创建信息。

###**下载加速默认缓存规则**
Azure CDN针对下载加速设置了默认缓存规则（见下文）。您也可以根据需求自定义设置缓存规则，具体请参考Azure CDN管理门户高级管理的帮助文档“域名管理”。如果源站内容更改或者更新，同时设置的缓存生存时间未到期，可以通过手动刷新CDN缓存文件实时同步源站更新的内容，具体请参考Azure CDN管理门户高级管理的帮助文档“缓存刷新”。

**下载加速系统默认缓存规则**

1. 对php、aspx、asp、jsp、do等动态文件不缓存。
2. 对7z、apk、 wdf、 cab、 dhp、exe、flv、gz、ipa、iso、mpk、MPQ、pbcv、pxl、qnp、r00、rar、xy、xy2、zip、CAB等文件缓存一个月。

###**创建下载类型加速域名**

1. 在 Azure 管理门户的导航窗格中，单击“CDN”。
2. 在功能区上，单击“新建”。在“新建”对话框上，依次选择“应用服务”、“CDN”和“快速创建”。
3. 在“加速类型”下拉列表中选择“下载加速”。

    ![011](./media/cdn-doc/011.png)

4. 在“原始域类型”下拉列表中，选择云服务，存储账户，WEB应用，或者自定义原始域。**注意**“下载加速”不支持“媒体服务”原始域类型。
5. 在“原始域”下拉列表中，从可用的云服务，存储帐户，或者WEB应用列表中选择一个用于创建CDN终结点。  
    ![012](./media/cdn-doc/012.png)

    如果“原始域类型”选择的是“自定义原始域”，那么请在“原始域”里输入您自己的原始域地址。您可以填写一个或者多个原始域ip地址，多个请以“;”分隔，如“126.1.1.1;172.1.1.1），或者原始域名，如origin.azurechina.com

    ![008](./media/cdn-doc/008.png)

6. 在“自定义域”中输入要使用的自定义域名如：cdn1.azurechinatest.com。自定义域支持泛域名加速。
7. 在“原点主机标头（origin host header）”中输入您的源站所接受的回源访问host header。当您输入完“自定义域”之后，系统会根据您所选择的“原始域类型”来自动填充一个默认值。具体的规则是，如果您的源站是在Azure上的话，默认值就是相应的源站地址。如果您的源站不在Azure上，默认值是您输入的“自定义域名”，当然您也可以根据自己源站的实际配置情况来修改。

    原始域类型是存储账户，对应的回源主机标头：

    ![007](./media/cdn-doc/007.png)  
    
    原始域类型是自定义原始域对应的回源主机标头：

    ![013](./media/cdn-doc/013.png)
    
      
8. 在“ICP编号”中输入和上一步中所输入的自定义域名相对应的ICP备案号（如：京ICP备XXXXXXXX号-X）。
     
    ![009](./media/cdn-doc/009.png) 

9. 单击“创建”按钮以创建新的终结点。

终结点创建后将出现在订阅的终结点的列表中。列表视图显示了用于访问缓存内容的自定义域以及原始域。
原始域是 CDN 所缓存内容的原始位置。自定义域是用于访问CDN缓存内容的URL。

   ![010](./media/cdn-doc/010.png)

> **注意** 为终结点创建的配置将不能立即可用，需要审核所提供的ICP自定义域名和ICP编号是否匹配，详情请参考[使用Azure CDN](/documentation/articles/cdn-how-to-use/)中步骤2：创建新的CDN终结点的后半部分。

