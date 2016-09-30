<!-- not suitable for Mooncake -->

<properties 
	pageTitle="应用服务 API 应用触发器 | Azure" 
	description="如何在 Azure 应用服务的 API 应用中实现触发器" 
	services="logic-apps" 
	documentationCenter=".net" 
	authors="guangyang"
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="logic-apps"
	ms.date="03/16/2016"
	wacn.date=""/>

# Azure 应用服务 API 应用触发器

>[AZURE.NOTE] 此版文章适用于 API 应用 2014-12-01-preview 架构版本。


## 概述

本文介绍如何实现 API 应用触发器，并从逻辑应用使用这些触发器。

本主题中的所有代码片段复制自 [FileWatcher API 应用代码示例](http://go.microsoft.com/fwlink/?LinkId=534802)。

请注意，需要下载以下 nuget 包才能生成和运行本文所述的代码：[http://www.nuget.org/packages/Microsoft.Azure.AppService.ApiApps.Service/](http://www.nuget.org/packages/Microsoft.Azure.AppService.ApiApps.Service/)。

## 什么是 API 应用触发器？

API 应用经常需要激发事件，使 API 应用客户端采取适当的措施来响应事件。支持此方案的基于 REST API 的机制称为 API 应用触发器。

例如，假设客户端代码使用 [Twitter 连接器 API 应用](/documentation/articles/app-service-logic-connector-twitter/)，并且代码需要根据包含特定文本的新推文执行操作。在此情况下，可以设置轮询或推送触发器来方便解决此需求。

## 轮询触发器与推送触发器

目前支持两种类型的触发器：

- 轮询触发器 - 客户端在 API 应用中轮询已激发事件的通知
- 推送触发器 - 激发事件时，API 应用通知客户端

### 轮询触发器

轮询触发器实现为常规 REST API，预期客户端（例如逻辑应用）轮询会它来获取通知。客户端可以保留状态，而轮询触发器本身是无状态的。

下面有关请求和响应数据包的信息演示了轮询触发器合约的一些重要层面：

- 请求
    - HTTP 方法：GET
    - 参数
        - triggerState - 此可选参数可让客户端指定其状态，以便轮询触发器可以根据指定的状态正确确定是否要返回通知。
        - 特定于 API 的参数
- 响应
    - 状态代码 **200** - 请求有效，触发器未发送通知。通知内容为响应正文。响应中的“Retry-After”标头指示，必须通过后续请求调用检索其他通知数据。
    - 状态代码 **202** - 请求有效，但触发器未发送新通知。
    - 状态代码 **4xx** - 请求无效。客户端不应重试请求。
    - 状态代码 **5xx** - 请求导致内部服务器错误和/或暂时性问题。客户端应重试请求。

以下代码片段示范了如何实现轮询触发器。

    // Implement a poll trigger.
    [HttpGet]
    [Route("api/files/poll/TouchedFiles")]
    public HttpResponseMessage TouchedFilesPollTrigger(
        // triggerState is a UTC timestamp
        string triggerState,
        // Additional parameters
        string searchPattern = "*")
    {
        // Check to see whether there is any file touched after the timestamp.
        var lastTriggerTimeUtc = DateTime.Parse(triggerState).ToUniversalTime();
        var touchedFiles = Directory.EnumerateFiles(rootPath, searchPattern, SearchOption.AllDirectories)
            .Select(f => FileInfoWrapper.FromFileInfo(new FileInfo(f)))
            .Where(fi => fi.LastAccessTimeUtc > lastTriggerTimeUtc);

        // If there are files touched after the timestamp, return their information.
        if (touchedFiles != null && touchedFiles.Count() != 0)
        {
            // Extension method provided by the AppService service SDK.
            return this.Request.EventTriggered(new { files = touchedFiles });
        }
        // If there are no files touched after the timestamp, tell the caller to poll again after 1 mintue.
        else
        {
            // Extension method provided by the AppService service SDK.
            return this.Request.EventWaitPoll(new TimeSpan(0, 1, 0));
        }
    }

若要测试此轮询触发器，请遵循以下步骤：

1. 部署身份验证设置为 **public anonymous** 的 API 应用。
2. 调用 **touch** 操作来修改文件。下图显示通过 Postman 的示例请求。![通过 Postman 调用 Touch 操作](./media/app-service-api-dotnet-triggers/calltouchfilefrompostman.PNG)
3. 结合步骤 2 之前设置为时间戳的 **triggerState** 参数调用轮询触发器。下图显示通过 Postman 的示例请求。![通过 Postman 调用轮询触发器](./media/app-service-api-dotnet-triggers/callpolltriggerfrompostman.PNG)

### 推送触发器

推送触发器实现为常规 REST API，将通知推送到已注册为希望在激发特定事件时收到通知的客户端。

下面有关请求和响应数据包的信息演示了推送触发器合约的一些重要层面。

- 请求
    - HTTP 方法：PUT
    - Parameters
        - triggerId：必需 - 不透明的字符串（例如 GUID），表示推送触发器的注册。
        - callbackUrl：必需 - 激发事件时要调用的回调的 URL。该调用是一个简单的 POST HTTP 调用。
        - 特定于 API 的参数
- 响应
    - 状态代码 **200** - 注册客户端的请求成功。
    - 状态代码 **4xx** - 请求无效。客户端不应重试请求。
    - 状态代码 **5xx** - 请求导致内部服务器错误和/或暂时性问题。客户端应重试请求。
- 回调
    - HTTP 方法：POST
    - 请求正文：通知内容。

以下代码片段示范了如何实现推送触发器：

    // Implement a push trigger.
    [HttpPut]
    [Route("api/files/push/TouchedFiles/{triggerId}")]
    public HttpResponseMessage TouchedFilesPushTrigger(
        // triggerId is an opaque string.
        string triggerId,
        // A helper class provided by the AppService service SDK.
        // Here it defines the input of the push trigger is a string and the output to the callback is a FileInfoWrapper object.
        [FromBody]TriggerInput<string, FileInfoWrapper> triggerInput)
    {
        // Register the trigger to some trigger store.
        triggerStore.RegisterTrigger(triggerId, rootPath, triggerInput);

        // Extension method provided by the AppService service SDK indicating the registration is completed.
        return this.Request.PushTriggerRegistered(triggerInput.GetCallback());
    }

    // A simple in-memory trigger store.
    public class InMemoryTriggerStore
    {
        private static InMemoryTriggerStore instance;

        private IDictionary<string, FileSystemWatcher> _store;

        private InMemoryTriggerStore()
        {
            _store = new Dictionary<string, FileSystemWatcher>();
        }

        public static InMemoryTriggerStore Instance
        {
            get
            {
                if (instance == null)
                {
                    instance = new InMemoryTriggerStore();
                }
                return instance;
            }
        }

        // Register a push trigger.
        public void RegisterTrigger(string triggerId, string rootPath,
            TriggerInput<string, FileInfoWrapper> triggerInput)
        {
            // Use FileSystemWatcher to listen to file change event.
            var filter = string.IsNullOrEmpty(triggerInput.inputs) ? "*" : triggerInput.inputs;
            var watcher = new FileSystemWatcher(rootPath, filter);
            watcher.IncludeSubdirectories = true;
            watcher.EnableRaisingEvents = true;
            watcher.NotifyFilter = NotifyFilters.LastAccess;

            // When some file is changed, fire the push trigger.
            watcher.Changed +=
                (sender, e) => watcher_Changed(sender, e,
                    Runtime.FromAppSettings(),
                    triggerInput.GetCallback());

            // Assoicate the FileSystemWatcher object with the triggerId.
            _store[triggerId] = watcher;

        }

        // Fire the assoicated push trigger when some file is changed.
        void watcher_Changed(object sender, FileSystemEventArgs e,
            // AppService runtime object needed to invoke the callback.
            Runtime runtime,
            // The callback to invoke.
            ClientTriggerCallback<FileInfoWrapper> callback)
        {
            // Helper method provided by AppService service SDK to invoke a push trigger callback.
            callback.InvokeAsync(runtime, FileInfoWrapper.FromFileInfo(new FileInfo(e.FullPath)));
        }
    }

若要测试此轮询触发器，请遵循以下步骤：

1. 部署身份验证设置为 **public anonymous** 的 API 应用。
2. 浏览到 [http://requestb.in/](http://requestb.in/)，创建 RequestBin 作为回调 URL。
3. 结合 GUID **triggerId** 和 RequestBin URL **callbackUrl** 来调用推送触发器。
   ![通过 Postman 调用推送触发器](./media/app-service-api-dotnet-triggers/callpushtriggerfrompostman.PNG)
4. 调用 **touch** 操作来修改文件。下图显示通过 Postman 的示例请求。
   ![通过 Postman 调用 Touch 操作](./media/app-service-api-dotnet-triggers/calltouchfilefrompostman.PNG)
5. 检查 RequestBin，确认是否使用属性输出调用了推送触发器回调。
   ![通过 Postman 调用轮询触发器](./media/app-service-api-dotnet-triggers/pushtriggercallbackinrequestbin.PNG)

### 在 API 定义中描述触发器

实现触发器并将 API 应用部署到 Azure 之后，请导航到 Azure 门户预览版中的“API 定义”边栏选项卡，然后查看 UI 中是否已自动识别触发器（这种识别由 API 应用的 Swagger 2.0 API 定义驱动）。

![API 定义边栏选项卡](./media/app-service-api-dotnet-triggers/apidefinitionblade.PNG)

如果单击“下载 Swagger”按钮并打开 JSON 文件，将看到类似于下面的结果：

    "/api/files/poll/TouchedFiles": {
      "get": {
        "operationId": "Files_TouchedFilesPollTrigger",
        ...
        "x-ms-scheduler-trigger": "poll"
      }
    },
    "/api/files/push/TouchedFiles/{triggerId}": {
      "put": {
        "operationId": "Files_TouchedFilesPushTrigger",
        ...
        "x-ms-scheduler-trigger": "push"
      }
    }

扩展属性 **x-ms-schedular-trigger** 是 API 定义中描述触发器的方式，通过网关请求 API 定义时，如果请求符合以下条件之一，则 API 应用网关会自动添加该属性。（也可以手动添加此属性。）

- 轮询触发器
    - 如果 HTTP 方法为 **GET**。
    - 如果 **operationId** 属性包含字符串 **trigger**。
    - 如果 **parameters** 属性包含参数的 **name** 属性设置为 **triggerState**。
- 推送触发器
    - 如果 HTTP 方法为 **PUT**。
    - 如果 **operationId** 属性包含字符串 **trigger**。
    - 如果 **parameters** 属性包含参数的 **name** 属性设置为 **triggerId**。

## 在逻辑应用中使用 API 应用触发器

### 在逻辑应用设计器中列出和配置 API 应用触发器

如果在与 API 应用相同的资源组中创建逻辑应用，则只要单击该逻辑应用即可将它添加到设计器画布。下图演示了此操作：

![逻辑应用设计器中的触发器](./media/app-service-api-dotnet-triggers/triggersinlogicappdesigner.PNG)

![逻辑应用设计器中的“配置轮询触发器”](./media/app-service-api-dotnet-triggers/configurepolltriggerinlogicappdesigner.PNG)

![逻辑应用设计器中的“配置推送触发器”](./media/app-service-api-dotnet-triggers/configurepushtriggerinlogicappdesigner.PNG)

## 为逻辑应用优化 API 应用触发器

将触发器添加到 API 应用后，可通过多种方式来改善在逻辑应用中使用 API 应用的体验。

例如，轮询触发器的 **triggerState** 参数应该在逻辑应用中设置为以下表达式。此表达式应该评估逻辑应用中触发器的最后一个调用，并返回该值。

	@coalesce(triggers()?.outputs?.body?['triggerState'], '')

注意：有关上述表达式中使用的函数的说明，请参阅 [Logic App Workflow Definition Language](https://msdn.microsoft.com/zh-cn/library/azure/dn948512.aspx)（逻辑应用工作流定义语言）文档。

使用触发器时，逻辑应用用户需要为 **triggerState** 参数提供上述表达式。逻辑应用设计器可通过扩展属性 **x-ms-scheduler-recommendation** 预先设置此值。可将 **x-ms-visibility** 扩展属性的值设置为 *internal*，使参数本身不会显示在设计器中。以下代码片段对此做了演示。

    "/api/Messages/poll": {
      "get": {
	    "operationId": "Messages_NewMessageTrigger",
        "parameters": [
          {
            "name": "triggerState",
            "in": "query",
            "required": true,
            "x-ms-visibility": "internal",
            "x-ms-scheduler-recommendation": "@coalesce(triggers()?.outputs?.body?['triggerState'], '')",
            "type": "string"
          }
        ]
        ...
        "x-ms-scheduler-trigger": "poll"
      }
    }

对于推送触发器，**triggerId** 参数必须唯一标识逻辑应用。建议的最佳实践是使用以下表达式将此属性设置为工作流的名称：

    @workflow().name

在 API 定义中使用 **x-ms-scheduler-recommendation** 和 **x-ms-visibility** 扩展属性，API 应用可与逻辑应用设计器沟通，自动为用户设置此表达式。

        "parameters":[  
          {  
            "name":"triggerId",
            "in":"path",
            "required":true,
            "x-ms-visibility":"internal",
            "x-ms-scheduler-recommendation":"@workflow().name",
            "type":"string"
          },


### 在 API 定义中添加扩展属性

可以通过以下两种方式在 API 定义中添加其他元数据信息（例如扩展属性 **x-ms-scheduler-recommendation** 和 **x-ms-visibility**）：静态或动态。

对于静态元数据，可以直接编辑项目中的 */metadata/apiDefinition.swagger.json* 文件，并手动添加属性。

对于使用动态元数据的 API 应用，可以编辑 SwaggerConfig.cs 文件来添加操作筛选器用于添加这些扩展。

    GlobalConfiguration.Configuration 
        .EnableSwagger(c =>
            {
                ...
                c.OperationFilter<TriggerStateFilter>();
                ...
            }


下面是如何实现此类以方便处理动态元数据方案的示例。

    // Add extension properties on the triggerState parameter
    public class TriggerStateFilter : IOperationFilter
    {

        public void Apply(Operation operation, SchemaRegistry schemaRegistry, System.Web.Http.Description.ApiDescription apiDescription)
        {
            if (operation.operationId.IndexOf("Trigger", StringComparison.InvariantCultureIgnoreCase) >= 0)
            {
                // this is a possible trigger
                var triggerStateParam = operation.parameters.FirstOrDefault(x => x.name.Equals("triggerState"));
                if (triggerStateParam != null)
                {
                    if (triggerStateParam.vendorExtensions == null)
                    {
                        triggerStateParam.vendorExtensions = new Dictionary<string, object>();
                    }

                    // add 2 vendor extensions
                    // x-ms-visibility: set to 'internal' to signify this is an internal field
                    // x-ms-scheduler-recommendation: set to a value that logic app can use
                    triggerStateParam.vendorExtensions.Add("x-ms-visibility", "internal");
                    triggerStateParam.vendorExtensions.Add("x-ms-scheduler-recommendation",
                                                           "@coalesce(triggers()?.outputs?.body?['triggerState'], '')");
                }
            }
        }
    }
 

<!---HONumber=Mooncake_0919_2016-->