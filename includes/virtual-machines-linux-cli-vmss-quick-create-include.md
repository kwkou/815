## <a name="prerequisites"></a>先决条件

获取 [Azure 订阅试用版](/pricing/1rmb-trial/)并安装 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2)（如果尚未这样做）。

## <a name="create-the-scale-set"></a>创建规模集

首先，创建一个要将规模集部署到其中的资源组：

    az group create --location chinanorth --name myResourceGroup

现在，使用 `az vmss create` 命令创建规模集。 以下示例在名为 `myrg` 的资源组中创建名为 `myvmss` 的 Linux 规模集：

    az vmss create --resource-group myResourceGroup --name myVmss \
        --image UbuntuLTS --admin-username azureuser \
        --authentication-type password --admin-password P4$$w0rd \
        --use-unmanaged-disk

以下示例将创建具有相同配置的 Windows 规模集：

    az vmss create --resource-group myResourceGroup --name myVmss \
        --image Win2016Datacenter --admin-username azureuser \
        --authentication-type password --admin-password P4$$w0rd \
        --use-unmanaged-disk

如果想选择不同的操作系统映像，可使用命令 `az vm image list` 或 `az vm image list --all` 查看可用映像。 若要查看规模集中 VM 的连接信息，请使用命令 `az vmss list_instance_connection_info`：

    az vmss list_instance_connection_info --resource-group myResourceGroup --name myVmss