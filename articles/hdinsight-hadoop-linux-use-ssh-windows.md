<properties
   pageTitle="在 Windows 中的基于 Linux 的 HDInsight 上将 SSH 密钥与 Hadoop 配合使用 | Azure"
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


## 在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用（预览版）

基于 Linux 的 HDInsight 群集提供了使用密码或 SSH 密钥保护 SSH 访问的选项。本文档提供有关使用 PuTTY SSH 客户端从 Windows 客户端连接到 HDInsight 的信息。

> [AZURE.NOTE] 本文中的步骤假设你使用的是 Windows 客户端。如果使用的是 Linux、Unix 或 OS X 客户端，请参阅[在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 密钥与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## 先决条件

* 适用于 Windows 客户端的 **PuTTY** 和 **PuTTYGen**。可以从 <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html" target="_blank">http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html</a> 获取这些实用工具

* 支持 HTML5 的现代 Web 浏览器

或者

* Azure PowerShell

或者

* Azure 跨平台命令行工具

## 什么是 SSH？

SSH 是用于登录远程服务器以及在其上远程执行命令的实用工具。与基于 Linux 的 HDInsight 配合使用时，SSH 可与群集头节点建立安全连接，并提供一个用来键入命令的命令行。然后，你便可以在服务器上直接执行命令。

## 创建 SSH 密钥（可选）

在创建基于 Linux 的 HDInsight 群集时，可以选择使用密码或 SSH 密钥（如果使用 SSH）向服务器进行身份验证。SSH 密钥基于证书，因此被认为更安全。如果你打算在群集上使用 SSH 密钥，请使用以下信息。

1. 打开 **PuTTYGen**。

2. 对于"要生成的密钥类型"，请选择"SSH-2 RSA"，然后单击"生成"。

	![PuTTYGen interface](./media/hdinsight-hadoop-linux-use-ssh-windows/puttygen.png)

3. 在进度条下面的区域中移动鼠标，直到进度条填满。移动鼠标可生成用于生成密钥的随机数据。

	![moving the mouse around](./media/hdinsight-hadoop-linux-use-ssh-windows/movingmouse.png)

	生成密钥后，将显示公钥。

4. 为了提高安全性，可以在"密钥通行短语"字段中输入通行短语，然后键入在"确认通行短语"字段中键入相同的值。

	![passphrase](./media/hdinsight-hadoop-linux-use-ssh-windows/key.png)
	
	> [AZURE.NOTE] 强烈建议你为密钥使用安全的通行短语。但是，如果你忘记了通行短语，将没有办法恢复它。

5. 单击"保存私钥"以在 **.ppk** 文件中保存密钥。在基于 Linux 的 HDInsight 群集上进行身份验证时，将要用到此密钥。

	> [AZURE.NOTE] 应该将此密钥存储在安全位置，因为它可以用来访问基于 Linux 的 HDInsight 群集。

6. 单击"保存公钥"以在 **.txt** 文件中保存此密钥。当你以后创建其他基于 Linux 的 HDInsight 群集时，可以重复使用该公钥。

	> [AZURE.NOTE] 公钥还会显示在 **PuTTYGen** 的顶部。你可以右键单击此字段，复制值，然后将它粘贴到某个窗体（例如，Azure 管理门户中的 HDInsight 向导）中。

## 创建基于 Linux 的 HDInsight 群集

在创建基于 Linux 的 HDInsight 群集时，必须提供以前创建的**公钥**。在 Windows 客户端中，可通过以下两种方法创建基于 Linux 的 HDInsight 群集：

* **Azure 管理门户** - 使用基于 Web 的门户创建群集

* **Azure 跨平台命令行界面 (xplat-cli)** - 使用命令行命令创建群集

上述每种方法都需要**公钥**。有关创建基于 Linux 的 HDInsight 群集的完整信息，请参阅<a href="/documentation/articles/hdinsight-hadoop-provision-linux-clusters/" target="_blank">设置基于 Linux 的 HDInsight 群集</a>。

### Azure 管理门户

使用门户创建基于 Linux 的 HDInsight 群集时，必须使用以下格式输入用户名和密码或公钥。

![Image of form asking for public key](./media/hdinsight-hadoop-linux-use-ssh-windows/ssh-key.png)

这将为指定的用户创建登录名，并启用密码身份验证或 SSH 密钥身份验证。

### Azure 跨平台命令行界面

可以在 <a href="/documentation/articles/xplat-cli/" target="_brad">Azure 跨平台命令行界面</a>中使用  `azure hdinsight cluzter create` 命令创建新的群集。

有关使用此命令的详细信息，请参阅<a href="/documentation/articles/hdinsight-hadoop-provision-linux-clusters/" target="_blank">使用自定义选项在 HDInsight 中设置 Hadoop Linux 群集</a>

## <a id="connect"></a>连接到基于 Linux 的 HDInsight 群集

1. 打开 PuTTY。

	![putty interface](./media/hdinsight-hadoop-linux-use-ssh-windows/putty.png)

2. 如果你在创建用户帐户时提供了 **SSH 密钥**，则必须执行以下步骤，以选择向群集进行身份验证时要使用的**私钥**。

	在"类别"，展开"连接"、"SSH"，然后选择"身份验证"。最后，单击"浏览"，然后选择包含**私钥**的 **.ppk**。

	![putty interface, select private key](./media/hdinsight-hadoop-linux-use-ssh-windows/puttykey.png)

3. 在"类别"中，选择"会话"。在"PuTTY 会话的基本选项"屏幕中，将 HDInsight 服务器的 SSH 地址输入到"主机名(或 IP 地址)"字段中。SSH 地址是群集名称后接 **-ssh.azurehdinsight.cn**。例如，**mycluster-ssh.azurehdinsight.cn**。

	![putty interface with ssh address entered](./media/hdinsight-hadoop-linux-use-ssh-windows/puttyaddress.png)

4. 若要保存连接信息供日后使用，请在"保存的会话"下面输入此连接的名称，然后单击"保存"。该连接将会添加到已保存会话的列表中。

5. 单击"打开"以连接到群集。

	> [AZURE.NOTE] 如果这是第一次连接到群集，你将收到安全警报。这是正常情况 - 请选择"是"以缓存服务器的 RSA2 密钥并继续。

6. 出现提示时，请输入你在创建群集时输入的用户。如果为用户提供了**密码**，系统也会提示你输入该密码。

## 添加其他帐户

2. 根据前面所述，为新用户帐户生成新的**公钥**和**私钥**

1. 在与群集建立的 SSH 会话中，使用以下命令添加新用户。

		sudo adduser --disabled-password <username> 

	这将创建一个新的用户帐户，但会禁用密码身份验证。

2. 使用以下命令创建用于保存密钥的目录和文件。

		sudo mkdir -p /home/<username>/.ssh
		sudo touch /home/<username>/.ssh/authorized_keys
		sudo nano /home/<username>/.ssh/authorized_keys

3. 当 nano 编辑器打开时，请复制并粘贴新用户帐户的**公钥**内容。最后，使用 **CTRL-X** 保存文件并退出编辑器。

	![image of nano editor with example key](./media/hdinsight-hadoop-linux-use-ssh-windows/nano.png)

4. 使用以下命令将 .ssh 文件夹和内容的所有权更改为新用户帐户。

		sudo chown -hR <username>:<username> /home/<username>/.ssh

5. 现在，你应该可以使用新用户帐户和**私钥**向服务器进行身份验证。

## <a id="tunnel"></a>SSH 隧道

还可以使用 SSH 来以隧道方式将本地请求（例如 Web 请求）传送到 HDInsight 群集。然后，请求将路由到请求的资源，就像它是来源于 HDInsight 群集头节点一样。

在 HDInsight 群集上访问使用群集中头节点或辅助节点内部域名的基于 Web 的服务时，此功能特别有用。例如，Ambari 网页上的某些部分使用类似于 **headnode0.mycluster.d1.internal.chinacloudapp.cn** 的内部域名。无法从群集外部解析这些名称，但是，通过 SSH 以隧道方式传送的请求将源自于群集内部，并可正确解析。

使用以下步骤创建 SSH 隧道，并将浏览器配置为使用 SSH 隧道连接到群集。

1. 打开 PuTTY，然后根据[连接](#connect) 部分中所述输入你的连接信息。

2. 在对话框左侧的"类别"部分中，展开"连接"、"SSH"，最后选择"隧道"。

2. 在"用于控制 SSH 端口转发的选项"窗体中提供以下信息。

	* **源端口** - 客户端上要转发的端口。例如 **9876**

	* **目标** - 基于 Linux 的 HDInsight 群集的 SSH 地址。例如 **mycluster-ssh.azurehdinsight.cn**

	* **动态** - 启用动态 SOCKS 代理路由

	![image of tunneling options](./media/hdinsight-hadoop-linux-use-ssh-windows/puttytunnel.png)

3. 最后，选择"添加"以添加这些设置，然后选择"打开"以打开 SSH 连接。

4. 出现提示时，请登录到服务器。这将会建立 SSH 会话并启用隧道。

2. 将客户端程序（例如 Firefox）配置为使用 **localhost:9876** 作为 **SOCKS v5** 代理。Firefox 中的设置如下所示。

	![image of Firefox settings](./media/hdinsight-hadoop-linux-use-ssh-windows/socks.png)

	> [AZURE.NOTE] 选择"远程 DNS"会使用 HDInsight 群集解析 DNS 请求。如果取消选择此项，将在本地解析 DNS。

	<!--You can verify that traffic is being routed through the tunnel by vising a site such as <a href="http://www.whatismyip.com/" target="_blank">http://www.whatismyip.com/</a> with the proxy settings enabled and disabled in Firefox. While enabled, the IP address will be for a machine in the Microsoft Azure datacenter.-->

### 浏览器扩展

虽然你可以将浏览器配置为使用隧道，但是，你通常不想要通过隧道路由所有流量。FoxyProxy 等浏览器扩展支持 URL 请求的模式匹配（仅限 FoxyProxy Standard 或 Plus），以便只通过隧道发送特定 URL 的请求。

如果安装了 **FoxyProxy Standard**，请使用以下步骤将它配置为仅通过隧道转发 HDInsight 的流量。

1. 在浏览器中打开 FoxyProxy 扩展。例如，在 Firefox 中，选择地址字段旁边的 FoxyProxy 图标。

	![foxyproxy icon](./media/hdinsight-hadoop-linux-use-ssh-windows/foxyproxy.png)

2. 选择"添加新代理"、"常规"选项卡上，然后输入 **HDInsight** 的代理名称。

	![foxyproxy general](./media/hdinsight-hadoop-linux-use-ssh-windows/foxygeneral.png)

3. 选择"代理详细信息"选项卡并填充以下字段。

	* **主机或 IP 地址** - localhost，因为我们要在本地计算机上使用 SSH 隧道

	* **端口** - 用于 SSH 隧道的端口

	* **SOCKS 代理** - 选择此选项可让浏览器使用隧道作为代理

	* **SOCKS v5** - 选择此选项可为代理设置所需的版本

	![foxyproxy proxy](./media/hdinsight-hadoop-linux-use-ssh-windows/foxyproxyproxy.png)

4. 选择"URL 模式"选项卡，然后选择"添加新模式"。使用以下设置来定义模式，然后单击"确定"。

	* **模式名称** - **headnode** - 这是模式的友好名称

	* **URL 模式** - **\*headnode\*** - 定义将任何 URL 与其包含的单词 **headnode** 匹配的模式。

	![foxyproxy pattern](./media/hdinsight-hadoop-linux-use-ssh-windows/foxypattern.png)

4. 选择"确定"以添加该代理并关闭"代理设置"

5. 在 FoxyProxy 对话框的顶部，将"选择模式"更改为"根据预定义的模式和优先级使用代理"，然后选择"关闭"。

	![foxyproxy select mode](./media/hdinsight-hadoop-linux-use-ssh-windows/selectmode.png)

执行这些步骤后，只会通过 SSL 隧道路由包含字符串 **headnode** 的 URL 的请求。

## 后续步骤

现在，你已了解如何使用 SSH 密钥进行身份验证，以及如何将 MapReduce 与 HDInsight 上的 Hadoop 配合使用。

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)
 

<!--HONumber=50-->