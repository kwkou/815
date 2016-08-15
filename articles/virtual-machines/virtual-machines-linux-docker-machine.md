<properties
	pageTitle="使用 Docker 计算机在 Azure 中创建 Docker 主机 | Azure"
	description="介绍如何使用 Docker 计算机在 Azure 中创建 Docker 主机。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/20/2016"
	wacn.date="06/13/2016"/>

# 通过 Azure 驱动程序使用 Docker 计算机

[Docker](https://www.docker.com/) 是最常用的虚拟化技术之一，它使用 Linux 容器而不是虚拟机作为在共享资源上隔离应用程序数据和执行计算的方法。本主题介绍何时及如何使用 [Docker 计算机](https://docs.docker.com/machine/)（`docker-machine` 命令）在 Azure 中创建新的 Linux VM，并将其启用为 Linux 容器的 Docker 主机。


## 使用 Docker 计算机创建 VM

使用驱动程序选项 (`-d`) 的 `azure` 驱动程序参数和任何其他参数，以 `docker-machine create` 命令在 Azure 中创建 Docker 主机 VM。

以下示例依赖于默认值，但它确实打开了连接 Internet 的 VM 端口 80 以使用 nginx 容器来进行测试、让 `ops` 成为 SSH 的登录用户，以及调用新的 VM `machine`。

键入 `docker-machine create --driver azure` 查看选项及其默认值；你也可以参阅 [Docker Azure 驱动程序文档](https://docs.docker.com/machine/drivers/azure/)。（请注意，如果启用了双重身份验证，系统将提示你使用第二个因素进行身份验证）。

	docker-machine create -d azure \
	  --azure-ssh-user ops \
	  --azure-subscription-id <Your AZURE_SUBSCRIPTION_ID> \
	  --azure-open-port 80 \
	  machine

输出应如下所示，具体取决于你的帐户中是否配置了双重身份验证。

	Creating CA: /Users/user/.docker/machine/certs/ca.pem
	Creating client certificate: /Users/user/.docker/machine/certs/cert.pem
	Running pre-create checks...
	(machine) Azure: To sign in, use a web browser to open the page https://aka.ms/devicelogin. Enter the code <code> to authenticate.
	(machine) Completed machine pre-create checks.
	Creating machine...
	(machine) Querying existing resource group.  name="machine"
	(machine) Creating resource group.  name="machine" location="chinaeast"
	(machine) Configuring availability set.  name="docker-machine"
	(machine) Configuring network security group.  name="machine-firewall" location="chinaeast"
	(machine) Querying if virtual network already exists.  name="docker-machine-vnet" location="chinaeast"
	(machine) Configuring subnet.  name="docker-machine" vnet="docker-machine-vnet" cidr="192.168.0.0/16"
	(machine) Creating public IP address.  name="machine-ip" static=false
	(machine) Creating network interface.  name="machine-nic"
	(machine) Creating storage account.  name="vhdsolksdjalkjlmgyg6" location="chinaeast"
	(machine) Creating virtual machine.  name="machine" location="chinaeast" size="Standard_A2" username="ops" osImage="canonical:UbuntuServer:15.10:latest"
	Waiting for machine to be running, this may take a few minutes...
	Detecting operating system of created instance...
	Waiting for SSH to be available...
	Detecting the provisioner...
	Provisioning with ubuntu(systemd)...
	Installing Docker...
	Copying certs to the local machine directory...
	Copying certs to the remote machine...
	Setting Docker configuration on the remote daemon...
	Checking connection to Docker...
	Docker is up and running!
	To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env machine

## 配置 Docker 外壳

现在请键入 `docker-machine env <VM name>`，查看需要执行哪些操作来配置外壳。

	docker-machine env machine

这会列显如下所示的环境信息。请注意，IP 地址已分配，需要使用它来测试 VM。

	export DOCKER_TLS_VERIFY="1"
	export DOCKER_HOST="tcp://191.237.46.90:2376"
	export DOCKER_CERT_PATH="/Users/rasquill/.docker/machine/machines/machine"
	export DOCKER_MACHINE_NAME="machine"
	# Run this command to configure your shell:
	# eval $(docker-machine env machine)

你可以运行建议的配置命令，或自行设置环境变量。

## 运行容器

现在，你可以运行一台简单的 Web 服务器来测试所有组件是否正常工作。我们在此处使用了标准的 nginx 映像，指定它应侦听端口 80，并且如果 VM 重新启动，则容器也应该重新启动 (`--restart=always`)。

	docker run -d -p 80:80 --restart=always nginx

输入应如下所示：

	Unable to find image 'nginx:latest' locally
	latest: Pulling from library/nginx
	efd26ecc9548: Pull complete
	a3ed95caeb02: Pull complete
	83f52fbfa5f8: Pull complete
	fa664caa1402: Pull complete
	Digest: sha256:12127e07a75bda1022fbd4ea231f5527a1899aad4679e3940482db3b57383b1d
	Status: Downloaded newer image for nginx:latest
	25942c35d86fe43c688d0c03ad478f14cc9c16913b0e1c2971cb32eb4d0ab721

## 测试容器

使用 `docker ps` 检查正在运行的容器：

	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
	d5b78f27b335        nginx               "nginx -g 'daemon off"   5 minutes ago       Up 5 minutes        0.0.0.0:80->80/tcp, 443/tcp   goofy_mahavira

检查正在运行的容器，键入 `docker-machine ip <VM name>` 以查找 IP 地址（如果你忘记了 `env` 命令返回的地址）：

![运行 ngnix 容器](./media/virtual-machines-linux-docker-machine/nginxsuccess.png)

## 后续步骤

如有兴趣，你可以试用 [Azure Docker VM 扩展](/documentation/articles/virtual-machines-linux-dockerextension/)，通过 Azure CLI 或 Azure Resource Manager 模板执行相同的操作。


<!---HONumber=Mooncake_0606_2016-->