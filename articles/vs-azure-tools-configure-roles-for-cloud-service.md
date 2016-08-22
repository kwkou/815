<properties
   pageTitle="使用 Visual Studio 配置 Azure 云服务的角色 | Azure"
   description="了解如何使用 Visual Studio 设置和配置 Azure 云服务的角色。"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />  

<tags
   ms.service="multiple"
   ms.date="06/01/2016"
   wacn.date="08/22/2016" />

# 使用 Visual Studio 配置 Azure 云服务的角色

一个 Azure 云服务可以有一个辅助角色或 Web 角色。对于每个角色，需要定义该角色的设置方式，并配置该角色的运行方式。若要详细了解云服务中的角色，请观看视频 [Introduction to Azure Cloud Services（Azure 云服务简介）](https://channel9.msdn.com/Series/Windows-Azure-Cloud-Services-Tutorials/Introduction-to-Windows-Azure-Cloud-Services)。云服务信息存储在以下文件中：

- **ServiceDefinition.csdef**

    服务定义文件定义了云服务的运行时设置，包括需要什么角色、终结点和虚拟机大小。当角色正在运行时，无法更改此文件中存储的任何数据。

- **ServiceConfiguration.cscfg**

    服务配置文件则配置了角色有多少实例在运行以及为角色定义的设置的值。当角色正在运行时，可以更改此文件中存储的数据。

若要能够存储角色运行方式设置的不同值，可选择多种服务配置。对于每个部署环境可使用不同的服务配置。例如，可以在本地服务配置中设置存储帐户连接字符串以使用本地 Azure 存储模拟器并在云中创建另一个服务配置以使用 Azure 存储空间。

在 Visual Studio 中创建一个新的 Azure 云服务时，默认情况下将创建两个服务配置。这些配置将添加到 Azure 项目中。命名的配置：

- ServiceConfiguration.Cloud.cscfg

- ServiceConfiguration.Local.cscfg

## 配置 Azure 云服务

可从 Visual Studio 中的解决方案资源管理器配置 Azure 云服务，如下图所示。

![配置云服务](./media/vs-azure-tools-configure-roles-for-cloud-service/IC713462.png)

### 配置 Azure 云服务

1. 若要从“解决方案资源管理器”配置 Azure 项目的每个角色，请在 Azure 项目打开角色的快捷菜单，然后选择“属性”。

    包含角色名称的页将显示在 Visual Studio 编辑器中。该页面显示“配置”选项卡的字段。

1. 在“服务配置”列表中，选择你要编辑的服务配置名称。

    如果要对角色的所有服务配置进行更改，则可选择“所有配置”。

    >[AZURE.IMPORTANT] 如果要选择特定的服务配置，需禁用一些属性，因为其只能设置为所有配置。若要编辑这些属性，必须选择“所有配置”。

    现在可选择选项卡以更新在该视图上任意启动的属性。

## 更改角色实例的数目

若要提高云服务的性能，可根据用户或某个特定角色的预期负载的数目，更改正在运行的角色实例的数目。当云服务在 Azure 中运行时，将为每个角色实例创建单独的虚拟机。这将会影响部署此云服务的计费。有关计费的详细信息，请参阅[了解你的 Azure 帐单](/documentation/articles/billing-understand-your-bill/)。

### 更改角色的实例数目

1. 选择“配置”选项卡。

1. 在“服务配置”列表中，选择你要更新的服务配置。

    >[AZURE.NOTE] 可为特定的服务配置或所有的配置设置实例计数。

1. 在“实例计数”文本框中，键入想要启动此角色的实例数。

    >[AZURE.NOTE] 当你将云服务发布到 Azure 时，每个实例将在单独的虚拟机上运行。

1. 选择工具栏中的“保存”按钮，将这些更改保存到服务配置文件。

## 管理存储帐户的连接字符串

可添加、删除或修改服务配置的连接字符串。对于不同的服务的配置，你可能需要不同的连接字符串。例如，你可能希望本地服务配置的本地连接字符串，该本地服务配置具有 `UseDevelopmentStorage=true` 值。你可能还希望将云服务配置为使用 Azure 中的存储帐户。

>[AZURE.WARNING] 当输入存储帐户连接字符串的 Azure 存储帐户关键信息时，此信息存储在本地服务配置文件中。但是，此信息当前未存储为加密文本。

由于每个服务配置使用不同的值，当你将云服务发布到 Azure 时，不必在云服务中使用不同的连接字符串或修改你的代码。可以在代码中对连接字符串使用同一名称，但值会不同，该值基于在生成云服务或发布云服务时选择的服务配置。

### 管理存储帐户的连接字符串

1. 选择“设置”选项卡。

1. 在“服务配置”列表中，选择你要更新的服务配置。

    >[AZURE.NOTE] 可更新特定服务配置的连接字符串，但是，如果需要添加或删除连接字符串，则必须选择“所有配置”。

1. 若要添加连接字符串，请选择“添加设置”按钮。一个新条目将添加到列表中。

1. 在“名称”文本框中，键入你想要用于连接字符串的名称。

1. 在“类型”下拉列表中，选择“连接字符串”。

1. 若要更改连接字符串的值，请选择省略号 (...) 按钮。此时将显示“创建存储连接字符串”对话框。

1. 若要使用本地存储帐户模拟器，请选择“Azure 存储模拟器”选项按钮，然后选择“确定”按钮。

1. 若要使用 Azure 中的存储帐户，请选择“你的订阅”选项并选择所需的存储帐户。

1. 若要使用自定义凭据，请选择“手动输入的凭据”选项按钮。输入存储帐户名称和主密钥或辅助密钥。有关如何创建存储帐户以及如何在“创建存储连接字符串”对话框中输入存储帐户详细信息的更多信息，请参阅[准备从 Visual Studio 发布或部署 Azure 应用程序](/documentation/articles/vs-azure-tools-cloud-service-publish-set-up-required-services-in-visual-studio/)。

1. 若要删除某个连接字符串，请选择该连接字符串，然后选择“删除设置”按钮。

1. 选择工具栏中的“保存”图标，将这些更改保存到服务配置文件。

1. 若要访问服务配置文件中的连接字符串，必须获取配置设置的值。以下代码显示了这样一个示例：当用户从 Azure 云服务 Web 角色的 Default.aspx 页中选择 **Button1** 时，将使用服务配置文件中的连接字符串 `MyConnectionString` 创建 blob 存储区并上载数据。将以下 using 语句添加到 Default.aspx.cs：

    ```
    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.ServiceRuntime;
    ```

1. 在设计视图中打开 Default.aspx.cs，然后从工具箱添加一个按钮。将以下代码添加到 `Button1_Click` 方法中。此代码使用 `GetConfigurationSettingValue` 为连接字符串获取服务配置文件中的值。然后，存储帐户中创建一个连接字符串 `MyConnectionString` 引用的 Blob。最后，程序将文本添加到该 Blob。

    ```
    protected void Button1_Click(object sender, EventArgs e)
    {
        // Setup the connection to Azure Storage
        var storageAccount = CloudStorageAccount.Parse(RoleEnvironment.GetConfigurationSettingValue("MyConnectionString"));
        var blobClient = storageAccount.CreateCloudBlobClient();
        // Get and create the container
        var blobContainer = blobClient.GetContainerReference("quicklap");
        blobContainer.CreateIfNotExists();
        // upload a text blob
        var blob = blobContainer.GetBlockBlobReference(Guid.NewGuid().ToString());
        blob.UploadText("Hello Azure");

    }
    ```

## 将要使用的自定义设置添加到 Azure 云服务中

服务配置文件中的自定义设置可让你为特定服务配置的字符串添加名称和值。你可以选择使用此设置来配置云服务中的功能，具体方法是：读取设置的值，然后使用该值来控制你的代码中的逻辑。无需重新生成服务包就可更改这些服务配置值；运行云服务时也可以进行更改。设置发生更改时，你的代码可以检查通知。请参阅 [RoleEnvironment.Changing 事件](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.changing.aspx)。

可添加、移除或修改服务配置的自定义设置。对于不同的服务的配置，你可能需要这些字符串的不同值。

由于每个服务配置使用不同的值，当你将云服务发布到 Azure 时，不必在云服务中使用不同的字符串或修改你的代码。可以在代码中对字符串使用同一名称，但值会不同，该值基于在生成云服务或发布云服务时选择的服务配置。

### 将要使用的自定义设置添加到 Azure 云服务

1. 选择“设置”选项卡。

1. 在“服务配置”列表中，选择你要更新的服务配置。

    >[AZURE.NOTE] 可更新特定服务配置的字符串，但是，如果需要添加或删除字符串，则必须选择“所有配置”。

1. 若要添加字符串，请选择“添加设置”按钮。一个新条目将添加到列表中。

1. 在“名称”文本框中，键入你想要用于字符串的名称。

1. 在“类型”下拉列表中，选择“字符串”。

1. 若要添加或更改字符串的值，请在“值”文本框中键入新值。

1. 若要删除某个字符串，请选择该字符串，然后选择“删除设置”按钮。

1. 选择工具栏中的“保存”按钮，将这些更改保存到服务配置文件。

1. 若要访问服务配置文件中的字符串，必须获取配置设置的值。

    需确保将以下 using 语句添加到 Default.aspx.cs，如前一过程中所述：

    ```
    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.ServiceRuntime;
    ```

1. 将以下代码添加到 `Button1_Click` 方法，以使用访问连接字符串的相同方式来访问此字符串。然后代码可以根据所用服务配置文件的设置字符串值来执行一些特定代码。

    ```
    var settingValue = RoleEnvironment.GetConfigurationSettingValue("MySetting");
    if (settingValue == “ThisValue”)
    {
    // Perform these lines of code
    }
    ```

## 管理每个角色实例的本地存储

可以为角色的每个实例添加本地文件系统存储。可在此处存储其他角色不需要访问的本地数据。不需要保存到表、Blob 或 SQL 数据库存储的任何数据都可以存储在此处。例如，可以使用此本地存储缓存可能需要再次使用的数据。角色的其他实例无法访问此处存储的数据。有关本地存储资源的详细信息，请参阅[配置本地存储资源](/documentation/articles/cloud-services-configure-local-storage-resources/)。

本地存储设置将应用到所有服务配置。你只能添加、删除或修改所有服务配置的本地存储。

### 管理每个角色实例的本地存储

1. 选择“本地存储”选项卡。

1. 在“服务配置”列表中，选择“所有配置”。

1. 若要添加本地存储条目，请选择“添加本地存储”按钮。一个新条目将添加到列表中。

1. 在“名称”文本框中，键入要用于此本地存储的名称。

1. 在“大小”文本框中，键入此本地存储所需的大小（以 MB 为单位）。

1. 若要在回收此角色的虚拟机时将此本地存储中的数据删除，请选中“角色回收时清理”复选框。

1. 若要编辑现有的本地存储条目，请选择需要更新的行。然后可以按前面步骤中所述编辑字段。

1. 若要删除某个本地存储条目，请在列表中选择该存储条目，然后选择“删除本地存储”按钮。

1. 若要将这些更改保存到服务配置文件，请选择工具栏上的“保存”图标。

1. 若要访问已添加到服务配置文件的本地存储，必须获取本地资源配置设置的值。使用以下代码行访问此值，创建名为 **MyStorageTest.txt** 的文件，然后将一行测试数据写入该文件。可以将此代码添加到前面过程中使用的 `Button_Click` 方法：

1. 需确保将以下 using 语句添加到 Default.aspx.cs：

    ```
    using System.IO;
    using System.Text;
    ```

1. 将以下代码添加到 `Button1_Click` 方法中。这将在本地存储中创建文件并将测试数据写入该文件。

    ```
    // Retrieve an object that points to the local storage resource
    LocalResource localResource = RoleEnvironment.GetLocalResource("LocalStorage1");

    //Define the file name and path
    string[] paths = { localResource.RootPath, "MyStorageTest.txt" };
    String filePath = Path.Combine(paths);

    using (FileStream writeStream = File.Create(filePath))
    {
          Byte[] textToWrite = new UTF8Encoding(true).GetBytes("Testing Web role storage");
          writeStream.Write(textToWrite, 0, textToWrite.Length);
    }
    ```

1. （可选）若要查看在本地运行云服务时创建的此文件，请使用以下步骤：

  1. 运行 Web 角色，然后选择“Button1”确保调用 `Button1_Click` 中的代码。

  1. 在通知区域中，打开 Azure 图标的快捷菜单并选择“显示计算模拟器 UI”。此时将显示“Azure 计算模拟器”对话框。

  1. 选择 Web 角色。

  1. 在菜单栏上，选择“工具”、“打开本地存储”。此时将显示“Windows 资源管理器”窗口。

  1. 在菜单栏上的“搜索”文本框中输入 **MyStorageTest.txt**，然后按 **Enter** 开始搜索。

    文件将显示在搜索结果中。

  1. 若要查看文件的内容，请打开文件的快捷菜单并选择“打开”。

## 收集云服务诊断数据

你可以为 Azure 云服务收集诊断数据。此数据将添加到存储帐户。对于不同的服务的配置，你可能需要不同的连接字符串。例如，对于具有 UseDevelopmentStorage=true 值的本地服务配置，你可能想要使用本地存储帐户。你可能还希望将云服务配置为使用 Azure 中的存储帐户。有关 Azure 诊断的详细信息，请参阅“Collect Logging Data by Using Azure Diagnostics”（使用 Azure 诊断收集日志记录数据）。

>[AZURE.NOTE] 本地服务已配置为使用本地资源。如果使用云服务配置来发布 Azure 云服务，在发布时指定的连接字符串也将用于诊断连接字符串，除非已指定连接字符串。如果你使用 Visual Studio 打包云服务，不会更改服务配置中的连接字符串。

### 收集云服务诊断数据

1. 选择“配置”选项卡。

1. 在“服务配置”列表中，选择你要更新的服务配置或选择“所有配置”。

    >[AZURE.NOTE] 你可以更新特定服务配置的存储帐户，但如果想要启用或禁用诊断，则必须选择“所有配置”。

1. 若要启用诊断，请选中“启用诊断”复选框。

1. 若要更改存储帐户的值，请选择省略号 (...) 按钮。

    此时将显示“创建存储连接字符串”对话框。

1. 若要使用本地连接字符串，请选择 Azure 存储模拟器选项，然后选择“确定”按钮。

1. 若要使用与 Azure 订阅关联的存储帐户，请选择“你的订阅”选项。

1. 若要使用本地连接字符串的存储帐户，请选择“手动输入的凭据”选项。

    有关如何创建存储帐户以及如何在“创建存储连接字符串”对话框中输入存储帐户详细信息的信息，请参阅[准备从 Visual Studio 发布或部署 Azure 应用程序](/documentation/articles/vs-azure-tools-cloud-service-publish-set-up-required-services-in-visual-studio/)。

1. 在“帐户名称”中选择你要使用的存储帐户。

    如果手动输入存储帐户凭据，请在“帐户密钥”中复制或键入你的主密钥。可从 [Azure 经典门户](http://manage.windowsazure.cn)复制此密钥。若要复制此密钥，请在 [Azure 经典门户](http://manage.windowsazure.cn)的“存储帐户”视图中遵循以下步骤：
    
  1. 选择要用于云服务的存储帐户。

  1. 选择位于屏幕底部的“管理访问密钥”按钮。此时将显示“管理访问密钥”对话框。

  1. 若要复制访问密钥，请选择“复制到剪贴板”按钮。现在可以将此密钥粘贴到“帐户密钥”字段中。

1. 若要使用提供的存储帐户作为将云服务发布到 Azure 时的诊断（和缓存）连接字符串，请选中“在发布到 Azure 时使用 Azure 存储帐户凭据更新诊断和缓存的开发存储连接字符串”复选框。

1. 选择工具栏中的“保存”按钮，将这些更改保存到服务配置文件。

## 更改用于每个角色的虚拟机大小

你可以设置每个角色的虚拟机大小。只能针对所有服务配置设置此大小。如果选择较小的计算机大小，将分配较少的 CPU 核心、内存和本地磁盘存储空间。分配的带宽也将较小。有关这些大小和分配的资源的详细信息，请参阅[云服务的大小](/documentatioin/articles/cloud-services-sizes-specs)。

在 Azure 中每个虚拟机所需的资源将影响在 Azure 中运行云服务的成本。有关 Azure 计费的详细信息，请参阅[了解你的 Azure 帐单](/documentation/articles/billing-understand-your-bill/)。

### 更改虚拟机的大小

1. 选择“配置”选项卡。

1. 在“服务配置”列表中，选择“所有配置”。

1. 若要选择此角色的虚拟机大小，请从“VM 大小”列表中选择适当的大小。

1. 选择工具栏中的“保存”按钮，将这些更改保存到服务配置文件。

## 管理角色的终结点和证书

可通过指定协议、端口号和（针对 HTTPS）SSL 证书信息来配置网络终结点。2012 年 6 月以前的版本支持 HTTP、HTTPS 和 TCP。2012 年 6 月版支持上述协议和 UDP。不能将 UDP 用于计算模拟器中的输入终结点。只能将该协议用于内部终结点。

若要提高 Azure 云服务的安全性，可以创建使用 HTTPS 协议的终结点。例如，如果你有一个云服务，由客户用于采购订单，你希望通过使用 SSL 确保客户的信息是安全的。

你还可以添加可在内部或外部使用的终结点。外部终结点称为输入终结点。输入终结点可为用户提供云服务的另一个访问点。如果你有一个 WCF 服务，你可能要公开使用 Web 角色的内部终结点来访问此服务。

>[AZURE.IMPORTANT] 只能针对所有服务配置更新终结点。

如果添加 HTTPS 终结点，需要使用 SSL 证书。为此，你可以将证书与所有服务配置的角色相关联，并将其用于终结点。

>[AZURE.IMPORTANT] 这些证书不会与服务一起打包。必须通过 Azure 平台经典管理门户将证书分别上载到 Azure

与服务配置关联的任何管理证书仅当云服务在 Azure 中运行时适用。当云服务在本地开发环境中运行时，将使用由 Azure 计算模拟器管理的标准证书。

### 将证书添加到角色

1. 选择“证书”选项卡。

1. 在“服务配置”列表中，选择“所有配置”。

    >[AZURE.NOTE] 若要添加或删除证书，必须选择“所有配置”。可以根据需要更新特定服务配置的名称和指纹。

1. 若要添加此角色的证书，请选择“添加证书”按钮。一个新条目将添加到列表中。

1. 在“名称”文本框中，输入证书的名称。

1. 在“存储位置”列表中，选择要添加的证书所在的位置。

1. 在“存储名称”列表中，选择要用于选择证书的存储。

1. 若要添加证书，请选择省略号 (...) 按钮。此时将显示“Windows 安全性”对话框。

1. 从列表中选择要使用的证书，然后选择“确定”按钮。

    >[AZURE.NOTE] 当你添加来自证书存储区中的证书时，系统会自动将任何中间证书添加到配置设置。若要正确为服务配置 SSL，还必须将这些中间证书上载到 Azure。

1. 若要删除某个证书，请选择该证书，然后选择“删除证书”按钮。

1. 选择工具栏中的“保存”图标，将这些更改保存到服务配置文件。

### 管理角色的终结点

1. 选择“终结点”选项卡。

1. 在“服务配置”列表中，选择“所有配置”。

1. 若要添加终结点，请选择“添加终结点”按钮。一个新条目将添加到列表中。

1. 在“名称”文本框中，键入你想要用于此终结点的名称。

1. 从“类型”列表中选择所需的终结点类型。

1. 从“协议”列表中选择所需终结点的协议。

1. 如果它是输入终结点，请在“公共端口”文本框中输入要使用的公共端口。

1. 在“专用端口”文本框中键入要使用的专用端口。

1. 如果终结点需要 https 协议，请在“SSL 证书名称”列表中选择要使用的证书。

    >[AZURE.NOTE] 此列表将显示你在“证书”选项卡中为此角色添加的证书。

1. 选择工具栏中的“保存”按钮，将这些更改保存到服务配置文件。

## 后续步骤
阅读[配置 Azure 项目](/documentation/articles/vs-azure-tools-configuring-an-azure-project/)以详细了解 Visual Studio 中的 Azure 项目。阅读[架构参考](https://msdn.microsoft.com/zh-cn/library/azure/dd179398)以详细了解云服务架构。

<!---HONumber=Mooncake_0815_2016-->