<properties linkid="dev-java-compute-load" urlDisplayName="TSP on Virtual Machine" pageTitle="VM 上需要进行大量计算的 Java 应用程序 &ndash; Azure" metaKeywords="Azure virtual machine Java, Azure Java app, Azure Java application" description="了解如何创建运行可由其他 Java 应用程序监视的、需要进行大量计算的 Java 应用程序的 Azure 虚拟机。" metaCanonical="" services="virtual-machines" documentationCenter="Java" title="如何在虚拟机上通过 Java 运行需要进行大量计算的任务" authors="robmcm" videoId="" scriptId="" solutions="" manager="wpickett" editor="mollybos" />

# 如何在虚拟机上通过 Java 运行需要进行大量计算的任务

借助 Azure，您可以使用虚拟机来处理需要进行大量计算的任务。例如，虚拟机可以处理任务并将结果传送给客户端计算机或移动应用程序。完成本指南后，您将了解如何创建运行可由其他 Java 应用程序监视的、需要进行大量计算的 Java 应用程序的虚拟机。

本教程假定您知道如何创建 Java 控制台应用程序、将库导入 Java 应用程序以及生成 Java 存档 (JAR)。而不了解 Azure。

您将了解到以下内容：

-   如何创建已安装 JDK 的虚拟机。
-   如何远程登录到虚拟机。
-   如何创建 Service Bus 命名空间。
-   如何创建 Java 应用程序来执行需要进行大量计算的任务。
-   如何创建 Java 应用程序来监视需要进行大量计算的任务的进度。
-   如何运行 Java 应用程序。
-   如何停止 Java 应用程序。

本教程会将旅行商问题用于需要进行大量计算的任务。下面是运行需要进行大量计算的任务的 Java 应用程序的示例：

![旅行商问题解算器][旅行商问题解算器]

以下是监视需要进行大量计算的任务的 Java 应用程序的示例：

![旅行商问题客户端][旅行商问题客户端]

[WACOM.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

## 创建虚拟机

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  依次单击“新建”、“计算”、“虚拟机”和“从库中”。
3.  在“虚拟机映像选择”对话框中，选择 **JDK 7 Windows Server 2012**。
    请注意，万一您安装的是还不能在 JDK 7 中运行的旧应用程序，可选择 **JDK 6 Windows Server 2012**。
4.  单击“下一步”。
5.  在“虚拟机配置”对话框中：

    1.  指定虚拟机的名称。
    2.  指定要用于虚拟机的大小。
    3.  在“用户名”字段中输入管理员的名称。记住您下次要输入的此名称和密码，远程登录虚拟机时您将使用它们。
    4.  在“新密码”字段中输入密码，然后在“确认”字段中重新输入一次。这是 Administrator 帐户密码。
    5.  单击“下一步”。

6.  在下一个“虚拟机配置”对话框中：

    1.  对于“云服务”，使用默认的“创建新云服务”。
    2.  “云服务 DNS 名称”的值在 cloudapp.net 中必须是唯一的。如有必要，请修改此值，直至 Azure 指出它是唯一的值。
    3.  指定区域、地缘组或虚拟网络。在本教程中，请指定区域，如“美国西部”。
    4.  对于“存储帐户”框，选择“使用自动生成的存储帐户”。
    5.  对于“可用性集”，请选择“(无)”。
    6.  单击“下一步”。

7.  在最后一个“虚拟机配置”对话框中：

    1.  接受默认的终结点项。
    2.  单击“完成”。

## 远程登录到虚拟机

1.  登录到[“管理门户”][Azure 管理门户]。
2.  单击“虚拟机”。
3.  单击您要登录的虚拟机名称。
4.  单击“连接”。
5.  根据需要响应提示以连接到虚拟机。提示需要管理员名称和密码时，请使用您创建虚拟机时提供的值。

请注意，Azure Service Bus 功能需要将 Baltimore CyberTrust 根证书作为您的 JRE 的 **cacerts** 存储的一部分安装。此证书将自动包含在本教程使用的 JRE 中。如果您的 JRE **cacerts** 存储中没有此证书，请参阅[将证书添加到 Java CA 证书存储][将证书添加到 Java CA 证书存储]，以获取有关添加该证书的信息（以及有关在您的 cacerts 存储中查看证书的信息）。

## 如何创建 Service Bus 命名空间

若要开始在 Azure 中使用 Service Bus 队列，必须先
创建一个服务命名空间。服务命名空间提供了用于对应用程序
中的 Service Bus 资源进行寻址的范围容器。

创建服务命名空间：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在管理门户的左下方导航窗格中，单击“Service Bus、Access Control 和 Caching”。
3.  在管理门户的左上方窗格中，单击“Service
    Bus”节点，然后单击“新建”按钮。
    ![Service Bus 节点屏幕快照][Service Bus 节点屏幕快照]
4.  在“新建服务命名空间”对话框中，输入一个
    命名空间，然后单击
    “检查可用性”按钮以确保该命名空间是唯一的。
    ![新建命名空间屏幕快照][新建命名空间屏幕快照]
5.  确保该命名空间名称可用之后，选择应该托管您的命名空间的
    国家或地区，然后单击“创建命名空间”按钮。

    您创建的命名空间随后将显示在管理门户中，
    然后要花费一段时间来激活。请等到状态变为“活动”后再继续下一步。

## 获得命名空间的默认管理凭据

若要在
新命名空间上执行管理操作（如创建队列），则需要获取该
命名空间的管理凭据。

1.  在左侧导航窗格中，单击“Service Bus”节点，
    显示可用命名空间的列表：
    ![可用命名空间屏幕快照][可用命名空间屏幕快照]
2.  从显示的列表中选择刚创建的命名空间：
    ![命名空间列表屏幕快照][命名空间列表屏幕快照]
3.  右侧的“属性”窗格将列出
    新命名空间的属性：
    ![属性窗格屏幕快照][属性窗格屏幕快照]
4.  将隐藏**默认密钥**。单击“查看”按钮以显示
    安全凭据：
    ![默认密钥屏幕快照][默认密钥屏幕快照]
5.  记下**默认颁发者**和**默认密钥**，因为您
    将在下面使用此信息来对
    命名空间执行操作。

## 如何创建 Java 应用程序来执行需要进行大量计算的任务

1.  在您的开发计算机（不必是您创建的虚拟机）上，下载 [Azure SDK for Java][Azure SDK for Java]。
2.  使用本节末尾的示例代码创建 Java 控制台应用程序。在本教程中，我们将使用 **TSPSolver.java** 作为 Java 文件名。将 **your\_service\_bus\_namespace**、**your\_service\_bus\_owner** 和 **your\_service\_bus\_key** 占位符修改为分别使用您的 Service Bus 的“命名空间”、“默认颁发者”和“默认密钥”值。
3.  编码后，将应用程序导出至可运行的 Java 存档 (JAR)，并将所需的库打包到生成的 JAR 中。在本教程中，我们将使用 **TSPSolver.jar** 作为生成的 JAR 名称。

    // TSPSolver.java

    import com.microsoft.windowsazure.services.core.Configuration;
    import com.microsoft.windowsazure.services.core.ServiceException;
    import com.microsoft.windowsazure.services.serviceBus.*;
    import com.microsoft.windowsazure.services.serviceBus.models.*;
    import java.io.*;
    import java.text.DateFormat;
    import java.text.SimpleDateFormat;
    import java.util.ArrayList;
    import java.util.Date;
    import java.util.List;

    public class TSPSolver {

        //  Value specifying how often to provide an update to the console.
        private static long loopCheck = 100000000;  

        private static long nTimes = 0, nLoops=0;

        private static double[][] distances;
        private static String[] cityNames;
        private static int[] bestOrder;
        private static double minDistance;
        private static ServiceBusContract service;

        private static void buildDistances(String fileLocation, int numCities) throws Exception{
            try{
                BufferedReader file = new BufferedReader(new InputStreamReader(new DataInputStream(new FileInputStream(new File(fileLocation)))));
                double[][] cityLocs = new double[numCities][2];
                for (int i = 0; i<numCities; i++){
                    String[] line = file.readLine().split(", ");
                    cityNames[i] = line[0];
                    cityLocs[i][0] = Double.parseDouble(line[1]);
                    cityLocs[i][1] = Double.parseDouble(line[2]);               
                }
                for (int i = 0; i<numCities; i++){
                    for (int j = i; j<numCities; j++){
                        distances[i][j] = Math.hypot(Math.abs(cityLocs[i][0] - cityLocs[j][0]), Math.abs(cityLocs[i][1] - cityLocs[j][1]));
                        distances[j][i] = distances[i][j];
                    }
                }
            } catch (Exception e){
                throw e;
            }
        }

        private static void permutation(List<Integer> startCities, double distSoFar, List<Integer> restCities) throws Exception {

            try
            {
                nTimes++;
                if (nTimes == loopCheck)
                {
                    nLoops++;
                    nTimes = 0;
                    DateFormat dateFormat = new SimpleDateFormat("MM/dd/yyyy HH:mm:ss");
                    Date date = new Date();
                    System.out.print("Current time is " + dateFormat.format(date) + ". ");
                    System.out.println(  "Completed " + nLoops + " iterations of size of " + loopCheck + ".");
                }
        
                if ((restCities.size() == 1) && ((minDistance == -1) || (distSoFar + distances[restCities.get(0)][startCities.get(0)] + distances[restCities.get(0)][startCities.get(startCities.size()-1)] < minDistance))){
                    startCities.add(restCities.get(0));
                    newBestDistance(startCities, distSoFar + distances[restCities.get(0)][startCities.get(0)] + distances[restCities.get(0)][startCities.get(startCities.size()-2)]);
                    startCities.remove(startCities.size()-1);
                }
                else{
                    for (int i=0; i<restCities.size(); i++){
                        startCities.add(restCities.get(0));
                        restCities.remove(0);
                        permutation(startCities, distSoFar + distances[startCities.get(startCities.size()-1)][startCities.get(startCities.size()-2)],restCities);
                        restCities.add(startCities.get(startCities.size()-1));
                        startCities.remove(startCities.size()-1);
                    }
                }
            }
            catch (Exception e)
            {
                throw e;
            }
        }

        private static void newBestDistance(List<Integer> cities, double distance) throws ServiceException, Exception {
            try 
            {
                minDistance = distance;
                String cityList = "Shortest distance is "+minDistance+", with route: ";
                for (int i = 0; i<bestOrder.length; i++){
                    bestOrder[i] = cities.get(i);
                    cityList += cityNames[bestOrder[i]];
                    if (i != bestOrder.length -1)
                        cityList += ", ";
                }
                System.out.println(cityList);
                service.sendQueueMessage("TSPQueue", new BrokeredMessage(cityList));
            } 
            catch (ServiceException se) 
            {
                throw se;
            }
            catch (Exception e) 
            {
                throw e;
            }
        }

        public static void main(String args[]){

            try {

                Configuration config = ServiceBusConfiguration.configureWithWrapAuthentication(
                        "your_service_bus_namespace", "your_service_bus_owner",
                        "your_service_bus_key",
                        ".servicebus.windows.net",
                        "-sb.accesscontrol.windows.net/WRAPv0.9");

                service = ServiceBusService.create(config);

                int numCities = 10;  // Use as the default, if no value is specified at command line. 
                if (args.length != 0) 
                {
                    if (args[0].toLowerCase().compareTo("createqueue")==0)
                    {
                        // No processing to occur other than creating the queue.
                        QueueInfo queueInfo = new QueueInfo("TSPQueue");

                        service.createQueue(queueInfo);

                        System.out.println("Queue named TSPQueue was created.");

                        System.exit(0);
                    }

                    if (args[0].toLowerCase().compareTo("deletequeue")==0)
                    {
                        // No processing to occur other than deleting the queue.
                        service.deleteQueue("TSPQueue");

                        System.out.println("Queue named TSPQueue was deleted.");

                        System.exit(0);
                    }

                    // Neither creating or deleting a queue.
                    // Assume the value passed in is the number of cities to solve.
                    numCities = Integer.valueOf(args[0]);  
                }

                System.out.println("Running for " + numCities + " cities.");

                List<Integer> startCities = new ArrayList<Integer>();
                List<Integer> restCities = new ArrayList<Integer>();
                startCities.add(0);
                for(int i = 1; i<numCities; i++)
                    restCities.add(i);
                distances = new double[numCities][numCities];
                cityNames = new String[numCities];
                buildDistances("c:\\TSP\\cities.txt", numCities);
                minDistance = -1;
                bestOrder = new int[numCities];
                permutation(startCities, 0, restCities);
                System.out.println("Final solution found!");
                service.sendQueueMessage("TSPQueue", new BrokeredMessage("Complete"));
            } 
            catch (ServiceException se) 
            {
                System.out.println(se.getMessage());
                se.printStackTrace();
                System.exit(-1);
            }        
            catch (Exception e) 
            {
                System.out.println(e.getMessage());
                e.printStackTrace();
                System.exit(-1);
            }
        }

    }

## 如何创建 Java 应用程序来监视需要进行大量计算的任务的进度

1.  在开发计算机上，使用本节末尾的示例代码创建 Java 控制台应用程序。在本教程中，我们将使用 **TSPClient.java** 作为 Java 文件名。如上所述，将 **your\_service\_bus\_namespace**、**your\_service\_bus\_owner** 和 **your\_service\_bus\_key** 占位符修改为分别使用您的 Service Bus 的“命名空间”、“默认颁发者”和“默认密钥”值。
2.  将应用程序导出至可运行的 JAR，并将所需的库打包到生成的 JAR 中。在本教程中，我们将使用 **TSPClient.jar** 作为生成的 JAR 名称。

    // TSPClient.java

    import java.util.Date;
    import java.text.DateFormat;
    import java.text.SimpleDateFormat;
    import com.microsoft.windowsazure.services.serviceBus.*;
    import com.microsoft.windowsazure.services.serviceBus.models.*;
    import com.microsoft.windowsazure.services.core.*;

    public class TSPClient 
    {

        public static void main(String[] args) 
        {
                try
                {

                    DateFormat dateFormat = new SimpleDateFormat("MM/dd/yyyy HH:mm:ss");
                    Date date = new Date();
                    System.out.println("Starting at " + dateFormat.format(date) + ".");

                    String namespace = "your_service_bus_namespace";
                    String issuer = "your_service_bus_owner";
                    String key = "your_service_bus_key";

                    Configuration config;
                    config = ServiceBusConfiguration.configureWithWrapAuthentication(
                            namespace, issuer, key,
                            ".servicebus.windows.net",
                            "-sb.accesscontrol.windows.net/WRAPv0.9");

                    ServiceBusContract service = ServiceBusService.create(config);

                    BrokeredMessage message;

                    int waitMinutes = 3;  // Use as the default, if no value is specified at command line. 
                    if (args.length != 0) 
                    {
                        waitMinutes = Integer.valueOf(args[0]);  
                    }

                    String waitString;

                    waitString = (waitMinutes == 1) ? "minute." : waitMinutes + " minutes."; 

                    // This queue must have previously been created.
                    service.getQueue("TSPQueue");

                    int numRead;

                    String s = null;

                    while (true)
                    {

                        ReceiveQueueMessageResult resultQM = service.receiveQueueMessage("TSPQueue");
                        message = resultQM.getValue();

                        if (null != message && null != message.getMessageId())
                        {                        

                            // Display the queue message.
                            byte[] b = new byte[200];

                            System.out.print("From queue: ");

                            s = null;
                            numRead = message.getBody().read(b);
                            while (-1 != numRead)
                            {
                                s = new String(b);
                                s = s.trim();
                                System.out.print(s);
                                numRead = message.getBody().read(b);
                            }
                            System.out.println();
                            if (s.compareTo("Complete") == 0)
                            {
                                // No more processing to occur.
                                date = new Date();
                                System.out.println("Finished at " + dateFormat.format(date) + ".");
                                break;
                            }
                        }
                        else
                        {
                            // The queue is empty.
                            System.out.println("Queue is empty. Sleeping for another " + waitString);
                            Thread.sleep(60000 * waitMinutes);
                        }
                    } 

            }
            catch (ServiceException se)
            {
                System.out.println(se.getMessage());
                se.printStackTrace();
                System.exit(-1);
            }
            catch (Exception e)
            {
                System.out.println(e.getMessage());
                e.printStackTrace();
                System.exit(-1);
            }

        }
        
    }

## 如何运行 Java 应用程序

运行需要进行大量计算的应用程序，首先创建队列，然后解决旅行商问题，这样会将当前最佳路线添加到 Service Bus 队列。需要进行大量计算的应用程序正在运行时（或运行后），运行客户端可显示来自 Service Bus 队列的结果。

### 如何运行需要进行大量计算的应用程序

1.  登录虚拟机。
2.  创建一个您要在其中运行应用程序的文件夹。例如，**c:\\TSP**。
3.  将 **TSPSolver.jar** 复制到 **c:\\TSP**，
4.  创建一个具有以下内容的名为 **c:\\TSP\\cities.txt** 的文件：

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

5.  在命令提示符处，将目录更改为 c:\\TSP。
6.  确保 JRE 的 bin 文件夹位于 PATH 环境变量中。
7.  在运行 TSP 解算器排列之前，您需要先创建 Service Bus 队列。运行以下命令以创建 Service Bus 队列：

        java -jar TSPSolver.jar createqueue

8.  在创建该队列后，您可以运行 TSP 解算器排列。例如，运行以下命令可对 8 个城市运行解算器。

        java -jar TSPSolver.jar 8

如果您没有指定数字，则它将对 10 个城市运行。在解算器找到当前最短的路线后，它会将这些路线添加到该队列中。

> [WACOM.NOTE]
> 您指定的数字越大，解算器运行的时间将越长。例如，对 14 个城市运行可能需要几分钟时间，而对 15 个城市运行可能需要几小时时间。增加到 16 个或更多个城市可能需要数天的运行时间（最终数周、数月和数年时间）。这是因为，随着城市数量的增加，解算器评估的排列数会迅速增加。

### 如何运行监视客户端应用程序

1.  登录到您要在其中运行客户端应用程序的计算机。虽然不需要是运行 **TSPSolver** 应用程序的同一计算机，但也可以是同一计算机。
2.  创建一个您要在其中运行应用程序的文件夹。例如，**c:\\TSP**。
3.  将 **TSPClient.jar** 复制到 **c:\\TSP**，
4.  确保 JRE 的 bin 文件夹位于 PATH 环境变量中。
5.  在命令提示符处，将目录更改为 c:\\TSP。
6.  运行以下命令：

        java -jar TSPClient.jar

    或者，也可以通过传入命令行参数来指定检查队列之间休眠的分钟数。用于检查队列的默认休眠期为 3 分钟，将在没有命令行参数传入 **TSPClient** 时使用该默认值。如果您要使用不同的休眠时间间隔值（如一分钟），请运行：

        java -jar TSPClient.jar 1

    客户端会一直运行，直到它看到“完成”的队列消息为止。请注意，如果您多次运行解算器而没有运行客户端，则您可能需要多次运行客户端才能完全清空队列。或者，您可以删除该队列，然后重新创建一个。若要删除队列，请运行以下 **TSPSolver**（不是 **TSPClient**）命令：

        java -jar TSPSolver.jar deletequeue

    解算器会一直运行，直到检测完所有路线为止。

## 如何停止 Java 应用程序

对于解算器和客户端应用程序，如果您希望在正常完成之前结束，则可以按 **Ctrl+C** 退出。

  [旅行商问题解算器]: ./media/virtual-machines-java-run-compute-intensive-task/WA_JavaTSPSolver.png
  [旅行商问题客户端]: ./media/virtual-machines-java-run-compute-intensive-task/WA_JavaTSPClient.png
  [Azure 管理门户]: https://manage.windowsazure.cn
  [将证书添加到 Java CA 证书存储]: ../java-add-certificate-ca-store
  [Service Bus 节点屏幕快照]: ./media/virtual-machines-java-run-compute-intensive-task/SvcBusQueues_02_SvcBusNode.jpg
  [新建命名空间屏幕快照]: ./media/virtual-machines-java-run-compute-intensive-task/SvcBusQueues_03_CreateNewSvcNamespace.jpg
  [可用命名空间屏幕快照]: ./media/virtual-machines-java-run-compute-intensive-task/SvcBusQueues_04_SvcBusNode_AvailNamespaces.jpg
  [命名空间列表屏幕快照]: ./media/virtual-machines-java-run-compute-intensive-task/SvcBusQueues_05_NamespaceList.jpg
  [属性窗格屏幕快照]: ./media/virtual-machines-java-run-compute-intensive-task/SvcBusQueues_06_PropertiesPane.jpg
  [默认密钥屏幕快照]: ./media/virtual-machines-java-run-compute-intensive-task/SvcBusQueues_07_DefaultKey.jpg
  [Azure SDK for Java]: http://azure.microsoft.com/zh-cn/develop/java/
