<properties
    pageTitle="在 Linux VM 上托管 Ruby on Rails 网站 | Azure"
    description="在 Azure 上使用 Linux 虚拟机设置和托管基于 Ruby on Rails 的网站。"
    services="virtual-machines-linux"
    documentationcenter="ruby"
    author="rmcmurray"
    manager="erikre"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="aad32685-3550-4bff-9c73-beb8d70b3291"
    ms.service="virtual-machines-linux"
    ms.workload="web"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="ruby"
    ms.topic="article"
    ms.date="12/22/2016"
    wacn.date="03/01/2017"
    ms.author="robmcm" />  


# Azure VM 上的 Ruby on Rails Web 应用程序
本教程介绍如何在 Azure 中使用 Linux 虚拟机托管 Ruby on Rails 网站。

本教程使用 Ubuntu Server 14.04 LTS 进行验证。如果使用其他 Linux 发行版，可能需要修改安装 Rails 的步骤。

> [AZURE.IMPORTANT]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型的情况。Azure 建议大多数新部署使用 Resource Manager 模型。
> 
> 

## 创建 Azure VM
首先，使用 Linux 映像创建 Azure VM。

若要创建 VM，可使用 Azure 经典管理门户或 Azure 命令行接口 \(CLI\)。

### Azure 经典管理门户
1. 登录到 [Azure 经典管理门户](http://manage.windowsazure.cn)
2. 单击“新建”\>“计算”\>“虚拟机”\>“快速创建”。选择 Linux 映像。
3. 输入密码。

预配 VM 后，单击 VM 名称，然后单击“仪表板”。找到“SSH 详细信息”下列出的 SSH 终结点。

### Azure CLI
执行[创建运行 Linux 的虚拟机][vm-instructions]中的步骤。

预配 VM 后，可通过运行以下命令获取 SSH 终结点：

    azure vm endpoint list <vm-name>  

## 在 Rails 上安装 Ruby
1. 使用 SSH 连接到 VM。
2. 从 SSH 会话中，使用以下命令在虚拟机上安装 Ruby：
   
        sudo apt-get update -y
        sudo apt-get upgrade -y
        sudo apt-get install ruby ruby-dev build-essential libsqlite3-dev zlib1g-dev nodejs -y
   
    安装可能需要几分钟时间。安装完成后，使用以下命令验证 Ruby 是否已安装：
   
        ruby -v
   
    这将返回已安装的 Ruby 的版本。
3. 使用以下命令安装 Rails：
   
        sudo gem install rails --no-rdoc --no-ri -V
   
    使用 --no-rdoc 和 --no-ri 标志可跳过安装文档，从而加快速度。执行此命令可能需要花费较长时间，添加 -V 可显示有关安装进度的信息。

## 创建并运行应用
在仍通过 SSH 登录时，运行以下命令：

    rails new myapp
    cd myapp
    rails server -b 0.0.0.0 -p 3000

[new](http://guides.rubyonrails.org/command_line.html#rails-new) 命令创建新的 Rails 应用。[server](http://guides.rubyonrails.org/command_line.html#rails-server)命令启动 Rails 附带的 WEBrick Web 服务器。（在生产环境中使用时，可能要使用其他服务器，例如 Unicorn 或 Passenger。）

输出应显示如下。

    => Booting WEBrick
    => Rails 4.2.1 application starting in development on http://0.0.0.0:3000
    => Run `rails server -h` for more startup options
    => Ctrl-C to shutdown server
    [2015-06-09 23:34:23] INFO  WEBrick 1.3.1
    [2015-06-09 23:34:23] INFO  ruby 1.9.3 (2013-11-22) [x86_64-linux]
    [2015-06-09 23:34:23] INFO  WEBrick::HTTPServer#start: pid=27766 port=3000

## 添加终结点
1. 转到 [Azure 经典管理门户][management-portal]并选择 VM。
   
    ![虚拟机列表][vmlist]  

2. 选择页面顶部的“终结点”，然后单击页面底部的“+添加终结点”。
   
    ![终结点页面][endpoints]
3. 在“添加终结点”对话框中，选择“添加独立终结点”，然后单击“下一步”箭头。
   
    ![新建终结点对话框][new-endpoint1]
4. 在下一个对话框页面中，输入以下信息：
   
    * **名称**：HTTP
    * **协议**：TCP
    * **公用端口**：80
    * **专用端口**：3000
     
     这将创建一个公用端口 80，将流量路由到专用端口 3000，即 Rails 服务器侦听的端口。
     
     ![新建终结点对话框][new-endpoint]
5. 单击复选标记以保存终结点。
6. 将出现一条消息，指出“正在进行更新”。此消息消失后，终结点将处于活动状态。现在，可以通过导航到虚拟机的 DNS 名称测试应用程序。网站应显示如下：
   
    ![默认 rails 页面][default-rails-cloud]  

## <a id="next"></a> 后续步骤
在本教程中，手动执行大多数步骤。在生产环境中，可在开发计算机上编写应用，然后将其部署到 Azure VM。此外，大多数生产环境都结合其他服务器进程（如 Apache 或 NginX）托管 Rails 应用程序，这些进程处理路由到多个 Rails 应用程序实例的请求并提供静态资源。有关详细信息，请参阅 http://rubyonrails.org/deploy/。

若要详细了解 Ruby on Rails，请参阅 [Ruby on Rails 指南][rails-guides]。

若要从 Ruby 应用程序使用 Azure 服务，请参阅：

* [使用 Blob 存储非结构化数据][blobs]
* [使用表存储键/值对][tables]
* [使用内容传送网络提供高带宽内容][cdn-howto]

<!-- WA.com links -->
[blobs]: /documentation/articles/storage-ruby-how-to-use-blob-storage/
[cdn-howto]: /develop/ruby/app-services/
[management-portal]: https://manage.windowsazure.cn/
[tables]: /documentation/articles/storage-ruby-how-to-use-table-storage/
[vm-instructions]: /documentation/articles/virtual-machines-linux-classic-createportal/

<!-- External Links -->
[rails-guides]: http://guides.rubyonrails.org/
[sqlite3]: http://www.sqlite.org/

<!-- Images -->


[default-rails-cloud]: ./media/virtual-machines-linux-classic-ruby-rails-web-app/basicrailscloud.png
[vmlist]: ./media/virtual-machines-linux-classic-ruby-rails-web-app/vmlist.png
[endpoints]: ./media/virtual-machines-linux-classic-ruby-rails-web-app/endpoints.png
[new-endpoint]: ./media/virtual-machines-linux-classic-ruby-rails-web-app/newendpoint.png
[new-endpoint1]: ./media/virtual-machines-linux-classic-ruby-rails-web-app/newendpoint1.png

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description: update meta properties & wording update-->