## 创建设备标识

在本部分中，你将创建一个 Node.js 控制台应用程序，用于在 IoT 中心的标识注册表中创建新的设备标识。设备无法连接到 IoT 中心，除非它在设备标识注册表中具有条目。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]的“设备识别注册表”部分。当你运行此控制台应用程序时，它将生成唯一的设备 ID 和密钥，当设备向 IoT 中心发送设备到云的消息时，可以用于标识设备本身。

1. 新建名为 **createdeviceidentity** 的空文件夹。在 createdeviceidentity 文件夹中，于命令提示符下使用以下命令创建新的 package.json 文件。接受所有默认值：

    ```
    npm init
    ```

2. 在 createdeviceidentity 文件夹中的命令提示符下，运行以下命令安装 azure iothub 包：

    ```
    npm install azure-iothub --save
    ```

3. 使用文本编辑器在 createdeviceidentity 文件夹中创建一个新的 CreateDeviceIdentity.js 文件。

4. 在 CreateDeviceIdentity.js 文件的开头添加以下 `require` 语句：

    ```
    'use strict';
    
    var iothub = require('azure-iothub');
    ```

5. 将以下代码添加到 CreateDeviceIdentity.js 文件，并将占位符值替换为在上一部分中为 IoT 中心创建的连接字符串：

    ```
    var connectionString = '{iothub connection string}';
    
    var registry = iothub.Registry.fromConnectionString(connectionString);
    ```

6. 添加以下代码，以便在 IoT 中心的设备标识注册表中创建新的设备定义。如果该设备 ID 在注册表中不存在，此代码将创建一个新的设备，否则将返回现有设备的密钥：

    ```
    var device = new iothub.Device(null);
    device.deviceId = 'myFirstDevice';
    registry.create(device, function(err, deviceInfo, res) {
      if (err) {
        registry.get(device.deviceId, printDeviceInfo);
      }
      if (deviceInfo) {
        printDeviceInfo(err, deviceInfo, res)
      }
    });

    function printDeviceInfo(err, deviceInfo, res) {
      if (deviceInfo) {
        console.log('Device id: ' + deviceInfo.deviceId);
        console.log('Device key: ' + deviceInfo.authentication.SymmetricKey.primaryKey);
      }
    }
    ```

7. 保存并关闭 CreateDeviceIdentity.js 文件。

8. 若要运行 create-device-identity应用程序，请在命令提示符下于 create-device-identity 文件夹中执行以下命令：

    ```
    node CreateDeviceIdentity.js 
    ```

9. 记下设备 ID 和设备密钥。稍后在创建连接到作为设备的 IoT 中心的应用程序时需要这些数据。

> [AZURE.NOTE] IoT 中心标识注册表只存储设备标识，以启用对中心的安全访问。它存储设备 ID 和密钥作为安全凭据，以及启用或禁用标志让你禁用对单个设备的访问。如果应用程序需要存储其他设备特定的元数据，则应使用应用程序特定的存储。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]。

## 接收设备到云的消息

在本部分中，你将创建一个 Node.js 控制台应用程序，用于读取来自 IoT 中心的设备到云消息。IoT 中心公开与[事件中心][lnk-event-hubs-overview]兼容的终结点，以让你读取设备到云的消息。为了简单起见，本教程创建的基本读取器不适用于高吞吐量部署。[处理设备到云的消息][lnk-processd2c-tutorial]教程说明了如何大规模处理设备到云的消息。[事件中心入门][lnk-eventhubs-tutorial]教程更详细说明了如何处理来自事件中心的消息，此教程也适用于 IoT 中心事件中心兼容的终结点。

> [AZURE.NOTE] 读取设备到云消息的事件中心兼容终结点始终使用 AMQPS 协议。

1. 新建名为 **readdevicetocloudmessages** 的空文件夹。在 **readdevicetocloudmessages** 文件夹中，于命令提示符下使用以下命令创建新的 package.json 文件。接受所有默认值：

    ```
    npm init
    ```

2. 在 **readdevicetocloudmessages** 文件夹中的命令提示符下，运行以下命令以安装 **amqp10** 和 **bluebird** 包：

    ```
    npm install amqp10 bluebird --save
    ```

3. 使用文本编辑器在 **readdevicetocloudmessages** 文件夹中创建一个新的 **ReadDeviceToCloudMessages.js** 文件。

4. 在 **ReadDeviceToCloudMessages.js** 文件的开头添加以下 `require` 语句：

    ```
    'use strict';

    var AMQPClient = require('amqp10').Client;
    var Policy = require('amqp10').Policy;
    var translator = require('amqp10').translator;
    var Promise = require('bluebird');
    ```

5. 添加以下变量声明，并将占位符替换为前面记下的值。**{your event hub-compatible namespace}** 占位符的值来自门户中的“事件中心兼容的终结点”字段，其格式为 **xxxxnamespace.servicebus.chinacloudapi.cn**（不需要 **sb://* 前缀）。

    ```
    var protocol = 'amqps';
    var eventHubHost = '{your event hub-compatible namespace}';
    var sasName = 'iothubowner';
    var sasKey = '{your iot hub key}';
    var eventHubName = '{your event hub-compatible name}';
    var numPartitions = 2;
    ```

    > [AZURE.NOTE] 此代码假设已在 F1（免费）层创建 IoT 中心。免费 IoT 中心有“0”和“1”这两个分区。如果使用另一种定价层创建 IoT 中心，则应调整代码，为每个分区创建 **MessageReceiver**。

6. 添加以下筛选器定义。在创建开始运行后只读取发送到 IoT 中心的消息的接收方时，此应用程序将使用筛选器。这很适合测试环境，因为这样可以看到当前的消息集，但在生产环境中，代码应该要确保它能处理所有消息。有关详细信息，请参阅[如何处理 IoT 中心设备到云的消息][lnk-processd2c-tutorial]教程。

    ```
    var filterOffset = new Date().getTime();
    var filterOption;
    if (filterOffset) {
      filterOption = {
      attach: { source: { filter: {
      'apache.org:selector-filter:string': translator(
        ['described', ['symbol', 'apache.org:selector-filter:string'], ['string', "amqp.annotation.x-opt-enqueuedtimeutc > " + filterOffset + ""]])
        } } }
      };
    }
    ```

7. 添加以下代码以创建接收地址和 AMQP 客户端：

    ```
    var uri = protocol + '://' + encodeURIComponent(sasName) + ':' + encodeURIComponent(sasKey) + '@' + eventHubHost;
    var recvAddr = eventHubName + '/ConsumerGroups/$default/Partitions/';
    
    var client = new AMQPClient(Policy.EventHub);
    ```

8. 添加以下两个函数用于在控制台中列显输出：

    ```
    var messageHandler = function (partitionId, message) {
      console.log('Received(' + partitionId + '): ', message.body);
    };
    
    var errorHandler = function(partitionId, err) {
      console.warn('** Receive error: ', err);
    };
    ```

9. 添加以下函数，充当使用筛选器的给定分区的接收方：

    ```
    var createPartitionReceiver = function(partitionId, receiveAddress, filterOption) {
      return client.createReceiver(receiveAddress, filterOption)
        .then(function (receiver) {
          console.log('Listening on partition: ' + partitionId);
          receiver.on('message', messageHandler.bind(null, partitionId));
          receiver.on('errorReceived', errorHandler.bind(null, partitionId));
        });
    };
    ```

10. 添加以下代码，以连接到事件中心兼容的终结点并启动接收方：

    ```
    client.connect(uri)
      .then(function () {
        var partitions = [];
        for (var i = 0; i < numPartitions; ++i) {
          partitions.push(createPartitionReceiver(i, recvAddr + i, filterOption));
        }
        return Promise.all(partitions);
    })
    .error(function (e) {
        console.warn('Connection error: ', e);
    });
    ```

11. 保存并关闭 **ReadDeviceToCloudMessages.js** 文件。

<!-- Links -->

[lnk-eventhubs-tutorial]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[lnk-devguide-identity]: /documentation/articles/iot-hub-devguide/#identityregistry
[lnk-event-hubs-overview]: /documentation/articles/event-hubs-overview/
[lnk-processd2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/

<!---HONumber=Mooncake_0321_2016-->