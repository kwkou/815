<properties
    pageTitle="Node JS 如何使用 MQTT 协议调用“直接方法”"
    description="Node JS 如何使用 MQTT 协议调用“直接方法”"
    service=""
    resource="iot-hub"
    authors=""
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="IoT Hub, MQTT, NodeJS"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="iot-hub-aog"
    ms.date=""
    wacn.date="02/21/2017" />

# Node JS 如何使用 MQTT 协议调用“直接方法”

Azure IOT SDK 中封装了使用 MQTT 协议调用“直接方法”的用法，如果想直接使用 MQTT 协议自己调用 Azure IOT 来实现“直接方法”的调用，虽然官方文档也有介绍，但依照官方文档的步骤，依然无法调用，这是因为文档中讲解到的一些参数并不详细，具体可以参考以下文档和代码 :

使用 NodeJS 调用 MQTT 协议，需要依赖 [MQTT.JS](https://github.com/mqttjs/MQTT.js)。

首先建立 NodeJS 项目目录，然后安装 MQTT 依赖包 :

    npm install mqtt --save

其次，在代码中配置连接 IOT 字符串，连接并接受云端通过调用“直接方法”发送来的消息 :

    var mqtt = require('mqtt')
    var URL = require('url');
    var QueryString = require('querystring');

    var options = {
        cmd: 'connect',
        protocolId: 'MQTT',
        protocolVersion: 4,
        clean: false,
        clientId: 'kevindevice',
        rejectUnauthorized: true,
        username: 'xuhuaIOT.azure-devices.cn/kevindevice/api-version=2016-11-14',
        reconnectPeriod: 0, // Client will handle reconnection at the higher level
        password : new Buffer('SharedAccessSignature sr=xuhuaIOT.azure-devices.cn%2Fdevices%2Fkevindevice&sig=wU3c8uu6AqTKoKF0OAtBE3K7ZcAkhelhFux6vNYEiho%3D&se=1485411010')
    };
    var client = mqtt.connect('mqtts://xuhuaIOT.azure-devices.cn', options);

    client.on('connect', () => {
        client.subscribe('$iothub/methods/POST/#', { qos: 0 }, function (err) {
            if (err) {
                console.error("Direct method error: " + err.message);
            } else {
                console.log("Successfully subscribe to direct method");
            }
        });

    })

    client.on('message', (topic, message) => {
        console.log(topic);
        var methodMessage = parseMessage(topic, message);
        console.log('message' + methodMessage);

        client.publish('$iothub/methods/res/200/?$rid=' + methodMessage.requestId, JSON.stringify({"message":"reboot successfully"}), { qos: 0, retain: false }, function(err) {});
    })

    var handleRebootRequest = function(message) {
        console.log('Response to method \'' + message + '\' sent successfully.');
    }

    var parseMessage = function(topic, body) {
    var url, path, query;
    try {
        url = URL.parse(topic);
        path = url.path.split('/');
        query = QueryString.parse(url.query);
    }
    catch(err) {
        console.error(err)
        return undefined;
    }

    // if the topic has a querystring then 'path' will include it; so
    // we strip it out
    var lastPathComponent = path[path.length - 1];
    if(lastPathComponent.indexOf('?') !== -1) {
        path[path.length - 1] = lastPathComponent.substr(
        0, lastPathComponent.indexOf('?')
        );
    }

    if(path.length > 0 && path[0] === '$iothub') {
        var message = {};
        if(path.length > 1 && path[1].length > 0) {
        // create an object for the module; for example, $iothub/twin/...
        // would result in there being a message.twin object
        var mod = message[path[1]] = {};

        // parse the request ID if there is one
        if(!!(query.$rid)) {
            message.requestId = query.$rid;
        }

        // parse the other properties properties (excluding $rid)
        message.properties = query;
        delete message.properties.$rid;

        // save the body
        message.body = body;

        // parse the verb
        if(path.length > 2 && path[2].length > 0) {
            mod.verb = path[2];

            // This is a topic that looks like this:
            //  $iothub/methods/POST/{method name}?$rid={request id}&{serialized properties}
            // We parse the method name out.
            if(path.length > 3 && path[3].length > 0) {
            mod.methodName = path[3];
            } else {
            // The service published a message on a strange topic name. This is
            // probably a service bug. At any rate we don't know what to do with
            // this strange topic so we throw.
            throw new Error('Device method call\'s MQTT topic name does not include the method name.');
            }
        }
        }
        return message;
    }
    return undefined;
    }

本示例是设备接受 Azure IOTHub 云端调用“直接方法”发来的消息，并进行响应处理。

[AZURE.NOTE] IOT 的用户名必需是 `{iothubhostname}/{device_id}/ api-version={api-version}`, 官方文档中没有提及要记入 api-version，但实际测试下来，这里是必须要有的。

参考文档：

[Azure IoT 中心使用直接方法](/documentation/articles/iot-hub-node-node-direct-methods/)<br>
[Azure IoT 中心直接方法的介绍](/documentation/articles/iot-hub-devguide-direct-methods/)
