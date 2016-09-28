<properties
	pageTitle="Azure 云服务操作手册 - 第一部分 | Azure"
	description=""
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

[Azure 云服务操作手册 - 第二部分](/documentation/articles/azure-cloud-services-user-manual-part-2)



##<a id="azure-cloud-services-related-content"></a>1. Azure 云服务相关服务  

###<a id="azure-cloud-services-network-bandwidth"></a>1.1 Azure 云服务的带宽问题  

####1.1.1 独享带宽与共享带宽 <br/>

在中国，互联网接入带宽的费用是非常昂贵的，费用大约是美国的 20 倍。

云计算适合的场景包括：开关模式、爆发增长模式等。**购买独享带宽不能体现云计算弹性的优势。**在购买了独享带宽的情况下，无论客户是否使用云计算，带宽成本都必须支付。如果用户购买了 200M 独享带宽，结果项目上线后发现用户量很少，互联网带宽闲置，但是这 200M 带宽费用还是必须支付的。

Azure 带宽虽然是共享带宽，其价格还是非常具有竞争优势的，**其带宽随着负载均衡服务器数量的增加而逐渐增加。**

实际项目上线后如果发现互联带宽不够，可以把更多的 Azure VM 加入到负载均衡器上。如果发现互联网带宽闲置，则将部分 Azure VM 关闭即可。

比如一个证券行业的客户，他们在业务高峰期的时候 ( 股票开市，早上 9：30-下午 3 点  )，利用约 100 台 Azure 虚拟机横向扩展的能力，来处理大量的客户端并发。在业务低谷的时候 ( 股票休市 )，利用少量的 Azure 虚拟机横向扩展，用来节省成本。**( Azure 具备 Auto Scaling 的能力，可以按照计划任务开启或者关闭多台 Azure 虚拟机，整个过程都是自动化完成的。**) 这种横向扩展的方式比该公司以往购买固定带宽的成本要大大的减少。

这种 Azure 虚拟机动态增加/减少的优势，可以帮助客户极大的减少成本。

####1.1.2 分析互联网带宽内容

场景：某客户，其应用需要支撑 10 万用户在线进行视频播放功能，因此需要独享带宽保证用户体验。

分析一下该场景。当浏览一个网站的时候，其内容可以分为以下几个部分：

**(1) 静态的 HTML 页面；**

**(2) 静态的文件，如视频、照片、文档等；**

**(3) 动态的编译页面，如 ASPX，JSP 等。**

当用户访问的视频都是走主机 ( Azure VM ) 带宽的话，毫无疑问主机带宽越大越好。

**请不要忘记，Azure Block Blob 每个文件提供 60MB/S 的互联网带宽，一个 Storage Account 提供 10GB/S 的互联网带宽。**

**Azure Block Blob 就类似于云网盘。**

我们只需要改变一下软件架构：

* 动态编译的页面还是走主机带宽；
* 诸如视频等静态文件保存到 Azure Block Blob；
* 静态的照片文件保存到 Azure Block Blob。

**通过将静态内容请求发送到 Azure Storage，将动态内容的请求发送到 Azure 云主机，就可以大大减少云主机独享带宽的的压力。**

  

###<a id="azure-cloud-services-what-is"></a>1.2 什么是 Azure 云服务  

通常开发 Web 应用程序时， IT 人员需要准备 Windows OS 或者 Linux OS 的 Web Server，安装相应的 Web 组件，比如 IIS, Tomcat 等，然后开发人员把相应的代码部署到 Web Server 上并进行配置。  

对于 IT 运维人员来说，Web Server 是 IaaS (Infrastructure-as-a-Service，基础设置即服务)，IT 运维人员需要维护 Web Server 的操作系统等内容。  

对于开发人员来说，Web Server 是 PaaS (Platform-as-a-Service，平台即服务)，开发人员只需要维护 Web 应用即可。底层的操作系统等交给 IT 运维人员。  

同样，Azure 云服务提供 PaaS 服务。开发人员只需要把开发的代码直接部署到 Azure Web 应用，无需管理操作系统，降低了管理成本。  

###<a id="azure-cloud-service-virtual-machine-relationship"></a>1.3 Azure 云服务和虚拟机的关系  

使用 Azure 云服务时常常碰到这样的问题： 

(1) 问题一：创建新的 Azure 虚拟机时，会同时创建同样名称的云服务。 报价里说虚拟机会收费，云服务也会收费。这样的话，使用虚拟机，是不是收取虚拟机 + 云服务 = 2 倍的费用？

(2) 问题二：使用 VS2013，将 asp.net 的应用程序部署到 Azure 的 PaaS 平台时，只有云服务，没有虚拟机，为什么呢?

首先，什么是云服务？  

云服务其实有两层含义：  

(1)	第一层含义，对于 IaaS 平台 (Azure 虚拟机) 来说，云服务是容纳虚拟机的容器 (container )。如下图  

![虚拟机的容器][1]

对于上图来说，云服务是一个容器，可以同时容纳两个虚拟机。  

新建虚拟机时，因为不存在容纳这个虚拟机的容器，所以 Azure 会默认创建一个新的云服务，然后将虚拟机加入到这个容器当中去。

那是否会收取云服务 + 虚拟机两份费用呢? 答案是否定的，Azure 只会收取虚拟机的费用。对于上图中的例子来说，Azure 只会收取 2 个虚拟机的费用。

(2)	第二层含义，对于 PaaS 云服务来说，云服务是一个多层的 Web 应用程序。  

本文主要介绍的是第二层含义，PaaS 服务，多层的 Web 应用程序。

用户可以定义前端的 Web Role，来响应客户端的请求；还可以定义后端的 Worker Role，来处理复杂的业务逻辑。

因为 Azure PaaS 平台使用的是 Web Role 和 Worker Role，并不存在任何的虚拟机。所以在使用 PaaS 平台的时候，不会创建虚拟机。

假设用户在使用 PaaS 平台的虚拟机，创建了 2 台 A2 的 Web Role 和 2 台 A2 的 Worker Role，那该用户需要支持的费用 = 2 * A2 单价 + 2 * A2 单价 = 4 * A2 的单价费用。

以下 Demo 介绍如何查看一个云服务，背后是 PaaS 服务，还是 IaaS 虚拟机呢?  

(1) 首先登陆 Azure [管理门户](https://manage.windowsazure.cn)；

(2) 点击云服务，如下图：  

   ![云服务][2]

   如果发现云服务，实例中显示生产、过渡，且名称无法点击跳转的，则这个服务器必定是 PaaS 云服务。

(3) 点击云服务，如下图：  

   ![点击云服务][3]

   如果发现云服务，实例中没有显示生产、过渡，且名称可以点击跳转的，则这个服务器必定是 IaaS 虚拟机。  

###<a id="azure-cloud-services-difference-from-other-compute-services"></a>1.4 Azure 云服务和其他计算服务的区别  

所谓计算服务，就是提供 CPU 和内存的计算能力。  

熟悉 Azure 平台的读者都知道，在 Azure 平台的计算服务分为三种： 
 
(1) Azure Web 应用  

(2) Azure 虚拟机  

(3) Azure 云服务  

对于一个企业级的用户，面对这三种不同的托管服务，应该选择哪一种服务，将现有的企业应用迁移到 Azure 云计算平台上呢？  

在这里介绍一下 Azure 三种服务的区别，如下图：   

![三种服务的区别][4]

以上三种计算服务，优势和使用场景如下表：  

<table width="100%" border="1" cellspacing="0" cellpadding="0">
	<tr>
		<td>计算服务</td>
		<td>优势、劣势和使用场景</td>
	</tr>
	<tr>
		<td>Azure 虚拟机</td>
		<td>
			<b>优势：</b>
			<p>1. 同时支持 Windows 和 Linux 虚拟机</p>
			<p>2. 支持托管机房 Hyper-V VHD 迁移到云端</p>
			<p>3. 支持虚拟网络，将企业内网与云端网络连接，实现混合云</p>
			<p>4. 支持高级存储 (SSD,固态硬盘)</p>
			<p>5. 单个虚拟机最大计算资源 16 Core / 112 GB</p><br />
			<b>劣势：</b>
			<p>1. 需要 IT 运维人员，管理操作系统</p>
			<p>2. 使用成本相对其他计算服务来说，比较昂贵</p><br />
			<b>使用场景：</b>
			<p>1. 适合传统应用的迁移</p>
			<p>2. 需要使用 Linux 操作系统的应用部署</p>
			<p>3. 将多个应用程序部署在同一台 VM 上，比如 SQL Server, MySQL, MongoDB, SharePoint</p>
			<p>4. 企业内网+云端的混合云部署</p>
		</td>
	</tr>
	<tr>
		<td>Azure Web 应用</td>
		<td>
			<b>优势：</b>
			<p>1. 在 Azure 上构建高度可扩展的 Web 站点</p>
			<p>2. 通过 FTP, Git 或者 TFS 快速部署</p><br />
			<b>劣势：</b>
			<p>1. 不支持管理操作系统</p>
			<p>2. 单个节点最大计算资源 4 Core / 7 GB</p>
			<p>3. 横向扩展最多 10 个节点负载均衡</p>
			<p>4. 国内由世纪互联运维的 Azure China，暂时不支持将 Web 应用加入到Azure 虚拟机虚拟网络里</p>
			<br />
			<b>使用场景：</b>
			<p>1. 新应用的开发部署</p>
			<p>2. 持续开发。直接从源代码库使用 Git 或者 Team Foundation 进行部署</p>
		</td>
	</tr>
	<tr>
		<td>云服务</td>
		<td>
			<b>优势：</b>
			<p>1. 利用丰富的 PaaS 环境，创建高度可用的，可扩展的应用程序和服务。支持高级的多层架构，自动部署和弹性计算</p>
			<p>2. 使用 Web Role 和 Worker Role，Web Role 可以响应前端的显示，而将复杂的业务放在后端进行处理</p>
			<p>3. 支持虚拟网络，将企业内网与云端网络连接，实现混合云</p>
			<p>4. 相比 Web 应用，可以使用 Startup Task 在 PaaS VM 注册第三方组件、安装外部软件</p>
			<p>5. 单个计算节点，最大计算资源 16 Core / 112 GB</p>
			<br />
			<b>劣势：</b>
			<p>1. 不支持管理操作系统</p>
			<p>2. 非持久化虚拟机</p>
			<br />
			<b>使用场景：</b>
			<p>1. 新应用的开发部署</p>
			<p>2. 企业内网+云端的混合云部署</p>
			<p>3. 多层的应用程序，每层都可以自我扩展。使用 Web Role 和 Worker Role，Web Role 可以响应前端的显示，而将复杂的业务放在后端进行处理。</p>
		</td>
	</tr>
</table>

###<a id="azure-cloud-service-supported-languages"></a>1.5 Azure 云服务支持的开发语言  

Azure Web 应用支持的开发语言包括：.NET, Java, PHP, Python, Node.js, Ruby。  

开发 Azure 云服务必须先安装 [Azure SDK](/downloads/?sdk=net)。

###<a id="azure-cloud-service-concurrent"></a>1.6 Azure 云服务如何解决大并发  

建议使用多台 Azure 云服务，利用横向扩展的方式来解决大量的并发。  

单个节点向上扩展是有限的。受限于现有的 CPU 制造技术，无法将大量的计算资源都堆积到 1 台 300 Core 甚至 400 Core 的计算节点上。对于需要大量计算资源的情况，可通过横向扩展的方法来解决。  

所谓横向扩展，就是由 1 个计算节点，横向扩展到多个计算节点上并行计算，比如 50 个、100 个计算节点。比如一个互联网业务需要大量的计算资源，那可以将这些计算需求由 100 台 4 Core 的计算节点进行并行计算。  

###<a id="azure-cloud-service-single-instance-size"></a>1.7 Azure 云服务单个实例大小 

Azure 云服务单个实例，分为 A 系列和 D 系列：  

[ 注意事项 ] Azure 云服务计算节点的 CPU 和 RAM 是固定搭配的，不可以按照用户的想法随意更改。  

####<a id="azure-cloud-service-a-series"></a>1.7.1 A 系列计算节点  

A系列计算节点的类型如下：  

<table width="100%" border="1" cellspacing="0" cellpadding="0">
	<tr>
		<td>虚拟机类型</td>
		<td>CPU</td>
		<td>内存</td>
	</tr>
	<tr>
		<td>A0</td>
		<td>共享</td>
		<td>768 MB</td>
	</tr>
	<tr>
		<td>A1</td>
		<td>1</td>
		<td>1.75 GB</td>
	</tr>
	<tr>
		<td>A2</td>
		<td>2</td>
		<td>3.5 GB</td>
	</tr>
	<tr>
		<td>A3</td>
		<td>4</td>
		<td>7 GB</td>
	</tr>
	<tr>
		<td>A4</td>
		<td>8</td>
		<td>14 GB</td>
	</tr>
	<tr>
		<td>A5</td>
		<td>2</td>
		<td>14 GB</td>
	</tr>
	<tr>
		<td>A6</td>
		<td>4</td>
		<td>28 GB</td>
	</tr>
	<tr>
		<td>A7</td>
		<td>8</td>
		<td>56 GB</td>
	</tr>
</table>

除了 A0 的虚拟机类型，它的 CPU 是和别的用户共享的。其他类型的虚拟机，比如 A1-A7，它的 CPU 是独占的，不是和别的用户共享的。 

####<a id="azure-cloud-service-d-series"></a>1.7.2 D 系列计算节点

D系列虚拟机的类型如下：  

<table width="100%" border="1" cellspacing="0" cellpadding="0">
	<tr>
		<td>虚拟机类型</td>
		<td>CPU</td>
		<td>内存</td>
	</tr>
	<tr>
		<td>D1</td>
		<td>1</td>
		<td>3.5 GB</td>
	</tr>
	<tr>
		<td>D2</td>
		<td>2</td>
		<td>7 GB</td>
	</tr>
	<tr>
		<td>D3</td>
		<td>4</td>
		<td>14 GB</td>
	</tr>
	<tr>
		<td>D4</td>
		<td>8</td>
		<td>28 GB</td>
	</tr>
	<tr>
		<td>D11</td>
		<td>2</td>
		<td>14 GB</td>
	</tr>
	<tr>
		<td>D12</td>
		<td>4</td>
		<td>28 GB</td>
	</tr>
	<tr>
		<td>D13</td>
		<td>8</td>
		<td>56 GB</td>
	</tr>
	<tr>
		<td>D14</td>
		<td>16</td>
		<td>112 GB</td>
	</tr>
</table>

相比 A 系列的 Azure 虚拟机，D 系列的优势在于：  

(1)  **单台 VM 更多的 CPU Core**  

   相比 A 系列单台 VM 最大 8 Core/56 GB RAM 的配置，D 系列虚拟机单台最大的配置为 16 Core/112 GB RAM

(2) **D 系列的 CPU 性能比 A 系列提升约 60%** 

   Azure A 系列的虚拟机，Intel E5 的 CPU 是经过调试的，性能是人为降低的。  
   因为 Azure 数据中心在建设时，有 AMD 的 CPU 和 Intel 的 CPU，为了让 Intel CPU 性能和 AMD CPU 性能接近，保证运算能力的一致性，人为降低了 Intel CPU 的计算能力。  
   所以用户如果用 Super PI 等测试 A 系列的虚拟机，会发现性能与物理机是有差距的。  
   最新的 D 系列虚拟机，100% 体现了 Intel E5 的处理能力，CPU 性能比 A 系列提升 60%。

(3) Azure 云服务所在的计算资源是无状态的  

   熟悉 Azure 虚拟机的用户都知道，Azure 虚拟机分为不同的类型，分别是 A 系列、D 系列和 DS 系列。  
   这里的 Azure 云服务单个实例大小，与虚拟机的类型和配置相同。但是请记住，Azure 云服务没有本地磁盘的概念。

   在下面的内容中，读者将看到，虽然可以通过 Visual Studio，将 Cloud Project 发布到 Azure 云服务，同时配置远程桌面连接，但是 Azure 云服务本地磁盘上的资源，都是非持久化保存的。

   如果需要保存持久化的文件，一定要使用 Azure 存储保存文件。

###<a id="azure-cloud-service-static-resource"></a>1.8 Azure 云服务使用静态资源  

有不少开发人员抱怨：我的 Web 项目文件包含了很多视频内容，大概 1 TB，发布到 Azure 云服务非常慢，应该怎么办？  

建议把 Web 项目文件的静态资源，比如图片、照片、视频等都保存在 Azure 存储里，这样的好处有以下几点：  

(1) 加快网站部署速度；  

   去掉静态资源(图片、照片、视频)的 Web 项目文件只包含了源代码信息，文件容量减小，上传速度加快。  

(2) 静态资源可以直接访问 Azure 存储。  

   通过将静态内容请求发送到 Azure 存储，将动态内容请求发送到 Azure 云主机，可以大大减少云主机独享带宽的压力。
 

###<a id="azure-cloud-service-cost-analysis"></a>1.9 Azure 云服务成本分析

Azure 云服务按照分钟计费，计费的单价为每小时。  

Azure 云服务运行时，会收取计算费用；关闭后，继续收取计算费用。

举例来说，比如某个客户采用 10 台 A7 配置的 Azure 云服务，对外提供服务，每天 10 小时。  

每天计算费用 = 10 (Azure 云服务数量) X 每台每小时费用 X 10小时  

###<a id="azure-cloud-service-stop-charging"></a>1.10 Azure 云服务停止计费  

最后强调一下，当通过 Azure 管理界面，关闭 Azure 云服务时，还是会继续计费。如下图：  

![计费][5]

在上图中，虽然 Web Role 和 Worker Role 状态变为“已停止”，但是云服务还是会继续收费。  

为了避免继续产生费用，请将不使用的 Azure 云服务删除，这样才不会继续收费。  

###<a id="azure-cloud-service-limits"></a>1.11 Azure 云服务限制  

(1) 单个 Azure 云服务大小。  

   单个 Azure 云服务实例，最大为 D14 (16 Core / 112 GB)。

(2) 只支持 Windows Server 2012 R2 操作系统。  

   Azure 云服务背后运行的是非持久化虚拟机，操作系统是 Windows Server 2012 R2。

(3) 如果客户需要的计算单元，必须使用 Linux 操作系统，请使用 Azure 虚拟机。

(4) 不支持安装软件。

   Azure 云服务背后运行的非持久化虚拟机，任何安装软件的操作都是没有意义的。

(5) 非持久化虚拟机。  

   虽然可以通过 Visual Studio，将云服务发布到 Azure 云服务，同时可以配置远程桌面连接。但是保存在 Azure 云服务本地磁盘上的资源，都是非持久化保存的。

(6) 如果需要保存持久化的文件，一定要使用 Azure 存储！  


<!--image reference-->
[1]: ./media/azure-cloud-services-user-manual-part-1/1.png
[2]: ./media/azure-cloud-services-user-manual-part-1/2.png
[3]: ./media/azure-cloud-services-user-manual-part-1/3.png
[4]: ./media/azure-cloud-services-user-manual-part-1/4.png
[5]: ./media/azure-cloud-services-user-manual-part-1/5.png


