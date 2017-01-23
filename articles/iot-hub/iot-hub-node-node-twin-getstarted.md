<properties
    pageTitle="Azure IoT 中心设备孪生入门 (Node) | Azure"
    description="如何使用 Azure IoT 中心设备孪生添加标记，然后使用 IoT 中心查询。使用 Azure IoT SDK for Node.js 实现模拟设备应用，并实现可添加标记和运行 IoT 中心查询的服务应用。"
    services="iot-hub"
    documentationcenter="node"
    author="fsautomata"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="314c88e4-cce1-441c-b75a-d2e08e39ae7d"
    ms.service="iot-hub"
    ms.devlang="node"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="09/13/2016"
    wacn.date="01/13/2017"
    ms.author="elioda" />  


# 设备孪生入门 \(Node\)

[AZURE.INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

在本教程结束时，你将拥有两个 Node.js 控制台应用：

* **AddTagsAndQuery.js**（Node.js 后端应用），用于添加标记和查询设备孪生。
* **TwinSimulatedDevice.js**，一个 Node.js 应用，它模拟使用早前创建的设备标识连接到 IoT 中心的设备，并报告其连接条件。

> [AZURE.NOTE]
[Azure IoT SDK][lnk-hub-sdks] 文章介绍了可用于构建设备和后端应用的 Azure IoT SDK。
> 
> 

完成本教程需具备以下条件：

+ Node.js 版本 0.10.x 或更高版本。

+ 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## 创建服务应用
在此部分，会创建一个 Node.js 控制台应用，将位置元数据添加到与 **myDeviceId** 关联的设备孪生。该应用随后会选择位于中国的设备来查询存储在 IoT 中心的设备孪生，然后查询报告手机网络连接的设备孪生。

1. 新建名为 **addtagsandqueryapp** 的空文件夹。在命令提示符下的 **addtagsandqueryapp** 文件夹中，使用以下命令创建新的 package.json 文件。接受所有默认值：
   
    
	npm init
    
2. 在命令提示符下的 **addtagsandqueryapp** 文件夹中，运行以下命令以安装 **azure-iothub** 包：
   
    
	npm install azure-iothub --save
    
3. 在 **addtagsandqueryapp** 文件夹中使用文本编辑器新建 **AddTagsAndQuery.js** 文件。
4. 将以下代码添加到 **AddTagsAndQuery.js** 文件，并将 **{iot hub connection string}** 占位符替换为创建中心时复制的 IoT 中心连接字符串：
   
        'use strict';
        var iothub = require('azure-iothub');
        var connectionString = '{iot hub connection string}';
        var registry = iothub.Registry.fromConnectionString(connectionString);
   
        registry.getTwin('myDeviceId', function(err, twin){
            if (err) {
                console.error(err.constructor.name + ': ' + err.message);
            } else {
                var patch = {
                    tags: {
                        location: {
                            region: 'CN',
                            plant: 'Redmond43'
                      }
                    }
                };
   
                twin.update(patch, function(err) {
                  if (err) {
                    console.error('Could not update twin: ' + err.constructor.name + ': ' + err.message);
                  } else {
                    console.log(twin.deviceId + ' twin updated successfully');
                    queryTwins();
                  }
                });
            }
        });
   
    **Registry** 对象会公开从服务与设备孪生进行交互所需的所有方法。前面的代码首先初始化 **Registry** 对象，然后检索 **myDeviceId** 的设备孪生，最后使用所需位置信息更新其标记。
   
    更新标记之后，它会调用 **queryTwins** 函数。
5. 在 **AddTagsAndQuery.js** 末尾添加以下代码即可实现 **queryTwins** 函数：
   
        var queryTwins = function() {
            var query = registry.createQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
            query.nextAsTwin(function(err, results) {
                if (err) {
                    console.error('Failed to fetch the results: ' + err.message);
                } else {
                    console.log("Devices in Redmond43: " + results.map(function(twin) {return twin.deviceId}).join(','));
                }
            });
   
            query = registry.createQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
            query.nextAsTwin(function(err, results) {
                if (err) {
                    console.error('Failed to fetch the results: ' + err.message);
                } else {
                    console.log("Devices in Redmond43 using cellular network: " + results.map(function(twin) {return twin.deviceId}).join(','));
                }
            });
        };
   
    前面的代码执行两个查询：第一个只选择位于 **Redmond43** 工厂中的设备的设备孪生，第二个会优化查询以只选择还通过手机网络连接的设备。
   
    请注意，前面的代码在创建 **query** 对象时会指定最大返回文档数。**query** 对象包含 **hasMoreResults** 布尔属性，可以用于多次调用 **nextAsTwin** 方法以检索所有结果。名为 **next** 的方法可用于不是设备孪生的结果（例如聚合查询的结果）。
6. 使用以下内容运行应用程序：
   
        node AddTagsAndQuery.js
   
    对于寻找位于 **Redmond43** 的所有设备的查询，应在结果中看到一个设备，而对于将结果限制为使用手机网络连接的设备的查询，不会看到任何设备。
   
    ![][1]  


在下一部分，用户创建的设备应用会报告连接信息并更改上一部分中查询的结果。

## 创建设备应用
在此部分，会创建一个 Node.js 控制台应用作为 **myDeviceId** 连接到中心，然后更新其设备孪生的报告属性，说明它是使用手机网络进行连接的。

> [AZURE.NOTE] 目前，只能从使用 MQTT 协议连接到 IoT 中心的设备访问设备孪生。有关如何转换现有设备应用以使用 MQTT 的说明，请参阅 [MQTT 支持][lnk-devguide-mqtt]一文。

1. 新建名为 **reportconnectivity** 的空文件夹。在 **reportconnectivity** 文件夹的命令提示符处，使用以下命令创建新的 package.json 文件。接受所有默认值：
   
    
	npm init
    
2. 在 **reportconnectivity** 文件夹的命令提示符处，运行以下命令来安装 **azure-iot-device** 和 **azure-iot-device-mqtt** 包：
   
    
	npm install azure-iot-device azure-iot-device-mqtt --save
    
3. 在 **reportconnectivity** 文件夹中使用文本编辑器新建 **ReportConnectivity.js** 文件。
4. 将以下代码添加到 **ReportConnectivity.js** 文件，并将 **{device connection string}** 占位符替换为创建 **myDeviceId** 设备标识时复制的设备连接字符串：
   
        'use strict';
        var Client = require('azure-iot-device').Client;
        var Protocol = require('azure-iot-device-mqtt').Mqtt;
   
        var connectionString = '{device connection string}';
        var client = Client.fromConnectionString(connectionString, Protocol);
   
        client.open(function(err) {
        if (err) {
            console.error('could not open IotHub client');
        }  else {
            console.log('client opened');
   
            client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                var patch = {
                    connectivity: {
                        type: 'cellular'
                    }
                };
   
                twin.properties.reported.update(patch, function(err) {
                    if (err) {
                        console.error('could not update twin');
                    } else {
                        console.log('twin state reported');
                        process.exit();
                    }
                });
            }
            });
        }
        });
   
    **Client** 对象会公开从设备与设备孪生进行交互所需的所有方法。前面的代码在初始化 **Client** 对象后会检索 **myDeviceId** 的设备孪生，并使用连接信息更新其报告属性。
5. 运行设备应用
   
        node ReportConnectivity.js
   
    此时会显示消息`twin state reported`。
6. 现在设备报告了其连接信息，应出现在两个查询中。返回到 **addtagsandqueryapp** 文件夹，然后再次运行查询：
   
        node AddTagsAndQuery.js
   
    这次 **myDeviceId** 应出现在两个查询结果中。
   
    ![][3]  


## 后续步骤
本教程中，在 Azure 门户预览中配置了新的 IoT 中心，然后在 IoT 中心的标识注册表中创建了设备标识。已从后端应用以标记形式添加了设备元数据，并编写了模拟的设备应用，用于报告设备孪生中的设备连接信息。还学习了如何使用类似 SQL 的 IoT 中心查询语言来查询此信息。

充分利用以下资源：

- 通过 [Get started with IoT Hub][lnk-iothub-getstarted]（IoT 中心入门）教程学习如何从设备发送遥测；
- 通过[使用所需属性配置设备][lnk-twin-how-to-configure]教程学习如何使用设备孪生的所需属性配置设备，
- 通过 [Use direct methods][lnk-methods-tutorial]（使用直接方法）教程学习如何以交互方式控制设备（例如如何从用户控制的应用打开风扇）。

<!-- images -->

[1]: ./media/iot-hub-node-node-twin-getstarted/service1.png
[3]: ./media/iot-hub-node-node-twin-getstarted/service2.png

<!-- links -->

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-free-trial]: /pricing/free-trial/

[lnk-d2c]: /documentation/articles/iot-hub-devguide-messaging/#device-to-cloud-messages
[lnk-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-identity]: /documentation/articles/iot-hub-devguide-identity-registry/

[lnk-iothub-getstarted]: /documentation/articles/iot-hub-node-node-getstarted/
[lnk-device-management]: /documentation/articles/iot-hub-node-node-device-management-get-started/
[lnk-gateway-SDK]: /documentation/articles/iot-hub-linux-gateway-sdk-get-started/
[lnk-connect-device]: /develop/iot/

[lnk-twin-how-to-configure]: /documentation/articles/iot-hub-node-node-twin-how-to-configure/
[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup/

[lnk-methods-tutorial]: /documentation/articles/iot-hub-node-node-direct-methods/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->