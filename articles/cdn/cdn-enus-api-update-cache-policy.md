<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN API doc-update cache policy"
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
    ms.service="cdn"
    ms.author="v-jijes"
    ms.topic="article"
    ms.date="5/4/2017"
    wacn.date="5/4/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-api-update-cache-policy/)
- [English](/documentation/articles/cdn-enus-api-update-cache-policy/)

# <a name="-"></a>Node management – set cache rules


You can use the API to configure cache rules for directories, filenames and full paths.

* Directory-based configuration: Directories must begin with “/”, for example, /pic, /doc, or /htdoc/data. The back end matches all files within the designated directory, including subdirectories*.
* File extension-based configuration: Common file extensions, such as .jpg, .png, .gif, .txt, .m4v, or .mp3. The back-end will match the specified file suffixes within all folders.
* Full path-based configuration: Used to specify a single file, must start with a forward slash (/). For example, "/sites/doc/example.doc". Note: If the path entered by the user is “/”, it will match the homepage.

>Note: that the character strings entered by the user when configuring rules must not include any of the following special characters: { } ( ) [ ] . ? * \ ^ or $.

>Note: **A time value of 0 indicates that caching is disabled, and the no-cache rules are prioritized.** 

## <a name=""></a>Request
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Method</strong>
    </td>
  <th align="left"><strong>Request URI</strong>
    </td>  
  <tr>
    <td>PUT</td>
    <td>https://api-preview.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}/cacherules?apiVersion=1.0</td>
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

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>
  <tr>
    <td>TTL</td>
    <td>Cache time in seconds</td>
  </tr>
  <tr>
    <td>IgnoreCacheControl</td>
    <td>Whether to ignore the cache-control header in the returned header and cache the request content
    </td>
  </tr>
  <tr>
    <td>IgnoreCookie</td>
    <td>Whether to ignore the set-cookie header in the returned header and cache the request content
    </td>
  </tr>
  <tr>
    <td>IgnoreQueryString</td>
    <td>Whether to ignore the query parameter and cache the request content
    </td>
  </tr>
  <tr>
    <td>Type</td>
    <td>Cache rule type.
        <ul>
         <li>Suffix: Cache based on the file extension</li>
         <li>Dir: Cache all files in the specified directory</li>
         <li>FullUri: Cache files at a specific path</li>
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
**JSON example for request succeeded**:

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