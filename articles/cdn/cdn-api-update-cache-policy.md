<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure China CDN API doc-update cache policy" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-on, Live Streaming, 流媒体加速, CDN加速,CDN服务,主流CDN, 流媒体直播加速, 媒体服务, Azure Media Service, 缓存规则, HLS, CDN技术文档, CDN帮助文档, 视频直播加速, 直播加速" description="Learn How to create Live Streaming acceleration type CDN on Azure Management Portal and default caching rules for Live Streaming CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="6/6/2017"
    wacn.date="6/6/2017"
    wacn.lang="cn"
    />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-api-update-cache-policy/)
- [English](/documentation/articles/cdn-enus-api-update-cache-policy/) 

# 节点管理-设置缓存规则

通过该API可以针对目录，文件名和全路径配置缓存规则。

* 根据目录进行配置：目录必须以 "/" 开头，比如： "/pic", "/doc", "/htdoc/data" 等等。后台会匹配指定目录下的所有文件，包括子目录*
* 根据文件后缀配置：常用文件后缀名，比如："jpg", "png", "gif", "txt", "m4v", "mp3" 等等。后台会匹配 所有文件夹下 指定的文件后缀。
* 根据全路径配置：用来指定 一个文件，必须以 "/" 开头。比如："/sites/doc/example.doc"。注意：如果用户填的全路径是"/"，则它匹配首页。

>注意 用户填写配置规则时，字符串中不要包含 “{”, “}”, “(”, “)”, “[”, “]”, “.”, “?", “*”, “\”, “^”, “$” 等特殊字符。

>注意 **时间填为0表示禁止缓存，不缓存规则优先执行。**

## 请求
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>方法</strong>
    </td>
  <th align="left"><strong>请求 URI</strong>
    </td>  
  <tr>
    <td>PUT</td>
    <td>https://restapi.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}/cacherules?apiVersion=1.0</td>
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
```
{
  "Rules": [
    {
      "Type": "Dir",
      "Items": [
        "/test1/",
        "/test2/"
      ],
      "TTL": 86400
    },
    {
      "Type": "Suffix",
      "Items": [
        "7z",
        "apk",
        "wdf",
        "cab"
      ],
      "TTL": 2592000
    },
    {
      "Type": "FullUri",
      "Items": [
        "/test1/test.xml",
        "/test2/test.txt"
      ],
      "TTL": 86400
    },
  ],
  "IgnoreCacheControl": false,
  "IgnoreCookie": false,
  "IgnoreQueryString": false
}
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名称</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>
  <tr>
    <td>TTL</td>
    <td>缓存时间,单位为秒</td>
  </tr>
  <tr>
    <td>IgnoreCacheControl</td>
    <td>是否忽略返回头中的cache-control头部对请求内容进行缓存
    </td>
  </tr>
  <tr>
    <td>IgnoreCookie</td>
    <td>是否忽略返回头中的set-cookie头部对请求内容进行缓存
    </td>
  </tr>
  <tr>
    <td>IgnoreQueryString</td>
    <td>是否忽略查询参数对请求内容进行缓存
    </td>
  </tr>
  <tr>
    <td>Type</td>
    <td>缓存规则类型。
        <ul>
         <li>Suffix: 基于文件拓展名进行缓存</li>
         <li>Dir: 对指定目录下的所有文件进行缓存</li>
         <li>FullUri: 对指定路径的文件进行缓存</li>
        </ul>
    </td>
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
**请求成功的JSON示例**:

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
    <td>操作唯一标识，可用于查询进度</td>
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
