<properties
	pageTitle="应用程序网关中如何获取访问者源 IP"
	description="介绍如何在应用程序网关中获取访问者源 IP。"
	services="application-gateway"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags=""/>

<tags
	ms.service="application-gateway-aog"
	ms.date="10/27/2016"
	wacn.date="11/03/2016"/>

# 应用程序网关中如何获取访问者源 IP

###问题：

在使用应用程序网关时需要获取访问者源 IP

###解决方法：

对于 HTTP 的请求应用程序网关会通过自身的 CA 地址（指应用程序网关实例在虚拟网络内的地址）对后端池内的服务器实例发起新的 HTTP 连接，相当于将接收到的 HTTP 请求以新的 TCP 连接的方式对后端服务器做转发，之后应用程序网关将接收到的数据再返回给客户端。

由于客户端不是直接访问应用程序网关后端的服务器实例，所以没有办法直接从后端服务器上面获取访问者的源 IP，但是我们可以抓取网络包并从网络包内获取该信息。

1.在后端服务器执行抓包操作。

- 对于 Linux 的机器，可以使用 tcpdump 工具执行下面的命令进行抓包。

		tcpdump -i eth0 -w server.cap

- 对于 Windows 的机器，可以使用 Wireshark 进行抓包，Wireshark 可以从[这个网站](https://www.wireshark.org/#download)进行下载。
    
2.完成抓包后，使用 Wireshark 打开抓包文件并在 filter 框内输入 http.x_forwarded_for 并点击回车。

![](./media/aog-web-app-get-user-ip/filter.png)
 
3.展开检索之后的任意一条记录，查看 X-Forwarded-For 字段可以拿到访问者的源 IP。在下面示例中，我们可以得知访问者的源 IP 为 16.22.255.29，而使用的源端口为 61532。

	Frame 35: 727 bytes on wire (5816 bits), 727 bytes captured (5816 bits)
	Ethernet II, Src: AristaNe_d0:44:61 (00:1c:73:d0:44:61), Dst: Microsof_00:6c:8f (00:17:fa:00:6c:8f)
	Internet Protocol Version 4, Src: 10.1.2.5, Dst: 10.1.0.4
	Transmission Control Protocol, Src Port: 49166 (49166), Dst Port: 80 (80), Seq: 3247603946, Ack: 451509128, Len: 673
    ……
    X-P2P-PeerDist: Version=1.1\r\n
    X-P2P-PeerDistEx: MinContentInformation=1.0, MaxContentInformation=2.0\r\n
    X-FORWARDED-PROTO: http\r\n
    X-FORWARDED-PORT: 80\r\n
    X-Original-URL: /\r\n
    X-Forwarded-For: 16.22.255.29:61532\r\n
    X-ARR-LOG-ID: d84257ea-5b31-4722-abfd-e7f8adb4013e\r\n
    \r\n
    [Full request URI: http://139.219.238.60/]
    [HTTP request 2/2]
    [Prev request in frame: 32]
    [Response in frame: 36]
