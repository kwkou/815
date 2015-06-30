<properties
   pageTitle="在 Linux、Unix 或 OS X 中的基于 Linux 的 HDInsight 上将 SSH 密钥与 Hadoop 配合使用 | Azure"
   description="了解如何创建和使用 SSH 密钥，以便向基于 Linux 的 HDInsight 群集进行身份验证。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="03/20/2015"
    wacn.date="04/15/2015"
    />


## 在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 密钥与基于 Linux 的 Hadoop 配合使用（预览版）

基于 Linux 的 HDInsight 群集提供了使用密码或 SSH 密钥保护 SSH 访问的选项。本文档提供有关在 Linux、Unix 或 OS X 客户端中将 SSH 与 hdinsight 配合使用的信息。

> [AZURE.NOTE] 本文中的步骤假设你使用 Linux、Unix 或 OS X 客户端。虽然可以在装有提供 `ssh` 和 `ssh-keygen` 的包（例如，适用于 Windows 的 Git）的Windows 客户端上执行这些步骤，但是，我们建议在 Windows 客户端上遵循[在 Windows 中将 SSH 与基于 Linux 的 HDInsight (Hadoop) 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows)中的步骤。

## 先决条件

* 适用于 Linux、Unix 和 OS X 客户端的 **ssh-keygen** 和 **ssh**。你的操作系统通常会随附此实用工具，你也可以通过包管理系统获取此实用工具

* 支持 HTML5 的现代 Web 浏览器

或者

* Azure 跨平台命令行工具

## 什么是 SSH？

SSH 是用于登录远程服务器以及在其上远程执行命令的实用工具。与基于 Linux 的 HDInsight 配合使用时，SSH 可与群集头节点建立安全连接，并提供一个用来键入命令的命令行。然后，你便可以在服务器上直接执行命令。

## 创建 SSH 密钥（可选）

在创建基于 Linux 的 HDInsight 群集时，可以选择使用密码或 SSH 密钥（如果使用 SSH）向服务器进行身份验证。SSH 密钥基于证书，因此被认为更安全。如果你打算在群集上使用 SSH 密钥，请使用以下信息。

1. 打开一个终端会话，然后使用以下命令来查看是否存在任何现有的 SSH 密钥

		ls -al ~/.ssh

	在目录列表中查找以下文件。这是公共 SSH 密钥的公用名。

	* id\_dsa.pub
	* id\_ecdsa.pub
	* id\_ed25519.pub
	* id\_rsa.pub

2. 如果你不想要使用现有文件，或者没有任何现有的 SSH 密钥，请使用以下命令生成新文件。

		ssh-keygen -t rsa

	系统将提示你输入以下信息：

	* 文件位置 - 默认为 ~/.ssh/id\_rsa。
	* 通行短语 - 系统将提示你重新输入此信息
	
		> [AZURE.NOTE] 强烈建议你为密钥使用安全的通行短语。但是，如果你忘记了通行短语，将没有办法恢复它。

	完成命令后，你将获得两个新文件：一个是私钥（例如，**id\_rsa**），一个是公钥（例如，**id\_rsa.pub**）。

## 创建基于 Linux 的 HDInsight 群集

在创建基于 Linux 的 HDInsight 群集时，必须提供以前创建的**公钥**。在 Linux、Unix 或 OS X 客户端中，可通过以下两种方式创建 HDInsight 群集：

* **Azure 管理门户** - 使用基于 Web 的门户创建群集

* **Azure 跨平台命令行界面 (xplat-cli)** - 使用命令行命令创建群集

其中的每种方法都需要**密码**或**公钥**。有关创建基于 Linux 的 HDInsight 群集的完整信息，请参阅<a href="/documentation/articles/hdinsight-hadoop-provision-linux-clusters/" target="_blank">设置基于 Linux 的 HDInsight 群集</a>。

### Azure 管理门户

使用门户创建基于 Linux 的 HDInsight 群集时，必须输入"SSH 用户名"，然后选择输入"密码"或"SSH 公钥"。如果你选择了"SSH 公钥"，则必须将公钥（包含在扩展名为 **.pub** 的文件中）粘贴到以下表单中。

![Image of form asking for public key](./media/hdinsight-hadoop-linux-use-ssh-unix/ssh-key.png)

> [AZURE.NOTE] 密钥文件只是一个文本文件。其内容应如下所示。
> ```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCelfkjrpYHYiks4TM+r1LVsTYQ4jAXXGeOAF9Vv/KGz90pgMk3VRJk4PEUSELfXKxP3NtsVwLVPN1l09utI/tKHQ6WL3qy89WVVVLiwzL7tfJ2B08Gmcw8mC/YoieT/YG+4I4oAgPEmim+6/F9S0lU2I2CuFBX9JzauX8n1Y9kWzTARST+ERx2hysyA5ObLv97Xe4C2CQvGE01LGAXkw2ffP9vI+emUM+VeYrf0q3w/b1o/COKbFVZ2IpEcJ8G2SLlNsHWXofWhOKQRi64TMxT7LLoohD61q2aWNKdaE4oQdiuo8TGnt4zWLEPjzjIYIEIZGk00HiQD+KCB5pxoVtp user@system
> ```

这将使用你提供的密码或公钥为指定的用户创建登录名。

### Azure 跨平台命令行界面

可以在 <a href="/documentation/articles/xplat-cli/" target="_brad">Azure 跨平台命令行界面</a>中使用  `azure hdinsight cluster create` 命令创建新的群集。

有关使用此命令的详细信息，请参阅<a href="/documentation/articles/hdinsight-hadoop-provision-linux-clusters/" target="_blank">使用自定义选项在 HDInsight 中设置 Hadoop Linux 群集</a>

## 连接到基于 Linux 的 HDInsight 群集

在终端会话中，使用 SSH 命令并通过提供用户名和群集地址来连接到群集。

* **SSH 地址** - 群集名称，后接 **-ssh.azurehdinsight.cn**。例如，**mycluster-ssh.azurehdinsight.cn**。

* **用户名** - 创建群集时提供的 SSH 用户名

以下示例将以用户 **me** 的身份连接到群集 **mycluster**。

	ssh me@mycluster-ssh.azurehdinsight.cn

如果为用户帐户使用了**密码**，系统将提示你输入密码。

如果使用了由通行短语保护的 **SSH 密钥**，系统将提示你输入通行短语；否则，SSH 将尝试在客户端上使用某个本地私钥自动进行身份验证。

> [AZURE.NOTE] 如果 SSH 未使用正确的**私钥**自动进行身份验证，请使用 **-i** 参数并指定私钥的路径。以下示例将从 `~/.ssh/id_rsa` 加载**私钥**。
> 
> `ssh -i ~/.ssh/id_rsa me@mycluster-ssh.azurehdinsight.cn`

## 添加其他帐户

2. 根据[创建 SSH 私钥](#create) 部分中所述，为新用户帐户生成新的**公钥**和**私钥**。

	> [AZURE.NOTE] 私钥应该是在用户用来连接到群集的客户端上生成的，或者在创建后已安全传输到此类客户端。

1. 在与群集建立的 SSH 会话中，使用以下命令添加新用户。

		sudo adduser --disabled-password <username> 

	这将创建一个新的用户帐户，但会禁用密码身份验证。

2. 使用以下命令创建用于保存密钥的目录和文件。

		sudo mkdir -p /home/<username>/.ssh
		sudo touch /home/<username>/.ssh/authorized_keys
		sudo nano /home/<username>/.ssh/authorized_keys

3. 当 nano 编辑器打开时，请复制并粘贴新用户帐户的**公钥**内容。最后，使用 **CTRL-X** 保存文件并退出编辑器。

	![image of nano editor with example key](./media/hdinsight-hadoop-linux-use-ssh-unix/nano.png)

4. 使用以下命令将 .ssh 文件夹和内容的所有权更改为新用户帐户。

		sudo chown -hR <username>:<username> /home/<username>/.ssh

5. 现在，你应该可以使用新用户帐户和**私钥**向服务器进行身份验证。

## <a id="tunnel"></a>SSH 隧道

还可以使用 SSH 来以隧道方式将本地请求（例如 Web 请求）传送到 HDInsight 群集。然后，请求将路由到请求的资源，就像它是来源于 HDInsight 群集头节点一样。

在 HDInsight 群集上访问使用群集中头节点或辅助节点内部域名的基于 Web 的服务时，此功能特别有用。例如，Ambari 网页上的某些部分使用类似于 **headnode0.mycluster.d1.internal.chinacloudapp.cn** 的内部域名。无法从群集外部解析这些名称，但是，通过 SSH 以隧道方式传送的请求将源自于群集内部，并可正确解析。

使用以下步骤创建 SSH 隧道，并将浏览器配置为使用 SSH 隧道连接到群集。

1. 可使用以下命令来与群集头节点建立 SSH 隧道。

		ssh -C2qTnNf -D 9876 username@clustername-ssh.azurehdinsight.cn

	这会创建一个通过 SSH 将流量路由到群集本地端口 9876 的连接。选项包括：

	* **D 8080** - 通过隧道路由流量的本地端口

	* **C** - 压缩所有数据，因为 Web 流量大多为文本

	* **2** - 强制 SSH 仅尝试协议版本 2

	* **q** - 静默模式

	* **T** - 禁用 pseudo-tty 分配，因为我们只要转发端口
	
	* **n** - 防止读取 STDIN，因为我们只要转发端口

	* **N** - 不执行远程命令，因为我们只要转发端口

	* **f** - 在后台运行

	如果使用 **SSH 密钥**配置了群集，则可能需要使用 `-i` 参数，并指定 SSH 私钥的路径。

	完成命令后，发送到本地计算机上的端口 9876 的流量将通过 SSL 路由到群集头节点，并且看上去来源于该节点。

2. 将客户端程序（例如 Firefox）配置为使用 **localhost:9876** 作为 **SOCKS v5** 代理。Firefox 中的设置如下所示。

	![image of Firefox settings](./media/hdinsight-hadoop-linux-use-ssh-unix/socks.png)

	> [AZURE.NOTE] 选择"远程 DNS"会使用 HDInsight 群集解析 DNS 请求。如果取消选择此项，将在本地解析 DNS。

	<!--You can verify that traffic is being routed through the tunnel by vising a site such as <a href="http://www.whatismyip.com/" target="_blank">http://www.whatismyip.com/</a> with the proxy settings enabled and disabled in Firefox. While enabled, the IP address will be for a machine in the Microsoft Azure datacenter.-->

### 浏览器扩展

虽然你可以将浏览器配置为使用隧道，但是，你通常不想要通过隧道路由所有流量。FoxyProxy 等浏览器扩展支持 URL 请求的模式匹配（仅限 FoxyProxy Standard 或 Plus），以便只通过隧道发送特定 URL 的请求。

如果安装了 **FoxyProxy Standard**，请使用以下步骤将它配置为仅通过隧道转发 HDInsight 的流量。

1. 在浏览器中打开 FoxyProxy 扩展。例如，在 Firefox 中，选择地址字段旁边的 FoxyProxy 图标。

	![foxyproxy icon](./media/hdinsight-hadoop-linux-use-ssh-unix/foxyproxy.png)

2. 选择"添加新代理"、"常规"选项卡上，然后输入 **HDInsight** 的代理名称。

	![foxyproxy general](./media/hdinsight-hadoop-linux-use-ssh-unix/foxygeneral.png)

3. 选择"代理详细信息"选项卡并填充以下字段。

	* **主机或 IP 地址** - localhost，因为我们要在本地计算机上使用 SSH 隧道

	* **端口** - 用于 SSH 隧道的端口

	* **SOCKS 代理** - 选择此选项可让浏览器使用隧道作为代理

	* **SOCKS v5** - 选择此选项可为代理设置所需的版本

	![foxyproxy proxy](./media/hdinsight-hadoop-linux-use-ssh-unix/foxyproxyproxy.png)

4. 选择"URL 模式"选项卡，然后选择"添加新模式"。使用以下设置来定义模式，然后单击"确定"。

	* **模式名称** - **headnode** - 这是模式的友好名称

	* **URL 模式** - **\*headnode\*** - 定义将任何 URL 与其包含的单词 **headnode** 匹配的模式。

	![foxyproxy pattern](./media/hdinsight-hadoop-linux-use-ssh-unix/foxypattern.png)

4. 选择"确定"以添加该代理并关闭"代理设置"

5. 在 FoxyProxy 对话框的顶部，将"选择模式"更改为"根据预定义的模式和优先级使用代理"，然后选择"关闭"。

	![foxyproxy select mode](./media/hdinsight-hadoop-linux-use-ssh-unix/selectmode.png)

执行这些步骤后，只会通过 SSL 隧道路由包含字符串 **headnode** 的 URL 的请求。

## 后续步骤

现在，你已了解如何使用 SSH 密钥进行身份验证，以及如何将 MapReduce 与 HDInsight 上的 Hadoop 配合使用。

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive)

* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig)

* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce)
 

<!--HONumber=50-->