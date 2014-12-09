<properties linkid="dev-ruby-web-app-with-linux-vm-capistrano" urlDisplayName="Ruby on Rails Azure VM Capistrano" pageTitle="使用 Capistrano 将 Ruby on Rails Web 应用程序部署到 Azure 虚拟机 &ndash; 教程" metaKeywords="ruby on rails, ruby on rails azure, rails azure, rails vm, capistrano azure vm, capistrano azure rails, unicorn azure vm, unicorn azure rails, unicorn nginx capistrano, unicorn nginx capistrano azure, nginx azure" description="了解如何使用 Capistrano、Unicorn 和 Nginx 向 Azure 虚拟机部署 Ruby on Rails 应用程序。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" title="使用 Capistrano 向 Azure VM 部署 Ruby on Rails Web 应用程序" authors="larryfr" />

# 使用 Capistrano 向 Azure VM 部署 Ruby on Rails Web 应用程序

本教程介绍如何使用 [Capistrano 3][Capistrano 3] 将 Ruby on Rails 网站部署到 Azure 虚拟机中。部署完成后，您将使用 [Nginx][Nginx] 和 [Unicorn][Unicorn] 来托管该网站。[PostgreSQL][PostgreSQL] 将存储已部署的应用程序的应用程序数据。

本教程假定您之前未使用过 Azure，但假定您熟悉 Ruby、Rails、Git 和 Linux。完成本教程之后，您将能够在云中启动和运行基于 Ruby on Rails 的应用程序。

您将了解如何执行以下操作：

-   设置开发和生产环境

-   安装 Ruby 和 Ruby on Rails

-   安装 Unicorn、Nginx 和 PostgreSQL

-   新建 Rails 应用程序

-   创建 Capistrano 部署并使用它来部署应用程序

下面是已完成应用程序的屏幕快照：

![显示 Listing Posts 的浏览器窗口][显示 Listing Posts 的浏览器窗口]

> [WACOM.NOTE] 本教程使用的应用程序包括本机二进制组件。如果您的开发环境不是基于 Linux 的环境，则部署到虚拟机时您可能会遇到错误。在部署过程中使用的 Gemfile.lock 文件将包含特定于平台的 gem，其中可能不包括虚拟机上所需的本机 Linux 版本的 gem 的条目。
>
> 使用 Windows 开发环境需要特定的步骤。不过，如果本文未涵盖您在部署期间或之后遇到的错误，您不妨从基于 Linux 的开发环境重试本文中的步骤。

## 本文内容

-   [设置开发环境][设置开发环境]

-   [创建 Rails 应用程序][创建 Rails 应用程序]

-   [测试应用程序][测试应用程序]

-   [创建源存储库][创建源存储库]

-   [创建 Azure 虚拟机][创建 Azure 虚拟机]

-   [测试 Nginx][测试 Nginx]

-   [准备部署][准备部署]

-   [部署][部署]

-   [后续步骤][后续步骤]

## <span id="setup"></span></a>设置开发环境

1.  在开发环境中安装 Ruby。具体步骤因操作系统而异。

    -   **Apple OS X** – 有多个适用于 OS X 的 Ruby 版本。本教程通过使用 [Homebrew][Homebrew] 安装 **rbenv**、**ruby-build** 和 **Ruby 2.0.0-p451** 在 OS X 上进行了验证。可在 <https://github.com/sstephenson/rbenv/> 中找到安装信息。

    -   **Linux** - 使用版本程序包管理系统。本教程使用 **rbenv**、**ruby-build** 和 **Ruby 2.0.0-p451** 在 Ubuntu 12.10 上进行了验证。

    -   **Windows** - 有多个适用于 Windows 的 Ruby 版本。本教程通过使用 [RubyInstaller][RubyInstaller] 安装 **Ruby 2.0.0-p451** 进行了验证。命令是使用 [Git for Windows][Git for Windows] 提供的 **GitBash** 命令行发出的。

2.  打开一个新命令行或终端会话并输入以下命令以安装 Ruby on Rails：

        gem install rails --no-rdoc --no-ri

    > [WACOM.NOTE] 在有些操作系统上，这可能需要管理员或 root 权限。如果在运行此命令时收到错误，请尝试按如下所示使用“sudo”。
    >
    > `sudo gem install rails`

    > [WACOM.NOTE] 本教程使用的是 Rails gem 版本 4.0.4。

3.  您还必须安装 JavaScript 解释程序，Rails 将使用它来编译您的 Rails 应用程序使用的 CoffeeScript 资产。<https://github.com/sstephenson/execjs#readme> 中提供了支持的解释程序列表。

    > [WACOM.NOTE] 本教程使用了 [Node.js]](http://nodejs.org/)，这是因为它可用于 OS X、Linux 和 Windows 操作系统。

## <span id="create"></span></a>创建 Rails 应用程序

1.  从命令行或终端会话中，使用以下命令创建一个名为“blog\_app”的新 Rails 应用程序：

        rails new blog_app

    这将新建一个名为 **blog\_app** 的目录，并使用 Rails 应用程序所需的文件和子目录填充它。

    > [WACOM.NOTE] 此命令可能需要一分钟或更长时间才能完成。它对默认应用程序所需的 gem 执行无提示安装，并且在此期间将无响应。

2.  将目录更改为 **blog\_app** 目录，然后使用以下命令创建一个基本博客基架：

        rails generate scaffold Post name:string title:string content:text

    这将创建用于将文章保存到博客的控制器、视图、模型和数据库迁移。每篇文章都有一个作者姓名、文章标题和文本内容。

3.  若要创建将存储博客文章的数据库，请使用以下命令：

        rake db:migrate

    这将通过对 Rails 使用默认数据库提供程序（即 [SQLite3 数据库][SQLite3 数据库]），来创建用于存储文章的数据库架构。

4.  若要将文章索引显示为主页，请修改 **config/routes.rb** 文件并在 `resources :posts` 行后添加以下内容。

        root 'posts#index'

    这将在用户访问该网站时显示一个文章列表。

## <span id="test"></span></a>测试应用程序

1.  如果您尚未位于 **blog\_app** 目录，则将目录更改为该目录，然后使用以下命令启动 rails 服务器。

        rails s

    您应该会看到与下面类似的输出。记下 Web 服务器正在侦听的端口。在下面的示例中，它正在侦听端口 3000。

        => Booting WEBrick
        => Rails 4.0.4 application starting in development on http://0.0.0.0:3000
        => Call with -d to detach
        => Ctrl-C to shutdown server
        [2013-03-12 19:11:31] INFO  WEBrick 1.3.1
        [2013-03-12 19:11:31] INFO  ruby 2.0.0 (2014-02-24) [x86_64-linux]
        [2013-03-12 19:11:31] INFO  WEBrick::HTTPServer#start: pid=9789 port=3000

2.  打开浏览器并导航到 http://localhost:3000/。您应看到与下图类似的页面。

    ![列出文章的页面][列出文章的页面]

    若要结束服务器进程，请在命令行按 Ctrl+C

## <span id="repository"></span></a>创建源存储库

在部署使用 Capistrano 的应用程序时，其文件是从存储库中提取的。对于本教程，我们会将 [Git][Git] 用于版本控制，将 [GitHub][GitHub] 用于存储库。

1.  在 [GitHub][GitHub] 上创建一个新存储库。如果您没有 GitHub 帐户，则可以免费注册一个帐户。下面的步骤假定存储库的名称是 **blog\_app**。

    > [WACOM.NOTE] 若要支持应用程序的自动部署，应使用 SSH 密钥对 GitHub 进行身份验证。有关详细信息，请参阅[生成 SSH 密钥][生成 SSH 密钥]上的 GitHub 文档。

2.  从命令提示符将目录更改为 **blog\_app** 目录，并运行以下命令将应用程序上载到 GitHub 存储库。将 **YourGitHubName** 替换为 GitHub 帐户的名称。

        git init
        git add .
        git commit -m "initial commit on azure"
        git remote add origin git@github.com:YourGitHubName/blog-azure.git
        git push -u origin master

在下一部分中，您将创建此应用程序将部署到的虚拟机。

## <span id="createvm"></span></a>创建 Azure 虚拟机

按照[此处][此处]提供的说明可创建托管 Linux 的 Azure 虚拟机。

1.  登录到 Azure [管理门户][管理门户]。在命令栏上，选择“新建”。

2.  选择“虚拟机”，然后选择“从库中”。

3.  从“选择映像”中选择“Ubuntu”，然后选择版本“12.04 LTS”。选择箭头以继续。

    > [WACOM.NOTE] 本教程中的步骤是在托管 Ubuntu 12.04 LTS 的 Azure 虚拟机上执行的。如果您使用的是不同的 Linux 版本，可能需要执行不同的步骤才能完成相同的任务。

4.  在“虚拟机名称”中，键入您想要使用的名称。此名称将用于创建此虚拟机的域名。

5.  在“新用户名”中，键入此计算机的管理员帐户的名称。

    > [WACOM.NOTE] 本教程中，管理员帐户也将用于部署应用程序。有关创建用于部署的单独帐户的信息，请参阅 [Capistrano][Capistrano] 文档。

6.  在“身份验证”下，检查“上载兼容 SSH 密钥以用于身份验证”，然后浏览并选择包含证书的“.pem”文件。最后，选择箭头以继续。

    > [WACOM.NOTE] 如果您不熟悉如何生成或使用 SSH 密钥，请参阅[如何在 Azure 上通过 Linux 使用 SSH][如何在 Azure 上通过 Linux 使用 SSH]，获取创建 SSH 密钥的说明。
    >
    > 您还可以启用密码身份验证，但还必须提供 SSH 密钥，这是因为它可用于自动部署。

7.  在“终结点”下，使用“输入或选择一个值”下拉列表来选择“HTTP”。此页上的其他字段可保留为默认值。记下“云服务 DNS 名称”，因为后面的步骤将使用此值。最后，选择箭头以继续。

8.  在最后一页上，选中复选标记以创建虚拟机。

### 安装 Git、Ruby 和 Nginx

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

> [WACOM.NOTE] 您不妨将上述内容保存到脚本（.sh 文件）中，以避免执行命令时发生拼写错误。

> [WACOM.NOTE] **~/.rbenv/bin/rbenv install 2.0.0-p451** 命令可能需要几分钟才能完成。

**rbenv-install.sh** 脚本将执行以下操作：

-   更新和升级当前已安装的程序包
-   安装生成工具
-   安装 Git
-   安装 Node.js
-   安装 Nginx
-   安装生成工具和 Ruby 和 Rails 所需的其他实用工具
-   设置 [rbenv] 的本地（用户） 部署
-   安装 Ruby 2.0.0-p451
-   安装 [Bundler] gem

安装完成后，使用以下命令验证 Ruby 是否已成功安装：

    ruby -v

此命令应返回版本 `ruby 2.0.0p451`。

### 安装 PostgreSQL

Rails 用于开发的默认数据库是 SQLite。通常，您会在生产中使用其他数据库。以下步骤在虚拟机上安装 PostgreSQL，然后创建用户和数据库。后面的步骤会对 Rails 应用程序进行配置，以在部署过程中使用 PostgreSQL。

1.  使用以下命令安装 PostgreSQL 和开发位数。

        sudo apt-get -y install postgresql postgresql-contrib libpq-dev

2.  使用以下命令为您的登录创建用户和数据库。将 **my\_username** 和 **my\_database** 替换为您的登录名。例如，如果您的虚拟机的登录名为 **deploy**，则将 **my\_username** 和 **my\_database** 替换为 **deploy**。

        sudo -u postgres createuser -D -A -P my_username
        sudo -u postgres createdb -O my_username my_database

    > [WACOM.NOTE] 此外，将用户名用于数据库名。对于此应用程序使用的 capistrano-postgresql gem，这是必需的。

    出现提示时，输入用户密码。当系统提示您允许用户创建新的角色时，请选择“y”，这是因为在部署期间将使用此用户来创建 Rails 应用程序将使用的数据库和登录名。

3.  若要验证您可以使用用户帐户连接到数据库，请使用以下命令。将 **my\_username** 和 **my\_database** 替换为以前使用的值。

        psql -U my_username -W my_database

    您应到达 `database=>` 提示符。若要退出 psql 实用程序，请在提示符下输入 `\q`。

### <span id="nginx"></span></a>测试 Nginx

在虚拟机的创建过程中添加的 HTTP 终结点允许它接受端口 80 上的 HTTP 请求。若要验证这一点，请使用以下步骤以验证您可以访问 Nginx 所创建的默认站点。

1.  从虚拟机的 SSH 会话中，启动 Nginx 服务：

        sudo service nginx start

    这将启动 Nginx 服务，该服务将侦听端口 80 上的传入流量。

2.  通过导航到虚拟机的 DNS 名称来测试您的应用程序。网站应类似下面这样：

    ![nginx 欢迎页][nginx 欢迎页]

    > [WACOM.NOTE] 本教程中稍后使用的部署脚本将使 blog\_app 成为 Nginx 提供的默认网站。

此时，使用 Ruby、Nginx 和 PostgreSQL 的 Azure 虚拟机已准备好进行部署。在下一部分中，您将修改您的 Rails 应用程序，以添加执行部署的脚本和信息。

## <span id="capify"></span></a>准备部署

在开发环境中，修改应用程序以使用 Unicorn Web 服务器 和 PostgreSQL、为部署启用 Capistrano，并创建用于部署和启动该应用程序的脚本。

1.  在您的开发计算机上修改 **Gemfile** 并添加以下行，以将用于 Unicorn、PostgreSQL、Capistrano 的 gem 和相关的 gem 添加到您的 Rails 应用程序

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

    > [WACOM.NOTE] Unicorn 在 Windows 上不可用。如果您使用 Windows 作为开发环境，请通过在指定 unicorn gem 时使用以下命令来修改 **Gemfile**，以确保它只在部署到虚拟机时才尝试安装 Unicorn。
    >
    > `platforms :ruby do`
    > `gem 'unicorn'`
    > `end`

    大多数 capistraon-\* gem 是帮助程序，可与我们要在生产服务器 (rbenv ) 或框架 (rails) 上使用的特定程序一起使用。

    capistrano-unicorn-nginx gem 会在部署过长中自动创建与 Unicorn 和 Nginx 一起使用的脚本，因此我们无需手动创建这些脚本。capistrano-postgresql 将在您的应用程序的 PostgreSQL 中自动生成数据库、用户和密码。虽然您不一定要使用它们，但是它们都可使部署过程变得更加简单。

2.  使用以下命令安装新的 gem。

        bundle

3.  使用以下命令来创建 Capistrano 所使用的默认部署文件。

        cap install

    `cap install` 命令完成后，将向您的应用程序中添加以下文件和目录。

        ├── Capfile
        ├── config
        │   ├── deploy
        │   │   ├── production.rb
        │   │   └── staging.rb
        │   └── deploy.rb
        └── lib
            └── capistrano
                    └── tasks

    **Deploy.rb** 文件提供了用于所有类型的部署的通用配置和操作，而 **production.rb** 和 **staging.rb** 包含用于生产和过渡部署的其他配置。

    **capistrano** 文件夹包含任务和用作部署过程的一部分的其他文件。

4.  编辑应用程序的根目录中的 **Capfile**，并通过删除行开头的 **\#** 字符来取消注释以下行。

        require 'capistrano/rbenv'
        require 'capistrano/bundler'
        require 'capistrano/rails/assets'
        require 'capistrano/rails/migrations'

    取消注释以前的行之后，添加以下行。

        require 'capistrano/unicorn_nginx'
        require 'capistrano/postgresql'

    这些行指示 Capistrano 使用之前添加到 Gemfile 的 gem。

    进行上述更改之后，保存该文件。

5.  编辑 **config/deploy.rb** 文件，并将该文件的内容替换为以下内容。将 **YourApplicationName** 替换为您的应用程序的名称，并将 **https://github.com/YourGitHubName/YourRepoName.git** 替换为此项目的 GitHub 存储库的 URL。

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

6.  编辑 **config/deploy/production.rb** 文件，并将现有内容替换为以下内容。将 **YourVm.cloudapp.net** 替换为您的虚拟机的域名。将 **YourAzureVMUserName** 替换为以前为 Azure 虚拟机创建的登录帐户。

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

7.  运行以下命令，以将更改提交到在上一步中修改的文件，然后将所做的更改上载到 GitHub。

        git add .
        git commit -m "adding config files"
        git push

该应用程序现在应该已为部署做好准备。

> [WACOM.NOTE] 对于更复杂的应用程序，或者对于不同的数据库或应用程序服务器，您可能需要其他配置或部署脚本。

## <span id="deploy"></span></a>部署

1.  在本地开发计算机中使用以下命令，将应用程序所使用的配置文件部署到虚拟机中。

        bundle exec cap production setup

    Capistrano 将使用 SSH 连接到虚拟机，然后创建该应用程序将部署到的目录 (~/apps)。如果这是首次部署，则 capistrano-postgresql gem 还将在服务器上的 PostgreSQL 中创建角色和数据库。它还将创建 Rails 将用于连接到数据库的 database.yml 配置文件。

    > [WACOM.NOTE] 如果您在部署时收到错误：“从身份验证套接字读取响应长度时出错”，则可能需要使用 `ssh-agent` 命令在开发环境上启动 SSH 代理。例如，将 `eval $(ssh-agent)` 添加到 ~/.bash\_profile 文件。
    >
    > 您可能还需要使用 `ssh-add` 命令将 SSH 密钥添加到代理缓存。

2.  使用以下命令执行生产部署。这会将应用程序部署到虚拟机、启动 Unicorn 服务，并将 Nginx 配置为将流量路由到 Unicorn。

        bundle exec cap production deploy

    此命令会将应用程序部署到虚拟机、安装所需的 gem，然后启动/重新启动 Unicorn 和 Nginx。

    > [WACOM.NOTE] 在处理过程中，该过程可能会暂停几分钟。

    > [WACOM.NOTE] 部署的某些部分可能会返回“exit status 1 (failed)”。只要部署成功完成，通常可忽略这些返回结果。

    > [WACOM.NOTE] 在某些系统中，在对 GitHub 进行身份验证时，您可能会遇到 SSH 代理无法将凭据转发到远程虚拟机的情况。如果发生这种情况，则可以通过修改 **config/deploy.rb** 文件并将 `set :repo_url` 行更改为使用 HTTPS 访问 Github 来解决此错误。在使用 HTTPS 时，您必须指定作为 URL 的一部分的 GitHub 用户名和密码（或身份验证令牌）。例如：
    >
    > \`set :repo\_url, 'https://you:yourpassword@github.com/You/yourrepository.git'
    >
    > 虽然这应允许您绕过错误完成本教程，但是不建议对生产部署使用此解决方案，因为它会将身份验证凭据以纯文本格式作为应用程序的一部分进行储存。您在使用 SSH 代理进行转发时应该参考适用于您的操作系统的文档

此时，您的 Ruby on Rails 应用程序应该已经在 Azure 虚拟机上运行。为了验证这一点，请在 Web 浏览器中输入虚拟机的 DNS 名称。例如，http://railsvm.chinacloudapp.cn。此时应显示文章索引，并且您应该能够创建、编辑和删除文章。

## <span id="next"></span></a>后续步骤

在本文中，您学习了如何使用 Capistrano 创建基本 Rails 应用程序并将其发布到 Azure 虚拟机。使用基本的应用程序（如本文中的应用程序）只触及到了使用 Capistrano 进行部署时可实现的操作的皮毛。有关使用 Capistrano 的详细信息，请参阅：

-   [Capistranorb.com][Capistrano] – Capistrano 站点。
-   [Azure、Ruby on Rails、Capistrano 3 和 PostgreSQL][Azure、Ruby on Rails、Capistrano 3 和 PostgreSQL] – 一种用来部署到 Azure 的涉及自定义部署脚本的可选方法。
-   [Capistrano 3 教程][Capistrano 3 教程] – 使用 Capistrano 3 的教程。

有关创建 Rails 应用程序并仅使用 SSH 将其部署到 Azure 虚拟机的更多基础示例，请参阅[使用 Linux 虚拟机托管 Ruby on Rails Web 应用][使用 Linux 虚拟机托管 Ruby on Rails Web 应用]。

如果想要详细了解 Ruby on Rails，请访问 [Ruby on Rails 指南][Ruby on Rails 指南]。

若要了解如何使用 Azure SDK for Ruby 从 Ruby 应用程序访问 Azure 服务，请参阅：

-   [使用 Blob 存储非结构化数据][使用 Blob 存储非结构化数据]

-   [使用表存储键/值对][使用表存储键/值对]

-   [使用内容交付网络提供高带宽内容][使用内容交付网络提供高带宽内容]

  [Capistrano 3]: https://github.com/capistrano/capistrano/
  [Nginx]: http://nginx.org/
  [Unicorn]: https://github.com/blog/517-unicorn
  [PostgreSQL]: https://www.postgresql.org
  [显示 Listing Posts 的浏览器窗口]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/blograilscloud.png
  [设置开发环境]: #setup
  [创建 Rails 应用程序]: #create
  [测试应用程序]: #test
  [创建源存储库]: #repository
  [创建 Azure 虚拟机]: #createvm
  [测试 Nginx]: #nginx
  [准备部署]: #capify
  [部署]: #deploy
  [后续步骤]: #next
  [Homebrew]: http://brew.sh/
  [RubyInstaller]: http://RubyInstaller.org/
  [Git for Windows]: http://git-scm.com/download/win
  [SQLite3 数据库]: http://www.sqlite.org/
  [列出文章的页面]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/blograilslocal.png
  [Git]: http://git-scm.com/
  [GitHub]: https://github.com/
  [生成 SSH 密钥]: https://help.github.com/articles/generating-ssh-keys
  [此处]: /zh-cn/manage/linux/tutorials/virtual-machine-from-gallery/
  [管理门户]: https://manage.windowsazure.cn/
  [Capistrano]: http://capistranorb.com
  [如何在 Azure 上通过 Linux 使用 SSH]: http://azure.microsoft.com/zh-cn/documentation/articles/linux-use-ssh-key/
  [nginx 欢迎页]: ./media/virtual-machines-ruby-deploy-capistrano-host-nginx-unicorn/welcomenginx.png
  [Azure、Ruby on Rails、Capistrano 3 和 PostgreSQL]: http://wootstudio.ca/articles/tutorial-windows-azure-ruby-on-rails-capistrano-3-postgresql
  [Capistrano 3 教程]: http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/
  [使用 Linux 虚拟机托管 Ruby on Rails Web 应用]: /zh-cn/develop/ruby/tutorials/web-app-with-linux-vm/
  [Ruby on Rails 指南]: http://guides.rubyonrails.org/
  [使用 Blob 存储非结构化数据]: /zh-cn/develop/ruby/how-to-guides/blob-storage/
  [使用表存储键/值对]: /zh-cn/develop/ruby/how-to-guides/table-service/
  [使用内容交付网络提供高带宽内容]: /zh-cn/develop/ruby/app-services/
