<properties
	pageTitle="Azure 云服务操作手册 - 第二部分 | Azure"
	description="Azure 云服务操作手册 - 第二部分 | Azure"
	services=""
	documentationCenter=""
	authors="Lei Zhang"
	manager=""
	editor=""/>

<tags
	ms.service="cloud-services"
	ms.date=""
	wacn.date="09/21/2016"/>



#Azure 云服务操作手册

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

[Azure 云服务操作手册 - 第一部分](/documentation/articles/azure-cloud-services-user-manual-part-1/)

##<a id="azure-cloud-service-get-started"></a>2.	着手创建 Azure 云服务  

###<a id="azure-subscription-plan"></a>2.1 合理规划 Azure 订阅  

订阅是 Azure 账单分拆的最小单位。  

如果一个企业的 IT 部门、销售部门、市场部门均使用 Azure，并且需要根据不同部门的 Azure 实际使用量进行内部成本核算，就要合理规划三种不同的 Azure 订阅。创建 Azure IaaS 相关资源时，将这些资源创建在不同的订阅下。

具体请参考 [Azure 企业门户管理手册](/documentation/articles/azure-ea-portal-user-manual/)

###<a id="azure-subscription-choose"></a>2.2 选择订阅  

登陆 Azure [管理门户](http://manage.windowsazure.cn)，输入账户和密码。点击右上角的订阅按钮，如下图:  

![登陆][6]


###<a id="azure-emulator"></a>2.3 Azure Emulator 模拟器  

安装完 Azure SDK 后，会默认安装 Azure Emulator 模拟器。如下图：  

![Azure Emulator 模拟器][7]

上图中，模拟器分为两种：

1. Compute Emulator  

	计算模拟器。通过该模拟器模拟云服务在云端执行的情况。

2. Storage Emulator  

	存储模拟器。通过该模拟器模拟 Azure 存储执行情况。

需要在本地调试时，可以直接使用 Azure Emulator，模拟云服务在云端执行的大部分功能。  

###<a id="web-role-and-worker-role"></a>2.4 Web Role / Worker Role  

开发 Azure 云服务项目时，有一个非常重要的概念，那就是 Web Role 和 Worker Role。  

创建 Azure 云服务时，会提示如下选项：  

![选项][8]

1. Web Role 是响应客户端的 HTTP 请求；  
2. Worker Role 则是在后台执行的应用程序，概念上类似于 Windows Service；  
3. Web Role 可以通过 Azure 列队存储，向 Worker Role 发送队列消息 (Queue.AddMessage())，让 Worker Role 在后端执行自己需要的逻辑。

一个云服务可以：

1. 只有 Web Role，没有 Worker Role；  
2. 只有 Worker Role，没有 Web Role；  
3. 既有 Web Role，又有 Worker Role。

> [AZURE.NOTE] 上图中的 Worker Role 分为三类 (Worker Role, Cache Worker Role 和 Worker Role with Service Bus Queue)。本文只介绍 Worker Role。

为什么 Azure 有 Web Role / Worker Role？它的好处在哪里？接下来举例说明。

比如有一个信息管理系统，需要上传 Excel 文档来进行解析和处理。从软件设计的角度来说，有 2 种解决方法：  

1. 在 ASP.NET 应用程序中新建一个 upload control，在 upload control 里写函数：一旦 Excel 文件上传完毕，则在 .cs 文件执行对 Excel 的处理工作。

	但这样有一个缺点，如果 Excel 文件包含内容过多，需要花费很长时间处理，前台的 ASP.NET 页面会停滞或者无响应。虽然可以通过增加 progressbar 或者 loading 图片来增强用户体验，但是从软件设计上来说不是最好的解决方法。

2. 前端仍用原来的处理方式，即使用 upload control。服务器端增加一个 Windows Service，按时序查询某一个文件夹，一旦发现前端页面上传了 Excel 文件，则 Windows Service 开始处理 Excel 的工作。这样前端页面会及时响应，实现更好的用户体验。

	但是这样处理仍有一个缺陷，前端页面和 windows service 是一对一的关系，如果附件上传的数量很大，会出现 Windows Service 来不及处理的情况。

3. 如果有 Worker Role，一个 ASP.NET 页面后端可以有多个 Worker Role 来进行分布式计算，它们是一对多的关系，能够有效的利用云上的计算资源，Worker Role 可以处理高负载的数据访问。

这样的架构亮点如下：  

1. 异步处理，Web Role 只对客户端的 HTTP 请求进行快速响应；而 Worker Role 在后端处理 Web Role 发送过来的消息 (Queue)时，两者是松耦合的。

	在传统的 Web 应用中，如果我们把复杂的处理逻辑写在 ASPX 页面，该页面可能会停滞，无法实现良好的用户体验。 Azure PaaS 使用 Web Role 和 Worker Role，Web Role 只响应客户端的 HTTP 请求；而 Worker Role 可以在后端对业务逻辑进行异步处理。

2. Web Role 和 Worker Role 的关系是多对多，比如可在 Web Role 的配置中设置 Instance count 为10。如下图：

	![设置 Instance count][9]

	在 Worker Role 的配置中，设置Instance Count为3  

	![设置Instance Count][10]

这种架构就好比一个餐厅，里面有 10 个服务员 (Web Role) 和 3 个厨师 (Worker Role)：  

1. 服务员 (Web Role) 只负责招待客人(响应客户端请求)，并将客人的点菜信息通过队列消息交给厨房(Queue.AddMessage())；  
2. 厨师 (Worker Role)读取点菜信息(Queue.GetMessage)，随后负责烧菜做饭(进行后端逻辑)，饭菜准备完毕后，删除点菜信息；  
3. 如果前端压力过大(客户端请求过多)，则 Web Role 可以横向扩展; 如果后端压力过大(后端逻辑处理过多)，则 Worker Role 也可以横向扩展；  
4. 这种松耦合、多对多的关系非常适合企业级应用架构。

由此可见，Web Role 和 Worker Role 相互通信的重要途径是 Azure 队列存储，因此了解队列对于 Azure PaaS 架构设计非常重要。  

###<a id="create-an-azure-cloud-service"></a>2.5 创建 Azure 云服务

创建前用户需准备：  

- Azure 账户<br/>
- Visual Studio 2013<br/>
- 安装 [Azure SDK](/downloads/?sdk=net)

1. 首先以管理员身份，运行 Visual Studio 2013；  
2. 点击 File -> New -> Project；   
3. 在弹出的界面中，选择 Cloud -> Azure 云服务；  

	![界面][11]


4. 点击上图的 OK 按钮，依次添加 ASP.NET Web Role 和 Worker Role；  

	![添加][12]

	上图中，还可以对 Web Role 和 Worker Role 进行重命名。  

	一个云服务可以：
	* 只有 Web Role，没有 Worker Role；  
	* 只有 Worker Role，没有 Web Role；  
	* 既有 Web Role，又有 Worker Role。

	上图中，请注意：

	Azure Cache Worker Role 功能会在 2016 年 11 月 30 日下线。 

	如果用户想使用 Cache 缓存服务，请使用 [Azure Redis 缓存](/documentation/services/redis-cache/)。


5. 然后可以根据需求，选择相应的 ASP.NET 模板。这里选择 Web Forms，如下图：  

	![Web Forms][13]

###<a id="check-azure-cloud-service"></a>2.6 查看 Azure 云服务  

一个 Azure 云服务解决方案，包含三个 Project，分别是:  

1. Azure 云服务  
2. Web Role  
3. Worker Role  

####<a id="azure-cloud-service-components"></a>2.6.1 云服务  

Azure 云服务的项目结构如下图所示：  

![项目结构][14]

从目录结构上，可以观察到：  

1. 该项目不同于传统的 Visual Studio Project，是一个云服务。(带一个云的 Logo)；  
2. 该云服务包含 2 个 Role，分别是 Web Role 和 Worker Role；  
3. 该云服务包含了 2 个配置文件(CSCFG, 云服务 Configuration)，文件名分别是 Cloud 和 Local；  
4. 该云服务包含了 1 个定义文件(CSDEF, 云服务 Definition)。


####<a id="azure-cloud-service-web-role"></a>2.6.2 Web Role

Web Role 项目结构如下图：  

![Web Role项目结构][18]

上图即 Web Form 项目，有 Default.aspx，Global.asax，Web.config。  

但是这个项目文件包含了 WebRole.cs，代码如下：  

![WebRole.cs][15]

WebRole.cs 的代码逻辑是：  

1. OnStart() 方法，在 Web Role 启动的时候执行；  
2. 我们还可以根据自己的需求，override OnStop() 方法和 Run() 方法。

####<a id="azure-cloud-service-worker-role"></a>2.6.3 Worker Role  

Worker Role 的项目文件如下图：  

![项目文件][16]

WorkerRole.cs 的代码片段如下：  

![WorkerRole.cs 的代码][17]

从上图的 2 个代码片段可以看到，Worker Role 的代码逻辑是：  

1. OnStart() 方法，在 Worker Role 启动的时候执行；  
2. Run() 方法，在 Worker Role 运行的时候执行；  
3. Run() 方法，包含了一个异步执行的方法 RunAsync。具体的业务逻辑是，每 1000 毫秒，输出内容 ”Working”。我们可以根据自己的需求，修改 RunAsync() 方法的具体业务逻辑；  
4. OnStop() 方法，在 Worker Role 关闭的时候执行。

从上面的代码片段中，可以观察到 Worker Role .cs 的代码逻辑就像是一个 Windows Service，在背后默默运行。  

####<a id="azure-cloud-service-look-back"></a>2.6.4 回顾  

本节内容回顾：  

![回顾][19]

1. 新创建的云服务有 2 个 Role，即Web Role 和 Worker Role；  
2. Web Role 是前端进行展示的用户界面，使用的 ASP.NET Web Form 代码；  
3. Worker Role 是在后端默默执行的函数，概念上类似于 Windows Service，只有一个类 WorkerRole.cs。  

###<a id="cscfg-and-csdef"></a>2.7 cscfg 和 csdef  

上节中，笔者简要介绍了 Azure 云服务项目和项目的基本构成，接下来介绍 cscfg 文件和 csdfg 文件。

在此之前，先介绍 2 个开发概念，接口 (Interface) 和类 (Class)。  

1. 接口定义了行为；  
2. 类继承了接口，实现了接口中具体的行为。

cscfg 和 csdef 也是相同的概念：  

1. csdef — 全称是云服务 Definition，定义了云服务的参数，概念上类似于接口；  
2. cscfg — 全称是云服务 Configuration，配置了云服务的参数，概念上类似于类。  

在 Azure 云服务里，配置文件(云服务 Configuration)分为两种，Cloud 和 Local。 

####<a id="local-cscfg"></a>2.7.1 ServiceConfiguration.Local.cscfg  

如果需要在本地通过 Azure Emulator 模拟器调试 Azure 云服务，可以把调试的参数信息保存到 Local.cscfg 里。  

####<a id="cloud-cscfg"></a>2.7.2 ServiceConfiguration.Cloud.cscfg  

本地调试通过后，如果云服务需要部署到 Azure 云端执行，可以把 Azure 云端的参数保存到 Cloud.cscfg 里。  

通过 Local.cscfg 和 Cloud.cscfg，一个 Azure 云服务可以对应本地模拟器环境和云端环境，分别读取不同的配置参数信息，方便进行 debug 调试和生产部署。  

####<a id="csdef"></a>2.7.3 csdef文件  

csdef 文件 — 全称是云服务 Definition，定义了云服务的参数，概念类似于接口。这里不再做介绍。  

####<a id="cscfg-csdef-considerations"></a>2.7.4 注意事项  

请注意，修改 cscfg 和 csdef 时，最好通过 Visual Studio 等集成开发环境(Integrated Development Environment, IDE)，不要通过记事本等。  

在 csdef 中定义的参数，如果没有在 cscfg 中进行配置，会产生 Visual Studio 无法识别 cscfg 和 csdef，导致配置失败。  

###<a id="azure-cloud-service-config"></a>2.8 配置 Azure 云服务  

####<a id="azure-cloud-service-config-web-role"></a>2.8.1 配置 Web Role  

点击 Web Role，右键，属性。如下图：  

![属性][20]

#####<a id="azure-cloud-service-config-web-role-configuration"></a>2.8.1.1 Configuration

![Configuration][21]

上图说明了 Azure 云服务的 Web Role 配置信息：  

1. Service Configuration — 表示配置文件保存到 Cloud.cscfg 还是 Local.cscfg；  
2. Instance Count — 表示 Web Role 横向扩展的节点数量；  
3. VM Size — 表示 Web Role 在横向扩展的时候，每一个计算节点的 CPU 内存信息；  
4. Enable Diagnostics — 表示在 Azure 云服务 Web Role 启用诊断功能，可以点击 Configure 按钮，设置需要诊断的具体参数；  
5. Specify the storage account credentials for the Diagnostics result — 表示需要把诊断的结果数据，保存到 Azure 表存储中。这里需要配置 Azure 存储的连接字符串。  

	> [AZURE.NOTE] Azure 云服务背后运行的是非持久化虚拟机，任何保存在非持久化虚拟机本地磁盘的文件都会有丢失风险，所以需要把诊断数据保存到 Azure 表存储中。  

6. 以上图为例，把 Web Role 横向扩展为 10 台，每台计算节点的 CPU 内存为 2 Core/3.5 GB (A 系列)。如果是 D 系列虚拟机，显示如下： 

    ![D 系列虚拟机][22]

#####<a id="azure-cloud-service-config-web-role-settings"></a>2.8.1.2 Settings

点击 Settings 页面，显示如下：  

![Settings][23]

上图中：  

1. Service Configuration — 表示配置文件保存到 Cloud.cscfg 还是 Local.cscfg；  
2. 一般情况下，把 ASP.NET Web Form 的配置信息保存到 Web.config 文件里。但是如果多台 Web Role 横向扩展时，需要手工一台一台更新每个 Web.config 文件，费时费力；  
3. 建议现在把以前保存在 Web.config 文件里的 Web Role 的配置信息，保存在 Web Role 的 Setting 页面中。这样修改了配置文件，可以在多台的 Web Role 实例同时生效，后面有 DEMO 演示；  
4. 上图中笔者点击 Add Setting，增加一个参数名 Setting1，参数值为 Hello World。  

#####<a id="azure-cloud-service-config-web-role-endpoints"></a>2.8.1.3 Endpoints  

点击 Endpoints，如下图:  

![Endpoints][24]

上图中：  

1. 默认只有 Endpoint1，Public Port 为 80；  
2. 表示 Web Role 默认打开的端口为 80，协议为 http；  
3. 根据需要，设置打开 8080 等其他端口，并且可修改协议为 https。  

#####<a id="azure-cloud-service-config-web-role-local-storage"></a>2.8.1.4 Local Storage

点击 Local Storage，如下图:  

![Local Storage][25]

使用默认的设置即可，不需要进行修改。  

#####<a id="azure-cloud-service-config-web-role-certificates"></a>2.8.1.5 Certificates 

点击 Certificates，如下图：  

![Certificates][26]

上图中，可以上传 Web Role 通过 HTTPS 访问需要的证书，具体不作演示，请参考[Blog](http://www.cnblogs.com/threestone/p/3951227.html)。 

#####<a id="azure-cloud-service-config-web-role-caching"></a>2.8.1.6 Caching  

该配置项供 Azure Cache Worker Role 使用，但是 Azure Cache Worker Role 功能会在 2016 年 11 月 30 日下线。

如果用户想使用 Cache 缓存服务，请使用 Azure Redis 缓存，请参考[此处](/home/features/redis-cache/)。  

#####<a id="azure-cloud-service-config-web-role-look-back"></a>2.8.1.7 回顾  

本节对 Web Role 进行了以下配置。

在 Configuration 进行的配置如下：  

1. Web Role 需要横向扩展，共 10 个节点；  
2. 每个节点的硬件配置为 A2 ( 2 Core / 3.5 GB)；  
3. Enable Diagnostics — 表示在 Azure 云服务启用诊断功能；  
4. Specify the storage account credentials for the Diagnostics result — 表示我们需要把诊断的结果数据，保存到 Azure 表存储中。

在 Settings 进行了如下配置：  

- 增加了一个参数，参数名 Setting1，参数值为 Hello World

在 Endpoints 进行了如下配置：  

- Web Role 默认打开的端口为 80，协议为 http  

#####<a id="azure-cloud-service-config-worker-role"></a>2.8.2 配置 Worker Role  

点击 Worker Role，右键，属性。如下图：  

![属性][27]

######<a id="azure-cloud-service-config-worker-role-configuration"></a>2.8.2.1 Configuration

![configuration][28]  

上图说明了 Azure 云服务的 Worker Role 配置信息：  

1. Service Configuration — 表示配置文件保存到 Cloud.cscfg 还是 Local.cscfg；  
2. Instance Count — 表示 Worker Role 横向扩展的节点数量；  
3. VM Size — 表示 Worker Role 在横向扩展时，每一个计算节点的 CPU 内存信息；  
4. Enable Diagnostics — 表示在 Azure 云服务 Worker Role 启用诊断功能，可以点击 Configure 按钮，设置需要诊断的具体参数；  
5. Specify the storage account credentials for the Diagnostics result — 表示需要把诊断的结果数据保存到 Azure 表存储中，这里需要配置 Azure 存储的连接字符串。  

    > [ 注意事项 ] Azure 云服务背后运行的是非持久化虚拟机，任何保存在非持久化虚拟机本地磁盘的文件都会有丢失风险，所以需要把诊断数据保存到 Azure 表存储中。  


6. 以上图为例，把 Worker Role 横向扩展为 10 台，每台计算节点的 CPU 内存为 2 Core/3.5 GB (A 系列)。如果是 D 系列虚拟机，显示如下：  

	![][29]

######<a id="azure-cloud-service-config-worker-role-settings"></a>2.8.2.2 Settings  

点击 Settings 页面，显示如下：  

![Settings][30]

上图中：  

1. Service Configuration — 表示配置文件保存到 Cloud.cscfg 还是 Local.cscfg；<br/>
2. 这里不增加任何参数。  

######<a id="azure-cloud-service-config-worker-role-endpoints"></a>2.8.2.3 Endpoints  

点击 Endpoints，如下图：  

![Endpoints][31]

上图中：  

1. 默认没有任何 Endpoint，也没有 Public Port；<br/>
2. Internet 用户无法直接访问 Azure Worker Role。  

######<a id="azure-cloud-service-config-worker-role-local-storage"></a>2.8.2.4 Local Storage  

使用默认的设置即可，不需要进行修改。  

######<a id="azure-cloud-service-config-worker-role-certificates"></a>2.8.2.5 Certificates  

点击 Certificates，如下图：  

![Certificates][32]

上图中，可以上传 Web Role 通过 HTTPS 访问需要的证书。 

######<a id="azure-cloud-service-config-worker-role-caching"></a>2.8.2.6 Caching  

该配置项供 Azure Cache Worker Role 使用，但是 Azure Cache Worker Role 功能会在 2016 年 11 月 30 日下线。

如果用户想使用 Cache 缓存服务，请参考 [Azure Redis 缓存](/home/features/redis-cache/)。  

######<a id="azure-cloud-service-config-worker-role-look-back"></a>2.8.2.7 回顾  

本节对 Worker Role 进行了以下配置。

在 Configuration 进行的配置如下：  

1. Worker Role 需要横向扩展，一共 10 个节点；  
2. 每个节点的硬件配置为 A2 (2 Core / 3.5 GB)；  
3. Enable Diagnostics，表示在 Azure 云服务启用诊断功能；  
4. Specify the storage account credentials for the Diagnostics result，表示需要把诊断的结果数据，保存到 Azure 表存储中；

在 Worker Role 的 Endpoints 配置页面里，没有任何的 Endpoint，也没有 Public Port。Internet 用户无法直接访问到 Azure Worker Role。  

###<a id="check-csdef-and-cscfg"></a>2.9 查看 csdef 和 cscfg   

前几节介绍了 Azure 云服务的 cscfg 和 csdef 有如下关系：  

1. csdef — 全称是云服务 Definition，是定义了云服务的参数，概念上类似于接口；<br/>
2. cscfg — 全称是云服务 Configuration，是配置了云服务的参数，概念上类似于类。  

####<a id="check-csdef-and-cscfg-servicedefinition"></a>2.9.1 查看 ServiceDefinition.csdef  

通过 Visual Studio 2013 双击打开 ServiceDefinition.csdef 文件，如下图：

![ServiceDefinition.csdef][33]

打开后的显示结果如下：  

![ServiceDefinition.csdef][34]

上图中，定义了以下内容：  

项目包含了 Web Role，计算节点大小为 Medium (A2, 2 Core / 3.5 GB)  

1. Web Role 项目包含了访问节点 Endpoint1；  
2. Web Role 项目包含了配置信息，参数名为 ConnectionString 和 Setting1；  
3. Endpoint1 访问的协议是 http，访问的端口是 80。

项目包含了 Worker Role，计算节点大小为 Medium (A2, 2 Core / 3.5 GB)  

1. Worker Role 只包含了配置信息，参数名为 ConnectionString。  

####<a id="check-csdef-and-cscfg-serviceconfiguration"></a>2.9.2 查看 ServiceConfiguration.Cloud.cscfg
通过 Visual Studio 2013 双击打开 ServiceConfiguration.Cloud.cscfg 文件，如下图：  

![ ServiceConfiguration.Cloud.cscfg][35]

在 ServiceConfiguration 节点，可以看到如下参数:  

![ServiceConfiguration][36]

上图中的参数含义是:  

1. osFamily = ”4” — 表示云服务后台的操作系统是 Windows Server 2012 R2。 所以 Azure 云服务背后运行的操作系统，一定是 Windows Server 2012 R2。 有关osFamily的含义请参考 [MSDN文档](https://msdn.com/zh-cn/library/azure/ee758710.aspx)  
2. osVersion = ”*” — 表示由 Azure 来帮助用户升级操作系统补丁，并且保证补丁最新。   

还可以查看 ServiceConfiguration.Cloud.cscfg，如下图：  

![ServiceConfiguration.Cloud.cscfg][37]

上图中，定义了 Web Role 信息：  

1. 项目文件包含了 WebRole1，实例数量为 10 个；  
2. 定义了参数名 ConnectionString 和参数值；  
3. 定义了参数名 Setting1 和参数值 Hello World。

还定义了 Worker Role 信息：  

1. 项目文件包含了 WorkerRole1，实例数量为 10 个；  
2. 定义了参数名 ConnectionString 和参数值。  

###<a id="debug-with-azure-emulator"></a>2.10 使用模拟器调试  

####<a id="debug-with-azure-emulator-config"></a>2.10.1 修改配置
注意：
使用本地模拟器调试时，请配置文件 ServiceConfiguration.Local.cscfg 修改：  

1. 把 WebRole1 和 Worker Role1 的 Instance Count 都修改为 2。上几节内容把 Instance Count 修改为 10，但是在本地调试的话会耗费过多的资源。  
2. 将 ConnectionString 配置信息修改为 UseDevelopmentStorage = true。

如下图红色部分所示：  

![ConnectionString][38]

####<a id="debug-with-azure-emulator-coding"></a>2.10.2 修改代码  

同时还要修改以下内容：  

1. 在 WebRole1 的 Default.aspx 页面，增加 2 个 Label：LblInstance 和 LblSetting；
2. 在Default.aspx.cs的Page_Load方法，增加以下代码：  

    ![代码][39]

    上面的代码有 2 点非常重要的地方：

    1. lblInstance 表示响应客户端请求的 Web Role Instance；  

    2. LblSetting 的值，是读取 Web Role Settings1 参数值。  

    关于读取 Web Role Setting1 参数值：  

    1. 一般情况下，把 ASP.NET Web Form 的配置信息保存到 Web.config 文件里。但是如果多台 Web Role 横向扩展时，需要手工一台一台更新每个 Web.config 文件，费时费力； 

    2. 读取 Web.config 的.NET API 是：ConfigurationManager.AppSettings；  

   	3. 笔者建议以前保存在 Web.config 文件里 Web Role 的配置信息，现在可以保存到 Web Role的Setting 页面里。这样修改了配置文件后，可以在多台 Web Role 实例同时生效。后面 DEMO 中会有演示；  

    4. 读取 ServiceConfiguration.Cloud.cscfg 配置文件的.NET API 是 RoleEnvironment.GetConfigurationSettingValue。

3. 修改 WebRole1 的 WebRole.cs，增加以下代码：  

	![代码][40]

上面的代码表示 Configuration 的变更和通知机制，后面几章会做详细介绍。  

####<a id="debug-with-azure-emulator-debugging"></a>2.10.3 观察模拟器  

现在可以通过本地的 Azure Storage Emulator 和 Azure Storage Emulator 来进行调试。  

1. 点击 Visual Studio 2013，启动项目；  
2. Azure Compute Emulator 和 Azure Storage Emulator 会启动，在 Windows 右下角会出现模拟器图标，如下图：

	![模拟器图标][41]

3. 点击上图的蓝色图标，点击右键；  

    ![蓝色图标][42]

    点击上图中的 Show Compute Emulator UI，可以查看到 Azure 云服务有 2 个 Web Role 和 2 个 Worker Role，如下图：

    ![Show Compute Emulator U][43]

    上图的 Instance Count 就是 4.10.1 小节中修改的 Instance count = 2，Web Role 和 Worker Role 分别是 2 个实例。

4. 点击 Worker Role 的绿色图标，如下图：  

    ![绿色图标][44]

    可以查看到 Work Role 的打印输出，这个输出其实就是在 WorkerRole1 的 WorkerRole.cs 中 RunAsync() 的输出：  

    ![RunAsync()][45]

上面的代码片段里 await Task.Delay (1000) 表示每 1000 毫秒(1 秒)执行打印输出。

由此可见，其实 Worker Role 就像是 Windows Service 一样，没有界面，在后台默默执行。当然也可以改变 Task.Delay(1000)，修改每次执行的时间。  

####<a id="debug-with-azure-emulator-run"></a>2.10.4 代码运行结果

启动模拟器时，IE 会打开 Web Role 的页面内容，如下图：  

![Web Role][46]

上图可观察到红色的部分。当前的客户端请求，是由服务器端的 WebRole1_IN_0 实例来处理的。

不停的按 IE 的 F5，对当前页面刷新，会观察到下图所示界面：  

![F5][47]

上图又观察到红色的部分。客户端的请求，是由服务器端的 WebRole1_IN_1 实例来处理的。  

所以默认情况下，Azure 云服务是绝对负载均衡，无法保留 Session(会话)。  

###<a id="azure-cloud-service-packaging"></a>2.11 Azure 云服务打包

上节中笔者介绍了通过本地 Azure Emulator 模拟器，模拟 Azure 云服务运行情况。  

接下来，需要把代码通过 Visual Studio 打包，观察打包的情况。  

####<a id="azure-cloud-service-package"></a>2.11.1 云服务打包

1.  点击 Azure 云服务，右键 Package，如下图：

    ![Package][48]

2. 在弹出的窗口中，Service Configuration 选择 Cloud，即使用 ServiceConfiguration.Cloud.cscfg 的配置信息 Build Configuration 选择 Release。如下图:  

    ![Release][49]

3. 点击上图中的 Package，Visual Studio 会对整个 Azure Cloud Project 的 Web Role 和 Worker Role 进行打包。打包结束后，会弹出新的文件夹窗口，如下图：  

    ![文件夹窗口][50]

4. 上图中，会有 2 个项目文件 cspkg 和 cscfg 。

####<a id="azure-cloud-service-package-cspkg"></a>2.11.2 CSPKG

1.  cspkg 全称是云服务 Package，即云服务中 Web Role 和 Worker Role，进行压缩后打包的项目。我们可以用 7Zip 打开 cspkg 文件。如下图：
    ![cspkg][51]

    可以看到在这个 cspkg 文件中，包含了很多项目文件。而且我们从文件名中可以看到 WebRole 和 WorkerRole。如上图红色部分。

2. 再用 7Zip 打开上图的 Web Role 文件里的 approot 文件夹，显示结果如下图：  

    ![approot][52]

    可以看到上图中包含了 About.aspx，Default.aspx, Global.asax 还有 Web.config 文件。这个文件夹包含的就是我们在 Visual Studio 中，Web Role 项目的源代码。

3. 再用 7Zip，打开 WorkerRole 文件夹里的 approot 文件夹。显示结果如下图：  

    ![approot][53]

    可以看到，上图中的 WorkerRole1.dll，就是我们在 Visual Studio 中，Worker Role 项目的编译后的类库。

####<a id="azure-cloud-service-package-cscfg"></a>2.11.3 CSCFG

在云服务打包时，Service Configuration 选择 Cloud，即使用 ServiceConfiguration.Cloud.cscfg 的配置信息，如下图:  

![cscfg][54]

###<a id="azure-cloud-service-publish"></a>2.12 Azure 云服务发布

之前介绍了如何通过模拟器在本机调试云服务，现在介绍如何将 Azure 云服务发布到 Azure 云端。  

因为中国的 Azure 由世纪互联运营，其云服务的服务端点 (Service Endpoint) 和海外的 Azure 不同，这里详细介绍一下。

####<a id="azure-cloud-service-publish-download-publish-certification"></a>2.12.1 使用 Azure PowerShell 下载发布凭据

通过 Visual Studio 发布云服务，需要借助 Azure PowerShell 下载发布凭据 (publish certification)。

1.  首先下载 [Azure PowerShell](/documentation/articles/powershell-install-configure/), 选择命令行工具，点击安装。如下图：  

    ![Azure PowerShell][55]

2. 以管理员身份，运行 Azure PowerShell，执行以下命令:  

    	Get-AzurePublishSettingsFile -Environment AzureChinaCloud

    请注意上图中-Environment 部分的命令，与国际版 Azure 不同，是专为中国版 Azure 的 PowerShell 做的特殊的参数。  
    输入命令后，计算机会弹出新的 IE 窗口，导航至中国版 Azure 网站，并要求我们输入 Org ID 和密码进行登陆。

3. 登陆完毕后，系统会要求保存扩展名为 publishsettings 的文件，将其保存至本地计算机的 D 盘上。如下图:  

    ![][56]

    这个 publishsettings 的文件，就是使用 Visual Studio 发布云服务需要的发布凭据。

####<a id="azure-cloud-service-publish-import-publish-certification"></a>2.12.2 使用 Visual Studio 导入发布凭据

1.  回到 Visual Studio 2013，点击 Server Explorer，选择 Azure，右键，选择 Manage and Filter Subscriptions。如下图：

    ![Manage and Filter Subscriptions][57]

2. 在弹出的窗口中，选择 Certificates，点击 Import，如下图：

    ![Import][58]

3. 导入上节中下载的 publishsettings 文件，如下图：

    ![导入][59]

4. 导入成功后，即可查看到对应的订阅信息，如下图：

    ![订阅信息][60]

####<a id="azure-cloud-service-publish-config-rdp"></a>2.12.3 发布云服务并配置远程桌面连接

1.  回到 Visual Studio，点击项目文件 Azure 云服务，右键，点击 Publish，如下图：

    ![Publish][61]

2. 首先选择相应的订阅，#  #如下图：

    ![选择相应的订阅][62]

3. 首次发布需要指定该云服务的 DNS Name 和所在的数据中心。如下图所示，指定 DNS Name 为 Lei 云服务
    所在数据中心，选择 China East 中国东部，最后点击 Create；  

    ![Create][63]

4. 上图 Create 完毕后，进入下一步。还可以配置远程桌面连接，点击下图的 Settings；  

    ![Settings][64]

5. 在弹出的窗口中，输入远程桌面连接的用户，密码，确认密码和 RDP 账户过期时间。如下图：  

    ![远程桌面连接][65]

6. 点击上图的 OK 后，弹出窗口关闭。然后点击下图的 Next 按钮；  

    ![Next][66]

7. 最后点击 Publish，直接将 Azure 云服务发布到 Azure 云端。  

    ![Publish][67]

####<a id="azure-cloud-service-publish-output"></a>2.12.4 观察发布过程和发布结果

#####<a id="azure-cloud-service-publish-process"></a>2.12.4.1 发布过程

点击上一节图片中的 Publish，发布 Azure 云服务，具体发布过程在下一节中会详细的介绍。本节只需要看发布过程和发布结果。
点击发布以后，Visual Studio 会把 Cloud Project 中 的 Web Role 和 Worker Role 发布到 Azure 云端，如下图：  

![发布过程][68]

上图红色区域的发布输出，如下：

	15:27:37 - Warning: There are package validation warnings.
	15:27:37 - Checking for Remote Desktop certificate...
	15:27:38 - Uploading Certificates...
	15:27:56 - Applying Diagnostics extension.
	15:28:35 - Preparing deployment for AzureCloudService1 - 2016/6/8 15:27:12 with Subscription ID 'e2eaa986-29d9-48c9-8302-1e2900a4504b' using Service Management URL 'https://management.core.chinacloudapi.cn/'...
	15:28:35 - Connecting...
	15:28:35 - Verifying storage account 'leicloudservice'...
	15:28:37 - Uploading Package...
	15:28:45 - Creating...
	15:29:21 - Created Deployment ID: f9f99ee5f486449995afc75178d8faa1.
	15:29:21 - Instance 0 of role WebRole1 is stopped
	15:29:21 - Instance 1 of role WebRole1 is creating the virtual machine
	15:29:21 - Instance 0 of role WorkerRole1 is creating the virtual machine
	15:29:21 - Instance 1 of role WorkerRole1 is creating the virtual machine
	15:29:21 - Starting...
	15:29:42 - Initializing...
	15:29:42 - Instance 0 of role WebRole1 is creating the virtual machine
	15:29:42 - Instance 1 of role WebRole1 is starting the virtual machine
	15:29:42 - Instance 0 of role WorkerRole1 is starting the virtual machine
	15:29:42 - Instance 1 of role WorkerRole1 is starting the virtual machine
	15:30:14 - Instance 0 of role WebRole1 is starting the virtual machine
	15:31:48 - Instance 0 of role WebRole1 is in an unknown state
	15:31:48 - Instance 1 of role WebRole1 is busy
		Details: Preparing to start role... System is initializing. [2016-06-08T07:31:35Z]
	15:31:48 - Instance 0 of role WorkerRole1 is in an unknown state
	15:31:48 - Instance 1 of role WorkerRole1 is busy
		Details: Preparing to start role... System is initializing. [2016-06-08T07:31:40Z]
	15:32:19 - Instance 0 of role WebRole1 is busy
		Details: Initializing role... System is initializing. [2016-06-08T07:31:45Z]
	15:32:19 - Instance 1 of role WebRole1 is busy
		Details: Initializing role... System is initializing. [2016-06-08T07:31:35Z]
	15:32:19 - Instance 0 of role WorkerRole1 is busy
		Details: Initializing role... System is initializing. [2016-06-08T07:31:43Z]
	15:32:19 - Instance 1 of role WorkerRole1 is busy
		Details: Initializing role... System is initializing. [2016-06-08T07:31:40Z]
	15:33:53 - Instance 0 of role WebRole1 is busy
		Details: Bringing role online... Calling OnRoleRun. [2016-06-08T07:33:40Z]
	15:33:53 - Instance 1 of role WebRole1 is busy
		Details: Bringing role online... Calling OnRoleRun. [2016-06-08T07:33:31Z]
	15:33:53 - Instance 0 of role WorkerRole1 is busy
		Details: Bringing role online... Calling OnRoleRun. [2016-06-08T07:33:46Z]
	15:33:53 - Instance 1 of role WorkerRole1 is busy
		Details: Waiting for role to start... Calling OnRoleRun. [2016-06-08T07:33:48Z]
	15:35:58 - Instance 0 of role WebRole1 is ready
	15:35:58 - Instance 1 of role WebRole1 is ready
	15:35:58 - Instance 0 of role WorkerRole1 is ready
	15:35:58 - Instance 1 of role WorkerRole1 is ready
	15:35:58 - Created web app URL: http://leicloudservice.chinacloudapp.cn/ 
	15:35:58 - Complete.

#####<a id="azure-cloud-service-publish-result"></a>2.12.4.2 发布结果

1.  登陆 Azure [管理门户](https://manage.windowsazure.cn)；  
2. 点击云服务，点击之前设置的 DNS Name：LeiCloudService；  
3. 页面跳转，点击，实例，生产。如下图：  

    ![生产][69]

4. 可以看到 Web Role 有 2 个实例，每个实例的大小为 A2；  
5. 2 个 Web Role 在不同的更新域 (Upgrade Domain) 和故障域 (Fault Domain)；  
6. Worker Role 有 2 个实例，每个实例的大小为 A1；  
7. 2 个 Worker Role 在不同的更新域 (Upgrade Domain) 和故障域 (Fault Domain)；  
8. 在 Visual Studio 云服务项目中，分别配置了 Web Role 和 Worker Role 横向扩展的数量和实例大小；  
9. 云服务自动把 Visual Studio 项目中，Web Role 和 Worker Role 的源代码打包，分发到若干个 Instance 实例上，并且自动设置了高可用。

打开浏览器，查看发布结果：http://leicloudservice.chinacloudapp.cn/  

![查看发布结果][70]

这个页面就是 Web Role 项目的 Default.aspx 页面，显示的是 Web Form 的 Template。  

####<a id="azure-cloud-service-life-cycle"></a>2.12.5 云服务部署和运行过程

本节进一步了解 Azure 云服务 Role 的生命周期。首先，走到幕后看看一个 Role Instance 是怎么被发布到虚拟机上并启动起来的。

#####<a id="azure-cloud-service-role"></a>2.12.5.1 Role 在虚拟机上部署和运行的过程

以下是角色实例 (Role Instance) 发布和启动的简要过程： 

1. Azure 在服务器池中选择一个有足够 CPU 内核数的宿主服务器，或者启动一个新的满足需求的宿主服务器；  
2. Azure 将云服务包 (CSPKG) 和配置文件 (CSCFG) 复制到宿主服务器上。宿主服务器上的一个宿主代理程序 (Host Agent) 会启动虚拟操作系统； 
3. 虚拟操作系统上有一个名为 WaAppAgent 的代理程序。这个代理程序负责配置虚拟操作系统并启动一个名为 WaHostBootstrapper 的进程； 
4. 如果角色上定义了启动任务，WaHostBootstrapper 会运行这些启动任务并等待所有标记为 Simple 类型的启动任务正确执行；  
5. 如果角色是 Web Role(Web Role)，WaHostBootstrapper 会启动 IISConfigurator 来配置 IIS；  
6. 如果角色是 Worker Role，WaHostBootstrapper 启动一个名为 WaWorkerHost 的进程；如果角色是 Web Role，WaHostBootstrapper 启动一个名为 WaIISHost 的进程；  
7. 在上述进程中，载入角色的程序集并搜索其实施的 RoleEntryPoint 子类；  
8. 调用 OnStart() 方法；  
9. 调用 Run() 方法。同时，该实例被标记为 "Ready" 并加入负载均衡器；  
10. 如果 Run() 方法退出，OnStop() 方法被调用。WaWorkerHost / WaIISHost 结束运行，实例重启；  
11. WaHostBootstrapper 开始循环监测实例的运行状态。  

Web Role 不一定需要实施 RoleEntryPoint 类，因为 Web Role 最终部署在 IIS 上，IIS 接受和转发用户请求。  

这也是把 WebRole.cs 从 web 项目中删除不会影响到网站运行的原因。  

但是，如果 Web Role 实施了 RoleEntryPoint 类，要确保 Run()方法不退出，否则实例会重启。  

![简要过程][71]

#####<a id="azure-cloud-service-role-instance-status"></a>2.12.5.2 Role Instance 的状态  

Role Instance 可以处于不同状态：Busy 或者 Ready。  

Role 只有在处于 Ready 状态时，才参与负载均衡任务的分配。  

如果用户想在代码中临时改变 Role Instance 的状态，可以响应 RoleEnvironment 类所定义的 StatusCheck (状态检查)事件，并且通过事件参数中 RoleInstanceStatusCheckEventArgs 的 Setbusy() 方法将实例标记为 Busy。这样，负载均衡器就不会把任务分配到当前实例，直至下一次 WaHostBootstrapper 对实例状态进行检查。  

如果 Role Instance 代码在运行过程中退出了 Run() 方法，或者抛出了未处理的异常 (Unhandledexceptions)，WaHostBootstrapper 会重新启动 Role Instance。这种自动恢复机制能够帮助提高系统的可用性。当然，在重新启动的过程中，这个实例在非 Ready 状态下不能接受用户请求。如果 WaHostBootstrapper 自身奔溃或者虚拟机宕机，Azure 会重启虚拟机以及 Role Instance。如果宿主服务器宕机，Azure 在若干恢复重试后将 Role Instance 重新部署到一台健康的服务器上。  

Azure 关闭 Role Instance 时，会触发 Stopping 事件，并调用 Role 的 OnStop() 方法。用户可以在此实现实例关闭过程中需要进行的处理，例如释放 Role 使用的资源等。  

最后，如果想强制重启实例，可以用 RoleEnvironment.RequestRecycle() 方法通知 Azure 进行实例重启。  

####<a id="azure-cloud-service-remote-connection"></a>2.12.6 远程连接云服务

1.  点击云服务，页面跳转，选择相应的实例，点击连接：  

    ![点击连接][72]

2. 在弹出的窗口，输入在 Visual Studio Project 配置的远程桌面连接的用户名和密码，就可以登陆到这台云服务背后的虚拟机：  

    ![配置的远程桌面连接][73]

3. 可以观察到这个虚拟机与传统的 Windows Server 2012 虚拟机的磁盘结构不一样；  

    C盘是模板文件  
    D盘是Windows操作系统  
    E盘approot目录下，是项目文件  

4. 通过远程桌面，运行 inetmgr，打开 IIS。运行结果如下：  

    ![IIS][74]

5. 可以看到，这台特殊版本的 Windows Server 2012 已配置好 IIS。  

###<a id="azure-cloud-service-non-persistent-vm"></a>2.13 Azure 云服务是非持久化虚拟机

本章介绍了 Azure 云服务是非持久化虚拟机，所有的部署都是通过 CSPKG 和 CSCFG 文件来提交。  

这个概念非常重要，下面举一个例子供大家参考和理解。  

1.	假设开发一个会员系统，通过 Azure 云服务的 Web Role 部署到 Azure 平台； 
2. Web Role 通过 CSPKG 和 CSCFG 部署到 Azure 云服务；  
3. 第一次部署的时间为 2016 年 1 月 1 日，Web Role 的 ASP.NET 项目，一共包含 3 个文件夹；  
4. 项目部署完毕后，Internet 用户开始使用这个会员系统，用户把头像、照片上传到云端虚拟机的本地磁盘，ASP.NET 项目变为 5 个文件夹。请注意，新增加的 2 个文件夹，并没有包含在 CSPKG 文件里；  
5. 2016 年 5 月 1 日，Web Role 所在的虚拟机宕机，Web Role 通过 CSPKG 和 CSCFG 文件，漂移到同一个数据中心的其他节点上；  
6. 请注意，因为新增加的 2 个文件夹，并没有包含在 CSPKG 文件里。所以在 Web Role 漂移以后，项目文件还原到 2016 年 1 月 1 日；
7. 2016 年 1 月 1 日至 2016 年 5 月 1 日期间的增量文件，即用户的头像、照片等全部丢失。
8. 把不包含在 CSPKG 文件里的应用系统运行期间产生的增量文件，保存到 Azure 存储里，请牢记。  

###<a id="azure-cloud-service-summary"></a>2.14 总结

1.	Azure 云服务是非持久化虚拟机，应用系统运行期间产生的增量文件，请保存到 Azure 存储账号中；  
2. Azure 云服务，可以同时包含 Web Role 和 Worker Role，或者只包含两者中的一种；  
3. Web Role 响应前端用户请求；  
4. Worker Role 在后端做任务处理；  
5. Visual Studio 项目中配置了 Web Role 的相关参数：  
    1. Configuration — 配置了 Instance Count，VM Size 和 Diagnostic；  
    2. Settings — 配置连接字符串，功能类似于 Web.config；  
    3. Endpoint — 配置了打开的端口，和端口映射，类似 Azure Virtual Machine 的 Public Port 和 Private Port；  
    4. Local Storage — 使用默认的设置即可；  
    5. Certificates — 设置 HTTPS 访问需要的证书；  
    6. Caching — 使用默认的设置，如果客户想使用 Cache 缓存服务，请使用 [Azure Redis 缓存](/home/features/redis-cache/)。  
6. 还配置了 Worker Role 的相关参数。

##<a id="developer-java"></a>3.Java 开发者必读

###<a id="developer-java-part-1"></a>3.1 第一部分

Azure 是一个开放、灵活的云平台，不仅支持自家的 .Net 平台，而且可以很好的支持各种开源软件，语言，工具，框架，比如 Java，Php，Python 等，开发人员可以使用自己喜欢的语言开发任何应用，部署在 Azure 的 web site 或者云服务中。  

![各种开源的软件][75]

Azure 云服务是 Azure 的一个 PAAS 平台，同样支持多种不同的语言和框架，并且可以基于诸如 CPU 负载，队列，定时等多种不同的阈值实现 Auto scaling 等高级功能，如下图所示：  

![Auto scaling][76]

本文简单介绍如何使用 Azure 提供的 Java Eclipse 插件，快速在 Azure 云服务中部署 Java Web 应用。正式使用此功能前，希望用户能了解以下限制，更好的设计云端体系架构：  

1.	目前 Azure 云服务底层的虚拟机是 Windows 2008/2012，如果一些跑在 Linux 上的应用需要迁移到 Azure 云服务，并且依赖于一些 Linux 的系统调用，那么需要进行代码修改；  
2. Java 应用在云服务目前只能是 workrole，没有 .Net 中 Web Role 和 work role 的定义和机制，但对于各项诸如队列，存储，数据库等云服务的使用并不受限，可通过开发实现。

首先需安装 JDK，Eclipse，用户应有 Azure 账号，这些基础部分不再赘述。关于 Azure 账号申请，请登录官方网页或者联系 IT 相关管理人员（如果公司已购买 Azure 服务） 

1.  打开 Eclipse，安装 Azure for Eclipse 插件。可使用 Eclipse 的各个不同版本或发行版本，比如 Spring Tool Suite， JBOSS IDE， Oracle IDE， IBM IDE 等，只要基于 Eclipse，安装过程大同小异。  
2. 选择 “Install New Software", 输入 Azure 插件的[安装地址](http://dl.msopentech.com/eclipse)，选择确定，在出现软件选择时选择全部安装，接受全部条款，然后一直点击 next，直到安装完成，Eclipse 会重启一次。  

    ![安装地址][77]

    ![全部安装][78]

3.  安装完成后，开始创建第一个 Azure Java 项目，因为本次主要展示如何将 Java 应用程序部署到云服务，所以用户需准备一个 WAR 包；如果没有，Azure 自带了一个测试的 helloworld.war，来进行简单的测试。打开 Eclipse，选择 New project，找到 Azure deployment project，选择新建项目。  

    ![新建项目][79]

4.  给项目命名，然后选择下一步：  

    ![下一步][80]

5.  在这个页面，需要指定部署到云服务中的 JDK，应用服务器以及应用。那么先来看一下 JDK 部分，有几种不同的选项，第一种是可以部署本地的 JDK 到云端，比如 Oracle 的 JDK 1.8；可以部署第三方的、从云端直接下载的 JDK，目前只支持 OpenJDK；可以制定一个远端的站点进行下载，但必须要注意的是，远端的 JDK 必须是 zip 包，因为不属实脚本只负责将 JDK 解开。本示例选择将本地的 Oracle JDK 1.8.0-60 部署到云端：  

    ![部署到云服务中的 JDK][81]

6.  第二个页面是选择需要部署的应用服务器，目前有多种服务器可供选择，常见的 Tomcat，Jetty，JBoss，GlassFish 等都在列表中。本例选择 Jetty 9 作为 Java 应用服务器；同样，如果有特殊设置，可以选择将本地的应用服务器上传到云端，只需要指定本地服务器的路径即可。  

    ![部署应用服务器][82]

7.  最后，选择需要部署的 Java Web 应用程序，它是一个标准的 war 包。默认情况下，Azure 应用程序会提供一个非常简单的 HelloWorld 的 war 包，其基本功能是输出经典的 Hello Wolrd，如果只想测试一下部署过程，可以选择该部署包，本例中笔者会部署一个自己的测试包 Greenhouse.war：  

    ![HelloWorld][83]

    ![Greenhouse][84]

8.  点击完成，创建新的项目。创建完成后，会看到如下图所示的项目结构。cert 目录会存放一些项目需要的证书，比如远程桌面连接的证书，cloudtools 是一些发布和构建工具，deploy 里面是打包完成需要发布的包，workrole1 是一些启动运行脚本和示例 HelloWorld 包，另外三个文件是包的定义，云端服务配置文件和定义文件。  

    ![项目结构][85]

###<a id="developer-java-part-2"></a>3.2 第二部分

接上文

9. 发布前，需要对订阅做一些设置，因为默认情况下，Azure 的 service end 指向的是国际版 Azure 的站点。如果要将服务发布在 Azure 的中国站点，需要对其进行简单的设置：在 Eclipse 中，打开偏好设置（preference），找到 Azure，在 service endpoint 页面中，选择 ”windowsazure.cn（China)，选择确定：  
	
	 ![service end][86]

10. 回到项目，选择 myazuredeploy 并单击右键，选择 Azure，properties,第一项是选择是否配置远程访问，因为云服务底层实际上是 Windows Server，所以本处实际是配置 RDP 访问，可以在 Azure portal 直接配置，本例选择不配置；  

	![选择 Azure][87]

	![RDP 访问][88]

11. 第二项是 role 的定义，本处可以选择 VM 虚拟机的大小，以及在云服务中需要启动的实例个数，点击修改，修改实例个数为 2，云服务中实例 2 个级以上才有 SLA 保障；  

	![role][89]

12. 最后添加订阅。告知部署脚本，需部署该应用的订阅名称。单击按钮 “import from PUBLISH-SETTING file” 会自动跳转到中国版 Azure 的登录界面，输入 Azure 帐号密码，会自动下载和导入 setting 文件，如下图所示，完成后点击 OK 按钮退出；

	![登录界面][90]

	![下载和导入][91]

13. 回到项目，单击右键选择 Azure，选择第一项 “Deploy to Azure Cloud”，可看到在该界面中，已经列出了你的订阅、要部署到云端订阅的默认存储帐号、云服务等。由于本次是新部署，所以选择新建存储，将该部署所有的实例、应用存放到一个存储帐号下，选择 “New” 按钮，在弹出的界面中输入存储账号名称，选择 Location，需要注意如果希望应用部署在 East 或者 North，那么对应的后续配置都需要选择同样的地区：  

	![选择 Azure][92]

	![新建存储][93]

14. 同样，选择新建云服务，例子中名称为 myhouse，同样选择 China East 作为地区：  

	![新建云服务][94]

15. 配置完成后如下图所示，点击发布，那么部署程序自动创建存储账号，云服务，创建虚拟机，发布应用：  

	![发布][95]

	![发布应用][96]

16. 显示部署完成后，可以登陆到 Azure 管理门户，查看部署的云服务和实例情况，在云服务的仪表板上，可以找打站点的 URL，选择实例页面，也可以看到按照定义，已经为改云服务创建了 2 个实例，并分布在不同的容错域：  

	![仪表板][97]  

	![容错域][98]

17. 最后，可以测试一下发布成果，在浏览器中输入站点名称和应用名称，例如:http://XXXXX.chinacloudapp.cn/greenhouse/,就可以看到 Java web 服务正常工作如下：  

	![Java web 服务][99]

##<a id="azure-cloud-service-enhanced-content"></a>4.高级内容  

###<a id="azure-cloud-service-enhanced-content-non-persistent-vm"></a>4.1 云服务是非持久化虚拟机  

之前已经介绍了 Azure 云服务是非持久化虚拟机，应用程序是通过 CSPKG 和 CSCFG 文件，打包上传到 Azure 云端。  

任何针对 Azure 云服务本地磁盘的操作，都是无效的，需要把增量文件保存到 Azure 存储里。  

###<a id="azure-cloud-service-enhanced-content-add-external-components"></a>4.2 添加外部组件

开发 Azure 云服务时，会调用第三方外部类库，用户需要在编译、部署时，把这些类库编译到 CSPKG 文件里。 

打开 Visual Studio，将需要的 dll 类库包含在项目里 (including in Project)，并且将属性中的 ”Copy to Output Directory” 设置成 ”Copy Always”  

![Copy Always][100]

这样这个 LegacyCom.dll 类库就会编译到 CSPKG 文件里。  

###<a id="azure-cloud-service-enhanced-content-register-third-party"></a>4.3 注册第三方组件

在做 Web 应用部署的时候，经常会遇到需要额外安装的第三方软件，比如安装 Office 和 FTP Server。而 Azure 云服务无法安装软件： 
 
####4.3.1 使用 Startup 注册 COM 组件
Azure 提供了 role start up 功能，方便我们在云平台注册 COM 组件。

1. 使用 VS2010 新建一个 Azure 项目，并且添加一个 Web Application (过程略)。

2. 假设我们有一个 C++ 项目，能够生成名为 LegacyCOM.dll,这个 dll 有一个方法叫做 Greeting()。<br/>
具体调用是这样的：先实例化这个 dll 对象，然后调用 Greeting 方法，传入一串 string 进去，然后这个 dll 会返回一串string。<br/>
Azure Project 会根据 Web 服务器的操作系统的版本来注册不同版本的 LegacyCOM.dll，比如探测到服务器端 64 位的 Win7 操作系统，则会自动注册 64 位的 LegacyCOM.dll。

3. 我们在 WebRole Solution 里新建两个文件夹，分别叫"amd64"和"x86"。

4. 将 64 位版本的 LegacyCOM.dll 复制"amd64"文件夹下，将 32 位版本的 dll 复制到"x86"文件夹下。

5. 将这两个 dll 包含在 VS 项目里(include in project)，并且将属性中的"Copy to Output Directory"设置成"Copy always"。
	
	![Copy Always][100]

6. 在 Web Application 工程下新建文件，文件夹设置成 Register.cmd。

7. Register.cmd 的内容如下，主要的功能是在 Azure VM 上注册这个 dll 组件。

		echo off
		regsvr32.exe /s "%~dp0%PROCESSOR_ARCHITECTURE%\LegacyCOM.dll"

	如果发现服务器 OS 是 64 位的，Register.cmd 会注册**amd64 文件夹下的 dll,否则注册 x86 下的 dll。**

8. 将 Register.cmd 的属性中的 "Copy to Output Directory" 设置成 "Copy always"。 

	![Copy Always][101]

9. 修改 ServiceDefinition.csdef，在 ServiceDefinition 节点里添加 Startup 任务。



	![Copy Always][102]

	其中，commandLine 指向我们写好的 cmd 文件。需要注意的是，这个脚本必须是以 ANSI 格式编码的。

	executionContext：脚本的执行上下文，也就是这个脚本的执行权限。可以选项包括：

	* limited：默认值。表示使用和当前 Role 所在进程相同的权限来执行这个脚本。
	* elevated：表示使用本机管理员权限来执行这个脚本。当脚本进行了系统级别的操作，比如对 IIS 进行配置，就必须指定为 elevated。

	TaskType 的可选值有三种，含义如下：

	* Simple：默认值，只有当 Task 执行完成之后系统才会执行下一个 Task。当所有 Task 执行完成之后才会开始启动Role。
	* Background： startup 任务在后台运行，系统不会等待 Task 执行完成就开始启动 Role
	* Foreground： 和 Background 方式类似，系统不会等待 Task 执行完成就开始启动 Role。不同之处在于，只有当所有Task 都执行完成之后 Role 才可能被重启。<br/>

10. 在 Default.aspx 添加 TextBox,Button 和 Label，并且在 Button_Click 添加如下代码：

		LegacyCOMLib.Helper helper = new LegacyCOMLib.Helper();
		Message.Text = helper.Greeting(Username.Text);

11. 按 F5 运行 VS 工程，在 pre-build 过程中，Azure Project 会先侦察 CSDEF 是否有 Startup 节点，如果有的话则执行 CommandLine 指向的 cmd 文件。我们这里的 cmd 文件的功能是：根据操作系统的版本，自动注册所对应的dll。并且会弹出窗口，表明注册成功。我们按"OK"，让项目继续执行。

	![Copy Always][103]

12. 在 aspx 页面里，用户在 textbox 输入 name 的值，然后按"Greet me!"，dll 会调用自动返回一串字符串并且显示在页面上。
	
	![Copy Always][104]

通过 Startup 功能，用户还可以通过系统命令修改系统配置，或者通过 PowerShell 执行更加复杂的操作(比如使用 InstallUtil 注册 Windows Service)。另外和通过远程桌面手动修改系统配置不同，Startup 功能会在系统迁移、故障恢复的时候自动执行，因此可以作为持久化的操作。

再回顾一下 Windows Azure Startup Task

	<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
   		<WebRole name="WebRole1">
     		 <Startup>
        		 <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
         		</Task>
     		 </Startup>
  		 </WebRole>
	</ServiceDefinition>

在以上代码中，最核心的就是 Startup.cmd，这个 cmd 可以是 PowerShell scripts。在这个 cmd 文件中你可以执行您自己的逻辑，比如

* 注册一个 dll(regsvr32..)
* 注册一个 Windows Service (InstallUtil..)
* 其他 PowerShell scripts (比如安装其他第三方的组件，如 Crystal Report)

为什么需要 Startup Task 呢？

我们知道，Windows Azure VM 是非持久性的。在某些异常情况下(比如断电，硬件故障等)，某个正常工作的 Azure VM 会宕机而无法正常使用。后台的 Windows Azure Fabric Controller  会自动把该 VM 上的 Azure Apps 迁移到同一数据中心的另外一台正常的计算节点之上，保证服务的持久性和健壮性。

如果没有 StartUp Task，那在 Fabric Controller 迁移之后，Azure Apps 管理员不得不远程登录到新的 VM 上，然后手工执行 regsvr32 命令。否则 Azure Apps 无法正常运行(的确，因为在新的 Azure VM 上没有注册 COM 组件)。但这样岂不是非常麻烦：迁移之后还需要人工来注册，不能实现自动化。

有了 Startup Task 功能会在系统迁移、故障恢复的时候自动执行命令，因此每次修改都是持久性的，不需要人为干预操作。

###<a id="azure-cloud-service-enhanced-content-change-time-zone"></a>4.4 修改时区

本节内容需要使用到的工具有：

1. Azure Visual Studio 2013 Update4；<br/>
2. 安装 Azure SDK；<br/>
3. Azure China 账户。<br/>

接下来开始本章的内容：

1. 首先我们以管理员身份，运行 Visual Studio 2013；
2. 创建一个新的 Cloud Project，命名为 LeiCloudServiceTimeZone，并添加 Web Role；
3. 在 Project 的根目录下，创建一个新的文件，重命名为 changetimezone.cmd；
4. 在 Cloud Project 里，点击 changetimezone.cmd，选择右键属性，Copy to Output Directory 选择 Copy always。

	![Copy Always][105]

5. 在 changetimezone.cmd，设置如下逻辑：

		tzutil /s "China Standard Time" 

6. 修改 ServiceDefinition.csdef，修改 Startup 节点，让 Cloud Service 项目启动的时候，执行changetimezone.cmd 命令。如下图：

	![Copy Always][106]
	
7. 最后我们在 aspx 页面中，输出当前的系统时间：

		Label1.Text = DateTime.Now.ToString();

8. 最后我们部署 Cloud Service，显示如下：

	![Copy Always][107]

可以看到部署的当前时间，就是 UTC+8 北京时间。
###<a id="azure-cloud-service-enhanced-content-change-iis-mode"></a>4.5 修改 IIS 托管管道模式为 4.0 经典模式

一般情况下，Visual Studio 开发的 Cloud Service 在部署到 Azure PaaS 平台后，是 IIS 托管管道模式。有的时候，客户的 Web 应用程序需要 4.0 经典模式，就需要在启动 Web Role 时自定义 IIS 配置。

1. 在项目中添加引用：C:\Windows\System32\inetsrv\Microsoft.Web.Administration.dll。Microsoft.Web.Administration命名空间下，提供了一系列管理IIS的操作。

	![Copy Always][108]

2. 修改 ServiceDefinition.csdef，添加<Runtime executionContext="elevated"/>，使之有权限修改 IIS 配置。

	![Copy Always][109]

3. 修改 public override bool OnStart()方法（位于 WebRole.cs），调用 Microsoft.Web.Administration 命名空间下面的操作设置 IIS。参考以下示例代码（Cloud Service IIS 站点的名称格式为：RoleEnvironment.CurrentRoleInstance.Id + "_Web"）：


		using (ServerManager serverManager = new ServerManager())
      		{
          		var siteName = RoleEnvironment.CurrentRoleInstance.Id + "_Web";
          		Trace.Write(string.Format("Site Name: {0}.", siteName));
          		var siteApplication = serverManager.Sites[RoleEnvironment.CurrentRoleInstance.Id + "_Web"].Applications.First();

          		Trace.Write(string.Format("Application Pool: {0}.", siteApplication.ApplicationPoolName));

         		 var appPoolName = siteApplication.ApplicationPoolName;
          		var appPool = serverManager.ApplicationPools[appPoolName];

          		if (appPool != null)
          		{
              		appPool.ManagedPipelineMode = ManagedPipelineMode.Classic;
              		serverManager.CommitChanges();
          		}
      		}

4. 部署云服务，成功修改云服务 IIS 托管管道模式为 4.0 经典模式。

	![Copy Always][110]

	![Copy Always][111]
###4.6 云服务虚拟目录

本节介绍如何使用 Azure 的 Full IIS 模式部署多站点和虚拟目录。

首先新建一个 Azure Project，然后添加一个 ASP.NET 的 Web Role。

随后在 Solutio 里添加一个新的 Web Application，命名为 WebCRM。注意，这里是直接添加了一个 Web Application 而不是添加一个新的 Web Role。创建完毕后，修改 Web CRM 下的 default.aspx 页面，方便区分所浏览的页面属于哪个项目。修改如下：

![Copy Always][112]

然后在发布 Azure 项目之前需要手动编译一下新加入的 WebCRM 项目。这是因为在 Visual Studio 的项目依赖关系中并没有把这个 WebCRM 项目加入到 Windows Azure Roles 中，因此直接发布 Windows Azure 项目的话这个 WebCRM不会被自动编译。

然后我们按 F5 来调试效果。

![Copy Always][113]

最后我们把这个站点发布到 Windows Azure 平台上，通过远程桌面进入 Azure VM 的 IIS 查看，可以看到 IIS 中部署了两个 web 站点：

![Copy Always][114]

我们也可以通过 IE 浏览器访问发布后的 web 站点。

![Copy Always][115]

除此之外，用户还可以实现虚拟应用程序和虚拟目录的功能。打开 CSDEF 文件，在第一个 Site 节点加入VirtualApplication 节点，其 name 属性设置为 LocalCRM，PhysicalDirectory 属性设置为 WebCRM 项目的相对路径。这样，便可以通过这个 Web 站点的 LocalCRM 路径访问 WebCRM 页面。

然后再 VirtualApplication 节点下面加入虚拟目录，比如可以将 Scripts 和 Styles 路径分别指向 WebCRM 下面的 Scripts 和 Styles 路径。完成后的配置文件如下：

![Copy Always][116]

然后我们重新发布到 Azure 平台上。可以看到 LocalCRM 页面中显示的是运行于 WebCRM 中的内容。在 CSDEF 中配置的Virtual Application 和 Virtual Directory 都起了作用。

![Copy Always][117]

###<a id="azure-cloud-service-enhanced-content-speed-up-publish"></a>4.7 加快部署速度

####4.7.1 AzCopy 命令行工具

AzCopy 命令行工具，是经过优化的、高性能 Azure Storage 管理工具，可以管理 Azure Blob, File 和 Table。最新的下载地址是：[http://aka.ms/downloadazcopy](http://aka.ms/downloadazcopy)，下载后直接安装运行。

####4.7.2 Azure Blob Browser

上面介绍的 AzCopy 毕竟是命令行工具，如果对命令行工具不熟悉的话，可以使用 BlobBrowser。


BlobBrowser 是 GUI 图形界面的工具，背后其实是利用 AzCopy 命令行工具。

安装完 BlobBrowser 之后，可以在界面上进行配置。如下图：

![Copy Always][118]

上图中，点击 Add，增加新的存储账号信息：
	
在弹出窗口中，输入以下信息:

![Copy Always][119]

上图中，

* Name输入显示名称；

* Storage Account Name，输入存储账号名称；

* Storage Account Key，输入存储账号的密钥。

注意，国内由世纪互联运维的 Azure，需要在上图中勾选 Use China Endpoint。

 

登录完毕后，UI 结构如下，左侧为本地文件目录，右侧为 Azure Storage Container。

![Copy Always][120]


点击上图中的 Upload 就可以上传一个文件或者文件夹。

注意：文件夹默认是递归复制的，也就是把文件夹下的多级目录都上传到 Azure Storage 中。
	
上传完毕后，点击右侧列表中的文件。右键属性。如下图：

![Copy Always][121]

就可以看到这个文件的URL：

![Copy Always][122]

####4.7.3 结合 AzCopy，加快 Azure Cloud Service 部署速度。

1. 首先在 Azure Management Portal，创建一个空的 Cloud Service，如下图:

	![Copy Always][123]

2. 然后使用上面介绍的 AzCopy，将 CSPKG 和 CSDEF 文件，复制到 Azure Blob 中。

3. 回到 Management Portal，找到步骤 1 中创建 LeiPaaS 项目，选择 Configure，并点击Upload A New Production Deployment。如下图:

	![Copy Always][124]

4. 在弹出的窗口中，PACKAGE 和 CONFIGURATION 都选择 From Storage，浏览到我们在步骤 2 中上传的 CSPKG 和 CSDEF 文件。

	![Copy Always][125]

这样就会极大的加快部署Cloud Service的速度。

###<a id="azure-cloud-service-enhanced-content-config-ssl"></a>4.8 配置 SSL 证书

1. 创建证书；

	以管理员身份运行 CMD，使用 Makecert 命令，安装 Azure 证书。具体的命令如下：

		makecert -sky exchange -r -n "CN=<CertificateName>" -pe -a sha1 -len 2048 -ss My "<CertificateName>.cer"

	比如创建一个证书名叫 SSLCertificate:

		makecert -sky exchange -r -n "CN=SSLCertificate" -pe -a sha1 -len 2048 -ss My "SSLCertificate.cer"

2. 导出私钥；

	在运行里，输入 mmc.exe。

	在控制台，选择文件->添加删除管理单元，添加"证书"，选择"我的用户账户"。

	展开"证书-当前用户"->"个人"->"证书"。选择 SSLCertificate，右键，"所有任务"，点击"导出"。

	选择"是，导出私钥”，输入自己的私钥(本文使用 123456)。然后将 pfx 保存到 C 盘的根目录，重命名为 PrivateCertificate.pfx。

3. 创建空的 Cloud Service；

	在 Azure Portal，创建一个空的云服务，命名为 LeiSSL。如下图：

	![Copy Always][126]

4. 点击 Cloud Service，选择步骤三创建的 LeiSSL，然后点击 Certificates，最后选择 Upload a certificate；

	![Copy Always][127]

	上传完毕后，可以查看到 ThumbPrint，如下图:

	![Copy Always][128]

5. 将保存在 C:\PrivateCertificate.pfx的文件进行上传，同时输入步骤二里导出的私钥(123456)。如下图：

	![Copy Always][129]

6. 然后在本地，使用管理员身份，运行 VS2013。创建一个 Cloud Project，并且添加 ASP.NET Web Role；

7. 在项目文件里，选择 WindowsAzure1，Role，双击 WebRole1；

	![Copy Always][130]

8. 在 Certificates 菜单栏，点击 Add Certificate，将步骤 4 的 Thumbprint 输入到单元格中；

	![Copy Always][131]

9. 再点击 Endpoints 菜单栏，点击 Add Certificate，设置名称为 HttpsEndpoint，Protocol 设置为 https，SSL Certificate Name 选择步骤 8 中创建的 Certificate1；

	![Copy Always][132]

10. 设置完毕后，用 VS2013 发布 Cloud Service；


11. 可以通过 [https://leissl.chinacloudapp.cn:8080/](https://leissl.chinacloudapp.cn:8080/ "https://leissl.chinacloudapp.cn:8080/") 来访问发布成功的应用程序。

	![Copy Always][133]


###<a id="azure-cloud-service-enhanced-content-internal-public-ip"></a>4.9 固定云服务内网 IP 和公网 IP
####4.9.1 管理 Virtual IP Address(公网 IP 地址)
可通过 Azure PowerShell 来申请 Virtual IP Address。

第一次运行 Azure PowerShell 时，需要在本地创建证书文件，以便本地计算机和 Azure 建立可靠的安全连接。

1. 以管理员身份，运行 Azure PowerShell，下载 publishsettings 文件；

		Get-AzurePublishSettingsFile -Environment AzureChinaCloud

	如果不想运行 Azure PoweShell 的话，请在浏览器中输入地址：[http://go.microsoft.com/fwlink/?LinkID=301776](http://go.microsoft.com/fwlink/?LinkID=301776 "http://go.microsoft.com/fwlink/?LinkID=301776")，在登陆框中，输入你的OrgID和密码。

2. 将 publishsettings 下载到本地磁盘，然后执行上传 publishsettings 命令；

		Import-AzurePublishSettingsFile <PathToFile>

	上面步骤 1、2 执行成功后，下次运行 Azure PowerShell 不必再次运行上面的运行，接下来可以运行命令。

3. 创建新的存储账号(步骤略)，选择当前的订阅，并设置存储账号；

		Set-AzureSubscription -SubscriptionName '[SubscriptionName]' -CurrentStorageAccount '[StorageName]'

4. 在上海数据中心(China East)，获得固定的 Public IPV4 地址；

		$NginxReservedIP = New-AzureReservedIP -ReservedIPName 'NginxPublicIP' -Label 'NginxPublicIP' -Location 'China East'
	查看这个 IP 地址

		Get-AzureReservedIP -ReservedIPName 'NginxPublicIP'

5. 创建虚拟网络 Virtual Network，命名为 MyVNet (位置选择 China East)。注意 Virtual Network 不能属于地缘组里。

	* MyVNet IP Rang 为 10.0.0.0-10.0.0.255，
	* 同时创建 2 个 Subnet：Nginx-subnet 和 Nodejs-subnet

	![Copy Always][134]

6. 通过模糊查询，查询到 CentOS 7.0 镜像；

		$imageList = Get-AzureVMImage `
		| where {$_.ImageName -like "*CentOS-70*"}

		$image=$imageList[0]
7. 创建 3 台虚拟机：
	* DNS 为 MyNginx，并且绑定 Public IP (NginxPublicIP)；
	* 机器名分别为 Nginx01，Nginx02 和 Nginx03；
	* 三台机器加入虚拟机网络 MyVNet。子网为 Nginx-subnet (10.0.0.0-10.0.0.127)，设置内网IP分别为10.0.0.4，10.0.0.5和10.0.0.6；
	* 虚拟机大小为 Large；
	* 管理员用户名为：adminuser。 密码为：MyVM@6789；
	* 高可用性集名称为：NginxAvbSet；
	* 并设置该虚拟机的时区为 UTC+8 时区(北京时间)。

	创建第 1 台虚拟机(Nginx01，内网 IP 是 10.0.0.4)的命令如下：

			New-AzureVMConfig -Name 'Nginx01' -InstanceSize 'Large' -ImageName $image.ImageName  -AvailabilitySetName 'NginxAvbSet' ` | Add-AzureProvisioningConfig -Linux -LinuxUser 'adminuser' -Password 'MyVM@6789' -TimeZone 'China Standard Time' | Set-AzureSubnet -SubnetNames 'Nginx-subnet' | Set-AzureStaticVNetIP -IPAddress '10.0.0.4' | New-AzureVM -ServiceName 'MyNginx' -VNetName 'MyVNet' –ReservedIPName 'NginxPublicIP' -Location 'China East'
	创建第 2 台虚拟机(Nginx02，内网 IP 是 10.0.0.5)的命令如下：

			New-AzureVMConfig -Name 'Nginx02' -InstanceSize 'Large' -ImageName $image.ImageName  -AvailabilitySetName 'NginxAvbSet' ` | Add-AzureProvisioningConfig -Linux -LinuxUser 'adminuser' -Password 'MyVM@6789' -TimeZone 'China Standard Time' | Set-AzureSubnet -SubnetNames 'Nginx-subnet' | Set-AzureStaticVNetIP -IPAddress '10.0.0.5' | New-AzureVM -ServiceName 'MyNginx' -VNetName 'MyVNet' 
	创建第 3 台虚拟机(Nginx03，内网 IP 是 10.0.0.6)的命令如下：

			New-AzureVMConfig -Name 'Nginx03' -InstanceSize 'Large' -ImageName $image.ImageName  -AvailabilitySetName 'NginxAvbSet' ` | Add-AzureProvisioningConfig -Linux -LinuxUser 'adminuser' -Password 'MyVM@6789' -TimeZone 'China Standard Time' | Set-AzureSubnet -SubnetNames 'Nginx-subnet' | Set-AzureStaticVNetIP -IPAddress '10.0.0.6' | New-AzureVM -ServiceName 'MyNginx' -VNetName 'MyVNet' 
 
8. 以上介绍的是创建 Linux 虚拟机，接下来介绍一下如何使用 PowerShell，创建 Windows 虚拟机。从上面的步骤 6开始，通过模糊查询，查询到 Windows Server 2012虚拟机；

		$imageList = Get-AzureVMImage `
		| where {$_.ImageName -like "*Windows-Server-2012-Datacenter*"}

		$image=$imageList[0]
或者通过精确查询，查询到 Windows Server 2008 R2 SP 中文版 OS；

		$imageList = Get-AzureVMImage `
		| where {$_.ImageName -eq "55bc2b193643443bb879a78bda516fc8__Win2K8R2SP1-Datacenter-201503.01-zh.cn-127GB.vhd"}

		$image=$imageList[0]
55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-201504.01-zh.cn-127GB.vhd

9. 创建 2 台虚拟机。

	* DNS 为 LeiVM，并且绑定 Public IP (NginxPublicIP)；
	* 机器名分别为 LeiVM01，LeiVM02；
	* 三台机器加入虚拟机网络 MyVNet。子网为 subnet-1 (10.0.0.0-10.0.0.127)，设置内网 IP 分别为 10.0.0.4，10.0.0.5；
	* 虚拟机大小为 Large；
	* 管理员用户名为：adminuser。 密码为：MyVM@6789；
	* 高可用性集名称为：LeiAvbSet。

	创建 LeiVM01 的 PowerShell 如下：

			New-AzureVMConfig -Name 'LeiVM01' -InstanceSize Large -ImageName $image.ImageName -AvailabilitySetName 'LeiAvbSet' ` | Add-AzureProvisioningConfig -Windows -AdminUsername 'adminuser' -Password 'MyVM@6789' | Set-AzureSubnet -SubnetNames 'Subnet-1' | Set-AzureStaticVNetIP -IPAddress '10.0.0.4' | New-AzureVM -ServiceName 'LeiVM' -VNetName 'MyVNet' –ReservedIPName 'NginxPublicIP' -Location 'China East'
	创建 LeiVM02 的 PowerShell 如下：

			New-AzureVMConfig -Name 'LeiVM02' -InstanceSize Large -ImageName $image.ImageName -AvailabilitySetName 'LeiAvbSet' ` | Add-AzureProvisioningConfig -Windows -AdminUsername 'adminuser' -Password 'MyVM@6789' | Set-AzureSubnet -SubnetNames 'Subnet-1' | Set-AzureStaticVNetIP -IPAddress '10.0.0.4' | New-AzureVM -ServiceName 'LeiVM' -VNetName 'MyVNet'
	创建一台 SQL Server 2012 SP1 的 Azure Virtual Machine，并设置该虚拟机的时区为 UTC+8 时区(北京时间)，同时关闭该虚拟机的自动更新功能。

			$imageList = Get-AzureVMImage | where {$_.ImageName -eq "74bb2f0b8dcc47fbb2914b60ed940c35__SQL-Server-2012SP1-Enterprise-SQL11-SP1-CU3-11.0.3350.0-Win2012-ENU"}

			$image=$imageList[0]

			New-AzureVMConfig -Name 'LeiSQLVM01' -InstanceSize 'Medium' -ImageName $image.ImageName  -AvailabilitySetName 'LeiSQLAvbSet' ` | Add-AzureProvisioningConfig -Windows -AdminUsername 'azureadmin' -Password 'MyVM@6789' -TimeZone 'China Standard Time' -DisableAutomaticUpdates | Set-AzureSubnet -SubnetNames 'SQL-Subnet' | Set-AzureStaticVNetIP -IPAddress '10.0.0.132' | New-AzureVM -ServiceName 'LeiSQLCS' -VNetName 'LeiSQLAlwaysOnVNet' -Location 'China East'
	此 Azure PowerShell 可以导出所有的 Azure VM Template Name，方便以后使用：

		$images = Get-AzureVMImage
		$count = $images.Count
		for($i=0;$i -lt $count;$i++){ $i.ToString() + " : " + $images[$i].ImageName; }

####4.9.2 管理 Azure Virtual Network

创建一个新的 Azure Virtual Network，命名为 PaaSVNet。

增加 2 个 subnet，命名为 Web-Subnet 和 DB-Subnet，如下图:

![Copy Always][135]

**Web-Subnet IP Rang为10.0.0.0-10.0.0.127**

**DB-Subnet IP Rang为10.0.0.128-10.0.0.255**
####4.9.3 配置 Azure Cloud Service
新建一个 Cloud Service，项目命名为 LeiPaaSReservedIP，只添加 WebRole 即可，截图略。

1. 将 Instance Count 数量修改为 2；
2. 修改 ServiceConfiguration.Cloud.cscfg 文件配置，增加以下内容：

![Copy Always][136]

上图中的设置中：

(1) Instance Count 为 2，需要 2 个 Instance；<br/>
(2) VirtualNetworksSite 节点，设置为虚拟机网络名称，PaaSVNet；<br/>
(3) InstanceAddress 节点设置为 WebRole1；<br/>
(4) Subnet 设置为步骤二中的子网名称，为 Web-Subnet；<br/>
(5) ReservedIP 设置为我们在步骤一中的 Reserved IP Name (LeiTestVMReservedIP)。<br/>

然后将 Cloud Service 部署到 Azure 平台上，观察 Reserved IP Address，已经使用在步骤一的 Public IP：191.234.18.61)。

![Copy Always][137]

然后观察虚拟网络 PaaSVNet 中的 Private IP 使用情况。

![Copy Always][138]

Web Role 已经加入了 Web-Subnet 子网。

因为将 Instance Count 设置为 2，这 2 个 PaaS Instance 占用了 10.0.0.4 和 10.0.0.5 这 2 个 IP 地址。
###<a id="azure-cloud-service-enhanced-content-log"></a>4.10 收集日志记录
1. 先创建一个 Cloud Project 并且添加一个 Web Role；
2. 在 Default.aspx 里添加一个如下内容：

		<asp:TextBox ID="txbInput" runat="server"></asp:TextBox>
       
		<asp:Button ID="btnOK" runat="server" Text="确认" onclick="btnOK_Click" />

3. 在 Default.aspx.cs 添加如下代码：

		public override bool OnStart()
        {
            // 获取用于Windows Azure诊断的默认初始配置

            DiagnosticMonitorConfiguration diagConfig = DiagnosticMonitor.GetDefaultInitialConfiguration();
            diagConfig.Logs.ScheduledTransferLogLevelFilter = LogLevel.Verbose;

            // 制定预定传输间隔
            diagConfig.Logs.ScheduledTransferPeriod = System.TimeSpan.FromMinutes(1);

            // 制定预定传输间隔
            DiagnosticMonitor.Start("Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString", diagConfig);

            // For information on handling configuration changes
            // see the MSDN topic at http://go.microsoft.com/fwlink/?LinkId=166357.

            return base.OnStart();

        }
4. 在 WebRole.cs 中添加如下代码：

        System.Diagnostics.Trace.WriteLine(System.DateTime.Now.ToLongTimeString() + "Page_Load is called");
        }

        protected void btnOK_Click(object sender, EventArgs e)
        {
            string inputValue = txbInput.Text.Trim();

            if (!string.IsNullOrEmpty(inputValue))
            {
                System.Diagnostics.Trace.WriteLine(System.DateTime.Now.ToLongTimeString() + "Your Input is " + inputValue);
            }
        }
5. 分析代码：

	我们需要关心的代码主要有：

	diagConfig.Logs.ScheduledTransferLogLevelFilter =LogLevel.Verbose;

	该代码设置了日志过滤级别。有些情况下我们只关心严重的错误，这时候我们可以设置过滤级别为 LogLevel.Error 或者LogLevel.Critical。这样只有符合条件的日志才会被传输到 cloud storage 上。

	diagConfig.Logs.ScheduledTransferPeriod = System.TimeSpan.FromMinutes(1);

	这段代码设置了传输间隔。这里我们设置为每隔一分钟传输一次日志。

6. 修改配置文件；

	Cloud Project-->Roles-->右键-->属性

	![Copy Always][139]

	修改Enable Diagnostics，输入Azure Storage Account Name和Account Key

	把Azure诊断内容保存到Windows Azure Storage里。
7. 最后把 Web 站点发布到 Windows Azure 托管服务上(过程略)，并且登陆该站点；
8. 使用 Visual Studio 中的 Server Explorer，右键-->New Account, Add Account Name 和 Account Key，可以在 threestone 帐号下看到 WADLogsTable 看到一条记录：

	![Copy Always][140]

	可以看到之前在代码里添加的跟踪信息已经被记录到了Table Storage中了。实际应用中可以通过 try-catch 来捕获异常，并且添加错误日志。这些日志能够被传送到 cloud storage 中以便分析问题。

###<a id="azure-cloud-service-enhanced-content-interactive"></a>4.11 Web Role 和 Worker Role 交互

[Azure 云服务 (11) PaaS之Web Role, Worker Role(上)](http://www.cnblogs.com/threestone/p/3410510.html)
[Azure 云服务 (12) PaaS之Web Role, Worker Role, Azure Storage Queue(下)](http://www.cnblogs.com/threestone/p/4201065.html)  
我们知道，在 Azure PaaS，有一个非常重要的概念，就是 Web Role 和 Worker Role。

在创建 Azure Cloud Service 的时候，会提示如下的选项：

![Copy Always][141]

* Web Role 是响应客户端的HTTP请求
* Worker Role 则是在后台执行的应用程序，概念上类似于 Windows Service。
* Web Role 可以通过 Azure Storage 中的 Queue，向 Worker Role 发送队列消息 (Queue.AddMessage())，以便让 Worker Role 在后端执行自己需要的逻辑。

注意：上图中 Worker Role 分为三类。本节会介绍 PaaS Worker Role。

为什么微软要有 Web Role/Worker Role？它的好处在哪里？

举例说明。比如有一个信息管理系统，需要上传 Excel 文档来进行解析和处理，从软件设计的角度来说有 2 种方法来解决

(1) 在 ASP.NET 应用程序里新建个 upload control,在 upload control 里面写函数：一旦 Excel 文件上传完毕，则在.cs文件执行对于 Excel 的处理工作。

但这样会有一个缺点，如果 Excel 文件里包含的内容非常大的话，需要时间来处理这些内容，所以前台的 ASP.NET 的页面会停滞或者无响应。虽然我们也可以通过增加 progressbar 或者 loading 图片来增强用户的体验，但是从软件设计上来说不是最好的。

(2) 前端还是用原来的处理方式，使用 upload control。服务器端增加一个 Windows Service，时序的查询某一个文件夹，一旦发现前端页面上传了一个 Excel 文件，则 Windows Service 执行处理 Excel 的工作。这样前端的页面会及时的响应并且得到更好的用户体验。

但是这还是有一个缺陷，前端的页面和windows service是一对一的关系，如果附件上传的数量很大的话会出现Windows Service来不及处理的情况。
　　
(3)有了 Worker Role，可以让一个 ASP.NET 页面后端有多个 Worker Role 来进行分布式的计算，是一对多的关系，能够有效的利用云上的计算资源，Worker Role 可以处理高负载的数据访问。

这样的架构好处有两点：

1. **异步处理**，Web Role 只响应客户端的 HTTP 请求，进行快速的响应。而 Worker Role 在后端处理 Web Role 发送过来的消息(Queue)，两者是松耦合的。

	在传统的 Web 应用中，如果我们把复杂的处理逻辑写在 ASPX 页面，则 ASPX 页面可能会停滞，造成的用户体验不好。Azure PaaS 使用 Web Role 和 Worker Role，Web Role 只关注于响应客户端的 HTTP 请求；而 Worker Role 可以在后端处理业务逻辑，进行异步处理。
2. Web Role 和 Worker Role 的关系是**多对多**的，比如我可以在 Web Role 的配置中，设置 Instance count 为10。如下图：

	![Copy Always][142]

	在 Worker Role 的配置中，设置 Instance count 为 3

	![Copy Always][143]


这种架构就好比一个餐厅，里面有 10 个服务员( Web Role )和 3 个厨师( Worker Role )。

* 服务员(Web Role)只负责招待客人(响应客户端请求)，并将客人的点菜信息通过队列消息交给厨房(Queue.AddMessage())；
* 厨师(Worker Role)读取点菜信息(Queue.GetMessage)，随后负责烧菜做饭(进行后端逻辑)，当饭菜准备完毕后，则删除点菜信息；
* 如果前端压力过大(客户端请求过多)，则 Web Role 可以横向扩展；
* 如果后端压力过大(后端逻辑处理过多)，则 Worker Role 也可以横向扩展；
* 这种松耦合，多对多的关系非常适合企业级应用架构。

可以看到，Web Role和Worker Role的进行通信的重要途径是通过Queue，了解Queue对于Azure PaaS架构设计非常重要。
<!--image reference-->
[6]: ./media/azure-cloud-services-user-manual-part-2/6.png
[7]: ./media/azure-cloud-services-user-manual-part-2/7.png
[8]: ./media/azure-cloud-services-user-manual-part-2/8.png
[9]: ./media/azure-cloud-services-user-manual-part-2/9.png
[10]: ./media/azure-cloud-services-user-manual-part-2/10.png
[11]: ./media/azure-cloud-services-user-manual-part-2/11.png
[12]: ./media/azure-cloud-services-user-manual-part-2/12.png
[13]: ./media/azure-cloud-services-user-manual-part-2/13.png
[14]: ./media/azure-cloud-services-user-manual-part-2/14.png
[15]: ./media/azure-cloud-services-user-manual-part-2/15.png
[16]: ./media/azure-cloud-services-user-manual-part-2/16.png
[17]: ./media/azure-cloud-services-user-manual-part-2/17.png
[18]: ./media/azure-cloud-services-user-manual-part-2/18.png
[19]: ./media/azure-cloud-services-user-manual-part-2/19.png
[20]: ./media/azure-cloud-services-user-manual-part-2/20.png
[21]: ./media/azure-cloud-services-user-manual-part-2/21.png
[22]: ./media/azure-cloud-services-user-manual-part-2/22.png
[23]: ./media/azure-cloud-services-user-manual-part-2/23.png
[24]: ./media/azure-cloud-services-user-manual-part-2/24.png
[25]: ./media/azure-cloud-services-user-manual-part-2/25.png
[26]: ./media/azure-cloud-services-user-manual-part-2/26.png
[27]: ./media/azure-cloud-services-user-manual-part-2/27.png
[28]: ./media/azure-cloud-services-user-manual-part-2/28.png
[29]: ./media/azure-cloud-services-user-manual-part-2/29.png
[30]: ./media/azure-cloud-services-user-manual-part-2/30.png
[31]: ./media/azure-cloud-services-user-manual-part-2/31.png
[32]: ./media/azure-cloud-services-user-manual-part-2/32.png
[33]: ./media/azure-cloud-services-user-manual-part-2/33.png
[34]: ./media/azure-cloud-services-user-manual-part-2/34.png
[35]: ./media/azure-cloud-services-user-manual-part-2/35.png
[36]: ./media/azure-cloud-services-user-manual-part-2/36.png
[37]: ./media/azure-cloud-services-user-manual-part-2/37.png
[38]: ./media/azure-cloud-services-user-manual-part-2/38.png
[39]: ./media/azure-cloud-services-user-manual-part-2/39.png
[40]: ./media/azure-cloud-services-user-manual-part-2/40.png
[41]: ./media/azure-cloud-services-user-manual-part-2/41.png
[42]: ./media/azure-cloud-services-user-manual-part-2/42.png
[43]: ./media/azure-cloud-services-user-manual-part-2/43.png
[44]: ./media/azure-cloud-services-user-manual-part-2/44.png
[45]: ./media/azure-cloud-services-user-manual-part-2/45.png
[46]: ./media/azure-cloud-services-user-manual-part-2/46.png
[47]: ./media/azure-cloud-services-user-manual-part-2/47.png
[48]: ./media/azure-cloud-services-user-manual-part-2/48.png
[49]: ./media/azure-cloud-services-user-manual-part-2/49.png
[50]: ./media/azure-cloud-services-user-manual-part-2/50.png
[51]: ./media/azure-cloud-services-user-manual-part-2/51.png
[52]: ./media/azure-cloud-services-user-manual-part-2/52.png
[53]: ./media/azure-cloud-services-user-manual-part-2/53.png
[54]: ./media/azure-cloud-services-user-manual-part-2/54.png
[55]: ./media/azure-cloud-services-user-manual-part-2/55.png
[56]: ./media/azure-cloud-services-user-manual-part-2/56.png
[57]: ./media/azure-cloud-services-user-manual-part-2/57.png
[58]: ./media/azure-cloud-services-user-manual-part-2/58.png
[59]: ./media/azure-cloud-services-user-manual-part-2/59.png
[60]: ./media/azure-cloud-services-user-manual-part-2/60.png
[61]: ./media/azure-cloud-services-user-manual-part-2/61.png
[62]: ./media/azure-cloud-services-user-manual-part-2/62.png
[63]: ./media/azure-cloud-services-user-manual-part-2/63.png
[64]: ./media/azure-cloud-services-user-manual-part-2/64.png
[65]: ./media/azure-cloud-services-user-manual-part-2/65.png
[66]: ./media/azure-cloud-services-user-manual-part-2/66.png
[67]: ./media/azure-cloud-services-user-manual-part-2/67.png
[68]: ./media/azure-cloud-services-user-manual-part-2/68.png
[69]: ./media/azure-cloud-services-user-manual-part-2/69.png
[70]: ./media/azure-cloud-services-user-manual-part-2/70.png
[71]: ./media/azure-cloud-services-user-manual-part-2/71.png
[72]: ./media/azure-cloud-services-user-manual-part-2/72.png
[73]: ./media/azure-cloud-services-user-manual-part-2/73.png
[74]: ./media/azure-cloud-services-user-manual-part-2/74.png
[75]: ./media/azure-cloud-services-user-manual-part-2/75.png
[76]: ./media/azure-cloud-services-user-manual-part-2/76.png
[77]: ./media/azure-cloud-services-user-manual-part-2/77.png
[78]: ./media/azure-cloud-services-user-manual-part-2/78.png
[79]: ./media/azure-cloud-services-user-manual-part-2/79.png
[80]: ./media/azure-cloud-services-user-manual-part-2/80.png
[81]: ./media/azure-cloud-services-user-manual-part-2/81.png
[82]: ./media/azure-cloud-services-user-manual-part-2/82.png
[83]: ./media/azure-cloud-services-user-manual-part-2/83.png
[84]: ./media/azure-cloud-services-user-manual-part-2/84.png
[85]: ./media/azure-cloud-services-user-manual-part-2/85.png
[86]: ./media/azure-cloud-services-user-manual-part-2/86.png
[87]: ./media/azure-cloud-services-user-manual-part-2/87.png
[88]: ./media/azure-cloud-services-user-manual-part-2/88.png
[89]: ./media/azure-cloud-services-user-manual-part-2/89.png
[90]: ./media/azure-cloud-services-user-manual-part-2/90.png
[91]: ./media/azure-cloud-services-user-manual-part-2/91.png
[92]: ./media/azure-cloud-services-user-manual-part-2/92.png
[93]: ./media/azure-cloud-services-user-manual-part-2/93.png
[94]: ./media/azure-cloud-services-user-manual-part-2/94.png
[95]: ./media/azure-cloud-services-user-manual-part-2/95.png
[96]: ./media/azure-cloud-services-user-manual-part-2/96.png
[97]: ./media/azure-cloud-services-user-manual-part-2/97.png
[98]: ./media/azure-cloud-services-user-manual-part-2/98.png
[99]: ./media/azure-cloud-services-user-manual-part-2/99.png
[100]: ./media/azure-cloud-services-user-manual-part-2/100.png
[101]: ./media/azure-cloud-services-user-manual-part-2/101.png
[102]: ./media/azure-cloud-services-user-manual-part-2/102.png
[103]: ./media/azure-cloud-services-user-manual-part-2/103.png
[104]: ./media/azure-cloud-services-user-manual-part-2/104.png
[105]: ./media/azure-cloud-services-user-manual-part-2/105.png
[106]: ./media/azure-cloud-services-user-manual-part-2/106.png
[107]: ./media/azure-cloud-services-user-manual-part-2/107.png
[108]: ./media/azure-cloud-services-user-manual-part-2/108.png
[109]: ./media/azure-cloud-services-user-manual-part-2/109.png
[110]: ./media/azure-cloud-services-user-manual-part-2/110.png
[111]: ./media/azure-cloud-services-user-manual-part-2/111.png
[112]: ./media/azure-cloud-services-user-manual-part-2/112.png
[113]: ./media/azure-cloud-services-user-manual-part-2/113.png
[114]: ./media/azure-cloud-services-user-manual-part-2/114.png
[115]: ./media/azure-cloud-services-user-manual-part-2/115.png
[116]: ./media/azure-cloud-services-user-manual-part-2/116.png
[117]: ./media/azure-cloud-services-user-manual-part-2/117.png
[118]: ./media/azure-cloud-services-user-manual-part-2/118.png
[119]: ./media/azure-cloud-services-user-manual-part-2/119.png
[120]: ./media/azure-cloud-services-user-manual-part-2/120.png
[121]: ./media/azure-cloud-services-user-manual-part-2/121.png
[122]: ./media/azure-cloud-services-user-manual-part-2/122.png
[123]: ./media/azure-cloud-services-user-manual-part-2/123.png
[124]: ./media/azure-cloud-services-user-manual-part-2/124.png
[125]: ./media/azure-cloud-services-user-manual-part-2/125.png
[126]: ./media/azure-cloud-services-user-manual-part-2/126.png
[127]: ./media/azure-cloud-services-user-manual-part-2/127.png
[128]: ./media/azure-cloud-services-user-manual-part-2/128.png
[129]: ./media/azure-cloud-services-user-manual-part-2/129.png
[130]: ./media/azure-cloud-services-user-manual-part-2/130.png
[131]: ./media/azure-cloud-services-user-manual-part-2/131.png
[132]: ./media/azure-cloud-services-user-manual-part-2/132.png
[133]: ./media/azure-cloud-services-user-manual-part-2/133.png
[134]: ./media/azure-cloud-services-user-manual-part-2/134.png
[135]: ./media/azure-cloud-services-user-manual-part-2/135.png
[136]: ./media/azure-cloud-services-user-manual-part-2/136.png
[137]: ./media/azure-cloud-services-user-manual-part-2/137.png
[138]: ./media/azure-cloud-services-user-manual-part-2/138.png
[139]: ./media/azure-cloud-services-user-manual-part-2/139.png
[140]: ./media/azure-cloud-services-user-manual-part-2/140.png
[141]: ./media/azure-cloud-services-user-manual-part-2/141.png
[142]: ./media/azure-cloud-services-user-manual-part-2/142.png
[143]: ./media/azure-cloud-services-user-manual-part-2/143.png

