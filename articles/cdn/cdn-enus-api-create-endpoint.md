<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN API doc- create endpoint"
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
- [中文](/documentation/articles/cdn-api-create-endpoint/)
- [English](/documentation/articles/cdn-enus-api-create-endpoint/)

# <a name="-"></a>Node management – create node


You can create a CDN node using the API.

>Note: Configurations created for endpoints will not be immediately available:

>The custom domain name and ICP number that you entered must first be reviewed to ensure that they match and are valid. This process can take up to one business day to complete. If the details do not pass the ICP review, you must delete the Content Delivery Network endpoint you created and create a new endpoint by using the correct custom domain name and ICP number. If the details pass the Internet Content Provider (ICP) review, the Content Delivery Network service will be registered within 60 minutes, so that it can be propagated by the network. At the same time, you also need to configure the CNAME mapping details, as indicated by the notifications in the interface, before the cache content can finally be accessed via the custom domain name.

## <a name=""></a>Request
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Method</strong>
    </td>
  <th align="left"><strong>Request URI</strong>
    </td>  
  <tr>
    <td>POST</td>
    <td>https://api-preview.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints?apiVersion=1.0</td>
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
  </tr>
</table>

### <a name="-headers"></a>Request header

| Request header | Description |
|:-----------|:-----------|
| x-azurecdn-request-date | Must be entered. Current UTC request time in yyyy-MM-dd hh:mm:ss format |
| Authorization | Must be entered. Please refer to [CDN API signing mechanism](/documentation/articles/cdn-enus-api-signature/) for authorization headers. |
| content-type | Must be entered. application/json |

### <a name="-body"></a>Request body
To create a CDN node, you must specify the following parameters. An example JSON file is provided below:

    {
      "CustomDomain": "www.example.com",
      "Host": "www.example.com",
      "ICP": "ICP123456",
      "Origin": {
        "Addresses": [
          "www.origin.com"
        ]
      },
      "ServiceType": "Web"
    }

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>Parameter name</strong>
  </th>
  <th align="left"><strong>Description</strong>
  </th>
  <tr>
    <td>CustomDomain</td>
    <td>Accelerated domain names</td>
  </tr>
  <tr>
    <td>Host</td>
    <td>Return-to-source host header</td>
  </tr>
  <tr>
    <td>ICP</td>
    <td>ICP record number</td>
  </tr>
  <tr>
    <td>Addresses</td>
    <td>Return-to-source address collection</td>
  </tr>
  <tr>
    <td>ServiceType</td>
    <td>Acceleration type.
      <ul>
         <li>Web: Web page acceleration</li>
         <li>Download: Download acceleration</li>
         <li>VOD: On-demand acceleration</li>
         <li>LiveStreaming: Live streaming acceleration</li>
         <li>ImageProcessing: Image processing acceleration</li>
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
**An example JSON file for a request that has been successfully accepted is provided below**:

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
        "Enabled": "false",
        "IcpVerifyStatus": "IcpVerifying",
        "LifetimeStatus": "Creating",
        "CNameConfigured": "false",
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
    <td>Node unique identifier</td>
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
    <td>Node status. <ul>
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