<properties
    pageTitle="Azure 事件中心存档演练 | Azure"
    description="此示例使用 Azure Python SDK 演示如何使用事件中心存档功能。"
    services="event-hubs"
    documentationcenter=""
    author="djrosanova"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="bdff820c-5b38-4054-a06a-d1de207f01f6"
    ms.service="event-hubs"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/13/2016"
    wacn.date="01/23/2017"
    ms.author="darosa;sethm" />

# 事件中心存档演练：Python
事件中心存档是事件中心的一项新功能，允许用户自动将事件中心中的流数据传送到所选的 Azure Blob 存储帐户。这样易于对实时流数据执行批处理操作。本文介绍如何通过 Python 使用事件中心存档功能。

此示例使用 Azure Python SDK 演示如何使用存档功能。Sender.py 程序以 JSON 格式将模拟的环境遥测数据发送到事件中心。事件中心已配置为使用存档功能将此数据成批地写入到 blob 存储。然后 archivereader.py 应用将读取这些 blob，并为每个设备创建一个附加文件，然后将数据写入到 .csv 文件中。

将要完成的任务

1.  使用 Azure 门户预览创建 Azure Blob 存储帐户及其中的 blob 容器

2.  使用 Azure 门户预览创建事件中心命名空间

3.  使用 Azure 门户预览创建启用了存档功能的事件中心

4.  使用 Python 脚本将数据发送到事件中心

5.  使用另一个 Python 脚本从存档中读取文件并处理这些文件

先决条件

1.  Python 2.7.x

2.  一个 Azure 订阅

[AZURE.INCLUDE [create-account-note](../../includes/create-account-note.md)]

## 创建 Azure 存储帐户

1. 登录到 [Azure 门户预览][Azure portal]。
2. 在门户的左侧导航窗格中，依次单击“新建”、“数据 + 存储”和“存储帐户”。
3. 填写“存储帐户”边栏选项卡中的字段，然后单击“创建”。
   
   ![][1]  

4. 看到“部署成功”消息后，单击新存储帐户的名称，然后在“概要”边栏选项卡中单击“Blob”。“Blob 服务”边栏选项卡打开时，单击顶部的“+ 容器”。将容器命名为“archive”，然后关闭“Blob 服务”边栏选项卡。
5. 单击左侧边栏选项卡中的“访问密钥”，复制存储帐户名称和 **key1** 的值。将这些值保存到记事本或其他临时位置。

[AZURE.INCLUDE [event-hubs-create-event-hub](../../includes/event-hubs-create-event-hub.md)]

## 创建用于将事件发送到事件中心的 Python 脚本
1. 打开常用的 Python 编辑器，如 [Visual Studio Code][Visual Studio Code]。

2.  创建名为 **sender.py** 的脚本。此脚本将向事件中心发送 200 个事件。它们是以 JSON 格式发送的简单环境读数。

3.  将以下代码粘贴到 sender.py 中：

	
    	import uuid
    	import datetime
    	import random
    	import json
    	from azure.servicebus import ServiceBusService
    	
    	sbs = ServiceBusService(service_namespace='INSERT YOUR NAMESPACE NAME', shared_access_key_name='RootManageSharedAccessKey', shared_access_key_value='INSERT YOUR KEY')
    	devices = []
    	for x in range(0, 10):
    	    devices.append(str(uuid.uuid4()))
    	
    	for y in range(0,20):
    	    for dev in devices:
    	        reading = {'id': dev, 'timestamp': str(datetime.datetime.utcnow()), 'uv': random.random(), 'temperature': random.randint(70, 100), 'humidity': random.randint(70, 100)}
    	        s = json.dumps(reading)
    	        sbs.send\_event('myhub', s)
    	    print y
	
4.  更新前面的代码，以使用在创建事件中心命名空间时获得的命名空间名称和密钥值。

## 创建用于读取存档文件的 Python 脚本

1.  填写边栏选项卡，然后单击“创建”。

2.  创建名为 **archivereader.py** 的脚本。此脚本将读取存档文件，并为每个设备创建一个文件，用于仅写入该设备的数据。

3.  将以下代码粘贴到 archivereader.py 中：

	
        import os
    	import string
    	import json
    	import avro.schema
    	from avro.datafile import DataFileReader, DataFileWriter
    	from avro.io import DatumReader, DatumWriter
    	from azure.storage.blob import BlockBlobService
    	
    	def processBlob(filename):
    	    reader = DataFileReader(open(filename, 'rb'), DatumReader())
    	    dict = {}
    	    for reading in reader:
    	        parsed\_json = json.loads(reading["Body"])
    	        if not 'id' in parsed\_json:
    	            return
    	        if not dict.has\_key(parsed\_json['id']):
    	        list = []
    	        dict[parsed\_json['id']] = list
    	    else:
    	        list = dict[parsed\_json['id']]
    	        list.append(parsed\_json)
    	    reader.close()
    	    for device in dict.keys():
    	        deviceFile = open(device + '.csv', "a")
    	        for r in dict[device]:
    	            deviceFile.write(", ".join([str(r[x]) for x in r.keys()])+'\\n')
    
    	def startProcessing(accountName, key, container):
    	    print 'Processor started using path: ' + os.getcwd()
    	    block\_blob\_service = BlockBlobService(account\_name=accountName, account\_key=key)
    	    generator = block\_blob\_service.list\_blobs(container)
    	    for blob in generator:
    	        if blob.properties.content\_length != 0:
    	            print('Downloaded a non empty blob: ' + blob.name)
    	            cleanName = string.replace(blob.name, '/', '\_')
    	            block\_blob\_service.get\_blob\_to\_path(container, blob.name, cleanName)
    	            processBlob(cleanName)
    	            os.remove(cleanName)
    	        block\_blob\_service.delete\_blob(container, blob.name)
    	startProcessing('YOUR STORAGE ACCOUNT NAME', 'YOUR KEY', 'archive')
    

4.  请务必在调用 `startProcessing` 时粘贴存储帐户名称和密钥的相应值。

## 运行脚本

1.  打开其路径中包含 Python 的命令提示符，然后运行以下命令，安装 Python 必备组件包：

	
        pip install azure-storage
    	pip install azure-servicebus
    	pip install avro
    
  
    如果使用的是早期版本的 Azure 存储或 Azure，可能需要使用 **-upgrade** 选项

    还可能需要运行以下命令（在大多数系统上不必要）：

    
        pip install cryptography
    

2.  将目录更改到保存 sender.py 和 archivereader.py 的位置，然后运行以下命令：

    
        start python sender.py
    
    
    这将启动一个新的 Python 进程，用于运行发送程序。

3. 现在等待几分钟让存档运行。然后，在原始命令窗口中键入以下命令：

    
        python archivereader.py
    

    此存档处理器使用本地目录下载存储帐户/容器中的所有 blob。它将处理任何不为空的内容，并将结果以 .csv 文件形式写入到本地目录。

## 后续步骤

访问以下链接可以了解有关事件中心的详细信息：

- [使用事件中心的完整示例应用程序][]
- [使用事件中心扩大事件处理][]示例
- [事件中心概述][]
 

[Azure portal]: https://portal.azure.cn/

[1]: ./media/event-hubs-archive-python/event-hubs-python1.png
[About Azure storage accounts]: /documentation/articles/storage-create-storage-account/
[Visual Studio Code]: https://code.visualstudio.com/
[事件中心概述]: /documentation/articles/event-hubs-overview/
[使用事件中心的完整示例应用程序]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[使用事件中心扩大事件处理]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3

<!---HONumber=Mooncake_0116_2017-->