<properties
   pageTitle="在 Azure 上部署三节点 Deis 群集"
   description="本文介绍如何使用 Azure 资源管理器模板在 Azure 上创建三节点 Deis 群集"
   services="virtual-machines"
   documentationCenter=""
   authors="HaishiBai"
   manager="larar"
   editor="08/29/2015"/>

<tags
   ms.service="virtual-machines"
   ms.date="06/24/2015"
   wacn.date=""/>

# 部署三节点 Deis 群集

本文将逐步指导你完成在 Azure 上设置 [Deis](http://deis.io/) 群集的过程。其中包括所有的步骤，从创建必要的证书，到在新设置的群集上部署和缩放示例 **Go** 应用程序。

下图显示了部署的系统的体系结构。系统管理员可以使用 Deis 工具（如 **deis** 和 **deisctl**）来管理群集。连接是通过会将连接转发到群集上某个成员节点的 Azure 负载平衡器建立的。客户端也通过负载平衡器访问部署的应用程序。在此情况下，负载平衡器会将流量转发到 Deis 路由器网络，从者进一步将流量路由到托管在群集上的对应 Docker 容器。

  ![部署的 Desis 群集的体系结构示意图](./media/virtual-machines-deis-cluster/architecture-overview.png)

若要完成以下步骤，你需要：

 * 一个有效的 Azure 订阅。如果没有，你可以在 [azure.com](http://www.windowsazure.cn/) 上获取免费试用帐户。
 * 用来使用 Azure 资源组的工作或学校 ID。如果你有个人帐户并使用 Microsoft ID 登录，则需要<!--[-->基于你的个人 ID 创建工作 ID<!--](/documentation/articles/resource-group-create-work-id-from-personal)-->。
 * 以下组件之一（取决于客户端操作系统）[Azure PowerShell](/documentation/articles/powershell-install-configure) 或[适用于 Mac、Linux 和 Windows 的 Azure CLI](/documentation/articles/xplat-cli-install)。
 * [OpenSSL](https://www.openssl.org/)。OpenSSL 用于生成必要的证书。
 * Git 客户端，如 [Git Bash](https://git-scm.com/)。
 * 若要测试示例应用程序，你还需要一个 DNS 服务器。可以使用任何 DNS 服务器或支持通配符 A 记录的服务。
 * 用于运行 Deis 客户端工具的计算机。可以使用本地计算机或虚拟机。几乎可以在任何 Linux 分发版中运行这些工具，但以下说明使用的是 Ubuntu。

## 设置群集

在本部分中，你将使用开放源代码存储库 [azure-quickstart-templates](https://github.com/Azure/azure-quickstart-templates) 中的 <!--[-->Azure 资源管理员<!--](/documentation/articles/resource-group-overview)-->模板。首先复制该模板。其次，创建用于身份验证的新 SSH 密钥对。再次，为群集配置新的标识符。最后，使用 Shell 脚本或 PowerShell 脚本设置群集。

1. 克隆存储库：[https://github.com/Azure/azure-quickstart-templates](https://github.com/Azure/azure-quickstart-templates)。

        git clone https://github.com/Azure/azure-quickstart-templates

2. 转到模板文件夹：

        cd azure-quickstart-templates\deis-cluster-coreos

3. 使用 ssh-keygen 创建新的 SSH 密钥对：

        ssh-keygen -t rsa -b 4096 -c "[your_email@domain.com]"

4. 使用上述私钥生成证书：

        openssl req -x509 -days 365 -new -key [your private key file] -out [cert file to be generated]

5. 转到 [https://discovery.etcd.io/new](https://discovery.etcd.io/new)，生成如下所示的新群集令牌：

        https://discovery.etcd.io/6a28e078895c5ec737174db2419bb2f3
<br /> 每个 CoreOS 群集需要有此免费服务提供的唯一令牌。有关详细信息，请参阅 [CoreOS 文档](https://coreos.com/docs/cluster-management/setup/cluster-discovery/)。

6. 修改 **cloud-config.yaml**，将现有 **discovery** 令牌替换为新令牌：

        #cloud-config
        ---
        coreos:
          etcd:
            # generate a new token for each unique cluster from https://discovery.etcd.io/new
            # uncomment the following line and replace it with your discovery URL
            discovery: https://discovery.etcd.io/3973057f670770a7628f917d58c2208a
        ...

7. 修改 **azuredeploy parameters.json** 文件：在文字编辑器中打开步骤 4 创建的证书。将 `----BEGIN CERTIFICATE-----` 与 `-----END CERTIFICATE-----` 之间的所有文本复制到 **sshKeyData** 参数中（需要删除所有换行符）。

8. 修改 **newStorageAccountName** 参数。这是 VM OS 磁盘的存储帐户。此帐户名称必须全域唯一。

9. 修改 **publicDomainName** 参数。此参数将成为与负载平衡器公共 IP 关联的 DNS 名称的一部分。最终的 FQDN 格式是 _[此参数的值]_ _[区域]_.cloudapp.azure.com。例如，如果你将名称指定为 deishbai32，资源组已部署到美国西部区域，则负载平衡器的最终 FQDN 是 deishbai32.westus.cloudapp.azure.com。

10. 保存参数文件。接下来，你可以使用 Azure PowerShell 设置群集：

        .\deploy-deis.ps1 -ResourceGroupName [resource group name] -ResourceGroupLocation "West US" -TemplateFile
        .\azuredeploy.json -ParametersFile .\azuredeploy-parameters.json -CloudInitFile .\cloud-config.yaml

  也可以使用 Azure CLI：

        ./deploy-deis.sh -n "[resource group name]" -l "West US" -f ./azuredeploy.json -e ./azuredeploy-parameters.json
        -c ./cloud-config.yaml  

11. 设置资源组后，可以在 Azure 门户上看到组中的所有资源。如以下屏幕截图所示，资源组中有一个包含三个 VM 的虚拟网络，这些 VM 已加入同一个可用性集。该组还包含具有关联公共 IP 的负载平衡器。

  ![Azure 门户上显示的已设置资源组](./media/virtual-machines-deis-cluster/resource-group.png)

## 安装客户端

需要使用 **deisctl** 来控制 Deis 群集。尽管 deisctl 会自动安装在所有群集节点中，但较好的做法是在独立的管理计算机上使用 deisctl。此外，因为所有节点上只配置了专用 IP 地址，因此你需要通过负载平衡器（有公共 IP）使用 SSH 隧道，来连接到节点计算机。以下是在独立 Ubuntu 物理机或虚拟机上设置 deisctl 的步骤。

1. 安装 deisctl:mkdir deis

        cd deis
        curl -sSL http://deis.io/deisctl/install.sh | sh -s 1.6.1
        sudo ln -fs $PWD/deisctl /usr/local/bin/deisctl

2. 将私钥添加 SSH 代理：

        eval `ssh-agent -s`
        ssh-add [path to the private key file, see step 1 in the previous section]

3. 配置 deisctl：

        export DEISCTL_TUNNEL=[public ip of the load balancer]: /documentation/articles/2223
模板定义了将 2223 映射到实例 1、将 2224 映射到实例 2、将 2225 映射到实例 3 的入站 NAT 规则。这提供了使用 deisctl 工具时的冗余。可以在 Azure 门户中检查这些规则：

![负载平衡器上的 NAT 规则](./media/virtual-machines-deis-cluster/nat-rules.png)

> [AZURE.NOTE]模板目前仅支持三节点群集。这是因为 Azure 资源管理器模板 NAT 规则定义存在限制，它不支持循环语法。

## 安装和启动 Deis 平台

现在，你可以使用 deisctl 来安装和启动 Deis 平台：

    deisctl config platform set domain=[some domain]
    deisctl config platform set sshPrivateKey=[path to the private key file]
    deisctl install platform
    deisctl start platform

> [AZURE.NOTE]平台需要一段时间来启动（最多 10 分钟）。启动构建器服务需要很长的时间。有时需要尝试多次才能成功：如果操作看上去已挂起，请尝试键入 `ctrl+c` 来中断命令的执行，然后重试。

可以使用 `deisctl list` 来验证所有服务是否都在运行：

    deisctl list
    UNIT                            MACHINE                 LOAD    ACTIVE          SUB
    deis-builder.service            ebe3005e.../10.0.0.6    loaded  active          running
    deis-controller.service         ebe3005e.../10.0.0.6    loaded  active          running
    deis-database.service           9c79bbdd.../10.0.0.5    loaded  active          running
    deis-logger.service             9c79bbdd.../10.0.0.5    loaded  active          running
    deis-logspout.service           8d658d5a.../10.0.0.4    loaded  active          running
    deis-logspout.service           9c79bbdd.../10.0.0.5    loaded  active          running
    deis-logspout.service           ebe3005e.../10.0.0.6    loaded  active          running
    deis-publisher.service          8d658d5a.../10.0.0.4    loaded  active          running
    deis-publisher.service          9c79bbdd.../10.0.0.5    loaded  active          running
    deis-publisher.service          ebe3005e.../10.0.0.6    loaded  active          running
    deis-registry@1.service         ebe3005e.../10.0.0.6    loaded  active          running
    deis-router@1.service           ebe3005e.../10.0.0.6    loaded  active          running
    deis-router@2.service           8d658d5a.../10.0.0.4    loaded  active          running
    deis-router@3.service           9c79bbdd.../10.0.0.5    loaded  active          running
    deis-store-daemon.service       8d658d5a.../10.0.0.4    loaded  active          running
    deis-store-daemon.service       9c79bbdd.../10.0.0.5    loaded  active          running
    deis-store-daemon.service       ebe3005e.../10.0.0.6    loaded  active          running
    deis-store-gateway@1.service    9c79bbdd.../10.0.0.5    loaded  active          running
    deis-store-metadata.service     8d658d5a.../10.0.0.4    loaded  active          running
    deis-store-metadata.service     9c79bbdd.../10.0.0.5    loaded  active          running
    deis-store-metadata.service     ebe3005e.../10.0.0.6    loaded  active          running
    deis-store-monitor.service      8d658d5a.../10.0.0.4    loaded  active          running
    deis-store-monitor.service      9c79bbdd.../10.0.0.5    loaded  active          running
    deis-store-monitor.service      ebe3005e.../10.0.0.6    loaded  active          running
    deis-store-volume.service       8d658d5a.../10.0.0.4    loaded  active          running
    deis-store-volume.service       9c79bbdd.../10.0.0.5    loaded  active          running
    deis-store-volume.service       ebe3005e.../10.0.0.6    loaded  active          running

祝贺你！ 你已在 Azure 上运行了一个 Deis 群集！ 接下来，让我们部署一个简单的 Go 应用程序，来了解该群集的运行方式。

## 部署和缩放 Hello World 应用程序

以下步骤说明如何将“Hello World”Go 应用程序部署到群集。这些步骤基于 [Deis 文档](http://docs.deis.io/en/latest/using_deis/using-dockerfiles/#using-dockerfiles)。

1. 为了使路由网络正确工作，指向负载平衡器公共 IP 的域需要有通配符 A 记录。以下屏幕截图显示了在 GoDaddy 上注册的示例域的 A 记录：

    ![Godaddy A 记录](./media/virtual-machines-deis-cluster/go-daddy.png)
<p />
2. 安装 deis：

        mkdir deis
        cd deis
        curl -sSL http://deis.io/deis-cli/install.sh | sh
        ln -fs $PWD/deis /usr/local/bin/deis
        
3. 创建新的 SSH 密钥，然后将公钥添加 GitHub（当然，你也可以使用现有的密钥）。若要创建新的 SSH 密钥对，请使用：

        cd ~/.ssh
        ssh-keygen (press [Enter]s to use default file names and empty passcode)

4. 在 GitHub 添加 id_rsa.pub 或所选的公钥。可以使用 SSH 密钥配置屏幕上的“添加 SSH 密钥”按钮来执行此操作。

  ![Github 密钥](./media/virtual-machines-deis-cluster/github-key.png) <p /> 5.注册新用户：

        deis register http://deis.[your domain]
<p />
6. 添加 SSH 密钥：

        deis keys:add [path to your SSH public key]
  <p />
7. 创建应用程序。

        git clone https://github.com/deis/helloworld.git
        cd helloworld
        deis create
        git push deis master
<p />
8. git push 将触发 Docker 映像的构建和部署，这需要几分钟来完成。根据我的经验，步骤 10（将映像推送到专用存储库）偶尔会挂起。如果发生此情况，你可以停止过程，使用 `deis apps:destroy –a <application name>` 删除应用程序，然后重试。你也可以使用 `deis apps:list` 找出应用程序的名称。如果一切正常，你应会在命令输出的末尾看到如下内容：

        -----> Launching...
               done, lambda-underdog:v2 deployed to Deis
               http://lambda-underdog.artitrack.com
               To learn more, use `deis help` or visit http://deis.io
        To ssh://git@deis.artitrack.com:2222/lambda-underdog.git
         * [new branch]      master -> master
<p />
9. 验证应用程序是否工作：

        curl -S http://[your application name].[your domain]
  你应会看到：

        Welcome to Deis!
        See the documentation at http://docs.deis.io/ for more information.
        (you can use geis apps:list to get the name of your application).
<p />
10. 将应用程序缩放为 3 个实例：

        deis scale cmd=3
<p />
11. （可选）你可以使用 deis info 来检查应用程序的详细信息。以下是我的应用程序部署的输出：

        deis info
        === lambda-underdog Application
        {
          "updated": "2015-05-22T06:14:10UTC",
          "uuid": "10c74ee7-b7ff-4786-967a-7e65af7eabc3",
          "created": "2015-05-22T06:07:55UTC",
          "url": "lambda-underdog.artitrack.com",
          "owner": "haishi",
          "id": "lambda-underdog",
          "structure": {
            "cmd": 3
          }
        }

        === lambda-underdog Processes
        --- cmd:
        cmd.1 up (v2)
        cmd.2 up (v2)
        cmd.3 up (v2)

        === lambda-underdog Domains
        No domains

## 后续步骤

本文已指导你完成使用 Azure 资源管理器模板在 Azure 上设置新 Deis 群集的步骤。该模板支持工具连接冗余，以及对已部署应用程序的负载平衡。该模板还避免使用成员节点上的公共 IP，因此可以节省宝贵的公共 IP 资源，并为宿主应用程序提供更安全的环境。若要了解更多信息，请参阅下列文章：

[Azure 资源管理器概述][resource-group-overview]  
[如何使用 Azure CLI][azure-command-line-tools]  
[将 Azure PowerShell 与 Azure 资源管理器配合使用][powershell-azure-resource-manager]  

[azure-command-line-tools]: /documentation/articles/xplat-cli
[resource-group-overview]: /documentation/articles/resource-group-overview
[powershell-azure-resource-manager]: /documentation/articles/powershell-azure-resource-manager
<!---HONumber=67-->