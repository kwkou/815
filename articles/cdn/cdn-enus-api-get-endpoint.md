<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN API doc-get endpoint"
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
- [中文](/documentation/articles/cdn-api-get-endpoint/)
- [English](/documentation/articles/cdn-enus-api-get-endpoint/)

# <a name="-"></a>CDN endpoint Management – Get information on CDN endpoints


## <a name=""></a>Request
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Method</strong>
    </td>
  <th align="left"><strong>Request URI</strong>
    </td>  
  <tr>
    <td>GET</td>
    <td>https://restapi.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints/{endpointId}?apiVersion=1.0</td>
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
| Authorization | Must be entered. Please refer to [CDN API signature mechanism](/documentation/articles/cdn-enus-api-signature/) for authorization headers. |

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
      "EndpointID": "779bff4d-ef38-4fce-82d8-6b50cc4c183b",
      "Setting": {
        "CustomDomain": "www.example.com",
        "Host": "string",
        "ICP": "ICP123456",
        "Origin": {
          "Addresses": [
            "www.origin.com"
          ]
        },
        "ServiceType": "Web"
      },
      "Status": {
        "Enabled": "true",
        "IcpVerifyStatus": "IcpVerified",
        "LifetimeStatus": "Normal",
        "CNameConfigured": "true",
        "FreeTrialExpired": "false",
        "TimeLastUpdated": "2017-04-28T07:34:54.849Z"
      }
    }

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
    </td>
  <th align="left"><strong>Description</strong>
    </td>

  <tr>
    <td>EndpointID</td>
    <td>CDN endpoint unique identifier</td>
  </tr>
  <tr>
    <td>Enabled</td>
    <td>Task status. 
     <ul>
          <li>NotSet: State not set</li>
          <li>Processing: Currently processing</li>
          <li>Succeeded: Succeeded</li>
          <li>Failed: Failed</li>
        </ul>
  </tr>
  <tr>
    <td>IcpVerifyStatus</td>
    <td>ICP record verification information. <ul>
         <li>IcpVerifying: Currently being verified</li>
         <li>IcpVerifyFailed: Verification has failed</li>
         <li>IcpVerified: Verification has succeeded</li>
        </ul>
    </td>
  </tr>
  <tr>
    <td>LifetimeStatus</td>
    <td>CDN endpoint status.
     <ul>
         <li>Normal: Normal</li>
         <li>Creating: Creating</li>
         <li>CreationFailed: Creation failed</li>
         <li>Deleting: Deleting</li>
         <li>Deleted: Deleted</li>
         <li>Updating: Updating</li>
         <li>Enabling: Activating</li>
         <li>Disabling: Disabling</li>
    </td>
  </tr>
  <tr>
    <td>CNameConfigured</td>
    <td>Whether the accelerated domain name CNAME record is already configured</td>
  </tr>
  <tr>
    <td>FreeTrialExpired</td>
    <td>Whether the trial period has expired</td>
  </tr>
  <tr>
    <td>TimeLastUpdated</td>
    <td>Time of last update</td>
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