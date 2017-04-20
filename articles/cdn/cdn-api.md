<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure China CDN API doc" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-on, Live Streaming, 流媒体加速, CDN加速,CDN服务,主流CDN, 流媒体直播加速, 媒体服务, Azure Media Service, 缓存规则, HLS, CDN技术文档, CDN帮助文档, 视频直播加速, 直播加速" description="Learn How to create Live Streaming acceleration type CDN on Azure Management Portal and default caching rules for Live Streaming CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="4/20/2017"
    wacn.date="4/20/2017"
    wacn.lang="cn"
    />
 
# Restful API

##简述

用户可以使用该接口管理Azure China CDN资源。通过向CDN API服务器发送HTTPS请求，并按照接口说明在请求中加入相应的参数即可获得相应结果。

- 节点管理- 在指定Azure订阅下创建、删除、启用、禁用CDN节点；查询、更新CDN节点信息，比如修改CDN源站等；设定缓存规则
- 流量查询-  按月、天、小时、分钟，根据订阅或者单个域名查看流量信息
- 内容管理- 缓存刷新、预加载
##密钥申请

验证和授权方式基于接口密钥，密钥申请需要到[Azure CDN新版管理门户](https://www.azure.cn/documentation/articles/cdn-management-v2-portal-how-to-use/)申请。

##查看API文档

请参照如下步骤调用Azure China CDN API：

1. 下载[JSON格式的API描述文件](https://github.com/mccdn/publicdoc/blob/master/API/REST-API-v1.json)
2. 打开[Swagger](http://editor.swagger.io/#!/)
2. 点击“File”->"Import File"，导入第一步操作中下载的JSON格式的API描述文件
