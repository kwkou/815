<properties
   pageTitle="使用 Chef 部署 Azure 虚拟机 | Azure"
   description="了解如何使用 Chef 在 Azure 中自动执行虚拟机的部署和配置"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="diegoviso"
   manager="timlt"
   tags="azure-service-management,azure-resource-manager"
   editor=""/>

<tags 
	ms.service="virtual-machines-windows" 
	ms.date="05/19/2015" 
	wacn.date="11/12/2015"/>

# 使用 Chef 自动执行 Azure 虚拟机部署

Chef 是一个强大的工具，用于提供自动化和所需的状态配置。

使用我们的最新 cloud-api 版本，Chef 提供了与 Azure 的无缝集成，使得你能够通过单个命令设置和部署配置状态。

在本文中，我将向你展示如何设置你的 Chef 环境来设置 Azure 虚拟机，并引导你完成创建一个策略或“食谱”并将此指南部署到 Azure VM 的过程。

让我们开始吧！

## Chef 基础知识

在开始之前，建议你首先熟悉一下 Chef 的基本概念。<a href="http://www.chef.io/chef" target="_blank">此处</a>有大量资料，建议你在尝试此演练之前快速阅读一下。不过，在我们开始之前，我会扼要重述一下基础知识。

下图描绘了概要的 Chef 体系结构。

![][2]

Chef 有三个主要的体系结构组件。**Chef 服务器、Chef 客户端（节点）**和 **Chef 工作站**。

**Chef 服务器**是我们的管理点，对于 Chef 服务器有两种选择：托管解决方案和本地解决方案。我们将使用托管解决方案。

**Chef 客户端（节点）**是位于你在管理的服务器上的代理。

**Chef 工作站**是我们的管理工作站，我们将在其中创建策略并执行管理命令。我们从 Chef 工作站运行**“knife”**命令来管理我们的基础结构。

我们还引入了“食谱”和“配方”的概念。它们实际上是我们定义并应用于我们的服务器的策略。

## 准备工作站

首先，让我们准备工作站。我使用的是标准 Windows 工作站。我们需要创建一个目录来存储我们的配置文件和食谱。

首先，创建一个名为 **C:\\chef** 的目录。

然后，创建另一个目录 **c:\\chef\\cookbooks**

现在，我们需要下载 Azure 设置文件，以便 Chef 可以与我们的 Azure 订阅进行通信。

从此处下载你的发布设置：<a href="https://manage.windowsazure.cn/publishsettings/" target="_blank">https://manage.windowsazure.cn/publishsettings/</a>

将发布设置文件保存到 **C:\\chef** 中

##创建托管的 Chef 帐户

注册托管的 Chef 帐户：<a href="https://manage.chef.io/signup" target="_blank">https://manage.chef.io/signup</a>

在注册过程中，会要求你创建一个新组织。

![][3]

在创建你的组织后，下载初学者工具包。

![][4]

**注意：**如果你收到提示，警告你将重置你的密钥，可以继续操作，因为我们尚未配置现有基础结构。

此初学者工具包 zip 文件包含你的组织的配置文件和密钥。

##配置 Chef 工作站

将 chef-starter.zip 的内容提取到 **C:\\chef**

将 **chef-starter\\chef-repo.chef** 下的所有文件都复制到 **c:\\chef** 中

你的目录现在看起来应当与下图类似：

![][5]

你现在应当有 4 个文件，包括位于根目录 c:\\chef 中的 Azure 发布文件

PEM 文件包含你的组织和用于通信的管理私钥，而 **knife.rb** 文件包含你的 knife 配置。你将需要编辑 **knife.rb** 文件。

在你选择的编辑器中打开该文件并修改“cookbook_path”：从路径中删除 /../，使其如下所示：

	cookbook_path  ["#{current_dir}/cookbooks"]

另外，请添加以下行以反映你的 Azure 发布设置文件的名称。

	knife[:azure_publish_settings_file] = "yourfilename.publishsettings" 

你的 knife.rb 文件现在看起来应当如下所示：

![][6]

这些行将确保 Knife 在执行 Azure 操作期间引用 c:\\chef\\cookbooks 下的 cookbooks 目录并且还使用我们的 Azure 发布设置文件。

## 安装 Chef 开发工具包

接下来，下载并安装 ChefDK（Chef 开发工具包）来设置你的 Chef 工作站。

<a href="http://downloads.getchef.com/chef-dk/windows" target="_blank">http://downloads.getchef.com/chef-dk/windows</a>

![][7]

这简单明了。让其安装在默认位置 c:\\opscode 中。此安装将花费大约 10 分钟时间。

确认你的 PATH 变量包含以下条目：C:\\opscode\\chefdk\\bin;C:\\opscode\\chefdk\\embedded\\bin;c:\\users\\yourusername.chefdk\\gem\\ruby\\2.0.0\\bin

如果未包含，请务必添加这些路径！

**请注意，该路径的顺序非常重要！** 如果你的 opscode 路径未采用正确的顺序，你将会遇到问题。

在继续操作之前重新启动你的工作站。

接下来，我们将安装 Knife Azure 扩展。这将为 Knife 提供“Azure 插件”。

运行以下命令：

	chef gem install knife-azure ––pre

**注意：**–pre 参数确保你收到 knife azure 插件的最新 RC 版本，该版本提供对最新 API 集的访问。

同时，可能还会安装许多依赖项。

![][8]


若要确保所有项都已正确配置，请运行：

	knife azure image list

如果所有项都已正确配置，你会看到可用 Azure 映像的列表滚动显示。

祝贺你。工作站已设置！

##创建食谱

食谱由 Chef 用来定义你希望在托管客户端上执行的一组命令。创建食谱简单明了，我们将使用命令 chef generate cookbook 来生成我们的食谱模板。我将像自动部署 IIS 的策略一样调用我的食谱 Web 服务器。

在 C:\\Chef 目录下运行以下命令：

	chef generate cookbook webserver

这将在 **C:\\Chef\\cookbooks\\webserver** 目录下生成一组文件。 现在，我们需要定义我们希望 Chef 客户端在托管 VM 上执行的一组命令。

这些命令存储在文件 **default.rb** 中。 在此文件中，我将定义一组命令来安装 IIS、启动 IIS 并将模板文件复制到 wwwroot 文件夹中。

修改 **C:\\chef\\cookbooks\\webserver\\recipes\\default.rb** 并添加以下行：

	powershell_script 'Install IIS' do
 		action :run
 		code 'add-windowsfeature Web-Server'
	end

	service 'w3svc' do
 		action [ :enable, :start ]
	end

	template 'c:\inetpub\wwwroot\Default.htm' do
 		source 'Default.htm.erb'
 		rights :read, 'Everyone'
	end

在完成后，保存该文件。

## 创建模板

如上文提到的那样，我们需要生成一个模板文件，该文件将用作我们的 default.html 页面。

执行以下命令来生成模板：

	chef generate template webserver Default.htm

现在导航到文件 **C:\\chef\\cookbooks\\webserver\\templates\\default\\Default.htm.erb** 并编辑该文件。

添加一些简单的“Hello World”html 代码并保存该文件。

## 将食谱上载到 Chef 服务器

在此步骤中，我们将制作我们已在本地计算机上创建的食谱的副本并将其上载到 Chef 托管服务器。上载完成后，食谱将显示在“策略”选项卡下。

	knife cookbook upload webserver

![][9]

## 使用 Knife Azure 部署虚拟机

现在，我们将部署 Azure 虚拟机并应用“Webserver”食谱，该食谱将安装我们的 IIS Web 服务和默认网页。

若要执行此操作，请使用 **knife azure server create** 命令。

下面是该命令的示例：

	knife azure server create --azure-dns-name 'diegotest01' --azure-vm-name 'testserver01' --azure-vm-size 'Small' --azure-storage-account 'portalvhdsxxxx' --bootstrap-protocol 'cloud-api' --azure-source-image 'a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-Datacenter-201411.01-en.us-127GB.vhd' --azure-service-location 'China North' --winrm-user azureuser --winrm-password 'myPassword123' --tcp-endpoints 80,3389 --r 'recipe[webserver]'

各个参数不言自明。替换你的特定变量并运行。

**注意：**通过命令行，我还通过使用 –tcp-endpoints 参数自动执行终结点网络筛选器规则。我已打开端口 80 和 3389 来提供对我的网页和 RDP 会话的访问。

在运行该命令后，访问 Azure 经典管理门户，你会看到你的计算机已开始设置。

![][13]

返回到命令提示符：

![][10]

在部署完成后，我们应当能够通过端口 80 连接到 web 服务，因为我们在通过 knife azure 命令设置 VM 时已打开了该端口。因为此 VM 是我的云服务中的唯一 VM，所以我将使用云服务 URL 来连接它。

![][11]

如你所见，我的 html 代码富有创造性:)

不要忘记，我们还可以从 Azure 经典管理门户使用端口 3389 通过 RDP 会话进行连接。

希望这对你有所帮助！ 现在就使用 Azure 开始你的基础结构即代码之旅吧！



<!--Image references-->
[2]: ./media/virtual-machines-windows-chef-automation/2.png
[3]: ./media/virtual-machines-windows-chef-automation/3.png
[4]: ./media/virtual-machines-windows-chef-automation/4.png
[5]: ./media/virtual-machines-windows-chef-automation/5.png
[6]: ./media/virtual-machines-windows-chef-automation/6.png
[7]: ./media/virtual-machines-windows-chef-automation/7.png
[8]: ./media/virtual-machines-windows-chef-automation/8.png
[9]: ./media/virtual-machines-windows-chef-automation/9.png
[10]: ./media/virtual-machines-windows-chef-automation/10.png
[11]: ./media/virtual-machines-windows-chef-automation/11.png
[13]: ./media/virtual-machines-windows-chef-automation/13.png


<!--Link references-->

<!---HONumber=79-->