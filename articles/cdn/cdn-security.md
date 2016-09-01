<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Overview of Azure CDN in China - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN, CDN加速, CDN服务, 主流CDN, 多场景加速, 免费CDN, CDN网站加速, 网站加速, 网页加速, 静态加速, 下载加速, VOD加速, 流媒体直播加速, 云服务,  存储账户,缓存刷新, 回源, 云加速, 加速效果, 节点, 流量, CNAME, 带宽, 网速, 防盗链,https加速, 低成本带宽, 访问加速, 小文件加速, 下载加速, 大文件加速, 流媒体加速, HTTPS安全加速, 缓存刷新, 内容预加载, 防盗链, 日志下载, CDN技术文档, CDN帮助文档, CDN FAQ" description="Learn the overview of Azure CDN, advantages, typical scenarios and key features." metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="4/7/2016"
    wacn.date="4/7/2016"
    wacn.lang="cn"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-overview/)

#Azure CDN安全传输机制

内容劫持是指运营商/链路供应者/黑客在路由器上接入旁路设备来侦测经过的HTTP请求，运营商/链路提供者/黑客按照一定的劫持策略识别出目标HTTP请求，并在源站反馈数据之前抢先客户端发送302相应，客户端被迫跳转到其他服务器请求内容，即对URL请求进行劫持。

不法分子可能通过篡改，用利己的内容替代终端用户请求的原始内容，以达到自己的目的，如非法获利、恶意宣传、竞争手段等。

内容劫持或者内容篡改导致终端用户访问出错或者下载了错误的文件。

通过CDN加速的端到端访问为终端用户访问CDN节点->CDN网内文件传输->CDN节点回源。Azure CDN保证CDN网内文件的安全传输，同时针对终端用户访问CDN节点和CDN节点回源之间的传输，Azure CDN制定了证书加密和智能选路的方式，来预防和处理劫持和内容篡改。源站本身的安全需要客户采取相应的措施。

##Azure CDN网内文件安全传输
Azure CDN加速保证CDN网内文件安全传输以及CDN节点文件存储安全。

###Azure CDN网内文件安全传输

CDN获取到的内容在CDN网内进行传输，通Azure CDN厂商私有传输协议，对数据包进行加密传输。

###CDN节点文件存储安全

1.	CDN节点文件缓存采用经Azure CDN合作厂商优化过的特殊HASH算法将文件映射到不规则的目录下，没有Azure CDN合作厂商HASH算法，是无法找到文件的存储路径的。通过这种方式，可确保文件安全性及隐蔽性，避免非法用户检索存储内容，杜绝缓存的内容被篡改。

2.	 Azure CDN合作厂商对节点存储文件内容进行了加密存储，即使文件存储路径被找到，由于Azure CDN合作厂商对存储文件进行了加密处理，在不知道文件加密算法的情况下，文件的内容即便被获取，也将变成不可识别的内容。可杜绝不法分子通过篡改内容来实现不法目的。


##终端用户访问CDN节点和CDN节点回源安全传输

###HTTPS加速
请求加密方案，即HTTPS加速解决方案。HTTPS协议是由SSL+HTTP协议构建的可进行加速传输、身份认证的网络协议。服务端和客户端之间通过建立加密的通信通道，保证数据在网络上之传输过程中不会被截取及窃听，这也就避免了被劫持的可能。

####HTTPS加速访问模式
Azure CDN合作厂商代为客户申请并将SSL证书部署到CDN节点->终端用户发起HTTPS请求连接-> SSL握手成功后将请求数据返回给用户。

###防劫持智能路由选择
小运营商在做内容劫持时通常根据一定概率拦截，且通常对特定链路的访问进行劫持。当客户有多个源时，相当于有多条回源路径，而多条链路同时发生运营商劫持的概率远远低于单条链路发生劫持。基于这个特点，当CDN回源被劫持，通过判断302跳转的location地址合法，选择相应或者尝试下一个回源路径。智能判断回源路径，可大大降低内容劫持的概率。

####防劫持智能路由选择访问模式
客户端发起请求到CDN节点-> CDN节点没有缓存->CDN节点请求回源A->回源时发生302跳转->判断302跳转合法性，判断为劫持->请求回源自动尝试下一个源B->源B正常响应。



