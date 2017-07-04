## 了解 Azure 资源模板和资源组

大多数部署和运行在 Azure 中的应用程序是通过不同云资源类型的组合（例如，一个或多个 VM 和存储帐户、一个 SQL 数据库、一个虚拟网络、一个 CDN，等等）生成的。[Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)使你能够集中部署和管理这些不同的资源，只需对资源和关联的配置及部署参数进行 JSON 描述即可。

订阅基于 JSON 的资源模板之后，你就可以执行它，并使用 PowerShell 命令将其中定义的资源部署到 Azure 中。你可以在 PowerShell 命令外壳中单独运行此 PowerShell 命令，也可以将其集成到包含其他自动化逻辑的 PowerShell 脚本中。

使用 Azure 资源管理器模板创建的资源将部署到新的或现有的 Azure 资源组。Azure 资源组允许你以单个逻辑组的形式同时管理多个已部署的资源。通常，一个组将包含与特定应用程序相关的资源。Azure 资源组提供了一种方法来管理组/应用程序的整个生命周期，并提供管理 API 来让你一次性停止/启动/删除组内的所有资源、应用基于角色的访问控制 (RBAC) 规则来锁定其中的安全权限、审核操作，以及使用其他元数据来标记资源以方便跟踪。若要了解有关 Azure 资源组的详细信息，请参阅 [Azure 资源管理器概述](/documentation/articles/resource-group-overview/)。

下面的自动化示例演示了如何使用 Azure 资源管理器模板，以及如何使用 PowerShell 或 CLI 来部署资源组。

<!---HONumber=Mooncake_1207_2015-->