<properties
 pageTitle="如何生成 IoT 中心安全令牌 | Azure"
 description="说明 IoT 中心使用的不同类型的安全令牌和如何生成这些令牌。"
 services="iot-hub"
 documentationCenter=".net"
 authors="fsautomata"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="03/02/2016"
wacn.date="05/05/2016"/>

# 使用 IoT 中心安全令牌

IoT 中心使用安全令牌对设备和服务进行身份验证，以避免在线发送密钥。并且安全令牌的有效期和范围有限。
[Azure IoT 中心 SDK][lnk-apis-sdks] 无需任何特殊配置即可自动生成令牌。但在某些情况下，需要用户生成并直接使用安全令牌。包括 AMQP、MQTT 或 HTTP 应用层协议的直接使用，以及令牌服务模式的实现（如 [IoT 中心指南][lnk-guidance-security]中所述）。

本文将介绍：

* 安全令牌的格式以及如何生成安全令牌。
* 使用安全令牌对设备和后端服务进行身份验证的主要用例。

## 安全令牌结构
可以使用安全令牌向设备和服务授予限时访问 IoT 中心特定功能的权限。为确保只有经过授权的设备和服务能够连接，安全令牌必须使用共享访问策略密钥或存储在标识注册表中并带有设备标识的对称密钥进行签名。

使用共享访问策略密钥进行签名的令牌可以授权访问与共享访问策略权限相关的所有功能。请参阅 [IoT 中心开发人员指南的安全性部分][lnk-devguide-security]。而使用设备标识的对称密钥进行签名的令牌只能向相关设备标识授予 **DeviceConnect** 权限。

安全令牌采用以下格式：

    SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}

以下是预期值：

| 值 | 说明 |
| ----- | ----------- |
| {signature} | HMAC-SHA256 签名字符串的格式为：`{URL-encoded-resourceURI} + "\n" + expiry`。**重要说明**：密钥是从 base64 解码得出的，用作执行 HMAC-SHA256 计算的密钥。 |
| {resourceURI} | 此令牌可以访问的终结点的 URI 前缀（根据分段）以 IoT 中心的主机名开始（无协议）。例如 `myHub.azure-devices.net/devices/device1` |
| {expiry} | 从纪元 1970 年 1 月 1日 00:00:00 UTC 时间至今秒数的 UTF8 字符串。 |
| {URL-encoded-resourceURI} | 小写资源 URI 的小写 URL 编码 |
| {policyName} | 此令牌所引用的共享访问策略名称。在令牌引用设备注册表凭据的情况下不存在。 |

**有关前缀的说明**：URI 前缀是根据分段而不是字符计算的。例如，`/a/b` 是 `/a/b/c` 的前缀，而不是 `/a/bc` 的前缀。

这是一个根据 `resourceUri, signingKey, policyName, expiresInMins` 输入计算令牌的 Node 函数。以下各节将详细讲解如何初始化不同令牌用例的不同输入。

    var crypto = require('crypto');

    var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
        resourceUri = encodeURIComponent(resourceUri.toLowerCase()).toLowerCase();

        // Set expiration in seconds
        var expires = (Date.now() / 1000) + expiresInMins * 60;
        expires = Math.ceil(expires);
        var toSign = resourceUri + '\n' + expires;

        // using crypto
        var decodedPassword = new Buffer(signingKey, 'base64').toString('binary');
        const hmac = crypto.createHmac('sha256', decodedPassword);
        hmac.update(toSign);
        var base64signature = hmac.digest('base64');
        var base64UriEncoded = encodeURIComponent(base64signature);

        // construct autorization string
        var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
        + base64UriEncoded + "&se=" + expires;
        if (policyName) token += "&skn="+policyName;
        // console.log("signature:" + token);
        return token;
    };

> [AZURE.NOTE] 由于 IoT 中心计算机会验证令牌的有效期，因此生成令牌的计算机的时间偏差必须很小。

## 将 SAS 令牌当做设备使用

有两种方法可以使用安全令牌来获取 IoT 中心的 **DeviceConnect** 权限：使用设备标识密钥，或者使用共享访问策略密钥。

此外，必须注意的是，可从设备访问的所有功能都故意显示在前缀为 `/devices/{deviceId}` 的终结点上。

> [AZURE.IMPORTANT] IoT 中心对某个特定设备进行身份验证的唯一方法是使用设备标识对称密钥。使用共享访问策略访问设备功能时，解决方案必须考虑将安全令牌作为受信任的子组件进行颁发的组件。

面向设备的终结点包括（无论任何协议）：

| 终结点 | 功能 |
| ----- | ----------- |
| `{iot hub host name}/devices/{deviceId}/messages/events` | 发送设备到云的消息。 |
| `{iot hub host name}/devices/{deviceId}/devicebound` | 接收云到设备的消息。 |

### 使用标识注册表中的对称密钥

使用设备标识的对称密钥生成令牌时，将省略令牌的 policyName (`skn`) 元素。

例如，创建的用于访问所有设备功能的令牌应具有以下参数：

* 资源 URI：`{IoT hub name}.azure-devices.net/devices/{device id}`；
* 签名密钥：`{device id}` 标识的任何对称密钥；
* 无策略名称；
* 任何过期时间。

使用上述 Node 函数的示例如下：

    var endpoint ="myhub.azure-devices.net/devices/device1";
    var deviceKey ="...";

    var token = generateSasToken(endpoint, deviceKey, null, 60);

授权访问设备 1 的所有功能的安全令牌是：

    SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697

> [AZURE.NOTE] 可以使用 .NET 工具[设备资源管理器][lnk-device-explorer]来生成安全令牌。

### 使用共享访问策略

使用共享访问策略创建令牌时，必须将策略名称字段 `skn` 设置为所使用的策略的名称。并且要求策略授予 **DeviceConnect** 权限。

使用共享访问策略访问设备功能的两个主要方案是：

* [云协议网关][lnk-azure-protocol-gateway]，
* 用于实现自定义身份验证方案的[令牌服务][lnk-devguide-security]。

由于共享访问策略可潜在授权访问任何连接设备的权限，因此创建安全令牌时必须使用正确的资源 URI。这对令牌服务尤其重要，它必须使用资源 URI 将令牌的范围限定到特定设备。这一点与协议网关的关系不大，因为协议网关是对所有设备的通信进行调节。

例如，使用名为 **device** 的预创建共享访问策略的令牌服务所创建的令牌将包含以下参数：

* 资源 URI：`{IoT hub name}.azure-devices.net/devices/{device id}`；
* 签名密钥：`device` 策略的密钥之一；
* 策略名称：`device`；
* 任何过期时间。

使用上述 Node 函数的示例如下：

    var endpoint ="myhub.azure-devices.net/devices/device1";
    var policyName = 'device';
    var policyKey = '...';

    var token = generateSasToken(endpoint, policyKey, policyName, 60);

授权访问设备 1 的所有功能的安全令牌是：

    SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697&skn=device

协议网关对所有设备使用相同的令牌，只需将资源 URI 设置为 `myhub.azure-devices.net/devices`。

## 使用服务组件提供的安全令牌

如[IoT 中心开发人员指南的安全性部分][lnk-devguide-security]所述，服务组件只能使用共享访问策略生成安全令牌，授予适当权限。

以下是终结点上显示的服务功能：

| 终结点 | 功能 |
| ----- | ----------- |
| `{iot hub host name}/devices` | 创建、更新、检索和删除设备标识。 |
| `{iot hub host name}/messages/events` | 接收设备到云的消息 |
| `{iot hub host name}/servicebound/feedback` | 接收云到设备的消息的反馈。 |
| `{iot hub host name}/devicebound` | 发送云到设备的消息。 |

例如，使用名为 **registryRead** 的预创建共享访问策略的服务生成所创建的令牌将包含以下参数：

* 资源 URI：`{IoT hub name}.azure-devices.net/devices`；
* 签名密钥：`registryRead` 策略的密钥之一；
* 策略名称：`registryRead`；
* 任何过期时间。

```
    var endpoint ="myhub.azure-devices.net/devices";
    var policyName = 'device';
    var policyKey = '...';

    var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

授权读取所有设备标识权限的安全令牌是：

    SharedAccessSignature sr=myhub.azure-devices.net%2fdevices&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=registryRead


[lnk-apis-sdks]: https://github.com/Azure/azure-iot-sdks/blob/master/readme.md
[lnk-guidance-security]: /documentation/articles/iot-hub-guidance/#customauth
[lnk-devguide-security]: /documentation/articles/iot-hub-devguide/#security
[lnk-azure-protocol-gateway]: /documentation/articles/iot-hub-protocol-gateway
[lnk-device-explorer]: https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/doc/how_to_use_device_explorer.md
<!---HONumber=Mooncake_0418_2016-->