<properties
    pageTitle="测试 Azure VM 网络吞吐量 | Azure"
    description="了解如何测试 Azure 虚拟机的网络吞吐量。"
    services="virtual-network"
    documentationcenter="na"
    author="steveesp"
    manager="Gerald DeGrace"
    editor="" />
<tags
    ms.assetid=""
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/21/2017"
    wacn.date="03/31/2017"
    ms.author="steveesp" />  


# 带宽/吞吐量测试 (NTTTCP)

在 Azure 中测试网络吞吐量性能时，最好使用以要测试的网络为目标并能最大程度减少其他资源（这些资源可能会对性能产生影响）的使用的工具。建议使用 NTTTCP。

将此工具复制到大小相同的两个 Azure VM 中。一个 VM 充当发送方，另一个充当接收方。

#### 部署 VM 以进行测试
为了达到此测试的目的，两个 VM 应位于同一云服务或同一可用性集中，这样便可使用其内部 IP 并从测试中排除负载均衡器。也可以使用 VIP 进行测试，但这类测试不在本文档的讨论范围内。
 
记下接收方的 IP 地址。暂且将该 IP 称为“a.b.c.r”

记下 VM 上的核心数。暂且将其称为“#num\_cores”
 
在发送方 VM 和接收方 VM 上运行 NTTTCP 测试 300 秒（5 分钟）。

提示：第一次设置此测试时，可以尝试更短的测试时间，以更快地获取反馈。在工具按预期工作后，将测试时间延长到 300 秒，以获取最准确的结果。

> [AZURE.NOTE]
发送方**和**接收方必须指定**相同的**测试持续时间参数 (-t)。

测试单个 TCP 流 10 秒：

接收方参数：`ntttcp -r -t 10 -P 1`

发送方参数：`ntttcp -s10.27.33.7 -t 10 -n 1 -P 1`

> [AZURE.NOTE]
以上示例应仅用于确认配置。本文档稍后会介绍测试的有效示例。

## 测试运行 WINDOWS 的 VM：

#### 在 VM 上安装 NTTTCP。

下载最新版本：
<https://gallery.technet.microsoft.com/NTttcp-Version-528-Now-f8b12769>

如果已转移，则进行搜索：<https://www.bing.com/search?q=ntttcp+download>< -- 应该是第一个链接

请考虑将 NTTTCP 放在单独的文件夹中，如 c:\\tools

#### 允许 NTTTCP 通过 Windows 防火墙
在接收方上，在 Windows 防火墙上创建允许规则，以允许接收 NTTTCP 流量。最简单的方法是按名称允许整个 NTTTCP 程序，而不是允许特定的 TCP 端口入站。

允许 ntttcp 通过 Windows 防火墙，如下所示：

    netsh advfirewall firewall add rule program=<PATH>\ntttcp.exe name="ntttcp" protocol=any dir=in action=allow enable=yes profile=ANY

例如，如果已将 ntttcp.exe 复制到“c:\\tools”文件夹中，则此命令为：

    netsh advfirewall firewall add rule program=c:\tools\ntttcp.exe name="ntttcp" protocol=any dir=in action=allow enable=yes profile=ANY

#### 运行 NTTTCP 测试

在接收方上启动 NTTTCP（**从 CMD 运行**，而不是从 PowerShell 运行）：

    ntttcp -r -m [2*#num_cores],*,a.b.c.r -t 300

如果 VM 有四个核心且 IP 地址为 10.0.0.4，则为如下所示：

    ntttcp -r -m 8,*,10.0.0.4 -t 300

在发送方上启动 NTTTCP（**从 CMD 运行**，而不是从 PowerShell 运行）：

    ntttcp -s -m 8,*,10.0.0.4 -t 300

等待结果。

## 测试运行 LINUX 的 VM：

使用 nttcp-for-linux。可从 <https://github.com/Microsoft/ntttcp-for-linux> 中获取

在 Linux VM（发送方和接收方）上，运行以下命令以在 VM 上准备 nttcp-for-linux：

CentOS — 安装 Git：

      yum install gcc -y  
      yum install git -y

Ubuntu — 安装 Git：

     apt-get -y install build-essential  
     apt-get -y install git

在两个 VM 上同时获取并安装：

     git clone <https://github.com/Microsoft/ntttcp-for-linux>
     cd ntttcp-for-linux/src
     make && make install

如 Windows 示例中一样，假设 Linux 接收方的 IP 为 10.0.0.4

在接收方上启动 NTTTCP-for-Linux：

    ntttcp -r -t 300

然后在发送方上运行：

    ntttcp -s10.0.0.4 -t 300

 
如果未给定时间参数，默认的测试持续时间为 60 秒

## 后续步骤
* 根据得到的结果，也许能够为方案[优化网络吞吐量计算机](/documentation/articles/virtual-network-optimize-network-bandwidth/)。
* 通过 [Azure 虚拟网络常见问题解答 (FAQ)](/documentation/articles/virtual-networks-faq/) 了解详细信息

<!---HONumber=Mooncake_0327_2017-->