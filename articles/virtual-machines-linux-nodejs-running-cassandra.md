<properties linkid="services-linux-cassandra-with-linux" urlDisplayName="Cassandra with Linux" pageTitle="Run Cassandra with Linux on Azure" metaKeywords="" description="Explains how to run a Cassandra cluster on Linux in Azure Virtual Machines." metaCanonical="" services="virtual-machines" documentationCenter="Node.js" title="Running Cassandra with Linux on Azure and Accessing it from Node.js" authors="" solutions="" manager="" editor="" />

# <span></span></a>在 Azure 上通过 Linux 运行 Cassandra 并从 Node.js 访问 Cassandra

**作者：** Hanu Kommalapati

## 目录

-   [概述][概述]
-   [Cassandra 部署示意][Cassandra 部署示意]
-   [复合部署][复合部署]
-   [Azure 虚拟机部署][Azure 虚拟机部署]
-   [任务 1：部署 Linux 群集][任务 1：部署 Linux 群集]
-   [任务 2：在每台虚拟机上设置 Cassandra][任务 2：在每台虚拟机上设置 Cassandra]
-   [任务 3：从 Node.js 访问 Cassandra 群集][任务 3：从 Node.js 访问 Cassandra 群集]
-   [结束语][结束语]

## <span id="overview"></span> </a>概述

Azure 通过允许对业务对象进行无模式存储的 Azure 表存储提供一个 NoSQL 数据库。可通过 Node.JS、.NET、Java 以及任何其他可使用 HTTP 和 REST 的语言使用此服务。不过，还存在一些其他的常用 NoSQL 数据库（如 Cassandra 和 Couchbase），这些数据库因其无状态云服务模型而无法在 Azure PaaS 上运行。Azure 虚拟机现在允许在 Azure 上运行这些 NoSQL 数据库，而无需更改代码库。本文旨在说明如何在虚拟机上运行 Cassandra 群集以及如何从 Node.js 访问该群集。本文未介绍适用于实际生产操作的 Cassandra 部署，其中用户需要使用关联的备份和恢复策略来查看多数据中心 Cassandra 群集。在本练习中，我们将使用 Ubuntu 12.04 版的 Linux 和 Cassandra 1.0.10；不过，可以针对任何 Linux 发行版调整此过程。

## <span id="schematic"></span> </a>Cassandra 部署示意

利用 Azure 虚拟机功能，可以在 Microsoft 公有云上运行 NoSQL 数据库（如 [Cassandra][Cassandra]），这就像在私有云环境中运行它们一样轻松，只不过特定于 Azure 虚拟机基础结构的虚拟网络配置不同。在撰写本文时，Cassandra 尚未作为 Azure 上的托管服务提供，因此在本文中，我们将着眼于如何在虚拟机上设置 Cassandra 群集以及如何从虚拟机中托管的其他 Linux 实例访问该群集。还可以从 PaaS 托管的 Web 应用程序或 Web 服务中使用显示的 node.js 代码段。Azure 的一个核心优势是，允许充分利用 PaaS 和 IaaS 的复合应用程序模型。

有两种适用于 Cassandra 应用程序环境的部署模型：独立虚拟机部署和复合部署。在复合部署中，将通过负载平衡器使用 Thrift 接口来从 PaaS 托管的 Azure Web 应用程序（或 Web 服务）使用虚拟机托管的 Cassandra 群集。即使每个 Cassandra 节点在出现密钥空间故障时将代理向其他对等节点发出的请求，负载平衡器也会帮助实现这些请求的入门级别的负载平衡。此外，负载平衡还会创建一个受防火墙保护的沙盒以便更好地控制数据。

## <span id="composite"></span> </a>复合部署

复合部署的目标是，最大程度地利用 PaaS 并将虚拟机的占用率保持绝对最小值，以便节省因对虚拟机进行基础结构管理所产生的开销。由于服务器管理开销，因此仅部署那些需要有状态的行为的组件，这些行为出于各种原因（包括上市时间、缺少对源代码的可见性以及对操作系统的较低级别的访问权）而无法轻易修改。

![复合部署图][复合部署图]

## <span id="deployment"></span> </a>Azure 虚拟机部署

![虚拟机部署][虚拟机部署]

在上面显示的图中，将一个四节点 Cassandra 群集部署在配置为允许 Thrift 通信的负载平衡器后面的虚拟机中。Azure 托管的 PaaS 应用程序使用语言特定的 Thrift 库访问群集。存在针对各种语言（包括 Java、C\#、Node.js、Python 和 C++）的库。第二个图中显示的独立虚拟机部署通过在虚拟机上托管的其他云服务中运行的应用程序来使用数据。

## <span id="task1"></span> </a>任务 1：部署 Linux 群集

在虚拟机预览版发行期间，为了使 Linux VM 成为同一虚拟网络的一部分，必须将所有计算机部署到相同的云服务中。创建群集的典型顺序为：

![有关创建群集的顺序图][有关创建群集的顺序图]

**步骤 1：生成 SSH 密钥对**

Azure 在进行配置时需要用 PEM 或 DER 编码的 X509 公钥。按照[如何在 Azure 上通过 Linux 使用 SSH（可能为英文页面）][如何在 Azure 上通过 Linux 使用 SSH（可能为英文页面）]上的说明进行操作来生成公/私钥对。如果你打算在 Windows 或 Linux 上将 putty.exe 用作 SSH 客户端，则必须使用 puttygen.exe 将 PEM 编码的 RSA 私钥转换为 PPK 格式。可在[在 Azure 上为 Linux VM 部署生成 SSH 密钥对（可能为英文页面）][在 Azure 上为 Linux VM 部署生成 SSH 密钥对（可能为英文页面）]上找到有关此操作的说明。

**步骤 2：创建 Ubuntu VM**

若要创建第一个 Ubuntu VM，请登录到 Azure 预览版门户，依次单击“新建”、“虚拟机”、“从库中”、“Unbuntu Server 12.xx”和右箭头。有关介绍如何创建 Linux VM 的教程，请参阅[创建运行 Linux 的虚拟机（可能为英文页面）][创建运行 Linux 的虚拟机（可能为英文页面）]。

然后，在“VM 配置”屏幕上输入下列信息：

<table>
    <tr>
<th>字段名称</th>
<th>字段值</th>
<th>备注</th>
    </tr>
    <tr>
<td>虚拟机名称</td>
<td>hk-cas1</td>
<td>这是 VM 的主机名</td>
    </tr>
    <tr>
<td>新用户名</td>
<td>localadmin</td>
<td>&ldquo;admin&rdquo;是 Ubuntu 12.xx 中保留的用户名</td>
    </tr>
    <tr>
<td>新密码</td>
<td><i>强密码</i></td>
        <td></td>
    </tr>
    <tr>
<td>确认密码</td>
<td><i>强密码</i></td>
        <td></td>
    </tr>
    <tr>
<td>大小</td>
<td>小型</td>
<td>根据 IO 需求选择 VM。 </td>
    </tr>
    <tr>
<td>使用用于身份验证的 SSH 密钥进行保护</td>
<td>单击复选框</td>
<td>检查是否要使用 SSH 密钥进行保护</td>
    </tr>
    <tr>
<td>证书</td>
<td><i>公钥证书的文件名</i></td>
<td>使用 OpenSSL 或其他工具生成的 DER 或 PEM 编码的 SSH 公钥</td>
    </tr>
</table>

在“VM 模式”屏幕上输入下列信息：

<table>
    <tr>
<th>字段名称</th>
<th>字段值</th>
<th>备注</th>
    </tr>
    <tr>
<td>独立 VM</td>
<td>&ldquo;选中&rdquo;单选框</td>
<td>此选项适用于第一个 VM，对于后续 VM，我们将使用&ldquo;连接到现有 VM&rdquo;选项</td>
    </tr>
    <tr>
<td>DNS 名称</td>
<td><i>唯一名称</i>.cloudapp.net</td>
<td>为计算机提供不可知的负载平衡器名称</td>
    </tr>
    <tr>
<td>存储帐户</td>
<td><i>默认存储帐户</i></td>
<td>使用你创建的默认存储帐户</td>
    </tr>
    <tr>
<td>区域/地缘组/虚拟网络</td>
<td>美国西部</td>
<td>选择你的 Web 应用程序从中访问 Cassandra 群集的区域</td>
    </tr>
</table>

为将作为 Cassandra 群集的一部分的所有虚拟机重复上述过程。此时，所有计算机将成为同一网络的一部分，并可相互进行 ping 操作。如果 ping 操作无法执行，请检查 VM 的防火墙（例如 iptables）配置以确保允许 ICMP。确保在成功测试网络连接后禁用 ICMP 以减小攻击向量。

**步骤 3：添加负载平衡 Thrift 终结点**

完成步骤 1 和步骤 2 后，每个 VM 应具有已定义的 SSH 终结点。现在，我们使用公用端口 9160 添加负载平衡 Thrift 终结点。顺序如下：

a. 从第一个 VM 的详细信息视图中，单击“添加终结点”

b. 在“将终结点添加到虚拟机”屏幕上，选择“添加终结点”单选按钮

c. 单击右箭头

d. 在“指定终结点详细信息”屏幕上，输入以下内容

<table>

<tr>

<th>
字段名称

</th>

<th>
字段值

</th>

<th>
备注

</th>

</tr>

<tr>

<td>
名称

</td>

<td>
cassandra

</td>

<td>
任何唯一的终结点名称都适用

</td>

</tr>

<tr>

<td>
协议

</td>

<td>
TCP

</td>

<td>
</td>

</tr>

<tr>

<td>
公用端口

</td>

<td>
9160

</td>

<td>
默认 Thrift 端口。

</td>

</tr>

<tr>

<td>
专用端口

</td>

<td>
9160

</td>

<td>
除非你在 cassandra.yaml 中更改此端口

</td>

</tr>

</table>
完成上述操作后，第一个 VM 将显示 LOAD BALANCED 字段为“NO”的 cassandra 终结点。请暂时将其忽略，因为在将此终结点添加到后续 VM 后，该字段将变为“YES”

</p>
e. 此时选择第二个 VM，并通过重复上述过程添加终结点，唯一的细微差异是，你将选择“现有终结点上的负载平衡流量”并使用下拉框中的“cassandra-960”。在此阶段，映射到这两个 VM 的终结点会将状态从 LOAD BALANCED 状态“NO”更改为“YES”。

对群集中的后续节点重复“e”。

现在，我们已拥有 VM，可以开始在每个 VM 上设置 Cassandra 了。由于 Cassandra 不是许多 Linux 分发的标准部分，因此我们采用手动部署过程。

[请注意，我们在每个 VM 上使用的是手动安装软件的方法。不过，可通过设置一个功能完备的 Cassandra VM 加快此过程，将该 VM 作为基本映像捕获并从此基本映像创建其他实例。有关捕获 Linux 映像的说明，请参阅[如何捕获运行 Linux 的虚拟机的映像（可能为英文页面）][请注意，我们在每个 VM 上使用的是手动安装软件的方法。不过，可通过设置一个功能完备的 Cassandra VM 加快此过程，将该 VM 作为基本映像捕获并从此基本映像创建其他实例。有关捕获 Linux 映像的说明，请参阅[如何捕获运行 Linux 的虚拟机的映像（可能为英文页面）]。]

## <span id="task2"></span> </a>任务 2：在每台虚拟机上设置 Cassandra

**步骤 1：安装必备组件**

Cassandra 需要 Java 虚拟机，因此使用适用于 Debian 衍生工具（包括 Ubuntu）的以下命令安装最新的 JRE：

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java7-installer

**步骤 2：Cassandra 安装**

1.  使用 SSH 登录到 Linux (Ubuntu) VM 实例。

2.  使用 wget 将 Cassandra 组件作为 apache-cassandra-bin.tar.gz 从 (<http://cassandra.apache.org/download/>)[<http://cassandra.apache.org/download/>] 处建议的镜像中下载到“~/downloads”目录。请注意，未将版本号包含在下载的文件中以确保发布活动不需了解具体的版本。

3.  通过执行以下命令将 tar 文件提取到默认登录目录：

        tar -zxvf downloads/apache-cassandra-bin.tar.gz

    上述操作会将存档展开到 apache-cassandra- [version] 目录。

4.  创建以下两个默认目录以保存日志和数据：

        $ sudo chown -R /var/lib/cassandra
        $ sudo chown -R /var/log/cassandra

5.  向 Cassandra 在其下运行的用户标识授予写入权限

        a.  sudo chown -R <user>:<group> /var/lib/cassandra
        b.  sudo chown -R <user>:<group> /var/log/cassandra
        To use current user context, replace the <user> and <group> with $USER and $GROUP

6.  使用以下命令从 apache-cassandra-[version]/bin 目录启动 Cassandra：

        $ ./cassandra

上述操作会将 Cassandra 节点作为后台进程启动。使用“cassandra –f”在前台模式中启动该进程。

日志可能会显示 mx4j 错误。Cassandra 不需要 mx4j 即可正常运行，但有必要管理 Cassandra 安装。先终止 Cassandra 进程，然后再继续下一步。

**步骤 3：安装 mx4j**

    a)  Download mx4j: wget [http://sourceforge.net/projects/mx4j/files/MX4J%20Binary/3.0.2/mx4j-3.0.2.tar.gz/download](http://sourceforge.net/projects/mx4j/files/MX4J%20Binary/3.0.2/mx4j-3.0.2.tar.gz/download) -O mx4j.tar.gz
    b)  tar -zxvf mx4j.tar.gz
    c)  cp mx4j-23.0.2/lib/*.jar ~/apache-cassandra-<version>/lib
    d)  rm -rf mx4j-23.0.2
    e)  rm mx4j.tar.gz

在此阶段重新启动 Cassandra 进程

**步骤 4：测试 Cassandra 安装**

从 Cassandra 的 bin 目录中执行以下命令以便使用 Thrift 客户端进行连接：

    cassandra-cli -h localhost -p 9160

**步骤 5：为外部连接启用 Cassandra**

默认情况下，Cassandra 仅设置为侦听环回地址，因此对于外部连接而言，此更改是强制性的。

编辑“conf/cassandra.yaml”以将 **listen\_address** 和 **rpc\_address** 更改为服务器的 IP 地址或主机名，使当前节点对其他节点和外部负载平衡器可见。

对群集中的所有节点重复步骤 1 到步骤 5。

此时，所有单独的 VM 已配备了必要软件，可以通过种子配置在节点之间建立通信了。有关多节点群集配置的详细信息，请查看 [][]<http://wiki.apache.org/cassandra/MultinodeCluster></a> 上的信息。

**步骤 6：设置多节点群集**

编辑 cassandra.yaml 以更改所有 VM 中的下列属性：

**a) cluster\_name**

默认群集名称设置为“Test Cluster”；将其更改为可反映你的应用程序的名称。示例：“AppStore”。如果你已在安装期间启动用于测试的带“Test Cluster”的群集，则将收到群集名称不匹配错误。若要纠正此错误，请删除 /var/lib/cassandra/data/system 目录下的所有文件。

**b) seeds**

新节点将使用此处指定的 IP 地址来了解环形拓扑。采用逗号分隔的形式将最可靠的节点设置为你的种子："*host1*,*host2*”。示例设置：“hk-ub1,hk-ub2”。

由于此内容不是本练习的重心，因此我们将接受种子服务器提供的默认令牌。若要优化令牌生成情况，请查看位于以下位置的 python 脚本：
[][1]<http://wiki.apache.org/cassandra/GettingStarted></a>。

在所有节点上重新启动 Cassandra 可应用上述更改。

**步骤 7：测试多节点群集**

已安装到 Cassandra 的 bin 目录中的 Nodetool 将有助于进行群集操作。我们将使用 nodetool 通过以下命令格式验证群集设置：

    $ bin/nodetool -h <hostname> -p 7199 ring

如果配置正确，则应显示三节点群集的信息，如下所示：

<table>
<colgroup>
<col width="12%" />
<col width="12%" />
<col width="12%" />
<col width="12%" />
<col width="12%" />
<col width="12%" />
<col width="12%" />
<col width="12%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left">地址</td>
<td align="left">DC</td>
<td align="left">机架</td>
<td align="left">状态</td>
<td align="left">状况</td>
<td align="left">加载</td>
<td align="left">所有</td>
<td align="left">令牌</td>
</tr>
<tr class="even">
<td align="left"></td>
<td align="left"></td>
<td align="left"></td>
<td align="left"></td>
<td align="left"></td>
<td align="left"></td>
<td align="left"></td>
<td align="left">149463697837832744402916220269706844972</td>
</tr>
<tr class="odd">
<td align="left">10.26.196.68</td>
<td align="left">datacenter1</td>
<td align="left">rack1</td>
<td align="left">正常</td>
<td align="left">一般</td>
<td align="left">15.69 KB</td>
<td align="left">25.98%</td>
<td align="left">114445918355431753244435008039926455424</td>
</tr>
<tr class="even">
<td align="left">10.26.198.81</td>
<td align="left">datacenter1</td>
<td align="left">rack1</td>
<td align="left">正常</td>
<td align="left">一般</td>
<td align="left">15.69 KB</td>
<td align="left">53.44%</td>
<td align="left">70239176883275351288292106998553981501</td>
</tr>
<tr class="odd">
<td align="left">10.26.198.84</td>
<td align="left">datacenter1</td>
<td align="left">rack1</td>
<td align="left">正常</td>
<td align="left">一般</td>
<td align="left">18.35 KB</td>
<td align="left">25.98%</td>
<td align="left">149463697837832744402916220269706844972</td>
</tr>
</tbody>
</table>

在此阶段，群集通过在“部署 Linux 群集”任务执行期间创建的云服务 URL（创建第一个 VM 时提供的 DNS 名称）为 Thrift 客户端做好了准备。

## <span id="task3"></span> </a>任务 3：从 Node.js 访问 Cassandra 群集

使用前面的任务所描述的过程在 Azure 上创建 Linux VM。确保此 VM 是独立 VM，因为我们会将此 VM 用作访问 Cassandra 群集的客户端。我们将先从 github 中安装 Node.js、NPM 和 [cassandra-client][cassandra-client]，然后从该 VM 连接到 Cassandra 群集：

**步骤 1：安装 Node.js 和 NPM**

a) 安装必备组件

    sudo apt-get install g++ libssl-dev apache2-utils make

b) 我们将使用 GitHub 中的源进行编译和安装；我们需要先安装 git 核心运行时，然后才能克隆存储库：

    sudo apt-get install git-core

c) 克隆节点存储库

    git clone git://github.com/joyent/node.git

d) 上述操作将创建名为“node”的目录。执行以下命令序列可编译安装 node.js：

    cd node
    ./configure
    make
    sudo make install

e) 通过执行以下命令从稳定的二进制文件安装 NPM

    curl http://npmjs.org/install.sh | sh

**步骤 2：安装 cassandra-client 包**

    npm cassandra-client 

**步骤 3：准备 Cassandra 存储**

Cassandra 存储使用 KEYSPACE 和 COLUMNFAMILY 的概念，二者大致相当于 RDBMS 行话中的 DATABASE 和 TABLE 结构。KEYSAPCE 将包含一组 COLUMNFAMILY 定义。每个 COLUMNFAMILY 将包含一组行，而每个行又包含多个列，如下面的复合视图中所示：

![行和列][行和列]

我们将使用之前部署的 Cassandra 群集来通过创建并查询上述数据结构演示 node.js 访问。我们将创建一个简单的 node.js 脚本，该脚本执行群集的基本准备工作以便存储客户数据。可在 node.js Web 应用程序或 Web 服务中轻松使用脚本中显示的方法。请记住，代码段仅说明了所有项目的工作方式，而对于实际解决方案，显示的代码还有很大的改进空间（例如，安全性、日志记录、可伸缩性等）。

下面我们在脚本范围内定义所需的变量以包含 cassandra-client 模块中的 PooledConnection、常用密钥空间名称和密钥空间连接参数：

    casdemo.js: 
    var pooledCon = require('cassandra-client').PooledConnection;
    var ksName = "custsupport_ks";
    var ksConOptions = { hosts: ['<azure_svc_name>.chinacloudapp.cn:9160'], 
                         keyspace: ksName, use_bigints: false };

为了准备好存储客户数据，我们需要先使用以下脚本示例创建 KEYSPACE：

    casdemo.js: 
    function createKeyspace(callback){
       var cql = 'CREATE KEYSPACE ' + ksName + ' WITH 
       strategy_class=SimpleStrategy AND strategy_options:replication_factor=1';
       var sysConOptions = { hosts: ['<azure_svc_name>.chinacloudapp.cn:9160'],  
                             keyspace: 'system', use_bigints: false };
       var con = new pooledCon(sysConOptions);
       con.execute(cql,[],function(err) {
       if (err) {
         console.log("Failed to create Keyspace: " + ksName);
         console.log(err);
       }
       else {
         console.log("Created Keyspace: " + ksName);
         callback(ksConOptions, populateCustomerData);
       }
       });
       con.shutdown();
    } 

createKeysapce 函数会将一个回调函数用作参数，这意味着，将 COLUMNFAMILY 创建函数作为 KEYSPACE 执行是创建列系列的先决条件。请注意，我们需要连接到应用程序 KEYSPACE 定义的“system”KEYSPACE。[Cassandra 查询语言 (CQL)][Cassandra 查询语言 (CQL)] 一直用于与所有这些代码段中的群集进行交互。由于上述脚本中组合的 CQL 没有任何参数标记，因此我们在使用 PooledConnection.execute() 方法时将使用空参数集合（“[]”）。

密钥空间创建成功后，将执行下面的代码段中显示的函数 createColumnFamily() 以创建所需的 COLUMNFAMILY 定义：

    casdemo.js: 
    //Creates COLUMNFAMILY
    function createColumnFamily(ksConOptions, callback){
      var params = ['customers_cf','custid','varint','custname',
                    'text','custaddress','text'];
      var cql = 'CREATE COLUMNFAMILY ? (? ? PRIMARY KEY,? ?, ? ?)';
    var con =  new pooledCon(ksConOptions);
      con.execute(cql,params,function(err) {
          if (err) {
             console.log("Failed to create column family: " + params[0]);
             console.log(err);
          }
          else {
             console.log("Created column family: " + params[0]);
             callback();
          }
      });
      con.shutdown();
    } 

Parameterized CQL 模板将与 params 对象结合使用以便为 COLUMNFAMILY 创建生成有效的 CQL。成功创建 COLUMNFAMILY 后，提供的回调（本示例中为 populateCustomerData()）将作为异步调用链的一部分调用。

    casdemo.js: 
    //populate Data
    function populateCustomerData() {
       var params = ['John','Infinity Dr, TX', 1];
       updateCustomer(ksConOptions,params);

       params = ['Tom','Fermat Ln, WA', 2];
       updateCustomer(ksConOptions,params);
    }

    //update will also insert the record if none exists
    function updateCustomer(ksConOptions,params)
    {
      var cql = 'UPDATE customers_cf SET custname=?,custaddress=? where 
                 custid=?';
      var con = new pooledCon(ksConOptions);
      con.execute(cql,params,function(err) {
          if (err) console.log(err);
          else console.log("Inserted customer : " + params[0]);
      });
      con.shutdown();
    }

populateCustomerData() 将两个行插入名为 customers\_cf 的 COLUMNFAMILY 中。在 Cassandra 查询语言中，如果记录未在生成 INSERT CQL 语句冗余的过程中显示，则 UPDATE 将插入记录。

目前，我们连接了回调链：createKeyspace() 到 createColumnFamily() 再到 populateCustomerData()。现在，可以通过以下代码段执行此代码：

    casdemo.js:
    var pooledCon = require('cassandra-client').PooledConnection;
    var ksName = "custsupport_ks";
    var ksConOptions = { hosts: ['<azure_svc_name>.chinacloudapp.cn:9160'], 
                         keyspace: ksName, use_bigints: false };

    createKeyspace(createColumnFamily);
    //rest of the not shown

从外壳程序提示符处执行以下命令来执行该脚本：

    //the following command will create the KEYSPACE, COLUMNFAMILY and //inserts two customer records
    $ node casdemo.js

readCustomer() 方法将访问 Azure 托管的群集，并显示执行 CQL 查询后检索到的 JSON 代码段：

    casdemo.js: 
    //read the two rows inserted above
    function readCustomer(ksConOptions)
    {
      var cql = 'SELECT * FROM customers_cf WHERE custid IN (1,2)';
      var con = new pooledCon(ksConOptions);
      con.execute(cql,[],function(err,rows) {
          if (err) 
             console.log(err);
          else 
             for (var i=0; i<rows.length; i++)
                console.log(JSON.stringify(rows[i]));
        });
       con.shutdown();
    } 

修改 casdemo.js 以添加上面的函数，并在注释之前调用的 createKeyspace() 方法后调用该函数，如下所示：

    casdemo.js: 
    var pooledCon = require('cassandra-client').PooledConnection;
    var ksName = "custsupport_ks";
    var ksConOptions = { hosts: ['<azure_svc_name>.chinacloudapp.cn:9160'], 
                         keyspace: ksName, use_bigints: false };

    //createKeyspace(createColumnFamily);
    readCustomer(ksConOptions)
    //rest of the code below not shown
        

## <span id="conclusion"></span> </a>结束语

利用 Azure 虚拟机功能可创建 Linux（由 Microsoft 合作伙伴提供的映像），利用 Windows 虚拟机可迁移现有服务器产品和应用程序而不进行任何更改。本文中讨论的 Cassandra NoSQL 数据库服务器就是这样一个示例。可通过 Windows 和 Linux 操作系统环境中的 Azure 托管的云服务、第三方公有云和私有云访问本文中设置的 Cassandra 群集。在本文中，我们将 node.js 用作客户端；但是，可以从 .NET、Java 和其他语言环境访问 Cassandra。

  [概述]: #overview
  [Cassandra 部署示意]: #schematic
  [复合部署]: #composite
  [Azure 虚拟机部署]: #deployment
  [任务 1：部署 Linux 群集]: #task1
  [任务 2：在每台虚拟机上设置 Cassandra]: #task2
  [任务 3：从 Node.js 访问 Cassandra 群集]: #task3
  [结束语]: #conclusion
  [Cassandra]: http://wiki.apache.org/cassandra/
  [复合部署图]: ./media/virtual-machines-linux-nodejs-running-cassandra/cassandra-linux1.png
  [虚拟机部署]: ./media/virtual-machines-linux-nodejs-running-cassandra/cassandra-linux2.png
  [有关创建群集的顺序图]: ./media/virtual-machines-linux-nodejs-running-cassandra/cassandra-linux4.png
  [如何在 Azure 上通过 Linux 使用 SSH（可能为英文页面）]: http://azure.microsoft.com/zh-cn/documentation/articles/linux-use-ssh-key/
  [在 Azure 上为 Linux VM 部署生成 SSH 密钥对（可能为英文页面）]: http://blogs.msdn.com/b/hanuk/archive/2012/06/07/generating-ssh-key-pair-for-linux-vm-deployment-on-windows-azure.aspx
  [创建运行 Linux 的虚拟机（可能为英文页面）]: http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-linux-tutorial/
  [如何捕获运行 Linux 的虚拟机的映像（可能为英文页面）]: http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-linux-capture-image/
  []: http://wiki.apache.org/cassandra/MultinodeCluster
  [1]: http://wiki.apache.org/cassandra/GettingStarted
  [cassandra-client]: https://github.com/racker/node-cassandra-client
  [行和列]: ./media/virtual-machines-linux-nodejs-running-cassandra/cassandra-linux3.png
  [Cassandra 查询语言 (CQL)]: http://cassandra.apache.org/doc/cql/CQL.html
