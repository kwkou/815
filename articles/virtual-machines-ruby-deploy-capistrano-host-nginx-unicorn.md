<properties linkid="dev-ruby-web-app-with-linux-vm-capistrano" urlDisplayName="Ruby on Rails Azure VM Capistrano" pageTitle="使用 Capistrano 将 Ruby on Rails Web 应用程序部署到 Azure 虚拟机 - 教程" metaKeywords="ruby on rails, ruby on rails azure, rails azure, rails vm, capistrano azure vm, capistrano azure rails, unicorn azure vm, unicorn azure rails, unicorn nginx capistrano, unicorn nginx capistrano azure, nginx azure" description="了解如何使用 Capistrano、Unicorn 和 Nginx 向 Azure 虚拟机部署 Ruby on Rails 应用程序。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" title="Deploy a Ruby on Rails Web application to an Azure VM using Capistrano" authors="larryfr" />


#使用 Capistrano 向 Azure VM 部署 Ruby on Rails Web 应用程序

本教程介绍如何将 Ruby on Rails 的网站部署到 Azure 虚拟机使用[Capistrano 3](https://github.com/capistrano/capistrano/)。部署完成后，您将使用[Nginx](http://nginx.org/)和[Unicorn](https://github.com/blog/517-unicorn)来承载网站。[PostgreSQL](https://www.postgresql.org)将存储的已部署的应用程序的应用程序数据。

本教程假定您之前未过使用 Azure 中，但假定您熟悉 Ruby、Rails、Git 和 Linux。完成本教程之后，您将能够在云中启动和运行基于 Ruby on Rails 的应用程序。

您将了解如何执行以下操作：

* 设置开发和生产环境

* 安装 Ruby 和 Ruby on Rails

* 安装 Unicorn、Nginx 和 PostgreSQL

* 新建 Rails 应用程序

* 创建 Capistrano 部署并使用它来部署应用程序

下面是已完成应用程序的屏幕快照：

![a browser displaying Listing Posts][rails 指南]

> [WACOM.NOTE] 本教程中使用的应用程序包括本机二进制组件。如果您的开发环境不是基于 Linux 的部署到 VM 时，可能会遇到错误。在部署过程中使用的 Gemfile.lock 文件将包含特定于平台的 gem，其中可能不包括虚拟机上所需的本机 Linux 版本的 gem 的条目。
> 
> 使用 Windows 开发环境需要特定的步骤。不过，如果本文未涵盖您在部署期间或之后遇到的错误，您不妨从基于 Linux 的开发环境重试本文中的步骤。

##本文内容

* [设置开发环境](#setup)

* [创建 Rails 应用程序](#create)

* [测试应用程序](#test)

* [创建一个源代码存储库](#repository)

* [创建 Azure 虚拟机](#createvm)

* [测试 Nginx](#nginx)

* [为部署做好准备](#capify)

* [部署](#deploy)

* [接下来的步骤](#next)

##<a id="setup"></a>设置开发环境

1. 在开发环境中安装 Ruby。具体步骤因操作系统而异。

	* **Apple OS X** -有多个有关 OS X 的 Ruby 版本。本教程使用在 OS X 上验证[Homebrew](http://brew.sh/)安装**rbenv**， **ruby 生成**，和**Ruby 2.0.0-p451**。在找不到安装信息[https://github.com/sstephenson/rbenv/](https://github.com/sstephenson/rbenv/)。

	* **Linux** -使用您分发的包管理系统。本教程已验证在 Ubuntu 12.10 上使用**rbenv**， **ruby 生成**，和**Ruby 2.0.0-p451**。

	* **Windows** -有多个针对 Windows 的 Ruby 版本。本教程使用验证[RubyInstaller](http://RubyInstaller.org/)安装**Ruby 2.0.0-p451**。使用发出命令**GitBash**命令行可用于[Git for Windows](http://git-scm.com/download/win)。

2. 打开一个新命令行或终端会话并输入以下命令以安装 Ruby on Rails：

		gem install rails --no-rdoc --no-ri

	> [WACOM.NOTE] 这可能需要管理员或在某些操作系统上的根权限。如果在运行此命令时收到错误，请尝试按如下所示使用"sudo"。
	> 
	> `sudo gem install rails`

	> [WACOM.NOTE] 本教程使用的是 Rails gem 版本 4.0.4。

3. 您还必须安装 JavaScript 解释程序，Rails 将使用它来编译您的 Rails 应用程序使用的 CoffeeScript 资产。在提供了支持的解释程序的列表[https://github.com/sstephenson/execjs#readme](https://github.com/sstephenson/execjs#readme)。
	
	> [WACOM.NOTE] [Node.js](http://nodejs.org/)用于本教程中，因为它是适用于 OS X、 Linux 和 Windows 操作系统。

##<a id="create"></a>创建 Rails 应用程序

1. 从命令行或终端会话中，使用以下命令创建一个名为"blog_app"的新 Rails 应用程序：

		rails new blog_app

	这将创建一个名为的新目录**blog_app**，文件和子目录 Rails 应用程序所需并用其填充它。

	> [WACOM.NOTE] 此命令可能需要一分钟或更长时间才能完成。它对默认应用程序所需的 gem 执行无提示安装，并且在此期间将无响应。

2. 将目录更改为**blog_app**目录中，然后使用下面的命令创建一个基本博客基架：

		rails generate scaffold Post name:string title:string content:text

	这将创建用于将文章保存到博客的控制器、视图、模型和数据库迁移。每篇文章都有一个作者姓名、文章标题和文本内容。

3. 若要创建将存储博客文章的数据库，请使用以下命令：

		rake db:migrate

	这将创建数据库架构用于存储对 Rails，使用默认数据库提供程序的帖子即[SQLite3 数据库][sqlite3]。

4. 若要显示为主页上的文章的索引，请修改**config/routes.rb**文件并添加后的进行如下`resources :posts`行。

		root 'posts#index'

	当用户访问该网站，这将显示一个 posts 列表。

##<a id="test"></a>测试应用程序

1. 将目录更改为**blog_app**目录 （如果尚不存在，并启动 rails 服务器使用以下命令。

		rails s

	您应该会看到与下面类似的输出。记下 Web 服务器正在侦听的端口。在下面的示例中，它正在侦听端口 3000。

		=> Booting WEBrick
		=> Rails 4.0.4 application starting in development on http://0.0.0.0:3000
		=> Call with -d to detach
		=> Ctrl-C to shutdown server
		[2013-03-12 19:11:31] INFO  WEBrick 1.3.1
		[2013-03-12 19:11:31] INFO  ruby 2.0.0 (2014-02-24) [x86_64-linux]
		[2013-03-12 19:11:31] INFO  WEBrick::HTTPServer#start: pid=9789 port=3000

2. 打开您的浏览器并导航到http://localhost:3000/ /。您应看到与下图类似的页面。

	![a page listing posts][博客 rails]

	若要结束服务器进程，请在命令行按 Ctrl+C

##<a id="repository"></a>创建源存储库

在部署使用 Capistrano 的应用程序时，其文件是从存储库中提取的。对于本教程中，我们将使用[Git](http://git-scm.com/)进行版本控制和[GitHub](https://github.com/)的存储库。

1.	在创建新的存储库[GitHub](https://github.com/)。如果您没有 GitHub 帐户，则可以免费注册一个帐户。下面的步骤假定存储库的名称是**blog_app**。

	> [WACOM.NOTE] 若要支持您的应用程序的自动的部署，应使用 SSH 密钥进行身份验证到 GitHub。详细信息，请参阅 GitHub 文档上[生成 SSH 密钥](https://help.github.com/articles/generating-ssh-keys)。

2.	从命令提示符下，将目录更改为**blog_app**目录，并运行以下命令将上载到 GitHub 存储库的应用程序。替换**YourGitHubName**替换为您的 GitHub 帐户名称。

		git init
		git add .
		git commit -m "initial commit on azure"
		git remote add origin git@github.com:YourGitHubName/blog-azure.git
		git push -u origin master

在下一部分中，您将创建此应用程序将部署到的虚拟机。

##<a id="createvm"></a>创建 Azure 虚拟机

请按照提供的说明[此处][vm 说明]创建托管 Linux 的 Azure 虚拟机。

1. 在登录到 Azure 的[管理门户][管理门户]。在命令栏中，选择**新建**。

2. 选择**虚拟机**，然后选择**从库**。

3. 从**选择一个图像**，选择**Ubuntu**，然后选择版本**12.04 LTS**。选择箭头以继续。

	> [WACOM.NOTE] 在本教程中的步骤被执行 Azure 虚拟机上承载 Ubuntu 12.04 LTS。如果您使用的是不同的 Linux 版本，可能需要执行不同的步骤才能完成相同的任务。

4. 在**虚拟机名称**键入您想要使用的名称。此名称将用于创建此虚拟机的域名。

5. 在**新用户名**，键入此计算机的管理员帐户的名称。

	> [WACOM.NOTE] 对于本教程，管理员帐户也将使用部署该应用程序。有关创建用于部署单独的帐户的信息，请参阅[Capistrano][capistrano]文档。

6. 在下**身份验证**，检查**上载兼容 SSH 密钥作身份验证**，然后浏览并选择**.pem**包含证书文件。最后，选择箭头以继续。

	> [WACOM.NOTE] 如果您不熟悉如何生成或使用 SSH 密钥，请参阅[如何通过在 Azure 上的 Linux 使用 SSH][ssh 上-azure]有关创建 SSH 密钥的说明。
	> 
	> 您还可以启用密码身份验证，但还必须提供 SSH 密钥，这是因为它可用于自动部署。

7. 在下**终结点**，使用**输入或选择一个值**下拉列表中选择**HTTP**。此页上的其他字段可保留为默认值。记下**云服务 DNS 名称**，如后面的步骤将使用此值。最后，选择箭头以继续。

8. 在最后一页上，选中复选标记以创建虚拟机。

###安装 Git、Ruby 和 Nginx

创建虚拟机后，使用 SSH 连接到该虚拟机，并使用以下命令为您的 Ruby 应用程序准备托管环境。

	sudo apt-get update
	sudo apt-get -y upgrade
	sudo apt-get -y install git build-essential libssl-dev curl nodejs nginx
	git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
	echo 'export RBENV_ROOT=~/.rbenv' >> ~/.bash_profile
	echo 'export PATH="$RBENV_ROOT/bin:$PATH"' >> ~/.bash_profile
	echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
	source ~/.bash_profile
	eval "$(rbenv init -)"
	git clone git://github.com/sstephenson/ruby-build.git
	cd ruby-build
	sudo ./install.sh
	~/.rbenv/bin/rbenv install 2.0.0-p451
	~/.rbenv/bin/rbenv global 2.0.0-p451
	gem install bundler
	~/.rbenv/bin/rbenv rehash

> [WACOM.NOTE] 您可能希望将上述保存到脚本 （.sh 文件），以避免键入错误时执行的命令。


> [WACOM.NOTE] **~/.Rbenv/bin/rbenv 安装 2.0.0-p451**命令可能需要几分钟才能完成。

**Rbenv install.sh**脚本将执行以下操作：
	
* 更新和升级当前已安装的程序包
* 安装生成工具
* 安装 Git
* 安装 Node.js
* 安装 Nginx
* 安装生成工具和 Ruby 和 Rails 所需的其他实用工具
* 设置的本地 （用户） 部署[rbenv]
* 安装 Ruby 2.0.0-p451
* 安装[捆绑包]gem

安装完成后，使用以下命令验证 Ruby 是否已成功安装：

	ruby -v

此操作应返回"ruby 2.0.0p451"的版本。

###安装 PostgreSQL

Rails 用于开发的默认数据库是 SQLite。通常，您将使用生产中的其他内容。以下步骤在虚拟机上安装 PostgreSQL，然后创建用户和数据库。后面的步骤会对 Rails 应用程序进行配置，以在部署过程中使用 PostgreSQL。

1. 使用以下命令安装 PostgreSQL 和开发位数。

		sudo apt-get -y install postgresql postgresql-contrib libpq-dev

6. 使用以下命令为您的登录创建用户和数据库。替换**my_username**和**my_database**替换为您的登录名的名称。例如，如果您的虚拟机的登录名为**部署**，替换**my_username**和**my_database**与**部署**。

		sudo -u postgres createuser -D -A -P my_username
		sudo -u postgres createdb -O my_username my_database

	> [WACOM.NOTE] 使用的数据库名称的用户名。对于此应用程序使用的 capistrano-postgresql gem，这是必需的。

	出现提示时，输入用户密码。当系统提示您允许用户创建新的角色，选择**y**，因为此用户将用于在部署过程中创建数据库和 Rails 应用程序将使用的登录名。

7. 若要验证您可以使用用户帐户连接到数据库，请使用以下命令。替换**my_username**和**my_database**与以前使用的值。

		psql -U my_username -W my_database

	应到达`database=>`提示符。若要退出 psql 实用程序，请输入`\q`在提示符下。

###<a id="nginx"></a>测试 Nginx

在虚拟机的创建过程中添加的 HTTP 终结点可使其在端口 80 上接受 HTTP 请求。若要验证这一点，请使用以下步骤以验证可以访问由 Nginx 创建的默认网站。

2.	从虚拟机的 SSH 会话中，启动 Nginx 服务：

		sudo service nginx start

	这将启动 Nginx 服务，该服务将侦听端口 80 上的传入流量。

6. 通过导航到虚拟机的 DNS 名称来测试您的应用程序。该网站应类似于以下：

	![nginx welcome page][nginx 欢迎]

	> [WACOM.NOTE] 稍后在本教程中使用的部署脚本将使 blog_app 成为 nginx 提供的默认网站。

此时，使用 Ruby、Nginx 和 PostgreSQL 的 Azure 虚拟机已准备好进行部署。在下一部分中，您将修改您的 Rails 应用程序，以添加执行部署的脚本和信息。

##<a id="capify"></a>准备部署

在开发环境中，修改应用程序以使用 Unicorn Web 服务器 和 PostgreSQL、为部署启用 Capistrano，并创建用于部署和启动该应用程序的脚本。

1. 在您的开发计算机上修改**Gemfile**并添加以下行以将 gem for Unicorn、 PostgreSQL、 Capistrano 和相关的 gem 添加到您的 Rails 应用程序

		# Use Unicorn
		gem 'unicorn'
		# Use PostgreSQL
		gem 'pg', group: :production

		group :development do
		  # Use Capistrano for deployment
		  gem 'capistrano', '~> 3.1'
		  gem 'capistrano-rails', '~> 1.1.1'
		  gem 'capistrano-bundler'
		  gem 'capistrano-rbenv', '~> 2.0'
		  gem 'capistrano-unicorn-nginx', '~> 2.0'
		  gem 'capistrano-postgresql', '~> 3.0'
		end

	> [WACOM.NOTE] Unicorn 在 Windows 上不可用。如果您使用 Windows 作为开发环境，修改 __Gemfile__ 以确保它将仅尝试安装 Unicorn 时通过使用以下指定 unicorn gem 时部署到虚拟机。
	> 
	> `platforms :ruby do`
	> `  gem 'unicorn'`
	> `end`

	大多数 capistraon-* gem 是使用特定的事物，我们使用的在生产服务器 (rbenv,) 或 framework (rails) 的帮助器。

	capistrano-unicorn-nginx gem 会在部署过长中自动创建与 Unicorn 和 Nginx 一起使用的脚本，因此我们无需手动创建这些脚本。capistrano-postgresql 将在您的应用程序的 PostgreSQL 中自动生成数据库、用户和密码。虽然您不一定要使用它们，但是它们都可使部署过程变得更加简单。
 
5.	使用以下命令安装新的 gem。
		
		bundle

6. 使用以下命令来创建 Capistrano 所使用的默认部署文件。

		cap install

	之后`cap install`命令完成后，以下文件和目录将具有已添加到您的应用程序。

		├── Capfile
		├── config
		│   ├── deploy
		│   │   ├── production.rb
		│   │   └── staging.rb
		│   └── deploy.rb
		└── lib
		    └── capistrano
		            └── tasks

	**Deploy.rb**文件提供了通用配置和操作对于所有类型的部署，而**production.rb**和**staging.rb**包含用于生产和过渡部署的其他配置。

	**Capistrano**文件夹包含任务和作为部署过程的一部分使用的其他文件。

5. 编辑**Capfile**根目录中的应用程序中，并取消注释以下行，通过从行的开头删除 __ #__ 字符。

		require 'capistrano/rbenv'
		require 'capistrano/bundler'
		require 'capistrano/rails/assets'
		require 'capistrano/rails/migrations'

	取消注释以前的行之后，添加以下行。

		require 'capistrano/unicorn_nginx'
		require 'capistrano/postgresql'

	这些行指示 Capistrano 使用之前添加到 Gemfile 的 gem。

	进行上述更改之后，保存该文件。

6.  编辑**config/deploy.rb**文件，并将该文件的内容替换为以下。替换**YourApplicationName**替换为您的应用程序和替换名称**https://github.com/YourGitHubName/YourRepoName.git**替换此项目的 GitHub 存储库的 URL。

		lock '3.1.0'
		# application name and the github repository
		set :application, 'YourApplicationName'
		set :repo_url, 'git@github.com:YourGitHubName/YourRepoName.git'
		
		# describe the rbenv environment we are deploying into
		set :rbenv_type, :user
		set :rbenv_ruby, '2.0.0-p451'
		set :rbenv_prefix, "RBENV_ROOT=#{fetch(:rbenv_path)} RBENV_VERSION=#{fetch(:rbenv_ruby)} #{fetch(:rbenv_path)}/bin/rbenv exec"
		set :rbenv_map_bins, %w{rake gem bundle ruby rails}
		
		# dirs we want symlinked to the shared folder
		# during deployment
		set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}
		
		namespace :deploy do

		  desc 'Restart application'
		  task :restart do
		    on roles(:app), in: :sequence, wait: 5 do
		      # Your restart mechanism here, for example:
		      # execute :touch, release_path.join('tmp/restart.txt')
              #
			  # The capistrano-unicorn-nginx gem handles all this
			  # for this example
		    end
		  end
		
		  after :publishing, :restart
		
		  after :restart, :clear_cache do
		    on roles(:web), in: :groups, limit: 3, wait: 10 do
		      # Here we can do anything such as:
		      # within release_path do
		      #   execute :rake, 'cache:clear'
		      # end
		    end
		  end
		
		end

	此文件提供所有部署类型所共有的通用信息和任务。

5.  编辑**config/deploy/production.rb**文件，并将现有内容替换为以下。替换**YourVm.cloudapp.net**与您的虚拟机的域名。替换**YourAzureVMUserName**与先前创建的 Azure VM 的登录帐户。

		# production deployment
		set :stage, :production
		# use the master branch of the repository
		set :branch, "master"
		# the user login on the remote server
		# used to connect and deploy
		set :deploy_user, "YourAzureVMUserName"
		# the 'full name' of the application
		set :full_app_name, "#{fetch(:application)}_#{fetch(:stage)}"
		# the server(s) to deploy to
		server 'YourVM.chinacloudapp.cn', user: 'YourAzureVMUserName', roles: %w{web app db}, primary: true
		# the path to deploy to
		set :deploy_to, "/home/#{fetch(:deploy_user)}/apps/#{fetch(:full_app_name)}"
        # set to production for Rails
		set :rails_env, :production

	此文件提供了特定于生产部署的信息。

8.	运行以下命令，以将更改提交到在上一步中修改的文件，然后将所做的更改上载到 GitHub。

		git add .
		git commit -m "adding config files"
		git push

该应用程序现在应该已为部署做好准备。

> [WACOM.NOTE] 对于更复杂的应用程序，或者对于不同的数据库或应用程序服务器，您可能需要其他配置或部署脚本。

##<a id="deploy"></a>部署

2.	从本地开发计算机中，使用以下命令部署到虚拟机应用程序所使用的配置文件。

		bundle exec cap production setup

	Capistrano 将使用 SSH 连接到虚拟机，然后创建该应用程序将部署到的目录 (~/apps)。如果这是在首次部署，capistrano postgresql gem 还将创建角色和数据库在 PostgreSQL 中在服务器上。它还将创建 Rails 将用于连接到数据库的 database.yml 配置文件。

	> [WACOM.NOTE] 如果您收到一条错误**从身份验证的套接字读取响应长度时出错**部署时，您可能需要在开发环境使用上启动 SSH 代理`ssh-agent`命令。例如，添加`eval $(ssh-agent)`到 ~/.bash\_profile 文件。
	> 
	> 您可能还需要将 SSH 密钥添加到代理缓存使用`ssh-add`命令。

4.	使用以下命令执行生产部署。这会将应用程序部署到虚拟机、启动 Unicorn 服务，并将 Nginx 配置为将流量路由到 Unicorn。

		bundle exec cap production deploy

	此命令会将应用程序部署到虚拟机、安装所需的 gem，然后启动/重新启动 Unicorn 和 Nginx。

	> [WACOM.NOTE] 此过程可能会在处理期间的几分钟内暂停。

	> [WACOM.NOTE] 部署的某些部分可能会返回退出状态 1 （失败）。只要部署成功完成，通常可忽略这些返回结果。

	> [WACOM.NOTE] 在某些系统中，您可能会遇到其中 SSH 代理不能向远程 VM 转发凭据，到 GitHub 进行身份验证时的情况。如果发生这种情况，您可以通过修改变通解决此错误**config/deploy.rb**文件并且更改`set :repo_url`行时要使用 HTTPS 访问 Github。在使用 HTTPS 时，您必须指定作为 URL 的一部分的 GitHub 用户名和密码（或身份验证令牌）。例如：
	> 
	> `set :repo_url, 'https://you:yourpassword@github.com/You/yourrepository.git'
	> 
	> 虽然这应允许您绕过该错误并完成本教程中，这不推荐使用此解决方案对于生产部署，因为它将身份验证凭据以纯文本格式存储为应用程序的一部分。您在使用 SSH 代理进行转发时应该参考适用于您的操作系统的文档

此时，您的 Ruby on Rails 应用程序应该已经在 Azure 虚拟机上运行。为了验证这一点，请在 Web 浏览器中输入虚拟机的 DNS 名称。例如， http://railsvm.chinacloudapp.cn. 应显示文章索引，您应该能够创建、 编辑和删除的帖子。

##<a id="next"></a>后续步骤

在本文中，您学习了如何使用 Capistrano 创建基本 Rails 应用程序并将其发布到 Azure 虚拟机。使用基本的应用程序（如本文中的应用程序）只触及到了使用 Capistrano 进行部署时可实现的操作的皮毛。有关使用 Capistrano 的详细信息，请参阅：

* [Capistranorb.com](http://capistranorb.com) -Capistrano 站点。
* [Azure，Ruby on Rails、 Capistrano 3 和 PostgreSQL](http://wootstudio.ca/articles/tutorial-windows-azure-ruby-on-rails-capistrano-3-postgresql) -一种方法来将部署到 Azure 涉及自定义部署脚本。
* [Capistrano 3 教程](http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/) -使用 Capistrano 3 的教程。

有关创建和部署到 Azure VM 仅使用 SSH 的 Rails 应用程序的更基本示例，请参阅[托管 Ruby on Rails Web 应用程序中使用 Linux 虚拟机][ruby vm]。

如果您想要了解有关 Ruby on Rails 的详细信息，请访问[Ruby on Rails 指南][rails 指南]。

若要了解如何使用 Azure SDK for Ruby 从 Ruby 应用程序访问 Azure 服务，请参阅：

* [将非结构化的数据，使用 blob 存储][blob]

* [存储使用表的键/值对][表]

* [提供与内容传送网络的高带宽内容][cdn 如何]

[虚拟机说明]: /zh-cn/documentation/articles/virtual-machines-linux-tutorial/


[rails 指南]: http://guides.rubyonrails.org/
[blob]: /zh-cn/develop/ruby/how-to-guides/blob-storage/
[表]: /zh-cn/develop/ruby/how-to-guides/table-service/
[cdn 如何]: /zh-cn/develop/ruby/app-services/
[ruby 虚拟机]: /zh-cn/develop/ruby/tutorials/web-app-with-linux-vm/
 
[博客 rails]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/blograilslocal.png
[博客 rails 云]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/blograilscloud.png 
[默认 rails]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/basicrailslocal.png
[默认 rails 云]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/basicrailscloud.png
[vmlist]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/vmlist.png
[终结点]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/endpoints.png
[新建终结点]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/newendpoint80.png
[nginx 欢迎]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/welcomenginx.png

[管理门户]: https://manage.windowsazure.cn/
[sqlite3]: http://www.sqlite.org/
[ssh-on-azure]: /zh-cn/documentation/articles/linux-use-ssh-key/
[capistrano]: http://capistranorb.com
