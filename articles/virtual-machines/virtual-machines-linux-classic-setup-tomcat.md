<properties
    pageTitle="在 Linux 虚拟机上设置 Apache Tomcat | Azure"
    description="了解如何使用运行 Linux 的 Azure 虚拟机设置 Apache Tomcat7。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="NingKuang"
    manager="timlt"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="45ecc89c-1cb0-4e80-8944-bd0d0bbedfdc"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/15/2015"
    wacn.date="03/28/2017"
    ms.author="ningk" />  


# 使用 Azure 在 Linux 虚拟机上设置 Tomcat7
Apache Tomcat（简称 Tomcat，以前也称为 Jakarta Tomcat）是由 Apache Software Foundation \(ASF\) 开发的一个开源 Web 服务器和 servlet 容器。Tomcat 实现 Sun Microsystems 提出的 Java Servlet 和 JavaServer Pages \(JSP\) 规范。Tomcat 提供用于运行 Java 代码的纯 Java HTTP Web 服务器环境。在最简单的配置中，Tomcat 在单个操作系统进程中运行。此进程运行 Java 虚拟机 \(JVM\)。浏览器向 Tomcat 发出的每个 HTTP 请求在 Tomcat 进程中作为单独线程进行处理。

> [AZURE.IMPORTANT]
Azure 具有用于创建和处理资源的两个不同的部署模型：[Azure Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用经典部署模型。我们建议在大多数新部署中使用 Resource Manager 模型。若要使用 Resource Manager 模板通过 Open JDK 和 Tomcat 部署 Ubuntu VM，请参阅[此文](https://github.com/Azure/azure-quickstart-templates/tree/master/openjdk-tomcat-ubuntu-vm/)。

在本文中，我们将在 Linux 映像中安装 Tomcat7，并将其部署到 Azure。

你将学习以下内容：

* 如何在 Azure 中创建虚拟机。
* 如何准备适用于 Tomcat7 的虚拟机。
* 如何安装 Tomcat7。

本文假设读者已拥有 Azure 订阅。如果没有，可在 [Azure 网站](https://www.azure.cn/)上注册一个免费试用订阅。

本文假设读者具备 Tomcat 和 Linux 的基本实践知识。

## 阶段 1：创建映像
在此阶段，我们将在 Azure 中使用 Linux 映像创建虚拟机。

### 步骤 1：生成 SSH 身份验证密钥
SSH 是面向系统管理员的重要工具。但是，我们并不建议基于人工确定的密码配置访问安全性。恶意用户可以根据用户名和弱密码侵入你的系统。

好消息是，有办法使远程访问保持打开状态，而无需担心密码。此方法包括使用非对称加密进行身份验证。用户的私钥是授予身份验证的密钥。甚至可以锁定用户的帐户，以禁止密码身份验证。

此方法的另一个优点是不需要使用不同的密码来登录到不同的服务器。可以使用个人私钥在所有服务器上进行身份验证，因而不必要记住多个密码。


按照下列步骤进行操作可生成 SSH 身份验证密钥。

1. 从以下位置下载并安装 PuTTYgen：[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)
2. 运行 Puttygen.exe。
3. 单击“生成”以生成密钥。在此过程中，可以通过将鼠标放在窗口中的空白区域上来增加随机性。
![显示“生成新密钥”按钮的 PuTTY 密钥生成器屏幕截图][1]
4. 生成过程结束后，Puttygen.exe 将显示新的公钥。
![显示新公钥和“保存私钥”按钮的 PuTTY 密钥生成器屏幕截图][2]
5. 选择并复制公钥，然后将它保存到名为 publicKey.pem 的文件中。不要单击“保存公钥”，因为保存的公钥的文件格式不同于所需的公钥。
6. 单击“保存私钥”，并将其保存到名为 privateKey.ppk 的文件中。

### 步骤 2：在 Azure 门户中创建映像
1. 在[门户](https://portal.azure.cn/)中，单击任务栏中的“新建”创建映像。然后根据需要选择 Linux 映像。以下示例使用 Ubuntu 14.04 映像。
![显示“新建”按钮的门户屏幕截图][3]

2. 对于“主机名”，请指定你与 Internet 客户端用来访问此虚拟机的 URL 的名称。定义 DNS 名称的最后一个部分（例如 tomcatdemo）。然后，Azure 将生成 tomcatdemo.chinacloudapp.cn 作为 URL。

3. 对于“SSH 身份验证密钥”，请从 publicKey.pem 文件中复制密钥值，其中包含由 PuTTYgen 生成的公钥。
![门户中的“SSH 身份验证密钥”框][4]

4. 根据需要配置其他设置，然后单击**“创建”**。

## 阶段 2：准备用于 Tomcat7 的虚拟机
在此阶段，我们将为 Tomcat 流量配置终结点，然后连接到新的虚拟机。

### 步骤 1：打开 HTTP 端口，以允许 Web 访问
Azure 中的终结点由 TCP 或 UDP 协议以及公用和专用端口组成。专用端口是服务侦听虚拟机的端口。公用端口是 Azure 云服务从外部侦听基于 Internet 的传入流量的端口。

TCP 端口 8080 是 Tomcat 用来侦听的默认端口号。如果使用 Azure 终结点打开此端口，则你与其他 Internet 客户端可以访问 Tomcat 页。

1. 在门户中，单击“浏览”\>“虚拟机”，然后单击创建的虚拟机。
![虚拟机目录的屏幕截图][5]
2. 若要将终结点添加到虚拟机，单击“终结点”框。
![显示“终结点”框的屏幕截图][6]
3. 单击**“添加”**。

    1. 对于终结点，请在“终结点”中输入终结点的名称，然后在“公用端口”中输入 80。

      如果将其设置为 80，则无需在 URL 中包括用于访问 Tomcat 的端口号。例如，http://tomcatdemo.chinacloudapp.cn。

      如果将其设置为其他值（例如 81），则需要将端口号添加到 URL 才能访问 Tomcat。例如，http://tomcatdemo.chinacloudapp.cn:81/。
    2. 在“专用端口”中输入 8080。默认情况下，Tomcat 侦听 TCP 端口 8080。如果更改了 Tomcat 的默认侦听端口，应将“专用端口”更新为与 Tomcat 侦听端口相同。
    ![显示“添加”命令、“公共端口”和“专用端口”的 UI 屏幕截图][7]
4. 单击“确定”将该终结点添加到虚拟机。

### 步骤 2：连接到你创建的映像
可以选择用于连接到虚拟机的任何 SSH 工具。本示例使用 PuTTY。

1. 从门户获取虚拟机的 DNS 名称。
    1. 单击“浏览”\>“虚拟机”。
    2. 选择虚拟机的名称，然后单击“属性”。
    3. 在“属性”磁贴中，查看“域名”框中的 DNS 名称。

2. 从“SSH”框中获取 SSH 连接的端口号。
![显示 SSH 连接端口号的屏幕截图][8]

3. 下载 [PuTTY](http://www.putty.org/)。

4. 下载后，单击可执行文件 Putty.exe。在 PuTTY 配置中，使用从虚拟机的属性获取的主机名和端口号配置基本选项。
![显示 PuTTY 配置主机名和端口选项的屏幕截图][9]

5. 在左窗格中，单击“连接”\>“SSH”\>“身份验证”，然后单击“浏览”指定 privateKey.ppk 文件的位置。privateKey.ppk 文件包含 PuTTYgen 在本文“阶段 1：创建映像”部分中生成的私钥。
![显示“连接”目录层次结构和“浏览”按钮的屏幕截图][10]

6. 单击“打开”。可能会出现一个消息提示框。如果已正确配置 DNS 名称和端口号，单击“是”。
![显示通知的屏幕截图][11]

7. 系统会提示输入用户名。
![显示用户名输入位置的屏幕截图][12]

8. 输入在本文“阶段 1：创建映像”部分中创建虚拟机时使用的用户名。将显示如下内容：
![显示身份验证确认的屏幕截图][13]

## 阶段 3：安装软件
在此阶段，我们将安装 Java 运行时环境、Tomcat7 和其他 Tomcat7 组件。

### Java 运行时环境
Tomcat 用 Java 编写。有两种类型的 Java 开发工具包 \(JDK\)：OpenJDK 和 Oracle JDK。可以选择所需的工具包。

> [AZURE.NOTE]
这两个 JDK 对于 Java API 中的类，几乎包含相同的代码，但用于虚拟机的代码不同。OpenJDK 倾向于使用开放库，而 Oracle JDK 倾向于使用封闭库。Oracle JDK 包含更多类并且修复了一些 bug，比 OpenJDK 更稳定。

#### 安装 OpenJDK  

使用以下命令下载 OpenJDK。

    sudo apt-get update  
    sudo apt-get install openjdk-7-jre  

* 若要创建包含 JDK 文件的目录，请执行以下命令：

        sudo mkdir /usr/lib/jvm  
* 若要将 JDK 文件解压到 /usr/lib/jvm/ 目录，请执行以下命令：

        sudo tar -zxf jdk-8u5-linux-x64.tar.gz  -C /usr/lib/jvm/

#### 安装 Oracle JDK

使用以下命令从 Oracle 网站下载 Oracle JDK。

     wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u5-b13/jdk-8u5-linux-x64.tar.gz  
* 若要创建包含 JDK 文件的目录，请执行以下命令：

        sudo mkdir /usr/lib/jvm  
* 若要将 JDK 文件解压到 /usr/lib/jvm/ 目录，请执行以下命令：

        sudo tar -zxf jdk-8u5-linux-x64.tar.gz  -C /usr/lib/jvm/  
* 将 Oracle JDK 设置为默认 Java 虚拟机：

        sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_05/bin/java 100  

        sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_05/bin/javac 100  

#### 确认 Java 安装成功
可以使用如下命令测试是否已正确安装 Java 运行时环境：

    java -version  

如果已安装 OpenJDK，应会看到如下消息：
![指出成功安装 OpenJDK 的消息][14]

如果已安装 Oracle JDK，应会看到如下消息：
![指出成功安装 Oracle JDK 的消息][15]

### 安装 Tomcat7
使用以下命令安装 Tomcat7。

    sudo apt-get install tomcat7  

如果未使用 Tomcat7，请使用此命令的相应变体。

#### 确认 Tomcat7 安装成功
若要检查 Tomcat7 是否已成功安装，请浏览到 Tomcat 服务器的 DNS 名称。在本文中，示例 URL 为 http://tomcatexample.chinacloudapp.cn/。如果看到如下消息，则表示已正确安装 Tomcat7。
![指出成功安装 Tomcat7 的消息][16]

### 安装其他 Tomcat7 组件
可以安装其他可选 Tomcat 组件。

使用 **sudo apt-cache search tomcat7** 命令可查看所有可用组件。使用以下命令安装一些有用的组件。

    sudo apt-get install tomcat7-admin      #admin web applications

    sudo apt-get install tomcat7-user         #tools to create user instances  

## 阶段 4：配置 Tomcat7
在此阶段，可以管理 Tomcat。

### 启动和停止 Tomcat7
安装 Tomcat7 服务器时，该服务器会自动启动。也可以使用以下命令将它启动：

    sudo /etc/init.d/tomcat7 start

停止 Tomcat7：

    sudo /etc/init.d/tomcat7 stop

查看 Tomcat7 的状态：

    sudo /etc/init.d/tomcat7 status

重新启动 Tomcat 服务：

    sudo /etc/init.d/tomcat7 restart

### Tomcat7 管理
可以通过编辑 Tomcat 用户配置文件来设置管理员凭据。请使用以下命令：

    sudo vi  /etc/tomcat7/tomcat-users.xml   

以下是示例：
![显示 sudo vi 命令输出的屏幕截图][17]

> [AZURE.NOTE]
为管理员用户名创建强密码。

编辑此文件之后，应使用以下命令重新启动 Tomcat7 服务，确保所做的更改生效：

    sudo /etc/init.d/tomcat7 restart  

打开浏览器，然后输入 URL **http://\<Tomcat 服务器 DNS 名称\>/manager/html**。对于本文中的示例，URL 为 http://tomcatexample.chinacloudapp.cn/manager/html。

连接后，应显示如下内容：
![Tomcat Web 应用程序管理器的屏幕截图][18]

## 常见问题
### 无法使用 Tomcat 和 Moodle 通过 Internet 访问虚拟机
#### 症状  
  Tomcat 正在运行，但使用浏览器看不到 Tomcat 默认页。
#### 可能的根本原因   

  * Tomcat 侦听端口与用于 Tomcat 流量的虚拟机终结点的专用端口不同。

     检查公用端口和专用端口终结点设置，确保专用端口与 Tomcat 侦听端口相同。有关如何为虚拟机配置终结点的说明，请参阅本文的“阶段 1：创建映像”部分。

     若要确定 Tomcat 侦听端口，请打开 /etc/httpd/conf/httpd.conf（Red Hat 发行版）或 /etc/tomcat7/server.xml（Debian 发行版）。默认情况下，Tomcat 侦听端口为 8080。下面是一个示例：

        <Connector port="8080" protocol="HTTP/1.1"  connectionTimeout="20000"   URIEncoding="UTF-8"            redirectPort="8443" />  

     如果要使用 Debian 或 Ubuntu 等虚拟机并且要更改 Tomcat 侦听的默认端口（例如 8081），则还应为操作系统打开该端口。首先打开配置文件：

        sudo vi /etc/default/tomcat7  

     然后，取消注释最后一行并将“no”更改为“yes”。

        AUTHBIND=yes
  2. 防火墙已禁用 Tomcat 的侦听端口。

     只能在本地主机上看到 Tomcat 默认页。问题的原因很可能是 Tomcat 侦听的端口被防火墙阻止。可以使用 w3m 工具来浏览网页。以下命令安装 w3m 并浏览到 Tomcat 默认页面：

        sudo yum  install w3m w3m-img

        w3m http://localhost:8080  
#### 解决方案

* 如果 Tomcat 侦听端口与发往虚拟机的流量的终结点专用端口不同，则需要将该专用端口更改为与 Tomcat 侦听端口相同。
* 如果此问题由防火墙/iptables 导致，请将以下行添加到 /etc/sysconfig/iptables。只有 https 流量才需要第二行：

        -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT

        -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT  

    > [AZURE.IMPORTANT]
    确保将上述行放置在全局限制访问权限的所有行上面，如下所示：-A INPUT -j REJECT --reject-with icmp-host-prohibited


若要重新加载 iptables，请运行以下命令：

    service iptables restart

这样就已经在 CentOS 6.3 上进行了测试。

### 将项目文件上载到 /var/lib/tomcat7/webapps/ 时，权限被拒绝
#### 症状
  使用 SFTP 客户端（例如 FileZilla）连接到虚拟机并导航到 /var/lib/tomcat7/webapps/ 来发布站点时，收到如下错误消息：

     status:    Listing directory /var/lib/tomcat7/webapps
     Command:    put "C:\Users\liang\Desktop\info.jsp" "info.jsp"
     Error:    /var/lib/tomcat7/webapps/info.jsp: open for write: permission denied
     Error:    File transfer failed
#### 可能的根本原因
你无权访问 /var/lib/tomcat7/webapps 文件夹。
#### 解决方案  
  需要获取 root 帐户权限。可将该文件夹的所有权从 root 更改为在设置计算机时使用的用户名。以下是使用 azureuser 帐户名称的示例：

     sudo chown azureuser -R /var/lib/tomcat7/webapps

  也可以使用 -R 选项将权限应用到目录中的所有文件。

  此命令也适用于目录。-R 选项可更改目录内的所有文件和目录的权限。下面是一个示例：

     sudo chown -R username:group directory  

  此命令将更改该目录中所有文件和目录的所有权（用户和组）。

  以下命令只会更改文件夹目录的权限。该目录中的文件和文件夹的权限不会更改。

     sudo chown username:group directory

[1]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-01.png
[2]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-02.png
[3]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-03.png
[4]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-04.png
[5]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-05.png
[6]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-06.png
[7]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-07.png
[8]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-08.png
[9]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-09.png
[10]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-10.png
[11]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-11.png
[12]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-12.png
[13]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-13.png
[14]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-14.png
[15]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-15.png
[16]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-16.png
[17]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-17.png
[18]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-18.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording update-->