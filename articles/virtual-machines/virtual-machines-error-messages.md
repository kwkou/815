<properties
    pageTitle="Azure 虚拟机错误消息"
    description=""
    services="virtual-machines"
    documentationcenter=""
    author="xujing-ms"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure-services"
    ms.date="3/17/2017"
    wacn.date="04/17/2017"
    ms.author="xujing"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="3ab46147e7f0364b605f8566253aeaff4046ee4c"
    ms.lasthandoff="04/06/2017" />

# <a name="azure-virtual-machine-error-messages"></a>Azure 虚拟机错误消息
本文描述在管理 Azure 虚拟机 (VM) 时遇到的常见错误代码和消息。  

>[AZURE.NOTE]
>欢迎在本错误消息反馈页面上或者通过 [Azure 反馈](https://feedback.azure.com/forums/216843-virtual-machines)使用 #azerrormessage 标记给我们留言。
>
>

## <a name="error-response-format"></a>错误响应格式 
Azure VM 使用以下 JSON 格式提供错误响应。

    {
      "status": "status code",
      "error": {
        "code":"Top level error code",
        "message":"Top level error message",
        "details":[
         {
          "code":"Inner evel error code",
          "message":"Inner level error message"
         }
        ]
       }
    }

错误响应始终包含一个状态代码和一个错误对象。 每个错误对象始终包含错误代码和消息。 如果 VM 部署是通过模板创建的，则错误对象还包含一个详细信息节，其中包含错误代码和消息的内部级别。 通常，错误消息的最高内部级别是根级失败。 

## <a name="common-virtual-machine-management-error"></a>常见的虚拟机管理错误

本部分列出管理虚拟机时遇到的常见错误消息

|  错误代码  |  错误消息  |  
|  :------| :-------------|  
|  AcquireDiskLeaseFailed  |  使用 URI 为 {1} 的 Blob 创建磁盘“{0}”时无法获取租约。 Blob 已在使用中。  |  
|  AllocationFailed  |  分配失败。 请尝试减小 VM 大小或 VM 数量，稍后重试，或尝试部署到其他可用性集或其他 Azure 位置。  |  
|  AllocationFailed  |  VM 分配因内部错误而失败。 请稍后重试或尝试部署到其他位置。  |
|  ArtifactNotFound  |  无法在位置“{2}”中找到发布服务器为“\{0\}”且类型为“{1}”的 VM 扩展。  |
|  ArtifactNotFound  |  在扩展存储库中找不到发布服务器为“{0}”、类型为“{1}”且类型处理程序版本为“{2}”的扩展。  |
|  ArtifactVersionNotFound  |  在项目存储库中找不到满足所请求的版本“{0}”的版本。  |
|  ArtifactVersionNotFound  |  在项目存储库中找不到满足所请求的 VM 扩展的版本“{0}”的版本，其发布者为“{1}”，类型为“{2}”。  |
|  AttachDiskWhileBeingDetached  |  无法将数据磁盘“{0}”附加到 VM“{1}”，因为当前正在分离磁盘。 请等到磁盘完全分离，然后重试。  |
|  BadRequest  |  此区域尚不支持“对齐”可用性集。  |
|  BadRequest  |  不支持向非托管可用性集添加具有托管磁盘的 VM 或向托管可用性集添加具有基于 Blob 磁盘的 VM。 请创建具有“托管”属性集的可用性集，以便向其中添加具有托管磁盘的 VM。  |
|  BadRequest  |  此区域不支持托管磁盘。  |
|  BadRequest  |  OS 类型“{0}”不支持每个处理程序多个 VMExtension。 已在输入中添加或指定了 VMExtension“{1}”与处理程序“{2}”。  |
|  BadRequest  |  操作“{0}”在含托管磁盘的资源“{1}”上不受支持。  |
|  CertificateImproperlyFormatted  |  检索自 {0} 的机密的 JSON 表示形式中的数据字段可能是格式错误的 PFX 文件，或者所提供的密码无法正确解码 PFX 文件。  |
|  CertificateImproperlyFormatted  |  检索自 {0} 的数据不可反序列化为 JSON。  |
|  冲突  |  仅当创建 VM 或该 VM 已取消分配时，才允许磁盘重设大小。  |
|  ConflictingUserInput  |  无法附加磁盘“{0}”，因为磁盘已由 VM“{1}”拥有。  |
|  ConflictingUserInput  |  源和目标资源组相同。  |
|  ConflictingUserInput  |  磁盘 {0} 的源存储帐户和目标存储帐户不同。  |
|  ContainerAlreadyOnLease  |  已有针对保留 URI 为 {0} 的 blob 的存储容器的租约。  |
|  CrossSubscriptionMoveWithKeyVaultResources  |  移动资源请求包含由请求中一个或多个 {0} 引用的 KeyVault 资源。 跨订阅移动中当前不支持此操作。 请查看 KeyVault 资源 ID 的错误详细信息。  |
|  DiagnosticsOperationInternalError  |  处理 VM {0} 的诊断配置文件时出现内部错误。  |
|  DiskBlobAlreadyInUseByAnotherDisk  |  属于 VM“{1}”的其他磁盘正在使用 Blob {0}。 你可以检查 Blob 元数据以了解磁盘引用信息。  |
|  DiskBlobNotFound  |  找不到磁盘“{1}”的 URI 为 {0} 的 VHD Blob。  |
|  DiskBlobNotFound  |  找不到 URI 为 {0} 的 VHD Blob。  |
|  DiskEncryptionKeySecretMissingTags  |  {0} 机密没有 {1} 标志。 请更新机密版本，添加所需标志，然后重试。  |
|  DiskEncryptionKeySecretUnwrapFailed  |  未能使用密钥 {1} 解开机密 {0} 的值。  |
|  DiskImageNotReady  |  磁盘映像 {0} 处于 {1} 状态。 请在映像就绪后重试。  |
|  DiskPreparationError  |  准备 VM 磁盘时出现一个或多个错误。 有关详细信息，请参阅磁盘实例视图。  |
|  DiskProcessingError  |  磁盘处理已终止，因为 VM 有其他失败磁盘。  |
|  ImageBlobNotFound  |  找不到磁盘“{1}”的 URI 为 {0} 的 VHD Blob。  |
|  ImageBlobNotFound  |  找不到 URI 为 {0} 的 VHD Blob。  |
|  IncorrectDiskBlobType  |  磁盘 Blob 的类型只能为页 Blob。 磁盘“{1}”的 Blob {0} 的类型为块 Blob。  |
|  IncorrectDiskBlobType  |  磁盘 Blob 的类型只能为页 Blob。 Blob {0} 的类型为“{1}”。  |
|  IncorrectImageBlobType  |  磁盘 Blob 的类型只能为页 Blob。 磁盘“{1}”的 Blob {0} 的类型为块 Blob。  |
|  IncorrectImageBlobType  |  磁盘 Blob 的类型只能为页 Blob。 Blob {0} 的类型为“{1}”。  |
|  InternalOperationError  |  无法解析存储帐户 {0}。 请确保通过与计算资源位于相同位置的存储资源提供程序创建该帐户。  |
|  InternalOperationError  |  {0} 目标搜寻任务失败。  |
|  InternalOperationError  |  验证 VM“{0}”的网络配置文件时出错。  |
|  InvalidAccountType  |  AccountType {0} 无效。  |
|  InvalidParameter  |  参数 {0} 的值无效。  |
|  InvalidParameter  |  不允许指定的管理员密码。  |
|  InvalidParameter  |  提供的密码长度必须在 {0} 到 {1} 个字符之间，且必须满足以下至少 {2} 个密码复杂性要求: <ol><li> 包含一个大写字符</li><li>包含一个小写字符</li><li>包含一个数字</li><li>包含一个特殊字符。</li></ol>  |
|  InvalidParameter  |  不允许指定的管理员用户名。  |
|  InvalidParameter  |  如果 VM 是通过平台或用户映像创建的，则无法附加现有 OS 磁盘。  |
|  InvalidParameter  |  容器名称 {0} 无效。 容器名称的长度必须为 3-63 个字符，且仅可包含小写字母数字字符和连字符。 连字符前后必须为字母数字字符。  |
|  InvalidParameter  |  URL {1} 中的容器名称 {0} 无效。 容器名称的长度必须为 3-63 个字符，且仅可包含小写字母数字字符和连字符。 连字符前后必须为字母数字字符。  |
|  InvalidParameter  |  URL {0} 中的 Blob 名称包含斜杠。 目前磁盘不支持这种显示方式。  |
|  InvalidParameter  |  URI {0} 似乎不是正确的 blob URI。  |
|  InvalidParameter  |  名为“{0}”的磁盘已在使用相同的 LUN: {1}。  |
|  InvalidParameter  |  名为“{0}”的磁盘已存在。  |
|  InvalidParameter  |  无法为已在指定映像引用中定义的磁盘指定用户映像替代。  |
|  InvalidParameter  |  名为“{0}”的磁盘已在使用相同的 VHD URL {1}。  |
|  InvalidParameter  |  指定的容错域计数 {0} 必须在 {1} 到 {2} 的范围内。  |
|  InvalidParameter  |  许可证类型 {0} 无效。 有效的许可证类型为: Windows_Client 或 Windows_Server，区分大小写。  |
|  InvalidParameter  |  Linux 主机名长度不能超过 {0} 个字符，也不能包含以下字符: {1}。  |
|  InvalidParameter  |  因为 Linux 预配代理中的一个已知问题，SSH 公钥的目标路径当前限制为其默认值 {0}。  |
|  InvalidParameter  |  LUN {0} 处已存在磁盘。  |
|  InvalidParameter  |  请求的订阅 {0} 必须与托管磁盘 ID 中包含的订阅 {1} 匹配。  |
|  InvalidParameter  |  OSProfile 中的自定义数据必须采用 Base64 编码，且最大长度为 {0} 个字符。  |
|  InvalidParameter  |  URL {0} 中的 blob 名称必须以“{1}”扩展结尾。  |
|  InvalidParameter  |  “{0}”不是有效的已捕获 VHD Blob 名称前缀。 有效的前缀与正则表达式“{1}”匹配。  |
|  InvalidParameter  |  如果未预配 VM 代理，则无法将证书添加到 VM。  |
|  InvalidParameter  |  LUN {0} 处已存在磁盘。  |
|  InvalidParameter  |  无法创建 VM，因为请求的大小 {0} 不可用于当前分配有可用性集的群集。 可用的大小为: {1}。 访问 https://aka.ms/azure-resizevm 详细了解重设 VM 大小的策略。  |
|  InvalidParameter  |  请求的 VM 大小 {0} 不可用于当前区域。 可用于当前区域的大小为: {1}。 访问 https://aka.ms/azure-regions 详细了解每个区域可用的 VM 大小。  |
|  InvalidParameter  |  请求的 VM 大小 {0} 不可用于当前区域。 访问 https://aka.ms/azure-regions 详细了解每个区域可用的 VM 大小。  |
|  InvalidParameter  |  Windows 管理员用户名的长度不能超过 {0} 个字符，并且不能以句点(.)结束或包含以下字符: {1}。  |
|  InvalidParameter  |  Windows 计算机名的长度不能超过 {0} 个字符，并且不能全部都是数字或包含以下字符: {1}。  |
|  MissingMoveDependentResources  |  移动资源请求不包含所有从属资源。 请查看详细信息了解缺少的资源 ID。  |
|  MoveResourcesHaveInvalidState  |  移动资源请求包含的 VM 与无效的存储帐户相关。 请查看详细信息获取这些资源 ID 以及所引用的存储帐户名。  |
|  MoveResourcesHavePendingOperations  |  移动资源请求包含操作处于挂起状态的资源。 请查看详情获取这些资源 ID。 请在挂起操作完成后重试操作。  |
|  MoveResourcesNotFound  |  移动资源请求包含无法找到的资源。 请查看详情获取这些资源 ID。  |
|  NetworkingInternalOperationError  |  未知网络分配错误。  |
|  NetworkingInternalOperationError  |  未知网络分配错误  |
|  NetworkingInternalOperationError  |  处理 VM 的网络配置文件时出现内部错误。  |
|  NotFound  |  找不到可用性集 {0}。  |
|  NotFound  |  此 Azure 位置不存在请求中指定的源虚拟机“{0}”。  |
|  NotFound  |  找不到 ID 为 {0} 的租户。  |
|  NotFound  |  找不到映像 {0}。  |
|  NotSupported  |  许可证类型为 {0}，但映像 Blob {1} 不是来自本地。  |
|  OperationNotAllowed  |  无法删除可用性集 {0}。 请先确保可用性集中不包含任何 VM，然后再进行删除。  |
|  OperationNotAllowed  |  不允许将可用性集 SKU 从 'Aligned' 更改为 'Classic'。  |
|  OperationNotAllowed  |  VM 未运行时，无法修改 VM 中的扩展。  |
|  OperationNotAllowed  |  捕获操作仅在含基于 Blob 的磁盘的虚拟机上受支持。 请使用“映像”资源 API 从托管的虚拟机创建映像。  |
|  OperationNotAllowed  |  除非已成功创建了映像，否则不能从映像 {1} 中创建资源 {0}。  |
|  OperationNotAllowed  |  分配 VM 时，不允许更新 encryptionSettings，请在取消分配 VM 后重试  |
|  OperationNotAllowed  |  不支持向具有基于 Blob 磁盘的 VM 添加托管磁盘。  |
|  OperationNotAllowed  |  此大小的 VM 允许连接的数据磁盘数量最多为 {0}。  |
|  OperationNotAllowed  |  不支持向具有托管磁盘的 VM 添加基于 Blob 的磁盘。  |
|  OperationNotAllowed  |  由于映像已标记为待删除，因此不允许在映像“{1}”上执行“{0}”操作。 只能重试 Delete 操作(或等待当前操作完成)。  |
|  OperationNotAllowed  |  VM“{1}”上不允许操作“{0}”，因为 VM 已通用化。  |
|  OperationNotAllowed  |  不允许执行“{0}”操作，因为还原点集合“{1}”已标记为待删除。  |
|  OperationNotAllowed  |  VM 扩展“{1}”上不允许操作“{0}”，因为已将其标记为待删除。 只能重试 Delete 操作(或等待当前操作完成)。  |
|  OperationNotAllowed  |  由于正在使用映像“{2}”预配虚拟机“{1}”，不允许执行操作“{0}”。  |
|  OperationNotAllowed  |  由于虚拟机规模集“{1}”当前正在使用映像“{2}”，因此不允许操作“{0}”。  |
|  OperationNotAllowed  |  VM“{1}”上不允许操作“{0}”，因为已将 VM 标记为待删除。 只能重试 Delete 操作(或等待当前操作完成)。  |
|  OperationNotAllowed  |  VM“{1}”上不允许操作“{0}”，因为已将 VM 取消分配或标记为已取消分配。  |
|  OperationNotAllowed  |  VM“{1}”上不允许操作“{0}”，因为 VM 正在运行。 请明确关机，以防从来宾操作系统内部关闭 VM。  |
|  OperationNotAllowed  |  不允许在 VM“{1}”上执行“{0}”操作，因为 VM 未解除分配。  |
|  OperationNotAllowed  |  VM“{1}”上不允许操作“{0}”，因为 VM 的扩展“{2}”处于失败状态。  |
|  OperationNotAllowed  |  VM“{1}”上不允许操作“{0}”，因为另一个操作正在进行中。  |
|  OperationNotAllowed  |  操作“{0}”需要通用化虚拟机“{1}”。  |
|  OperationNotAllowed  |  该操作要求 VM 处于运行状态(或设置为运行)。  |
|  OperationNotAllowed  |  不允许使用大小为 {0}GB 的磁盘，它小于映像中对应磁盘的大小({1}GB)。  |
|  OperationNotAllowed  |  只有在创建 VM 规模集时才可添加处理程序“{0}”的 VM 规模集扩展。  |
|  OperationNotAllowed  |  只有在删除 VM 规模集时才可删除处理程序“{0}”的 VM 规模集扩展。  |
|  OperationNotAllowed  |  VM“{0}”已在使用托管磁盘。  |
|  OperationNotAllowed  |  VM“{0}”属于 'Classic' 可用性集“{1}”。 请将可用性集更新为使用 'Aligned' SKU，然后重试转换。  |
|  OperationNotAllowed  |  从映像创建的 VM 不能具有基于 Blob 的磁盘。 所有磁盘都必须是托管磁盘。  |
|  OperationNotAllowed  |  捕获操作无法完成，因为 VM 未通用化。  |
|  OperationNotAllowed  |  不允许在 VM“{0}”上执行管理操作，因 VM 磁盘将被转换为托管磁盘。  |
|  OperationNotAllowed  |  有一个进行中的操作正在将虚拟机的电源状态从 {0} 改为 {1}。 请稍后执行操作 {2}。  |
|  OperationNotAllowed  |  无法添加或更新 VM。 请求的 VM 大小 {0} 可能不可用于现有分配单元。 访问 https://aka.ms/azure-resizevm 详细了解重设 VM 大小的策略。  |
|  OperationNotAllowed  |  无法重设 VM 大小，因为请求的大小 {0} 不可用于当前分配有可用性集的群集。 可用的大小为: {1}。 访问 https://aka.ms/azure-resizevm 详细了解重设 VM 大小的策略。  |
|  OperationNotAllowed  |  无法重设 VM 大小，因为请求的大小 {0} 不可用于当前分配有 VM 的群集。 若要将 VM 的大小调整为 {1}，请解除分配（这在 Azure 门户中为“停止”操作），然后重试调整大小操作。 访问 https://aka.ms/azure-resizevm 详细了解重设 VM 大小的策略。  |
|  OSProvisioningClientError  |  VM“{0}”的 OS 预配失败，因为当前正在预配来宾 OS。  |
|  OSProvisioningClientError  |  VM“{0}”的 OS 预配失败。 错误详细信息: {1} 请确保已正确准备(通用化)映像。 <ul><li>适用于 Windows 的说明：https://www.azure.cn/documentation/articles/virtual-machines-windows-upload-image/  </li></ul> |
|  OSProvisioningClientError  |  SSH 主机密钥生成失败。 错误详细信息: {0}。 若要解决此问题，请验证 Linux 代理是否设置正确。 <ul><li>可以查看以下位置的说明：https://www.azure.cn/documentation/articles/virtual-machines-linux-agent-user-guide/ </li></ul> |
|  OSProvisioningClientError  |  为 VM 指定的用户名对于此 Linux 分发版无效。 错误详细信息: {0}。  |
|  OSProvisioningInternalError  |  VM“{0}”的 OS 预配因内部错误而失败。  |
|  OSProvisioningTimedOut  |  VM“{0}”的 OS 预配未在分配的时间内完成。 该 VM 仍可能成功完成预配。 请于稍后检查预配状态。  |
|  OSProvisioningTimedOut  |  VM“{0}”的 OS 预配未在分配的时间内完成。 该 VM 仍可能成功完成预配。 请于稍后检查预配状态。 此外，请确保已正确准备(通用化)映像。   <ul><li>适用于 Windows 的说明：https://www.azure.cn/documentation/articles/virtual-machines-windows-upload-image/ </li><li> 适用于 Linux 的说明：https://www.azure.cn/documentation/articles/virtual-machines-linux-capture-image/</li></ul>  |
|  OSProvisioningTimedOut  |  VM“{0}”的 OS 预配未在分配的时间内完成。 但是检测到 VM 来宾代理正在运行。 这表示来宾 OS 尚未准备好用作 VM 映像(其中 CreateOption=FromImage)。 若要解决此问题，可以按原样使用 VHD 并使 CreateOption=Attach，或者对其进行适当准备使其可用作映像:   <ul><li>适用于 Windows 的说明：https://www.azure.cn/documentation/articles/virtual-machines-windows-upload-image/ </li><li> 适用于 Linux 的说明：https://www.azure.cn/documentation/articles/virtual-machines-linux-capture-image/</li></ul>  |
|  OverConstrainedAllocationRequest  |  所需的 VM 大小当前在所选位置中不可用。  |
|  ResourceUpdateBlockedOnPlatformUpdate  |  此时无法更新资源，因为正在更新平台。 请稍后重试。  |
|  StorageAccountLimitation  |  存储帐户“{0}”不支持创建磁盘所需的页 Blob。  |
|  StorageAccountLimitation  |  存储帐户“{0}”已超出向其分配的配额。  |
|  StorageAccountLocationMismatch  |  无法解析存储帐户 {0}。 请确保通过与计算资源位于相同位置的存储资源提供程序创建该帐户。  |
|  StorageAccountNotFound  |  找不到存储帐户 {0}。 请确保存储帐户未被删除，且与 VM 位于相同的 Azure 位置。  |
|  StorageAccountNotRecognized  |  请使用由存储资源提供程序托管的存储帐户。 不支持使用 {0}。  |
|  StorageAccountOperationInternalError  |  访问存储帐户 {0} 时出现内部错误。  |
|  StorageAccountSubscriptionMismatch  |  存储帐户 {0} 不属于订阅 {1}。  |
|  StorageAccountTooBusy  |  存储帐户“{0}”当前太忙。 请考虑使用其他帐户。  |
|  StorageAccountTypeNotSupported  |  磁盘 {0} 使用的 {1} 是 Blob 存储帐户。 请使用常规用途存储帐户重试。  |
|  StorageAccountTypeNotSupported  |  存储帐户 {0} 的类型为 {1}。 启动诊断支持 {2} 种存储帐户类型。  |
|  SubscriptionNotAuthorizedForImage  |  未对订阅授权。  |
|  TargetDiskBlobAlreadyExists  |  Blob {0} 已存在。 请提供其他 blob URI 来创建新的空白数据磁盘“{1}”。  |
|  TargetDiskBlobAlreadyExists  |  无法继续捕获操作，因为目标映像 Blob {0} 已存在，并且未设置覆盖 VHD Blob 的标志。 请删除 Blob 或设置覆盖 VHD Blob 的标志，然后重试。  |
|  TargetDiskBlobAlreadyExists  |  捕获操作无法继续，因为目标映像 Blob {0} 对其具有活动租约。   |
|  TargetDiskBlobAlreadyExists  |  Blob {0} 已存在。 请提供不同的 Blob URI 作为磁盘“{1}”的目标。  |
|  TooManyVMRedeploymentRequests  |  为 VM“{0}”或与此 VM 位于同一可用性集中的 VM 收到过多重新部署请求。 请稍后重试。  |
|  VHDSizeInvalid  |  为具有 Blob {2} 的磁盘“{1}”指定的磁盘大小值 {0} 无效。 磁盘大小必须在 {3} 和 {4} 之间。  |
|  VMAgentStatusCommunicationError  |  VM“{0}”尚未报告 VM 代理或扩展的状态。 请验证 VM 是否有正在运行的 VM 代理，并可与 Azure 存储建立出站连接。  |
|  VMArtifactRepositoryInternalError  |  与项目存储库通信以检索 VM 项目详细信息时出错。  |
|  VMArtifactRepositoryInternalError  |  从项目存储库检索 VM 项目数据时出现内部错误。  |
|  VMExtensionHandlerNonTransientError  |  处理程序“{0}”报告了 VM 扩展“{1}”的故障，终端错误代码为“{2}”，错误消息为“{3}”  |
|  VMExtensionManagementInternalError  |  处理 VM 扩展“{0}”时出现内部错误。  |
|  VMExtensionManagementInternalError  |  准备 VM 扩展时出现多个错误。 有关详细信息，请参阅 VM 扩展实例视图。  |
|  VMExtensionProvisioningError  |  VM 在处理扩展“{0}”时报告失败。 错误消息:“{1}”。  |
|  VMExtensionProvisioningError  |  未能在 VM 上预配多个 VM 扩展。 有关详细信息，请参阅 VM 扩展实例视图。  |
|  VMExtensionProvisioningTimeout  |  预配 VM 扩展“{0}”超时。 扩展安装可能耗时过长，或无法获得扩展状态。  |
|  VMMarketplaceInvalidInput  |  从非应用商店映像创建虚拟机无需计划信息，请删除请求中的计划信息。 OS 磁盘名称为 {0}。  |
|  VMMarketplaceInvalidInput  |  购买信息不匹配。 无法从应用商店映像部署。 OS 磁盘名称为 {0}。  |
|  VMMarketplaceInvalidInput  |  从应用商店映像创建虚拟机需要请求中的计划信息。 OS 磁盘名称为 {0}。  |
|  VMNotFound  |  找不到 VM“{0}”。  |
|  VMRedeploymentFailed  |  VM“{0}”的重新部署因内部错误而失败。 请稍后重试。  |
|  VMRedeploymentTimedOut  |  VM“{0}”的重新部署未在分配的时间内完成。 不久即可成功完成。 否则，你可以重试该请求。  |
|  VMStartTimedOut  |  VM“{0}”未在分配的时间内启动。 该 VM 仍可能成功启动。 请于稍后检查电源状态。  |