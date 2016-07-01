<properties 
  pageTitle="在 Azure 上部署自己的私有 Docker 注册表 | Azure"
  description="介绍如何使用 Docker 注册表在 Azure Blob 存储服务上托管你的容器映像。"
  services="virtual-machines"
  documentationCenter="virtual-machines-linux"
  authors="ahmetalpbalkan"
  editor="squillace"
  manager="" 
  tags="azure-service-management,azure-resource-manager" />

<tags
  ms.service="virtual-machines-linux"
  ms.date="04/19/2016" 
  wacn.date="06/29/2016" />

# 在 Azure 上部署自己的私有 Docker 注册表

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-include.md)]


本文档描述什么是 Docker 私有注册表，并说明如何使用 Azure Blob 存储将 Docker 注册表 2.0 容器映像部署到 Azure 上的 Docker 私有注册表。

本文档假设：

1. 你知道如何使用 Docker 并具有要存储的 Docker 映像。（你不知道？ [了解 Docker](https://www.docker.com)）


## 什么是私有 Docker 注册表？

若要将容器化应用程序传送到云，你需要构建 Docker 容器映像，并将其存储到某个位置，供自己和他人使用。

创建容器映像并将其传送到云的过程十分轻松，但要可靠地存储生成的映像则是个难题。因此，Docker 提供了一个称为 [Docker Hub][docker-hub] 的集中式服务，可将容器映像存储在云中，并让你随时使用这些映像创建容器。

尽管 [Docker Hub][docker-hub] 是存储私有应用程序容器映像的付费服务，但 Docker 仍会遵循开发人员的需求并提供开放源代码工具集，用于在防火墙后面或本地你自己的私有 Docker 注册表中存储映像，而无需使用公共 Internet。由于可以轻松保护 Azure Blob 存储，因此你可以快速地使用它在自行控制的 Azure 中创建并使用私有 Docker 注册表。

## 为何应在 Azure 上托管 Docker 注册表？

通过在 Azure 上托管 Docker 注册表实例并将映像存储在 Azure Blob 存储上，可以获得多种好处：

**安全性：**Docker 映像不会离开 Azure 数据中心，因此它们不会像使用 Docker Hub 时一样跨公共 Internet。
  
**性能：**Docker 映像存储在与应用程序相同的数据中心或区域内。这意味着，可以比 Docker Hub 更快、更可靠地提取映像。

**可靠性：**通过使用 Azure Blob 存储，你可以利用许多存储属性，例如高可用性、冗余、高级存储 (SSD) 等等。

## 将 Docker 注册表配置为使用 Azure Blob 存储

（继续下一步之前，建议你先阅读 [Docker 注册表 2.0 文档][registry-docs]。）

你可以使用两种不同的方式[配置][registry-config] Docker 注册表。你可以：

1. 使用 `config.yml` 文件。在此情况下，必须在 `registry` 映像顶部创建独立的 Docker 映像。
2. 通过环境变量重写默认配置文件：在不创建和维护单独 Docker 映像的情况下完成操作。

为简单起见，本主题遵循选项 2，即使用环境变量。

若要运行 Docker 注册表实例：
* 使用 Azure 存储帐户存储映像 
* 侦听虚拟机的通信端口 5000 
* 未配置验证（不建议，请参阅以下注释）

需要在 bash 终端中运行以下 Docker 命令（将 `<storage-account>` 和 `<storage-key>` 和替换为你的凭据）：


	$ docker run -d -p 5000:5000 \
	     -e REGISTRY_STORAGE=azure \
	     -e REGISTRY_STORAGE_AZURE_ACCOUNTNAME="<storage-account>" \
	     -e REGISTRY_STORAGE_AZURE_ACCOUNTKEY="<storage-key>" \
	     -e REGISTRY_STORAGE_AZURE_CONTAINER="registry" \
	     --name=registry \
	     registry:2
	

命令退出后，你可以通过在主机上运行 `docker ps` 命令，查看托管私有 Docker 注册表实例的容器：


	$ docker ps
	CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                    NAMES
	3698ddfebc6f        registry:2          "registry cmd/regist   2 seconds ago       Up 1 seconds        0.0.0.0:5000->5000/tcp   registry
	

> [AZURE.IMPORTANT]本文档未涵盖配置 Docker 注册表安全性的操作，如果打开连接到虚拟机终结点上注册表端口的端口，则默认情况下，任何未经身份验证的用户都可以访问注册表；如果使用上述部署命令，则可以访问负载平衡器。
> <p>请参阅[配置 Docker 注册表][registry-config]文档，以了解如何保护注册表实例和映像。

## 后续步骤

设置注册表之后，你可以充分地利用它。首先从 docker [registry-docs] 开始。

[docker-hub]: https://hub.docker.com/
[registry]: https://github.com/docker/distribution
[registry-docs]: http://docs.docker.com/registry/
[registry-config]: http://docs.docker.com/registry/configuration/
 

<!---HONumber=79-->