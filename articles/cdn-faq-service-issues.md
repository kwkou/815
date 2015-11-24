<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Windows Azure CDN FAQ - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons" description="Find answers to common service consulting or inquiries related to Windows Azure CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service=""
    ms.date=""
    wacn.date="11/24/2015"
    />
#常见问题
##使用故障

- **为什么我的URL不能缓存？**

    URL不能被缓存，通常有以下几个原因： 

  1. 源站的该URL响应Header里含有以下信息：
     - Set-Cookie（且缓存规则里并未勾选忽略Set-Cookie选项）。注：Set-Cookie在用于用户登录和身份识别时是不能勾选忽略Set-Cookie选项的，否则可能引起功能性问题。
     - Cache-Control：no-store/no-cache/private（且缓存规则里并未勾选忽略Cache-Control选项）。
     - Expires的时间是过去的某个时间，Expires指定了缓存到期时间点，如果是过去时间，则将导致无法缓存。
     - Max-age的值很小，Max-age指定了缓存时间长度，单位为秒，如果太小，如小于两位数，那么很快就会过期，导致无法缓存。
      
  2. 缓存规则里没有配置或配置错误，URL无法命中任何一个缓冲规则，例如，有用户不小心录入以下规则:"\[任意字符\]\(.gif|.jpg|.bmp\) (\.gif|.jpg|.bmp\)"，那么即使是图片类型也无法命中规则，因为扩展名重复。
   
  3. 部分节点暂时还没有用户访问该URL，需要有访问之后才会缓存

- **为什么域名会解析到源站？**

    如果一个域名CDN的加速区域为中国大陆，海外网民或者中国网民使用非中国DNS解析就有可能会解析到源站。Azure CDN平台针对这种情况默认配置为用国内CDN节点来服务海外访问。

- **系统提示不能CNAME**

    有可能，虽然我们没有限制加速域名但是有的域名托管商不允许不带有主机名的域名进行cname，对此，没有更好的办法：要么更换加速域名，要么更换域名托管商。

- **为什么回原大于CDN流量？**

    一般回原流量<=CDN流量，特殊情况也可能出现回原流量大于CDN流量，比如：如果访问者发起一个请求，请求一个比较大的文件。比如100M，如果节点没有缓存的情况，CDN节点就会去源站获取，这么大文件必然需要点时间，然而此时，访问者又不继续等了于是就断开连接。这样，CDN节点还是会去把100M文件全都拿过来。此时发现访问者已经不要它了，它就没法再返回了，于是，回源的流量就有100M，而CDN流量没有。

- **用了CDN后打开速度更慢**

    缓存命中率不高，影响缓存命中率的几个原因：

  - 缓存配置的问题
     
  - Http Header导致无法缓存
     
  - 刚添加CDN加速节点，缓存的文件还不多
     
  - 源站类型，可缓存的内容少
     
  - 网站访问量低，过期时间短，命中的文件少

- **回源比例比较大** 

  1. 缓存规则配置问题
    
  2. 可缓存资源少
    
  3. 缓存时间短    

- **使用CDN后，网站打不开** 

    可能因素：
 
   1. 原站故障
     
   2. 原站有设置防火墙，屏蔽CDN节点
    
   3. 节点屏蔽客户IP
     
   4. 新添加域名/域名状态变更，节点配置下发问题，需要等一段时间一般60分钟左右
     
   5. 设备故障  
    
- **使用CDN后，网站登录异常**

    很有可能是在缓存规则中设置了不该缓存的资源，需要将用户后台登陆的文件夹设置不缓存（如： /user  或者 /admin等等）；如果是非登录或用户回话网站，缓存规则配置中请开启允许忽略Cookie。

- **缓存刷新失败**

    手动提交刷新后系统会统计正常服务的设备来下发任务，如果任务下发过程中某一台设备出现异常，系统会自动检测出来并探测关闭（服务状态关闭后客户不会解析到该节点，不影响加速服务），其他正常设备继续下发任务，直至完成，此时界面就会显示“刷新失败”，同时客服和运维会接到邮件，第一时间做处理，即便是刷新失败不影响CDN服务。

- **为什么没有CNAME到Windows Azure CDN却消耗CDN流量**

    有可能，因为只要加速域名的服务状态是开启的，即便没有CNAME到Windows Azure CDN， 我们的探测设备都会通过监控url探测原站和加速情况，这个也是为什么我们建议您选择监控url尽可能选择小一些的文件，如果您不想未CNAME的加速域名消耗流量可以将服务切换至关闭状态。