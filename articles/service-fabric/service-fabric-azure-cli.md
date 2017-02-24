<properties
   pageTitle="使用 CLI 来与 Service Fabric 群集交互 | Azure"
   description="如何使用 Azure CLI 来与 Service Fabric 群集交互"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/24/2016"
   wacn.date="02/24/2017"
   ms.author="subramar"/>  



# 使用 Azure CLI 来与 Service Fabric 群集交互

可以在 Linux 计算机上使用“Linux 上的 Azure CLI”来与 Service Fabric 群集交互。

第一步是通过 git rep 获取最新版本的 CLI，然后使用以下命令在路径中设置该版本：


	 git clone https://github.com/Azure/azure-xplat-cli.git
	 cd azure-xplat-cli
	 npm install
	 sudo ln -s \$(pwd)/bin/azure /usr/bin/azure
	 azure servicefabric


对于 CLI 支持的每个命令，可以键入该命令的名称获取相关的帮助。这些命令支持自动补全。例如，使用以下命令可以获取所有应用程序命令的帮助。


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


若要使用命名参数并了解其用途，可以在命令后面键入 --help。例如：


	 azure servicefabric node show --help
	 azure servicefabric application create --help


从**不属于群集**的计算机连接到多计算机群集时，可以使用以下命令：


 	azure servicefabric cluster connect http://PublicIPorFQDN:19080


将 PublicIPorFQDN 标记适当地替换为实际 IP 或 FQDN。从**属于群集**的计算机连接到多计算机群集时，可以使用以下命令：


 	azure servicefabric cluster connect --connection-endpoint http://localhost:19080 --client-connection-endpoint PublicIPorFQDN:19000


可以使用 PowerShell 或 CLI 来与通过 Azure 门户创建的 Linux Service Fabric 群集交互。

**注意：**这些群集不安全，因此，可能会由于在群集清单中添加公共 IP 地址而导致单机系统门户洞开。



## 使用 Azure CLI 连接到 Service Fabric 群集

以下 Azure CLI 命令说明如何连接到安全群集。证书详细信息必须与群集节点上的证书匹配。


	azure servicefabric cluster connect --connection-endpoint http://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert

 
如果证书具有证书颁发机构 (CA)，需要按以下示例所示添加 --ca-cert-path 参数：


 	azure servicefabric cluster connect --connection-endpoint http://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --ca-cert-path /tmp/ca1,/tmp/ca2 

如果有多个 CA，请使用逗号作为分隔符。
 
如果证书中的“公用名”与连接终结点不匹配，可以使用 `--strict-ssl` 参数绕过验证，如以下命令中所示：


	azure servicefabric cluster connect --connection-endpoint http://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --strict-ssl false 

 
如果想要跳过 CA 验证，可以添加 --reject-unauthorized 参数，如以下命令中所示：


	azure servicefabric cluster connect --connection-endpoint http://ip:19080 --client-key-path /tmp/key --client-cert-path /tmp/cert --reject-unauthorized false 

 
连接后，应该能够运行其他 CLI 命令来与群集交互。

## 部署 Service Fabric 应用程序

执行以下命令可以复制、注册和启动 Service Fabric 应用程序：


	azure servicefabric application package copy [applicationPackagePath] [imageStoreConnectionString] [applicationPathInImageStore]
	azure servicefabric application type register [applicationPathinImageStore]
	azure servicefabric application create [applicationName] [applicationTypeName] [applicationTypeVersion]



## 升级应用程序

此过程类似于 [Windows 中的过程](/documentation/articles/service-fabric-application-upgrade-tutorial-powershell/)。

从项目根目录中构建、复制、注册和创建应用程序。如果应用程序实例名为 fabric:/MySFApp 且类型为 MySFApp，则命令如下所示：


	 azure servicefabric cluster connect http://localhost:19080
	 azure servicefabric application package copy MySFApp fabric:ImageStore
	 azure servicefabric application type register MySFApp
	 azure servicefabric application create fabric:/MySFApp MySFApp 1.0


对应用程序进行更改，然后重新构建已修改的服务。使用更新的服务版本（在适当情况下，还包括代码、配置或数据）更新已修改的服务的清单文件 (ServiceManifest.xml)。同时，请使用更新的应用程序版本号和已修改的服务修改应用程序的清单 (ApplicationManifest.xml)。现在，使用以下命令复制并注册已更新的应用程序：


	 azure servicefabric cluster connect http://localhost:19080>
	 azure servicefabric application package copy MySFApp fabric:ImageStore
	 azure servicefabric application type register MySFApp


接下来，可以使用以下命令启动应用程序升级：


	 azure servicefabric application upgrade start -–application-name fabric:/MySFApp -–target-application-type-version 2.0  --rolling-upgrade-mode UnmonitoredAuto


现在，可以使用 SFX 监视应用程序升级。应用程序在几分钟内即可完成更新。还可以使用某个错误操作来尝试测试已更新的应用，检查 Service Fabric 中的自动回滚功能。

##<a name="troubleshooting"></a> 故障排除

### 未成功复制应用程序包

检查是否已安装 `openssh`。默认情况下，Ubuntu Desktop 未安装此软件。使用以下命令安装它：


 	sudo apt-get install openssh-server openssh-client**


如果该问题持续出现，请使用以下命令更改 **sshd\_config** 文件，尝试对 ssh 禁用 PAM：


	 sudo vi /etc/ssh/sshd_config
	#Change the line with UsePAM to the following: UsePAM no
	 sudo service sshd reload


如果问题还是没有解决，请运行以下命令尝试增加 ssh 会话数目：


	 sudo vi /etc/ssh/sshd\_config
	# Add the following to lines:
	# MaxSessions 500
	# MaxStartups 300:30:500
	 sudo service sshd reload

目前不支持使用密钥（而不是密码）进行 ssh 身份验证（因为平台使用 ssh 来复制包），请改用密码身份验证。


## 后续步骤

设置开发环境，并将 Service Fabric 应用程序部署到 Linux 群集。

<!---HONumber=Mooncake_1121_2016-->