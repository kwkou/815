# 保护 IoT 部署

本文提供保护基于 Azure IoT 的物联网 (IoT) 基础结构的进一步详细信息。它链接到配置和部署每个组件的实现级别详细信息。还提供多种竞争方式间的比较和选择。

保护 Azure IoT 部署可分为以下三个安全区域：

- **设备安全**：在现实中部署 IoT 设备时，保护 IoT 设备安全。
- **连接安全**：确保 IoT 设备和 IoT 中心之间传输的所有数据的机密性和防篡改性。
- **云安全**：数据移动或存储在云中时，提供一种数据保护方式。

![三个安全区域][img-overview]

## 安全的设备预配和身份验证

Azure IoT 套件通过以下两种方式保护 IoT 设备：

- 为每个设备提供唯一标识密钥（安全令牌），设备可使用该密钥与 IoT 中心通信。

- 使用设备内置 [X.509 证书][lnk-x509]和私钥作为一种向 IoT 中心验证设备的方式。此身份验证方式可确保任何时候都无法在设备外部获知设备上的私钥，从而提供更高级别的安全性。

安全令牌方式通过将对称密钥与每个呼叫关联，为每个设备向 IoT 中心作出的呼叫提供身份验证。基于 X.509 的身份验证允许物理层 IoT 设备的身份验证作为 TLS 连接建立的一部分。基于安全令牌的方式可在没有 X.509 身份验证的情况下使用，但这种模式的安全性较低。这两种方式的选择主要取决于设备验证需要达到的安全程度和设备上安全储存的可用性（以安全存储私钥）。

## IoT 中心安全令牌

IoT 中心使用安全令牌对设备和服务进行身份验证，以避免在网络上发送密钥。并且安全令牌的有效期和范围有限。Azure IoT 中心 SDK 无需任何特殊配置即可自动生成令牌。但在某些情况下，需要用户生成并直接使用安全令牌。包括 AMQP、MQTT 或 HTTP 应用层协议的直接使用，以及令牌服务模式的实现。

可在以下文章中找到有关安全令牌结构及其用法的详细信息：

-   [安全令牌结构][lnk-security-tokens]
-   [将 SAS 令牌当做设备使用][lnk-sas-tokens]

每个 IoT 中心都有一个[设备标识注册表][lnk-identity-registry]，可用于在服务中创建各设备的资源（例如包含即时云到设备消息的队列），以及允许访问面向设备的终结点。IoT 中心标识注册表针对解决方案为设备标识和安全密钥提供安全存储。可将单个或一组设备标识添加到允许列表或方块列表，以便完全控制设备访问。以下文章提供有关设备标识注册表的结构和受支持操作的详细信息。

[IoT 中心支持 AMQP、MQTT 和 HTTPS 等协议][lnk-protocols]。每个协议使用 IoT 设备到 IoT 中心的安全令牌的方式不同：

- AMQP：基于 SASL PLAIN 和 AMQP 声明的安全性（若是中心级别令牌，则为 {policyName}@sas.root.{iothubName}；若是设备范围令牌，则为 {deviceId}）。

- MQTT：CONNECT 包将 {deviceId} 用作 {ClientId}，“用户名”字段中为 {IoThubhostname}/{deviceId}；在“密码”字段中为 SAS 令牌。

- HTTP：有效令牌位于授权请求标头中。

IoT 中心设备标识注册表可用于配置每个设备的安全凭据和访问控制。如果 IoT 解决方案已大幅投资于[自定义设备标识注册表和/或身份验证方案][lnk-custom-auth]中具有，则可通过创建令牌服务，将该解决方案集成到具有 IoT 中心的现有基础结构中。

### 基于 X.509 证书的设备身份验证

使用[基于设备的 X.509 证书][lnk-use-x509]及其关联的私钥和公钥允许在物理层进行其他身份验证。私钥安全存储在设备中，无法在设备外发现。X.509 证书包含有关设备的信息（例如设备 ID）以及其他组织详细信息。使用公钥生成证书签名。

高级设备预配流：

- 将标识符关联到物理设备 - 设备制造或调试过程中将设备标识和/或 X.509 证书关联到设备。

- 在 IoT 中心创建对应的标识条目 - IoT 中心设备注册表中的设备标识和关联的设备信息。

- 将 X.509 证书指纹安全存储在 IoT 中心设备注册表中。

### 设备上的根证书

与 IoT 中心建立安全的 TLS 连接时，IoT 设备使用作为设备 SDK 的一部分的根证书验证 IoT 中心。对于 C 客户端 SDK，该证书位于存储库根下的“\\c\\certs”文件夹中。虽然这些根证书长期有效，但仍可能过期或被撤销。如果无法更新设备上的证书，该设备随后可能无法连接到 IoT 中心（或任何其他云服务）。IoT 设备部署后提供更新根证书的方法可有效减轻此风险。

## 保护连接安全

使用传输层安全性 (TLS) 标准来保护 IoT 设备和 IoT 中心之间的 Internet 连接安全。Azure IoT 支持 [TLS 1.2][lnk-tls12]、TLS 1.1 和 TLS 1.0（按此顺序）。对 TLS 1.0 的支持仅为向后兼容性提供。推荐使用 TLS 1.2，因为它提供最佳的安全性。

Azure IoT 套件支持以下密码套件（按此顺序）。

| 密码套件 | Length |
|--------------|--------|
| TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA384 (0xc028) ECDH secp384r1 (eq.7680 位 RSA) FS | 256 |
| TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA256 (0xc027) ECDH secp256r1 (eq.3072 位 RSA) FS | 128 |
| TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA (0xc014) ECDH secp384r1 (eq.7680 位 RSA) FS | 256 |
| TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA (0xc013) ECDH secp256r1 (eq.3072 位 RSA) FS | 128 |
| TLS\_RSA\_WITH\_AES\_256\_GCM\_SHA384 (0x9d) | 256 |
| TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256 (0x9c) | 128 |
| TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA256 (0x3d) | 256 |
| TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA256 (0x3c) | 128 |
| TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA (0x35) | 256 |
| TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA (0x2f) | 128 |
| TLS\_RSA\_WITH\_3DES\_EDE\_CBC\_SHA (0xa) | 112 |

## 保护云的安全

Azure IoT 中心允许为每个安全密钥定义[访问控制策略][lnk-protocols]。它使用以下一组权限向每个 IoT 中心的终结点授予访问权限。权限可根据功能限制对 IoT 中心的访问。

- **RegistryRead**。授予对设备标识注册表的读取访问权限。有关详细信息，请参阅[设备标识注册表][lnk-identity-registry]。

- **RegistryReadWrite**。授予对设备标识注册表的读取和写入访问权限。有关详细信息，请参阅[设备标识注册表][lnk-identity-registry]。

- **ServiceConnect**。授予对面向云服务的通信和监视终结点的访问权限。例如，它授权后端云服务接收设备到云的消息、发送云到设备的消息，以及检索对应的传送确认。

- **DeviceConnect**。授予对面向设备的通信终结点的访问权限。例如，它授予发送设备到云的消息和接收云到设备的消息的权限。此权限由设备使用。

有两种方法可以使用[安全令牌][lnk-sas-tokens]来获取 IoT 中心的 **DeviceConnect** 权限：使用设备标识密钥，或者使用共享访问策略密钥。此外，必须注意的是，可从设备访问的所有功能都故意显示在前缀为 `/devices/{deviceId}` 的终结点上。

[服务组件使用共享访问策略只能生成安全令牌][lnk-service-tokens]，授予适当权限。

Azure IoT 中心和其他可能是解决方案的一部分的服务允许使用 Azure Active Directory 管理用户。

Azure IoT 中心引入的数据可供多种服务（例如 Azure 流分析、Blob 存储等）使用。这些服务允许管理访问权限。了解以下有关这些服务和可用选项的详细信息：

- [Azure DocumentDB][lnk-docdb]：适用于半结构化数据的可缩放且已完全编制索引的数据库服务，可管理预配的设备的元数据，例如，属性、配置和安全属性。DocumentDB 提供高性能和高吞吐量处理、架构不可知的数据索引，以及丰富的 SQL 查询接口。

- [Azure 流分析][lnk-asa]：云中处理的实时流允许快速开发和部署低成本分析解决方案，以便从设备、传感器、基础结构和应用程序实时获取深入了解。来自这种完全托管服务的数据可缩放为任何数量，同时保持高吞吐量、低延迟和复原能力。

- [Blob 存储][lnk-blob]：可靠且符合经济效益的云存储，适用于设备要发送到云的数据。

## 结束语

本文概述了使用 Azure IoT 来设计和部署 IoT 基础结构的实现级别的详细信息。将每个组件配置为安全状态是保护 IoT 总体基础结构安全的关键。Azure IoT 中可用的设计选择提供了一定程度的灵活性和选择性；但是，每个选择都可能具有安全隐患。推荐通过风险/成本评估对这些选择进行评估。

[img-overview]: ./media/iot-secure-your-deployment/overview.png

[lnk-security-tokens]: /documentation/articles/iot-hub-sas-tokens/#security-token-structure
[lnk-sas-tokens]: /documentation/articles/iot-hub-sas-tokens#use-sas-tokens-as-a-device
[lnk-identity-registry]: /documentation/articles/iot-hub-devguide#device-identity-registry
[lnk-protocols]: /documentation/articles/iot-hub-devguide#security
[lnk-custom-auth]: /documentation/articles/iot-hub-guidance#custom-device-authentication
[lnk-x509]: http://www.itu.int/rec/T-REC-X.509-201210-I/en
[lnk-use-x509]: /documentation/articles/iot-hub-sas-tokens
[lnk-tls12]: https://tools.ietf.org/html/rfc5246
[lnk-service-tokens]: /documentation/articles/iot-hub-sas-tokens#using-security-tokens-from-service-components
[lnk-docdb]: /services/documentdb/
[lnk-asa]: /services/stream-analytics/
[lnk-appservices]: /services/app-service/
[lnk-logicapps]: /services/app-service/logic/
[lnk-blob]: /services/storage/

<!---HONumber=Mooncake_0822_2016-->