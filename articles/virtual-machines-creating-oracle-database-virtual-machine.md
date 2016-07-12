<properties
	pageTitle="使用 Azure 门户预览创建 Oracle Database VM | Azure"
	description="了解如何使用经典部署模型和 Azure 门户预览创建虚拟机，并在虚拟机中创建 Oracle Database。"
	services="virtual-machines-windows"
	authors="rickstercdn"
	manager="timlt"
	documentationCenter=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/17/2016"
	wacn.date="07/11/2016"/>

#在 Azure 中创建 Oracle Database 虚拟机

[AZURE.INCLUDE [virtual-machines-common-oracle-support](../includes/virtual-machines-common-oracle-support.md)]

以下示例演示如何在以前创建并安装了 Oracle Database 版本 12c 的 Windows Server VM 上创建 Oracle Database。执行以下两个步骤。首先，连接 VM，然后在 VM 中创建 Oracle Database。所示的示例采用 Oracle Database 版本 12c，但在 11g 中的步骤几乎相同。


##在 Azure 中使用 Oracle Database VM 创建数据库

1.	登录到 [Azure 门户预览](https://portal.azure.cn/)。

2.	单击**“虚拟机（经典）”**。

3.	单击你要登录的 VM 的名称。

4.	单击“连接”。

5.	根据需要响应提示以连接到 VM。提示需要管理员名称和密码时，请使用你创建 VM 时提供的值。

6. 下载并安装 [Oracle Database 12c](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/database12c-win64-download-2297732.html)。

6.	创建名为 **ORACLE\_HOSTNAME** 的环境变量，其值设置为 VM 的计算机名称。可以使用以下步骤创建环境变量：

	a.在 Windows 中，单击“开始”，键入“控制面板”，然后依次单击“控制面板”图标、“系统和安全性”、“系统”、“高级系统设置”。

	b.单击“高级”选项卡，然后单击“环境变量”。

	c.在“系统变量”部分下，单击“新建”以创建变量。

	d.在“新建系统变量”对话框中，输入 **ORACLE\_HOSTNAME** 作为变量名称，然后输入 VM 的计算机名称作为值。若要确定计算机名称，请打开命令提示符并运行 **SET COMPUTERNAME**（该命令的输出将包含计算机名称）。

	e.单击“确定”保存新环境变量并关闭“新建系统变量”对话框。

	f.关闭控制面板打开的其他对话框。

7.	在 Windows 中，单击“开始”，然后键入“数据库配置助手”。单击“数据库配置助手”图标。

8.	在“数据库配置助手”向导中，根据需要为每个对话框步骤提供值：

	a.**步骤 1：**单击“创建数据库”，然后单击“下一步”。

	![](./media/virtual-machines-windows-classic-create-oracle-database/image5.png)

	b.**步骤 2：**输入“全局数据库名称”的值。输入“管理密码”的值并确认。此密码供 Oracle 数据库的 **SYSTEM** 用户使用。清除“创建为容器数据库”。单击**“下一步”**。

	![](./media/virtual-machines-windows-classic-create-oracle-database/image6.png)

	c.**步骤 3：**此时将自动进行必备组件检查，然后转到**步骤 4**。

	d.**步骤 4：**查看“创建数据库 - 摘要”选项，然后单击“完成”。

	![](./media/virtual-machines-windows-classic-create-oracle-database/image7.png)

	e.**步骤 5：**“进度页”将报告数据库创建状态。

	![](./media/virtual-machines-windows-classic-create-oracle-database/image8.png)

	f.创建数据库后，你可以选择使用“密码管理”对话框。根据需要修改密码设置，然后关闭对话框以退出“数据库配置助手”向导。

##确认已安装数据库

1.	仍然登录到 VM，启动 SQL Plus 命令提示符。在 Windows 中，单击“开始”，然后键入“SQL Plus”。单击 **SQL Plus** 图标。

2.	出现提示时，使用用户名 **SYSTEM** 以及你在创建 Oracle 数据库时指定的密码登录。

3.	在 SQL Plus 命令提示符下运行以下命令。

		**select * from GLOBAL\_NAME;**

	结果应是你创建的数据库的全局名称。

	![](./media/virtual-machines-windows-classic-create-oracle-database/image9.png)

##允许远程访问数据库
若要允许远程访问你的数据库（例如，从运行 Java 代码的客户端计算机进行远程访问），你需要启动数据库侦听器，在虚拟机防火墙中打开端口 1521，并为端口 1521 创建一个公共终结点。

### 启动数据库侦听器
1.	登录到 VM。

2.	打开命令提示符。

3.	在命令提示符下运行以下命令。

		**lsnrctl start**

> [AZURE.NOTE] 可以运行 **lsnrctl status** 以检查侦听器的状态。如果你想要停止侦听器，可以运行 **lsnrctl stop**。

### 在虚拟机的防火墙中打开端口 1521

1.	在保持登录到虚拟机的情况下，在 Windows 中单击“开始”，键入“高级安全 Windows 防火墙”，然后单击“高级安全 Windows 防火墙”图标。这将打开“高级安全 Windows 防火墙”管理控制台。

2.	在防火墙管理控制台中，单击左窗格中的“入站规则”（如果未看到“入站规则”，请展开左窗格中的顶级节点），然后单击右窗格中的“新建规则”。

3.	对于“规则类型”，请选择“端口”，然后单击“下一步”。

4.	对于“协议和端口”，请选择“TCP”，选择“特定本地端口”，为端口输入“1521”，然后单击“下一步”。

5.	选择“允许连接”，然后单击“下一步”。

6.	接受要应用该规则的配置文件的默认值，然后单击“下一步”。

7.	指定规则的名称，并选择性地输入说明，然后单击“完成”。

### 为端口 1521 创建公共终结点

1.	登录到 [Azure 门户预览](https://portal.azure.cn/)。

2.	单击“浏览”。

3.  单击**“虚拟机（经典）”**。

4.	选择虚拟机。

5.	单击“设置”。

6.	单击“终结点”。

7.	单击**“添加”**。

8.	指定终结点的名称：

	a.使用“TCP”作为协议。

	b.使用“1521”作为公用端口。

	c.使用 **1521** 作为专用端口。

9.	将其余的选项保留原样。

10. 单击**“确定”**。

##启用 Oracle Database Enterprise Manager 远程访问
如果你想要启用对 Oracle Database Enterprise Manager 的远程访问，请在防火墙中打开端口 5500，并在 Azure 门户预览中为端口 5500 创建一个虚拟机终结点（使用前述步骤打开端口 1521，并为端口 1521 创建终结点）。然后，打开一个浏览器并导航到采用 `http://<<unique_domain_name>>:5500/em` 格式的 URL，以从远程计算机运行 Oracle Enterprise Manager。

> [AZURE.NOTE] （可以通过以下方式确定 *<<unique\_domain\_name>>* 的值：在 [Azure 门户预览](https://portal.azure.cn/)中单击“虚拟机（经典）”，然后选择你要用于运行 Oracle Database 的虚拟机）。

##配置常用选项和高级选项套装
如果选择“包含常用选项的 Oracle Database”或“包含高级选项套装的 Oracle Database”，则下一步是在 Oracle 安装中配置附加功能。由于配置可能会根据你对每个组件的需求而有很大的不同，因此，请参阅 Oracle 文档，以获取有关在 Windows 上设置这些产品的说明。

**包含常用选项套装的 Oracle Database** 包括 Oracle Database Enterprise Edition，以及按许可证提供的 [Partitioning](http://www.oracle.com/us/products/database/options/partitioning/overview/index.html)、[Active Data Guard](http://www.oracle.com/us/products/database/options/active-data-guard/overview/index.html)、[Oracle Tuning Pack for Database](http://docs.oracle.com/html/A86647_01/tun_ovw.htm)、[Oracle Diagnostics Pack for Database](http://docs.oracle.com/cd/B28359_01/license.111/b28287/options.htm#CIHIHDDJ) 和 [Oracle Lifecycle Management Pack for Database](http://www.oracle.com/technetwork/oem/lifecycle-mgmt-495331.html) 实例。

**包含高级选项套装的 Oracle Database** 包括常用选项套装中所有选项的按许可证提供的实例，以及[高级压缩](http://www.oracle.com/us/products/database/options/advanced-compression/overview/index.html)、[高级安全性](http://www.oracle.com/us/products/database/options/advanced-security/overview/index.html)、[标签安全性](http://www.oracle.com/us/products/database/options/label-security/overview/index.html)、[数据库保管库](http://www.oracle.com/us/products/database/options/database-vault/overview/index.html)、[高级分析](http://www.oracle.com/us/products/database/options/advanced-analytics/overview/index.html)、[OLAP](http://docs.oracle.com/cd/E11882_01/license.112/e47877/options.htm#CIHGDEEF)、[Spatial and Graph](http://docs.oracle.com/cd/E11882_01/license.112/e47877/options.htm#CIHGDEEF)、[内存中数据库缓存](http://www.oracle.com/technetwork/products/timesten/overview/timesten-imdb-cache-101293.html)、[数据屏蔽套件](http://docs.oracle.com/cd/E11882_01/license.112/e47877/options.htm#CHDGEEBB)，以及 Oracle 测试数据管理套件（作为数据屏蔽套件的一部分）。

##其他资源
在设置虚拟机并创建数据库后，请参阅以下主题以了解更多信息。

-	[Oracle 虚拟机映像 - 其他注意事项](/documentation/articles/virtual-machines-windows-classic-oracle-considerations/)

-	[从 Java 应用程序连接到 Oracle 数据库](http://docs.oracle.com/cd/E11882_01/appdev.112/e12137/getconn.htm#TDPJD136)

-	[Oracle Database 2 Day DBA 12c 发行版 1](http://docs.oracle.com/cd/E16655_01/server.121/e17643/toc.htm)

<!---HONumber=Mooncake_0704_2016-->