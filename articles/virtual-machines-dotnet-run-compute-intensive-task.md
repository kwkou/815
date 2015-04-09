<properties linkid="develop-net-tutorials-compute-intensive-task-on-a-virtual-machine" urlDisplayName="Compute Intensive .NET Task" pageTitle="虚拟机上需要进行大量计算的 .NET 任务 - Azure" metaKeywords="deploying compute .NET application, vm .NET application, Service Bus queue monitoring, remote monitoring" description="了解如何在 Azure 虚拟机上部署和运行需要进行大量计算的 .NET 应用以及如何使用服务总线队列远程监视进度。" metaCanonical="" services="virtual-machines" documentationCenter=".NET" title="How to run a compute-intensive task in .NET on an Azure virtual machine" authors="waltpo" solutions="" manager="keboyd" editor="mollybos" scriptId="" videoId="" />

# 如何在 Azure 虚拟机上的 .NET 中运行需要进行大量计算的任务

凭借 Azure，您可以使用虚拟机来处理需要进行大量计算的任务；例如，虚拟机可以处理任务并将结果传送给客户端计算机或移动应用程序。完成本指南后，您将了解如何创建虚拟机来运行需要进行大量计算的可由另一个 .NET 应用程序监视的 .NET 应用程序。

本教程假定您了解如何创建 .NET 控制台应用程序，而不了解 Azure。 

您将了解：

* 如何创建虚拟机。
* 如何远程登录到虚拟机。
* 如何创建服务总线命名空间。
* 如何创建 .NET 应用程序来执行需要进行大量计算的任务。
* 如何创建 .NET 应用程序来监视需要进行大量计算的任务的进度。
* 如何运行 .NET 应用程序。
* 如何停止 .NET 应用程序。

本教程会将旅行商问题用于需要进行大量计算的任务。以下是运行需要进行大量计算的任务的 .NET 应用程序示例：

![Traveling Salesman Problem solver][solver_output]

以下是监视需要进行大量计算的任务的 .NET 应用程序示例：

![Traveling Salesman Problem client][client_output]

[WACOM.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

<h2>创建虚拟机</h2>

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 单击"新建"****。
3. 单击"虚拟机"****。
4. 单击"快速创建"****。
5. 在"创建虚拟机"****屏幕中，为"DNS 名称"****输入一个值。
6. 从"映像"****下拉列表中，选择一个映像，例如"Windows Server 2012"****。
7. 在"用户名"****字段中为管理员输入一个名称。记住您下次要输入的此名称和密码，远程登录虚拟机时您将使用它们。
8. 在"新密码"****字段中输入一个密码，然后在"确认"****字段中重新输入一次。
9. 从"位置"****下拉列表中，选择虚拟机的数据中心位置。
10. 单击"创建虚拟机"****。即会开始创建虚拟机。您可以在管理门户的"虚拟机"****部分中监视状态。当其状态显示为"活动"****时，您可以登录该虚拟机。

<h2>远程登录到虚拟机</h2>

1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击"虚拟机"****。
3. 单击您要登录的虚拟机名称。
4. 单击"连接"****。
5. 根据需要响应提示以连接到虚拟机。提示需要管理员名称和密码时，请使用您创建虚拟机时提供的值。

<h2>如何创建服务总线命名空间</h2>

若要开始使用服务总线队列，必须先创建一个服务命名空间。服务命名空间提供用于对应用程序内服务总线资源进行寻址的范围容器。

创建服务命名空间：

1.  登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2.  在管理门户的左侧导航窗格中，单击"服务总线"****。
3.  在管理门户的下方窗格中，单击"创建"****。

    ![Create new service bus][create_service_bus]
4.  在"创建命名空间"****对话框中，输入命名空间名称。系统会立即检查该名称是否可用，因为该名称必须是唯一名称。

    ![Create a namespace dialog][create_namespace_dialog]
5.  在确保命名空间名称可用后，选择应该托管您的命名空间的区域（确保使用您托管虚拟机的相同区域）。
    <div class="dev-callout">
    <strong>重要说明</strong>
    <p>选取您用于或打算用于您的虚拟机的**同一区域**。这将为您提供最佳性能。</p>
    </div>
6. 如果您登录使用的帐户拥有多个 Azure 订阅，请选择要用于命名空间的订阅。（如果您登录使用的帐户只有一个订阅，则您不会看到包含订阅的下拉列表。）
7. 单击复选标记。系统现已创建您的服务命名空间并已将其启用。您可能需要等待几分钟，因为系统将为您的帐户配置资源。

	![Click create screenshot][click_create]

您创建的命名空间随后将显示在管理门户中，然后要花费一段时间来激活。请等到状态变为"活动"****后再继续下一步。

<h2>获取命名空间的默认管理凭据</h2>

为了在新的命名空间上执行管理操作（例如创建队列），您需要获取该命名空间的管理凭据。

1.  在左侧导航窗格中，单击"服务总线"****节点以显示可用命名空间列表：   
    ![Available namespaces screenshot][available_namespaces]
2.  从显示的列表中选择刚刚创建的命名空间：   
    ![Namespace list screenshot][namespace_list]
3. 单击"访问密钥"****。   
    ![Access key button][access_key_button]
4.  在对话框中，查找"默认颁发者"****和"默认密钥"****条目。记下这些值，因为您将在下面使用此信息来对命名空间执行操作。 

<h2>如何创建 .NET 应用程序来执行需要进行大量计算的任务</h2>

1. 在您的部署计算机（无需是您创建的虚拟机）上，下载 [Azure SDK for .NET](/zh-cn/develop/net/)。
2. 使用名为 **TSPSolver** 的项目创建 .NET 控制台应用程序。确保目标框架设置为用于 .**NET Framework 4**（而不是 **.NET Framework 4 Client Profile**）。可在创建项目后设置目标框架，具体方法如下：在 Visual Studio 的菜单中，依次单击"项目"****、"属性"****、"应用程序"****选项卡，然后设置"目标框架"****的值。
3. 加入 Microsoft ServiceBus 库。在 Visual Studio 解决方案资源管理器中，右键单击"TSPSolver"****、单击"添加引用"****、单击"浏览"****选项卡、浏览到 **C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK\2012-06\ref**，然后选择"Microsoft.ServiceBus.dll"****作为引用。
4. 加入 System Runtime Serialization 库。在 Visual Studio 解决方案资源管理器中，右键单击"TSPSolver"****、单击"添加引用"****、单击".NET"****选项卡，然后选择"System.Runtime.Serialization"****作为引用。
5. 将本节末尾的示例代码用于 **Program.cs** 的内容。
6. 将 **your\_service\_bus\_namespace**、**your\_service\_bus\_owner** 和 **your\_service\_bus\_key** 占位符分别修改为使用您的服务总线"命名空间"****、"默认颁发者"****和"默认密钥"****值。
7. 编译应用程序。这将在您的项目的 **bin** 文件夹中（**bin\release** 或 **bin\debug**，具体取决于您针对发布版本还是调试版本）创建 **TSPSolver.exe**。稍后您会将此可执行文件和 Microsoft.ServiceBus.dll 复制到您的虚拟机。

<p/>

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Text;
	using System.IO;
	
	using Microsoft.ServiceBus;
	using Microsoft.ServiceBus.Messaging;
	
	namespace TSPSolver
	{
	    class Program
	    {
	        // Value specifying how often to provide an update to the console.
	        private static long loopCheck = 100000000;
	        private static long nTimes = 0, nLoops = 0;
	
	        private static double[,] distances;
	        private static String[] cityNames;
	        private static int[] bestOrder;
	        private static double minDistance;
	
	        private static NamespaceManager namespaceManager;
	        private static QueueClient queueClient;
	        private static String queueName = "TSPQueue";
	
	        private static void BuildDistances(String fileLocation, int numCities)
	        {
	
	            try
	            {
	                StreamReader sr = new StreamReader(fileLocation);
	                String[] sep1 = { ", " };
	
	                double[,] cityLocs = new double[numCities, 2];
	
	                for (int i = 0; i < numCities; i++)
	                {
	                    String[] line = sr.ReadLine().Split(sep1, StringSplitOptions.None);
	                    cityNames[i] = line[0];
	                    cityLocs[i, 0] = Convert.ToDouble(line[1]);
	                    cityLocs[i, 1] = Convert.ToDouble(line[2]);
	                }
	                sr.Close();
	
	                for (int i = 0; i < numCities; i++)
	                {
	                    for (int j = i; j < numCities; j++)
	                    {
	                        distances[i, j] = hypot(Math.Abs(cityLocs[i, 0] - cityLocs[j, 0]), Math.Abs(cityLocs[i, 1] - cityLocs[j, 1]));
	                        distances[j, i] = distances[i, j];
	                    }
	                }
	            }
	            catch (Exception e)
	            {
	                throw e;
	            }
	        }
	
	        private static double hypot(double x, double y)
	        {
	            return Math.Sqrt(x * x + y * y);
	        }
	
	        private static void permutation(List<int> startCities, double distSoFar, List<int> restCities)
	        {
	            try
	            {
	
	                nTimes++;
	                if (nTimes == loopCheck)
	                {
	                    nLoops++;
	                    nTimes = 0;
	                    DateTime dateTime = DateTime.Now;
	                    Console.Write("Current time is {0}.", dateTime);
	                    Console.WriteLine(" Completed {0} iterations of size of {1}.", nLoops, loopCheck);
	                }
	
	                if ((restCities.Count == 1) && ((minDistance == -1) || (distSoFar + distances[restCities[0], startCities[0]] + distances[restCities[0], startCities[startCities.Count - 1]] < minDistance)))
	                {
	                    startCities.Add(restCities[0]);
	                    newBestDistance(startCities, distSoFar + distances[restCities[0], startCities[0]] + distances[restCities[0], startCities[startCities.Count - 2]]);
	                    startCities.Remove(startCities[startCities.Count - 1]);
	                }
	                else
	                {
	                    for (int i = 0; i < restCities.Count; i++)
	                    {
	                        startCities.Add(restCities[0]);
	                        restCities.Remove(restCities[0]);
	                        permutation(startCities, distSoFar + distances[startCities[startCities.Count - 1], startCities[startCities.Count - 2]], restCities);
	                        restCities.Add(startCities[startCities.Count - 1]);
	                        startCities.Remove(startCities[startCities.Count - 1]);
	                    }
	                }
	            }
	            catch (Exception e)
	            {
	                throw e;
	            }
	        }
	
	        private static void newBestDistance(List<int> cities, double distance)
	        {
	            try
	            {
	                minDistance = distance;
	                String cityList = "Shortest distance is " + minDistance + ", with route: ";
	
	                for (int i = 0; i < bestOrder.Length; i++)
	                {
	                    bestOrder[i] = cities[i];
	                    cityList += cityNames[bestOrder[i]];
	                    if (i != bestOrder.Length - 1)
	                        cityList += ", ";
	                }
	                Console.WriteLine(cityList);
	                queueClient.Send(new BrokeredMessage(cityList));
	            }
	            catch (Exception e)
	            {
	                throw e;
	            }
	        }
	
	        static void Main(string[] args)
	        {
	            try
	            {
	
	                String serviceBusNamespace = "your_service_bus_namespace";
	                String issuer = "your_service_bus_owner";
	                String key = "your_service_bus_key";
	
	                String connectionString = @"Endpoint=sb://" +
	                       serviceBusNamespace +
	                       @".servicebus.chinacloudapi.cn/;SharedSecretIssuer=" +
	                       issuer + @";SharedSecretValue=" + key;
	
	                int numCities = 10; // Use as the default, if no value is specified
	                // at the command line.
	                if (args.Count() != 0)
	                {
	
	                    if (args[0].ToLower().CompareTo("createqueue") == 0)
	                    {
	                        // No processing to occur other than creating the queue.
	                        namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);
	                        namespaceManager.CreateQueue(queueName);
	                        Console.WriteLine("Queue named {0} was created.", queueName);
	                        Environment.Exit(0);
	                    }
	
	                    if (args[0].ToLower().CompareTo("deletequeue") == 0)
	                    {
	                        // No processing to occur other than deleting the queue.
	                        namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);
	                        namespaceManager.DeleteQueue("TSPQueue");
	                        Console.WriteLine("Queue named {0} was deleted.", queueName);
	                        Environment.Exit(0);
	                    }
	
	                    // Neither creating or deleting a queue.
	                    // Assume the value passed in is the number of cities to solve.
	                    numCities = Convert.ToInt32(args[0]);
	                }
	
	                Console.WriteLine("Running for {0} cities.", numCities);
	
	                queueClient = QueueClient.CreateFromConnectionString(connectionString, "TSPQueue");
	
	                List<int> startCities = new List<int>();
	                List<int> restCities = new List<int>();
	
	                startCities.Add(0);
	                for (int i = 1; i < numCities; i++)
	                {
	                    restCities.Add(i);
	                }
	                distances = new double[numCities, numCities];
	                cityNames = new String[numCities];
	                BuildDistances(@"c:\tsp\cities.txt", numCities);
	                minDistance = -1;
	                bestOrder = new int[numCities];
	                permutation(startCities, 0, restCities);
	                Console.WriteLine("Final solution found!");
	                queueClient.Send(new BrokeredMessage("Complete"));
	
	                queueClient.Close();
	                Environment.Exit(0);

	            }
	            catch (ServerBusyException serverBusyException)
	            {
	                Console.WriteLine("ServerBusyException encountered");
	                Console.WriteLine(serverBusyException.Message);
	                Console.WriteLine(serverBusyException.StackTrace);
	                Environment.Exit(-1);
	            }
	            catch (ServerErrorException serverErrorException)
	            {
	                Console.WriteLine("ServerErrorException encountered");
	                Console.WriteLine(serverErrorException.Message);
	                Console.WriteLine(serverErrorException.StackTrace);
	                Environment.Exit(-1);
	            }
	            catch (Exception exception)
	            {
	                Console.WriteLine("Exception encountered");
	                Console.WriteLine(exception.Message);
	                Console.WriteLine(exception.StackTrace);
	                Environment.Exit(-1);
	            }
	        }
	    }
	}



<h2>如何创建 .NET 应用程序来监视需要进行大量计算的任务的进度</h2>

1. 在您的开发计算机上，使用 **TSPClient** 作为项目名称创建 .NET 控制台应用程序。确保目标框架设置为用于 .**NET Framework 4** 或更高版本（而不是 **.NET Framework 4 Client Profile**）。可在创建项目后设置目标框架，具体方法如下：在 Visual Studio 的菜单中，依次单击"项目"****、"属性"****、"应用程序"****选项卡，然后设置"目标框架"****的值。
2. 加入 Microsoft ServiceBus 库。在 Visual Studio 解决方案资源管理器中，右键单击"TSPSolver"****、单击"添加引用"****、单击"浏览"****选项卡、浏览到 Azure .NET SDK（例如在位置 **C:\Program Files\Microsoft SDKs\Azure\.NET SDK\v2.5\ToolsRef** 中），然后选择"Microsoft.ServiceBus.dll"****作为引用。
3. 加入 System Runtime Serialization 库。在 Visual Studio 解决方案资源管理器中，右键单击"TSPClient"****、单击"添加引用"****、单击".NET"****选项卡，然后选择"System.Runtime.Serialization"****作为引用。
4. 将本节末尾的示例代码用于 **Program.cs** 的内容。
5. 将 **your\_service\_bus\_namespace**、**your\_service\_bus\_owner** 和 **your\_service\_bus\_key** 占位符分别修改为使用您的服务总线"命名空间"****、"默认颁发者"****和"默认密钥"****值。
5. 编译应用程序。这将在您的项目的 **bin** 文件夹中（**bin\release** 或 **bin\debug**，具体取决于您针对发布版本还是调试版本）创建 **TSPClient.exe**。您可以从开发计算机上运行此代码，或将此可执行文件和 Microsoft.ServiceBus.dll 复制到将要运行该客户端应用程序的计算计上（不需要放置于您的虚拟机上）。

<p/>

	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Text;
	using System.IO;
	
	using Microsoft.ServiceBus;
	using Microsoft.ServiceBus.Messaging;
	using System.Threading; // For Thread.Sleep
	
	namespace TSPClient
	{
	    class Program
	    {
	
	        static void Main(string[] args)
	        {
	
	            try
	            {
	
	                Console.WriteLine("Starting at {0}", DateTime.Now);
	
	                String serviceBusNamespace = "your_service_bus_namespace";
	                String issuer = "your_service_bus_owner";
	                String key = "your_service_bus_key";
	
	                String connectionString = @"Endpoint=sb://" +
	                       serviceBusNamespace +
	                       @".servicebus.chinacloudapi.cn/;SharedSecretIssuer=" +
	                       issuer + @";SharedSecretValue=" + key;
	
	                QueueClient queueClient = QueueClient.CreateFromConnectionString(connectionString, "TSPQueue");
	
	                BrokeredMessage message;
	
	                int waitMinutes = 3;  // Use as the default, if no value
	                // is specified at command line.
	
	                if (0 != args.Length)
	                {
	                    waitMinutes = Convert.ToInt16(args[0]);
	                }
	
	                String waitString;
	                waitString = (waitMinutes == 1) ? "minute" : waitMinutes.ToString() + " minutes";
	
	                while (true)
	                {
	                    message = queueClient.Receive();
	
	                    if (message != null)
	                    {
	                        try
	                        {
	                            string str = message.GetBody<string>();
	                            Console.WriteLine(str);
	
	                            // Remove message from queue
	                            message.Complete();
	
	                            if ("Complete" == str)
	                            {
	                                Console.WriteLine("Finished at {0}.", DateTime.Now);
	                                break;
	                            }
	                        }
	                        catch (Exception e)
	                        {
	                            // Indicates a problem. Unlock the message in the queue.
	                            message.Abandon();
	                            throw e;
	                        }
	                    }
	                    else
	                    {
	                        // The queue is empty.
	                        Console.WriteLine("Queue is empty. Sleeping for another {0}.", waitString);
	                        System.Threading.Thread.Sleep(60000 * waitMinutes);
	                    }
	                }
	                queueClient.Close();
	                Environment.Exit(0);
	            }
	            catch (ServerBusyException serverBusyException)
	            {
	                Console.WriteLine("ServerBusyException encountered");
	                Console.WriteLine(serverBusyException.Message);
	                Console.WriteLine(serverBusyException.StackTrace);
	                Environment.Exit(-1);
	            }
	            catch (ServerErrorException serverErrorException)
	            {
	                Console.WriteLine("ServerErrorException encountered");
	                Console.WriteLine(serverErrorException.Message);
	                Console.WriteLine(serverErrorException.StackTrace);
	                Environment.Exit(-1);
	            }
	            catch (Exception exception)
	            {
	                Console.WriteLine("Exception encountered");
	                Console.WriteLine(exception.Message);
	                Console.WriteLine(exception.StackTrace);
	                Environment.Exit(-1);
	            }
	        }
	    }
	}

<h2>如何运行 .NET 应用程序</h2>
运行需要进行大量计算的应用程序，首先创建队列，然后解决旅行商问题，这样会将当前最佳路线添加到服务总线队列。需要进行大量计算的应用程序正在运行时（或运行后），运行客户端可显示来自服务总线队列的结果。

<h3>如何运行需要进行大量计算的应用程序</h3>

1. 登录虚拟机。
2. 创建名为 **c:\TSP** 的文件夹。这是您将要运行应用程序的位置。
3. 将 TSPSolver.exe 和 Microsoft.ServiceBus.dll（您的 TSPSolver 项目的 **bin** 文件夹中提供这两者）复制到 **c:\TSP**。
4. 使用以下内容创建名为 **c:\TSP\cities.txt** 的文件：

		City_1, 1002.81, -1841.35
		City_2, -953.55, -229.6
		City_3, -1363.11, -1027.72
		City_4, -1884.47, -1616.16
		City_5, 1603.08, -1030.03
		City_6, -1555.58, 218.58
		City_7, 578.8, -12.87
		City_8, 1350.76, 77.79
		City_9, 293.36, -1820.01
		City_10, 1883.14, 1637.28
		City_11, -1271.41, -1670.5
		City_12, 1475.99, 225.35
		City_13, 1250.78, 379.98
		City_14, 1305.77, 569.75
		City_15, 230.77, 231.58
		City_16, -822.63, -544.68
		City_17, -817.54, -81.92
		City_18, 303.99, -1823.43
		City_19, 239.95, 1007.91
		City_20, -1302.92, 150.39
		City_21, -116.11, 1933.01
		City_22, 382.64, 835.09
		City_23, -580.28, 1040.04
		City_24, 205.55, -264.23
		City_25, -238.81, -576.48
		City_26, -1722.9, -909.65
		City_27, 445.22, 1427.28
		City_28, 513.17, 1828.72
		City_29, 1750.68, -1668.1
		City_30, 1705.09, -309.35
		City_31, -167.34, 1003.76
		City_32, -1162.85, -1674.33
		City_33, 1490.32, 821.04
		City_34, 1208.32, 1523.3
		City_35, 18.04, 1857.11
		City_36, 1852.46, 1647.75
		City_37, -167.44, -336.39
		City_38, 115.4, 0.2
		City_39, -66.96, 917.73
		City_40, 915.96, 474.1
		City_41, 140.03, 725.22
		City_42, -1582.68, 1608.88
		City_43, -567.51, 1253.83
		City_44, 1956.36, 830.92
		City_45, -233.38, 909.93
		City_46, -1750.45, 1940.76
		City_47, 405.81, 421.84
		City_48, 363.68, 768.21
		City_49, -120.3, -463.13
		City_50, 588.51, 679.33
	
5. 在命令提示符处，将目录更改为 c:\TSP。
6. 您将需要在运行 TSP 解算器排列之前创建服务总线队列。运行以下命令以创建服务总线队列：

        TSPSolver createqueue

7. 在创建该队列后，您可以运行 TSP 解算器排列。例如，运行以下命令可对 8 个城市运行解算器。 

        TSPSolver 8

 如果您没有指定数字，则解算器将对 10 个城市运行。在解算器找到当前最短的路线后，它会将这些路线添加到该队列中。

解算器会一直运行，直到检测完所有路线为止。

> [WACOM.NOTE]
> 您指定的数字越大，解算器运行的时间就越长。例如，对 14 个城市运行可能需要几分钟时间，而对 15 个城市运行可能需要几小时时间。增加到 16 个或更多个城市可能需要数天的运行时间（最终数周、数月和数年时间）。这是因为，随着城市数量的增加，解算器评估的排列数会迅速增加。
 
<h3>如何运行监视客户端应用程序</h3>
1. 登录到您要在其中运行客户端应用程序的计算机。它无需是（但也可以是）运行 **TSPSolver** 应用程序的同一计算机。
2. 创建一个您要在其中运行应用程序的文件夹。例如，**c:\TSP**。
3. 将 **TSPClient.exe** 和 Microsoft.ServiceBus.dll（两者都在您的 TSPClient 项目的 **bin** 文件夹中）复制到 c:\TSP 文件夹。
4. 在命令提示符处，将目录更改为 c:\TSP。
5. 运行以下命令：

        TSPClient

    或者，也可以通过传入命令行参数来指定检查队列之间休眠的分钟数。检查队列的默认休眠期是 3 分钟，如果没有命令行参数传入 **TSPClient**，则会使用该默认值。如果您要使用不同的休眠时间间隔值（如一分钟），请运行：

	    TSPClient 1

    客户端会一直运行，直到它看到"完成"的队列消息为止。请注意，如果您多次运行解算器而没有运行客户端，则您可能需要多次运行客户端才能完全清空队列。或者，您可以删除该队列，然后重新创建一个。若要删除该队列，请运行以下 **TSPSolver**（不是 **TSPClient**）命令：

        TSPSolver deletequeue

<h2>如何停止 .NET 应用程序</h2>

对于解算器和客户端应用程序，若要在正常完成之前结束，可以按 **Ctrl+C** 退出。

<h2>使用 TSPSolver 创建或删除队列的替代方法</h2>
您可以使用 [Azure 管理门户](https://manage.windowsazure.cn)创建或删除队列，而不是使用 TSPSolver 创建或删除队列。访问管理门户的服务总线部分即可访问用于创建或删除队列以及检索连接字符串、颁发者和访问密钥的用户界面。您也可以查看服务总线队列的仪表板，从而使您可以查看传入消息和传出消息的指标。 

[solver_output]: ./media/virtual-machines-dotnet-run-compute-intensive-task/WA_dotNetTSPSolver.png
[client_output]: ./media/virtual-machines-dotnet-run-compute-intensive-task/WA_dotNetTSPClient.png
[create_service_bus]: ./media/virtual-machines-dotnet-run-compute-intensive-task/ServiceBusCreateNew.png
[create_namespace_dialog]: ./media/virtual-machines-dotnet-run-compute-intensive-task/CreateNameSpaceDialog.png
[available_namespaces]: ./media/virtual-machines-dotnet-run-compute-intensive-task/AvailableNamespaces.png
[click_create]: ./media/virtual-machines-dotnet-run-compute-intensive-task/ClickCreate.png
[namespace_list]: ./media/virtual-machines-dotnet-run-compute-intensive-task/NamespaceList.png
[access_key_button]: ./media/virtual-machines-dotnet-run-compute-intensive-task/AccessKey.png
