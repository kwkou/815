<properties
	pageTitle="Azure 企业门户管理手册 | Azure"
	description="详细介绍如何管理Azure账户、订阅及查看相应的账单。"
	services="ea-portal"
	documentationCenter=""
	authors="Lei Zhang"
	manager=""
	editor=""/>

<tags
	ms.service="ea-portal"
	ms.date=""
	wacn.date="06/22/2016"/>

#Azure 企业门户管理手册

##<a id="foreword"></a>1. 前言  

**本文原文为微软云构架师 Lei Zhang 所写，可访问其[个人博客](http://www.cnblogs.com/threestone/archive/2012/01/06/2382322.html)。**

##<a id="reader"></a>2. 读者
Azure 是平台产品，本文只详细介绍如何管理 Azure 账户、订阅及查看相应的账单。本文适合 IT 管理人员和 Azure 账户运维人员阅读。

##<a id="concept"></a>3. Azure China 基本概念

###<a id="msdn-subscription"></a>3.1 MSDN 订阅
我在 Azure Global 拥有测试账号或者 MSDN 订阅账号，这个账号可以在国内 Azure China 使用吗？  

回答:不可以。 Azure 在国内和国外有 2 套系统。  

(1)	Azure Global。域名是 azure.microsoft.com。是由微软运营的，数据中心在中国大陆以外的地区。客户可以通过 Windows Live ID 登陆，MSDN 订阅用户每月可以免费试用一定额度的 Azure 资源。  

(2)	Azure China。域名是 www.azure.cn。是由世纪互联运维的，与全球其他地区由微软运营的 Azure 服务在物理上和逻辑上都是独立的，数据中心在中国大陆有两个：北京和上海。用户无法通过 Windows Live ID 登陆。只能通过 Org ID 来进行使用。  

###<a id="orgid"></a>3.2 Org ID
Org ID 是 Azure China 特殊的用户名系统。  

一个企业用户可以使用 <username\>@<OrgID\>.partner.onmschina.cn 来登陆使用 Azure China。 

比如公司名为 contoso 的企业，可以注册 Org ID 为 

	contoso.partner.onmschina.cn

注意后面的 partner.onmschina.cn 为固定的后缀。

###<a id="account"></a>3.3 账户
我们创建完 Org ID 后，可以使用这个 Org ID，创建不同的账户。  

比如我需要创建管理员账号，就可以创建的账户为  
	
	admin@contoso.partner.onmschina.cn

或者可以给员工 xiaowang 创建普通用户账号为  

	xiaowang@contoso.partner.onmschina.cn

以上 2 个账户的账户名称不同，密码也不同，通过分别使用不同的账户名称和密码，就可以管理使用 Azure China。

###<a id="subscription"></a>3.4 订阅
订阅也是 Azure 非常重要的概念。  

首先举个例子，大家都熟悉双卡双待的手机。比如有一台双卡双待的手机，购买了中国移动和中国联通的 SIM 卡(后付费)，我就可以通过中国移动或者中国联通的 SIM 卡，进行通话、发短信，或者上网浏览。  

到每个月底的时候，中国移动和中国联通会分别快递两张手机账单  

(1)	中国移动的手机账单会告诉用户，这个月的分类费用，包括通话时间为多少分钟，短信发送了多少条，上网浏览用了多少流量。同时也会显示总体费用是多少。  

(2)	中国联通的手机账号也会告诉用户，这个月话费的详细账单和总费用。

在 Azure 中，一个 EA (Enterprise Agreement) 合同可以创建无限多个订阅。  

订阅就好比一张 SIM 卡。比如用户可以创建 ABC，XYZ 两个不同的订阅，并且可以在这 2 个不同的订阅中，创建和使用 Azure 服务，比如：虚拟机，存储，SQL 数据库等等。订阅就类似资源池的概念。  

到月底的时候，Azure 系统会产生一张详细的使用清单：  

(1)	告诉用户 ABC 订阅中，虚拟机的总费用是多少，存储的总费用是多少，SQL 数据库的总费用是多少。  
(2)	同时也会显示 XYZ 订阅中，产生的费用的详细信息。  
(3)	最后会显示所有订阅的总体费用是多少。  

注意:测试用户只能创建和使用一个订阅，而正式商用的用户可以创建无数个订阅。

###<a id="account-subscription-relation"></a>3.5 账户和订阅的关系
在 Azure 中，账户和订阅的关系是多对多的。假设有 2 个订阅 IT 和 Market，账户管理员可以创建若干个账户，对应关系如下：
<table width="100%" border="1" cellspacing="0" cellpadding="0" style="text-align:center">
	<tr>
		<td>序号</td>
		<td>账户</td>
		<td>订阅</td>
	</tr>
	<tr>
		<td>1</td>
		<td>xiaozhang@contoso.partner.onmschina.cn</td>
		<td>IT</td>
	</tr>
	<tr>
		<td>2</td>
		<td>mike@contoso.partner.onmschina.cn</td>
		<td>Market</td>
	</tr>
	<tr>
		<td>3</td>
		<td>tom@contoso.partner.onmschina.cn</td>
		<td>Market</td>
	</tr>
	<tr>
		<td>4</td>
		<td>xiaozhang@contoso.partner.onmschina.cn</td>
		<td>Market</td>
	</tr>
</table> 

上表中，Mike 和 Tom 可以使用的订阅只有一个，他们可以使用订阅 Market，来使用 Azure 云服务。 xiaozhang 的账号不同，他可以使用 2 个订阅 (IT 和 Market)。xiaozhang 可以同时使用这 2 个不同的订阅，来使用 Azure 云服务。

###<a id="subscription-scenarios"></a>3.6 订阅的实际使用场景
对于企业级客户来说，账户和订阅可以方便进行内部成本核算。  

比如 Contoso 集团公司，在国内三个城市，北京、上海和杭州都拥有软件研发基地。为了使用 Azure China 云平台来提高服务水平，Contoso 集团只需要和世纪互联签署一份 Azure 商务合同，创建一个 Org ID: consoto.  

同时账户管理员可以创建三个订阅和三个账户。对应关系如下：
<table width="100%" border="1" cellspacing="0" cellpadding="0" style="text-align:center">
	<tr>
		<td>序号</td>
		<td>账户</td>
		<td>订阅</td>
	</tr>
	<tr>
		<td>1</td>
		<td>beijing@contoso.partner.onmschina.cn </td>
		<td>Beijing_sub</td>
	</tr>
	<tr>
		<td>2</td>
		<td>shanghai@contoso.partner.onmschina.cn </td>
		<td>Shanghai_sub</td>
	</tr>
	<tr>
		<td>3</td>
		<td>hangzhou@contoso.partner.onmschina.cn</td>
		<td>Hangzhou_sub</td>
	</tr>
</table> 

这样北京、上海和杭州的用户可以使用上面三个不同的账户，进行各自的软件研发和部署。同时，Contoso 集团总部在月底会收到世纪互联发送的 Azure 账单，里面显示了三个不同的订阅 (Beijing\_sub，Shanghai\_sub，Hangzhou\_sub) 分别产生的费用是多少，以及 Azure 总的费用是多少。根据账单的信息，Contoso 集团可以根据 Azure 提供的账单，对三个研发基地进行内部成本核算。

请注意：订阅和订阅之间的信息是互相隔离的，也就是说，订阅 Beijing\_sub 的用户看不到订阅 Shanghai\_sub 创建的虚拟机，存储，SQL 数据库等。同样的，订阅 Shanghai\_sub 用户也看不到订阅 Hangzhou\_sub 用户创建的虚拟机，存储，SQL 数据库等。这样可以保证服务的隔离，不会因为误操作而互相影响。

###<a id="account-permission"></a>3.7 账户权限  
在 Azure 中，权限从高到低分为三种，分别为  

-	**企业管理员**  
企业管理员能够向合约添加账户或将账户与注册关联，可跨所有账户查看使用量数据，还可以查看与注册关联的货币承诺余额。针对一个注册的企业管理员的数量不受限制。

-	**账户管理员**  
账户所有者可为其账户添加订阅，针对单独的订阅更新服务管理员和共同管理员、查看其账户的使用量数据，以及在企业管理员提供了访问权限的情况下查看账户费用。除非账户所有者同时具有企业管理员权限，否则将看不到资金承诺余额。

-	**服务管理员**  
服务管理员以及每个订阅的最多 199 个共同管理员能够访问和管理 Azure 管理门户内的订阅和开发项目。除非服务管理员同时具有其他两个角色之一，否则将没有企业门户的访问权限。  

![Enterprise Azure 角色和门户][1]  

注意：Azure 测试账号没有企业管理员。  

##<a id="use-azure-enterprise-portal"></a>4.使用 Azure 企业门户
Azure 企业门户拥有以下功能：  

-	创建部门  
-	创建账户  
-	创建订阅  
-	查看账单信息  

###<a id="enterprise-foreword"></a>4.1	前言  
本章节介绍如何使用 Azure 企业门户。

###<a id="enter-account"></a>4.2 录入账户

(1)	请先确认您拥有一个 Azure 测试账户  

(2)	在使用 Azure 企业门户之前，请先确认您已经签署了 Azure 企业协议 (Enterprise Agreement) 合同  

(3)	请确认您已经把测试账户的信息提交给世纪互联后台的运维团队，在后台进行录入


###<a id="simulations"></a>4.3 模拟场景

在本章中，我们将会模拟以下场景：  

(1)	Contoso 企业管理员拥有 Azure China 测试账号 admin@contoso.partner.onmschina.cn, 该测试账号有一个默认的测试订阅  

(2)	该企业管理员需要将该测试账号升级为企业管理员，并且将测试订阅切换为生产订阅  

(3)	该企业管理员需要为 Market 部门创建一个新 Azure 账户 market@contoso.partner.onmschina.cn, 为该账户创建新的订阅并且重命名  
  
(4)	Contoso 企业管理员将 admin 账号加到 Market 订阅下，作为协同管理员

###<a id="switch-to-production-subscription"></a>4.4 将测试订阅切换为生产订阅

(1)	Contoso 企业管理员登陆 [Azure 企业门户](http://ea.windowsazure.cn)，用户名为测试账号的用户名 admin@contoso.partner.onmschina.cn 点击”添加账户”，如下图：

![添加账户][2]

(2)	按照流程，将测试账户 (admin@contoso.partner.onmschina.cn) 添加为一个账户管理员  

![添加管理员][3]

(3)	添加成功后，测试账户 (admin@contoso.partner.onmschina.cn) 将列在账户列表中

![账户列表][4]  

(4)	注销当前用户，以测试账户 (admin@contoso.partner.onmschina.cn) 重新登陆 [Azure 企业门户](http://ea.windowsazure.cn)。登陆成功后，请点击”刷新订阅”  

![刷新订阅][5]  

(5)	在”刷新订阅”后，将会看到测试订阅被成功关联到企业门户网站中。并且会看到 ”Convert to EA” 的注释：  

![关联到企业门户][6] 

注意：当您的测试订阅切换为生产订阅后，在原订阅的下的所有服务都不会宕机，直接切换。但是您已经创建的所有 Azure 服务会开始计费。  

###<a id="create-new-department"></a>4.5 创建新的部门
部门这个概念在 Azure 企业门户里，是多个账户的集合。  

(1)	我们可以按照公司的职能，将部门区分为金融、市场营销  

(2)	还可以按照业务，将部门区分为汽车、航天  

(3)	或者按照地理位置，将部门区分为北美、欧洲  

如下图所示:  

![部门分类][7] 

(1)	我们以企业管理身份 (admin@contoso.partner.onmschina.cn)，登陆 [Azure 企业门户](http://ea.windowsazure.cn)。  

![部门][8] 

(2)	我们添加一个新的部门，命名为市场营销。如下图:  

![添加部门][9]   

**注意支出配额只是一个虚拟概念**。比如我们设置了 5 万元，当这个部门的所有订阅使用额度达到 5 万元，EA Portal 会发送系统邮件给相应的收件人。但是并不会停止订阅。

(3)	创建完毕后，显示如下：  

![添加部门成功][10]

###<a id="create-new-account"></a>4.6 创建新的账户
在上节中，Contoso 企业管理员 (Admin) 已经将测试订阅切换为生产订阅。现在为需要为 Market 部门创建一个新 Azure 账户 market@contoso.partner.onmschina.cn 

(1)	首先登陆Office 365平台，网址是 https://login.partner.microsoftonline.cn/ 输入用户名： admin@contoso.partner.onmschina.cn 和相应的密码。  

(2)	点击”+”，开始创建新的账户  

![创建新的账户][11]  

(3)	在弹出的窗口中，输入相应的信息。我们可以选择”自动生成的密码”或者”键入密码”。如下图:  

![生成密码][12] 

(4)	点击完上面的创建按钮后，记得先把创建的用户名和临时密码保存在本地的记事本上，防止忘记。

![创建新的账户成功][13]  

(5)	然后我们以 admin@contoso.partner.onmschina.cn 账户身份，登陆 [Azure 企业门户](https://ea.windowsazure.cn/)     

![添加账户][14]  

(6)	在弹出的界面中，输入我们在 Office 365 中创建的 Market 部门的账户。
Department 我们选择在上一节中创建的部门“市场营销”
点击”添加”按钮，如下图：  

![添加账户][15] 

添加完毕后，我们可以在企业门户的账户栏目中，查看到我们创建的 market 账户。如下图：  

![添加账户成功][16]   

注意上图中的 market 账户状态为等待。表示该账号已经被创建，但是暂未使用。

###<a id="create-subscription"></a>4.7 创建新的订阅
**创建订阅是不会产生任何费用的。**

在上节中，企业管理员 (admin) 已经为 market 部门创建了新的账户 
market@yumchina.partner.onmschina.cn，该账户系统分配了默认的登陆密码。

接下来我们需要为 market 账户创建订阅。  

(1)	在 [Azure 企业门户](https://ea.windowsazure.cn/)，我们注销当前的登陆信息。

(2)	重新以 market 账户登录企业门户，输入帐户名 
market@yumchina.partner.onmschina.cn 和系统分配的登陆密码 

(3)	因为我们在上节中要求用户登陆时更改密码，所以我们按照下图修改登录密码：  

![修改密码][17]   

然后我们以新密码重新登陆企业门户  

(4)	重新登陆后，我们就以 market 账户，登陆 [Azure 企业门户](https://ea.windowsazure.cn/)。 默认的 market 是没有任何的订阅的，我们点击下图的”添加订阅”  

![修改密码][18]

(5)	**注意请在浏览器设置”允许弹出窗口”**。 在弹出窗口中，输入相应的信息，并点击”注册”  

![注册][19]  

请保持窗口不要关闭，直至新的订阅创建完毕。  

###<a id="rename-subscription"></a>4.8 将订阅名称重命名
在默认情况下，新创建的 Azure 订阅名称都是 ”Azure 企业”。从可读性角度考虑，我们需要将订阅名称重命名，比如命名为 Marketing\_ Subscription，Sales\_Subscription，IT\_ Subscription 等。这样可读性、可管理性会更好。  

本节我们修改 Market 账号 (market@contoso.partner.onmschina.cn) 的订阅名称。 
 
(1)	打开浏览器，输入地址：https://account.windowsazure.cn/subscriptions  

(2)	输入 Market 账号 market@contoso.partner.onmschina.cn 和密码  

(3)	选中默认的订阅名称  

![选择订阅][20]  

(4)	页面跳转后，选择”编辑订阅详细信息”  

![编辑订阅详细信息][21]  

(5)	在弹出的界面中，修改订阅名称。**注意不要修改管理员**。然后勾选保存。  

![编辑订阅详细信息][22]  

(6)	这样就完成了修改订阅名称的步骤。  

![编辑订阅详细信息][23]  

###<a id="config-coadmin"></a>4.9 设置共同管理员
在上面的几节内容中，我们实现了  

(1)	Azure 企业管理员 (admin@contoso.partner.onmschina.cn)，将默认订阅切换为生产订阅  

(2)	我们以企业管理员 (admin) 账户身份，为 Market 部门创建新的账户
(market@contoso.partner.onmschina.cn)，而且为该 market 账户创建新的订阅并且重命名  

(3)	企业管理员 (Admin) 拥有的权限：  

&nbsp;&nbsp;&nbsp;&nbsp;a) 企业管理员，可以查看 Azure 账单使用情况  

&nbsp;&nbsp;&nbsp;&nbsp;b) 账户管理员，拥有自己的订阅  

(4)	Market账户拥有的权限：  

&nbsp;&nbsp;&nbsp;&nbsp;a)账户管理域，拥有自己的订阅  

请注意：上面的步骤完成后，从查看账单的权限来看：  

(1)	企业管理员 (Admin) 可以查看 Azure 账单使用情况  

(2)	Market 账户是账户管理员，没有查看 Azure 账单的权限  

从订阅角度来看，两个账户的关系如下：  

(1)	企业管理员 (Admin) 是服务管理员无法查看到 Market 账户的订阅 (Marketing\_Subscription)  

(2)	Market 账户也无法查看到企业管理员 (Admin) 的订阅  

现在我们需要让企业管理员 (Admin) 有查看 Market 账户下订阅的权限。就需要将企业管理域 (Admin) 设置为 Market 账户下订阅的共同管理员。  

(1)	首先我们以 Market 账户 (market@contoso.partner.onmschina.cn)，登陆 [Azure 管理门户](https://manage.windowsazure.cn)  

(2)	点击左侧列表中的”设置”，然后选择”管理员”，点击按钮”添加”  

![添加协同管理员账号][24]  

(3)	在弹出的窗口中，输入企业管理员的账户信息，并勾选订阅。如下图：  

![添加协同管理员账号][25]  

(4)	然后我们注销当前登录，以企业管理员 (admin) 身份，登陆 [Azure 管理门户](https://manage.windowsazure.cn)
可以查看到，企业管理员 (admin) 即使自己订阅的服务管理员，又是 market 账户订阅 (Market\_Subscription) 的协同管理员。  

![添加协同管理员账号][26] 

###<a id="enterprise-summary"></a>4.10 总结
经过本章节的操作后，我们可以发现：  

**(1)	企业管理员 (Admin) 拥有的权限：**  

&nbsp;&nbsp;&nbsp;&nbsp;a)	企业管理员，可以查看 Azure 账单使用情况  

&nbsp;&nbsp;&nbsp;&nbsp;b)	账户管理员，拥有自己的订阅  

&nbsp;&nbsp;&nbsp;&nbsp;c)	共同管理员，是 market 账户订阅 (Market_Subscription) 的协同管理员  

**(2)	Market 账户拥有的权限：**  

&nbsp;&nbsp;&nbsp;&nbsp;a)	账户管理域，拥有自己的订阅  

##<a id="azure-billing"></a>5. 查询 Azure 账单

###<a id="azure-billing-foreword"></a>5.1 前言
**请注意：只有企业管理员才拥有查询 Azure 账单的权限。**  
在开始本节内容之前，希望能对 Excel 数据透视表有一定的了解  

###<a id="azure-billing-overview"></a>5.2 查看总体使用情况  
我们以企业管理员身份 (admin@contoso.partner.onmschina.cn) ，登陆 [Azure 企业门户](http://ea.windowsazure.cn)。
点击”报表”，”使用量摘要”。如下图：  

![使用量摘要][27]  

下图中的图例具体含义是：  

1.	蓝色的线代表客户购买及调整及剩余金额  

2.	红色的柱表示客户每月超额情况  

3.	绿色的柱表示客户每月实际消耗  

**注意：Azure 允许客户超额使用，即客户实际使用量大于客户账户的余额。

比如一个客户 1 月份的账户余额为 10 万元，1 月份当月实际使用 11 万元。则超额消耗 1 万元。**

我们可以点击某一个柱状图，显示当月的总是使用情况。如下图:  

![使用量摘要][28] 

在上面的界面中，我们还可以点击右上角的滚动条。  

![使用量摘要][29]  

通过表格形式，查看 Azure 总体使用情况。如下图：  

![使用量摘要][30]  

最后我们下拉滚动条，可以查看到当月的详细使用情况。  

![使用量摘要][31]

###<a id="azure-billing-details"></a>5.3 查看账单详细使用情况  
Azure 可以通过下载 Excel 表示，将一段时间内 Azure 的详细账单，通过透视表的方式进行自定义查询。  

1.	我们点击报表，下载使用量  

![下载使用量][32]  

2.	点击上图的按钮，就可以下载使用量数据。如果未显示，则点击下图的”刷新”按钮  

![刷新][33]  

3.	下载完毕后，我们点击下载的 Excel 文件。全选第 3 行的列名，然后按 CTRL+SHIFT+END，选中所有的表格内容。
4.	然后点击”插入”->”数据透视表”。如下图：  

![数据透视表][34]  

5.	我们在透视表中，拖动字段。如下图：  

![字段][35]  

就可以展示相应的数据透视表了。如下图：  

![数据透视表][36]  

如果您熟悉数据透视表，可以按照上面的步骤，自定义创建报表。  

###<a id="email-notification"></a>5.4 使用邮件通知
当我们不想登陆 [Azure 企业门户](http://ea.windowsazure.cn)，想让系统定期发送 Azure 账单的情况。我们可以进行如下操作：  

1.	我们以企业管理员身份 (admin@contoso.partner.onmschina.cn) ，登陆 [Azure 企业门户](http://ea.windowsazure.cn)。  

![登陆][37]  

2.	在添加联系人的弹出框中，我们可以输入相应的收件人邮箱地址：  

![添加联系人][38]  

3.	这样，就可以按照一定的通知频率。通知相应的收件人。您会收到类似如下图的邮件内容：  

![添加联系人][39]  

###<a id="azure-billing-exceed"></a>5.5 关于 Azure 超额使用  
当用户累计的使用情况，达到整个合同金额的 75%，90%，100%，125% 的时候， Azure 会自动发送邮件给相应的联系人，提供合同金额并给予提示。





<!--image-->
[1]: ./media/azure-ea-portal-user-manual/enterprise-azure-role-and-portal.png
[2]: ./media/azure-ea-portal-user-manual/add-account.png
[3]: ./media/azure-ea-portal-user-manual/add-admin-account.png
[4]: ./media/azure-ea-portal-user-manual/account-list.png
[5]: ./media/azure-ea-portal-user-manual/refresh-subscription.png
[6]: ./media/azure-ea-portal-user-manual/convert-to-ea.png
[7]: ./media/azure-ea-portal-user-manual/department-classify.png
[8]: ./media/azure-ea-portal-user-manual/portal-department.png
[9]: ./media/azure-ea-portal-user-manual/add-department.png
[10]: ./media/azure-ea-portal-user-manual/add-department-success.png
[11]: ./media/azure-ea-portal-user-manual/create-new-account.png
[12]: ./media/azure-ea-portal-user-manual/generate-password.png
[13]: ./media/azure-ea-portal-user-manual/create-new-account-success.png
[14]: ./media/azure-ea-portal-user-manual/add-account-portal.png
[15]: ./media/azure-ea-portal-user-manual/add-account-portal-details.png
[16]: ./media/azure-ea-portal-user-manual/add-account-portal-success.png
[17]: ./media/azure-ea-portal-user-manual/update-password.png
[18]: ./media/azure-ea-portal-user-manual/add-subscription-portal.png
[19]: ./media/azure-ea-portal-user-manual/register-info.png
[20]: ./media/azure-ea-portal-user-manual/select-subscription.png
[21]: ./media/azure-ea-portal-user-manual/edit-subscription.png
[22]: ./media/azure-ea-portal-user-manual/edit-subscription-name.png
[23]: ./media/azure-ea-portal-user-manual/edit-subscription-name-success.png
[24]: ./media/azure-ea-portal-user-manual/add-admin-account-portal.png
[25]: ./media/azure-ea-portal-user-manual/add-admin-account-portal-2.png
[26]: ./media/azure-ea-portal-user-manual/add-admin-account-success.png
[27]: ./media/azure-ea-portal-user-manual/azure-billing-overview.png
[28]: ./media/azure-ea-portal-user-manual/azure-billing-month-usage.png
[29]: ./media/azure-ea-portal-user-manual/azure-billing-overview-scroll.png
[30]: ./media/azure-ea-portal-user-manual/azure-billing-overview-table.png
[31]: ./media/azure-ea-portal-user-manual/azure-billing-overview-details.png
[32]: ./media/azure-ea-portal-user-manual/azure-billing-details-download.png
[33]: ./media/azure-ea-portal-user-manual/azure-billing-details-download-refresh.png
[34]: ./media/azure-ea-portal-user-manual/azure-billing-details-download-data-pivot-table.png
[35]: ./media/azure-ea-portal-user-manual/azure-billing-details-download-data-pivot-table-setting.png
[36]: ./media/azure-ea-portal-user-manual/azure-billing-details-download-data-pivot-table-finish.png
[37]: ./media/azure-ea-portal-user-manual/email-notification-login.png
[38]: ./media/azure-ea-portal-user-manual/email-notification-add-contacts.png
[39]: ./media/azure-ea-portal-user-manual/email-notification.png