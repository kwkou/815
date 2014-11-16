<properties linkid="dev-ruby-web-app-with-linux-vm" urlDisplayName="Ruby on Rails Web App on Azure using Linux VM" pageTitle="Ruby on Rails Web App on Azure using Linux VM" metaKeywords="Azure Ruby web application, Azure Ruby application, Ruby app Azure, Ruby azure vm, ruby virthal machine, ruby linux vm" description="Host a Ruby on Rails-based web site on Azure using a Linux virtual machine. " metaCanonical="" services="virtual-machines" documentationCenter="Ruby" title="Ruby on Rails Web application on an Azure VM" authors="larryfr" solutions="" manager="" editor="" />

# Azure 虚拟机上的 Ruby on Rails Web 应用程序

本教程介绍如何在 Azure 中使用 Linux 虚拟机托管基于 Ruby on Rails 的网站。本教程假定你之前未使用过 Azure。完成本教程之后，你将能够在云中启动和运行基于 Ruby on Rails 的应用程序。

你将了解如何执行以下操作：

-   设置开发环境

-   设置 Azure 虚拟机以托管 Ruby on Rails。

-   新建 Rails 应用程序

-   使用 scp 将文件复制到虚拟机

下面是已完成应用程序的屏幕快照：

![显示 Listing Posts 的浏览器窗口][显示 Listing Posts 的浏览器窗口]

## 本文内容

-   [设置开发环境][设置开发环境]

-   [创建 Rails 应用程序][创建 Rails 应用程序]

-   [测试应用程序][测试应用程序]

-   [创建 Azure 虚拟机][创建 Azure 虚拟机]

-   [将应用程序复制到虚拟机][将应用程序复制到虚拟机]

-   [安装 gem 并启动应用程序][安装 gem 并启动应用程序]

-   [后续步骤][后续步骤]

## <span id="setup"></span></a>设置开发环境

1.  在开发环境中安装 Ruby。具体步骤因操作系统而异。

    -   **Apple OS X** - 有多个适用于 OS X 的 Ruby 版本。本教程通过使用 [Homebrew][Homebrew] 安装 **rbenv** 和 **ruby-build** 在 OS X 上进行了验证。可在 [][]<https://github.com/sstephenson/rbenv/></a> 中找到安装信息。

    -   **Linux** - 使用版本程序包管理系统。本教程使用 ruby1.9.1 和 ruby1.9.1-dev 程序包在 Ubuntu 12.10 上进行了验证。

    -   **Windows** - 有多个适用于 Windows 的 Ruby 版本。本教程使用 [RailsInstaller][RailsInstaller] 1.9.3-p392 进行了验证。

2.  打开一个新命令行或终端会话并输入以下命令以安装 Ruby on Rails：

        gem install rails --no-rdoc --no-ri

    <div class="dev-callout">
<strong>说明</strong>
<p>在有些操作系统上，此命令可能需要管理员或 root 权限。如果在运行此命令时收到错误，请尝试按如下所示使用&ldquo;sudo&rdquo;：</p>
<pre class="prettyprint">sudo gem install rails</pre>
</div>

    <div class="dev-callout">
    <strong>说明</strong>
    <p>本教程使用的是 Rails gem 版本 3.2.12。</p>

    </div>

3.  你还必须安装 JavaScript 解释程序，Rails 将使用它来编译你的 Rails 应用程序使用的 CoffeeScript 资产。[][1]<https://github.com/sstephenson/execjs#readme></a> 中提供了支持的解释程序列表。

    在验证本教程时，使用了 [Node.js][Node.js]，因为它适用于 OS X、Linux 和 Windows 操作系统。

## <span id="create"></span></a>创建 Rails 应用程序

1.  从命令行或终端会话中，使用以下命令创建一个名为“blog\_app”的新 Rails 应用程序：

        rails new blog_app

    此命令将新建一个名为 **blog\_app** 的目录，并使用 Rails 应用程序所需的文件和子目录填充它。

    <div class="dev-callout">
<strong>说明</strong>
<p>此命令可能需要一分钟或更长时间才能完成。它以静默方式安装默认应用程序所需的 gem，在此时间段，该命令看起来似乎已挂起。</p>
</div>

2.  将目录更改为 **blog\_app** 目录，然后使用以下命令创建一个基本博客基架：

        rails generate scaffold Post name:string title:string content:text

    这将创建用于将文章保存到博客的控制器、视图、模型和数据库迁移。每篇文章都有一个作者姓名、文章标题和文本内容。

3.  若要创建将存储博客文章的数据库，请使用以下命令：

        rake db:migrate

    这将对 Rails 使用默认数据库提供程序，即 [SQLite3 数据库][SQLite3 数据库]。虽然你可能希望对生产应用程序使用不同的数据库，但 SQLite 对于本教程而言已足够。

## <span id="test"></span></a>测试应用程序

执行以下步骤在你的开发环境中启动 Rails 服务器

1.  从命令行或终端会话中，使用以下命令启动 Rails 服务器：

        rails s

    你应该会看到与下面类似的输出。记下 Web 服务器正在侦听的端口。在下面的示例中，它正在侦听端口 3000。

        => Booting WEBrick
        => Rails 3.2.12 application starting in development on http://0.0.0.0:3000
        => Call with -d to detach
        => Ctrl-C to shutdown server
        [2013-03-12 19:11:31] INFO  WEBrick 1.3.1
        [2013-03-12 19:11:31] INFO  ruby 1.9.3 (2012-04-20) [x86_64-linux]
        [2013-03-12 19:11:31] INFO  WEBrick::HTTPServer#start: pid=9789 port=3000

2.  打开浏览器并导航到 <http://localhost:3000/>。你应看到与下图类似的页面：

    ![默认 rails 页面][默认 rails 页面]

    该页面是一个静态欢迎页面。若要查看基架命令生成的表单，请导航到 <http://localhost:3000/posts>。你应看到与下图类似的页面：

    ![列出文章的页面][列出文章的页面]

    若要结束服务器进程，请在命令行按 Ctrl+C

## <span id="createvm"></span></a>创建 Azure 虚拟机

按照[此处][此处]提供的说明可创建托管 Linux 的 Azure 虚拟机。

<div class="dev-callout">
<strong>说明</strong>

<p>本教程中的步骤是在托管 Ubuntu 12.10 的 Azure 虚拟机上执行的。如果你使用的是不同的 Linux 版本，可能需要执行不同的步骤才能完成相同的任务。</p></div>

<div class="dev-callout">
<strong>说明</strong>
<p>你<strong>只</strong>需创建虚拟机。在了解了如何使用 SSH 连接到虚拟机后停止。</p>
</div>

创建 Azure 虚拟机后，执行以下步骤在该虚拟机上安装 Ruby 和 Rails：

1.  从命令行或终端会话中，使用以下命令通过 SSH 连接到虚拟机：

        ssh username@vmdns -p port

    替换在创建虚拟机的过程中指定的用户名、虚拟机的 DNS 地址和 SSH 终结点的端口。例如：

        ssh railsdev@railsvm.chinacloudapp.cn -p 61830

    <div class="dev-callout">
<strong>说明</strong>
<p>如果使用 Windows 作为开发环境，则可以使用 <b>PuTTY</b> 之类的实用工具实现 SSH 功能。可从 <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html">PuTTY 下载页</a>获取 PuTTY。</p>
</div>

2.  从 SSH 会话中，使用以下命令在虚拟机上安装 Ruby：

        sudo apt-get update -y
        sudo apt-get upgrade -y
        sudo apt-get install ruby1.9.1 ruby1.9.l-dev build-essential libsqlite3-dev nodejs -y

    安装完成后，使用以下命令验证 Ruby 是否已成功安装：

        ruby -v

    这应该会返回虚拟机上安装的 Ruby 版本，它可能不是 1.9.1。例如 **ruby 1.9.3p194 (2012-04-20 修订版 35410) [x86\_64-linux]**。

3.  使用以下命令安装捆绑包：

        sudo gem install bundler --no-rdoc --no-ri

    捆绑包将用于安装 rails 应用程序被复制到服务器后所需的 gem。

## <span id="copy"></span></a>将应用程序复制到虚拟机

从开发环境中，打开一个新命令行或终端会话，并使用 **scp** 命令将 **blog\_app** 目录复制到虚拟机。此命令的格式如下所示：

    scp -r -P 54822 -C directory-to-copy user@vmdns:

例如：

    scp -r -P 54822 -C ~/blog_app railsdev@railsvm.chinacloudapp.cn:

<div class="dev-callout">
<strong>说明</strong>
<p>如果使用 Windows 作为开发环境，则可以使用 <b>pscp</b> 之类的实用工具实现 scp 功能。可从 <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html">PuTTY 下载页</a>获取 Pscp。</p>
</div>

用于此命令的参数具有以下效果：

-   **-r**：以递归方式复制指定目录和所有子目录中的内容

-   **-P**：使用指定的端口进行 SSH 通信

-   **-C**：启用压缩

-   **directory-to-copy**：要复制的本地目录

-   <**user@vmdns**>:要将文件复制到其中的计算机的地址以及用于登录的用户帐户

复制操作完成后，**blog\_app** 目录将位于用户的主目录中。在 SSH 会话中对虚拟机使用以下命令来查看已复制的文件：

    cd ~/blog_app
    ls

返回的文件列表应该与开发环境中 **blog\_app** 目录包含的文件相匹配。

## <span id="start"></span></a>安装 gem 并启动 Rails

1.  在虚拟机上，将目录更改为 **blog\_app** 目录，并使用以下命令安装 **Gemfile** 中指定的 gem：

        sudo bundle install

2.  若要创建数据库，请使用以下命令：

        rake db:migrate

3.  使用以下命令启动服务器：

        rails s

    你应该会看到与下面类似的输出。记下 Web 服务器正在侦听的端口。在下面的示例中，它正在侦听端口 3000。

        => Booting WEBrick
        => Rails 3.2.12 application starting in development on http://0.0.0.0:3000
        => Call with -d to detach
        => Ctrl-C to shutdown server
        [2013-03-12 19:11:31] INFO  WEBrick 1.3.1
        [2013-03-12 19:11:31] INFO  ruby 1.9.3 (2012-04-20) [x86_64-linux]
        [2013-03-12 19:11:31] INFO  WEBrick::HTTPServer#start: pid=9789 port=3000

4.  在浏览器中，导航到 [Azure 管理门户][Azure 管理门户]并选择你的虚拟机。

    ![虚拟机列表][虚拟机列表]

5.  选择页面顶部的“终结点”，然后单击页面底部的“+添加终结点”。

    ![终结点页面][终结点页面]

6.  在“添加终结点”对话框中，单击左下角的箭头以继续进行第二页，然后在表单中输入以下信息：

    -   **名称**：HTTP

    -   **协议**：TCP

    -   **公用端口**： 80

    -   **专用端口**：\<来自上述第 3 步的端口信息\>

    这将创建一个公用端口 80，以便将流量路由到专用端口 3000，即 Rails 服务器侦听的端口。

    ![新建终结点对话框][新建终结点对话框]

7.  单击复选标记保存该终结点。

8.  应该会出现一条消息，指出“正在进行更新”。此消息消失后，终结点即处于活动状态。现在你可以通过导航到虚拟机的 DNS 名称来测试你的应用程序。网站应类似下面这样：

    ![默认 rails 页面][2]

    向 URL 中追加 **/posts** 应该会显示基架命令生成的页面。

    ![文章页面][显示 Listing Posts 的浏览器窗口]

## <span id="next"></span></a>后续步骤

在本文中，你学习了如何创建基于表单的基本 Rails 应用程序并将其发布到 Azure 虚拟机。我们执行的大部分操作都是手动的，在生产环境中，可以自动完成这些操作。此外，大多数生产环境都结合其他服务器进程（如 Apache 或 NginX）来托管 Rails 应用程序，这些进程会处理路由到多个 Rails 应用程序实例的请求并提供静态资源。

有关自动部署 Rails 应用程序以及使用 Unicorn Web 服务器和 NginX 的信息，请参阅[在 Azure 虚拟机中使用 Unicorn+NginX+Capistrano][在 Azure 虚拟机中使用 Unicorn+NginX+Capistrano]。

如果想要详细了解 Ruby on Rails，请访问 [Ruby on Rails 指南][Ruby on Rails 指南]。

若要了解如何使用 Azure SDK for Ruby 从 Ruby 应用程序访问 Azure 服务，请参阅：

-   [使用 Blob 存储非结构化数据][使用 Blob 存储非结构化数据]

-   [使用表存储键/值对][使用表存储键/值对]

-   [使用内容交付网络提供高带宽内容][使用内容交付网络提供高带宽内容]

<!-- WA.com links --> 
<!-- External Links --> 
<!-- Images -->

  [显示 Listing Posts 的浏览器窗口]: ./media/virtual-machines-ruby-rails-web-app-linux/blograilscloud.png
  [设置开发环境]: #setup
  [创建 Rails 应用程序]: #create
  [测试应用程序]: #test
  [创建 Azure 虚拟机]: #createvm
  [将应用程序复制到虚拟机]: #copy
  [安装 gem 并启动应用程序]: #start
  [后续步骤]: #next
  [Homebrew]: http://brew.sh/
  []: https://github.com/sstephenson/rbenv/
  [RailsInstaller]: http://railsinstaller.org/
  [1]: https://github.com/sstephenson/execjs#readme
  [Node.js]: http://nodejs.org/
  [SQLite3 数据库]: http://www.sqlite.org/
  [默认 rails 页面]: ./media/virtual-machines-ruby-rails-web-app-linux/basicrailslocal.png
  [列出文章的页面]: ./media/virtual-machines-ruby-rails-web-app-linux/blograilslocal.png
  [此处]: /zh-cn/documentation/articles/virtual-machines-linux-tutorial
  [PuTTY 下载页]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [虚拟机列表]: ./media/virtual-machines-ruby-rails-web-app-linux/vmlist.png
  [终结点页面]: ./media/virtual-machines-ruby-rails-web-app-linux/endpoints.png
  [新建终结点对话框]: ./media/virtual-machines-ruby-rails-web-app-linux/newendpoint.png
  [2]: ./media/virtual-machines-ruby-rails-web-app-linux/basicrailscloud.png
  [在 Azure 虚拟机中使用 Unicorn+NginX+Capistrano]: /zh-cn/documentation/articles/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/
  [Ruby on Rails 指南]: http://guides.rubyonrails.org/
  [使用 Blob 存储非结构化数据]: /zh-cn/documentation/articles/storage-ruby-how-to-use-blob-storage
  [使用表存储键/值对]: /en-us/develop/ruby/how-to-guides/table-service/
  [使用内容交付网络提供高带宽内容]: /en-us/develop/ruby/app-services/
