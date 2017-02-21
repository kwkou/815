<properties
    pageTitle="在 Mac OS X 上设置开发环境 | Azure"
    description="安装运行时、SDK 和工具并创建本地开发群集。完成此设置后，可以在 Mac OS X 上构建应用程序。"
    services="service-fabric"
    documentationcenter=".net"
    author="seanmck"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="bf84458f-4b87-4de1-9844-19909e368deb"
    ms.service="service-fabric"
    ms.devlang="dotNet"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="12/27/2016"
    wacn.date="02/20/2017"
    ms.author="seanmck" />  


# 在 Mac OS X 上设置开发环境

> [AZURE.SELECTOR]
-[ Windows](/documentation/articles/service-fabric-get-started/)
- [Linux](/documentation/articles/service-fabric-get-started-linux/)
- [OSX](/documentation/articles/service-fabric-get-started-mac/)

可以构建能够在使用 Mac OS X 的 Linux 群集中运行的 Service Fabric 应用程序。本文介绍如何设置 Mac 以便进行开发。

## 先决条件

Service Fabric 不是在 OS X 上以本机方式运行。为了运行本地 Service Fabric 群集，我们提供了使用 Vagrant 和 VirtualBox 的预配置 Ubuntu 虚拟机。开始之前，需要：

- [Vagrant（v1.8.4 或更高版本）](http://www.vagrantup.com/downloads.html)
- [VirtualBox](http://www.virtualbox.org/wiki/Downloads)

## 创建本地 VM
若要创建包含 5 节点型 Service Fabric 群集的本地 VM，请执行以下步骤：

1. 克隆 **Vagrantfile** 存储库
   

    	git clone https://github.com/azure/service-fabric-linux-vagrant-onebox.git


2. 导航到存储库的本地副本


    	cd service-fabric-linux-vagrant-onebox


3. （可选）修改默认的 VM 设置

    默认情况下，本地 VM 的配置如下：

    - 3 GB 的分配内存
    - 在 IP 192.168.50.50 上配置了专用主机网络，可以传递 Mac 主机的流量

    可以在 Vagrantfile 中更改上述任何一项设置或将其他配置添加到 VM。有关配置选项的完整列表，请参阅 [Vagrant 文档](http://www.vagrantup.com/docs)。

4. 创建 VM


    	vagrant up


    此步骤下载预配置的 VM 映像、在本地将它启动，然后在其中设置一个本地 Service Fabric 群集。此过程预计需要几分钟时间。如果设置成功完成，输出中会显示一条消息，指出群集正在启动。

    ![预配 VM 后开始设置群集][cluster-setup-script]  


5. 导航到 Service Fabric Explorer (http://192.168.50.50:19080/Explorer)（假设保留了默认专用网络 IP），测试是否已正确设置群集。

    ![在 Mac 主机中查看 Service Fabric Explorer][sfx-mac]  



## 安装适用于 Eclipse Neon 的 Service Fabric 插件（可选）

Service Fabric 提供适用于 Eclipse Neon IDE 的插件，可简化构建和部署 Java 服务的过程。

1. 在 Eclipse 中，请确保已安装 Buildship 1.0.17 或更高版本。可以选择“帮助”>“安装详细信息”检查已安装的组件版本。可以使用[此处][buildship-update]的说明更新 Buildship。

2. 若要安装 Service Fabric 插件，请选择“帮助”>“安装新软件...”

3. 在“使用”文本框中，输入：http://dl.windowsazure.com/eclipse/servicefabric

4. 单击“添加”。

    ![适用于 Service Fabric 的 Eclipse Neon 插件][sf-eclipse-plugin-install]  


5. 选择 Service Fabric 插件，然后单击“下一步”。

6. 继续安装，并接受最终用户许可协议。

## 后续步骤
<!-- Links -->


- [在 Azure 门户中创建 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-via-portal/)
- [使用 Azure Resource Manager 创建 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-via-arm/)
- [了解 Service Fabric 应用程序模型](/documentation/articles/service-fabric-application-model/)

<!-- Images -->

[cluster-setup-script]: ./media/service-fabric-get-started-mac/cluster-setup-mac.png
[sfx-mac]: ./media/service-fabric-get-started-mac/sfx-mac.png
[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-mac/sf-eclipse-plugin-install.png
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording update-->