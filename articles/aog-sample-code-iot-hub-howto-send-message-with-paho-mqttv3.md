<properties
    pageTitle="如何使用 paho mqttv3 库发送消息到 Azure IoT 中心"
    description="如何使用 paho mqttv3 库发送消息到 Azure IoT 中心"
    service=""
    resource="iothub"
    authors="Allan Li"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Sample Code, IoT Hub, paho mqttv3, MQTT"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="sample-code-aog"
    ms.date=""
    wacn.date="04/18/2017" />

# 如何使用 paho mqttv3 库发送消息到 Azure IoT 中心

Azure IoT 中心支持多种协议，比如 AMQP， MQTT 等等，也支持多种语言，比如 Java， C 等等。这意味着，除了使用 Azure 官方提供的 IoT 中心 SDK 之外，还可以使用社区中一些常用的库来与 Azure IoT 中心进行交互。这篇文章将演示如何使用 paho mqttv3 库发送消息到 Azure IoT 中心，以及如何利用 property_bag 发送包含其他属性的消息。

## 添加引用

    <dependency>
        <groupId>org.eclipse.paho</groupId>
        <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
        <version>1.1.0</version>
    </dependency>
    <!-- for base64 decode and encode -->
    <dependency>
        <groupId>commons-codec</groupId>
        <artifactId>commons-codec</artifactId>
        <version>1.10</version>
    </dependency>

## 生成 SAS 令牌

SAS 令牌将作为基于 MQTT 协议连接 Azure IoT 中心时使用的密码。

    private static String generateSasToken(String serverUri, String key) throws UnsupportedEncodingException {
            String tokenFormat = "SharedAccessSignature sig=%s&se=%s&sr=%s";
            String scope = URLEncoder.encode(serverUri, StandardCharsets.UTF_8.toString());
            long expiryTime = System.currentTimeMillis() / 1000l + 600 + 1l;
            String signatureStr = generateSignature(scope, expiryTime, key);

            return String.format(tokenFormat, signatureStr, expiryTime, scope);
    }
    private static String generateSignature(String resrouceUri, long expiryTime, String deviceKey) throws UnsupportedEncodingException
    {
            byte[] rawSig = String.format("%s\n%s", resrouceUri, expiryTime).getBytes(StandardCharsets.UTF_8);
            byte[] decodedDeviceKey = Base64.decodeBase64(deviceKey.getBytes());
            String hmacSha256 = "HmacSHA256";
            SecretKeySpec secretKey = new SecretKeySpec(decodedDeviceKey, hmacSha256);
            byte[] encryptedSig = null;
            try
            {
                Mac hMacSha256 = Mac.getInstance(hmacSha256);
                hMacSha256.init(secretKey);
                encryptedSig = hMacSha256.doFinal(rawSig);
            }
            catch (NoSuchAlgorithmException e)
            {
                // should never happen, since the algorithm is hard-coded
            }
            catch (InvalidKeyException e)
            {
                // should never happen, since the algorithm is hard-coded
            }
            byte[] encryptedSigBase64 = Base64.encodeBase64(encryptedSig);
            String utf8Sig = new String(encryptedSigBase64, StandardCharsets.UTF_8);
            String signatureStr = URLEncoder.encode(utf8Sig, StandardCharsets.UTF_8.name());
            return signatureStr;
    }

## 连接 Azure IoT 中心

    String hostName = ""; // e.g. iothubdemo.azure-devices.cn
    String deviceId = "";
    String deviceKey = "";

    MqttAsyncClient client = new MqttAsyncClient(serverURI, deviceId, new MemoryPersistence());
    MqttConnectOptions connOpts = new MqttConnectOptions();
    connOpts.setUserName(hostName + "/" + deviceId);
    connOpts.setPassword(generateSasToken(hostName + "/devices/" + deviceId, deviceKey).toCharArray());
    connOpts.setMaxInflight(100);
    connOpts.setCleanSession(false);
    connOpts.setAutomaticReconnect(true);

    IMqttToken conToken = client.connect(connOpts);
    conToken.waitForCompletion();
    System.out.println("Connected to IOT Hub server " + hostName);

## 发送消息

    String msgStr = "message " + i;
    MqttMessage message = new MqttMessage(msgStr.getBytes());
    message.setQos(1);
    IMqttDeliveryToken token = client.publish(topic, message);
    // wait here will make everything serialization
    token.waitForCompletion();
    System.out.println("Sent message " + i);

## 利用 property_bag 发送包含其他属性的消息

### 添加 property_bag 到主题中

    // demo for topic name with property_bag: devices/{device_id}/messages/events/{property_bag}
    String topicProperty = URLEncoder.encode("testp", "UTF-8") + "=" + URLEncoder.encode("testv", "UTF-8");
    String topicWithProperty = "devices/" + deviceId + "/messages/events/" + topicProperty;

### 发送消息到带有 property_bag 的主题

    // demo for topic name with property_bag
    // in this way, all events will contains the property
    IMqttDeliveryToken delToken = client.publish(topicWithProperty, message);

## 资源链接

示例代码：[https://github.com/wacn/AOG-CodeSample/tree/master/IotHub/Java/nature-mqtt-sample/](https://github.com/wacn/AOG-CodeSample/tree/master/IotHub/Java/nature-mqtt-sample/)
