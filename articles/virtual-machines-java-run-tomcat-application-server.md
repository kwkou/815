<properties linkid="dev-java-vm-application-server" urlDisplayName="Tomcat on Virtual Machine" pageTitle="Tomcat on a virtual machine - Azure tutorial" metaKeywords="Azure vm, creating vm Tomcat, configuring vm Tomcat" description="Learn how to create a Windows Virtual machine and configure the machine to run a Apache Tomcat application server." metaCanonical="" services="virtual-machines" documentationCenter="Java" title="How to run a Java application server on a virtual machine" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" scriptId="" videoId="" />

# 如何在虚拟机上运行 Java 应用程序服务器

通过 Azure，你可使用虚拟机提供服务器功能。例如，在 Azure 上运行的虚拟机可配置为托管 Java 应用程序服务器，如 Apache Tomcat。完成本指南之后，你将会了解如何创建在 Azure 上运行的虚拟机并将其配置为运行 Java 应用程序服务器。

你将了解到以下内容：

-   如何创建已安装 JDK 的虚拟机。
-   如何远程登录到虚拟机。
-   如何在虚拟机上安装 Java 应用程序服务器。
-   如何为虚拟机创建终结点。
-   如何在防火墙中为应用程序服务器开放一个端口。

在本教程中，将在虚拟机上安装 Apache Tomcat 应用程序服务器。安装完成后将生成一个 Tomcat 安装，如下所示。

![运行 Apache Tomcat 的虚拟机][运行 Apache Tomcat 的虚拟机]

[WACOM.INCLUDE [create-account-and-vms-note][create-account-and-vms-note]]

## 创建虚拟机

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  依次单击“新建”、“计算”、“虚拟机”和“从库中”。
3.  在“虚拟机映像选择”对话框中，选择 **JDK 7 Windows Server 2012**。
    请注意，万一你安装的是还不能在 JDK 7 中运行的旧应用程序，可选择 **JDK 6 Windows Server 2012**。
4.  单击“下一步”。
5.  在“虚拟机配置”对话框中：

    1.  指定虚拟机的名称。
    2.  指定要用于虚拟机的大小。
    3.  在“用户名”字段中输入管理员的名称。记住你下次要输入的此名称和密码，远程登录虚拟机时你将使用它们。
    4.  在“新密码”字段中输入密码，然后在“确认”字段中重新输入一次。这是 Administrator 帐户密码。
    5.  单击“下一步”。

6.  在下一个“虚拟机配置”对话框中：

    1.  对于“云服务”，使用默认的“创建新云服务”。
    2.  “云服务 DNS 名称”的值在 cloudapp.net 中必须是唯一的。如有必要，请修改此值，直至 Azure 指出它是唯一的值。
    3.  指定区域、地缘组或虚拟网络。在本教程中，请指定区域，如“美国西部”。
    4.  对于“存储帐户”框，选择“使用自动生成的存储帐户”。
    5.  对于“可用性集”，请选择“(无)”。
    6.  单击“下一步”。

7.  在最后一个“虚拟机配置”对话框中：

    1.  接受默认的终结点项。
    2.  单击“完成”。

## 远程登录到虚拟机

1.  登录到[“管理门户”][Azure 管理门户]。
2.  单击“虚拟机”。
3.  单击你要登录的虚拟机名称。
4.  单击“连接”。
5.  根据需要响应提示以连接到虚拟机。提示需要管理员名称和密码时，请使用你创建虚拟机时提供的值。

## 在虚拟机上安装 Java 应用程序服务器

你可将 Java 应用程序服务器安装到虚拟机，也可以通过安装程序安装 Java 应用程序服务器。

在本教程中，将安装 Tomcat。

1.  登录到虚拟机后，将浏览器会话打开到 <http://tomcat.apache.org/download-70.cgi>。
2.  双击“32 位/64 位 Windows Service 安装程序”的链接。利用此方法，Tomcat 将作为 Windows 服务安装。
3.  系统提示时，请选择运行该安装程序。
4.  在“Apache Tomcat 安装程序”向导中，按照提示操作来安装 Tomcat。在本教程中，接受默认值即可。当显示“完成 Apache Tomcat 安装程序向导”对话框时，可以选择“运行 Apache Tomcat”以立即启动 Tomcat。单击“完成”以完成 Tomcat 安装过程。

## 启动 Tomcat

如果你未在“完成 Apache Tomcat 安装程序向导”对话框中选择运行 Tomcat，请通过在虚拟机上打开命令提示符并运行 **net start Tomcat7** 来启动它。

如果你运行虚拟机的浏览器并打开 <http://localhost:8080>，则应立即看到 Tomcat 在运行。

若要从外部计算机查看 Tomcat 的运行，则需要创建一个终结点并开放一个端口。

## 为虚拟机创建终结点

1.  登录到[“管理门户”][Azure 管理门户]。
2.  单击“虚拟机”。
3.  单击正在运行 Java 应用程序服务器的虚拟机的名称。
4.  单击“终结点”。
5.  单击“添加”。
6.  在“添加终结点”对话框中，确保选中“添加独立终结点”，然后单击“下一步”按钮。
7.  在“新建终结点详细信息”对话框中：

    1.  为终结点指定名称；例如，**HttpIn**。
    2.  指定 **TCP** 作为协议。
    3.  指定 **80** 作为公用端口。
    4.  指定 **8080** 作为私有端口。
    5.  单击“完成”按钮以关闭对话框。将立即为你创建终结点。

## 在防火墙上为虚拟机开放一个端口

1.  登录虚拟机。
2.  单击 Windows 的“开始”。
3.  单击“控制面板”。
4.  依次单击“系统和安全性”、“Windows 防火墙”和“高级设置”。
5.  单击“入站规则”，然后单击“新建规则”。

![新建入站规则][新建入站规则]

1.  对于新规则，请选择“端口”作为“规则类型”，然后单击“下一步”。

![新建入站规则端口][新建入站规则端口]

1.  选择“TCP”作为协议并指定“8080”作为端口，然后单击“下一步”。

![新建入站规则][1]

1.  选择“允许连接”，然后单击“下一步”。

![新建入站规则操作][新建入站规则操作]

1.  确保为配置文件选中“域”、“私有”和“公开”，然后单击“下一步”。

![新建入站规则配置文件][新建入站规则配置文件]

1.  指定规则的名称，如 **HttpIn**（但是，规则名称无需与终结点名称匹配），然后单击“完成”。

![新建入站规则名称][新建入站规则名称]

此时，应可从外部浏览器使用 **http://*your\_DNS\_name*.cloudapp.net** 格式的 URL 立即查看你的 Tomcat 网站，其中 **your\_DNS\_name** 是你创建虚拟机时指定的 DNS 名称。

## 应用程序生命周期注意事项

-   你可创建你自己的应用程序 Web 存档 (WAR) 并将其添加到 **webapps** 文件夹。例如，创建一个基本的 Java Service Page (JSP) 动态 Web 项目并将其导出为 WAR 文件，将此 WAR 复制到虚拟机上的 Apache Tomcat **webapps** 文件夹，然后在浏览器中运行它。
-   默认情况下，Tomcat 服务在安装后将会设置为手动启动。你可以使用“服务”管理单元将它切换为自动启动。请依次单击 Windows 的“开始”、“管理工具”和“服务”以启动“服务”管理单元。若要将 Tomcat 设置为自动启动，请在“服务”管理单元中双击“Apache Tomcat”服务，并将“启动类型”设置为“自动”，如下图所示。

    ![将服务设置为自动启动][将服务设置为自动启动]

    让 Tomcat 自动启动的好处是，当虚拟机重新启动时（例如，在安装需要重新启动的软件更新后），Tomcat 将再次启动。

## 后续步骤

-   通过查看 <http://www.windowsazure.com/zh-cn/develop/java/> 上提供的信息，了解要与 Java 应用程序一起包含的 Azure 存储、Service Bus、SQL Database 等其他服务。

  [运行 Apache Tomcat 的虚拟机]: ./media/virtual-machines-java-run-tomcat-application-server/WA_VirtualMachineRunningApacheTomcat.png
  [create-account-and-vms-note]: ../includes/create-account-and-vms-note.md
  [Azure 管理门户]: https://manage.windowsazure.cn
  [新建入站规则]: ./media/virtual-machines-java-run-tomcat-application-server/NewInboundRule.png
  [新建入站规则端口]: ./media/virtual-machines-java-run-tomcat-application-server/NewRulePort.png
  [1]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleProtocol.png
  [新建入站规则操作]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleAction.png
  [新建入站规则配置文件]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleProfile.png
  [新建入站规则名称]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleName.png
  [http://\*your\\\_DNS\\\_name]: http://*your\_DNS\_name
  [将服务设置为自动启动]: ./media/virtual-machines-java-run-tomcat-application-server/WA_TomcatServiceAutomaticStart.png
