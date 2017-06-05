<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN API doc-get subscription usage"
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
- [中文](/documentation/articles/cdn-api-get-subscription-volume/)
- [English](/documentation/articles/cdn-enus-api-get-subscription-volume/)

# <a name="-"></a>Traffic – Get traffic information for all nodes under the subscription


## <a name=""></a>Request
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Method</strong>
    </td>
  <th align="left"><strong>Request URI</strong>
    </td>  
  <tr>
    <td>GET</td>
    <td>https://api-preview.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}/volume?apiVersion=1.0</td>
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
  <tr>
    <td>startTime</td>
    <td>Data query start time; must be a UTC time in “yyyy-MM-ddThh:mm:ssZ” format 
  </td>
  </tr>
  <tr>
    <td>endTime</td>
    <td>Data query end time; must be a UTC time in “yyyy-MM-ddThh:mm:ssZ” format
    </td>
  </tr>
  <tr>
    <td>granularity</td>
    <td>Traffic statistic granularity.
    <ul>
       <li>PerFiveMinutes: Per five minutes</li>
       <li>PerHour: Per hour</li>
       <li>PerDay: Per day</li>
    </ul></td>
  </tr>
</table>

### <a name="-headers"></a>Request header

| Request header | Description |
|:-----------|:-----------|
| x-azurecdn-request-date | Must be entered. Current UTC request time in yyyy-MM-dd hh:mm:ss format |
| Authorization | Must be entered. Please refer to [CDN API signing mechanism](/documentation/articles/cdn-enus-api-signature/) for authorization headers. |

### <a name="-body"></a>Request body
None

## <a name=""></a>Response

A response comprises a status code, response headers, and a response body.
### <a name=""></a>Status code
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Status code</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>
  <tr>
    <td>200</td>
    <td>Indicates that the server has returned successfully</td>
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
      "Details": {
        "www.example1.com": {
          "DomainName": "www.example1.com",
          "Items": [
            {
              "Timestamp": "2017-05-01T16:00:00Z",
              "VolumeInMB": 100,
              "OriginVolumeInMB": 20
            },
            {
              "Timestamp": "2017-05-02T16:00:00Z",
              "VolumeInMB": 200,
              "OriginVolumeInMB": 40
            }
          ],
          "TotalCdnVolumeInMB": 300,
          "TotalOriginVolumeInMB": 60
        },
        "www.example2.com": {
          "DomainName": "www.example2.com",
          "Items": [
            {
              "Timestamp": "2017-05-01T16:00:00Z",
              "VolumeInMB": 100,
              "OriginVolumeInMB": 20
            },
            {
              "Timestamp": "2017-05-02T16:00:00Z",
              "VolumeInMB": 200,
              "OriginVolumeInMB": 40
            }
          ],
          "TotalCdnVolumeInMB": 300,
          "TotalOriginVolumeInMB": 60
        },
      "TotalCdnVolumeInMB": 600,
      "TotalOriginVolumeInMB": 120
    }

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>

  <tr>
    <td>DomainName</td>
    <td>Accelerated domain names</td>
  </tr>
  <tr>
    <td>VolumeInMB</td>
    <td>CDN traffic</td>
  </tr>
  <tr>
    <td>OriginVolumeInMB</td>
    <td>Back to source traffic</td>
  </tr>
  <tr>
    <td>TotalCdnVolumeInMB</td>
    <td>CDN total traffic</td>
  </tr>
  <tr>
    <td>TotalOriginVolumeInMB</td>
    <td>Back to source total traffic</td>
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