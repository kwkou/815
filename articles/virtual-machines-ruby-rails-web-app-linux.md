<properties linkid="dev-ruby-web-app-with-linux-vm" urlDisplayName="Ruby on Rails Web App on Azure using Linux VM" pageTitle="Ruby on Rails Web 应用程序在 Azure 中使用 Linux 虚拟机" metaKeywords="Azure Ruby web application, Azure Ruby application, Ruby app Azure, Ruby azure vm, ruby virthal machine, ruby linux vm" description="托管 Ruby on Rails 基于在 Azure 中使用 Linux 虚拟机的网站。 " metaCanonical="" services="virtual-machines" documentationCenter="Ruby" title="Ruby on Rails Web application on an Azure VM" authors="larryfr" solutions="" manager="" editor="" />





#Azure 虚拟机上的 Ruby on Rails Web 应用程序

本教程介绍如何以托管 Ruby on Rails 基于在 Azure 中使用 Linux 虚拟机的网站。本教程假定您之前未使用过 Azure。完成本教程之后，您将能够在云中启动和运行基于 Ruby on Rails 的应用程序。

您将了解如何执行以下操作：

* 设置开发环境

* 设置 Azure 虚拟机以托管 Ruby on Rails。

* 新建 Rails 应用程序

* 使用 scp 将文件复制到虚拟机 

下面是已完成应用程序的屏幕快照：

![a browser displaying Listing Posts][blog-rails-cloud]

##本文内容

* [设置开发环境](#setup)

* [创建 Rails 应用程序](#create)

* [测试应用程序](#test)

* [创建 Azure 虚拟机](#createvm)

* [将复制到虚拟机的应用程序](#copy)

* [安装 gem 并启动该应用程序](#start)

* [接下来的步骤](#next)

##<a id="setup"></a>设置开发环境

1. 在开发环境中安装 Ruby。具体步骤因操作系统而异。

	* **Apple OS X** -有多个有关 OS X 的 Ruby 版本。本教程使用在 OS X 上验证[Homebrew](http://brew.sh/)安装**rbenv**和**ruby 生成**。在找不到安装信息 [https://github.com/sstephenson/rbenv/](https://github.com/sstephenson/rbenv/).

	* **Linux** -使用您分发的包管理系统。本教程使用 ruby1.9.1 和 ruby1.9.1-dev 程序包在 Ubuntu 12.10 上进行了验证。

	* **Windows** -有多个针对 Windows 的 Ruby 版本。本教程使用验证[RailsInstaller](http://railsinstaller.org/) 1.9.3-p392。

2. 打开一个新命令行或终端会话并输入以下命令以安装 Ruby on Rails：

		gem install rails --no-rdoc --no-ri

	<div class="dev-callout">
	<strong>说明</strong>
	<p>此命令可能需要管理员或在某些操作系统上的根权限。如果在运行此命令时收到错误，请尝试按如下所示使用 "sudo"。</p>
	< pre 类 ="prettyprint"> sudo gem 安装导轨 < / pre >
	</div>

	<div class="dev-callout">
	<strong>说明</strong>
	<p>本教程使用的是 Rails gem 版本 3.2.12。</p>

	</div>

3. 您还必须安装 JavaScript 解释程序，Rails 将使用它来编译您的 Rails 应用程序使用的 CoffeeScript 资产。在提供了支持的解释程序的列表[https://github.com/sstephenson/execjs#readme](https://github.com/sstephenson/execjs#readme)。
	
	[Node.js](http://nodejs.org/)已使用的本教程中，验证过程中，因为它是适用于 OS X、 Linux 和 Windows 操作系统。

##<a id="create"></a>创建 Rails 应用程序

1. 从命令行或终端会话中，使用以下命令创建一个名为"blog_app"的新 Rails 应用程序：

		rails new blog_app

	此命令将创建一个名为的新目录**blog_app**，文件和子目录 Rails 应用程序所需并用其填充它。

	<div class="dev-callout">
	<strong>说明</strong>
	<p>此命令可能需要一分钟或更长时间才能完成。它执行无提示安装的默认应用程序中，为所需的 gem，并在此期间可能出现挂起现象。</p>
	</div>

2. 将目录更改为**blog_app**目录中，然后使用下面的命令创建一个基本博客基架：

		rails generate scaffold Post name:string title:string content:text

	这将创建用于将文章保存到博客的控制器、视图、模型和数据库迁移。每篇文章都有一个作者姓名、文章标题和文本内容。

3. 若要创建将存储博客文章的数据库，请使用以下命令：

		rake db:migrate

	这将对 Rails，这是使用默认数据库提供程序[SQLite3 数据库][sqlite3]。尽管您可能想要对生产应用程序使用不同的数据库，则 SQLite 就足够了本教程的目的。

##<a id="test"></a>测试应用程序

执行以下步骤在你的开发环境中启动 Rails 服务器

1. 从命令行或终端会话中，命令启动 rails 服务器使用以下命令：

		rails s

	您应该会看到与下面类似的输出。记下 Web 服务器正在侦听的端口。在下面的示例中，它正在侦听端口 3000。

		=> Booting WEBrick
		=> Rails 3.2.12 application starting in development on http://0.0.0.0:3000
		=> Call with -d to detach
		=> Ctrl-C to shutdown server
		[2013-03-12 19:11:31] INFO  WEBrick 1.3.1
		[2013-03-12 19:11:31] INFO  ruby 1.9.3 (2012-04-20) [x86_64-linux]
		[2013-03-12 19:11:31] INFO  WEBrick::HTTPServer#start: pid=9789 port=3000

2. 打开浏览器并导航到 http://localhost:3000/。您应该看到类似于以下页面：

	![default rails page][default-rails]

	该页面是一个静态欢迎页面。若要查看基架命令生成的表单，请导航到 http://localhost:3000/文章。您应该看到类似于以下页面：

	![a page listing posts][blog-rails]

	若要结束服务器进程，请在命令行按 Ctrl+C

##<a id="createvm"></a>创建 Azure 虚拟机

请按照提供的说明[此处][vm 说明]创建托管 Linux 的 Azure 虚拟机。

<div class="dev-callout">
<strong>说明</strong>

<p>本教程中的步骤是在托管 Ubuntu 12.10 的 Azure 虚拟机上执行的。如果您使用的是不同的 Linux 版本，可能需要执行不同的步骤才能完成相同的任务。</p></div>

<div class="dev-callout">
<strong>说明</strong>
<p>您 <strong>仅</strong> 需要创建虚拟机。在了解了如何使用 SSH 连接到虚拟机后停止。</p>
</div> 

在创建 Azure 虚拟机后, 执行以下步骤以在虚拟机上安装 Ruby 和 Rails：

1. 从命令行或终端会话中，使用以下命令连接到虚拟机使用 SSH：

		ssh username@vmdns -p port

	替换在创建虚拟机的过程中指定的用户名、虚拟机的 DNS 地址和 SSH 终结点的端口。例如：

		ssh railsdev@railsvm.chinacloudapp.cn -p 61830

	<div class="dev-callout">
	<strong>说明</strong>
	<p>如果您使用 Windows 作为开发环境，您可以使用一个实用程序如 <b>PuTTY</b> SSH 功能。可以从获取 puTTY <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html">PuTTY 下载页</a>.</p>
	</div>

2. 从 SSH 会话中，使用以下命令以在 VM 上安装 Ruby：

		sudo apt-get update -y
		sudo apt-get upgrade -y
		sudo apt-get install ruby1.9.1 ruby1.9.1-dev build-essential libsqlite3-dev nodejs -y

	安装完成后，使用以下命令验证 Ruby 是否已成功安装：

		ruby -v

	这应返回安装在虚拟机，这可能不同于 1.9.1 的 Ruby 的版本。例如， **ruby 1.9.3 p 194 (2012年-04-20 修订版 35410) [x86_64-linux]**。

2. 使用以下命令安装捆绑包：

		sudo gem install bundler --no-rdoc --no-ri

	捆绑包将用于安装 rails 应用程序被复制到服务器后所需的 gem。

##<a id="copy"></a>将应用程序复制到虚拟机

从开发环境中，打开一个新的命令行或终端会话并使用**scp**命令以便将复制**blog_app**目录到虚拟机。此命令的格式为，如下所示：

	scp -r -P 54822 -C directory-to-copy user@vmdns:

例如：

	scp -r -P 54822 -C ~/blog_app railsdev@railsvm.chinacloudapp.cn:

<div class="dev-callout">
<strong>说明</strong>
<p>如果您使用 Windows 作为开发环境，您可以使用一个实用程序如 <b>pscp</b> 为实现 scp 功能。可以从获取 Pscp <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html">PuTTY 下载页</a>.</p>
</div>

用于此命令的参数具有以下作用：

* **-r**:以递归方式复制指定目录和所有子目录中的内容

* **-P**:使用指定的端口进行 SSH 通信

* **-C**:启用压缩

* **directory 副本**：要复制的本地目录

* **user@vmdns**：要将文件复制到其中的计算机的地址以及用于登录的用户帐户

在复制操作之后, **blog_app**目录将位于用户的主目录。在与虚拟机的 SSH 会话中使用以下命令查看已复制的文件：

	cd ~/blog_app
	ls

应与中包含的文件相匹配，则返回的文件列表**blog_app**目录在开发环境中。

##<a id="start"></a>安装 gem 并启动 Rails

1. 在虚拟机，将目录更改为**blog_app**目录，然后使用以下命令以安装中指定的 gem **Gemfile**：

		sudo bundle install

2. 若要创建数据库，请使用以下命令：

		rake db:migrate

3. 使用以下命令以启动服务器：
	
		rails s

	您应该会看到与下面类似的输出。记下 Web 服务器正在侦听的端口。在下面的示例中，它正在侦听端口 3000。

		=> Booting WEBrick
		=> Rails 3.2.12 application starting in development on http://0.0.0.0:3000
		=> Call with -d to detach
		=> Ctrl-C to shutdown server
		[2013-03-12 19:11:31] INFO  WEBrick 1.3.1
		[2013-03-12 19:11:31] INFO  ruby 1.9.3 (2012-04-20) [x86_64-linux]
		[2013-03-12 19:11:31] INFO  WEBrick::HTTPServer#start: pid=9789 port=3000

2. 在浏览器中，导航到[Azure 管理门户][管理门户]，然后选择您的虚拟机。

	![virtual machine list][vmlist]

3. 选择**终结点**页上，，然后单击顶部**+ 添加终结点**在页面的底部。

	![endpoints page][endpoints]

4. 在**添加终结点**对话框中，单击左下角以继续进行第二页中的箭头，然后在窗体中输入以下信息：

	* **NAME**: HTTP

	* **PROTOCOL**: TCP

	* **PUBLIC PORT**: 80

	* **PRIVATE PORT**: & l t; 端口信息从上面的步骤 3 & g t;

	这将创建一个公用端口 80，以便将流量路由到 3000-即 Rails 服务器侦听的专用端口。

	![new endpoint dialog][new-endpoint]

5. 单击复选标记保存该终结点。

6. 应显示一条消息，指出**更新正在进行的**。一旦此消息消失后，该终结点处于活动状态。现在你可以通过导航到虚拟机的 DNS 名称来测试你的应用程序。该网站应类似于以下：

	![default rails page][default-rails-cloud]

	追加**/posts**到 URL 应该会显示基架命令生成的页面。

	![posts page][blog-rails-cloud]

##<a id="next"></a>后续步骤

在这篇文章中，您已了解如何创建和发布的基本基于窗体的 Rails 应用程序到 Azure 虚拟机。我们执行的操作的大多数都是手动的而且在生产环境中，它会是一种需要自动执行。此外，大多数生产环境承载 Rails 应用程序结合使用与另一个服务器进程 （如 Apache 或 NginX，处理请求路由到 Rails 应用程序的多个实例，并提供静态资源。

有关自动部署 Rails 应用程序，以及使用 Unicorn web 服务器和 NginX 的信息，请参阅[Unicorn + NginX + Capistrano Azure 虚拟机与][unicorn-nginx capistrano]。

如果您想要了解有关 Ruby on Rails 的详细信息，请访问[Ruby on Rails 指南][rails 指南]。

若要了解如何使用 Azure SDK for Ruby 从 Ruby 应用程序访问 Azure 服务，请参阅：

* [将非结构化的数据，使用 blob 存储][blob]

* [存储使用表的键/值对][表]

* [提供与内容传送网络的高带宽内容][cdn 如何]



<!-- WA.com links -->
[blob]: /zh-cn/documentation/articles/storage-ruby-how-to-use-blob-storage

[cdn 如何]: /zh-cn/develop/ruby/app-services/

[管理门户]: https://manage.windowsazure.cn/

[表]: /zh-cn/develop/ruby/how-to-guides/table-service/

[unicorn-nginx capistrano]: /zh-cn/documentation/articles/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/

[虚拟机说明]: /zh-cn/documentation/articles/virtual-machines-linux-tutorial


<!-- External Links -->
[rails 指南]: http://guides.rubyonrails.org/

[sqlite3]: http://www.sqlite.org/

<!-- Images -->
[blog-rails]: ./media/virtual-machines-ruby-rails-web-app-linux/blograilslocal.png

[blog-rails-cloud]: ./media/virtual-machines-ruby-rails-web-app-linux/blograilscloud.png 

[default-rails]: ./media/virtual-machines-ruby-rails-web-app-linux/basicrailslocal.png

[default-rails-cloud]: ./media/virtual-machines-ruby-rails-web-app-linux/basicrailscloud.png

[vmlist]: ./media/virtual-machines-ruby-rails-web-app-linux/vmlist.png

[endpoints]: ./media/virtual-machines-ruby-rails-web-app-linux/endpoints.png

[new-endpoint]: ./media/virtual-machines-ruby-rails-web-app-linux/newendpoint.png

