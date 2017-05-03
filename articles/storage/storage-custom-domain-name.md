<properties
    pageTitle="为 Azure Blob 存储终结点配置自定义域名 | Azure"
    description="使用 Azure 门户预览将自己的规范名称 (CNAME) 映射到 Azure 存储帐户中的 Blob 存储终结点。"
    services="storage"
    documentationcenter=""
    author="mmacy"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="aaafd8c5-eacb-49dc-8c8b-3f7011ad5e92"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/17/2017"
    wacn.date="05/02/2017"
    ms.author="marsma"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="d7c928a77ca16f68d187953dfb876b6feb4c9321"
    ms.lasthandoff="04/22/2017" />

# <a name="configure-a-custom-domain-name-for-your-blob-storage-endpoint"></a>为 Blob 存储终结点配置自定义域名

你可以配置自定义域以便访问 Azure 存储帐户中的 Blob 数据。 Blob 存储的默认终结点是 `<storage-account-name>.blob.core.chinacloudapi.cn`。 如果将自定义域和子域（例如 **www.contoso.com**）映射到存储帐户的 Blob 终结点，则用户可以使用该域访问存储帐户中的 Blob 数据。 

> [AZURE.IMPORTANT]
> Azure 存储尚不支持对自定义域使用 HTTPS。 尽管我们暂时无法提供可以共享的具体时间表，但我们知道客户对此功能非常感兴趣。
>

下表显示了 **mystorageaccount** 存储帐户中的 Blob 数据的一些示例 URL。 为存储帐户注册的自定义域是 **www.contoso.com**：

| 资源类型 | 默认 URL | 自定义域 URL |
| --- | --- | --- |
| 存储帐户 | http://mystorageaccount.blob.core.chinacloudapi.cn | http://www.contoso.com |
| Blob |http://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myblob | http://www.contoso.com/mycontainer/myblob |
| 根容器 | http://mystorageaccount.blob.core.chinacloudapi.cn/myblob 或 http://mystorageaccount.blob.core.chinacloudapi.cn/$root/myblob| http://www.contoso.com/myblob 或 http://www.contoso.com/$root/myblob |

## <a name="direct-vs-intermediary-domain-mapping"></a>直接域映射与中间域映射

可以通过两种方法将自定义域映射到存储帐户的 Blob 终结点：直接 CNAME 映射和使用 asverify 中间子域。

### <a name="direct-cname-mapping"></a>直接 CNAME 映射

第一种方法是创建一个规范名称 (CNAME) 记录。此记录可以将自定义域和子域直接映射到 Blob 终结点。这种方法最简单。 CNAME 记录是一种域名系统 (DNS) 功能，用于将源域映射到目标域。 在本示例中，源域是自定义域和子域，例如 www.contoso.com。 目标域是 Blob 服务终结点，例如 mystorageaccount.blob.core.chinacloudapi.cn。

[注册自定义域](#register-a-custom-domain)中介绍了直接方法。

### <a name="intermediary-mapping-with-asverify"></a>使用 asverify 进行中间映射

第二种方法同样使用 CNAME 记录，但是先采用经 Azure 识别的特殊子域，以避免停机时间：**asverify**。

将自定义域映射到 Blob 终结点的过程会导致在 [Azure 门户预览](https://portal.azure.cn)中注册域时出现短暂的停机时间。 如果自定义域目前所支持的应用程序的服务级别协议 (SLA) 要求不能有故障时间，则可以使用 Azure asverify 子域作为中间注册步骤。 此中间步骤可确保用户能够在 DNS 映射进行时访问你的域。

[使用 *asverify* 子域注册自定义域](#register-a-custom-domain-using-the-asverify-subdomain)中介绍了中间方法。

## <a name="register-a-custom-domain"></a>注册自定义域
如果不担心域暂时对用户不可用，或者自定义域当前未托管应用程序，则可以使用本过程注册自定义域。

如果自定义域目前在支持不能有任何停机时间的应用程序，则请按照[使用 *asverify* 子域注册自定义域](#register-a-custom-domain-using-the-asverify-subdomain)中介绍的过程进行操作。

若要配置自定义域名，必须在 DNS 中创建一个新的 CNAME 记录。 该 CNAME 记录指定了域名的别名。 在本示例中，它会将自定义域的地址映射到存储帐户的 Blob 存储终结点。

通常情况下，可以在域注册机构的网站上管理域的 DNS 设置。 每个注册机构指定 CNAME 记录的方法类似但略有不同，但概念是相同的。 某些基本域注册程序包不提供 DNS 配置，因此可能需要首先升级域注册程序包，然后才能创建 CNAME 记录。

1. 在 [Azure 门户预览](https://portal.azure.cn)中导航到存储帐户。
1. 在菜单边栏选项卡上的“BLOB 服务”下，选择“自定义域”以打开“自定义域”边栏选项卡。
1. 登录到域注册机构的网站，然后转到用于管理 DNS 的页面。 可能会在“**域名**”、“**DNS**”或“**名称服务器管理**”等部分中找到此页。
1. 找到用于管理 CNAME 的部分。 你可能需要转至高级设置页面，并找到“**CNAME**”、“**别名**”或“**子域**”字样。
1. 创建一个新的 CNAME 记录，并且提供子域别名，例如 **www** 或 **photos**。 然后以 **mystorageaccount.blob.core.chinacloudapi.cn** 格式（其中，*mystorageaccount* 是存储帐户的名称）提供主机名，这是 Blob 服务终结点。 要使用的主机名将显示在 [Azure 门户预览](https://portal.azure.cn)中“自定义域”边栏选项卡的第 1 项中。
1. 在 [Azure 门户预览](https://portal.azure.cn)的“自定义域”边栏选项卡上的文本框中，输入自定义域的名称，包括子域。 例如，如果域是 **contoso.com**，子域别名是 **www**，请输入 **www.contoso.com**。 如果子域是 **photos**，请输入 **photos.contoso.com**。 必须输入子域。
1. 在“自定义域”边栏选项卡上选择“保存”注册自定义域。 如果注册成功，你将看到一条消息，提示存储帐户已成功更新。

新的 CNAME 记录通过 DNS 传播后，只要用户具有适当的权限，他们就可以通过使用你的自定义域查看 Blob 数据。

## <a name="register-a-custom-domain-using-the-asverify-subdomain"></a>使用 asverify 子域注册自定义域
如果你的自定义域当前所支持的应用程序的 SLA 要求没有停机时间，则使用此过程注册你的自定义域。 可以通过创建从 `asverify.<subdomain>.<customdomain>` 指向 `asverify.<storageaccount>.blob.core.chinacloudapi.cn` 的CNAME 使用 Azure 预注册域。 然后，可以创建第二个从 `<subdomain>.<customdomain>` 指向 `<storageaccount>.blob.core.chinacloudapi.cn` 的 CNAME，指向自定义域的通信量将被定向到 Blob 终结点。

**asverify** 子域是 Azure 能够识别的一个特殊子域。 将 `asverify` 追加到你自己的子域，这样可以使 Azure 能够识别自定义域且不需要修改该域的 DNS 记录。 修改该域的 DNS 记录后，它将映射到 Blob 终结点，并且没有停机时间。

1. 在 [Azure 门户预览](https://portal.azure.cn)中导航到存储帐户。
1. 在菜单边栏选项卡上的“BLOB 服务”下，选择“自定义域”以打开“自定义域”边栏选项卡。
1. 登录到 DNS 提供程序的网站，然后转到用于管理 DNS 的页面。 可能会在“**域名**”、“**DNS**”或“**名称服务器管理**”等部分中找到此页。
1. 找到用于管理 CNAME 的部分。 你可能需要转至高级设置页面，并找到“**CNAME**”、“**别名**”或“**子域**”字样。
1. 创建一个新的 CNAME 记录，并且提供包括 *asverify* 子域的子域别名。 例如，**asverify.www** 或 **asverify.photos**。 然后提供主机名，这是你的 Blob 服务终结点，格式为 **asverify.mystorageaccount.blob.core.chinacloudapi.cn**（其中 **mystorageaccount** 是存储帐户的名称）。 要使用的主机名将显示在 [Azure 门户预览](https://portal.azure.cn)中“自定义域”边栏选项卡的第 2 项中。
1. 在 [Azure 门户预览](https://portal.azure.cn)的“自定义域”边栏选项卡上的文本框中，输入自定义域的名称，包括子域。 请不要包含 asverify 例如，如果域是 **contoso.com**，子域别名是 **www**，请输入 **www.contoso.com**。 如果子域是 **photos**，请输入 **photos.contoso.com**。 必须输入子域。
1. 选择“使用间接 CNAME 验证”复选框。
1. 在“自定义域”边栏选项卡上选择“保存”注册自定义域。 如果注册成功，你将看到一条消息，提示存储帐户已成功更新。 此时，你的自定义域已由 Azure 进行了验证，但传输到你的域的流量尚未路由到你的存储帐户。
1. 返回到 DNS 提供程序的网站，创建将子域映射到 Blob 服务终结点的另一条 CNAME 记录。 例如，将子域指定为 **www** 或 **photos**（不含 *asverify*），将主机名指定为 **mystorageaccount.blob.core.chinacloudapi.cn**（其中，**mystorageaccount** 是存储帐户名称）。 完成此步骤后，也就完成了你的自定义域的注册。
1. 最后，可以删除之前创建的含 **asverify** 的 CNAME 记录，因为只需要将其作为中间步骤使用。

新的 CNAME 记录通过 DNS 传播后，只要用户具有适当的权限，他们就可以通过使用你的自定义域查看 Blob 数据。

## <a name="test-your-custom-domain"></a>测试自定义域

若要确认自定义域是否确实映射到了 Blob 服务终结点，请在存储帐户中的公共容器内创建一个 Blob。 然后在 Web 浏览器中，使用以下格式的 URI 来访问该 Blob：

`http://<subdomain.customdomain>/<mycontainer>/<myblob>`

例如，可以使用以下 URI 来访问 **photos.contoso.com** 自定义子域中的 **myforms** 容器中的 Web 窗体：

`http://photos.contoso.com/myforms/applicationform.htm`

## <a name="deregister-a-custom-domain"></a>取消注册自定义域

若要取消注册 Blob 存储终结点的自定义域，请使用以下过程之一。

### <a name="azure-cli-20"></a>Azure CLI 2.0

使用 [az storage account update](https://docs.microsoft.com/zh-cn/cli/azure/storage/account#update) CLI 命令，并为 `--custom-domain` 参数值指定一个空字符串 (`""`) 以删除自定义域注册。

* 命令格式：

          az storage account update \
              --name <storage-account-name> \
              --resource-group <resource-group-name> \
              --custom-domain ""

* 命令示例：

          az storage account update \
              --name mystorageaccount \
              --resource-group myresourcegroup \
              --custom-domain ""
    
### <a name="powershell"></a>PowerShell

使用 [Set-AzureRmStorageAccount](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.storage/Set-AzureRmStorageAccount) PowerShell cmdlet ，并为 `-CustomDomainName` 参数值指定一个空字符串 (`""`) 以删除自定义域注册。

* 命令格式：

          Set-AzureRmStorageAccount `
              -ResourceGroupName "<resource-group-name>" `
              -AccountName "<storage-account-name>" `
              -CustomDomainName ""

* 命令示例：

          Set-AzureRmStorageAccount `
              -ResourceGroupName "myresourcegroup" `
              -AccountName "mystorageaccount" `
              -CustomDomainName ""

### <a name="azure-portal-preview"></a>Azure 门户预览

目前无法使用 Azure 门户预览删除自定义域注册。 这是已知问题。 目前我们暂时无法提供解决方案日期，但问题解决后我们会更新本文。 在此之前，请使用 Azure CLI 2.0 或 Azure PowerShell 删除自定义域设置。
<!--Update_Description:update classic portal operations to Ibiza portal;add azure cli 2.0 commands-->
