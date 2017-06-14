<properties
    pageTitle="在经典 Azure VM 上运行 Java 应用程序服务器 | Azure"
    description="本教程利用使用经典部署模型创建的资源，显示如何创建 Windows 虚拟机并将其配置为运行 Apache Tomcat 应用程序服务器。"
    services="virtual-machines-windows"
    documentationcenter="java"
    author="rmcmurray"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="d627aa09-f7d6-4239-8110-f8fc5111b939"
    ms.service="virtual-machines-windows"
    ms.workload="web"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="Java"
    ms.topic="article"
    ms.date="03/16/2017"
    wacn.date="04/24/2017"
    ms.author="robmcm"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="18c30e40efa8a048220da4feba874d240b4ffbeb"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-run-a-java-application-server-on-a-virtual-machine-created-with-the-classic-deployment-model"></a>如何在使用经典部署模型创建的虚拟机上运行 Java 应用程序服务器
> [AZURE.IMPORTANT]
> Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 和经典模型](/documentation/articles/resource-manager-deployment-model/)。 本文介绍如何使用经典部署模型。 Azure 建议大多数新部署使用 Resource Manager 模型。 有关使用 Java 8 和 Tomcat 部署 webapp 的 Resource Manager 模板，请参阅[此处](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-java-tomcat/)。

通过 Azure，可使用虚拟机提供服务器功能。 例如，在 Azure 上运行的虚拟机可配置为托管 Java 应用程序服务器，如 Apache Tomcat。

完成本指南后，你将了解如何创建在 Azure 上运行的虚拟机并将其配置为运行 Java 应用程序服务器。 你将了解并执行以下任务：

* 如何创建已安装 Java 开发工具包 (JDK) 的虚拟机。
* 如何远程登录到虚拟机。
* 如何在虚拟机上安装 Java 应用程序服务器 (Apache Tomcat)。
* 如何为虚拟机创建终结点。
* 如何在防火墙中为应用程序服务器开放一个端口。

完成安装后，Tomcat 会在虚拟机上运行。

![运行 Apache Tomcat 的虚拟机][virtual_machine_tomcat]

[AZURE.INCLUDE [create-account-and-vms-note](../../includes/create-account-and-vms-note.md)]

## <a name="to-create-a-virtual-machine"></a>创建虚拟机
1. 登录 [Azure 门户](https://portal.azure.cn)。  
2. 依次单击“新建”和“计算”，然后在“特色应用”中单击“全部查看”。
3. 单击“JDK”，然后在“JDK”窗格中单击“JDK 8”。  
   如果安装的是尚不能在 JDK 8 中运行的旧版应用程序，可选择支持 **JDK 6** 和 **JDK 7** 的虚拟机映像。
4. 在 JDK 8 窗格中，选择“经典”，然后单击“创建”。
5. 在“基本信息”边栏选项卡中：
    1. 指定虚拟机的名称。
    2. 在“用户名”字段中输入管理员的姓名。 请记住此姓名以及下一字段中的相关密码。 远程登录虚拟机时需使用这些项。
    3. 在“新密码”字段中输入密码，然后在“确认密码”字段中再次输入密码。 这是管理员帐户的密码。
    4. 选择适当的**订阅**。
    5. 对于“资源组”，请单击“新建”并输入的新资源组的名称。 或者单击“使用现有项”并选择某个可用资源组。
    6. 选择虚拟机所在的位置，例如“中国东部”。
6. 单击“下一步” 。
7. 在“虚拟机映像大小”边栏选项卡中，选择“A1 标准”或其他合适映像。
8. 单击“选择”。

9. 在“**设置**”边栏选项卡中，单击“**确定**”。 可使用 Azure 提供的默认值。  
10. 在“**摘要**”边栏选项卡中，单击“**确定**”。

## <a name="to-remotely-sign-in-to-your-virtual-machine"></a>远程登录到虚拟机的步骤
1. 登录到 [Azure 门户](https://portal.azure.cn)。
2. 单击“虚拟机(经典)”。 必要时，请单击服务类别下方左下角的“更多服务”。 “虚拟机(经典)”条目列在“计算”组中。
3. 单击要登录到的虚拟机的名称。
4. 启动虚拟机后，可使用窗格顶部的菜单进行连接。
5. 单击“连接”。
6. 根据需要响应提示以连接到虚拟机。 通常，将保存或打开含有连接详细信息的 .rdp 文件。 可能需要复制 url:port（.rdp 文件第一行的最后部分）并将其粘贴到远程登录应用程序中。

## <a name="to-install-a-java-application-server-on-your-virtual-machine"></a>在虚拟机上安装 Java 应用程序服务器
可以将 Java 应用程序服务器安装到虚拟机，也可以通过安装程序安装 Java 应用程序服务器。

本教程将 Tomcat 用作 Java 应用程序服务器进行安装。

1. 登录到虚拟机以后，请将浏览器会话打开到 [Apache Tomcat](http://tomcat.apache.org/download-80.cgi)。
2. 双击“32 位/64 位 Windows Service 安装程序”的链接。 通过此技巧，Tomcat 将作为 Windows 服务进行安装。
3. 系统提示时，请选择运行该安装程序。
4. 在“Apache Tomcat 安装程序”  向导中，按照提示操作来安装 Tomcat。 在本教程中，接受默认值即可。 当显示“完成 Apache Tomcat 安装程序向导”对话框时，可以选择“运行 Apache Tomcat”以立即启动 Tomcat。 单击“完成”以完成 Tomcat 安装过程。

## <a name="to-start-tomcat"></a>启动 Tomcat

可在虚拟机上打开命令提示符并运行命令 **net&nbsp;start&nbsp;Tomcat8** 来手动启动 Tomcat。

Tomcat 运行后，可通过在虚拟机浏览器中输入 URL <http://localhost:8080> 来访问 Tomcat。

若要从外部计算机查看 Tomcat 的运行，则需要创建一个终结点并开放一个端口。

## <a name="to-create-an-endpoint-for-your-virtual-machine"></a>为虚拟机创建终结点
1. 登录 [Azure 门户](https://portal.azure.cn)。
2. 单击“虚拟机(经典)”。
3. 单击正在运行 Java 应用程序服务器的虚拟机的名称。
4. 单击“终结点” 。
5. 单击 **“添加”**。
6. 在“添加终结点”对话框中：
    1. 为终结点指定名称；例如，“HttpIn”。
    2. 为协议选择 **TCP**。
    3. 指定“80”作为公用端口。
    4. 指定“8080”作为专用端口。
    5. 为浮动 IP 地址选择“禁用”。
    6. 将访问控制列表保持原样。
    7. 单击“确定”按钮，以关闭对话框并创建终结点。

## <a name="to-open-a-port-in-the-firewall-for-your-virtual-machine"></a>在防火墙上为虚拟机开放一个端口
1. 登录到虚拟机。
2. 单击 Windows 的“开始” 。
3. 单击“控制面板”。
4. 依次单击“系统和安全性”、“Windows 防火墙”和“高级设置”。
5. 单击“入站规则”，然后单击“新建规则”。  
   ![新建入站规则][NewIBRule]
6. 对于“规则类型”，请选择“端口”，然后单击“下一步”。  
   ![新建入站规则端口][NewRulePort]
7. 在“协议和端口”屏幕上，选择“TCP”，指定“8080”作为“特定本地端口”，然后单击“下一步”。  
   ![新建入站规则][NewRuleProtocol]
8. 在“操作”屏幕上，选择“允许连接”，然后单击“下一步”。
   ![新建入站规则操作][NewRuleAction]
9. 在“配置文件”屏幕上，确保选中“域”、“专用”和“公共”，然后单击“下一步”。
   ![新建入站规则配置文件][NewRuleProfile]
10. 在“名称”屏幕上，指定规则的名称，如“HttpIn”（但是，规则名称无需与终结点名称匹配），然后单击“完成”。  
    ![新建入站规则名称][NewRuleName]

此时，应可从外部浏览器查看 Tomcat 网站。 在浏览器地址窗口中，以 **http://*your\_DNS\_name*.chinacloudapp.cn** 格式键入 URL，其中 ***your\_DNS\_name*** 是创建虚拟机时指定的 DNS 名称。

## <a name="application-lifecycle-considerations"></a>应用程序生命周期注意事项
* 可以创建自己的 Web 应用程序存档 (WAR) 并将其添加到 **webapps** 文件夹。 例如，创建基本的 Java 服务页 (JSP) 动态 Web 项目并将其作为 WAR 文件导出。 然后，将 WAR 复制到虚拟机上的 Apache Tomcat **webapps** 文件夹中，然后在浏览器中运行它。
* 默认情况下，Tomcat 服务在安装后会设置为手动启动。 可以使用“服务”管理单元将其切换为自动启动。 可通过依次单击 Windows 的“开始”、“管理工具”、“服务”来启动“服务”管理单元。 双击“Apache Tomcat”服务，并将“启动类型”设置为“自动”。

    ![将服务设置为自动启动][service_automatic_startup]

    让 Tomcat 自动启动的好处是，在虚拟机重启时（例如在安装需重启的软件更新后），Tomcat 即开始运行。

## <a name="next-steps"></a>后续步骤
可了解可能要在 Java 应用程序中包含的其他服务（例如 Azure 存储、服务总线和 SQL 数据库）。 查看 [Java 开发人员中心](/develop/java/)上提供的信息。

[virtual_machine_tomcat]: ./media/virtual-machines-windows-classic-java-run-tomcat-app-server/WA_VirtualMachineRunningApacheTomcat.png

[service_automatic_startup]: ./media/virtual-machines-windows-classic-java-run-tomcat-app-server/WA_TomcatServiceAutomaticStart.png

[NewIBRule]: ./media/virtual-machines-windows-classic-java-run-tomcat-app-server/NewInboundRule.png
[NewRulePort]: ./media/virtual-machines-windows-classic-java-run-tomcat-app-server/NewRulePort.png
[NewRuleProtocol]: ./media/virtual-machines-windows-classic-java-run-tomcat-app-server/NewRuleProtocol.png
[NewRuleAction]: ./media/virtual-machines-windows-classic-java-run-tomcat-app-server/NewRuleAction.png
[NewRuleName]: ./media/virtual-machines-windows-classic-java-run-tomcat-app-server/NewRuleName.png
[NewRuleProfile]: ./media/virtual-machines-windows-classic-java-run-tomcat-app-server/NewRuleProfile.png
<!--Update_Description: change to the new portal-->