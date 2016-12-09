<properties
	pageTitle="使用 PsPing & PaPing 进行 TCP 端口连通性测试"
	description="在 Windows 及 Linux 操作系统中分别使用 PsPing 和 PaPing 工具进行 TCP 端口连通性测试"
	services="virtual-network"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags="Azure,虚拟机,PSPing,PaPing,ping,IMCP"/>

<tags
    ms.service="virtual-network-aog"
    ms.date="12/08/2016"
    wacn.date="12/08/2016"/>

# 使用 PsPing & PaPing 进行 TCP 端口连通性测试 #

### PsPing & PaPing 介绍 ###

通常，我们测试数据包能否通过 IP 协议到达特定主机时，都习惯使用 ping 命令。工作时 ping 向目标主机发送一个 IMCP Echo 请求的数据包，并等待接收 Echo 响应数据包，通过响应时间和成功响应的次数来估算丢包率和网络时延。但是在 Azure 中，ICMP 包无法通过防火墙和负载平衡器，所以不能直接使用 ping 来测试 Azure 中的虚拟机和服务的连通性（VPN 和 Express Route 通道中的流量不经过负载平衡器，所以只要链路上的防火墙允许 ICMP 包传递，ping 依然可用）。

为了在 Azure 中进行连通性测试，例如测试 RDP、SSH 端口可用性，或者 HTTP、HTTPS 服务稳定性，甚至测试从 Azure 向外部服务的连接，我们都推荐使用 PsPing 或 PaPing。PsPing 是微软 PSTools 工具套件中的其中一个命令。除了ICMP ping 测试，它主要用来测试 TCP 端口的连通性，还可以测试 TCP/UDP 网络时延和带宽。不过， PsPing 只能在 Windows 中运行。如果您需要在 Linux 中发起 TCP 端口连通性和网路时延的测试，可以使用 PaPing 。PaPing 是一个跨平台的开源工具。它的功能相对 PsPing 而言更简单，只支持 TCP 端口的相关测试，不支持 UDP 端口的测试。

### PsPing ###

#### 下载和安装 ####

PsPing 可以的下载页面网址为：[https://technet.microsoft.com/zh-cn/sysinternals/jj729731.aspx](https://technet.microsoft.com/zh-cn/sysinternals/jj729731.aspx)。此页面上也包含它的详细使用方法，若有需要可以查看此页面上的帮助信息。下载完后，可以单独将 psping.exe 命令解压出来放在任意路径，然后从命令提示符来运行。当然，您也可以将整个压缩包解压到指定的路径来获取压缩包内完整的 PSTools 工具套件。

#### 使用方法 ####

打开命令行提示符窗口，进入到 psping.exe 所在的目录，就可以运行 PsPing 了。如前文所述，PsPing 支持的测试方法有很多，这里我们主要介绍针对 TCP 端口的连通性测试。最简单的测试方法就是直接在 psping.exe 命令后面加上要测试的主机名和端口，然后执行。这里以从 Azure 内部测试 www.azure.cn 的 TCP-80 端口为例，命令为`psping.exe www.azure.cn:80`。

	C:\Tools>psping www.azure.cn:80
	
	PsPing v2.10 - PsPing - ping, latency, bandwidth measurement utility
	Copyright (C) 2012-2016 Mark Russinovich
	Sysinternals - www.sysinternals.com
	
	TCP connect to 116.211.251.197:80:
	5 iterations (warmup 1) ping test:
	Connecting to 116.211.251.197:80 (warmup): from 10.91.1.4:51413: 34.69ms
	Connecting to 116.211.251.197:80: from 10.91.1.4:51414: 29.11ms
	Connecting to 116.211.251.197:80: from 10.91.1.4:51415: 30.56ms
	Connecting to 116.211.251.197:80: from 10.91.1.4:51416: 49.02ms
	Connecting to 116.211.251.197:80: from 10.91.1.4:51417: 43.84ms
	
	TCP connect statistics for 116.211.251.197:80:
	  Sent = 4, Received = 4, Lost = 0 (0% loss),
	  Minimum = 29.11ms, Maximum = 49.02ms, Average = 38.13ms

我们可以看到，PsPing 获取到 `www.azure.cn` 的IP为 `175.25.168.95`。随后进行了一次热身测试，热身测试的目的在于使正式的测试数据更准确。最终统计结果只计算 4 次正式测试数据。其中，统计结果第一行包含发送请求的次数，接收到回应的次数，连接丢失的次数以及丢失百分比。第二行为最小、最大以及平均的响应时延。
我们还可以在命令行中添加参数来定义 PsPing 进行测试的方式。以下为 PsPing 进行 TCP 连接测试时所支持的参数：

	-t 类似于 ICMP 的长 ping 测试，直到按下 Ctrl+C 停止测试，并显示统计结果；
	-n 指定测试次数。还可以指定测试的时间长度，以秒为单位，使用时在数字后加上 s，例如“10s”；
	-i 每次测试的间隔，默认为 1 秒。还可以指定为 0 来进行快速 ping 测试；
	-w 热身次数，默认为 1 次；
	-q 测试过程中不输出结果，结束后显示统计结果；
	-h 将时延结果统计为直方图打印（默认打印 20行），也可以指定结果行数，比如 -h 10，指定 10 行；另一种使用方法是统计自定义时延，比如 -h "65,70"，结果将统计时延分别为 65 和 70 毫秒的次数；
	-4 强制使用 IPv4；
	-6 强制使用 IPv6；

更多时候，我们指定测试次数，例如 500 次、1000 次。待测试结束后查看统计结果，根据连接成功率和 TCP 响应时延来判断被检测服务的可用性和稳定性。不过，由于是测试 TCP 连接，测试时不排除被测试服务有一定的防护机制，对连续、大量的 TCP 连接采取拒绝服务或者限制服务，导致测试结果看起来很槽糕。这需要测试人对被测试服务有一定的了解。

我们还是以测试 `www.azure.cn` 为例，测试 500 次连接的命令为 `psping.exe -n 500 www.azure.cn:80`

	C:\Tools>psping -n 500 www.azure.cn:80
	
	PsPing v2.10 - PsPing - ping, latency, bandwidth measurement utility
	Copyright (C) 2012-2016 Mark Russinovich
	Sysinternals - www.sysinternals.com
	
	TCP connect to 175.25.168.95:80:
	501 iterations (warmup 1) ping test:
	Connecting to 175.25.168.95:80 (warmup): from 10.91.1.4:51531: 2.28ms
	Connecting to 175.25.168.95:80: from 10.91.1.4:51532: 1.86ms
	Connecting to 175.25.168.95:80: from 10.91.1.4:51533: 2.67ms
	....................
	Connecting to 175.25.168.95:80: from 10.91.1.4:52029: 1.90ms
	Connecting to 175.25.168.95:80: from 10.91.1.4:52030: 2.69ms
	Connecting to 175.25.168.95:80: from 10.91.1.4:52031: 2.69ms
	Connecting to 175.25.168.95:80: from 10.91.1.4:52032: 2.39ms
	
	TCP connect statistics for 175.25.168.95:80:
	  Sent = 500, Received = 500, Lost = 0 (0% loss),
	  Minimum = 1.49ms, Maximum = 4.72ms, Average = 2.35ms

### PaPing ###

#### 下载和安装 ####

PaPing 的下载网址为：[https://code.google.com/archive/p/paping/downloads](https://code.google.com/archive/p/paping/downloads)。

其中 32 位 Linux 对应的压缩包为 `paping_1.5.5_x86_linux.tar.gz`，64 位的 Linux 对应的压缩包为 `paping_1.5.5_x86-64_linux.tar.gz`。下载完成后，直接解压到任意路径，就可以直接执行了。

以 64 位 Linux 为例：

	#cd ~
	#wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/paping/paping_1.5.5_x86-64_linux.tar.gz
	#tar zxvf paping_1.5.5_x86-64_linux.tar.gz

#### 使用方法 ####

PaPing 的使用方法与 PsPing 非常相似，甚至更简单，功能更单一。PaPing 所支持的参数如下：

	-p, --port N 指定被测试服务的 TCP 端口（必须）；
	--nocolor 屏蔽彩色输出；
	-t, --timeout	指定超时时长，单位为毫秒，默认值为 1000；
	-c, --count N	指定测试次数。

默认 PaPing 的结果会根据 Shell 的色彩配置输出不同颜色。如果您将结果通过“>”输出到文件，建议使用 `--nocolor` 参数。这样输出的文件中就不会包含色彩相关的字符，更方便后期处理。

同样以测试 500 次对 `www.azure.cn` 的 80 端口的 TCP 连接为例，跳转到 PaPing 所在的路径后，执行 `./paping -p 80 -c 500 www.azure.cn`。
	
	[kyle@centos7 ~]$ ./paping -p 80 -c 500 www.azure.cn
	paping v1.5.5 - Copyright (c) 2011 Mike Lovell
	
	Connecting to 1stcncloud.dtwscachev290.ourwebcdn.com [112.17.28.203] on TCP 80:
	
	Connected to 112.17.28.203: time=8.26ms protocol=TCP port=80
	Connected to 112.17.28.203: time=7.48ms protocol=TCP port=80
	Connected to 112.17.28.203: time=9.62ms protocol=TCP port=80
	Connected to 112.17.28.203: time=8.54ms protocol=TCP port=80
	....................
	
	Connected to 112.17.28.203: time=9.59ms protocol=TCP port=80
	Connected to 112.17.28.203: time=11.79ms protocol=TCP port=80
	Connected to 112.17.28.203: time=8.14ms protocol=TCP port=80
	Connected to 112.17.28.203: time=10.94ms protocol=TCP port=80
	Connected to 112.17.28.203: time=22.35ms protocol=TCP port=80
	
	Connection statistics:
		Attempted = 500, Connected = 500, Failed = 0 (0.00%)
	Approximate connection times:
		Minimum = 6.46ms, Maximum = 25.00ms, Average = 12.40ms
