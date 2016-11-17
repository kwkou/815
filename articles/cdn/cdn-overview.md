<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Overview of Azure CDN in China - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN, CDN加速, CDN服务, 主流CDN, 多场景加速, 免费CDN, CDN网站加速, 网站加速, 网页加速, 静态加速, 下载加速, VOD加速, 流媒体直播加速, 云服务,  存储账户,缓存刷新, 回源, 云加速, 加速效果, 节点, 流量, CNAME, 带宽, 网速, 防盗链,https加速, 低成本带宽, 访问加速, 小文件加速, 下载加速, 大文件加速, 流媒体加速, HTTPS安全加速, 缓存刷新, 内容预加载, 防盗链, 日志下载, CDN技术文档, CDN帮助文档, CDN FAQ" description="Learn the overview of Azure CDN, advantages, typical scenarios and key features." metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="cn"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-overview/)
- [English](/documentation/articles/cdn-enus-overview/) 
# Azure CDN （内容交付网络）概述



Azure CDN （内容传送网络） 通过遍布在中国大陆的众多物理节点上缓存Azure平台上的Storage Blob，Cloud Service和WebSites的静态内容，以及为媒体服务提供流式内容分发提供加速，为开发人员提供一个传送高带宽内容的解决方案。目前本CDN服务也同时支持没有部署在Azure平台上的源站。

有关 Azure CDN 的详细信息和价格，请参阅 [Azure CDN服务介绍](/home/features/cdn/)。

如果您已经是Azure CDN的现有用户，请访问[Azure CDN管理门户](https://manage.windowsazure.cn)来管理CDN加速域名。具体使用指南请参见[使用 Azure CDN](/documentation/articles/cdn-how-to-use/)

+ [什么是CDN](#step1)
+ [Azure CDN优势](#step2)
+ [Azure CDN功能](#step3)

## 什么是CDN<a id="step1"></a>

CDN 的全称是 Content Delivery Network，即内容传送网络。目前几乎所有大型网站都在使用这一技术，但该技术并非大型网站的专利。其基本思路是尽可能避开互联网上有可能 影响数据传输速度和稳定性的瓶颈和环节，使内容传输的更快、更稳定。通过在网络各处放置节点服务器，以互联网为基础构建一层更智能的虚拟网络，CDN 系统能够实时地根据网 络流量和各节点的连接、负载状况，以及到用户的距离和响应时间等综合信息，将用户的请 求重新导向距离用户最近的服务节点。
 
以微软公司的网站为例，微软是一家美国公司，但客户遍及全球。这也意味着全球各地都有 用户可能需要访问微软网站，或从 Microsoft Update 下载产品更新。如果只将网站服务器部署在美国的一个位置，该地区周围的用户访问时无疑可以获得满意的速度，毕竟距离近， 网络延迟也低。但如果中国的用户需要通过美国这一位置的服务器下载产品更新，就需要在中国和美国之间往返传输所有数据包。由于线路总长度高达数千公里，这会造成极大的延迟。

为了解决这个问题，可以在中国放置 CDN节点。CDN节点可以自动将数据缓存到全中国主要城市的数据中心内，让内容和用户之间的距离更短，进而降低传输数据所需的时间。通过 CDN 的缓存，用户可以就近获得所需内容，解决 Internet 网络拥挤的状况，提高用户访问网站和应用程序的响应速度。


![][4]


## 优势<a id="step2"></a>

### 对多种Azure服务内置支持

默认对包括Storage Blob，Cloud Service， Websites，媒体服务等在内的多种Azure服务的原生支持，为用户提供完整的一站式云服务支持。

![][1]


### 全自助化服务

传统的CDN服务需要很多复杂冗长的配置流程。对于Azure CDN用户来说，从创建CDN加速节点，到之后的对CDN加速节点整个生命周期的管理，以及各种统计报表的查询，原始访问日志下载，各种高级功能（如缓存规则配置，缓存内容强制刷新，内容预加载，防盗链等）的配置，均可以通过Azure管理门户以及专门的CDN管理门户自助完成。

![][2]  

![][3]

### 全网节点动态优化

整合国内多家主流CDN服务，提供全面的静态网页加速，软件安装包、游戏客户端、应用程序、影音等大文件的下载分发，以及在线视频网站、在线教育网站等以流媒体为主的视频点播和直播等多种业务类型加速，满足不同类型资源的分发需求；提供包含电信、联通、移动等主流电信运营商，以及其他 ISP运营商，全地区的全网覆盖，根据网络实时状况，通过负载均衡技术和智能调度策略，将用户请求分配到最优节点。


### 节省成本

依托和多家主流CDN服务的合作优势，Azure CDN可以给包括企业客户，网站支付用户在内的所有Azure用户，提供质优价廉的CDN服务。让更多的用户可以享受到CDN服务所带来的红利。

## 多场景加速

Azure CDN服务支持下面描述的多种加速场景，为用户打造出全方位多维度的加速服务。

![][8]

### 网站、小文件加速

CDN的一个典型使用场景就是针对所谓的众多网站所使用的“小文件”（html网页文件，图片文件，JavaScript，CSS文件等）进行加速，使网站得到更好的用户体验，从而带来更多的用户访问量，最终带动整个业务的总体收入。典型的用户群体是面向互联网用户提供网站服务的众多大中小企业。

### 大文件下载分发

CDN另外一个典型的应用场景就是针对大文件下载进行多节点分发，从而为最终的下载体验保驾护航。典型的用户场景如操作系统固件升级，新游戏发布上线（需要下载客户端安装包），手机APP更新等，如果没有使用CDN服务，就会对源站的带宽使用造成巨大的冲击，甚至导致源站停止服务。

### 流媒体加速

近年来，随着网路视频媒体服务的增加，越来越多的人们习惯于使用网络平台来收听观看各种音视频。再加上国内网络环境的条件限制，这就对音视频内容的最终分发提出了非常高的要求。典型的用户群体是面向互联网用户提供服务的各类媒体网站，手机APP客户端。

### HTTPS安全加速

安全永远是用户最关心的话题，具体到CDN服务，Azure CDN除了提供全面完善的HTTP访问类型的加速以外，还为对HTTPS访问协议有需要的用户提供专门的HTTPS类型加速服务。该加速服务从使用场景来说属于小文件加速服务，并能同时提供一定的动态路由访问优化服务。


## 功能<a id="step3"></a>

### 全自助化创建管理CDN加速节点

包括CDN加速节点的创建，删除，启用，禁用以及源站修改等功能在内，Azure CDN对CDN加速节点的整个生命周期，提供全面自助化的管理配置服务。

![][5]

### 流量带宽信息可视化查询

通过专有的Azure CDN管理门户可以方便、快捷、清晰的查看CDN加速域名相关的流量和带宽的使用情况。

![][6]

![][7]


### 缓存规则配置

系统已经针对不同的CDN加速类型提供了默认的缓存规则。用户也可以根据自己的实际需求进行定制化修改，以达到灵活控制缓存内容时间的目的。

![][9]

### 防盗链

针对国内用户的特殊需求，Azure CDN提供了防盗链等CDN内容访问控制功能。用户可以使用这些功能更好的对自己的加速内容做到有效的控制，以达到保护内容的目的。

![][10]


### 缓存刷新

有时用户更新完源站的某个文件之后，希望实时看到更新的结果能够反映在CDN服务节点上。但由于CDN有默认或用户设置过的缓存规则，所以一般不能实时的在所有的CDN节点上反映出更新变化。这时就是“缓存刷新”功能大显身手的时候了，用户可以对单个或者批量的文件进行强制缓存刷新。目的是让CDN服务将用户指定的文件从所有的CDN节点上清除，这样之后用户再次访问该文件时获取到的就是更新后的文件了。

![][11]

### 内容预加载

内容预加载是指预先将指定URL的内容从源站缓存到CDN节点，这样可以消除用户第一次访问该资源时的等待时间。内容预取一般被用在进行大文件分发时的场景，可以有效的提升用户访问体验。

![][12]

### 日志下载

用户有时需要对CDN的加速效果，原始访问信息等做一些统计分析，这时用户就可以通过“日志下载”功能来获取这些原始的访问信息。在使用这个功能时需要用户提供一个可使用的Azure Storage帐户，Azure CDN用来存放对应客户的日志访问文件。

![][13]

### 服务检查

用户在创建CDN服务终节点后，可以在“服务检查”视图查看源站是否可以访问，CDN部署是否完成，CDN缓存是否正常。

![][14]

<!--Image references-->
[1]: ./media/cdn-overview/overview01.png
[2]: ./media/cdn-overview/image005.png
[3]: ./media/cdn-overview/overview02.png
[4]: ./media/cdn-overview/overview03.png
[5]: ./media/cdn-overview/overview04.png
[6]: ./media/cdn-overview/overview05.png
[7]: ./media/cdn-overview/overview06.png
[8]: ./media/cdn-overview/overview07.png
[9]: ./media/cdn-overview/overview08.png
[10]: ./media/cdn-overview/overview09.png
[11]: ./media/cdn-overview/overview10.png
[12]: ./media/cdn-overview/overview11.png
[13]: ./media/cdn-overview/overview12.png
[14]: ./media/cdn-overview/overview13.png