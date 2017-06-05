<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure China CDN API doc- purge" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-on, Live Streaming, 流媒体加速, CDN加速,CDN服务,主流CDN, 流媒体直播加速, 媒体服务, Azure Media Service, 缓存规则, HLS, CDN技术文档, CDN帮助文档, 视频直播加速, 直播加速" description="Learn How to create Live Streaming acceleration type CDN on Azure Management Portal and default caching rules for Live Streaming CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="6/6/2017"
    wacn.date="6/6/2017"
    wacn.lang="cn"
    />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-api-add-purge/)
- [English](/documentation/articles/cdn-enus-api-add-purge/) 

# 缓存刷新-添加缓存刷新

源站内容更新后，希望将更新结果实时反映在CDN服务节点上。由于CDN有默认或用户设置过的缓存规则，需要使用缓存刷新功能清楚节点上的缓存内容，这样之后用户再次访问该文件时获取到的就是更新后的文件了。

可以对单个或者批量的文件进行强制缓存刷新。

## 请求
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>方法</strong>
    </td>
  <th align="left"><strong>请求 URI</strong>
    </td>  
  <tr>
    <td>POST</td>
    <td>https://restapi.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}/purges?apiVersion=1.0</td>
  </tr>
</table>

### URI参数
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>  
  <tr>
    <td>subscriptionId</td>
    <td>订阅唯一标识</td>
  </tr
  <tr>
    <td>endpointId</td>
    <td>目标节点唯一标识</td>
  </tr>
</table>

### 请求 Headers

| 请求包头 | 描述 |
|:-----------|:-----------|
| x-azurecdn-request-date | 必填。符合yyyy-MM-dd hh:mm:ss格式的UTC当前请求时间 |
| Authorization | 必填。授权头请参考[CDN API签名机制](https://www.azure.cn/documentation/articles/cdn-api-signature/) |
| content-type | 必填。application/json |

### 请求 Body
添加缓存刷新，需写明一下参数，JSON示例文件如下：
```
{
  "Files": [
    "http://example.com/pictures/city.png"
  ],
  "Directories": [
    "http://example.com/pictures/"
  ]
}
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名称</strong>
  </th>
  <th align="left"><strong>描述</strong>
  </th>
  <tr>
    <td>Files </td>
    <td>刷新文件列表，路径必须为绝对路径如：http://example.com/pictures/city.png</td>
  </tr>
  <tr>
    <td>Directories </td>
    <td>刷新目录列表，路径必须为绝对路径如：http://example.com/pictures/</td>
  </tr>
</table>

## 响应

响应由状态码，响应 headers以及响应 body组成。
### 状态码
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>状态码</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>
  <tr>
    <td>202</td>
    <td>表明服务器成功接受请求</td>
  </tr>
  <tr>
    <td>其他</td>
    <td>表示出错的通用回复</td>
  </tr>
</table>

### 响应 Headers

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>响应包头</strong>
    </th>
  <th align="left"><strong>描述</strong>
    </th>

  <tr>
    <td>X-Correlation-Id</td>
    <td>该请求唯一标识，用于追踪请求信息。</td>
  </tr>
</table>

### 响应 Body
**请求成功的JSON示例文件如下**：
```
{
  "Succeeded": true,
  "IsAsync": true,
  "AsyncInfo": {
    "TaskTrackId": "b520c544-ec34-4ac4-86f5-5394363919c3",
    "TaskStatus": "Processing"
  }
}
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名称</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>

  <tr>
    <td>TaskTrackId</td>
    <td>刷新操作唯一标识，可用于查询刷新进度</td>
  </tr>
  <tr>
    <td>TaskStatus</td>
    <td>任务状态。
      <ul>
        <li>NotSet: 状态未知</li>
        <li>Processing: 正在处理</li>
        <li>Succeeded: 成功</li>
        <li>Failed: 失败</li>
      </ul>
    </td>
  </tr>
</table>

**请求失败的JSON示例**:
```
{
  "Succeeded": false,
  "ErrorInfo": {
    "Type": "MissingAuthorizationHeader",
    "Message": "Missing authorization header."
  }
}
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名称</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>

  <tr>
    <td>Type</td>
    <td>错误类型
         <ul>
            <li>CredentialInvalid:凭据不合法</li>
            <li>ParameterMissing:缺少参数</li>
            <li>ParameterInvalid：参数不合法</li>
            <li>MissingAuthorizationHeader:缺少Authorization请求头</li>
            <li>InvalidRequestDateHeader:请求时间不合法</li>
            <li>MissingRequestDateHeader：缺少请求时间头</li>
            <li>AuthorizationHeaderExpired:Authorization请求头已失效</li>
            <li>InvalidAuthorizationHeader:Authorization请求头不合法</li>
            <li>ApiKeyNotFound：API密钥不存在</li>
            <li>InvalidApiKey:API密钥不合法</li>
            <li>WrongSignature:签名不对</li>
            <li>SubscriptionNotFound：订阅不存在</li>
            <li>EndpointDoesNotBelongToSubscription：节点不属于订阅</li>
            <li>EndpointNotInActiveState：节点不处于活跃状态</li>
            <li>EndpointNotFound：节点不存在</li>
            <li>MaliciousItemPathDetected：检查到恶意路径</li>
            <li>PermissionDenied：权限不够</li>
            <li>RequestThrottled：请求被限流</li>
         </ul>    
    </td>
  </tr>
  <tr>
    <td>Message</td>
    <td>错误信息</td>
  </tr>
</table>
