<properties
    pageTitle="了解 Azure IoT 中心安全性 | Azure"
    description="开发人员指南 - 如何控制设备应用和后端应用对 IoT 中心的访问。 其中包括安全令牌和 X.509 证书支持的相关信息。"
    services="iot-hub"
    documentationcenter=".net"
    author="dominicbetts"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="45631e70-865b-4e06-bb1d-aae1175a52ba"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/04/2017"
    wacn.date="04/24/2017"
    ms.author="dobett"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="57d3c3bcbe8a61de0c037638b1ca0fd14d7770cc"
    ms.lasthandoff="04/14/2017" />

# <a name="control-access-to-iot-hub"></a>控制 IoT 中心的访问权限
## <a name="overview"></a>概述
本文介绍用于保护 IoT 中心的选项。 IoT 中心使用 *权限* 向每个 IoT 中心终结点授予访问权限。 权限可根据功能限制对 IoT 中心的访问。

本文将介绍：

- 可以向要访问 IoT 中心的设备或后端应用授予的不同权限。
- 身份验证过程以及它用于验证权限的令牌。
- 如何限定凭据的作用域，以限制对特定资源的访问。
- IoT 中心支持 X.509 证书。
- 使用现有的设备标识注册表或身份验证方案的自定义设备身份验证机制。

### <a name="when-to-use"></a>何时使用
必须具有适当的权限，才能访问任何 IoT 中心终结点。 例如，设备必须随它发送到 IoT 中心的每条消息提供包含安全凭据的令牌。

## <a name="access-control-and-permissions"></a> 访问控制和权限
可以通过以下方式授予[权限](#iot-hub-permissions)：

* **IoT 中心级别的共享访问策略**。共享访问策略可以授予任意[权限](#iot-hub-permissions)组合。可以在 [Azure 门户预览][lnk-management-portal]中定义策略，或使用 [IoT 中心资源提供程序 REST API][lnk-resource-provider-apis] 以编程方式定义策略。新建的 IoT 中心有以下默认策略：
  
  * **iothubowner**：包含所有权限的策略。
  * **service**：包含 **ServiceConnect** 权限的策略。
  * **device**：包含 **DeviceConnect** 权限的策略。
  * **registryRead**：包含 **RegistryRead** 权限的策略。
  * **registryReadWrite**：包含 **RegistryRead** 和 RegistryWrite 权限的策略。
  * **每个设备的安全凭据**。 每个 IoT 中心均包含一个 [标识注册表][lnk-identity-registry]。 对于此标识注册表中的每个设备，可配置安全凭据，授予局限于相应设备终结点的 **DeviceConnect** 权限。

例如，在典型的 IoT 解决方案中：

* 设备管理组件使用 *registryReadWrite* 策略。
* 事件处理器组件使用 *service* 策略。
* 运行时设备业务逻辑组件使用 *service* 策略。
* 各个设备的连接使用 IoT 中心的标识注册表中存储的凭据。

> [AZURE.NOTE]
> 有关详细信息，请参阅[权限](#iot-hub-permissions)。

## <a name="authentication"></a>身份验证
Azure IoT 中心可根据共享访问策略和标识注册表安全凭据验证令牌，进而授予终结点的访问权限。

安全凭据（例如对称密钥）永远不会通过网络发送。

> [AZURE.NOTE]
> 如同 [Azure Resource Manager][lnk-azure-resource-manager] 中的所有提供程序一样，Azure IoT 中心资源提供程序也通过 Azure 订阅受到保护。
> 
> 

有关如何构造和使用安全令牌的详细信息，请参阅 [IoT 中心安全令牌][lnk-sas-tokens]。

### <a name="protocol-specifics"></a>协议详情
每个支持的协议（如 MQTT、AMQP 和 HTTP）以不同方式传输令牌。

使用 MQTT 时，CONNECT 数据包具有用作 ClientId 的 deviceId，在 Username 字段中具有 {iothubhostname}/{deviceId}，在 Password 字段中具有 SAS 令牌。 {iothubhostname} 应该是 IoT 中心的完整 CName（例如，contoso.azure-devices.cn）。

使用 [AMQP][lnk-amqp] 时，IoT 中心支持 [SASL PLAIN][lnk-sasl-plain] 和 [AMQP 基于声明的安全性][lnk-cbs]。

如果使用 AMQP 基于声明的安全性，标准将指定如何传输这些令牌。

对于 SASL PLAIN，**用户名**可以是：

* `{policyName}@sas.root.{iothubName}`（若使用 IoT 中心级别的令牌）。
* `{deviceId}@sas.{iothubname}`（若为设备范围的令牌）。

在这两种情况下，密码字段都包含令牌，如 [IoT Hub security tokens][lnk-sas-tokens]（IoT 中心安全令牌）中所述。

HTTP 通过在**授权**请求标头中包含有效的令牌来实施身份验证。

#### <a name="example"></a>示例
用户名（DeviceId 区分大小写）：
`iothubname.azure-devices.cn/DeviceId`

密码（使用[设备资源管理器][lnk-device-explorer]工具生成 SAS 令牌）：`SharedAccessSignature sr=iothubname.azure-devices.cn%2fdevices%2fDeviceId&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501`

> [AZURE.NOTE]
> [Azure IoT SDK][lnk-sdks] 在连接到服务时自动生成令牌。 某些情况下，Azure IoT SDK 不支持部分协议或身份验证方法。
> 
> 

### <a name="special-considerations-for-sasl-plain"></a>有关 SASL PLAIN 的特殊注意事项
将 SASL PLAIN 用于 AMQP 时，连接到 IoT 中心的客户端可为每个 TCP 连接使用单个令牌。 当令牌过期时，TCP 将从服务断开连接，并触发重新连接。 此行为虽然不会对后端应用造成问题，但对设备应用不利，原因如下：

*  网关通常代表许多设备连接。使用 SASL PLAIN 时，它们必须针对连接到 IoT 中心的每个设备创建不同的 TCP 连接。此方案会大幅提高电源与网络资源的消耗并增大每个设备连接的延迟。
* 在每个令牌过期后，增加使用要重新连接的资源通常会对资源受限的设备造成不良影响。

## <a name="scope-iot-hub-level-credentials"></a>设置 IoT 中心级凭据的范围
可通过使用受限资源 URI 创建令牌，设置 IoT 中心级安全策略的范围。 例如，要从设备发送从设备到云的消息的终结点是 **/devices/{deviceId}/messages/events**。 还可以使用包含 **DeviceConnect** 权限的 IoT 中心级别共享访问策略对 resourceURI 为 **/devices/{deviceId}** 的令牌进行签名。 此方法将创建一个令牌，该令牌仅可用于代表设备 **deviceId** 发送消息。

此机制类似于[事件中心发布者策略][lnk-event-hubs-publisher-policy]，可用于实施自定义身份验证方法。

## <a name="security-tokens"></a> 安全令牌
IoT 中心使用安全令牌对设备和服务进行身份验证，以避免在线发送密钥。并且安全令牌的有效期和范围有限。[Azure IoT SDK][lnk-sdks] 无需任何特殊配置即可自动生成令牌。但在某些情况下，需要用户生成并直接使用安全令牌。这些情况包括 MQTT、AMQP 或 HTTP 应用层协议的直接使用，以及令牌服务模式的实现（如[自定义设备身份验证][lnk-custom-auth]中所述）。

IoT 中心还允许设备使用 [X.509 证书][lnk-x509]向 IoT 中心进行身份验证。

### <a name="security-token-structure"></a> 安全令牌结构
可以使用安全令牌向设备和服务授予限时访问 IoT 中心特定功能的权限。若要确保只有经过授权的设备和服务能够连接，安全令牌必须使用共享访问密钥或对称密钥进行签名。这些密钥与设备标识一起存储在标识注册表中。

使用共享访问密钥进行签名的令牌可授权访问与共享访问策略权限相关的所有功能。另一方面，使用设备标识的对称密钥进行签名的令牌只能向相关设备标识授予 **DeviceConnect** 权限。

安全令牌采用以下格式：

    SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}

以下是预期值：

| 值 | 说明 |
| --- | --- |
| {signature} |HMAC-SHA256 签名字符串的格式为：`{URL-encoded-resourceURI} + "\n" + expiry`。**重要说明**：密钥是从 base64 解码得出的，用作执行 HMAC-SHA256 计算的密钥。 |
| {resourceURI} |使用此令牌可以访问的终结点的 URI 前缀（根据分段）以 IoT 中心的主机名开头（无协议）。例如 `myHub.azure-devices.cn/devices/device1` |
| {expiry} |从纪元 1970 年 1 月 1日 00:00:00 UTC 时间至今秒数的 UTF8 字符串。 |
| {URL-encoded-resourceURI} |小写资源 URI 的小写 URL 编码 |
| {policyName} |此令牌所引用的共享访问策略名称。如果此令牌引用设备注册表凭据，则空缺。 |

**有关前缀的说明**：URI 前缀是根据分段而不是字符计算的。例如，`/a/b` 是 `/a/b/c` 的前缀，而不是 `/a/bc` 的前缀。

以下 Node.js 代码片段显示名为 **generateSasToken** 的函数，该函数通过输入 `resourceUri, signingKey, policyName, expiresInMins` 计算令牌。以下各节将详细讲解如何初始化不同令牌用例的不同输入。

    var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
        resourceUri = encodeURIComponent(resourceUri);

        // Set expiration in seconds
        var expires = (Date.now() / 1000) + expiresInMins * 60;
        expires = Math.ceil(expires);
        var toSign = resourceUri + '\n' + expires;

        // Use crypto
        var hmac = crypto.createHmac('sha256', new Buffer(signingKey, 'base64'));
        hmac.update(toSign);
        var base64UriEncoded = encodeURIComponent(hmac.digest('base64'));

        // Construct autorization string
        var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
        + base64UriEncoded + "&se=" + expires;
        if (policyName) token += "&skn="+policyName;
        return token;
    };

作为对照，用于生成安全令牌的等效 Python 代码是：

    from base64 import b64encode, b64decode
    from hashlib import sha256
    from time import time
    from urllib import quote_plus, urlencode
    from hmac import HMAC

    def generate_sas_token(uri, key, policy_name, expiry=3600):
        ttl = time() + expiry
        sign_key = "%s\n%d" % ((quote_plus(uri)), int(ttl))
        print sign_key
        signature = b64encode(HMAC(b64decode(key), sign_key, sha256).digest())

        rawtoken = {
            'sr' :  uri,
            'sig': signature,
            'se' : str(int(ttl))
        }

        if policy_name is not None:
            rawtoken['skn'] = policy_name

        return 'SharedAccessSignature ' + urlencode(rawtoken)

> [AZURE.NOTE]
> 由于 IoT 中心计算机会验证令牌的有效期，因此生成令牌的计算机的时间偏差必须很小。
> 
> 

### <a name="use-sas-tokens-in-a-device-app"></a>在设备应用中使用 SAS 令牌
有两种方法可以使用安全令牌来获取 IoT 中心的 **DeviceConnect** 权限：使用[标识注册表中的对称设备密钥](#use-a-symmetric-key-in-the-identity-registry)，或者使用[共享访问密钥](#use-a-shared-access-policy)。

请记住，可从设备访问的所有功能都故意显示在前缀为 `/devices/{deviceId}` 的终结点上。

> [AZURE.IMPORTANT]
> IoT 中心对某个特定设备进行身份验证的唯一方法是使用设备标识对称密钥。 使用共享访问策略访问设备功能时，解决方案必须考虑将安全令牌作为受信任的子组件进行颁发的组件。

面向设备的终结点包括（无论任何协议）：

| 终结点 | 功能 |
| --- | --- |
| `{iot hub host name}/devices/{deviceId}/messages/events` | 发送设备到云的消息。 |
| `{iot hub host name}/devices/{deviceId}/devicebound`   | 接收云到设备的消息。 |


### <a name="use-a-symmetric-key-in-the-identity-registry"></a> 使用标识注册表中的对称密钥
使用设备标识的对称密钥生成令牌时，将省略令牌的 policyName (`skn`) 元素。

例如，创建的用于访问所有设备功能的令牌应具有以下参数：

* 资源 URI：`{IoT hub name}.azure-devices.cn/devices/{device id}`，
* 签名密钥：`{device id}` 标识的任何对称密钥，
* 无策略名称；
* 任何过期时间。

上述 Node js 函数的使用示例如下：

    var endpoint ="myhub.azure-devices.cn/devices/device1";
    var deviceKey ="...";

    var token = generateSasToken(endpoint, deviceKey, null, 60);

授权访问设备 1 的所有功能的安全令牌是：

    SharedAccessSignature sr=myhub.azure-devices.cn%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697

> [AZURE.NOTE]
> 可使用 .NET [设备资源管理器][lnk-device-explorer]工具或基于节点的跨平台 [iothub-explorer][lnk-iothub-explorer] 命令行实用程序生成 SAS 令牌。
> 
> 

### <a name="use-a-shared-access-policy"></a> 使用共享访问策略

使用共享访问策略创建令牌时，必须将策略名称字段 `skn` 设置为所使用的策略的名称。并且要求策略授予 **DeviceConnect** 权限。

使用共享访问策略访问设备功能的两个主要方案是：

* [云协议网关][lnk-endpoints]，
* [令牌服务][lnk-custom-auth] 。

由于共享访问策略可潜在授权访问任何连接设备的权限，因此创建安全令牌时必须使用正确的资源 URI。这对令牌服务尤其重要，它必须使用资源 URI 将令牌的范围限定到特定设备。这一点与协议网关的关系不大，因为协议网关是对所有设备的通信进行调节。

例如，使用名为 **device** 的预创建共享访问策略的令牌服务会使用以下参数创建令牌：

* 资源 URI：`{IoT hub name}.azure-devices.cn/devices/{device id}`，
* 签名密钥：`device` 策略的密钥之一，
* 策略名称：`device`，
* 任何过期时间。

上述 Node js 函数的使用示例如下：

    var endpoint ="myhub.azure-devices.cn/devices/device1";
    var policyName = 'device';
    var policyKey = '...';

    var token = generateSasToken(endpoint, policyKey, policyName, 60);

授权访问设备 1 的所有功能的安全令牌是：

    SharedAccessSignature sr=myhub.azure-devices.cn%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697&skn=device

协议网关可以对所有设备使用相同的令牌，只需将资源 URI 设置为 `myhub.azure-devices.cn/devices`。

### <a name="use-security-tokens-from-service-components"></a> 使用服务组件提供的安全令牌
如前所述，服务组件使用共享访问策略只能生成安全令牌，授予适当权限。

以下是终结点上显示的服务功能：

| 终结点 | 功能 |
| --- | --- |
| `{iot hub host name}/devices` | 创建、更新、检索和删除设备标识。 |
| `{iot hub host name}/messages/events` | 接收设备到云的消息 |
| `{iot hub host name}/servicebound/feedback` | 接收云到设备的消息的反馈。 |
| `{iot hub host name}/devicebound`  | 发送云到设备的消息。 |

例如，使用名为 **registryRead** 的预创建共享访问策略生成的服务会使用以下参数创建令牌：

* 资源 URI：`{IoT hub name}.azure-devices.cn/devices`，
* 签名密钥：`registryRead` 策略的密钥之一，
* 策略名称：`registryRead`，
* 任何过期时间。

        var endpoint ="myhub.azure-devices.cn/devices";
        var policyName = 'device';
        var policyKey = '...';
    
        var token = generateSasToken(endpoint, policyKey, policyName, 60)；

授权读取所有设备标识权限的安全令牌是：

    SharedAccessSignature sr=myhub.azure-devices.cn%2fdevices&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=registryRead

## <a name="supported-x509-certificates"></a> 支持的 X.509 证书
可以使用任何 X.509 证书通过 IoT 中心对设备进行身份验证。证书包括：

-   **现有的 X.509 证书**。设备可能已有与之关联的 X.509 证书。设备可以使用此证书向 IoT 中心进行身份验证。

-   **自行生成和自签名的 X-509 证书**。设备制造商或内部部署人员可以生成这些证书，并将相应的私钥（和证书）存储在设备上。你可以将工具（如 [OpenSSL][lnk-openssl] 和 [Windows SelfSignedCertificate][lnk-selfsigned] 实用程序）用于此目的。

-   **CA 签名的 X.509 证书**。还可以使用证书颁发机构 (CA) 生成和签名的的 X.509 证书来识别设备并通过 IoT 中心对设备进行身份验证。

设备可以使用 X.509 证书或安全令牌进行身份验证，但不能同时使用这两者。

### <a name="register-an-x509-certificate-for-a-device"></a>为设备注册 X.509 证书
[用于 C# 的 Azure IoT 服务 SDK][lnk-service-sdk]（版本 1.0.8+）支持注册使用 X.509 证书进行身份验证的设备。 其他 API（例如设备的导入/导出）也支持 X.509 证书。

### <a name="c-support"></a>C\# 支持
**RegistryManager** 类提供了用于注册设备的编程方式。 具体而言，使用 **AddDeviceAsync** 和 **UpdateDeviceAsync** 方法，用户可以在 IoT 中心标识注册表中注册和更新设备。 这两种方法均采用 **Device** 实例作为输入。 **Device** 类包括 **Authentication** 属性，允许用户指定主要和次要 X.509 证书指纹。 指纹表示 X.509 证书的 SHA-1 哈希值（使用二进制 DER 编码存储）。 用户可以选择指定主要指纹和/或次要指纹。 为了处理证书滚动更新方案，支持主要和次要指纹。

> [AZURE.NOTE]
> IoT 中心不需要也不存储整个 X.509 证书，它仅存储指纹。
> 
> 

下面是使用 X.509 证书注册设备的示例 C# 代码片段：

	var device = new Device(deviceId)
	{
	  Authentication = new AuthenticationMechanism()
	  {
	    X509Thumbprint = new X509Thumbprint()
	    {
	      PrimaryThumbprint = "921BC9694ADEB8929D4F7FE4B9A3A6DE58B0790B"
	    }
	  }
	};
	RegistryManager registryManager = RegistryManager.CreateFromConnectionString(deviceGatewayConnectionString);
	await registryManager.AddDeviceAsync(device);

### <a name="use-an-x509-certificate-during-run-time-operations"></a>在运行时操作期间使用 X.509 证书
[用于 .NET 的 Azure IoT 设备 SDK][lnk-client-sdk]（版本 1.0.11+）支持使用 X.509 证书。

### <a name="c-support"></a>C\# 支持
类 **DeviceAuthenticationWithX509Certificate** 支持使用 X.509 证书创建  **DeviceClient** 实例。 X.509 证书必须采用 PFX（也称为 PKCS #12）格式，其中包含私钥。 

下面是示例代码片段：

	var authMethod = new DeviceAuthenticationWithX509Certificate("<device id>", x509Certificate);

	var deviceClient = DeviceClient.Create("<IotHub DNS HostName>", authMethod);

## <a name="custom-device-authentication"></a>自定义设备身份验证
可以使用 IoT 中心[标识注册表][lnk-identity-registry]，通过[令牌][lnk-sas-tokens]配置每个设备的安全凭据和访问控制。 如果 IoT 解决方案已经大幅投资自定义标识注册表和/或身份验证方案，可以通过创建*令牌服务*，将此现有基础结构与 IoT 中心集成。 这样，便可以在解决方案中使用其他 IoT 功能。

令牌服务是自定义云服务。它使用包含 **DeviceConnect** 权限的 IoT 中心 *共享访问策略* 创建 *设备范围的* 令牌。这些令牌可让设备连接到 IoT 中心。

  ![令牌服务模式的步骤][img-tokenservice]

下面是令牌服务模式的主要步骤：

1. 为 IoT 中心创建包含 **DeviceConnect** 权限的 IoT 中心共享访问策略。可以在 [Azure 门户预览][lnk-management-portal]中或以编程方式创建此策略。令牌服务使用此策略为它创建的令牌签名。
2. 当设备需要访问 IoT 中心时，将向令牌服务请求已签名的令牌。设备可使用自定义标识注册表/身份验证方案进行身份验证，确定令牌服务用于创建令牌的设备标识。
3. 令牌服务返回令牌。使用 `/devices/{deviceId}` 作为 `resourceURI`、使用 `deviceId` 作为所要进行身份验证的设备创建令牌。令牌服务使用共享访问策略来构造令牌。
4. 设备直接通过 IoT 中心使用令牌。

> [AZURE.NOTE]
> 可以使用 .NET 类 [SharedAccessSignatureBuilder][lnk-dotnet-sas] 或 Java 类 [IotHubServiceSasToken][lnk-java-sas] 在令牌服务中创建令牌。
> 
> 

令牌服务可以根据需要设置令牌过期日期。令牌过期时，IoT 中心将断开设备连接。然后，设备必须向令牌服务请求新令牌。到期时间过短会增加设备和令牌服务上的负载。

设备若要连接到中心，即使连接使用的是令牌而不是设备密钥，仍须将设备添加 IoT 中心标识注册表。因此，你可以在设备使用令牌身份验证时，通过在 [IoT 中心识别注册表][lnk-identity-registry]中启用或禁用设备标识，来继续使用基于设备的访问控制。此方法可以减轻使用较长到期时间令牌的风险。

### <a name="comparison-with-a-custom-gateway"></a>与自定义网关的比较
令牌服务模式是使用 IoT 中心实现自定义标识注册表/身份验证方案的建议方式。 提供这种建议是因为 IoT 中心继续处理大部分解决方案流量。 但在某些情况下，自定义身份验证方案和协议过度交织，因此需要可处理所有流量（*自定义网关*）的服务。 使用[传输层安全 (TLS) 和预共享密钥 (PSK)][lnk-tls-psk] 就是这种情况的例子。 有关详细信息，请参阅[协议网关][lnk-protocols]主题。

## <a name="reference-topics"></a>参考主题：
以下参考主题提供有关控制对 IoT 中心的访问的详细信息。

## <a name="iot-hub-permissions"></a> IoT 中心权限
下表列出了可用于控制对 IoT 中心的访问的权限。

| 权限 | 说明 |
| --- | --- |
| **RegistryRead** |授予对标识注册表的读取访问权限。 有关详细信息，请参阅[标识注册表][lnk-identity-registry]。 <br/>后端云服务将使用此权限。 |
| **RegistryReadWrite** |授予对标识注册表的读取和写入访问权限。 有关详细信息，请参阅[标识注册表][lnk-identity-registry]。 <br/>后端云服务将使用此权限。 |
| **ServiceConnect** |授予对面向云服务的通信和监视终结点的访问权限。 <br/>授予接收设备到云消息、发送云到设备消息和检索相应传送确认的权限。 <br/>授予检索文件上载的传送确认的权限。 <br/>授予访问设备孪生以更新标记和所需属性、检索报告属性和运行查询的权限。 <br/>后端云服务将使用此权限。 |
| **DeviceConnect** |授予对面向设备的终结点的访问权限。 <br/>授予发送设备到云消息和接收云到设备消息的权限。 <br/>授予从设备执行文件上载的权限。 <br/>授予接收设备孪生所需属性通知和更新设备孪生报告属性的权限。 <br/>授予执行文件上载的权限。 <br/>此权限由设备使用。 |

## <a name="additional-reference-material"></a>其他参考资料
IoT 中心开发人员指南中的其他参考主题包括：

* [IoT 中心终结点][lnk-endpoints]，介绍了每个 IoT 中心针对运行时和管理操作公开的各种终结点。
* [限制和配额][lnk-quotas]，说明了适用于 IoT 中心服务的配额，以及使用服务时预期会碰到的限制行为。
* [Azure IoT 设备和服务 SDK][lnk-sdks]，列出了在开发与 IoT 中心交互的设备和服务应用时可使用的各种语言 SDK。
* [设备孪生和作业的 IoT 中心查询语言][lnk-query]，介绍了在 IoT 中心检索设备孪生和作业相关信息时可使用的 IoT 中心查询语言。
* [IoT 中心 MQTT 支持][lnk-devguide-mqtt]提供有关 IoT 中心对 MQTT 协议的支持的详细信息。

## <a name="next-steps"></a>后续步骤
现已了解如何控制对 IoT 中心的访问，你可能有兴趣了解以下 IoT 中心开发人员指南主题：

- [使用设备孪生同步状态和配置][lnk-devguide-device-twins]
- [在设备上调用直接方法][lnk-devguide-directmethods]
- [在多台设备上计划作业][lnk-devguide-jobs]

如果要尝试本文中介绍的一些概念，你可能对以下 IoT 中心教程感兴趣：

- [Azure IoT 中心入门][lnk-getstarted-tutorial]
- [如何使用 IoT 中心发送云到设备的消息][lnk-c2d-tutorial]
- [如何处理 IoT 中心设备到云的消息][lnk-d2c-tutorial]

<!-- links and images -->


[img-tokenservice]: ./media/iot-hub-devguide-security/tokenservice.png
[lnk-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/
[lnk-openssl]: https://www.openssl.org/
[lnk-selfsigned]: https://technet.microsoft.com/zh-cn/library/hh848633

[lnk-resource-provider-apis]: https://msdn.microsoft.com/zh-cn/library/mt548492.aspx
[lnk-sas-tokens]: /documentation/articles/iot-hub-devguide-security/#security-tokens
[lnk-amqp]: https://www.amqp.org/
[lnk-azure-resource-manager]: /documentation/articles/resource-group-overview/
[lnk-cbs]: https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc
[lnk-event-hubs-publisher-policy]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab
[lnk-management-portal]: https://portal.azure.cn
[lnk-sasl-plain]: http://tools.ietf.org/html/rfc4616
[lnk-identity-registry]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-dotnet-sas]: https://msdn.microsoft.com/zh-cn/library/microsoft.azure.devices.common.security.sharedaccesssignaturebuilder.aspx
[lnk-java-sas]: https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.service.auth._iot_hub_service_sas_token
[lnk-tls-psk]: https://tools.ietf.org/html/rfc4279
[lnk-protocols]: /documentation/articles/iot-hub-protocol-gateway/
[lnk-custom-auth]: /documentation/articles/iot-hub-devguide-security/#custom-device-authentication
[lnk-x509]: /documentation/articles/iot-hub-devguide-security/#supported-x509-certificates
[lnk-devguide-device-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-devguide-directmethods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-service-sdk]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/service
[lnk-client-sdk]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/device
[lnk-device-explorer]: https://github.com/Azure/azure-iot-sdk-csharp/blob/master/tools/DeviceExplorer
[lnk-iothub-explorer]: https://github.com/azure/iothub-explorer

[lnk-getstarted-tutorial]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-c2d-tutorial]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[lnk-d2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/


<!--Update_Description:update wording and link references-->