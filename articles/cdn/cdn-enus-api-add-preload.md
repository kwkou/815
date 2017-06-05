<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN API doc- preload"
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
    ms.date="5/4/2017"
    wacn.date="5/4/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-api-add-preload/)
- [English](/documentation/articles/cdn-enus-api-add-preload/)

# <a name="-"></a>Preload – add content prefetching


Preloading means caching the content of a designated URL from the source station to the network nodes, to eliminate the waiting time when a user accesses the resource for the first time. Preloading is generally used in scenarios that involve the delivery of large files, where it can effectively improve the user access experience.

You can configure content prefetching (preloading) for individual files or multiple files.

## <a name=""></a>Request
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Method</strong>
    </td>
  <th align="left"><strong>Request URI</strong>
    </td>  
  <tr>
    <td>POST</td>
    <td>https://api-preview.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}/preloads?apiVersion=1.0</td>
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
    <td>Target node unique identifier</td>
  </tr>
</table>

### <a name="-headers"></a>Request header

| Request header | Description |
|:-----------|:-----------|
| x-azurecdn-request-date | Must be entered. Current UTC request time in yyyy-MM-dd hh:mm:ss format |
| Authorization | Must be entered. Please refer to [CDN API signing mechanism](/documentation/articles/cdn-enus-api-signature/) for authorization headers. |
| content-type | Must be entered. application/json |

### <a name="-body"></a>Request body

To add content prefetching, you must specify the parameters. An example JSON file is provided below:

    {
      "Files": [
        "http://example.com/pictures/city.png"
      ]
    }

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
  </th>
  <th align="left"><strong>Description</strong>
  </th>
  <tr>
    <td>Files </td>
    <td>The paths for refreshing the file list must be absolute paths, e.g. http://example.com/pictures/city.png</td>
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
**An example JSON file for request succeeded is provided below**:

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
    <td>Prefetch operation unique identifier, can be used to query prefetch progress</td>
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