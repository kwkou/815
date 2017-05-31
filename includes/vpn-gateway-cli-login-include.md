使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。 有关登录的详细信息，请参阅 [Azure CLI 2.0 入门](https://docs.microsoft.com/zh-cn/cli/azure/get-started-with-azure-cli)。

    az login

> [AZURE.NOTE]
> 在 Azure 中国区使用 Azure CLI 2.0 之前，需修改 Azure 配置文件。 运行 `az configure` 即可查看配置文件的路径，该路径在 Windows 中类似于 `C:\Users\<user name>\.azure\config`，在 Linux 中类似于 `/var/users/<username>/.azure/config`。 打开文件，将 `AzureCloud` 替换为 `AzureChinaCloud`。

如果有多个 Azure 订阅，请列出该帐户的订阅。

    az account list --all

指定要使用的订阅。

    az account set --subscription <replace_with_your_subscription_id>