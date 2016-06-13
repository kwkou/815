<properties
    pageTitle="Azure 角色属性"
    description="了解如何使用 Azure Toolkit for Eclipse 来配置 Azure 角色设置。"
    services=""
    documentationCenter="java"
    authors="rmcmurray"
    manager="wpickett"
    editor=""/>

<tags
    ms.service="multiple"
    ms.date="05/04/2016" 
    wacn.date="04/11/2016"/>

<!-- Legacy MSDN URL = https://msdn.microsoft.com/library/azure/hh690945.aspx -->

# Azure 角色属性 #

可以在 Azure Toolkit for Eclipse 中设置 Azure 角色的各种配置设置。

## 配置 Azure 角色属性 ##

通过辅助角色的属性对话框来完成 Azure 角色属性的配置。在 Eclipse 的项目资源管理器窗格中打开该角色的上下文菜单，然后选择“Azure”子菜单。（如果你在 Eclipse 项目资源管理器中看不到该角色，请在项目资源管理器中展开你的 Azure 项目。）

![][ic789599]

本主题将介绍可在“属性”对话框中设置的各种属性。请注意，许多属性是在创建新的 Azure 部署项目时自动填充的。

为 Azure 角色提供了以下属性页。

* [虚拟机属性](#virtual_machine_properties)
* [缓存属性](#caching_properties)
* [证书属性](#certificates_properties)
* [组件属性](#components_properties)
* [调试属性](#debugging_properties)
* [终结点属性](#endpoints_properties)
* [环境变量属性](#environment_variables_properties)
* [负载平衡/会话相关性（即“粘滞会话”）属性](#session_affinity_properties)
* [本地存储属性](#local_storage_properties)
* [服务器配置属性](#server_configuration_properties)
* [SSL 卸载属性](#ssl_offloading_properties)
	
<a name="virtual_machine_properties"></a>
### 虚拟机属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“属性”，你将能够更改虚拟机大小，还可以更改实例数，如下图所示。

![][ic719499]

>[AZURE.NOTE] 仅限 Windows：将实例数设置为大于 1 的值，此外，还要配置应用程序服务器，不管此设置如何，工具包都只允许在模拟器中运行 1 个角色实例。这是为了避免不同服务器实例在同一台计算机上运行时，在这些实例之间出现端口绑定冲突（例如，所有实例都尝试绑定到端口 8080）。所需的实例计数设置已保留，但仅当部署到云中时生效。

<a name="caching_properties"></a>
### 缓存属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“缓存”。在此对话框中，你可以启用命名并置的 Memcache 兼容缓存，从而可以帮助你加快 Web 应用程序的运行速度。

![][ic719483]

在“缓存”属性页中，你可以指定以下项的全局设置：

* 是否启用并置缓存。
* 以内存的百分比形式表示的缓存大小。
* 当应用程序作为云服务运行时，用于保存缓存状态的存储帐户名称；如果你不想保存缓存状态，则为“无”。（在计算模拟器中运行应用程序时，不使用存储帐户名称。） 如果将存储帐户名称设置为“(自动)”（这是默认值），则缓存配置将自动使用你在“发布到 Azure”对话框中选择的同一存储帐户。

>[AZURE.NOTE] 仅当你使用 Eclipse 工具包的发布向导发布部署时，“(自动)”设置才能产生所需的效果。如果改为使用外部机制（如 [Azure 管理门户][]）手动发布 .cspkg 文件，部署将不能正常工作。

以下对话框显示缓存的属性。

![][ic719501]

* **名称：**并置缓存的名称。
* **端口号：**要供缓存使用的端口号。
* **过期策略：**可以是以下值之一，指定缓存中的密钥何时过期。
    * **Absolute：**当达到“生存分钟数”指定的时间时密钥过期。
    * **NeverExpires：**密钥没有过期时间。
    * **SlidingWindow：**如果在“生存分钟数”指定的时间内未访问密钥，密钥将过期；每次访问密钥时，都会重置到期时钟。
* **生存分钟数：**Memcache 密钥生存的最大分钟数，受过期策略约束。
* **使用不同角色实例上的复制备份实现高可用性：**如果启用，可帮助利用不同角色实例上的复制备份提供高可用性。请注意，在你的部署中必须至少启用两个角色实例，此功能才能起作用。

若要添加新的缓存，请单击“缓存”属性页中的“添加”按钮，此时将打开“配置命名缓存”对话框。请为上述属性提供值。

若要修改某个命名缓存，请在“缓存”属性页中选择该缓存，然后单击“编辑”按钮。此时将打开一个对话框让你修改缓存属性。按“确定”保存缓存值。

若要删除某个缓存，请在“缓存”属性页中选择该缓存并单击“删除”按钮，然后单击“是”以确认删除。

有关如何使用缓存的详细信息，请参阅[如何使用并置缓存][]。

<a name="certificates_properties"></a>
### 证书属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“证书”。

![][ic710964]

在此对话框中，可以添加或删除由 Eclipse 项目引用的证书。请注意，此处列出的证书不会自动存储在任何 Java 密钥库内，因此不会自动可供从 Java 应用程序中进行任何使用。它们仅注册到 Azure 中，以便可以预先加载到运行你的部署的虚拟机上的 Windows 证书存储中，随后供其他 Windows 软件使用。目前，使用“证书”对话框中以这种方式引用的证书的工具包的唯一功能是“[SSL 卸载][]”，因为它依赖于 Internet Information Services (IIS) 和应用程序请求路由 (ARR)，这些软件和模块需要正确的证书才能以这种方式可供使用。

当你使用发布向导将项目部署到 Azure 时，系统将提示你指向与这些证书对应的个人信息交换 (PFX) 文件并提供其密码，以便自动将其上载到 Azure 服务，但前提是这些文件以前尚未上载到那里。

<a name="components_properties"></a>
### 组件属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“组件”。在此对话框中，你能够添加、修改或删除你的角色的组件，以及更改它们的处理顺序。

![][ic719502]

使用组件功能可以向 Azure 部署项目中添加依赖项，如 Java 应用程序项目、特殊文件以及你的部署所需的可执行命令行语句。

对于每个组件，你可以指定：

* 在生成 Azure 部署项目时将组件导入项目所要执行的步骤。
* 在 Azure 云中部署该组件时要执行的步骤。

>[AZURE.NOTE] 在指定组件文件或命令行时，请记住，你的部署将发布到 Windows 虚拟机，因此你的自定义步骤必须对于基于 Windows 的操作系统是有效的。

组件具有以下属性：

* **Import：**指示在生成项目时如何将组件导入项目的方法。这可以是以下值之一：
    * **copy：**组件将从 **From** 属性指定的本地路径复制到角色的 **approot** 目录。
    * **EAR：**组件是从 **From** 属性指定的本地路径上的企业应用程序项目导入的 Java 企业存档文件 (EAR)。（这是工具包根据该位置中的项目性质自动检测到的）。
    * **JAR：**组件是 Java 存档文件 (JAR)，从 **From** 属性指定的本地路径上的 Java 项目导入。（这是工具包根据该位置中的项目性质自动检测到的）。
    * **none：**不执行任何操作即可导入组件。此项适用于假定组件已存在于角色的 **approot** 目录中，或者组件只是一个可执行的命令行语句（当 **Deploy** 方法是 **exec** 时在 **As** 属性中指定）。
    * **WAR：**组件是 Java Web 应用程序存档文件 (WAR) 并从 **From** 属性指定的本地路径上的动态 Web 项目导入。（这是工具包根据该位置中的项目性质自动检测到的）。
    * **zip：**组件是一个 zip 文件，通过压缩 **From** 属性指定的目录或文件导入。
* **From：**表示要导入到部署的项目的文件夹或文件在本地计算机上的源路径。可以在此属性中使用 Windows 环境变量。生成项目时，所有可导入组件都将导入到角色的 **approot** 目录。
	
	请注意，你可以在部署到云（而非计算模拟器）时通过下载部署组件。请参阅下面有关添加组件的相关信息。
	
* **As：**将组件导入到角色的 **approot** 目录并最终部署到 Azure 云中时所用的文件名。将此属性留空可使该名称保持与本地计算机上的名称相同。（对于可执行组件，即 **Deploy** 方法设为 **exec** 的组件，这可以是任意 Windows 命令行语句。）

	>[AZURE.IMPORTANT] 如果在此值中使用空格字符，则处理空格字符的方式将因部署方法而异。如果部署方法是 **exec**，则空格将解释为命令行参数分隔符，而不是文件名的一部分。对于所有其他部署方法，空格将解释为文件名的一部分。
	
* **Deploy：**指示在启动部署时应用于组件的操作的方法。这可以是以下值之一：
    * **copy：**组件将复制到 **To** 属性指定的目标路径。
    * **exec：**组件是一个可执行的 Windows 命令行语句，启动部署时，在 **To** 属性指定的路径上下文中执行。
    * **none：**启动部署时，不对组件应用任何操作。
    * **zip：**组件将解压缩到 **To** 属性指定的目标路径。仅当 **Import** 属性是 **zip** 时，此方法才可用。
* **To：**将组件部署到的虚拟机上的目标路径。可以在此属性中使用 Windows 环境变量，并且文件路径是相对于 **approot** 的路径。
	
若要添加新组件，请单击“组件”属性页上的“添加”按钮，此时将打开“Azure 角色组件”对话框。请为上述属性提供值。

下面显示一个用于添加新的 WAR 组件的示例。

![][ic719503]

部署到云（而非计算模拟器）时，如果要通过下载部署组件，请确保选中“在云中时，不是包含在程序包中，而是从以下位置部署”。如果要从 Azure 存储帐户下载，请从“存储帐户”下拉列表中选择存储帐户（你可以单击“帐户”链接以修改列表中的内容），这将部分填充“URL”字段，然后再填写 URL 的剩余部分。如果不希望使用 Azure 存储空间，请在“存储帐户”下拉列表中选择“(无)”，然后在“URL”字段中输入组件的 URL。指定以下方法之一：

* **copy：**下载组件将复制到 **To Directory** 路径指定的目标路径。
* **same：**对“通过下载部署”使用与“从程序包部署”相同的方法。
* **zip：**下载组件将解压缩到 **To Directory** 路径指定的目标路径。

若要修改某个组件，请在“组件”属性页中选择该组件，然后单击“编辑”按钮。此时将打开一个对话框让你修改组件属性。按“确定”保存组件值。

若要删除某个组件，请在“组件”属性页中选择该组件并单击“删除”按钮，然后单击“是”以确认删除。

将按所列顺序处理组件。使用“上移”和“下移”按钮可排列顺序。

>[AZURE.NOTE] 服务器配置功能也依赖于组件。如果不删除相应的服务器配置，则无法删除或编辑这些组件。当你尝试对此类组件进行更改时，将向你提示该信息。

<a name="debugging_properties"></a>
### 调试属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“调试”。在此对话框中，你可以启用或禁用远程调试，并可以创建调试配置，如下图所示。

![][ic719504]

有关调试的信息，请参阅[在 Eclipse 中调试 Azure 应用程序][]。

<a name="endpoints_properties"></a>
### 终结点属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“终结点”。在此对话框中，你可以创建终结点以及编辑或删除终结点，如下图所示。

![][ic719505]

若要添加终结点，请单击“终结点”属性页上的“添加”按钮，此时将打开“添加终结点”对话框。

![][ic710897]

输入终结点的名称，选择类型（“输入”、“内部”，或“实例输入”），并指定公用和专用端口。按“确定”保存新的终结点值。

根据终结点类型，你可以使用端口范围，如下所示：

* 对于实例输入终结点，公用端口可以是端口范围（例如 **2000-2010**），专用端口是固定值。
* 对于内部终结点，不使用公用端口，专用端口可以是端口范围或留空，或者设置为星号以指示它由 Azure 自动设置。
* 对于输入终结点，公用端口只能是固定值，专用端口可以是固定值或留空，或者设置为星号以指示它由 Azure 自动设置。

如果要使用单个端口号（而不是范围），请将范围结束的文本框留空。

对于设为自动的端口，如果需要确定在运行时实际使用哪个端口，应用程序可以使用 [com.microsoft.windowsazure.serviceruntime 包摘要][]中所述的 Azure 服务运行时 API。

若要了解如何使用实例输入终结点来帮助调试多实例部署，请参阅[调试多实例部署中的特定角色实例][]。

若要修改某个终结点，请在“终结点”属性页上选择该终结点，然后单击“编辑”按钮。此时将打开一个对话框让你修改终结点名称、类型以及公用和专用端口。按“确定”保存修改的终结点值。

若要删除某个终结点，请在“终结点”属性页中选择该终结点并单击“删除”按钮，然后单击“是”以确认删除。

为了正确配置用户对角色启用的某些功能（例如缓存、远程调试、会话相关性或 SSL 卸载），该工具包可能会自动配置将与用户定义的终结点一起列出的特殊终结点。只要启用了关联的功能，该工具包就会禁止用户编辑或删除此类自动生成的终结点。

<a name="environment_variables_properties"></a>
### 环境变量属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“环境变量”。在此对话框中，你可以创建环境变量以及修改或删除环境变量，如下图所示。

![][ic719506]

在角色启动时，环境变量可用于启动脚本。

>[AZURE.NOTE] 指定环境变量时，请记住，你的部署将发布到 Windows 虚拟机，因此你的环境变量必须对于基于 Windows 的操作系统有效。

为了举例说明在角色启动时可用的环境变量，请通过单击“添加”按钮创建一个新的环境变量。下面显示了所创建的名为 **MyRoleVersion** 的环境变量并为其分配值 **1.0**。

![][ic659268]

在 jsp 代码中，可以使用 `System.getenv` 方法显示该值：

    <body>
      <b> Hello World!</b>
      <p>Running role version: <%= System.getenv("MyRoleVersion") %></p>
    </body>

在应用程序运行时将生成此输出：

![][ic552233]

若要修改某个环境变量，请在“环境变量”属性页中选择该环境变量，然后单击“编辑”按钮。此时将打开一个对话框让你修改环境变量属性。按“确定”保存环境变量值。

若要删除某个环境变量，请在“环境变量”属性页中选择该环境变量并单击“删除”按钮，然后单击“是”以确认删除。

为了正确配置用户对角色启用的某些功能（如服务器配置、远程调试或本地存储），该工具包可能会自动配置将与用户定义的环境变量一起列出的特殊环境变量。只要启用了关联的功能，该工具包就会禁止用户编辑或删除此类自动生成的环境变量。

<a name="session_affinity_properties"></a>
### 负载平衡/会话相关性（即“粘滞会话”）属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“负载平衡”。在此对话框中，你可以启用或禁用会话相关性，如下图所示。

![][ic719492]

如需相关信息，请参阅[会话相关性][]。另外，请注意此功能在 SSL 卸载上下文中的行为，如 [SSL 卸载][]中所述。

<a name="local_storage_properties"></a>
### 本地存储属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“本地存储”。在此对话框中，你可以为运行你的应用程序的虚拟机创建、修改或删除临时本地存储。可以为本地存储大小设置特定值以及设置在回收角色时是否保留内容，如下图所示。

![][ic719508]

（可选）还可以指定对应于本地存储的环境变量。

默认情况下，部署到 Azure 中的所有内容都放置（并解压缩）在角色实例的 **approot** 文件夹中。虽然大多数简单部署在解压缩后都适合那里，但为 **approot** 目录分配的空间是有限的并且未明确定义（小于 1 GB 是合理的经验法则）。因此，为了确保 Azure 为可能不适合 **approot** 文件夹的较大部署分配足够的磁盘空间，你应该使用“本地存储”对话框设置本地存储资源。有关执行此操作的简单方法，请参阅[实施大型部署][]。

可以轻松地使用由 Eclipse 工具包自动关联到资源的环境变量从启动脚本（例如，**startup.cmd**）引用存储资源，如“本地存储”对话框中所示。该环境变量将包含在执行启动脚本时已配置的本地资源的完整路径。

若要修改某个本地存储资源，请在“本地存储”属性页中选择该本地存储资源，然后单击“编辑”按钮。此时将打开一个对话框让你修改本地存储资源属性。按“确定”保存本地存储资源值。

若要删除某个本地存储资源，请在“本地存储”属性页中选择该本地存储资源并单击“删除”按钮，然后单击“是”以确认删除。

<a name="server_configuration_properties"></a>
### 服务器配置属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“服务器配置”。在此对话框中，你可以添加、删除和修改部署使用的 JDK 和 Java 应用程序服务器，以及添加或删除部署使用的应用程序（例如 WAR、JAR 或 EAR 文件）。

### JDK 配置 ###

此对话框允许你指定要用于部署的 JDK 包。如果在 Windows 上使用 Eclipse，则可以指定在 Azure 模拟器中运行时要在本地使用的 JDK 包，并可以选择将该本地安装部署到 Azure。在非 Windows 操作系统上，模拟器 JDK 设置不适用，并且无法部署本地安装的 JDK，因为它与 Windows 不兼容。但是，无论你使用哪个操作系统，你可以始终选择要部署到 Azure 的第三方 JDK 包，或者从备用下载位置指向你自己的 Windows 兼容 JDK 包。

下面是说明如何在 Windows 上指定 JDK 的一个示例：

![][ic780647]

如果在 Windows 上使用 Eclipse，则可以指定要用于计算模拟器的 JDK；为此，请确保在“模拟器部署”部分中选中了“使用此文件路径中的 JDK 进行本地测试”。然后，指定 JDK 的本地路径；如果未自动选择你要使用的 JDK，则可以浏览到不同 JDK。你还可以选择将 JDK 部署到你的 Azure 云服务；为此，请在“云部署”部分中选择“部署我的本地 JDK (自动上载到云存储)”选项。

注意：在非 Windows 操作系统上，“模拟器部署”设置和“部署我的本地 JDK”选项不可用。下面的示例说明了如何在 Mac 或其他受支持的非 Windows 操作系统上指定 JDK：

![][ic789643]

无论你使用的是什么操作系统，都针对 JDK 包的源和类型提供了以下两个“云部署”选项：

* **部署 Azure 上提供的第三方 JDK 包** 
* **通过自定义下载部署** 

如果你使用的是“部署 Azure 上提供的第三方 JDK 包”选项：

1. 选中名为“部署 Azure 上提供的第三方 JDK 包”复选框。
1. 从下拉列表中选择 Azure 上提供的第三方 JDK 包。
1. 在 Windows 上，你的“JDK”选项卡将显示如下：
	![][ic780648]
	在 Mac OS 或其他受支持的非 Windows 操作系统上，该选项卡将显示如下：
	![][ic789643]
1. 单击“确定”保存你的更改。
1. 当系统提示你接受第三方 JDK 包提供程序的许可协议时，查看许可条款。假设你接受这些条款，请单击“是”以关闭“接受许可协议”对话框。
	请注意，可以自定义哪些项显示在“部署 Azure 上提供的第三方 JDK 包”选项的下拉列表中的基本逻辑。若要自定义项目，请在“JDK”对话框中，单击“自定义”链接。这将关闭“JDK”属性页并在 Eclipse 中打开 **componentsets.xml** 文件，然后你可以根据需要对其进行修改。**componentsets.xml** 的文档在 **componentsets.xml** 文件本身中提供。

如果你使用的是“通过自定义下载部署 JDK”选项：

1. 创建 JDK 安装目录的 ZIP，确保目录节点本身是 ZIP 结构的子级，而不是其内容。记下目录的名称，因为你稍后需要扩眼它，并请记住此 JDK 安装将部署到 Windows 虚拟机。
1. 将该 ZIP 作为 Blob 上载到 Azure 存储帐户。你可以使用可从外部使用用于将 Blob 上载到 Azure 存储空间的工具来执行此操作。建议使用专用 Blob。记下 ZIP 内容的 Blob URL。
1. 选中名为“通过自定义下载部署 JDK”的复选框。
	如果要从 Azure 存储帐户下载，请从“存储帐户”下拉列表中选择存储帐户（你可以单击“帐户”链接以修改列表中的内容），这将部分填充“URL”字段，然后再填写 URL 的剩余部分。如果不希望使用 Azure 存储空间，请在“存储帐户”下拉列表中选择“(无)”，然后在“URL”字段中输入 JDK 下载链接的 URL。如果使用 Azure 存储空间，URL 中的 Blob 名称必须小写。
1. 确保 **JAVA\_HOME** 文本框引用正确的目录名称。默认情况下，它将引用与为本地使用选择的值相同的 JDK 目录名称。但如果 ZIP 中包含的目录具有不同名称（例如，由于使用不同版本），请相应地更新 **JAVA\_HOME** 文本框中的目录名称，因为此设置将在云中（而不是在计算模拟器中）使用。
1. 单击“确定”保存你的更改。

就这么简单。现在，当你针对云进行构建时，你会发现包大小将小得多，生成过程通常需要更少的时间，并且在发布到云时部署本身也应需要更少的时间。请注意，仅当将应用程序部署到云中时，“部署我的本地 JDK (自动上载到云存储)”或“通过自定义下载部署 JDK”选项才有效。它们不会影响你的计算模拟器体验；当你部署到计算模拟器时，仍将使用本地版本的组件。

### 服务器配置 ###

下面举例说明如何指定应用程序服务器。

![][ic796926]

确认已选中“部署此类型的服务器”复选框，然后选择你要使用的应用程序服务器类型。

要指定用于云部署的服务器，可以利用以下选项：

1. **部署 Azure 上提供的第三方服务器** - 这在部署效率和简单性是优先考虑项且服务器不需要自定义配置的开发/测试方案中特别适用。或者，当你想要使用这些服务器之一作为起点，但在部署的启动逻辑中包括相应的服务器自定义步骤时，此选项也适用。
1. **通过自定义下载部署** - 当你有要在云中使用的已特别准备和配置的服务器时，此选项特别适用于生产方案。
1. **部署我的本地服务器安装** - 如果本地服务器安装已针对你的使用进行自定义配置，则此选项特别适用。如果选择此选项，则还必须在下面的“本地服务器路径”文本框中指定本地服务器的路径。

如果你使用的是“部署 Azure 上提供的第三方服务器”选项：

1. 选中名为“部署 Azure 上提供的第三方服务器”的复选框。
1. 从下拉菜单中选择要用于云中的部署的所需服务器软件。请注意，如果你在前面指定了本地服务器，则将限制你只能选择与该服务器类型属于同一系列的云服务器。但是，如果你未选择服务器类型，则可以选择当前在 Azure 上可用的任何服务器，系统将自动为你选择服务器类型。
1. 单击“确定”保存你的更改。

如果使用的是“通过自定义下载部署”选项：

1. 请确保你已按前面的步骤选择了本地服务器。这将告知插件如何通过自定义下载部署服务器，因为该服务器必须与所选的服务器类型属于同一系列。
1. 选中名为“通过自定义下载部署”的复选框。
	如果你要从 Azure 存储帐户下载，请从“存储帐户”下拉列表中选择存储帐户（你可以单击“帐户”链接以修改列表中的内容），这将部分填充“URL”字段，然后填写服务器下载 ZIP 的 URL 的剩余部分（当使用 Azure 存储空间时，URL 中的 Blob 名称必须小写）。如果不希望使用 Azure 存储空间，请在“存储帐户”下拉列表中选择“(无)”，然后在“URL”字段中输入服务器下载 ZIP 的 URL。该 ZIP 将包含表示应用程序服务器安装目录的子文件夹。例如，如果你对 Apache Tomcat 7.0.35 使用 zip，则 zip 内将包含表示安装目录的子文件夹，如 **apache-tomcat-7.0.35**。 
1. 指定主目录环境变量的值。它将默认为用于本地应用程序服务器的值，但如果你的云应用程序服务器不同于本地应用程序服务器，则可以指定一个不同的值。但是，你需要确保云应用程序服务器与前面选择的服务器类型属于同一个系列。
	如果将来你要更新云应用程序服务器 zip，可以手动更改主目录设置，或者，让它再次与本地设置匹配（如果也更改了本地应用程序服务器）。
1. 单击“确定”保存你的更改。

可以自定义哪些项显示在“服务器配置”属性页的“服务器”选项卡中的基本逻辑。这是一项高级功能，如果你的需求超出默认值，或者如果你要添加其他服务器，则可能需要此功能。若要自定义逻辑，请在“服务器”对话框中，单击“自定义”链接。这将关闭“服务器配置”属性页并在 Eclipse 中打开 **componentsets.xml** 文件，然后你可以根据需要修改该文件以扩展服务器配置模板。**componentsets.xml** 的文档在 **componentsets.xml** 文件本身中提供。

如果你使用的是“部署我的本地服务器(自动上载到云存储)”选项：

1. 选中名为“部署我的本地服务器(自动上载到云存储)”的复选框。
1. 使用“存储帐户”下拉列表，选择“(自动)”。如果你在此处指定“(自动)”，则 Eclipse 工具包将对服务器使用你在“发布到 Azure”对话框中为部署选择的同一存储帐户。
1. 单击“确定”保存你的更改。

如果存在以下任何情况，请在“本地服务器路径”文本框中选择计算机上的服务器安装路径：

* 你要在模拟器中测试你的部署（仅适用于 Windows）。
* 你要将本地安装的服务器部署到云。
* 你要在云中使用你自己的自定义服务器下载，在这种情况下，还应确保选中上述“部署我的本地服务器(自动上载到云存储)”选项。

如果上述任何选项均不适用于你的情况，本地服务器设置是可选的。

### 应用程序配置 ###

下面举例说明如何指定应用程序。

![][ic719512]

单击“添加”以添加另一个应用程序，或者单击“删除”以删除某个应用程序。出于效率考虑，如果在部署到云时你要对应用程序的源使用下载，请使用“[组件属性](#components_properties)”以指定 URL、存储帐户等。

从 2014 年 4 月发行版开始，你的应用程序将自动上载到你为部署选择的同一存储帐户（在 **eclipsedeploy** 容器下）。你的部署的启动逻辑包含首先从该存储帐户下载这些应用程序的步骤。这意味着你无需重新生成和重新部署整个程序包即可升级部署中的应用程序，方法是手动将更新版本的应用程序直接上载到该存储帐户（例如，使用 Azure 门户），替换工具包最初上载到那里的 WAR 文件。然后，只需再次使用 Azure 管理门户，或通过命令行实用工具启动所有这些角色实例的回收。（目前不支持直接从 Eclipse 工具包中触发角色回收。）

### 有关服务器配置的说明 ###

通过“服务器配置”属性页所做的更改将反映到 package.xml 文件的 `<component>` 元素中。

当你对 JDK 或应用程序服务器使用“自动上载...”或“通过下载部署...”选项并且是针对云（而不是计算模拟器）构建并且已连接到网络时，你可能会在控制台输出中发现如下生成消息（当 Ant 生成器验证下载的可用性时）：

`[windowsazurepackage] Verifying blob availability (https://example.blob.core.windows.net/temp/tomcat6.zip)...`

如果你选择了“通过下载部署...”选项，则可能会显示以下警告，但生成将继续：

`[windowsazurepackage] warning: Failed to confirm blob availability! Make sure the URL and/or the access key is correct (https://example.blob.core.windows.net/temp/tomcat6.zip).`

此警告是尚未验证下载的可用性的唯一指示。因此，如果由于某种原因，云中的部署失败，请检查是否收到此警告。

如果你要禁用下载验证（例如，如果你认为它不必要地降低生成速度），请在 package.xml 的 `<windowsazurepackage>` 元素中将 `verifydownloads` 属性设为 `false`：

`<windowsazurepackage verifydownloads="false" ...>`

如果你选择了“自动上载...”选项，则每当需要上载时，就会在控制台窗口中看到每隔 5 秒钟报告上载进度的生成消息。

<a name="ssl_offloading_properties"></a>
### SSL 卸载属性 ###

在 Eclipse 的项目资源管理器窗格中打开角色的上下文菜单，单击“Azure”，然后单击“SSL 卸载”。

![][ic719481]

在此对话框中，你可以启用 SSL 卸载，以便可以在 Azure 上的 Java 部署中轻松地启用安全超文本传输协议 (HTTPS) 支持，而无需在 Java 应用程序服务器中配置 SSL。有关详细信息，请参阅 [SSL 卸载][]和[如何使用 SSL 卸载][]。

## 另请参阅 ##

[适用于 Eclipse 的 Azure 工具包][]

[安装 Azure Toolkit for Eclipse][]

[在 Eclipse 中为 Azure 创建 Hello World 应用程序][]

[Azure 项目属性][]

[Azure 存储帐户列表][]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心][]。

<!-- URL List -->

[Azure Java 开发人员中心]: /develop/java/
[Azure 管理门户]: http://manage.windowsazure.cn
[适用于 Eclipse 的 Azure 工具包]: /documentation/articles/azure-toolkit-for-eclipse/
[Azure 项目属性]: /documentation/articles/azure-toolkit-for-eclipse-azure-project-properties/
[Azure 存储帐户列表]: /documentation/articles/azure-toolkit-for-eclipse-azure-storage-account-list/
[com.microsoft.windowsazure.serviceruntime 包摘要]: http://azure.github.io/azure-sdk-for-java/com/microsoft/windowsazure/serviceruntime/package-summary.html
[在 Eclipse 中为 Azure 创建 Hello World 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-creating-a-hello-world-application/
[调试多实例部署中的特定角色实例]: /documentation/articles/azure-toolkit-for-eclipse-debugging-azure-applications/#debugging_specific_role_instance
[在 Eclipse 中调试 Azure 应用程序]: /documentation/articles/azure-toolkit-for-eclipse-debugging-azure-applications/
[实施大型部署]: /documentation/articles/azure-toolkit-for-eclipse-deploying-large-deployments/
[如何使用并置缓存]: /develop/java/
[如何使用 SSL 卸载]: /develop/java/
[安装 Azure Toolkit for Eclipse]: /documentation/articles/azure-toolkit-for-eclipse-installation/
[会话相关性]: /documentation/articles/azure-toolkit-for-eclipse-enable-session-affinity/
[]: /develop/java/
[SSL 卸载]: /develop/java/

<!-- IMG List -->

[ic789599]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic789599.png
[ic719499]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719499.png
[ic719483]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719483.png
[ic719501]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719501.png
[ic710964]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic710964.png
[ic719502]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719502.png
[ic719503]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719503.png
[ic719504]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719504.png
[ic719505]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719505.png
[ic710897]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic710897.png
[ic719506]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719506.png
[ic659268]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic659268.png
[ic552233]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic552233.png
[ic719492]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719492.png
[ic719508]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719508.png
[ic780647]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic780647.png
[ic789643]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic789643.png
[ic780648]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic780648.png
[ic789643]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic789643.png
[ic796926]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic796926.png
[ic719512]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719512.png
[ic719481]: ./media/azure-toolkit-for-eclipse-azure-role-properties/ic719481.png

<!---HONumber=Mooncake_0606_2016-->