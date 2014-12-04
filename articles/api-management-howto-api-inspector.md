<properties pageTitle="如何使用 API 检查器跟踪 Azure API 管理中的调用" metaKeywords="" description="了解如何使用 API 检查器跟踪 Azure API 管理中的调用。" metaCanonical="" services="" documentationCenter="API Management" title="如何使用 API 检查器跟踪 Azure API 管理中的调用" authors="sdanie" solutions="" manager="" editor="" />

# 如何使用 API 检查器跟踪 Azure API 管理中的调用

API 管理（预览版）提供了一个 API 检查器工具，帮助您进行调试和故障诊断 API。API 检查器可以从您的应用程序以编程方式使用，还可以直接从开发人员门户中使用。本指南提供使用 API 检查器的演练。

## 本主题内容

-   [使用 API 检查器跟踪调用][使用 API 检查器跟踪调用]
-   [检查跟踪][检查跟踪]
-   [后续步骤][后续步骤]

## <a name="trace-call"> </a>使用 API 检查器跟踪调用

要使用 API 检查器，请添加 **ocp-apim-trace:true**请求标头到您的操作调用，然后使用 **ocp-apim-trace-location** 响应标头指定的 URL 下载并检查跟踪。这可以编程方式完成，也可以直接从开发人员门户完成。

本教程演示如何使用 API 检查器对跟踪使用 Echo API 的操作。

> 每个预先配置 Echo API 的 API 管理服务实例，都可用于试验和了解 API 管理。Echo API 返回任何发送给它的输入。要使用它，您可以调用任何 HTTP 谓词，并且返回的值将只是您发送的内容。

要开始，请单击您的 API 管理服务的 Azure 门户中的**开发门户**。这使您转到 API 管理管理门户。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![API 管理开发人员门户][API 管理开发人员门户]

可以直接从开发人员门户调用操作，这样可以方便地查看和测试 API 的操作。在此教程步骤中，将调用 **Echo API** 的**获取资源**方法。

单击顶部菜单的 **API**，然后单击 **Echo API**。

![Echo API][Echo API]

> 如果必须只有一个 API 得到配置或对您的帐户可见，然后单击 API 使您直接进入该 API 的操作。

选择 **GET 资源**操作，单击**打开控制台**。

![打开控制台][打开控制台]

保持默认参数值，然后从 **subscription-key** 下拉列表选择您的订阅密钥。

将 **ocp-apim-trace:true** 键入**请求标头**文本框，然后单击 **HTTP Get**。

![HTTP Get][HTTP Get]

响应标头将是 **ocp-apim-trace-location**，值类似于下面的示例。

    ocp-apim-trace-location : https://contosoltdxw7zagdfsprykd.blob.core.windows.net/apiinspectorcontainer/ZW3e23NsW4wQyS-SHjS0Og2-2?sv=2013-08-15&sr=b&sig=Mgx7cMHsLmVDv%2B%2BSzvg3JR8qGTHoOyIAV7xDsZbF7%2Bk%3D&se=2014-05-04T21%3A00%3A13Z&sp=r&verify_guid=a56a17d83de04fcb8b9766df38514742

可以从指定的位置下载和查看跟踪，如下一步中所示。

## <a name="inspect-trace"> </a>检查跟踪

要查看在跟踪中的值，请从 **ocp-apim-trace-location** URL 下载跟踪文件。它是采用 JSON 格式的文本文件，并包含类似下面示例的条目。

    {
      "validityGuid":"a43a07a03de04fcb8b1425df38514742",
      "logEntries":[
        {
            "timestamp":"2014-05-03T21:00:13.2182473Z",
            "source":"Microsoft.WindowsAzure.ApiManagement.Proxy.Gateway.Handlers.DebugLoggingHandler",
            "data":{
            "originalRequest":{
                "method":"GET",
                "url":"https://contosoltd.current.int-azure-api.net/echo/resource?param1=sample&subscription-key=...",
                "headers":[
                {
                    "name":"ocp-apim-tracing",
                    "value":"true"
                },
                {
                    "name":"Host",
                    "value":"contosoltd.current.int-azure-api.net"
                }
                ]
            }
            }
        },
        {
          "timestamp":"2014-05-03T21:00:13.2182473Z",
          "source":"request handler",
          "data":{
            "configuration":{
              "api":{
                "from":"echo",
                "to":"http://echoapi.cloudapp.net/api"
              },
              "operation":{
                "method":"GET",
                "uriTemplate":"/resource"
              },
              "user":{
                "id":1,
                "groups":[
              
                ]
              },
              "product":{
                "id":1
              }
            }
          }
        },
        {
          "timestamp":"2014-05-03T21:00:13.2182473Z",
          "source":"backend call handler",
          "data":{
            "message":"Sending request to the service.",
            "request":{
              "method":"GET",
              "url":"http://echoapi.cloudapp.net/api/resource?param1=sample&subscription-key=...",
              "headers":[
                {
                  "name":"X-Forwarded-For",
                  "value":"138.91.78.77"
                },
                {
                  "name":"ocp-apim-tracing",
                  "value":"true"
                }
              ]
            }
          }
        },
        {
          "timestamp":"2014-05-03T21:00:13.2182473Z",
          "source":"backend call handler",
          "data":{
            "status":{
              "code":200,
              "reason":"OK"
            },
            "headers":[
              {
                "name":"Pragma",
                "value":"no-cache"
              },
              {
                "name":"Host",
                "value":"echoapi.cloudapp.net"
              },
              {
                "name":"X-Forwarded-For",
                "value":"138.91.78.77"
              },
              {
                "name":"ocp-apim-tracing",
                "value":"true"
              },
              {
                "name":"Content-Length",
                "value":"0"
              },
              {
                "name":"Cache-Control",
                "value":"no-cache"
              },
              {
                "name":"Expires",
                "value":"-1"
              },
              {
                "name":"Server",
                "value":"Microsoft-IIS/8.0"
              },
              {
                "name":"X-AspNet-Version",
                "value":"4.0.30319"
              },
              {
                "name":"X-Powered-By",
                "value":"Azure API Management - http://api.azure.com/,ASP.NET"
              },
              {
                "name":"Date",
                "value":"Sat, 03 May 2014 21:00:17 GMT"
              }
            ]
          }
        }
      ]
    }

## <a name="next-steps"> </a>后续步骤

-   查看[开始使用高级 API 配置][开始使用高级 API 配置]教程中的其他主题。

  [使用 API 检查器跟踪调用]: #trace-call
  [检查跟踪]: #inspect-trace
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [API 管理开发人员门户]: ./media/api-management-howto-api-inspector/api-management-developer-portal-menu.png
  [Echo API]: ./media/api-management-howto-api-inspector/api-management-echo-api.png
  [打开控制台]: ./media/api-management-howto-api-inspector/api-management-open-console.png
  [HTTP Get]: ./media/api-management-howto-api-inspector/api-management-http-get.png
  [开始使用高级 API 配置]: ../api-management-get-started-advanced
