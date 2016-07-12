<properties 
   pageTitle="事件中心身份验证和安全模型概述 | Azure"
   description="事件中心常见问题"
   services="event-hubs"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="" />
<tags 
   ms.service="event-hubs"
   ms.date="01/26/2016"
   wacn.date="04/01/2016" />

# 事件中心身份验证和安全模型概述

事件中心安全模型满足以下要求：

- 只有可提供有效凭据的设备才能将数据发送到事件中心。
- 一台设备不能模拟另一台设备。
- 可以阻止恶意设备向事件中心发送数据。

## 设备身份验证

事件中心安全模型基于[共享访问签名 (SAS)](/documentation/articles/service-bus-shared-access-signature-authentication/) 令牌与发布者的组合。事件发布者定义事件中心的虚拟终结点。发布者只能用于将消息发送到事件中心。无法从发布者接收消息。

通常，事件中心为每个设备使用一个发布者。发送到事件中心的任何发布者的所有消息都将在该事件中心内排队。发布者允许进行精细的访问控制和限制。

为每个设备分配一个唯一令牌，该令牌将上载到该设备。生成令牌后，每个唯一令牌将授予对不同唯一发布者的访问权限。拥有令牌的设备只能向一个发布者发送消息，但不能向其他发布者发送消息。如果多个设备共享同一令牌，则其中每个设备将共享一个发布者。

可为设备配置令牌用于授予对事件中心的直接访问权限，但不建议这样做。持有此令牌的任何设备都可以直接将消息发送到该事件中心内。此类设备将不会受到限制。此外，无法将设备列入方块列表，使其无法该事件中心发送消息。

所有令牌使用 SAS 密钥进行签名。通常，所有令牌使用同一密钥进行签名。设备不知道该密钥；这可以防止设备生成令牌。

### 创建 SAS 密钥

创建命名空间时，服务总线将生成名为 **RootManageSharedAccessKey** 的 256 位 SAS 密钥。此密钥授予对命名空间的发送、侦听和管理权限。你可以创建其他密钥。建议你生成一个密钥用于授予对特定事件中心的发送权限。建议你生成一个密钥用于授予对特定事件中心的发送权限。本主题的余下内容假设你已将此密钥命名为 `EventHubSendKey`。

在创建事件中心时，以下示例将创建一个仅限发送的密钥：

```
// Create namespace manager.
string serviceNamespace = "YOUR_NAMESPACE";
string namespaceManageKeyName = "RootManageSharedAccessKey";
string namespaceManageKey = "YOUR_ROOT_MANAGE_SHARED_ACCESS_KEY";
Uri uri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, string.Empty);
TokenProvider td = TokenProvider.CreateSharedAccessSignatureTokenProvider(namespaceManageKeyName, namespaceManageKey);
NamespaceManager nm = new NamespaceManager(namespaceUri, namespaceManageTokenProvider);

// Create Event hub with a SAS rule that allows sending to that Event hub
EventHubDescription ed = new EventHubDescription("MY_EVENT_HUB") { PartitionCount = 32 };
string eventHubSendKeyName = "EventHubSendKey";
string eventHubSendKey = SharedAccessAuthorizationRule.GenerateRandomKey();
SharedAccessAuthorizationRule eventHubSendRule = new SharedAccessAuthorizationRule(eventHubSendKeyName, eventHubSendKey, new[] { AccessRights.Send });
ed.Authorization.Add(eventHubSendRule); 
nm.CreateEventHub(ed);
```

### 生成令牌

可以使用 SAS 密钥生成令牌。对于每个设备，只能生成一个令牌。然后，可以使用以下方法生成令牌。所有令牌都使用 **EventHubSendKey** 密钥生成。将为每个令牌分配一个唯一 URI。

```
public static string SharedAccessSignatureTokenProvider.GetSharedAccessSignature(string keyName, string sharedAccessKey, string resource, TimeSpan tokenTimeToLive)
```

调用此方法时，应将 URI 指定为 `//<NAMESPACE>.servicebus.chinacloudapi.cn/<EVENT_HUB_NAME>/publishers/<PUBLISHER_NAME>`。所有令牌的 URI 都是相同的，但每个令牌的 `PUBLISHER_NAME` 应该不同。`PUBLISHER_NAME` 最好是表示要接收该令牌的设备的 ID。

此方法将生成具有以下结构的令牌：

```
SharedAccessSignature sr={URI}&sig={HMAC_SHA256_SIGNATURE}&se={EXPIRATION_TIME}&skn={KEY_NAME}
```

令牌过期时间以从 1970 年 1 月 1 日开始算起的秒数指定。下面是一个令牌示例：

```
SharedAccessSignature sr=contoso&sig=nPzdNN%2Gli0ifrfJwaK4mkK0RqAB%2byJUlt%2bGFmBHG77A%3d&se=1403130337&skn=RootManageSharedAccessKey
```

通常，令牌的使用期限相当于或长于设备的使用期限。如果设备能够获取新令牌，可以使用使用期限较短的令牌。

### 发送数据的设备

创建令牌后，将为每个设备设置其自身唯一的令牌。

当设备向事件中心发送数据时，设备会在发送请求中标记其令牌。为了防止攻击者窃听和盗取令牌，设备与事件中心之间的通信必须通过加密通道进行。

### 将设备列入方块列表

如果盗取了令牌，攻击者可以模拟令牌已失窃的设备。将设备列入方块列表会导致它不可用，直到为该设备提供了使用不同发布者的新令牌。

## 后端应用程序的身份验证

为了对使用设备生成的数据的后端应用程序进行身份验证，事件中心采用了与服务总线主题所用模型类似的安全模型。事件中心使用者组等效于服务总线主题的订阅。如果创建使用者组的请求附带了一个用于授予对事件中心或者对事件中心所属命名空间的管理权限的令牌，则客户端可以创建该使用者组。如果接收请求附带了一个用于授予对使用者组、事件中心或者事件中心所属命名空间的接收权限的令牌，则允许客户端使用该使用者组的数据。

当前版本的服务总线不支持针对单个订阅的 SAS 规则。对于事件中心使用者组也是如此。将来会添加对这两项功能的 SAS 支持。

在无法为单个使用者组进行 SAS 身份验证的情况下，使用 SAS 密钥来保护具有一个公用密钥的所有使用者组。此方法能使应用程序使用事件中心的所有使用者组的数据。

### 在 ACS 中创建服务标识、信赖方和规则

ACS 支持通过多种方法创建服务标识、信赖方和规则，但最简单的方法是使用 [SBAZTool](http://code.msdn.microsoft.com/Authorization-SBAzTool-6fd76d93)。例如：

1. 为 **EventHubSender** 创建服务标识。这将返回已创建的服务标识的名称及其密钥：

	```
	sbaztool.exe exe -n <namespace> -k <key>  makeid eventhubsender
	```

2. 授予 **EventHubSender** 对事件中心的“发送声明”权限。

	```
	sbaztool.exe -n <namespace> -k <key> grant Send /AuthTestEventHub eventhubsender
	```

3. 为使用者组 1 的接收者创建服务标识：

	```
	sbaztool.exe exe -n <namespace> -k <key> makeid consumergroup1receiver
	```

4. 授予 `consumergroup1receiver` 对 **ConsumerGroup1** 的“侦听声明”权限：

	```
	sbaztool.exe -n <namespace> -k <key> grant Listen /AuthTestEventHub/ConsumerGroup1 consumergroup1receiver
	```

5. 为**使用者组 2** 的接收者创建服务标识：

	```
	sbaztool.exe exe -n <namespace> -k <key>  makeid consumergroup2receiver
	```

6. 授予 `consumergroup2receiver` 对 **ConsumerGroup2** 的“侦听声明”权限：

	```
	sbaztool.exe -n <namespace> -k <key> grant Listen /AuthTestEventHub/ConsumerGroup2 consumergroup2receiver
	```

## 后续步骤

若要了解有关事件中心的详细信息，请访问以下主题：

- [事件中心概述]。
- [使用事件中心的完整示例应用程序]。
- 使用服务总线队列的[队列消息解决方案]。

[事件中心概述]: /documentation/articles/event-hubs-overview/
[使用事件中心的完整示例应用程序]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[队列消息解决方案]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/
 

<!---HONumber=Mooncake_0321_2016-->