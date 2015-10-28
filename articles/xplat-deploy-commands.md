<properties
   pageTitle="使用适用于 Mac、Linux 和 Windows 的 Azure CLI 部署模板"
   description="描述部署或更新模板的基本步骤。"
   services="virtual-machines"
   documentationCenter=""
   authors="squillace"
   manager="timlt"
   editor=""/>

<tags
   ms.service="virtual-machines"
   ms.date="04/21/2015"
   wacn.date="06/26/2015"/>

# 使用适用于 Mac、Linux 和 Windows 的 Azure CLI 部署模板

## 安装 curl、wget 或其他下载工具
本主题使用 **curl**。使用操作系统的程序包管理器，或者从[此处](http://curl.haxx.se/download.html)下载该管理器。

## 下载模板参数文件（或模板，如有必要）

以下步骤可用于部署一个 Azure 模板，即使该模板很复杂。本主题用作示例的模板可通过已安装的自定义脚本 VM 扩展来创建基本的 Ubuntu 服务器。在主题末尾还提供了用作参考的文件。

### 下载 azuredeploy-parameters.json 文件

下载 azuredeploy-parameters.json 文件（如果你要部署的模板存在此文件）。

    curl -O https://github.com/azure/azurermtemplates/raw/master/linux-virtual-machine-with-customdata/azuredeploy-parameters.json

## 输入资源组部署信息

使用你喜爱的编辑器打开此文件。你会看到，你需要为多个密钥指定值，尤其是 **adminUsername**、**adminPassword**（记住复杂性规则！）、你需要的存储帐户名称和 DNS 名称。

    {
      "newStorageAccountName": {
        "value": "uniquestorageaccountname"
      },
      "adminUsername": {
        "value": ""
      },
      "adminPassword": {
        "value": ""
      },
      "dnsNameForPublicIP": {
        "value": "uniquednsnameforpublicip"
      },
      "vmSourceImageName": {
        "value": "b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04_2_LTS-amd64-server-20150309-en-us-30GB"
      },
      "location": {
        "value": "China North"
      },
      "virtualNetworkName": {
        "value": "myVNET"
      },
      "vmSize": {
        "value": "Standard_A0"
      },
      "vmName": {
        "value": "myVM"
      },
      "publicIPAddressName": {
        "value": "myPublicIP"
      },
      "nicName": {
        "value": "myNic"
      }
    }

添加新值（Azure 将尽可能为你创建新的存储和 DNS 资源），或者使用你已创建的资源。下面的 azuredeploy-parameters.json 文件显示了一个示例：




下面的 url 从“空”azuredeploy-parameters.json 获取参数文件，如果使用交互式方法，则这种方法会成功。如果使用下载的自定义参数文件方法，则需改用 --template-file <template-file> 选项。我也编写可提取这些文件任何位置的单个部分的脚本，具体取决于你要执行的操作。你可能需要指出，为了执行 json 分析，可能需要使用 jq: curl http://stedolan.github.io/jq/download/linux64/jq -o /usr/bin/jq


### 部署模板和参数文件


[AZURE.NOTE]你可能会发现，某些模板可能没有相应的 azuredeploy-parameters.json 文件。

需要对参数进行设置，除非这些参数已经是模板自身的一部分，具体取决于你使用的是什么模板。在这些情况下，你可能会

如果你的模板直接包含其参数，或者你需要自行提取参数数据。有关模板结构的详细信息，请参阅 [Azure 资源管理器模板语言](https://msdn.microsoft.com/zh-cn/library/azure/dn835138.aspx)。


https://github.com/azure/azurermtemplates/raw/master/linux-virtual-machine-with-customdata/azuredeploy.json（或者仅 azuredeploy-parameters.json 文件）你可以提取模板的参数部分，这种情况下，你需要在模板中将其保存回去；你也可以将其另存为单独的 azuredeploy-parameters.json 文件。你需要获取将置于参数中的值。

## 修改 azuredeploy-templates.json 文件

收集所需值


## 后续步骤

Vestibul ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nullam ultricies, ipsum vitae volutpat hendrerit, purus diam pretium eros, vitae tincidunt nulla lorem sed turpis: [指向其他 www.windowsazure.cn 文档主题的链接 3](/documentation/articles/storage-whatis-account)。

<!--Image references-->
[5]: ./media/markdown-template-for-new-articles/octocats.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png
[8]: ./media/markdown-template-for-new-articles/copytemplate.png

<!---HONumber=61-->