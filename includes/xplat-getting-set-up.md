## <a name="using-azure-cli"></a>使用 Azure CLI
完成以下步骤即可轻松使用包含相应订阅的最新版 Azure CLI。 如果需要安装 Azure CLI 并首先连接到帐户，请参阅[Azure 命令行接口 (Azure CLI)](/documentation/articles/cli-install-nodejs/)。

### <a name="step-1-update-azure-cli-version"></a>步骤 1：更新 Azure CLI 版本
若要将 Azure CLI 用于带服务管理模式的祈使性命令，应尽可能安装最新版。 要验证你的版本，请键入 `azure --version`。 你应看到类似如下的内容：

    $ azure --version
    0.8.17 (node: 0.10.25)

如果想要更新 Azure CLI 版本，请参阅 [Azure CLI](https://github.com/Azure/azure-xplat-cli)。

### <a name="step-2-set-the-azure-account-and-subscription"></a>步骤 2：设置 Azure 帐户和订阅
将 Azure CLI 与要使用的帐户关联后，可能会发现自己有多个订阅。 如果有多个订阅，应通过键入 `azure account list` 来查看帐户可用的订阅，然后通过键入 `azure account set <subscription id or name> true` 来选择要使用的订阅，其中，*subscription id or name* 是要在当前会话中使用的订阅 ID 或订阅名称。 你会看到下面这样的内容：

    $ azure account set "Visual Studio Ultimate with MSDN" true
    info:    Executing command account set
    info:    Setting subscription to "Visual Studio Ultimate with MSDN" with id "0e220bf6-5caa-4e9f-8383-51f16b6c109f".
    info:    Changes saved
    info:    account set command OK

> [AZURE.NOTE]
> 如果还没有 Azure 帐户，但有 MSDN 订阅的订阅，则可以使用试用版。 这两种方式都可以用来进行 Azure 访问。
> 
>

