<properties
	pageTitle="在 HDInsight 中调试 Hadoop：查看日志和解释错误消息 | Azure"
	description="了解你在使用 PowerShell 管理 HDInsight 时可能会收到的错误消息，以及恢复正常的步骤。"
	services="hdinsight"
	tags="azure-portal"
	editor="cgronlun"
	manager="paulettm"
	authors="mumian"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.date="01/28/2016"
	wacn.date="03/28/2016"/>

# 在 HDInsight 中调试 Hadoop：查看日志和解释错误消息

本主题中列举的错误消息旨在帮助 Azure HDInsight 中的 Hadoop 用户了解在使用 Azure PowerShell 管理服务时可能会遇到的错误情况，并向他们建议从错误中恢复时可以执行哪些步骤。

## <a id="hdi-error-codes"></a>HDInsight 错误代码

用户在 Azure PowerShell 或管理门户中可能会遇到的错误在下面按名称的字母顺序列出。这些错误反过来又会链接到[错误描述和缓解](#discription-mitigation-errors)一节中提供该错误的以下信息的相应条目：

## <a id="discription-mitigation-errors"></a>错误的诊断和缓解措施


### <a id="AtleastOneSqlMetastoreMustBeProvided"></a>AtleastOneSqlMetastoreMustBeProvided
- **说明**：请至少为一个组件提供 Azure SQL 数据库以便对配置单元和 Oozie 元存储使用自定义设置。
- **缓解**：用户需要提供有效的 SQL Azure 元存储，然后重试该请求。  

### <a id="AzureRegionNotSupported"></a>AzureRegionNotSupported
- **说明**：无法在区域 *nameOfYourRegion* 中创建群集。使用有效的 HDInsight 区域并且重试请求。
- **缓解**：用户应该创建当前支持它们的群集区域：中国北部、中国东部。  

### <a id="ClusterContainerRecordNotFound"></a>ClusterContainerRecordNotFound
- **说明**：服务器无法找到请求的群集记录。  
- **缓解**：重试操作。

### <a id="ClusterDnsNameInvalidReservedWord"></a>ClusterDnsNameInvalidReservedWord
- **说明**：群集 DNS 名称 *yourDnsName* 无效。请确保名称以字母数字开头和结尾，并且只能包含“-”特殊符号  
- **缓解**：确保你将有效的 DNS 名称用于群集，该名称以字母数字开头和结尾，并且不包含除了短划线“-”之外的任何特殊字符，然后重试操作。

### <a id="ClusterNameUnavailable"></a>ClusterNameUnavailable
- **说明**：群集名称 *yourClusterName* 不可用。请选取另一个名称。  
- **缓解**：用户应该指定唯一且不存在的群集名称，然后重试。如果用户正在使用门户，则 UI 将通知他们该群集名称是否已在创建步骤期间使用。


### <a id="ClusterPasswordInvalid"></a>ClusterPasswordInvalid
- **说明**：群集密码无效。密码的长度必须至少为 10 个字符，并且必须包含至少一个数字、大写字母、小写字母和特殊字符且没有空格，不应包含用户名作为密码的一部分。  
- **缓解**：提供有效的群集密码，然后重试操作。

### <a id="ClusterUserNameInvalid"></a>ClusterUserNameInvalid
- **说明**：群集用户名无效。请确保用户名不包含特殊字符或空格。  
- **缓解**：提供有效的群集用户名，然后重试操作。

### <a id="ClusterUserNameInvalidReservedWord"></a>ClusterUserNameInvalidReservedWord
- **说明**：群集 DNS 名称 *yourDnsClusterName* 无效。请确保名称以字母数字开头和结尾，并且只能包含“-”特殊符号  
- **缓解**：提供有效的 DNS 群集用户名，然后重试操作。

### <a id="ContainerNameMisMatchWithDnsName"></a>ContainerNameMisMatchWithDnsName
- **说明**：URI *yourcontainerURI* 中的容器名称和请求正文中的 DNS 名称 *yourDnsName* 必须相同。  
- **缓解**：确保容器名称和 DNS 名称相同，然后重试操作。

### <a id="DataNodeDefinitionNotFound"></a>DataNodeDefinitionNotFound
- **说明**：无效的群集配置。在节点大小中找不到任何数据节点定义。  
- **缓解**：重试操作。

### <a id="DeploymentDeletionFailure"></a>DeploymentDeletionFailure
- **说明**：针对群集的部署删除失败  
- **缓解**：重试删除操作。

### <a id="DnsMappingNotFound"></a>DnsMappingNotFound
- **说明**：服务配置错误。未找到请求的 DNS 映射信息。  
- **缓解**：删除群集，然后创建一个新群集。

### <a id="DuplicateClusterContainerRequest"></a>DuplicateClusterContainerRequest
- **说明**：重复群集容器创建尝试。存在针对 *nameOfYourContainer* 的记录，但 Etags 不匹配。
- **缓解**：为容器提供唯一名称，然后重试创建操作。

### <a id="DuplicateClusterInHostedService"></a>DuplicateClusterInHostedService
- **说明**：托管服务 *nameOfYourHostedService* 已包含群集。托管服务不能包含多个群集  
- **缓解**：在其他托管服务中托管群集。

### <a id="FailureToUpdateDeploymentStatus"></a>FailureToUpdateDeploymentStatus
- **说明**：服务器无法更新群集部署的状态。  
- **缓解**：重试操作。如果此情况多次发生，请与 CSS 联系。

### <a id="HdiRestoreClusterAltered"></a>HdiRestoreClusterAltered
- **说明**：作为维护的一部分删除了群集 *yourClusterName*。请重新创建群集。
- **缓解**：重新创建群集。

### <a id="HeadNodeConfigNotFound"></a>HeadNodeConfigNotFound
- **说明**：无效的群集配置。在节点大小中找不到所需头节点配置。
- **缓解**：重试操作。

### <a id="HostedServiceCreationFailure"></a>HostedServiceCreationFailure
- **说明**：无法创建托管服务 *nameOfYourHostedService*。请重试请求。  
- **缓解**：重试请求。

### <a id="HostedServiceHasProductionDeployment"></a>HostedServiceHasProductionDeployment
- **说明**：托管服务 *nameOfYourHostedService* 已有生产部署。托管服务不能包含多个生产部署。请使用不同的群集名称重试请求。
- **缓解**：使用不同的群集名称重试请求。

### <a id="HostedServiceNotFound"></a>HostedServiceNotFound
- **说明**：无法找到群集的托管服务 *nameOfYourHostedService*。  
- **缓解**：如果该群集处于错误状态，则删除该群集，然后重试。

### <a id="HostedServiceWithNoDeployment"></a>HostedServiceWithNoDeployment
- **说明**：托管服务 *nameOfYourHostedService* 没有关联的部署。  
- **缓解**：如果该群集处于错误状态，则删除该群集，然后重试。

### <a id="InsufficientResourcesCores"></a>InsufficientResourcesCores
- **说明**：SubscriptionId *yourSubscriptionId* 没有可供创建群集 *yourClusterName* 的内核。必需：*resourcesRequired*，可用：*resourcesAvailable*。  
- **缓解**：释放你的订阅中的资源或者增加可用于订阅的资源，然后尝试再次创建群集。

### <a id="InsufficientResourcesHostedServices"></a>InsufficientResourcesHostedServices
- **说明**：订阅 ID *yourSubscriptionId* 没有用于新 HostedService 的配额以便创建群集 *yourClusterName*。  
- **缓解**：释放你的订阅中的资源或者增加可用于订阅的资源，然后尝试再次创建群集。

### <a id="InternalErrorRetryRequest"></a>InternalErrorRetryRequest
- **说明**：服务器遇到内部错误。请重试请求。  
- **缓解**：重试请求。

### <a id="InvalidAzureStorageLocation"></a>InvalidAzureStorageLocation
- **说明**：Azure 存储位置 *dataRegionName* 不是有效位置。请确保区域正确并重试请求。
- **缓解**：选择支持 HDInsight 的存储位置，检查你的群集是否是共置的，然后重试操作。

### <a id="InvalidNodeSizeForDataNode"></a>InvalidNodeSizeForDataNode
- **说明**：数据节点的 VM 大小无效。所有数据节点仅支持“大型 VM”大小。  
- **缓解**：指定数据节点支持的节点大小，然后重试操作。

### <a id="InvalidNodeSizeForHeadNode"></a>InvalidNodeSizeForHeadNode
- **说明**：头节点的 VM 大小无效。头节点仅支持“特大型 VM”大小。  
- **缓解**：指定头节点支持的节点大小，然后重试操作。

### <a id="InvalidRightsForDeploymentDeletion"></a>InvalidRightsForDeploymentDeletion
- **说明**：要使用的订阅 ID *yourSubscriptionId* 没有对群集 *yourClusterName* 执行删除操作所需的足够权限。  
- **缓解**：如果该群集处于错误状态，则删除该群集，然后重试。  

### <a id="InvalidStorageAccountBlobContainerName"></a>InvalidStorageAccountBlobContainerName
- **说明**：外部存储帐户 Blob 容器名称 *yourContainerName* 无效。请确保名称以字母开头，并且仅包含小写字母、数字和短划线。  
- **缓解**：指定有效的存储帐户 Blob 容器名称，然后重试操作。

### <a id="InvalidStorageAccountConfigurationSecretKey"></a>InvalidStorageAccountConfigurationSecretKey
- **说明**：针对外部存储帐户 *yourStorageAccountName* 的配置需要设置密钥详细信息。  
- **缓解**：为存储帐户指定有效密钥，然后重试操作。

### <a id="InvalidVersionHeaderFormat"></a>InvalidVersionHeaderFormat
- **说明**：版本标头 *yourVersionHeader* 的格式不是有效的 yyyy-mm-dd 格式。  
- **缓解**：为版本标头指定有效格式，然后重试请求。

### <a id="MoreThanOneHeadNode"></a>MoreThanOneHeadNode
- **说明**：无效的群集配置。找到了多个头节点配置。  
- **缓解**：对配置进行编辑，以便仅指定一个头节点。

### <a id="OperationTimedOutRetryRequest"></a>OperationTimedOutRetryRequest
- **说明**：无法在允许的时间内或可能的最大重试尝试次数内完成该操作。请重试请求。  
- **缓解**：重试请求。

### <a id="ParameterNullOrEmpty"></a>ParameterNullOrEmpty
- **说明**：参数 *yourParameterName* 不能为 Null 或为空。  
- **缓解**：为该参数指定有效值。

### <a id="PreClusterCreationValidationFailure"></a>PreClusterCreationValidationFailure
- **说明**：一个或多个群集创建请求输入是无效的。请确保输入值正确，然后重试请求。  
- **缓解**：请确保输入值正确，然后重试请求。

### <a id="RegionCapabilityNotAvailable"></a>RegionCapabilityNotAvailable
- **说明**：区域容量不可用于 *yourRegionName* 和订阅 ID *yourSubscriptionId*。  
- **缓解**：指定支持 HDInsight 群集的区域。公开支持的区域：中国北部、中国东部。


### <a id="StorageAccountNotColocated"></a>StorageAccountNotColocated
- **说明**：存储帐户 *yourStorageAccountName* 位于区域 *currentRegionName* 中。它应该与群集区域 *yourClusterRegionName* 相同。  
- **缓解**：在与你的群集所在区域相同的区域中指定存储帐户；或者如果你的数据已处于该存储帐户中，则在与现有存储帐户相同的区域中创建新群集。


### <a id="SubscriptionIdNotActive"></a>SubscriptionIdNotActive
- **说明**：给定的订阅 ID *yourSubscriptionId* 不是活动的。  
- **缓解**：重新激活你的订阅，或者获取新的有效订阅。

### <a id="SubscriptionIdNotFound"></a>SubscriptionIdNotFound
- **说明**：找不到订阅 ID *yourSubscriptionId*。  
- **缓解**：检查你的订阅 ID 是否有效，然后重试操作。

### <a id="UnableToResolveDNS"></a>UnableToResolveDNS
- **说明**：无法解析 DNS *yourDnsUrl*。请确保提供针对 Blob 终结点的完全限定 URL。  
- **缓解**：提供有效的 Blob URL。该 URL 必须完全有效，包括以 *http://* 开头和以 *.com* 结尾。

### <a id="UnableToVerifyLocationOfResource"></a>UnableToVerifyLocationOfResource
- **说明**：无法验证资源 *yourDnsUrl* 的位置。请确保提供针对 Blob 终结点的完全限定 URL。  
- **缓解**：提供有效的 Blob URL。该 URL 必须完全有效，包括以 *http://* 开头和以 *.com* 结尾。

### <a id="VersionCapabilityNotAvailable"></a>VersionCapabilityNotAvailable
- **说明**：版本功能不可用于版本 *specifiedVersion* 和订阅 ID *yourSubscriptionId*。  
- **缓解**：选择一个可用版本，然后重试操作。

### <a id="VersionNotSupported"></a>VersionNotSupported
- **说明**：不受支持的版本 *specifiedVersion*。
- **缓解**：选择一个支持的版本，然后重试操作。

### <a id="VersionNotSupportedInRegion"></a>VersionNotSupportedInRegion
- **说明**：版本 *specifiedVersion* 在 Azure 区域 *specifiedRegion* 中不可用。  
- **缓解**：选择一个在指定的区域中支持的版本，然后重试操作。

### <a id="WasbAccountConfigNotFound"></a>WasbAccountConfigNotFound
- **说明**：无效的群集配置。在外部帐户中找不到所需的 WASB 帐户配置。  
- **缓解**：确认该帐户存在并且在配置中正确指定，然后重试操作。

## <a id="resources"></a>其他调试资源

* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation]

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/azure/dn469975.aspx

[image-hdi-debugging-error-messages-portal]: ./media/hdinsight-debug-jobs/hdi-debug-errormessages-portal.png

<!---HONumber=79-->