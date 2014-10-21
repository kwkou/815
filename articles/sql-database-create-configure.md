<properties linkid="manage-services-how-to-configure-a-sqldb" urlDisplayName="How to configure" pageTitle="How to configure a SQL Database - Azure" metaKeywords="Azure creating SQL Server, Azure configuring SQL Server" description="Learn how to create and configure a logical server using SQL Server in Azure." metaCanonical="" services="sql-database" documentationCenter="" title="How to Create and Configure SQL Database" authors="" solutions="" manager="" editor="" />

# <span id="configLogical"></span></a>如何创建和配置 SQL Database

在本主题中，你将逐步了解逻辑服务器的创建和配置过程。在新的 Azure（预览版）管理门户中，你可以先使用已修订的工作流创建数据库，然后再创建服务器。

但是在本主题中，你将首先创建服务器。如果你有要上载的现成 SQL Server 数据库，你可能更喜欢此方法。

## 目录

-   [如何：创建逻辑服务器][如何：创建逻辑服务器]
-   [如何：配置逻辑服务器的防火墙][如何：配置逻辑服务器的防火墙]

## <span id="createLogical"></span></a>如何：创建逻辑服务器

1.  登录到[管理门户][管理门户]。

2.  单击“SQL Database”，然后单击 SQL Database 主页上的“服务器”。

3.  单击页面底部的“添加”。

4.  在“服务器设置”中，输入一个没有空格的单词作为管理员名称。

    SQL Database 使用 SQL 身份验证进行加密连接。将使用你提供的名称创建一个分配给 sysadmin 固定服务器角色的新 SQL Server 身份验证登录名。

    该登录名不能是电子邮件地址、Windows 用户帐户或 Windows Live ID。SQL Database 不支持声明，也不支持 Windows 身份验证。

5.  提供由大小写值以及数字或符号共同组成的 8 个以上字符的强密码。

6.  选择区域。区域将确定服务器的地理位置。区域不能随意切换，因此要选择一个对此服务器有效的区域。选择一个最靠近你的位置。将 Azure 应用程序和数据库放置在同一区域可以降低出口带宽成本以及减少数据延迟情况。

7.  确保“允许服务”选项处于选中状态，以便你能够使用 SQL Database 管理门户、存储服务以及 Azure 上的其他服务连接到此数据库。

8.  完成后，请单击页面底部的复选标记。

请注意，你没有指定服务器名称。SQL Database 会自动生成服务器名称以确保没有重复的 DNS 条目。服务器名称是一个由 10 个字符组成的字母数字字符串。你不能更改 SQL Database 服务器的名称。

在下一步中，你将配置防火墙以便允许你网络上运行的应用程序通过建立连接来访问相关数据。

## <span id="configFWLogical"></span></a>如何：配置逻辑服务器的防火墙

1.  在[管理门户][管理门户]中，单击“SQL Database”，单击“服务器”，然后单击你刚才创建的服务器。

2.  单击**“配置”**。

3.  复制当前客户端 IP 地址。如果你从某网络进行连接，则为你的路由器或代理服务器侦听的 IP 地址。SQL Database 会检测当前连接所使用的 IP 地址，以便你可以创建一个接受来自该设备的连接请求的防火墙规则。

4.  将 IP 地址粘贴到起始和结束地址范围中。日后，如果你遇到指示该范围太窄的连接错误，则可以编辑此规则来扩大范围。

    如果客户端计算机使用动态分配的 IP 地址，你必须指定较宽范围的地址，使其足以包含分配给你网络中计算机的 IP 地址。从较窄的范围开始，然后仅在你需要时对其进行扩展。

5.  为防火墙规则输入名称，例如，你的计算机或公司的名称。

6.  单击复选标记保存该规则。

7.  单击页面底部的“保存”以完成该步骤。如果没有看到“保存”，请刷新浏览器页面。

现在，你已创建逻辑服务器、允许来自你的 IP 地址的入站连接的防火墙规则以及管理员登录名。在下一步中，你将切换到本地计算机以完成剩余配置步骤。

**注意：**你刚才创建的逻辑服务器是临时的，它将动态托管在数据中心的物理服务器上。如果你删除该服务器，应当事先知道这是一个不可恢复的操作。一定要备份你随后上载到该服务器的所有数据库。

  [如何：创建逻辑服务器]: #createLogical
  [如何：配置逻辑服务器的防火墙]: #configFWLogical
  [管理门户]: http://manage.windowsazure.cn
