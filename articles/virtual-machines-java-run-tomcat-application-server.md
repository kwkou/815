<properties linkid="dev-java-vm-application-server" urlDisplayName="Tomcat on Virtual Machine" pageTitle="虚拟机上的 Tomcat - Azure 教程" metaKeywords="Azure vm, creating vm Tomcat, configuring vm Tomcat" description="了解如何创建 Windows 虚拟机并将其配置为运行 Apache Tomcat 应用程序服务器。" metaCanonical="" services="virtual-machines" documentationCenter="Java" title="How to run a Java application server on a virtual machine" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" scriptId="" videoId="" />

# 如何在虚拟机上运行 Java 应用程序服务器

借助 Azure，您可以使用虚拟机提供服务器功能。例如，在 Azure 上运行的虚拟机可配置为托管 Java 应用程序服务器，例如 Apache Tomcat。完成本指南后，您将了解如何创建在 Azure 上运行的虚拟机并将其配置为运行 Java 应用程序服务器。

您将了解：

* 如何创建已安装 JDK 的虚拟机。
* 如何远程登录到虚拟机。
* 如何在虚拟机上安装 Java 应用程序服务器。
* 如何为虚拟机创建终结点。
* 如何在防火墙中为应用程序服务器开放一个端口。

在本教程中，将在虚拟机上安装 Apache Tomcat 应用程序服务器。安装完成后将生成一个 Tomcat 安装，如下所示。

![Virtual machine running Apache Tomcat][virtual_machine_tomcat]

[WACOM.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

## 创建虚拟机

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 依次单击"新建"****、"计算"****、"虚拟机"****和"从库中"****。
3. 在"虚拟机映像选择"****对话框中，选择"JDK 7 Windows Server 2012"****。
请注意，**JDK 6 Windows Server 2012** 在以下情况下可用：您有尚未准备好在 JDK 7 中运行的旧版应用程序。
4. 单击"下一步"****。
5. 在"虚拟机配置"<strong></strong>对话框中：
    1. 指定虚拟机的名称。
    2. 指定要用于虚拟机的大小。
    3. 在"用户名"****字段中为管理员输入一个名称。记住您下次要输入的此名称和密码，远程登录虚拟机时您将使用它们。
    4. 在"新密码"****字段中输入一个密码，然后在"确认"****字段中重新输入一次。这是 Administrator 帐户密码。
    5. 单击"下一步"****。
6. 在下一个"虚拟机配置"<strong></strong>对话框中：
    1. 对于"云服务"****，请使用默认的"创建新云服务"****。
    2. "云服务 DNS 名称"****的值在 cloudapp.net 上必须是唯一的。如果需要，请修改此值，以使 Azure 指示它是唯一的。
    2. 指定区域、地缘组或虚拟网络。在本教程中，请指定一个区域，如"美国西部"****。
    2. 对于"存储帐户"****，选择"使用自动生成的存储帐户"****。
    3. 对于"可用性集"****，请选择"(无)"****。
    4. 单击"下一步"****。
7. 在最后的"虚拟机配置"<strong></strong>对话框中：
    1. 接受默认的终结点项。
    2. 单击"完成"****。

## 远程登录到虚拟机

1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击"虚拟机"****。
3. 单击您要登录的虚拟机名称。
4. 当虚拟机启动后，页面底部的弹出菜单将允许连接。
5. 单击"连接"****。
6. 根据需要响应提示以连接到虚拟机。这应该需要保存或打开包含连接详细信息的 .rdp 文件。您可能需要复制 url:port 作为 .rdp 文件的第一行的最后一部分，并在远程登录应用程序中粘贴它。

## 在虚拟机上安装 Java 应用程序服务器

您可以将 Java 应用程序服务器复制到虚拟机，或者通过安装程序安装 Java 应用程序。 

在本教程中，将安装 Tomcat。

1. 登录到虚拟机后，将浏览器会话打开到 <http://tomcat.apache.org/download-70.cgi>。
2. 双击"32 位/64 位 Windows 服务安装程序"****的链接。使用此方法，Tomcat 将作为 Windows 服务安装。
3. 出现提示时，请选择运行安装程序。
4. 在"Apache Tomcat 安装"****向导中，请按照提示安装 Tomcat。在本教程中，接受默认值即可。当显示"完成 Apache Tomcat 安装向导"****对话框时，您可以选择选中"运行 Apache Tomcat"****以立即启动 Tomcat。单击"完成"****以完成 Tomcat 安装过程。

## 启动 Tomcat
如果您未在"完成 Apache Tomcat 安装向导"****对话框中选择运行 Tomcat，请通过在虚拟机上打开命令提示符并运行 **net start Tomcat7** 来启动它。

现在，如果您运行虚拟机的浏览器并打开 <http://localhost:8080>，您应该看到 Tomcat 在运行。

若要从外部计算机看到 Tomcat 运行，您将需要创建一个终结点并开放一个端口。

## 为虚拟机创建终结点
1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击"虚拟机"****。
3. 单击正在运行 Java 应用程序服务器的虚拟机的名称。
4. 单击"终结点"****。
5. 单击"添加"****。
6. 在"添加终结点"****对话框中，确保选中"添加独立终结点"****，然后单击"下一步"****。
7. 在<strong>"新终结点详细信息"</strong>对话框中：
    1. 为终结点指定一个名称：例如 **HttpIn**。
    2. 指定协议为 **TCP**。
    3. 指定公用端口为 **80**。
    4. 指定专用端口为 **8080**。
    5. 单击"完成"****按钮以关闭该对话框。将立即为您创建终结点。

## 在防火墙上为虚拟机开放一个端口
1. 登录虚拟机。
2. 单击"Windows 的'开始'屏幕"****。
3. 单击"控制面板"****。
4. 依次单击"系统和安全"****、"Windows 防火墙"****和"高级设置"****。
5. 单击"入站规则"****，然后单击"新规则"****。

 ![New inbound rule][NewIBRule]

6. 对于新规则，请为"规则类型"****选择"端口"****，然后单击"下一步"****。

 ![New inbound rule port][NewRulePort]

7. 选择"TCP"****作为协议，并指定"8080"****作为端口，然后单击"下一步"****。

 ![New inbound rule ][NewRuleProtocol]

8. 选择"允许连接"****，然后单击"下一步"****。

 ![New inbound rule action][NewRuleAction]

9. 确保为配置文件选中"域"****、"专用"****和"公用"****，然后单击"下一步"****。

 ![New inbound rule profile][NewRuleProfile]

10. 为规则指定一个名称，例如 **HttpIn**（但是，规则名称无需与终结点名称匹配），然后单击"完成"****。  

 ![New inbound rule name][NewRuleName]

此时，您的 Tomcat 网站现在应该可使用 **http://*your\_DNS\_name*.cloudapp.net** 形式的 URL 从外部浏览器查看，其中 ***your\_DNS\_name*** 是您在创建虚拟机时指定的 DNS 名称。

## 应用程序生命周期注意事项
* 您可以创建您自己的应用程序 Web 存档 (WAR) 并将其添加到 **webapps** 文件夹。例如，创建一个基本 Java Service Page (JSP) 动态 Web 项目并将其导出为 WAR 文件、将该 WAR 复制到虚拟机上的 Apache Tomcat **webapps** 文件夹，然后在浏览器中运行它。
* 默认情况下，安装 Tomcat 服务时，它将设置为手动启动。您可以使用服务管理单元将其切换为自动启动。通过依次单击"Windows 的'开始'屏幕"****、"管理工具"****和"服务"****启动服务管理单元。若要将 Tomcat 设置为自动启动，请在服务管理单元中双击"Apache Tomcat"****服务，然后将"启动类型"****设置为"自动"****，如下所示：

    ![Setting a service to start automatically][service_automatic_startup]

    使 Tomcat 自动启动的优势是，当虚拟机重新启动时（例如，在安装了需要重新启动的软件更新后），它将再次启动。

## 后续步骤
* 通过查看 <http://www.windowsazure.com/zh-cn/develop/java/> 中提供的信息，了解有关其他服务（例如 Azure 存储、服务总线、SQL Database 以及您希望包含在 Java 应用程序中的其他服务）的详细信息。

[virtual_machine_tomcat]: ./media/virtual-machines-java-run-tomcat-application-server/WA_VirtualMachineRunningApacheTomcat.png

[service_automatic_startup]: ./media/virtual-machines-java-run-tomcat-application-server/WA_TomcatServiceAutomaticStart.png









[NewIBRule]: ./media/virtual-machines-java-run-tomcat-application-server/NewInboundRule.png
[NewRulePort]: ./media/virtual-machines-java-run-tomcat-application-server/NewRulePort.png
[NewRuleProtocol]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleProtocol.png
[NewRuleAction]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleAction.png
[NewRuleName]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleName.png
[NewRuleProfile]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleProfile.png
