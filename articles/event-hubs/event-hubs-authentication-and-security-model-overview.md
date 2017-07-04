<properties 
    pageTitle="Azure 事件中心身份验证和安全模型概述 | Azure"
    description="事件中心身份验证和安全模型概述。"
    services="event-hubs"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="93841e30-0c5c-4719-9dc1-57a4814342e7"
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/29/2017"
    wacn.date="05/08/2017"
    ms.author="sethm;clemensv"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="675adc8e460ea3f63b1f0c7741f636ba17a30b5a"
    ms.lasthandoff="04/28/2017" />

# <a name="event-hubs-authentication-and-security-model-overview"></a>事件中心身份验证和安全模型概述

Azure 事件中心安全模型满足以下要求：

* 只有可提供有效凭据的客户端才能将数据发送到事件中心。
* 一个客户端无法模拟另一个客户端。
* 可以阻止恶意客户端向事件中心发送数据。

## <a name="client-authentication"></a>客户端身份验证

事件中心安全模型基于[共享访问签名 (SAS)](/documentation/articles/service-bus-sas/) 令牌与*事件发布者*的组合。 事件发布者定义事件中心的虚拟终结点。 发布者只能用于将消息发送到事件中心。 无法从发布者接收消息。

通常，事件中心为每个客户端使用一个发布者。 发送到事件中心的任何发布者的所有消息都将在该事件中心内排队。 发布者允许进行精细的访问控制和限制。

为每个事件中心客户端分配一个唯一令牌，该令牌将上传到客户端。 生成令牌后，每个唯一令牌将授予对不同唯一发布者的访问权限。 拥有令牌的客户端只能向一个发布者发送消息，但不能向其他发布者发送消息。 如果多个客户端共享同一令牌，则其中每个客户端都将共享一个发布者。

可为客户端和设备配置令牌用于授予对事件中心的直接访问权限，但不建议这样做。 持有此令牌的任何客户端都可以直接将消息发送到该事件中心内。 此类客户端不会受到限制。 此外，无法将客户端列入阻止列表，使其无法向该事件中心发送消息。

所有令牌都使用 SAS 密钥进行签名。 通常，所有令牌使用同一密钥进行签名。 客户端不知道密钥；这可以防止其他客户端生成令牌。

### <a name="create-the-sas-key"></a>创建 SAS 密钥
创建 Azure 事件中心命名空间时，该服务会生成名为 **RootManageSharedAccessKey**的 256 位 SAS 密钥。 此密钥授予对命名空间的发送、侦听和管理权限。 你可以创建其他密钥。 建议生成一个向特定事件中心授予发送权限的密钥。 本主题的余下部分假设已将此密钥命名为 **EventHubSendKey**。

创建事件中心时，以下示例将创建一个仅限发送的密钥：

    // Create namespace manager.
    string serviceNamespace = "YOUR_NAMESPACE";
    string namespaceManageKeyName = "RootManageSharedAccessKey";
    string namespaceManageKey = "YOUR_ROOT_MANAGE_SHARED_ACCESS_KEY";
    Uri uri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, string.Empty);
    TokenProvider td = TokenProvider.CreateSharedAccessSignatureTokenProvider(namespaceManageKeyName, namespaceManageKey);
    NamespaceManager nm = new NamespaceManager(namespaceUri, namespaceManageTokenProvider);

    // Create Event Hub with a SAS rule that enables sending to that Event Hub
    EventHubDescription ed = new EventHubDescription("MY_EVENT_HUB") { PartitionCount = 32 };
    string eventHubSendKeyName = "EventHubSendKey";
    string eventHubSendKey = SharedAccessAuthorizationRule.GenerateRandomKey();
    SharedAccessAuthorizationRule eventHubSendRule = new SharedAccessAuthorizationRule(eventHubSendKeyName, eventHubSendKey, new[] { AccessRights.Send });
    ed.Authorization.Add(eventHubSendRule); 
    nm.CreateEventHub(ed);

### <a name="generate-tokens"></a>生成令牌

可以使用 SAS 密钥生成令牌。 对于每个客户端只能生成一个令牌。 然后，可以使用以下方法生成令牌。 所有令牌都使用 **EventHubSendKey** 密钥生成。 将为每个令牌分配一个唯一 URI。

    public static string SharedAccessSignatureTokenProvider.GetSharedAccessSignature(string keyName, string sharedAccessKey, string resource, TimeSpan tokenTimeToLive)

调用此方法时，应将 URI 指定为 `//<NAMESPACE>.servicebus.chinacloudapi.cn/<EVENT_HUB_NAME>/publishers/<PUBLISHER_NAME>`。 所有令牌的 URI 都是相同的，但每个令牌的 `PUBLISHER_NAME` 应该不同。 `PUBLISHER_NAME` 最好是表示接收该令牌的客户端 ID。

此方法将生成具有以下结构的令牌：

    SharedAccessSignature sr={URI}&sig={HMAC_SHA256_SIGNATURE}&se={EXPIRATION_TIME}&skn={KEY_NAME}

令牌过期时间以从 1970 年 1 月 1 日开始算起的秒数指定。 下面是一个令牌示例：

    SharedAccessSignature sr=contoso&sig=nPzdNN%2Gli0ifrfJwaK4mkK0RqAB%2byJUlt%2bGFmBHG77A%3d&se=1403130337&skn=RootManageSharedAccessKey

通常，令牌的使用期限相当于或长于客户端的使用期限。 如果客户端能够获取新令牌，则可以使用使用期限较短的令牌。

### <a name="sending-data"></a>发送数据

创建令牌后，将为每个客户端设置其自身唯一的令牌。

当客户端向事件中心发送数据时，客户端会在发送请求中标记其令牌。 为了防止攻击者窃听和盗取令牌，客户端与事件中心之间的通信必须通过加密通道进行。

### <a name="blacklisting-clients"></a>将客户端列入黑名单

如果令牌被攻击者盗取，攻击者可能会模拟令牌已被盗的客户端。 将客户端列入黑名单会导致客户端不可用，直至该客户端收到使用其他发布者的新令牌。

## <a name="authentication-of-back-end-applications"></a>后端应用程序的身份验证

为了对使用事件中心客户端生成的数据的后端应用程序进行身份验证，事件中心采用了与服务总线主题所用模型类似的安全模型。 事件中心使用者组相当于服务总线主题的订阅。 如果创建使用者组的请求附带了一个用于授予对事件中心或者对事件中心所属命名空间的管理权限的令牌，则客户端可以创建该使用者组。 如果接收请求附带了一个用于授予对使用者组、事件中心或者事件中心所属命名空间的接收权限的令牌，则允许客户端使用该使用者组的数据。

当前版本的服务总线不支持对单个订阅使用 SAS 规则。 对于事件中心使用者组也是如此。 将来会添加对这两项功能的 SAS 支持。

在无法为单个使用者组进行 SAS 身份验证的情况下，使用 SAS 密钥来保护具有一个公用密钥的所有使用者组。 此方法能使应用程序使用事件中心的所有使用者组的数据。

## <a name="next-steps"></a>后续步骤

若要了解有关事件中心的详细信息，请访问以下主题：

* [事件中心概述]
* [共享访问签名概述]
* [使用事件中心的示例应用程序]

[事件中心概述]: /documentation/articles/event-hubs-what-is-event-hubs/
[使用事件中心的示例应用程序]: https://github.com/Azure/azure-event-hubs/tree/master/samples
[共享访问签名概述]: /documentation/articles/service-bus-sas/

<!-- Update_Description:update meta properties;wording update -->