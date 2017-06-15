<properties
    pageTitle="使用 CLI 与 Service Fabric 群集交互 | Azure"
    description="如何使用 Azure CLI 与 Service Fabric 群集交互"
    services="service-fabric"
    documentationcenter=".net"
    author="mani-ramaswamy"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="c3ec8ff3-3b78-420c-a7ea-0c5e443fb10e"
    ms.service="service-fabric"
    ms.devlang="dotNet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/02/2017"
    wacn.date="06/15/2017"
    ms.author="subramar"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="2cab73a861604346910f95249ba1d6023dff6904"
    ms.lasthandoff="04/14/2017" />

# <a name="using-the-azure-cli-to-interact-with-a-service-fabric-cluster"></a>使用 Azure CLI 来与 Service Fabric 群集交互
可以在 Linux 计算机上使用“Linux 上的 Azure CLI”来与 Service Fabric 群集交互。

第一步是通过 git rep 获取最新版本的 CLI，然后使用以下命令在路径中设置该版本：

     git clone https://github.com/Azure/azure-xplat-cli.git
     cd azure-xplat-cli
     npm install
     sudo ln -s \$(pwd)/bin/azure /usr/bin/azure
     azure servicefabric

对于 CLI 支持的每个命令，可以键入该命令的名称获取相关的帮助。 这些命令支持自动补全。 例如，使用以下命令可以获取所有应用程序命令的帮助。 

     azure servicefabric application 

可以进一步筛选特定命令的帮助，如以下示例所示：

     azure servicefabric application  create

若要在 CLI 中启用自动补全，请运行以下命令：

    azure --completion >> ~/azure.completion.sh
    echo 'source ~/azure.completion.sh' >> ~/.sh\_profile
    source ~/azure.completion.sh

以下命令连接到群集，并显示群集中的节点：

     azure servicefabric cluster connect http://localhost:19080
     azure servicefabric node show

若要使用命名参数并了解其用途，可以在命令后面键入 --help。 例如：

     azure servicefabric node show --help
     azure servicefabric application create --help

从 **不属于群集**的计算机连接到多计算机群集时，可以使用以下命令：

     azure servicefabric cluster connect http://PublicIPorFQDN:19080

将 PublicIPorFQDN 标记适当地替换为实际 IP 或 FQDN。 从 **属于群集**的计算机连接到多计算机群集时，可以使用以下命令：

     azure servicefabric cluster connect --connection-endpoint http://localhost:19080 --client-connection-endpoint PublicIPorFQDN:19000

> [AZURE.WARNING]
> 这些群集不安全，因此，可以通过在群集清单中添加公共 IP 地址打开你的单机。

## <a name="using-the-azure-cli-to-connect-to-a-service-fabric-cluster"></a>使用 Azure CLI 连接到 Service Fabric 群集
以下 Azure CLI 命令说明如何连接到安全群集。 证书详细信息必须与群集节点上的证书匹配。

    azure servicefabric cluster connect --connection-endpoint http://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert

如果证书具有证书颁发机构 (CA)，需要按以下示例所示添加 --ca-cert-path 参数： 

     azure servicefabric cluster connect --connection-endpoint http://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --ca-cert-path /tmp/ca1,/tmp/ca2 

如果有多个 CA，请使用逗号作为分隔符。

如果证书中的公用名与连接终结点不匹配，则可以使用参数 `--strict-ssl-false` 跳过验证，如下面的命令中所示： 

    azure servicefabric cluster connect --connection-endpoint http://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --strict-ssl-false 

如果想要跳过 CA 验证，可以添加 -reject-unauthorized-false 参数，如下面的命令中所示： 

    azure servicefabric cluster connect --connection-endpoint http://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --reject-unauthorized-false 

在连接后，你应能够运行其他 CLI 命令以与群集进行交互。 

## <a name="deploying-your-service-fabric-application"></a>部署 Service Fabric 应用程序
执行以下命令可以复制、注册和启动 Service Fabric 应用程序：

    azure servicefabric application package copy [applicationPackagePath] [imageStoreConnectionString] [applicationPathInImageStore]
    azure servicefabric application type register [applicationPathinImageStore]
    azure servicefabric application create [applicationName] [applicationTypeName] [applicationTypeVersion]

## <a name="upgrading-your-application"></a>升级应用程序
该进程类似于 [Windows 中的进程](/documentation/articles/service-fabric-application-upgrade-tutorial-powershell/)。

从项目根目录生成、复制、注册并创建你的应用程序。 如果应用程序实例名为 fabric:/MySFApp 且类型为 MySFApp，则命令如下所示：

     azure servicefabric cluster connect http://localhost:19080
     azure servicefabric application package copy MySFApp fabric:ImageStore
     azure servicefabric application type register MySFApp
     azure servicefabric application create fabric:/MySFApp MySFApp 1.0

对应用程序进行更改，然后重新构建已修改的服务。  使用更新的服务版本（在适当情况下，还包括代码、配置或数据）更新已修改的服务的清单文件 (ServiceManifest.xml)。 同时，请使用更新的应用程序版本号和已修改的服务修改应用程序的清单 (ApplicationManifest.xml)。  现在，使用以下命令复制并注册已更新的应用程序：

     azure servicefabric cluster connect http://localhost:19080>
     azure servicefabric application package copy MySFApp fabric:ImageStore
     azure servicefabric application type register MySFApp

接下来，可以使用以下命令启动应用程序升级：

     azure servicefabric application upgrade start -–application-name fabric:/MySFApp -–target-application-type-version 2.0  --rolling-upgrade-mode UnmonitoredAuto

现在，可以使用 SFX 监视应用程序升级。 应用程序在几分钟内即可完成更新。  你还可以尝试出现错误的更新应用程序，检查 Service Fabric 中的自动回滚功能。

## <a name="converting-from-pfx-to-pem-and-vice-versa"></a>从 PFX 转换到 PEM，反之亦然

可能需要在本地计算机（搭载 Windows 或 Linux）中安装证书以访问不同环境中的安全群集。 例如，从 Windows 计算机访问受保护的 Linux 群集时，需要将证书从 PFX 转换到 PEM；反之，需要从 PEM 转换到 PFX。 

要将 PEM 文件转换为 PFX 文件，请使用以下命令：

    openssl pkcs12 -export -out certificate.pfx -inkey mycert.pem -in mycert.pem -certfile mycert.pem

要将 PFX 文件转换为 PEM 文件，请使用以下命令：

    openssl pkcs12 -in certificate.pfx -out mycert.pem -nodes


<a id="troubleshooting"></a>
## <a name="troubleshooting"></a>故障排除
### <a name="copying-of-the-application-package-does-not-succeed"></a>未成功复制应用程序包
检查是否已安装 `openssh` 。 默认情况下，Ubuntu Desktop 未安装此软件。 使用以下命令安装它：

     sudo apt-get install openssh-server openssh-client**

如果问题仍然存在，请尝试通过使用以下命令更改 **sshd_config** 文件来对 ssh 禁用 PAM：

     sudo vi /etc/ssh/sshd_config
    #Change the line with UsePAM to the following: UsePAM no
     sudo service sshd reload

如果问题仍然存在，请尝试通过执行以下命令增加 ssh 会话数：

     sudo vi /etc/ssh/sshd\_config
    # Add the following to lines:
    # MaxSessions 500
    # MaxStartups 300:30:500
     sudo service sshd reload

目前不支持使用密钥（而不是密码）进行 ssh 身份验证（因为平台使用 ssh 来复制包），请改用密码身份验证。



## <a name="next-steps"></a>后续步骤
设置开发环境，并将 Service Fabric 应用程序部署到 Linux 群集。
<!--Update_Description:add "从 PFX 转换到 PEM，反之亦然" section;add anchors to sub titles-->