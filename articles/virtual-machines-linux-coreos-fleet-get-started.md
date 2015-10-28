<properties
	pageTitle="在 Azure 上的 CoreOS 上使用 Fleet 入门"
	description="提供在 Azure 上的 CoreOS Linux 虚拟机上使用 Fleet 和 Docker 的基本示例。"
	services="virtual-machines"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="08/03/2015"
	wacn.date="09/18/2015"/>

# 在 Azure 上的 CoreOS 上使用 Fleet 入门

本文提供两个在 [CoreOS] 虚拟机的群集上使用 [fleet](https://github.com/coreos/fleet) 和 [Docker](https://www.docker.com/) 运行应用程序的快速示例。

若要使用这些示例，需要先设置三节点 CoreOS 群集，如[如何在 Azure 上使用 CoreOS] 中所述。完成该操作后，你将了解非常基本的 CoreOS 部署元素，并拥有可以工作的群集和客户端计算机。在这些示例中，我们将使用完全相同的群集名称。此外，这些示例还假定你使用本地 Linux 主机运行 **fleetctl** 命令。




## <a id='simple'>示例 1：使用 Docker 的 Hello World</a>

下面是一个在单个 Docker 容器中运行的简单“Hello World”应用程序。此应用程序使用 [busybox Docker Hub 映像]。

在 Linux 客户端计算机上，使用你最喜欢的文本编辑器创建以下 **systemd** 单元文件并将其命名为 `helloworld.service`。（有关语法的详细信息，请参阅[单元文件]。）

```
[Unit]
Description=HelloWorld
After=docker.service
Requires=docker.service

[Service]

TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill busybox1
ExecStartPre=-/usr/bin/docker rm busybox1
ExecStartPre=/usr/bin/docker pull busybox
ExecStart=/usr/bin/docker run --name busybox1 busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"
ExecStop=/usr/bin/docker stop busybox1

```

现在连接到 CoreOS 群集，并通过运行以下 **fleetctl** 命令启动单元。输出将显示单元已启动以及它所在的位置。


```
fleetctl --tunnel coreos-cluster.chinacloudapp.cn:22 start helloworld.service


Unit helloworld.service launched on 62f0f66e.../100.79.86.62
```

>[AZURE.NOTE]若要运行不带 **--tunnel** 参数的远程 **fleetctl** 命令，可以选择设置 FLEETCTL\_TUNNEL 环境变量以通过隧道传送请求。例如：`export FLEETCTL_TUNNEL=coreos-cluster.chinacloudapp.cn:22`。


你可以连接到容器，以查看服务的输出：

```
fleetctl --tunnel coreos-cloudapp.cluster.net:22 journal helloworld.service

Mar 04 21:29:26 node-1 docker[57876]: Hello World
Mar 04 21:29:27 node-1 docker[57876]: Hello World
Mar 04 21:29:28 node-1 docker[57876]: Hello World
Mar 04 21:29:29 node-1 docker[57876]: Hello World
Mar 04 21:29:30 node-1 docker[57876]: Hello World
Mar 04 21:29:31 node-1 docker[57876]: Hello World
Mar 04 21:29:32 node-1 docker[57876]: Hello World
Mar 04 21:29:33 node-1 docker[57876]: Hello World
Mar 04 21:29:34 node-1 docker[57876]: Hello World
Mar 04 21:29:35 node-1 docker[57876]: Hello World
```

若要清理，请停止并卸载单元。

```
fleetctl --tunnel coreos-cluster.chinacloudapp.cn:22 stop helloworld.service
fleetctl --tunnel coreos-cluster.chinacloudapp.cn:22 unload helloworld.service
```


## <a id='highavail'>示例 2：高度可用的 Apache 服务器</a>

使用 CoreOS、Docker 和 **fleet** 的一个优点是很容易以高度可用的方式运行服务。在此示例中，你将部署一个由三个运行 Apache Web 服务器的相同容器组成的服务。这些容器将在群集中的三个虚拟机上运行。此示例类似于[使用 fleet 启动容器]中的示例，并使用 [CoreOS Apache Docker Hub 映像]。

>[AZURE.IMPORTANT]若要运行高度可用的 Apache 服务器，需要在虚拟机（公共端口 80、专用端口 80）上配置负载平衡 HTTP 终结点。你可以在创建 CoreOS 群集后，使用 Azure 门户或 **azure vm endpoint** 命令执行此操作。有关详细信息，请参阅[配置负载平衡集]。

在客户端计算机上，使用你最喜欢的文本编辑器创建名为 apache@.service 的 **systemd** 模板单元文件。你将使用该模板来启动名为 apache@1.service、apache@2.service 和 apache@3.service 的三个单独实例：

```
[Unit]
Description=High Availability Apache
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill apache1
ExecStartPre=-/usr/bin/docker rm apache1
ExecStartPre=/usr/bin/docker pull coreos/apache
ExecStart=/usr/bin/docker run -rm --name apache1 -p 80:80 coreos/apache /usr/sbin/apache2ctl -D FOREGROUND
ExecStop=/usr/bin/docker stop apache1

[X-Fleet]
X-Conflicts=apache@*.service
```

>[AZURE.NOTE]`X-Conflicts` 属性告知 CoreOS 只有此容器的一个实例可以在给定的 CoreOS 主机上运行。有关详细信息，请参阅[单元文件]。

现在启动 CoreOS 群集中的单元实例。你应看到这三个实例在三个不同的计算机上运行：

```
fleetctl --tunnel coreos-cluster.chinacloudapp.cn:22 start apache@{1,2,3}.service

unit apache@3.service launched on 00c927e4.../100.79.62.16
unit apache@1.\service launched on 62f0f66e.../100.79.86.62
unit apache@2.service launched on df85f2d1.../100.78.126.15

```
若要访问运行其中一个单元的 Apache 服务器，请向托管 CoreOS 群集的云服务发送一个简单请求。

`curl http://coreos-cluster.chinacloudapp.cn`

你将看到从 Apache 服务器返回的默认文本类似于：

```
<html><body><h1>It works!</h1>
<p>This is the default web page for this server.</p>
<p>The web server software is running but no content has been added, yet.</p>
</body></html>
```

你可以尝试关闭群集中的一个或多个虚拟机，以验证 Apache 服务是否继续运行。

完成后，请停止并卸载单元。

```
fleetctl --tunnel coreos-cluster.chinacloudapp.cn:22 stop apache@{1,2,3}.service
fleetctl --tunnel coreos-cluster.chinacloudapp.cn:22 unload apache@{1,2,3}.service

```

## 后续步骤

* 你可以尝试使用 Azure 上的三节点 CoreOS 群集执行更多操作。通过阅读 [Tim Park 的 CoreOS 教程]、[Patrick Chanezon 的 CoreOS 教程]、[Docker] 文档和 [CoreOS 概述]，探索如何创建更复杂的群集、使用 Docker 和创建更有趣的应用程序。

* 请参阅 [Azure 上的 Linux 和开源计算]，了解有关在 Azure 中的 Linux VM 上使用开源环境的详细信息。

<!--Link references-->
[Azure Command-Line Interface (Azure)]: /documentation/articles/xplat-cli
[CoreOS]: https://coreos.com/
[CoreOS 概述]: https://coreos.com/using-coreos/
[CoreOS with Azure]: https://coreos.com/docs/running-coreos/cloud-providers/azure/
[Tim Park 的 CoreOS 教程]: https://github.com/timfpark/coreos-azure
[Patrick Chanezon 的 CoreOS 教程]: https://github.com/chanezon/azure-linux/tree/master/coreos/cloud-init
[Docker]: http://docker.io
[YAML]: http://yaml.org/
[如何在 Azure 上使用 CoreOS]: /documentation/articles/virtual-machines-linux-coreos-how-to
[配置负载平衡集]: https://msdn.microsoft.com/zh-CN/library/azure/dn655055.aspx
[使用 fleet 启动容器]: https://coreos.com/docs/launching-containers/launching/launching-containers-fleet/
[单元文件]: https://coreos.com/docs/launching-containers/launching/fleet-unit-files/
[busybox Docker Hub 映像]: https://registry.hub.docker.com/_/busybox/
[CoreOS Apache Docker Hub 映像]: https://registry.hub.docker.com/u/coreos/apache/
[Azure 上的 Linux 和开源计算]: /documentation/articles/virtual-machines-linux-opensource
<!---HONumber=70-->