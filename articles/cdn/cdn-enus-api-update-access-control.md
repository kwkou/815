<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN API doc-update access control"
    metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-on, Live Streaming, Streaming media acceleration, CDN acceleration, CDN services, mainstream CDN, live streaming media acceleration, media services, Azure Media Service, cache rules, HLS, CDN technology files, CDN help files, live video acceleration, live broadcast acceleration"
    description="Learn How to Create Live Streaming Acceleration Type CDNs on Azure Management Portal and Default Caching Rules for Live Streaming CDNs"
    metaCanonical=""
    services="cdn"
    documentationCenter=".NET"
    authors="v-jijes"
    solutions=""
    manager=""
    editor="" />
<tags
    ms.service="cdn_en"
    ms.author="v-jijes"
    ms.topic="article"
    ms.date="6/6/2017"
    wacn.date="6/6/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-api-update-access-control/)
- [English](/documentation/articles/cdn-enus-api-update-access-control/)

# <a name="-"></a>CDN endpoint management – configure access control rules


You can use access control management to set up and configure referer blacklists and whitelists, and thereby implement an anti-theft chain.

- If you set up a blacklist, access is denied if the referer is in the blacklist, but is otherwise permitted.
- If you set up a whitelist, access is permitted only if the referer is one of the domain names in the whitelist.

## <a name=""></a>Request
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Method</strong>
    </td>
  <th align="left"><strong>Request URI</strong>
    </td>  
  <tr>
    <td>PUT</td>
    <td>https://restapi.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}/accesscontrol?apiVersion=1.0</td>
  </tr>
</table>

### <a name="uri"></a>URI parameter
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>  
  <tr>
    <td>subscriptionId</td>
    <td>Subscription unique identifier</td>
  </tr
  <tr>
    <td>endpointId</td>
    <td>Target CDN endpoint unique identifier</td>
  </tr>
  </table>

### <a name="-headers"></a>Request header

| Request header | Description |
|:-----------|:-----------|
| x-azurecdn-request-date | Must be entered. Current UTC request time in yyyy-MM-dd hh:mm:ss format |
| Authorization | Must be entered. Please refer to [CDN API signing mechanism](/documentation/articles/cdn-enus-api-signature/) for authorization headers. |
| content-type | Must be entered. application/json |

### <a name="-body"></a>Request body

    {
      "ForbiddenIps": [
        "192.160.1.1",
        "220.100.100.100"
      ],
      "RefererControl": {
        "Enabled": "true",
        "PathPatterns": [
          "/test1/test1.jpg"
        ],
        "Referers": [
          "http://www.abc.com",
          "http://www.micorosft.com"
        ],
        "RefererControlType": "AllowList"
      }
    }

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>
  <tr>
    <td>ForbiddenIps</td>
    <td>List of forbidden IPs</td>
  </tr>
  <tr>
    <td>PathPatterns</td>
    <td>[anti-]theft link file path connection
    </td>
  </tr>
  <tr>
    <td>Referers</td>
    <td>[anti-]theft link connection
    </td>
  </tr>
  <tr>
    <td>RefererControlType</td>
    <td>Anti-theft chain type.
         <ul>
         <li>AllowList: Whitelist, only allows links for the matching referrers to access the matching PathPatterns paths</li>
         <li>BlockList: Blacklist, links with matching referrers will be denied access if they attempt to access the matching PathPatterns paths</li>
        </ul>
    </td>
  </tr>
</table>

## <a name=""></a>Response

A response comprises a status code, response headers, and a response body.
### <a name=""></a>Status code
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Status code</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>
  <tr>
    <td>202</td>
    <td>Indicates that the server has successfully accepted the request</td>
  </tr>
  <tr>
    <td>Other</td>
    <td>General response indicating that an error has occurred</td>
  </tr>
</table>

### <a name="-headers"></a>Response header

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Response header</strong>
    </th>
  <th align="left"><strong>Description</strong>
    </th>

  <tr>
    <td>X-Correlation-Id</td>
    <td>The request’s unique identifier is used to track request information.</td>
  </tr>
</table>

### <a name="-body"></a>Response body
**Example JSON file for request succeeded is provided below**:

    {
      "Succeeded": true,
      "IsAsync": true,
      "AsyncInfo": {
        "TaskTrackId": "b520c544-ec34-4ac4-86f5-5394363919c3",
        "TaskStatus": "Processing"
      }
    }

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>

  <tr>
    <td>TaskTrackId</td>
    <td>Operation unique identifier, can be used to query progress</td>
  </tr>
  <tr>
    <td>TaskStatus</td>
    <td>Task status.
        <ul>
         <li>NotSet: State not set</li>
         <li>Processing: Currently processing</li>
         <li>Succeeded: Succeeded</li>
         <li>Failed: Failed</li>
        </ul>
    </td>
  </tr>
</table>

**JSON example for request failed**:

    {
      "Succeeded": false,
      "ErrorInfo": {
        "Type": "MissingAuthorizationHeader",
        "Message": "Missing authorization header."
      }
    }

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>

  <tr>
    <td>Type</td>
    <td>Error type 
         <ul>
            <li>CredentialInvalid: Invalid credentials</li>
            <li>ParameterMissing: Parameter missing</li>
            <li>ParameterInvalid: Invalid Parameter</li>
            <li>MissingAuthorizationHeader: Authorization header missing</li>
            <li>InvalidRequestDateHeader: Invalid request date header</li>
            <li>MissingRequestDateHeader: Missing request date header</li>
            <li>AuthorizationHeaderExpired: Authorization header expired</li>
            <li>InvalidAuthorizationHeader: Invalid authorization header</li>
            <li>ApiKeyNotFound: API key not found</li>
            <li>InvalidApiKey: Invalid API key</li>
            <li>WrongSignature: Wrong signature</li>
            <li>SubscriptionNotFound: Subscription does not exist</li>
            <li>EndpointDoesNotBelongToSubscription: Endpoint does not belong to subscription</li>
            <li>EndpointNotInActiveState: Endpoint not in active state</li>
            <li>EndpointNotFound: Endpoint does not exist</li>
            <li>MaliciousItemPathDetected: Malicious item path detected</li>
            <li>PermissionDenied: Insufficient permissions</li>
            <li>RequestThrottled: Request throttled</li>
         </ul>    
    </td>
  </tr>
  <tr>
    <td>Message</td>
    <td>Error information</td>
  </tr>
</table>

<!--HONumber=May17_HO3-->