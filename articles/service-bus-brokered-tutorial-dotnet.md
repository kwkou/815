<properties 
   pageTitle="服务总线中转消息传送 .NET 教程 | Azure"
   description="中转消息传送 .NET 教程。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="tysonn" />
<tags 
   ms.service="service-bus"
   ms.date="09/14/2015"
   wacn.date="01/14/2016" />

# 服务总线中转消息传送 .NET 教程

Azure 服务总线提供两个综合性消息传送解决方案：一是通过在云中运行的集中“中继”服务，它支持各种不同的传输协议和 Web 服务标准（包括 SOAP、WS-* 和 REST）。客户端不需要与本地服务建立直接连接，也不需要了解服务所在的位置，并且本地服务无需在防火墙上打开任何入站端口。

第二个消息传送解决方案启用了“中转”消息传送功能。可将它们视为异步或分离式消息传送功能，支持使用服务总线消息传送基础结构的发布-订阅、临时分离和负载平衡方案。分离式通信具有很多优点；例如，客户端和服务器可以根据需要进行连接并以异步方式执行其操作。

本教程旨在提供有关队列的概述和实践经验，队列是服务总线中转消息传送的一个核心组件。完成本教程中的一系列主题后，你将获得一个应用程序，它能填充消息列表、创建队列和向队列发送消息。最后，该应用程序从队列接收消息并将其显示出来，然后清理其资源并退出。有关介绍如何构建使用“中继”消息传送功能的应用程序的相应教程，请参阅[服务总线中继消息传送教程](/documentation/articles/service-bus-relay-tutorial/)。

## 简介和先决条件

队列为一个或多个竞争使用方提供“先入先出 (FIFO)”消息传递方式。FIFO 表示接收方通常按照消息排队的临时顺序来接收并处理消息，并且每条消息将仅由一个消息使用方接收并处理。使用队列的主要优点是实现应用程序组件的*暂时分离*：换而言之，创建方和使用方无需同时发送和接收消息，因为消息被持久存储在队列中。相关的优点是*负载分级*，它允许创建方和使用方以不同速率发送和接收消息。

以下是开始本教程之前应遵循的一些管理步骤和前提步骤。首先是创建服务命名空间，并获取共享的访问签名 (SAS) 密钥。服务命名空间为每个通过服务总线公开的应用程序提供应用程序边界。创建服务命名空间时，系统将自动生成 SAS 密钥。服务命名空间与 SAS 密钥的组合提供了一个凭据，服务总线可用其验证应用程序访问权限。

### 创建服务命名空间并获取 SAS 密钥

1. 若要创建服务命名空间，请遵循[如何：创建或修改服务总线服务命名空间](https://msdn.microsoft.com/zh-cn/library/azure/hh690931.aspx)中概述的步骤。

1. 在 [Azure 经典门户][]的主窗口中，单击在上一步中创建的命名空间的名称。

3. 单击**“配置”**。

4. 在“共享访问签名生成器”部分中，记下与 **RootManagerSharedAccessKey** 策略关联的主密钥，或将其复制到剪贴板。你将在本教程的后面部分使用此值。

下一步是创建一个 Visual Studio 项目并编写两个帮助程序函数，用于将以逗号分隔的消息列表加载到强类型的 [BrokeredMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) .NET [List](https://msdn.microsoft.com/zh-cn/library/6sh2ey19.aspx) 对象。

### 创建 Visual Studio 项目

1. 在“开始”菜单中右键单击 Visual Studio，以便以管理员身份启动该程序，然后单击“以管理员身份运行”。

2. 创建新的控制台应用程序项目。单击“文件”菜单并选择“新建”，然后单击“项目”。在“新建项目”对话框中，选择“Visual C#”（如果不显示“Visual C#”，则在“其他语言”下方查看），单击“控制台应用程序”模板，然后将其命名为 **QueueSample**。使用默认“位置”。单击“确定”以创建该项目。

3. 使用 NuGet 包管理器将服务总线库添加到你的项目：
	1. 在解决方案资源管理器中，右键单击项目文件夹，然后单击“管理 NuGet 包”。
	2. 在“管理 Nuget 包”对话框中，在线搜索“服务总线”并单击“安装”。<br />
1. 在解决方案资源管理器中，双击 Program.cs 文件以在 Visual Studio 编辑器中将其打开。将命名空间名称从其默认名称 `QueueSample` 更改为 `Microsoft.ServiceBus.Samples`。

	```
	Microsoft.ServiceBus.Samples
	{
	    …
	```

2. 修改 `using` 语句，如以下代码中所示。

	```
	using System;
	using System.Collections.Generic;
	using System.Data;
	using System.IO;
	using System.Threading;
	using Microsoft.ServiceBus.Messaging;
	```

3. 创建一个名为 Data.csv 的文本文件，并将以下逗号分隔文本中的内容复制到其中。

	```
	IssueID,IssueTitle,CustomerID,CategoryID,SupportPackage,Priority,Severity,Resolved
	1,Package lost,1,1,Basic,5,1,FALSE
	2,Package damaged,1,1,Basic,5,1,FALSE
	3,Product defective,1,2,Premium,5,2,FALSE
	4,Product damaged,2,2,Premium,5,2,FALSE
	5,Package lost,2,2,Basic,5,2,TRUE
	6,Package lost,3,2,Basic,5,2,FALSE
	7,Package damaged,3,7,Premium,5,3,FALSE
	8,Product defective,3,2,Premium,5,3,FALSE
	9,Product damaged,4,6,Premium,5,3,TRUE
	10,Package lost,4,8,Basic,5,3,FALSE
	11,Package damaged,5,4,Basic,5,4,FALSE
	12,Product defective,5,4,Basic,5,4,FALSE
	13,Package lost,6,8,Basic,5,4,FALSE
	14,Package damaged,6,7,Premium,5,5,FALSE
	15,Product defective,6,2,Premium,5,5,FALSE
	```

	保存并关闭该 Data.csv 文件，并记住保存位置。

4. 在解决方案资源管理器中，右键单击项目的名称（此示例中为 **QueueSample**），并依次单击“添加”和“现有项”。

5. 浏览到你在步骤 6 中创建的 Data.csv 文件。单击该文件，然后单击“添加”。确保选择了文件类型列表中的“所有文件”(*.*)。

### 创建用于解析消息列表的函数

1. 在 `Main()` 方法之前，声明两个变量，用于包含 Data.csv 中的消息列表：其中之一为 **DataTable** 类型。另一个应为 List 对象类型，强类型化为 [BrokeredMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx)。后者是中转消息列表，本教程中的后续步骤将用到它。

	```
	namespace Microsoft.ServiceBus.Samples
	{
	    publicclass Program
	    {
	
	        privatestatic DataTable issues;
	        privatestatic List<BrokeredMessage> MessageList;
	```

2. 在 `Main()` 之外，定义 `ParseCSV()` 方法，用于解析 Data.csv 中的消息列表并将消息加载到 [DataTable](https://msdn.microsoft.com/zh-cn/library/azure/system.data.datatable.aspx) 表，如下所示。该方法将返回 **DataTable** 对象。

	```
	static DataTable ParseCSVFile()
	{
	    DataTable tableIssues = new DataTable("Issues");
	    string path = @"..\..\data.csv";
	    try
	    {
	        using (StreamReader readFile = new StreamReader(path))
	        {
	            string line;
	            string[] row;
	
	            // create the columns
	            line = readFile.ReadLine();
	            foreach (string columnTitle in line.Split(','))
	            {
	                tableIssues.Columns.Add(columnTitle);
	            }
	
	            while ((line = readFile.ReadLine()) != null)
	            {
	                row = line.Split(',');
	                tableIssues.Rows.Add(row);
	            }
	        }
	    }
	    catch (Exception e)
	    {
	        Console.WriteLine("Error:" + e.ToString());
	    }
	
	    return tableIssues;
	}
	```

3. 在 `Main()` 方法中，添加一条用于调用 `ParseCSVFile()` 方法的语句：

	```
	public static void Main(string[] args)
	{
	
	    // Populate test data
	    issues = ParseCSVFile();
	
	}
	```

### 创建用于加载消息列表的函数

1. 在 `Main()` 之外，定义 `GenerateMessages()` 方法，用于接收 `ParseCSVFile()` 返回的 **DataTable** 对象，并将该表加载到强类型化的中转消息列表中。该方法随后返回 **List** 对象，如下面的示例所示。 

	```
	static List<BrokeredMessage> GenerateMessages(DataTable issues)
	{
	    // Instantiate the brokered list object
	    List<BrokeredMessage> result = new List<BrokeredMessage>();
	
	    // Iterate through the table and create a brokered message for each row
	    foreach (DataRow item in issues.Rows)
	    {
	        BrokeredMessage message = new BrokeredMessage();
	        foreach (DataColumn property in issues.Columns)
	        {
	            message.Properties.Add(property.ColumnName, item[property]);
	        }
	        result.Add(message);
	    }
	    return result;
	}
	```

2. 在 `Main()` 中，在调用 `ParseCSVFile()` 后立即添加一条语句，从而以 `ParseCSVFile()` 的返回值作为参数调用 `GenerateMessages()` 方法：

	```
	public static void Main(string[] args)
	{
	
	    // Populate test data
	    issues = ParseCSVFile();
	    MessageList = GenerateMessages(issues);
	}
	```

### 获取用户凭据

1. 首先创建三个全局字符串变量，用于保存这些值。在以前的变量声明之后直接声明这些变量，例如：

	```
	namespace Microsoft.ServiceBus.Samples
	{
	    publicclass Program
	    {
	
	        privatestatic DataTable issues;
	        privatestatic List<BrokeredMessage> MessageList; 
	        // Add these variablesprivatestaticstring ServiceNamespace;
	        privatestaticstring sasKeyName = "RootManageSharedAccessKey";
	        privatestaticstring sasKeyValue;
	        …
	```

2. 接下来，创建一个函数，用于接受并存储服务命名空间和 SAS 密钥。在 `Main()` 之外添加此方法。例如：

	```
	static void CollectUserInput()
	{
	    // User service namespace
	    Console.Write("Please enter the service namespace to use: ");
	    ServiceNamespace = Console.ReadLine();
	
	    // Issuer key
	    Console.Write("Please enter the SAS key to use: ");
	    sasKeyValue = Console.ReadLine();
	}
	```

3. 在 `Main()` 中，在调用 `GenerateMessages()` 之下直接添加一条语句以调用 `CollectUserInput()` 方法：

	```
	public static void Main(string[] args)
	{
	
	    // Populate test data
	    issues = ParseCSVFile();
	    MessageList = GenerateMessages(issues);
	    
	    // Collect user input
	    CollectUserInput();
	}
	```

### 生成解决方案

在 Visual Studio 的“生成”菜单中，单击“生成解决方案”或按 F6 以确认到目前为止工作的准确性。

创建管理凭据

这是服务总线消息传送功能教程中的第二步。在此步骤中，你可以定义将用于创建共享访问签名 (SAS) 凭据（用于授权应用程序）的管理操作。

## 创建管理凭据

在此步骤中，你可以定义将用于创建共享访问签名 (SAS) 凭据（用于授权应用程序）的管理操作。

1. 为清楚起见，本教程将所有队列操作置于单独的方法中。在 `Program` 类中的 `Main()` 方法之下创建 `Queue()` 方法。例如：
 
	```
	public static void Main(string[] args)
	{
	…
	}
	staticvoid Queue()
	{
	}
	```

2. 下一步是使用 [TokenProvider](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.tokenprovider.aspx) 对象创建 SAS 凭据。此创建方法用于接受在 `CollectUserInput()` 方法中获取的 SAS 密钥名称和值。将以下代码添加到 `Queue()` 方法中：

	```
	staticvoid Queue()
	{
	    // Create management credentials
	    TokenProvider credentials = TokenProvider.CreateSharedAccessSignatureTokenProvider(sasKeyName,sasKeyValue);
	}
	```
### 创建命名空间管理器

1. 创建新的命名空间管理对象，以包含在在上一步中获得的命名空间名称和管理凭据的 URI 作为参数。直接在上一步中添加的代码之下添加以下代码：
	
	```
	NamespaceManager namespaceClient = new NamespaceManager(ServiceBusEnvironment.CreateServiceUri("sb", <namespaceName>, string.Empty), credentials);
	```

### 示例

此时，你的代码应如下所示：

```
using System;
using System.Collections.Generic;
using System.Data;
using System.IO;
using System.Threading;
using Microsoft.ServiceBus.Messaging;

namespace Microsoft.ServiceBus.Samples
{
  class Program
  {
    private static DataTable issues;
    private static List<BrokeredMessage> MessageList;
    private static string ServiceNamespace;
    private static string sasKeyName = "RootManageSharedAccessKey";
    private static string sasKeyValue;

    static void Main(string[] args)
    {
      // Populate test data
      issues = ParseCSVFile();
      MessageList = GenerateMessages(issues);

      // Collect user input
      CollectUserInput();

      // Add this call
      Queue();
    }

    static void Queue()
    {
      // Create management credentials
      TokenProvider credentials = TokenProvider.CreateSharedAccessSignatureTokenProvider(sasKeyName, sasKeyValue);
      NamespaceManager namespaceClient = new NamespaceManager(ServiceBusEnvironment.CreateServiceUri("sb", ServiceNamespace, string.Empty), credentials);
    }

    static DataTable ParseCSVFile()
    {
      DataTable tableIssues = new DataTable("Issues");
      string path = @"..\..\data.csv";
      try
      {
        using (StreamReader readFile = new StreamReader(path))
        {
          string line;
          string[] row;

          // create the columns
          line = readFile.ReadLine();
          foreach (string columnTitle in line.Split(','))
          {
            tableIssues.Columns.Add(columnTitle);
          }

          while ((line = readFile.ReadLine()) != null)
          {
            row = line.Split(',');
            tableIssues.Rows.Add(row);
          }
        }
      }
      catch (Exception e)
      {
        Console.WriteLine("Error:" + e.ToString());
      }

      return tableIssues;
    }

    static List<BrokeredMessage> GenerateMessages(DataTable issues)
    {
      // Instantiate the brokered list object
      List<BrokeredMessage> result = new List<BrokeredMessage>();

      // Iterate through the table and create a brokered message for each row
      foreach (DataRow item in issues.Rows)
      {
        BrokeredMessage message = new BrokeredMessage();
        foreach (DataColumn property in issues.Columns)
        {
          message.Properties.Add(property.ColumnName, item[property]);
        }
        result.Add(message);
      }
      return result;
    }

    static void CollectUserInput()
    {
      // User service namespace
      Console.Write("Please enter the service namespace to use: ");
      ServiceNamespace = Console.ReadLine();

      // Issuer key
      Console.Write("Please enter the issuer key to use: ");
      sasKeyValue = Console.ReadLine();
    }
  }
}
```

在下一步中，你可以创建要向其发送消息的队列。

## 将消息发送到队列

在此步骤中，你将创建一个队列，然后将中转消息列表中包含的消息发送到该队列。

### 创建队列并向队列发送消息

1. 首先创建队列。例如，将其命名为 `myQueue`，并在上一步中添加的管理操作后面直接声明它：

	```
	QueueDescription myQueue;
	myQueue = namespaceClient.CreateQueue("IssueTrackingQueue");
	```

2. 在 `Queue()` 方法中，使用新创建的服务总线 URI 作为参数创建一个消息工厂对象。在上一步中添加的管理操作后面直接添加以下代码：

	```
	MessagingFactory factory = MessagingFactory.Create(ServiceBusEnvironment.CreateServiceUri("sb", ServiceNamespace, string.Empty), credentials);
	```

3. 接下来，使用 [QueueClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.aspx) 类创建队列对象。在最后一步中添加的代码后直接添加以下代码：

	```
	QueueClient myQueueClient = factory.CreateQueueClient("IssueTrackingQueue");
	```

4. 然后添加以下代码，用于循环遍历你之前创建的中转消息列表，并将其中每条消息发送到队列。在上一步中的 `CreateQueueClient()` 声明后直接添加以下代码：
	
	```
	// Send messages
	Console.WriteLine("Now sending messages to the queue.");
	for (int count = 0; count < 6; count++)
	{
	    var issue = MessageList[count];
	    issue.Label = issue.Properties["IssueTitle"].ToString();
	    myQueueClient.Send(issue);
	    Console.WriteLine(string.Format("Message sent: {0}, {1}", issue.Label, issue.MessageId));
	}
	```

## 从队列接收消息

在此步骤中，你将从上一步中创建的队列获取消息列表。

### 创建接收器并从队列接收消息

在 `Queue()` 方法中，使用 [Microsoft.ServiceBus.Messaging.QueueClient.Receive](https://msdn.microsoft.com/zh-cn/library/azure/hh322678.aspx) 方法循环访问队列和接收消息，并将每条消息输出到控制台。在上一步中添加的代码之下直接添加以下代码：

	```
	Console.WriteLine("Now receiving messages from Queue.");
	BrokeredMessage message;
	while ((message = myQueueClient.Receive(new TimeSpan(hours: 0, minutes: 1, seconds: 5))) != null)
	    {
	        Console.WriteLine(string.Format("Message received: {0}, {1}, {2}", message.SequenceNumber, message.Label, message.MessageId));
	        message.Complete();
	
	        Console.WriteLine("Processing message (sleeping...)");
	        Thread.Sleep(1000);
	    }
	```

### 结束 `Queue()` 方法并清理资源

在前面的代码之下直接添加以下代码，以清理消息工厂和队列资源：

	```
	factory.Close();
	myQueueClient.Close();
	namespaceClient.DeleteQueue("IssueTrackingQueue");
	```

### 调用 `Queue()` 方法

最后一步是添加用于从 `Main()` 调用 `Queue()` 方法的代码。在 Main() 的末尾添加以下突出显示的行：
	
	```
	public static void Main(string[] args)
	{
	    // Collect user input
	    CollectUserInput();
	
	    // Populate test data
	    issues = ParseCSVFile();
	    MessageList = GenerateMessages(issues);
	
	    // Add this call
	    Queue();
	}
	```

### 示例

下面的代码包含完整的 **QueueSample** 应用程序。

```
using System;
using System.Collections.Generic;
using System.Data;
using System.IO;
using System.Threading;
using Microsoft.ServiceBus.Messaging;

namespace Microsoft.ServiceBus.Samples
{
  class Program
  {
    private static DataTable issues;
    private static List<BrokeredMessage> MessageList;
    private static string ServiceNamespace;
    private static string sasKeyName = "RootManageSharedAccessKey";
    private static string sasKeyValue;

    static void Main(string[] args)
    {
      // Populate test data
      issues = ParseCSVFile();
      MessageList = GenerateMessages(issues);

      // Collect user input
      CollectUserInput();

      // Add this call
      Queue();
    }

    static void Queue()
    {
      // Create management credentials
      TokenProvider credentials = TokenProvider.CreateSharedAccessSignatureTokenProvider(sasKeyName, sasKeyValue);
      NamespaceManager namespaceClient = new NamespaceManager(ServiceBusEnvironment.CreateServiceUri("sb", ServiceNamespace, string.Empty), credentials);

      QueueDescription myQueue;
      myQueue = namespaceClient.CreateQueue("IssueTrackingQueue");

      MessagingFactory factory = MessagingFactory.Create(ServiceBusEnvironment.CreateServiceUri("sb", ServiceNamespace, string.Empty), credentials);
      QueueClient myQueueClient = factory.CreateQueueClient("IssueTrackingQueue");

      // Send messages
      Console.WriteLine("Now sending messages to the Queue.");
      for (int count = 0; count < 6; count++)
      {
        var issue = MessageList[count];
        issue.Label = issue.Properties["IssueTitle"].ToString();
        myQueueClient.Send(issue);
        Console.WriteLine(string.Format("Message sent: {0}, {1}", issue.Label, issue.MessageId));
      }

      Console.WriteLine("Now receiving messages from Queue.");
      BrokeredMessage message;
      while ((message = myQueueClient.Receive(new TimeSpan(hours: 0, minutes: 1, seconds: 5))) != null)
      {
        Console.WriteLine(string.Format("Message received: {0}, {1}, {2}", message.SequenceNumber, message.Label, message.MessageId));
        message.Complete();

        Console.WriteLine("Processing message (sleeping...)");
        Thread.Sleep(1000);
      }

      factory.Close();
      myQueueClient.Close();
      namespaceClient.DeleteQueue("IssueTrackingQueue");
    }

    static DataTable ParseCSVFile()
    {
      DataTable tableIssues = new DataTable("Issues");
      string path = @"..\..\data.csv";
      try
      {
        using (StreamReader readFile = new StreamReader(path))
        {
          string line;
          string[] row;

          // create the columns
          line = readFile.ReadLine();
          foreach (string columnTitle in line.Split(','))
          {
            tableIssues.Columns.Add(columnTitle);
          }

          while ((line = readFile.ReadLine()) != null)
          {
            row = line.Split(',');
            tableIssues.Rows.Add(row);
          }
        }
      }
      catch (Exception e)
      {
        Console.WriteLine("Error:" + e.ToString());
      }

      return tableIssues;
    }

    static List<BrokeredMessage> GenerateMessages(DataTable issues)
    {
      // Instantiate the brokered list object
      List<BrokeredMessage> result = new List<BrokeredMessage>();

      // Iterate through the table and create a brokered message for each row
      foreach (DataRow item in issues.Rows)
      {
        BrokeredMessage message = new BrokeredMessage();
        foreach (DataColumn property in issues.Columns)
        {
          message.Properties.Add(property.ColumnName, item[property]);
        }
        result.Add(message);
      }
      return result;
    }

    static void CollectUserInput()
    {
      // User service namespace
      Console.Write("Please enter the service namespace to use: ");
      ServiceNamespace = Console.ReadLine();

      // Issuer key
      Console.Write("Please enter the issuer key to use: ");
      sasKeyValue = Console.ReadLine();
    }
  }
}
```

## 生成并运行 QueueSample 应用程序

完成上述步骤后，即可生成并运行 **QueueSample** 应用程序。

### 生成 QueueSample 应用程序

在 Visual Studio 中的“生成”菜单上，单击“生成解决方案”，或按 F6。如果遇到错误，请验证你的代码是否正确以上一步末尾提供的完整示例为基础。

### 运行 QueueSample 应用程序

1. 运行该应用程序之前，必须确保已创建服务命名空间并已获得 SAS 密钥，如[简介和先决条件](#introduction-and-prerequisites)中所述。

1. 打开浏览器并转到 [Azure 经典门户][]。

3. 单击左侧树中的“服务总线”。

4. 单击要使用的命名空间的名称。在页面底部，单击“连接信息”。记下包含 SAS 密钥的连接字符串或将其复制到剪贴板。

5. 在 Visual Studio 中的“调试”菜单中，单击“启动调试”，或按 F5。出现提示时，输入服务命名空间的名称，以及在上一步中获取的密钥。

## 后续步骤

本教程介绍了如何使用服务总线中转消息传送功能构建服务总线客户端应用程序和服务。有关使用服务总线[中继消息传送](/documentation/articles/service-bus-messaging-overview/#Relayed-messaging)的类似教程，请参阅[服务总线中继消息传送教程](/documentation/articles/service-bus-relay-tutorial/)。

若要了解有关[服务总线](/home/features/messaging)的详细信息，请参阅以下主题。

- [服务总线消息传送概述](/documentation/articles/service-bus-messaging-overview/)
- [服务总线基础知识](/documentation/articles/service-bus-fundamentals-hybrid-solutions/)
- [服务总线体系结构](/documentation/articles/service-bus-architecture/)

[Azure 经典门户]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_0104_2016-->