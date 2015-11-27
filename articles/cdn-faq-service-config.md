<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Windows Azure CDN FAQ - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN FAQ, CDN常见问题, CDN加速, CDN服务, 配置CNAME, CNAME, CNAME记录, 缓存刷新, 缓存规则, CDN边缘节点, CDN技术文档, CDN帮助文档" description="Find answers to service configuration related to Windows Azure CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service=""
    ms.date=""
    wacn.date="11/27/2015"
    />

#常见问题 - 服务配置

### **如何配置CNAME**
到域名托管商找到该域名解析管理—删除该域名的A记录—添加一条cname记录，cname的域名我们已经给出。

### **怎么确认我的CNAME记录生效了**
各地DNS的生效时间不一致，取决于域名对应的原有记录的生效时间（TTL时间）。当ping（或者dig）域名，给出的解析不再是您源站IP，说明已经生效了。

### **设置CDN后，如何保证内容和源站的同步**
 
1. 设置缓存规则，针对不同的内容设置不同的缓存刷新规则，对更新频繁的内容，可以设置较短的缓存时间； 对于不经常更新的内容，可以设置较长的缓存时间，从而减小源站压力。
      
2. 若设置的缓存刷新周期未到，但是有新内容发布或者删除部分内容，可以使用Windows Azure CDN管理平台提供的缓存刷新功能，进行手动强行刷新。

### **更换原站服务器如何操作**

首先要确保新的源站服务器能正常服务，然后在Windows Azure CDN高级管理平台—域名管理中将原站地址变更为新的地址，保存即可。

### **如何设置缓存刷新**

1. 设置缓存规则：可以到Windows Azure CDN管理平台上针对不同的内容设置不同的缓存刷新规则，对更新频繁的内容，可以设置较短的缓存时间； 对于不经常更新的内容，可以设置较长的缓存时间，从而减小源站压力。
   
2. 手动刷新：若设置的CDN缓存内容生存时间未到期，但是有新内容发布或者删除部分内容，可以根据需求使用Windows Azure CDN高级管理平台提供的缓存刷新功能，设置文件刷新和目录刷新，从而进行手动刷新。

### **源站日志中如何获取访问者的原始IP**

网站通过CDN加速后，其访问来源绝大部分将会来自于CDN缓存节点。CDN回源时，会在HTTP Header **X-Forwarded-For** 中填入原始IP，源站的Web服务器可以修改日志配置记下该信息。

以 Nginx 为例，其配置文件可以加入如下信息：

log_format logCDN '$remote_addr forwarded for $http_x_forwarded_for - $remote_user [$time_local]  '

'"$request" $status $body_bytes_sent '

'"$http_referer" "$http_user_agent"';
      
access_log /var/log/nginx/access.log logCDN;

###**如何绑定CDN边缘节点**
 
用户通过自定义域名 _cdn.mydomain.com_ 为自己的源站 _www.mydomain.com_ 在 Azure CDN 平台成功创建 CDN 服务后，获得加速域名 _cdn.mydomain.com.mschcdn.com_。用户可以绑定host进行一些基本排查。 步骤如下：

1. ping cdn.mydomain.com.mschcdn.com 得到边缘节点的IP地址，如：a.b.c.d
2. 修改本地hosts文件，添加纪录 "a.b.c.d www.domain.com
     
用户打开浏览器访问 _www.domain.com_，如果打开网站显示成功，则说明CDN没有问题。如果无法访问，而hosts文件中IP地址改为源站的IP地址后可以成功访问，则说明CDN服务器有问题。

>**注意**
>Windows下，hosts文件的路径为 C:\Windows\System32\drivers\etc\hosts。     
>Linux、BSD等UNIX类操作系统下该文件路径为 /etc/hosts。修改该文件需要管理员权限。
>**注意**
>对于Windows Azure Blob 和 Cloud Service，直接访问域名会得到404。此时可以通过访问一个有效URI来排查。




