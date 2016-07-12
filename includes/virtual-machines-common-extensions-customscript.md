
自定义脚本扩展自推出后已广泛应用于在 Windows 和 Linux VM 上配置工作负荷。随着 Azure 资源管理器模板的引入，用户现在可以创建单个模板，该模板不只可以预配 VM，还可以配置 VM 上的工作负荷。

## 关于 Azure 资源管理器模板

Azure 资源管理器模板可让你通过定义资源之间的依赖关系，使用 JSON 语言以声明方式指定 Azure IaaS 基础结构。有关 Azure 资源管理器模板的详细概述，请参阅以下文章：

- [资源组概述](/documentation/articles/resource-group-overview/)
- [使用 Azure Powershell 部署模板](/documentation/articles/virtual-machines-windows-ps-manage/)

### 先决条件

1. 从[此处](/downloads)安装最新的 Azure PowerShell Cmdlet 或 Azure CLI。
2. 如果脚本将在现有 VM 上运行，请确保已在该 VM 上启用了 VM 代理，如果没有启用，请根据 [Linux](/documentation/articles/virtual-machines-linux-classic-manage-extensions/) 或者 [Windows](/documentation/articles/virtual-machines-windows-classic-manage-extensions/) 指引来安装一个。
3. 将你要在 VM 上运行的脚本上载到 Azure 存储。脚本可以来自单个或多个存储容器。
4. 或者，也可以将脚本上载到 Github 帐户。
5. 脚本应当以下述方式编写：使用扩展启动入口脚本，然后入口脚本再调用其他脚本。

## 使用自定义脚本扩展

若要使用模板部署，我们需要使用 Azure 服务管理 API 中可用的同一版本的自定义脚本扩展。该扩展支持用于将文件上载到 Azure 存储帐户或 Github 位置的相同参数和方案。配合模板使用时，主要差异在于应该指定正确版本的扩展，而不是以 majorversion.* 格式指定版本。
