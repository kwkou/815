<properties services="virtual-machines" title="Setting up Azure CLI for service management" authors="squillace" solutions="" manager="timlt" editor="tysonn" />

<tags
   ms.service="virtual-machine"
   ms.date="04/13/2015"
   wacn.date="03/17/2016" />

## 使用 Azure CLI

完成以下步骤即可轻松使用包含相应订阅的最新版 Azure CLI。如果你需要安装 Azure CLI 并首先连接到你的帐户，请参阅[Azure 命令行接口 (Azure CLI)](/documentation/articles/xplat-cli-install/)。

### 步骤 1：更新 Azure CLI 版本

若要将 Azure CLI 用于带服务管理模式的祈使性命令，应尽可能安装最新版。要验证你的版本，请键入 `azure --version`。你应看到类似如下的内容：

    $ azure --version
    0.8.17 (node: 0.10.25)

如果想要更新 Azure CLI 版本，请参阅 [Azure CLI](https://github.com/Azure/azure-xplat-cli)。

### 步骤 2：设置 Azure 帐户和订阅

将 Azure CLI 与要使用的帐户连接后，你可能会有多个订阅。如果你有多个订阅，则需通过键入 `azure account list` 来查看你帐户可用的订阅，并通过键入 `azure account set <subscription id or name> true` 选择要使用的订阅。_订阅 ID 或名称_是你要在当前会话中使用的订阅 ID 或订阅名称。你会看到下面这样的内容：

    $ azure account set "Visual Studio Ultimate with MSDN" true
    info:    Executing command account set
    info:    Setting subscription to "Visual Studio Ultimate with MSDN" with id "0e220bf6-5caa-4e9f-8383-51f16b6c109f".
    info:    Changes saved
    info:    account set command OK

<!---HONumber=Mooncake_0307_2016-->