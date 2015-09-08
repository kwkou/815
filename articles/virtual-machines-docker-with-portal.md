<properties title="How to use the Docker VM Extension with the Azure Portal" pageTitle="使用适用于 Azure 上的 Linux 的 Docker VM 扩展" description="介绍 Docker 以及 Azure 虚拟机扩展，并说明如何在 Azure 上，使用 azure-cli 命令界面通过命令行以编程方式创建用作 Docker 主机的虚拟机。" metaKeywords="linux, virtual machines, vm, azure, docker, linux containers,  lxc, virtualization" services="virtual-machines" solutions="dev-test" documentationCenter="virtual-machines" authors="rasquill" videoId="" scriptId="" manager="timlt" />
<tags
	ms.service="virtual-machines"
	ms.date="05/25/2015"
	wacn.date="08/29/2015"/>

# 在 Azure 门户中使用 Docker VM 扩展

[Docker](https://www.docker.com/) 是最常用的虚拟化技术之一，它使用 [Linux 容器](http://zh.wikipedia.org/wiki/LXC)而不是虚拟机作为在共享资源上隔离数据和执行计算的方法。可以在 [Azure Linux 代理]中使用 Docker VM 扩展，以创建可在 Azure 上为应用程序托管任意数量的容器的 Docker VM。

本节内容

+ [从映像库创建新的 VM]
+ [创建 Docker 证书]
+ [添加 Docker VM 扩展]
+ [测试 Docker 客户端和 Azure Docker 主机]
+ [后续步骤]

> [AZURE.NOTE]本主题介绍如何从 Azure 门户创建 Docker VM。若要了解如何通过命令行创建 Docker VM，请参阅 [如何从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展]。

## 从映像库创建新的 VM
第一个步骤需要可支持 Docker VM 扩展的 Linux 映像中的 Azure VM，使用映像库的 Ubuntu 14.04 LTS 映像作为示例服务器映像，并将 Ubuntu 14.04 Desktop 用作客户端。在门户中，单击左下角的“+ 新建”以创建新的 VM 实例，并从可用选项或从完整映像库中选择 Ubuntu 14.04 LTS 映像，如下所示。

> [AZURE.NOTE]目前，只有 2014 年 7 月以后发布的 Ubuntu 14.04 LTS 映像才支持 Docker VM 扩展。

![创建新的 Ubuntu 映像](./media/virtual-machines-docker-with-portal/ChooseUbuntu.png)

## 创建 Docker 证书

在创建 VM 后，请确保客户端计算机上已安装 Docker。（有关详细信息，请参阅 [Docker 安装说明](https://docs.docker.com/installation/#installation)。）

根据[使用 https 运行 Docker] 来创建 Docker 通信的证书和密钥文件，并将其放置在客户端计算机上的 **`~/.docker`** 目录中。

> [AZURE.NOTE]门户中的 Docker VM 扩展目前需要 base64 编码的凭据。

在命令行中，使用 **`base64`** 或其他惯用的编码工具来创建 base64 编码的主题。使用一组简单的证书和密钥文件来执行此操作，如下所示：

```
 ~/.docker$ l
 ca-key.pem  ca.pem  cert.pem  key.pem  server-cert.pem  server-key.pem
 ~/.docker$ base64 ca.pem > ca64.pem
 ~/.docker$ base64 server-cert.pem > server-cert64.pem
 ~/.docker$ base64 server-key.pem > server-key64.pem
 ~/.docker$ l
 ca64.pem    ca.pem    key.pem            server-cert.pem   server-key.pem
 ca-key.pem  cert.pem  server-cert64.pem  server-key64.pem
```

## 添加 Docker VM 扩展
若要添加 Docker VM 扩展，请找到你创建的 VM 实例，向下滚动到“扩展”，然后单击它以打开“VM 扩展”，如下所示。
> [AZURE.NOTE]只有预览版门户支持此功能：https://manage.windowsazure.cn

![](./media/virtual-machines-docker-with-portal/ClickExtensions.png)
### 添加扩展
单击“+ 添加”显示可添加到此 VM 的可用 VM 扩展。

![](./media/virtual-machines-docker-with-portal/ClickAdd.png)
### 选择 Docker VM 扩展
选择“Docker VM 扩展”，随后会显示 Docker 说明和重要链接；然后单击底部的“创建”以开始执行安装过程。

![](./media/virtual-machines-docker-with-portal/ChooseDockerExtension.png)

![](./media/virtual-machines-docker-with-portal/CreateButtonFocus.png)
### 添加证书和密钥文件：

在窗体字段中，输入 base64 编码版本的 CA 证书、你的服务器证书以及服务器密钥，如下图所示。

![](./media/virtual-machines-docker-with-portal/AddExtensionFormFilled.png)

> [AZURE.NOTE]请注意，（如上图所示）已按默认填入了 2376。可以在此处输入任何终结点，但下一步是打开匹配的终结点。如果更改默认值，请确保在下一步打开匹配的终结点。

## 添加 Docker 通信终结点
在创建的资源组中查看 VM 时，可以向下滚动单击“终结点”以查看 VM 上的终结点，如下所示。

![](./media/virtual-machines-docker-with-portal/AddingEndpoint.png)

单击“+ 添加”以添加另一个终结点，在默认情况下，请输入终结点的名称（在此示例中为 **docker**），专用和公用端口都是 2376。保留显示 **TCP** 的通信协议值，然后单击“确定”以创建终结点。

![](./media/virtual-machines-docker-with-portal/AddEndpointFormFilledOut.png)


## 测试 Docker 客户端和 Azure Docker 主机
查找并复制 VM 的域名，然后在客户端计算机的命令行中，键入 `docker --tls -H tcp://`*dockerextension*`.chinacloudapp.cn:2376 info`（其中 *dockerextension* 已替换为 VM 的子域）。

结果应如下所示：

```
$ docker --tls -H tcp://dockerextension.chinacloudapp.cn:2376 info
Containers: 0
Images: 0
Storage Driver: devicemapper
 Pool Name: docker-8:1-131214-pool
 Pool Blocksize: 65.54 kB
 Data file: /var/lib/docker/devicemapper/devicemapper/data
 Metadata file: /var/lib/docker/devicemapper/devicemapper/metadata
 Data Space Used: 305.7 MB
 Data Space Total: 107.4 GB
 Metadata Space Used: 729.1 kB
 Metadata Space Total: 2.147 GB
 Library Version: 1.02.82-git (2013-10-04)
Execution Driver: native-0.2
Kernel Version: 3.13.0-36-generic
WARNING: No swap limit support
```

完成上述步骤后，Azure VM 上即会出现配置为从其他客户端远程连接的完全正常的 Docker 主机。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤

现在，你可以转到 [Docker 用户指南]并开始使用你的 Docker VM。如果你想要通过命令行界面在 Azure VM 上自动创建 Docker主机，请参阅 [如何从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展]

<!--Anchors-->
[从映像库创建新的 VM]: #createvm
[创建 Docker 证书]: #dockercerts
[添加 Docker VM 扩展]: #adddockerextension
[测试 Docker 客户端和 Azure Docker 主机]: #testclientandserver
[后续步骤]: #next-steps

<!--Image references-->
[StartingPoint]: ./media/StartingPoint.png
[StartingPoint]: ./media/StartingPoint.png
[StartingPoint]: ./media/StartingPoint.png
[StartingPoint]: ./media/StartingPoint.png
[StartingPoint]: ./media/StartingPoint.png
[StartingPoint]: ./media/StartingPoint.png
[StartingPoint]: ./media/StartingPoint.png
[StartingPoint]: ./media/StartingPoint.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png


<!--Link references-->
[How to use the Docker VM Extension from Azure Cross-Platform Interface (xplat-cli)]: /zh-cn/documentation/articles/virtual-machines-docker-with-xplat-cli/
[Azure Linux 代理]: /zh-cn/documentation/articles/virtual-machines-linux-agent-user-guide/
[Link 3 to another azure.microsoft.com documentation topic]: /zh-cn/documentation/articles/storage-whatis-account/

[使用 https 运行 Docker]: http://docs.docker.com/articles/https/
[Docker 用户指南]: https://docs.docker.com/userguide/

<!---HONumber=67-->