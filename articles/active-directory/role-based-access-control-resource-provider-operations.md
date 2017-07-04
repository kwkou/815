<properties
    pageTitle="Azure资源管理器提供程序操作 | Azure"
    description="可对 Azure资源管理器资源提供程序使用的操作的详细信息"
    services="active-directory"
    documentationcenter=""
    author="jboeshart"
    manager="" />
<tags
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="04/28/2017"
    ms.author="jaboes"
    wacn.date="06/12/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="a969653bf368ccd8fd6de57100fbf8eab990c26a"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="azure-resource-manager-resource-provider-operations"></a>Azure资源管理器资源提供程序操作

本文档列出可对每个 Azure资源管理器资源提供程序使用的操作。 可在自定义角色中使用这些操作，针对 Azure 中的资源提供基于角色的访问控制 (RBAC) 的精细权限。 请注意，此列表并不详尽，随着每个提供程序的更新，可能会添加或删除操作。 操作字符串遵循格式 `Microsoft.<ProviderName>/<ChildResourceType>/<action>`。 如需最新的详尽列表，请使用 `Get-AzureRmProviderOperation`（在 PowerShell 中）或 `azure provider operations show`（在 Azure CLI 中）列出 Azure 资源提供程序的操作。

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

| 操作 | 说明 |
|---|---|
|/configuration/action|更新租户配置。|
|/services/action|更新租户中的服务实例。|
|/configuration/write|创建租户配置。|
|/configuration/read|读取租户配置。|
|/services/write|在租户中创建服务实例。|
|/services/read|读取租户中的服务实例。|
|/services/delete|删除租户中的服务实例。|
|/services/servicemembers/action|在服务中创建服务成员实例。|
|/services/servicemembers/read|读取服务中的服务成员实例。|
|/services/servicemembers/delete|删除服务中的服务成员实例。|
|/services/servicemembers/alerts/read|读取服务成员的警报。|
|/services/alerts/read|读取服务的警报。|
|/services/alerts/read|读取服务的警报。|

## <a name="microsoftadvisor"></a>Microsoft.Advisor

| 操作 | 说明 |
|---|---|
|/generateRecommendations/action|生成建议|
|/suppressions/action|创建/更新禁止显示|
|/register/action|注册 Microsoft 顾问的订阅|
|/generateRecommendations/read|获取“生成建议”状态|
|/recommendations/read|读取建议|
|/suppressions/read|获取禁止显示|
|/suppressions/delete|删除禁止显示|

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

| 操作 | 说明 |
|---|---|
|/servers/read|检索指定 Analysis Server 的信息。|
|/servers/write|创建或更新指定的 Analysis Server。|
|/servers/delete|删除 Analysis Server。|
|/servers/suspend/action|暂停 Analysis Server。|
|/servers/resume/action|恢复 Analysis Server。|
|/servers/checkNameAvailability<br>/action|检查给定的 Analysis Server 名称是否有效且未被使用。|

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

| 操作 | 说明 |
|---|---|
|/checkNameAvailability/action|检查提供的服务名称是否可用|
|/register/action|注册 Microsoft.ApiManagement 资源提供程序的订阅|
|/unregister/action|取消注册 Microsoft.ApiManagement 资源提供程序的订阅|
|/service/write|创建 API 管理服务的新实例|
|/service/read|读取 API 管理服务实例的元数据|
|/service/delete|删除 API 管理服务实例|
|/service/updatehostname/action|设置、更新或删除 API 管理服务的自定义域名|
|/service/uploadcertificate/action|上传 API 管理服务的 SSL 证书|
|/service/backup/action|将 API 管理服务备份到用户提供的存储帐户中的指定容器|
|/service/restore/action|从用户提供的存储帐户中的指定容器还原 API 管理服务|
|/service/managedeployments/action|更改 API 管理服务的 SKU/单位，以及添加/删除其区域部署|
|/service/getssotoken/action|获取可用于以管理员身份登录到 API 管理服务旧版门户的 SSO 令牌|
|/service/applynetworkconfigurationupdates/action|更新虚拟网络中运行的 Microsoft.ApiManagement 资源，以提取更新的网络设置。|
|/service/operationresults/read|获取长时间运行的操作的当前状态|
|/service/networkStatus/read|获取资源的网络访问状态。|
|/service/loggers/read|获取记录器列表，或获取记录器详细信息|
|/service/loggers/write|添加新记录器，或更新现有记录器详细信息|
|/service/loggers/delete|删除现有的记录器|
|/service/users/read|获取已注册用户的列表，或获取用户的帐户详细信息|
|/service/users/write|注册新用户，或更新现有用户的帐户详细信息|
|/service/users/delete|删除用户帐户|
|/service/users/generateSsoUrl/action|生成 SSO URL。 可用于访问管理门户的 URL|
|/service/users/subscriptions/read|获取用户订阅的列表|
|/service/users/keys/read|获取用户密钥的列表|
|/service/users/groups/read|获取用户组的列表|
|/service/tenant/operationResults/read|获取操作结果的列表，或获取特定操作的结果|
|/service/tenant/policy/read|获取租户的策略配置|
|/service/tenant/policy/write|设置租户的策略配置|
|/service/tenant/policy/delete|删除租户的策略配置|
|/service/tenant/configuration/save/action|创建包含配置快照且目标为存储库中指定分支的提交内容|
|/service/tenant/configuration/deploy/action|运行部署任务，以便将指定的 git 分支中的更改应用到数据库中的配置。|
|/service/tenant/configuration/validate/action|验证指定的 git 分支中的更改|
|/service/tenant/configuration/operationResults/read|获取操作结果的列表，或获取特定操作的结果|
|/service/tenant/configuration/syncState/read|获取上次 git 同步的状态|
|/service/tenant/access/read|获取租户访问详细信息|
|/service/tenant/access/write|更新租户访问详细信息|
|/service/tenant/access/regeneratePrimaryKey/action|再生成主访问密钥|
|/service/tenant/access/regenerateSecondaryKey/action|再生成辅助访问密钥|
|/service/identityProviders/read|获取标识提供者的列表，或获取标识提供者的详细信息|
|/service/identityProviders/write|创建新的标识提供者，或更新现有标识提供者的详细信息|
|/service/identityProviders/delete|删除现有的标识提供者|
|/service/subscriptions/read|获取产品订阅的列表，或获取产品订阅的详细信息|
|/service/subscriptions/write|让现有用户订阅现有产品，或更新现有订阅的详细信息。 此操作可用于续订订阅|
|/service/subscriptions/delete|删除订阅。 此操作可用于删除订阅|
|/service/subscriptions/regeneratePrimaryKey/action|再生成订阅主密钥|
|/service/subscriptions/regenerateSecondaryKey/action|再生成订阅辅助密钥|
|/service/backends/read|获取后端列表，或获取后端详细信息|
|/service/backends/write|添加新后端，或更新现有后端详细信息|
|/service/backends/delete|删除现有后端|
|/service/apis/read|获取所有已注册 API 的列表，或获取 API 的详细信息|
|/service/apis/write|创建新 API，或更新现有 API 详细信息|
|/service/apis/delete|删除现有 API|
|/service/apis/policy/read|获取 API 的策略配置详细信息|
|/service/apis/policy/write|设置 API 的策略配置详细信息|
|/service/apis/policy/delete|从 API 中删除策略配置|
|/service/apis/operations/read|获取现有 API 操作的列表，或获取 API 操作的详细信息|
|/service/apis/operations/write|创建新的 API 操作，或更新现有的 API 操作|
|/service/apis/operations/delete|删除现有的 API 操作|
|/service/apis/operations/policy/read|获取操作的策略配置详细信息|
|/service/apis/operations/policy/write|设置操作的策略配置详细信息|
|/service/apis/operations/policy/delete|从操作中删除策略配置|
|/service/products/read|获取产品列表，或获取产品的详细信息|
|/service/products/write|创建新产品，或更新现有产品详细信息|
|/service/products/delete|删除现有产品|
|/service/products/subscriptions/read|获取产品订阅的列表|
|/service/products/apis/read|获取已添加到现有产品的 API 列表|
|/service/products/apis/write|将现有 API 添加到现有产品|
|/service/products/apis/delete|从现有产品中删除现有 API|
|/service/products/policy/read|获取现有产品的策略配置|
|/service/products/policy/write|设置现有产品的策略配置|
|/service/products/policy/delete|从现有产品中删除策略配置|
|/service/products/groups/read|获取与产品关联的开发人员组的列表|
|/service/products/groups/write|将现有开发人员组与现有产品相关联|
|/service/products/groups/delete|删除现有开发人员组与现有产品之间的关联|
|/service/openidConnectProviders/read|获取 OpenID Connect 提供程序的列表，或获取 OpenID Connect 提供程序的详细信息|
|/service/openidConnectProviders/write|创建新的 OpenID Connect 提供程序，或更新现有 OpenID Connect 提供程序的详细信息|
|/service/openidConnectProviders/delete|删除现有的 OpenID Connect 提供程序|
|/service/certificates/read|获取证书列表，或获取证书的详细信息|
|/service/certificates/write|添加新证书|
|/service/certificates/delete|删除现有证书|
|/service/properties/read|获取所有属性的列表，或获取指定属性的详细信息|
|/service/properties/write|创建新属性，或更新指定属性的值|
|/service/properties/delete|删除现有属性|
|/service/groups/read|获取组列表，或获取组的详细信息|
|/service/groups/write|创建新组，或更新现有组详细信息|
|/service/groups/delete|删除现有组|
|/service/groups/users/read|获取组用户列表|
|/service/groups/users/write|将现有用户添加到现有组|
|/service/groups/users/delete|从现有组中删除现有用户|
|/service/authorizationServers/read|获取授权服务器列表，或获取授权服务器的详细信息|
|/service/authorizationServers/write|创建新的授权服务器，或更新现有授权服务器的详细信息|
|/service/authorizationServers/delete|删除现有的授权服务器|
|/service/reports/bySubscription/read|获取按订阅聚合的报告。|
|/service/reports/byRequest/read|获取重新发布数据的请求|
|/service/reports/byOperation/read|获取按操作聚合的报告|
|/service/reports/byGeo/read|获取按地理区域聚合的报告|
|/service/reports/byUser/read|获取按开发人员聚合的报告。|
|/service/reports/byTime/read|获取按时间段聚合的报告|
|/service/reports/byApi/read|获取按 API 聚合的报告|
|/service/reports/byProduct/read|获取按产品聚合的报告。|

## <a name="microsoftappservice"></a>Microsoft.AppService

| 操作 | 说明 |
|---|---|
|/appidentities/Read|返回已注册到网关的资源（网站）。|
|/appidentities/Write|创建新的应用标识。|
|/appidentities/Delete|删除现有的应用标识。|
|/deploymenttemplates/listMetadata/Action|列出与 API 应用包关联的 UI 元数据。|
|/deploymenttemplates/generate/Action|返回用于预配 API 应用实例的部署模板。|
|/gateways/Read|返回网关实例。|
|/gateways/Write|创建新网关，或更新现有的网关。|
|/gateways/Delete|删除现有的网关实例。|
|/gateways/listLoginUris/Action|填充令牌存储并返回 OAuth 登录 URI。|
|/gateways/listKeys/Action|返回网关机密。|
|/gateways/tokens/Write|创建使用给定名称的新 Zumo 令牌。|
|/gateways/registrations/Read|返回已注册到网关的资源（网站）。|
|/gateways/registrations/Write|将资源（网站）注册到网关。|
|/gateways/registrations/Delete|从网关中取消注册资源（网站）。|
|/apiapps/Read|返回 API 应用实例。|
|/apiapps/Write|创建新的 API 应用，或更新现有的 API 应用。|
|/apiapps/Delete|删除现有的 API 应用实例。|
|/apiapps/listStatus/Action|返回 API 应用状态。|
|/apiapps/listKeys/Action|返回 API 应用机密。|
|/apiapps/apidefinitions/Read|返回 API 应用的 API 定义。|

## <a name="microsoftauthorization"></a>Microsoft.Authorization

| 操作 | 说明 |
|---|---|
|/elevateAccess/action|向调用方授予租户范围的“用户访问管理员”访问权限|
|/classicAdministrators/read|读取订阅的管理员。|
|/classicAdministrators/write|在订阅中添加或修改管理员。|
|/classicAdministrators/delete|从订阅中删除管理员。|
|/locks/read|获取指定范围的锁。|
|/locks/write|添加指定范围的锁。|
|/locks/delete|删除指定范围的锁。|
|/policyAssignments/read|获取有关策略分配的信息。|
|/policyAssignments/write|创建指定范围的策略分配。|
|/policyAssignments/delete|删除指定范围的策略分配。|
|/permissions/read|列出调用方在给定范围拥有的所有权限。|
|/roleDefinitions/read|获取有关角色定义的信息。|
|/roleDefinitions/write|使用指定的权限和可分配的范围创建或更新自定义角色定义。|
|/roleDefinitions/delete|删除指定的自定义角色定义。|
|/providerOperations/read|获取可在角色定义中使用的所有资源提供程序的操作。|
|/policyDefinitions/read|获取有关策略定义的信息。|
|/policyDefinitions/write|创建自定义策略定义。|
|/policyDefinitions/delete|删除策略定义。|
|/roleAssignments/read|获取有关角色分配的信息。|
|/roleAssignments/write|创建指定范围的角色分配。|
|/roleAssignments/delete|删除指定范围的角色分配。|

## <a name="microsoftautomation"></a>Microsoft.Automation

| 操作 | 说明 |
|---|---|
|/automationAccounts/read|获取 Azure 自动化帐户|
|/automationAccounts/write|创建或更新 Azure 自动化帐户|
|/automationAccounts/delete|删除 Azure 自动化帐户|
|/automationAccounts/configurations/readContent/action|获取 Azure 自动化 DSC 的内容|
|/automationAccounts/hybridRunbookWorkerGroups/read|读取混合 Runbook 辅助角色资源|
|/automationAccounts/hybridRunbookWorkerGroups/delete|删除混合 Runbook 辅助角色资源|
|/automationAccounts/jobSchedules/read|获取 Azure 自动化作业计划|
|/automationAccounts/jobSchedules/write|创建 Azure 自动化作业计划|
|/automationAccounts/jobSchedules/delete|删除 Azure 自动化作业计划|
|/automationAccounts/connectionTypes/read|获取 Azure 自动化连接类型资产|
|/automationAccounts/connectionTypes/write|创建 Azure 自动化连接类型资产|
|/automationAccounts/connectionTypes/delete|删除 Azure 自动化连接类型资产|
|/automationAccounts/modules/read|获取 Azure 自动化模块|
|/automationAccounts/modules/write|创建或更新 Azure 自动化模块|
|/automationAccounts/modules/delete|删除 Azure 自动化模块|
|/automationAccounts/credentials/read|获取 Azure 自动化凭据资产|
|/automationAccounts/credentials/write|创建或更新 Azure 自动化凭据资产|
|/automationAccounts/credentials/delete|删除 Azure 自动化凭据资产|
|/automationAccounts/certificates/read|获取 Azure 自动化证书资产|
|/automationAccounts/certificates/write|创建或更新 Azure 自动化证书资产|
|/automationAccounts/certificates/delete|删除 Azure 自动化证书资产|
|/automationAccounts/schedules/read|获取 Azure 自动化计划资产|
|/automationAccounts/schedules/write|创建或更新 Azure 自动化计划资产|
|/automationAccounts/schedules/delete|删除 Azure 自动化计划资产|
|/automationAccounts/jobs/read|获取 Azure 自动化作业|
|/automationAccounts/jobs/write|创建 Azure 自动化作业|
|/automationAccounts/jobs/stop/action|停止 Azure 自动化作业|
|/automationAccounts/jobs/suspend/action|暂停 Azure 自动化作业|
|/automationAccounts/jobs/resume/action|恢复 Azure 自动化作业|
|/automationAccounts/jobs/runbookContent/action|执行作业时获取 Azure 自动化 Runbook 的内容|
|/automationAccounts/jobs/output/action|获取作业的输出|
|/automationAccounts/jobs/read|获取 Azure 自动化作业|
|/automationAccounts/jobs/write|创建 Azure 自动化作业|
|/automationAccounts/jobs/stop/action|停止 Azure 自动化作业|
|/automationAccounts/jobs/suspend/action|暂停 Azure 自动化作业|
|/automationAccounts/jobs/resume/action|恢复 Azure 自动化作业|
|/automationAccounts/jobs/streams/read|获取 Azure 自动化作业流|
|/automationAccounts/connections/read|获取 Azure 自动化连接资产|
|/automationAccounts/connections/write|创建或更新 Azure 自动化连接资产|
|/automationAccounts/connections/delete|删除 Azure 自动化连接资产|
|/automationAccounts/variables/read|读取 Azure 自动化变量资产|
|/automationAccounts/variables/write|创建或更新 Azure 自动化变量资产|
|/automationAccounts/variables/delete|删除 Azure 自动化变量资产|
|/automationAccounts/runbooks/readContent/action|获取 Azure 自动化 Runbook 的内容|
|/automationAccounts/runbooks/read|获取 Azure 自动化 Runbook|
|/automationAccounts/runbooks/write|创建或更新 Azure 自动化 Runbook|
|/automationAccounts/runbooks/delete|删除 Azure 自动化 Runbook|
|/automationAccounts/runbooks/draft/readContent/action|获取 Azure 自动化 Runbook 草稿的内容|
|/automationAccounts/runbooks/draft/writeContent/action|创建 Azure 自动化 Runbook 草稿的内容|
|/automationAccounts/runbooks/draft/read|获取 Azure 自动化 Runbook 草稿|
|/automationAccounts/runbooks/draft/publish/action|发布 Azure 自动化 Runbook 草稿|
|/automationAccounts/runbooks/draft/undoEdit/action|撤消对 Azure 自动化 Runbook 草稿的编辑|
|/automationAccounts/runbooks/draft/testJob/read|获取 Azure 自动化 Runbook 草稿测试作业|
|/automationAccounts/runbooks/draft/testJob/write|创建 Azure 自动化 Runbook 草稿测试作业|
|/automationAccounts/runbooks/draft/testJob/stop/action|停止 Azure 自动化 Runbook 草稿测试作业|
|/automationAccounts/runbooks/draft/testJob/suspend/action|暂停 Azure 自动化 Runbook 草稿测试作业|
|/automationAccounts/runbooks/draft/testJob/resume/action|恢复 Azure 自动化 Runbook 草稿测试作业|
|/automationAccounts/webhooks/read|读取 Azure 自动化 Webhook|
|/automationAccounts/webhooks/write|创建或更新 Azure 自动化 Webhook|
|/automationAccounts/webhooks/delete|删除 Azure 自动化 Webhook |
|/automationAccounts/webhooks/generateUri/action|生成 Azure 自动化 Webhook 的 URI|

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

此提供程序不是完整的 ARM 提供程序，不提供任何 ARM 操作。

## <a name="microsoftbatch"></a>Microsoft.Batch

| 操作 | 说明 |
|---|---|
|/register/action|注册 Batch 资源提供程序的订阅，并启用 Batch 帐户的创建|
|/batchAccounts/write|创建新的 Batch 帐户，或更新现有的 Batch 帐户|
|/batchAccounts/read|列出 Batch 帐户，或获取 Batch 帐户的属性|
|/batchAccounts/delete|删除 Batch 帐户|
|/batchAccounts/listkeys/action|列出 Batch 帐户的访问密钥|
|/batchAccounts/regeneratekeys/action|再生成 Batch 帐户的访问密钥|
|/batchAccounts/syncAutoStorageKeys/action|同步针对 Batch 帐户配置的自动存储帐户的访问密钥|
|/batchAccounts/applications/read|列出应用程序，或获取应用程序的属性|
|/batchAccounts/applications/write|创建新的应用程序，或更新现有的应用程序|
|/batchAccounts/applications/delete|删除应用程序|
|/batchAccounts/applications/versions/read|获取应用程序包的属性|
|/batchAccounts/applications/versions/write|创建新的应用程序包，或更新现有的应用程序包|
|/batchAccounts/applications/versions/activate/action|激活应用程序包|
|/batchAccounts/applications/versions/delete|删除应用程序包|
|/locations/quotas/read|获取指定 Azure 区域中指定订阅的 Batch 配额|

## <a name="microsoftbilling"></a>Microsoft.Billing

| 操作 | 说明 |
|---|---|
|/invoices/read|列出提供的发票|

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

| 操作 | 说明 |
|---|---|
|/mapApis/Read|读取操作|
|/mapApis/Write|写入操作|
|/mapApis/Delete|删除操作|
|/mapApis/regenerateKey/action|再生成密钥|
|/mapApis/listSecrets/action|列出机密|
|/mapApis/listSingleSignOnToken/action|读取资源的单一登录授权令牌|
|/Operations/read|操作说明。|

## <a name="microsoftcache"></a>Microsoft.Cache

| 操作 | 说明 |
|---|---|
|/checknameavailability/action|检查名称是否可用于新的 Redis 缓存|
|/register/action|将“Microsoft.Cache”资源提供程序注册到订阅|
|/unregister/action|从订阅中取消注册“Microsoft.Cache”资源提供程序|
|/redis/write|在管理门户中修改 Redis 缓存的设置和配置|
|/redis/read|在管理门户中查看 Redis 缓存的设置和配置|
|/redis/delete|删除整个 Redis 缓存|
|/redis/listKeys/action|在管理门户中查看 Redis 缓存访问密钥的值|
|/redis/regenerateKey/action|在管理门户中更改 Redis 缓存访问密钥的值|
|/redis/import/action|将多个 Blob 中指定格式的数据导入 Redis|
|/redis/export/action|将 Redis 数据以指定的格式导出到带前缀的存储 Blob|
|/redis/forceReboot/action|强制重新启动缓存实例（可能会发生数据丢失）。|
|/redis/stop/action|停止缓存实例。|
|/redis/start/action|启动缓存实例。|
|/redis/metricDefinitions/read|获取 Redis 缓存的可用指标|
|/redis/firewallRules/read|获取 Redis 缓存的 IP 防火墙规则|
|/redis/firewallRules/write|编辑 Redis 缓存的 IP 防火墙规则|
|/redis/firewallRules/delete|删除 Redis 缓存的 IP 防火墙规则|
|/redis/listUpgradeNotifications/read|列出缓存租户的最新升级通知。|
|/redis/linkedservers/read|获取与 Redis 缓存关联的链接服务器。|
|/redis/linkedservers/write|将链接服务器添加到 Redis 缓存|
|/redis/linkedservers/delete|从 Redis 缓存中删除链接服务器|
|/redis/patchSchedules/read|获取 Redis 缓存的修补计划|
|/redis/patchSchedules/write|修改 Redis 缓存的修补计划|
|/redis/patchSchedules/delete|删除 Redis 缓存的修补计划|

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

| 操作 | 说明 |
|---|---|
|/provisionGlobalAppServicePrincipalInUserTenant/Action|为服务应用主体预配服务主体|
|/validateCertificateRegistrationInformation/Action|验证证书购买对象但不提交该对象|
|/register/action|注册订阅的 Microsoft 证书资源提供程序|
|/certificateOrders/Write|添加新的或更新现有的 certificateOrder|
|/certificateOrders/Delete|删除现有的 AppServiceCertificate|
|/certificateOrders/Read|获取 CertificateOrders 列表|
|/certificateOrders/reissue/Action|重新颁发现有的 certificateorder|
|/certificateOrders/renew/Action|续订现有的 certificateorder|
|/certificateOrders/retrieveCertificateActions/Action|检索证书操作的列表|
|/certificateOrders/retrieveEmailHistory/Action|检索证书电子邮件历史记录|
|/certificateOrders/resendEmail/Action|重新发送证书电子邮件|
|/certificateOrders/verifyDomainOwnership/Action|验证域所有权|
|/certificateOrders/resendRequestEmails/Action|将请求电子邮件重新发送到另一个电子邮件地址|
|/certificateOrders/resendRequestEmails/Action|检索颁发的应用服务证书的站点封印|
|/certificateOrders/certificates/Write|添加新的证书，或更新现有的证书|
|/certificateOrders/certificates/Delete|删除现有证书|
|/certificateOrders/certificates/Read|获取证书列表|

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

| 操作 | 说明 |
|---|---|
|/register/action|注册到经典计算|
|/checkDomainNameAvailability/action|检查给定域名的可用性。|
|/moveSubscriptionResources/action|将所有经典资源移到不同的订阅。|
|/validateSubscriptionMoveAvailability/action|验证订阅是否可用于经典移动操作。|
|/operatingSystemFamilies/read|列出 Azure 中可用的来宾操作系统系列，同时列出可用于每个系列的操作系统版本
|/capabilities/read|显示功能|
|/operatingSystems/read|列出 Azure 中当前可用的来宾操作系统版本。|
|/resourceTypes/skus/read|获取受支持资源类型的 SKU 列表。|
|/domainNames/read|返回资源的域名。|
|/domainNames/write|添加或修改资源的域名。|
|/domainNames/delete|删除资源的域名。|
|/domainNames/swap/action|将过渡槽交换到生产槽。|
|/domainNames/serviceCertificates/read|返回使用的服务证书。|
|/domainNames/serviceCertificates/write|添加或修改使用的服务证书。|
|/domainNames/serviceCertificates/delete|删除使用的服务证书。|
|/domainNames/serviceCertificates/operationStatuses/read|读取域名服务证书的操作状态。|
|/domainNames/capabilities/read|显示域名功能|
|/domainNames/extensions/read|返回域名扩展。|
|/domainNames/extensions/write|添加域名扩展。|
|/domainNames/extensions/delete|删除域名扩展。|
|/domainNames/extensions/operationStatuses/read|读取域名扩展的操作状态。|
|/domainNames/active/write|设置活动域名。|
|/domainNames/slots/read|显示部署槽。|
|/domainNames/slots/write|创建或更新部署。|
|/domainNames/slots/delete|删除给定的部署槽。|
|/domainNames/slots/start/action|启动部署槽。|
|/domainNames/slots/stop/action|暂停部署槽。|
|/domainNames/slots/operationStatuses/read|读取域名槽的操作状态。|
|/domainNames/slots/roles/read|获取部署槽的角色。|
|/domainNames/slots/roles/extensionReferences/read|返回部署槽角色的扩展引用。|
|/domainNames/slots/roles/extensionReferences/write|添加或修改部署槽角色的扩展引用。|
|/domainNames/slots/roles/extensionReferences/delete|删除部署槽角色的扩展引用。|
|/domainNames/slots/roles/extensionReferences/operationStatuses/read|读取域名槽角色扩展引用的操作状态。|
|/domainNames/slots/roles/roleInstances/read|获取角色实例。|
|/domainNames/slots/roles/roleInstances/restart/action|重新启动角色实例。|
|/domainNames/slots/roles/roleInstances/reimage/action|重置角色实例的映像。|
|/domainNames/slots/roles/roleInstances/operationStatuses/read|读取域名槽角色角色实例的操作状态。|
|/domainNames/slots/state/start/write|将部署槽状态更改为已停止。|
|/domainNames/slots/state/stop/write|将部署槽状态更改为已启动。|
|/domainNames/slots/upgradeDomain/write|逐步升级域。|
|/domainNames/internalLoadBalancers/read|获取内部负载均衡器。|
|/domainNames/internalLoadBalancers/write|创建新的内部负载均衡。|
|/domainNames/internalLoadBalancers/delete|删除新的内部负载均衡。|
|/domainNames/internalLoadBalancers/operationStatuses/read|读取域名内部负载均衡器的操作状态。|
|/domainNames/loadBalancedEndpointSets/read|显示负载均衡的终结点集|
|/domainNames/loadBalancedEndpointSets/operationStatuses/read|读取域名负载均衡的终结点集的操作状态。|
|/domainNames/availabilitySets/read|显示资源的可用性集。|
|/quotas/read|获取订阅的配额。|
|/virtualMachines/read|检索虚拟机列表。|
|/virtualMachines/write|添加或修改虚拟机。|
|/virtualMachines/delete|删除虚拟机。|
|/virtualMachines/start/action|启动虚拟机。|
|/virtualMachines/redeploy/action|重新部署虚拟机。|
|/virtualMachines/restart/action|重新启动虚拟机。|
|/virtualMachines/stop/action|停止虚拟机。|
|/virtualMachines/shutdown/action|关闭虚拟机。|
|/virtualMachines/attachDisk/action|将数据磁盘附加到虚拟机。|
|/virtualMachines/detachDisk/action|从虚拟机中分离数据磁盘。|
|/virtualMachines/downloadRemoteDesktopConnectionFile/action|下载虚拟机的 RDP 文件。|
|/virtualMachines/networkInterfaces/<br>associatedNetworkSecurityGroups/read|获取与网络接口关联的网络安全组。|
|/virtualMachines/networkInterfaces/<br>associatedNetworkSecurityGroups/write|添加与网络接口关联的网络安全组。|
|/virtualMachines/networkInterfaces/<br>associatedNetworkSecurityGroups/delete|删除与网络接口关联的网络安全组。|
|/virtualMachines/networkInterfaces/<br>associatedNetworkSecurityGroups/operationStatuses/read|读取与网络安全组关联的虚拟机的操作状态。|
|/virtualMachines/providers/Microsoft.Insights/metricDefinitions/read|获取指标定义。|
|/virtualMachines/providers/Microsoft.Insights/diagnosticSettings/read|获取诊断设置。|
|/virtualMachines/providers/Microsoft.Insights/diagnosticSettings/write|添加或修改诊断设置。|
|/virtualMachines/metrics/read|获取指标。|
|/virtualMachines/operationStatuses/read|读取虚拟机的操作状态。|
|/virtualMachines/extensions/read|获取虚拟机扩展。|
|/virtualMachines/extensions/write|放置虚拟机扩展。|
|/virtualMachines/extensions/operationStatuses/read|读取虚拟机扩展的操作状态。|
|/virtualMachines/asyncOperations/read|获取可能的异步操作|
|/virtualMachines/disks/read|检索数据磁盘的列表|
|/virtualMachines/associatedNetworkSecurityGroups/read|获取与虚拟机关联的网络安全组。|
|/virtualMachines/associatedNetworkSecurityGroups/write|添加与虚拟机关联的网络安全组。|
|/virtualMachines/associatedNetworkSecurityGroups/delete|删除与虚拟机关联的网络安全组。|
|/virtualMachines/associatedNetworkSecurityGroups/operationStatuses/read|读取与网络安全组关联的虚拟机的操作状态。|

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

| 操作 | 说明 |
|---|---|
|/register/action|注册到经典网络|
|/gatewaySupportedDevices/read|检索受支持设备的列表。|
|/reservedIps/read|获取保留 IP|
|/reservedIps/write|添加新的保留 IP|
|/reservedIps/delete|删除保留 IP。|
|/reservedIps/link/action|链接保留 IP|
|/reservedIps/join/action|联接保留 IP|
|/reservedIps/operationStatuses/read|读取保留 IP 的操作状态。|
|/virtualNetworks/read|获取虚拟网络。|
|/virtualNetworks/write|添加新的虚拟网络。|
|/virtualNetworks/delete|删除虚拟网络。|
|/virtualNetworks/peer/action|在两个不同的虚拟网络之间建立对等互连。|
|/virtualNetworks/join/action|加入虚拟网络。|
|/virtualNetworks/checkIPAddressAvailability/action|检查虚拟网络中给定 IP 地址的可用性。|
|/virtualNetworks/capabilities/read|显示功能|
|/virtualNetworks/subnets/<br>associatedNetworkSecurityGroups/read|获取与子网关联的网络安全组。|
|/virtualNetworks/subnets/<br>associatedNetworkSecurityGroups/write|添加与子网关联的网络安全组。|
|/virtualNetworks/subnets/<br>associatedNetworkSecurityGroups/delete|删除与子网关联的网络安全组。|
|/virtualNetworks/subnets/<br>associatedNetworkSecurityGroups/operationStatuses/read|读取与网络安全组关联的虚拟网络子网的操作状态。|
|/virtualNetworks/operationStatuses/read|读取虚拟网络的操作状态。|
|/virtualNetworks/gateways/read|获取虚拟网关。|
|/virtualNetworks/gateways/write|添加虚拟网关。|
|/virtualNetworks/gateways/delete|删除虚拟网关。|
|/virtualNetworks/gateways/startDiagnostics/action|启动虚拟网关的诊断。|
|/virtualNetworks/gateways/stopDiagnostics/action|停止虚拟网关的诊断。|
|/virtualNetworks/gateways/downloadDiagnostics/action|下载网关诊断。|
|/virtualNetworks/gateways/listCircuitServiceKey/action|检索线路服务密钥。|
|/virtualNetworks/gateways/downloadDeviceConfigurationScript/action|下载设备配置脚本。|
|/virtualNetworks/gateways/listPackage/action|列出虚拟网关包。|
|/virtualNetworks/gateways/operationStatuses/read|读取虚拟网关的操作状态。|
|/virtualNetworks/gateways/packages/read|获取虚拟网关包。|
|/virtualNetworks/gateways/connections/read|检索连接列表。|
|/virtualNetworks/gateways/connections/connect/action|建立站点到站点网关连接。|
|/virtualNetworks/gateways/connections/disconnect/action|断开站点到站点网关连接。|
|/virtualNetworks/gateways/connections/test/action|测试站点到站点网关连接。|
|/virtualNetworks/gateways/clientRevokedCertificates/read|读取已吊销的客户端证书。|
|/virtualNetworks/gateways/clientRevokedCertificates/write|吊销客户端证书。|
|/virtualNetworks/gateways/clientRevokedCertificates/delete|取消吊销客户端证书。|
|/virtualNetworks/gateways/clientRootCertificates/read|查找客户端根证书。|
|/virtualNetworks/gateways/clientRootCertificates/write|上传新的客户端根证书。|
|/virtualNetworks/gateways/clientRootCertificates/delete|删除虚拟网关客户端证书。|
|/virtualNetworks/gateways/clientRootCertificates/download/action|按指纹下载证书。|
|/virtualNetworks/gateways/clientRootCertificates/listPackage/action|列出虚拟网关证书包。|
|/networkSecurityGroups/read|获取网络安全组。|
|/networkSecurityGroups/write|添加新的网络安全组。|
|/networkSecurityGroups/delete|删除网络安全组。|
|/networkSecurityGroups/operationStatuses/read|读取网络安全组的操作状态。|
|/networkSecurityGroups/securityRules/read|获取安全规则。|
|/networkSecurityGroups/securityRules/write|添加或更新安全规则。|
|/networkSecurityGroups/securityRules/delete|删除安全规则。|
|/networkSecurityGroups/securityRules/operationStatuses/read|读取网络安全组安全规则的操作状态。|
|/quotas/read|获取订阅的配额。|

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

| 操作 | 说明 |
|---|---|
|/register/action|注册到经典存储|
|/checkStorageAccountAvailability/action|检查存储帐户的可用性。|
|/capabilities/read|显示功能|
|/publicImages/read|获取公共虚拟机映像。|
|/images/read|返回映像。|
|/storageAccounts/read|返回包含给定帐户的存储帐户。|
|/storageAccounts/write|添加新的存储帐户。|
|/storageAccounts/delete|删除存储帐户。|
|/storageAccounts/listKeys/action|列出存储帐户的访问密钥。|
|/storageAccounts/regenerateKey/action|再生成存储帐户的现有访问密钥。|
|/storageAccounts/operationStatuses/read|读取资源的操作状态。|
|/storageAccounts/images/read|返回存储帐户映像。|
|/storageAccounts/images/delete|删除给定的存储帐户映像。|
|/storageAccounts/disks/read|返回存储帐户磁盘。|
|/storageAccounts/disks/write|添加存储帐户磁盘。|
|/storageAccounts/disks/delete|删除给定的存储帐户磁盘。|
|/storageAccounts/disks/operationStatuses/read|读取资源的操作状态。|
|/storageAccounts/osImages/read|返回存储帐户操作系统映像。|
|/storageAccounts/osImages/delete|删除给定的存储帐户操作系统映像。|
|/storageAccounts/services/read|获取可用服务。|
|/storageAccounts/services/metricDefinitions/read|获取指标定义。|
|/storageAccounts/services/metrics/read|获取指标。|
|/storageAccounts/services/diagnosticSettings/read|获取诊断设置。|
|/storageAccounts/services/diagnosticSettings/write|添加或修改诊断设置。|
|/disks/read|返回存储帐户磁盘。|
|/osImages/read|返回操作系统映像。|
|/quotas/read|获取订阅的配额。|

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

| 操作 | 说明 |
|---|---|
|/accounts/read|读取 API 帐户。|
|/accounts/write|写入 API 帐户。|
|/accounts/delete|删除 API 帐户|
|/accounts/listKeys/action|列出密钥|
|/accounts/regenerateKey/action|再生成密钥|
|/accounts/skus/read|读取现有资源的可用 SKU。|
|/accounts/usages/read|获取现有资源的配额用量。|
|/Operations/read|操作说明。|

## <a name="microsoftcommerce"></a>Microsoft.Commerce

| 操作 | 说明 |
|---|---|
|/RateCard/read|返回给定订阅的产品数据、资源/计量元数据和费率。|
|/UsageAggregates/read|检索订阅的 Azure 消耗量。 结果包含特定时间范围内的聚合用量数据、订阅和资源相关信息。|

## <a name="microsoftcompute"></a>Microsoft.Compute

| 操作 | 说明 |
|---|---|
|/register/action|将订阅注册到 Microsoft.Compute 资源提供程序|
|/restorePointCollections/read|获取还原点集合的属性|
|/restorePointCollections/write|创建新的或更新现有的还原点集合|
|/restorePointCollections/delete|删除恢复点集合以及包含的还原点|
|/restorePointCollections/restorePoints/read|获取还原点的属性|
|/restorePointCollections/restorePoints/write|创建新的还原点|
|/restorePointCollections/restorePoints/delete|删除还原点|
|/restorePointCollections/restorePoints/retrieveSasUris/action|获取还原点的属性以及 Blob SAS URI|
|/virtualMachineScaleSets/read|获取虚拟机规模集的属性|
|/virtualMachineScaleSets/write|创建新的或更新现有的虚拟机规模集|
|/virtualMachineScaleSets/delete|删除虚拟机规模集|
|/virtualMachineScaleSets/start/action|启动虚拟机规模集的实例|
|/virtualMachineScaleSets/powerOff/action|关闭虚拟机规模集的实例|
|/virtualMachineScaleSets/restart/action|重新启动虚拟机规模集的实例|
|/virtualMachineScaleSets/deallocate/action|关闭并释放虚拟机规模集实例的计算资源 |
|/virtualMachineScaleSets/manualUpgrade/action|手动将实例更新到虚拟机规模集的最新模型|
|/virtualMachineScaleSets/scale/action|缩小/扩大现有虚拟机规模集的实例计数|
|/virtualMachineScaleSets/instanceView/read|获取虚拟机规模集的实例视图|
|/virtualMachineScaleSets/skus/read|列出现有虚拟机规模集的有效 SKU|
|/virtualMachineScaleSets/virtualMachines/read|检索 VM 规模集中虚拟机的属性|
|/virtualMachineScaleSets/virtualMachines/delete|删除 VM 规模集中的特定虚拟机。|
|/virtualMachineScaleSets/virtualMachines/start/action|启动 VM 规模集中的虚拟机实例。|
|/virtualMachineScaleSets/virtualMachines/powerOff/action|关闭 VM 规模集中的虚拟机实例。|
|/virtualMachineScaleSets/virtualMachines/restart/action|重启 VM 规模集中的虚拟机实例。|
|/virtualMachineScaleSets/virtualMachines/deallocate/action|关闭并释放 VM 规模集中虚拟机的计算资源。|
|/virtualMachineScaleSets/virtualMachines/instanceView/read|检索 VM 规模集中虚拟机的实例视图。|
|/images/read|获取映像的属性|
|/images/write|创建新的映像，或更新现有的映像|
|/images/delete|删除映像|
|/operations/read|列出可对 Microsoft.Compute 资源提供程序使用的操作|
|/disks/read|获取磁盘的属性|
|/disks/write|创建新的磁盘，或更新现有的磁盘|
|/disks/delete|删除磁盘|
|/disks/beginGetAccess/action|获取用于 Blob 访问的磁盘 SAS URI|
|/disks/endGetAccess/action|吊销磁盘的 SAS URI|
|/snapshots/read|获取快照的属性|
|/snapshots/write|创建新的快照，或更新现有的快照|
|/snapshots/delete|删除快照|
|/availabilitySets/read|获取可用性集的属性|
|/availabilitySets/write|创建新的可用性集，或更新现有的可用性集|
|/availabilitySets/delete|删除可用性集|
|/availabilitySets/vmSizes/read|列出可在可用性集中创建或更新的虚拟机大小|
|/virtualMachines/read|获取虚拟机的属性|
|/virtualMachines/write|创建新的虚拟机，或更新现有的虚拟机|
|/virtualMachines/delete|删除虚拟机|
|/virtualMachines/start/action|启动虚拟机|
|/virtualMachines/powerOff/action|关闭虚拟机。 请注意，该虚拟机会继续产生费用。|
|/virtualMachines/redeploy/action|重新部署虚拟机|
|/virtualMachines/restart/action|重新启动虚拟机|
|/virtualMachines/deallocate/action|关闭虚拟机并释放计算资源|
|/virtualMachines/generalize/action|将虚拟机状态设置为通用化，并准备要捕获的虚拟机|
|/virtualMachines/capture/action|通过复制虚拟硬盘捕获虚拟机，并生成可用于创建类似虚拟机的模板|
|/virtualMachines/convertToManagedDisks/action|将虚拟机的基于 Blob 的磁盘转换为托管磁盘|
|/virtualMachines/vmSizes/read|列出可将虚拟机更新到的大小|
|/virtualMachines/instanceView/read|获取虚拟机的详细运行时状态及其资源|
|/virtualMachines/extensions/read|获取虚拟机扩展的属性|
|/virtualMachines/extensions/write|创建新的或更新现有的虚拟机扩展|
|/virtualMachines/extensions/delete|删除虚拟机扩展|
|/locations/vmSizes/read|列出某个位置的可用虚拟机大小|
|/locations/usages/read|获取某个位置中订阅的计算资源的服务限制和当前用量|
|/locations/operations/read|获取异步操作的状态|

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

| 操作 | 说明 |
|---|---|
|/register/action|注册容器注册表资源提供程序的订阅，并启用容器注册表的创建|
|/checknameavailability/read|检查注册表名称是否有效且未被使用。|
|/registries/read|返回容器注册表的列表，或获取指定容器注册表的属性。|
|/registries/write|使用指定的参数创建容器注册表，或更新指定容器注册表的属性或标记。|
|/registries/delete|删除现有的容器注册表。|
|/registries/listCredentials/action|列出指定容器注册表的登录凭据。|
|/registries/regenerateCredential/action|再生成指定容器注册表的登录凭据。|

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

| 操作 | 说明 |
|---|---|
|/containerServices/subscriptions/read|根据订阅获取指定的容器服务|
|/containerServices/resourceGroups/read|根据资源组获取指定的容器服务|
|/containerServices/resourceGroups/ContainerServiceName/read|获取指定的容器服务|
|/containerServices/resourceGroups/ContainerServiceName/write|放置或更新指定的容器服务|
|/containerServices/resourceGroups/ContainerServiceName/delete|删除指定的容器服务|

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

| 操作 | 说明 |
|---|---|
|/updateCommunicationPreference/action|更新通信首选项|
|/listCommunicationPreference/action|列出通信首选项|
|/applications/read|读取操作|
|/applications/write|写入操作|
|/applications/write|写入操作|
|/applications/delete|删除操作|
|/applications/listSecrets/action|列出机密|
|/applications/listSingleSignOnToken/action|读取单一登录令牌|
|/operations/read|读取操作|

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

| 操作 | 说明 |
|---|---|
|/hubs/read|读取任何 Azure Customer Insights 中心|
|/hubs/write|创建或更新任何 Azure Customer Insights 中心|
|/hubs/delete|删除任何 Azure Customer Insights 中心|
|/hubs/providers/Microsoft.Insights/metricDefinitions/read|获取资源的可用指标|
|/hubs/providers/Microsoft.Insights/diagnosticSettings/read|获取资源的诊断设置|
|/hubs/providers/Microsoft.Insights/diagnosticSettings/write|创建或更新资源的诊断设置|
|/hubs/providers/Microsoft.Insights/logDefinitions/read|获取资源的可用日志|
|/hubs/authorizationPolicies/read|读取任何 Azure Customer Insights 共享访问签名策略|
|/hubs/authorizationPolicies/write|创建或更新任何 Azure Customer Insights 共享访问签名策略|
|/hubs/authorizationPolicies/delete|删除任何 Azure Customer Insights 共享访问签名策略|
|/hubs/authorizationPolicies/regeneratePrimaryKey/action|再生成 Azure Customer Insights 共享访问签名策略主密钥|
|/hubs/authorizationPolicies/regenerateSecondaryKey/action|再生成 Azure Customer Insights 共享访问签名策略辅助密钥|
|/hubs/profiles/read|读取任何 Azure Customer Insights 配置文件|
|/hubs/profiles/write|写入任何 Azure Customer Insights 配置文件|
|/hubs/kpi/read|读取任何 Azure Customer Insights 关键性能指标|
|/hubs/kpi/write|创建或更新任何 Azure Customer Insights 关键性能指标|
|/hubs/kpi/delete|删除任何 Azure Customer Insights 关键性能指标|
|/hubs/views/read|读取任何 Azure Customer Insights 应用视图|
|/hubs/views/write|创建或更新任何 Azure Customer Insights 应用视图|
|/hubs/views/delete|删除任何 Azure Customer Insights 应用视图|
|/hubs/interactions/read|读取任何 Azure Customer Insights 交互|
|/hubs/interactions/write|创建或更新任何 Azure Customer Insights 交互|
|/hubs/roleAssignments/read|读取任何 Azure Customer Insights RBAC 分配|
|/hubs/roleAssignments/write|创建或更新任何 Azure Customer Insights RBAC 分配|
|/hubs/roleAssignments/delete|删除任何 Azure Customer Insights RBAC 分配|
|/hubs/connectors/read|读取任何 Azure Customer Insights 连接器|
|/hubs/connectors/write|创建或更新任何 Azure Customer Insights 连接器|
|/hubs/connectors/delete|删除任何 Azure Customer Insights 连接器|
|/hubs/connectors/mappings/read|读取任何 Azure Customer Insights 连接器映射|
|/hubs/connectors/mappings/write|创建或更新任何 Azure Customer Insights 连接器映射|
|/hubs/connectors/mappings/delete|删除任何 Azure Customer Insights 连接器映射|

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

| 操作 | 说明 |
|---|---|
|/checkNameAvailability/action|检查租户的目录名称可用性。|
|/catalogs/read|获取订阅或资源组下一个或多个目录的属性。|
|/catalogs/write|创建目录，或更新目录的标记和属性。|
|/catalogs/delete|删除目录。|

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

| 操作 | 说明 |
|---|---|
|/datafactories/read|读取数据工厂。|
|/datafactories/write|创建或更新数据工厂|
|/datafactories/delete|删除数据工厂。|
|/datafactories/datapipelines/read|读取管道。|
|/datafactories/datapipelines/delete|删除管道。|
|/datafactories/datapipelines/pause/action|暂停管道。|
|/datafactories/datapipelines/resume/action|恢复管道。|
|/datafactories/datapipelines/update/action|更新管道。|
|/datafactories/datapipelines/write|创建或更新管道|
|/datafactories/linkedServices/read|读取链接服务。|
|/datafactories/linkedServices/delete|删除链接服务。|
|/datafactories/linkedServices/write|创建或更新链接服务|
|/datafactories/tables/read|读取表。|
|/datafactories/tables/delete|删除表。|
|/datafactories/tables/write|创建或更新表|

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

| 操作 | 说明 |
|---|---|
|/accounts/read|获取有关 DataLakeAnalytics 帐户的信息|
|/accounts/write|创建或更新 DataLakeAnalytics 帐户。|
|/accounts/delete|删除 DataLakeAnalytics 帐户。|
|/accounts/firewallRules/read|获取有关防火墙规则的信息。|
|/accounts/firewallRules/write|创建或更新防火墙规则。|
|/accounts/firewallRules/delete|删除防火墙规则。|
|/accounts/storageAccounts/read|获取 DataLakeAnalytics 帐户的链接存储帐户。|
|/accounts/storageAccounts/write|将存储帐户链接到 DataLakeAnalytics 帐户。|
|/accounts/storageAccounts/delete|从 DataLakeAnalytics 帐户取消链接存储帐户。|
|/accounts/storageAccounts/Containers/read|获取存储帐户下的容器|
|/accounts/storageAccounts/Containers/listSasTokens/action|列出存储容器的 SAS 令牌|
|/accounts/dataLakeStoreAccounts/read|获取 DataLakeAnalytics 帐户的链接 DataLakeStore 帐户。|
|/accounts/dataLakeStoreAccounts/write|将 DataLakeStore 帐户链接到 DataLakeAnalytics 帐户。|
|/accounts/dataLakeStoreAccounts/delete|从 DataLakeAnalytics 帐户取消链接 DataLakeStore 帐户。|

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

| 操作 | 说明 |
|---|---|
|/accounts/read|获取有关现有 DataLakeStore 帐户的信息。|
|/accounts/write|创建新的 DataLakeStore 帐户，或更新现有的 DataLakeStore 帐户。|
|/accounts/delete|删除现有的 DataLakeStore 帐户。|
|/accounts/firewallRules/read|获取有关防火墙规则的信息。|
|/accounts/firewallRules/write|创建或更新防火墙规则。|
|/accounts/firewallRules/delete|删除防火墙规则。|
|/accounts/trustedIdProviders/read|获取有关受信任标识提供者的信息。|
|/accounts/trustedIdProviders/write|创建或更新受信任的标识提供者。|
|/accounts/trustedIdProviders/delete|删除受信任的标识提供者。|

## <a name="microsoftdevices"></a>Microsoft.Devices

| 操作 | 说明 |
|---|---|
|/register/action|注册 IotHub 资源提供程序的订阅，并启用 IotHub 资源的创建|
|/checkNameAvailability/Action|检查 IotHub 名称是否可用|
|/usages/Read|获取此提供程序的订阅用量详细信息。|
|/operations/Read|获取所有 ResourceProvider 操作|
|/iotHubs/Read|获取 IotHub 资源|
|/iotHubs/Write|创建或更新 IotHub 资源|
|/iotHubs/Delete|删除 IotHub 资源|
|/iotHubs/listkeys/Action|获取所有 IotHub 密钥|
|/iotHubs/exportDevices/Action|导出设备|
|/iotHubs/importDevices/Action|导入设备|
|/IotHubs/metricDefinitions/read|获取 IotHub 服务的可用指标|
|/iotHubs/iotHubKeys/listkeys/Action|获取给定名称的 IotHub 密钥|
|/iotHubs/iotHubStats/Read|获取 IotHub 统计信息|
|/iotHubs/quotaMetrics/Read|获取配额指标|
|/iotHubs/eventHubEndpoints/consumerGroups/Write|创建 EventHub 使用者组|
|/iotHubs/eventHubEndpoints/consumerGroups/Read|获取 EventHub 使用者组|
|/iotHubs/eventHubEndpoints/consumerGroups/Delete|删除 EventHub 使用者组|
|/iotHubs/routing/routes/$testall/Action|针对所有现有路由测试消息|
|/iotHubs/routing/routes/$testnew/Action|针对提供的测试路由测试消息|
|/IotHubs/diagnosticSettings/read|获取资源的诊断设置|
|/IotHubs/diagnosticSettings/write|创建或更新资源的诊断设置|
|/iotHubs/skus/Read|获取有效的 IotHub SKU|
|/iotHubs/jobs/Read|获取在给定 IotHub 上提交的作业详细信息|
|/iotHubs/routingEndpointsHealth/Read|获取 IotHub 的所有路由终结点的运行状况|

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

| 操作 | 说明 |
|---|---|
|/Subscription/register/action|注册订阅|
|/labs/delete|删除实验室。|
|/labs/read|读取实验室。|
|/labs/write|添加或修改实验室。|
|/labs/ListVhds/action|列出可用于创建自定义映像的磁盘映像。|
|/labs/GenerateUploadUri/action|生成用于将自定义磁盘映像上传到实验室的 URI。|
|/labs/CreateEnvironment/action|在实验室中创建虚拟机。|
|/labs/ClaimAnyVm/action|在实验室中声明随机可声明的虚拟机。|
|/labs/ExportResourceUsage/action|将实验室资源使用情况导出到存储帐户|
|/labs/users/delete|删除用户配置文件。|
|/labs/users/read|读取用户配置文件。|
|/labs/users/write|添加或修改用户配置文件。|
|/labs/users/secrets/delete|删除机密。|
|/labs/users/secrets/read|读取机密。|
|/labs/users/secrets/write|添加或修改机密。|
|/labs/users/environments/delete|删除环境。|
|/labs/users/environments/read|读取环境。|
|/labs/users/environments/write|添加或修改环境。|
|/labs/users/disks/delete|删除磁盘。|
|/labs/users/disks/read|读取磁盘。|
|/labs/users/disks/write|添加或修改磁盘。|
|/labs/users/disks/Attach/action|在虚拟机上附加和创建磁盘的租约。|
|/labs/users/disks/Detach/action|分离并破坏已附加到虚拟机的磁盘的租约。|
|/labs/customImages/delete|删除自定义映像。|
|/labs/customImages/read|读取自定义映像。|
|/labs/customImages/write|添加或修改自定义映像。|
|/labs/serviceRunners/delete|删除服务运行程序。|
|/labs/serviceRunners/read|读取服务运行程序。|
|/labs/serviceRunners/write|添加或修改服务运行程序。|
|/labs/artifactSources/delete|删除项目源。|
|/labs/artifactSources/read|读取项目源。|
|/labs/artifactSources/write|添加或修改项目源。|
|/labs/artifactSources/artifacts/read|读取项目。|
|/labs/artifactSources/artifacts/GenerateArmTemplate/action|为给定的项目生成 ARM 模板，将所需的文件上传到存储帐户，然后验证生成的项目。|
|/labs/artifactSources/armTemplates/read|读取 Azure 资源管理器模板。|
|/labs/costs/read|读取成本。|
|/labs/costs/write|添加或修改成本。|
|/labs/virtualNetworks/delete|创建虚拟网络。|
|/labs/virtualNetworks/read|读取虚拟网络。|
|/labs/virtualNetworks/write|添加或修改虚拟网络。|
|/labs/formulas/delete|删除公式。|
|/labs/formulas/read|读取公式。|
|/labs/formulas/write|添加或修改公式。|
|/labs/schedules/delete|删除计划。|
|/labs/schedules/read|读取计划。|
|/labs/schedules/write|添加或修改计划。|
|/labs/schedules/Execute/action|执行计划。|
|/labs/schedules/ListApplicable/action|列出所有适用的计划|
|/labs/galleryImages/read|读取库映像。|
|/labs/policySets/EvaluatePolicies/action|评估实验室策略。|
|/labs/policySets/policies/delete|删除策略。|
|/labs/policySets/policies/read|读取策略。|
|/labs/policySets/policies/write|添加或修改策略。|
|/labs/virtualMachines/delete|删除虚拟机。|
|/labs/virtualMachines/read|读取虚拟机。|
|/labs/virtualMachines/write|添加或修改虚拟机。|
|/labs/virtualMachines/Start/action|启动虚拟机。|
|/labs/virtualMachines/Stop/action|停止虚拟机|
|/labs/virtualMachines/ApplyArtifacts/action|将项目应用到虚拟机。|
|/labs/virtualMachines/AddDataDisk/action|将新的或现有的数据磁盘附加到虚拟机。|
|/labs/virtualMachines/DetachDataDisk/action|从虚拟机中分离指定的磁盘。|
|/labs/virtualMachines/Claim/action|获得现有虚拟机的所有权|
|/labs/virtualMachines/ListApplicableSchedules/action|列出所有适用的计划|
|/labs/virtualMachines/schedules/delete|删除计划。|
|/labs/virtualMachines/schedules/read|读取计划。|
|/labs/virtualMachines/schedules/write|添加或修改计划。|
|/labs/virtualMachines/schedules/Execute/action|执行计划。|
|/labs/notificationChannels/delete|删除 notificationchannels。|
|/labs/notificationChannels/read|读取 notificationchannels。|
|/labs/notificationChannels/write|添加或修改 notificationchannels。|
|/labs/notificationChannels/Notify/action|将通知发送到提供的通道。|
|/schedules/delete|删除计划。|
|/schedules/read|读取计划。|
|/schedules/write|添加或修改计划。|
|/schedules/Execute/action|执行计划。|
|/schedules/Retarget/action|更新计划的目标资源 ID。|
|/locations/operations/read|读取操作。|

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

| 操作 | 说明 |
|---|---|
|/databaseAccountNames/read|检查名称可用性。|
|/databaseAccounts/read|读取数据库帐户。|
|/databaseAccounts/write|更新数据库帐户。|
|/databaseAccounts/listKeys/action|列出数据库帐户的密钥|
|/databaseAccounts/regenerateKey/action|轮换数据库帐户的密钥|
|/databaseAccounts/listConnectionStrings/action|获取数据库帐户的连接字符串|
|/databaseAccounts/changeResourceGroup/action|更改数据库帐户的资源组|
|/databaseAccounts/failoverPriorityChange/action|更改数据库帐户的区域故障转移优先级。 用于执行手动故障转移操作|
|/databaseAccounts/delete|删除数据库帐户。|
|/databaseAccounts/metricDefinitions/read|读取数据库帐户指标定义。|
|/databaseAccounts/metrics/read|读取数据库帐户指标。|
|/databaseAccounts/usages/read|读取数据库帐户使用情况。|
|/databaseAccounts/databases/collections/metricDefinitions/read|读取集合指标定义。|
|/databaseAccounts/databases/collections/metrics/read|读取集合指标。|
|/databaseAccounts/databases/collections/usages/read|读取集合使用情况。|
|/databaseAccounts/databases/metricDefinitions/read|读取数据库指标定义|
|/databaseAccounts/databases/metrics/read|读取数据库指标。|
|/databaseAccounts/databases/usages/read|读取数据库使用情况。|
|/databaseAccounts/readonlykeys/read|读取数据库帐户只读密钥。|

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

| 操作 | 说明 |
|---|---|
|/generateSsoRequest/Action|生成登录域控制中心的请求。|
|/validateDomainRegistrationInformation/Action|验证域购买对象但不提交该对象|
|/checkDomainAvailability/Action|检查是否可购买某个域|
|/listDomainRecommendations/Action|根据关键字检索列表域建议|
|/register/action|注册订阅的 Microsoft 域资源提供程序|
|/domains/Read|获取域的列表|
|/domains/Write|添加新域，或更新现有域|
|/domains/Delete|删除现有域。|
|/domains/operationresults/Read|获取域操作|

## <a name="microsoftdynamicslcs"></a>Microsoft.DynamicsLcs

| 操作 | 说明 |
|---|---|
|/lcsprojects/read|显示属于用户的 Microsoft Dynamics Lifecycle Services 项目|
|/lcsprojects/write|创建和更新属于用户的 Microsoft Dynamics Lifecycle Services 项目。 只能更新名称和说明属性。 与项目关联的订阅和位置在创建后无法更新|
|/lcsprojects/delete|删除属于用户的 Microsoft Dynamics Lifecycle Services 项目|
|/lcsprojects/clouddeployments/read|显示属于用户的 Microsoft Dynamics Lifecycle Services 项目中的 Microsoft Dynamics AX 2012 R3 评估部署|
|/lcsprojects/clouddeployments/write|在属于用户的 Microsoft Dynamics Lifecycle Services 项目中创建 Microsoft Dynamics AX 2012 R3 评估部署。 可从 Azure 管理门户管理部署|
|/lcsprojects/connectors/read|读取属于 Microsoft Dynamics Lifecycle Services 项目的连接器|
|/lcsprojects/connectors/write|创建和更新属于 Microsoft Dynamics Lifecycle Services 项目的连接器|

## <a name="microsofteventhub"></a>Microsoft.EventHub

| 操作 | 说明 |
|---|---|
|/checkNameAvailability/action|检查给定订阅下的命名空间可用性。|
|/register/action|注册 EventHub 资源提供程序的订阅，并启用 EventHub 资源的创建|
|/namespaces/write|创建命名空间资源并更新其属性。 命名空间的标记和状态是可更新的属性。|
|/namespaces/read|获取命名空间资源说明列表|
|/namespaces/Delete|删除命名空间资源|
|/namespaces/metricDefinitions/read|获取命名空间指标资源说明列表|
|/namespaces/authorizationRules/read|获取命名空间授权规则说明列表。|
|/namespaces/authorizationRules/write|创建命名空间级授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/namespaces/authorizationRules/delete|删除命名空间授权规则。 无法删除默认的命名空间授权规则。 |
|/namespaces/authorizationRules/listkeys/action|获取命名空间的连接字符串|
|/namespaces/authorizationRules/regenerateKeys/action|再生成资源的主密钥或辅助密钥|
|/namespaces/eventhubs/write|创建或更新 EventHub 属性。|
|/namespaces/eventhubs/read|获取 EventHub 资源说明列表|
|/namespaces/eventhubs/Delete|用于删除 EventHub 资源的操作|
|/namespaces/eventHubs/consumergroups/write|创建或更新 ConsumerGroup 属性。|
|/namespaces/eventHubs/consumergroups/read|获取 ConsumerGroup 资源说明列表|
|/namespaces/eventHubs/consumergroups/Delete|用于删除 ConsumerGroup 资源的操作|
|/namespaces/eventhubs/authorizationRules/read| 获取 EventHub 授权规则列表|
|/namespaces/eventhubs/authorizationRules/write|创建 EventHub 授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/namespaces/eventhubs/authorizationRules/delete|用于删除 EventHub 授权规则的操作|
|/namespaces/eventhubs/authorizationRules/listkeys/action|获取 EventHub 的连接字符串|
|/namespaces/eventhubs/authorizationRules/regenerateKeys/action|再生成资源的主密钥或辅助密钥|
|/namespaces/diagnosticSettings/read|获取命名空间诊断设置资源说明列表|
|/namespaces/diagnosticSettings/write|获取命名空间诊断设置资源说明列表|
|/namespaces/logDefinitions/read|获取命名空间日志资源说明列表|

## <a name="microsoftfeatures"></a>Microsoft.Features

| 操作 | 说明 |
|---|---|
|/providers/features/read|获取给定资源提供程序中某个订阅的功能。|
|/providers/features/register/action|在给定的资源提供程序中注册某个订阅的功能。|
|/features/read|获取订阅的功能。|

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

| 操作 | 说明 |
|---|---|
|/clusters/write|创建或更新 HDInsight 群集|
|/clusters/read|获取有关 HDInsight 群集的详细信息|
|/clusters/delete|删除 HDInsight 群集|
|/clusters/changerdpsetting/action|更改 HDInsight 群集的 RDP 设置|
|/clusters/configurations/action|更新 HDInsight 群集配置|
|/clusters/configurations/read|获取 HDInsight 群集配置|
|/clusters/roles/resize/action|调整 HDInsight 群集的大小|
|/locations/capabilities/read|获取订阅功能|
|/locations/checkNameAvailability/read|检查名称可用性|

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

| 操作 | 说明 |
|---|---|
|/register/action|注册导入/导出资源提供程序的订阅，并启用导入/导出作业的创建。|
|/jobs/write|使用指定的参数创建作业，或更新指定作业的属性或标记。|
|/jobs/read|获取指定作业的属性，或返回作业列表。|
|/jobs/listBitLockerKeys/action|获取指定作业的 BitLocker 密钥。|
|/jobs/delete|删除现有的作业。|
|/locations/read|获取指定位置的属性，或返回位置列表。|

## <a name="microsoftinsights"></a>Microsoft.Insights

| 操作 | 说明 |
|---|---|
|/Register/Action|注册 Microsoft.Insights 提供程序|
|/AlertRules/Write|写入警报规则配置|
|/AlertRules/Delete|删除警报规则配置|
|/AlertRules/Read|读取警报规则配置|
|/AlertRules/Activated/Action|已激活警报规则|
|/AlertRules/Resolved/Action|已解决警报规则|
|/AlertRules/Throttled/Action|已限制警报规则|
|/AlertRules/Incidents/Read|读取警报规则事件配置|
|/MetricDefinitions/Read|读取指标定义|
|/eventtypes/values/Read|读取管理事件类型值|
|/eventtypes/digestevents/Read|读取管理事件类型摘要|
|/Metrics/Read|添加指标|
|/LogProfiles/Write|写入日志配置文件配置|
|/LogProfiles/Delete|删除日志配置文件配置|
|/LogProfiles/Read|读取日志配置文件|
|/AutoscaleSettings/Write|写入自动缩放设置配置|
|/AutoscaleSettings/Delete|删除自动缩放设置配置|
|/AutoscaleSettings/Read|读取自动缩放设置配置|
|/AutoscaleSettings/Scaleup/Action|自动缩放增加操作|
|/AutoscaleSettings/Scaledown/Action|自动缩放减少操作|
|/AutoscaleSettings/providers/Microsoft.Insights/MetricDefinitions/Read|读取指标定义|
|/ActivityLogAlerts/Activated/Action|已触发活动日志警报|
|/DiagnosticSettings/Write|写入诊断设置配置|
|/DiagnosticSettings/Delete|删除诊断设置配置|
|/DiagnosticSettings/Read|读取诊断设置配置|
|/LogDefinitions/Read|读取日志定义|
|/ExtendedDiagnosticSettings/Write|写入扩展的诊断设置配置|
|/ExtendedDiagnosticSettings/Delete|删除扩展的诊断设置配置|
|/ExtendedDiagnosticSettings/Read|读取扩展的诊断设置配置|

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

| 操作 | 说明 |
|---|---|
|/register/action|注册订阅|
|/checkNameAvailability/read|检查密钥保管库名称是否有效且未被使用|
|/vaults/read|查看密钥保管库的属性|
|/vaults/write|创建新的密钥保管库，或更新现有密钥保管库的属性|
|/vaults/delete|删除密钥保管库|
|/vaults/deploy/action|部署 Azure 资源时启用对密钥保管库中机密的访问|
|/vaults/secrets/read|查看机密的属性，但不查看其值|
|/vaults/secrets/write|创建新的机密，或更新现有机密的值|
|/vaults/accessPolicies/write|通过合并或替换操作更新现有的访问策略，或者将新的访问策略添加到保管库。|
|/deletedVaults/read|查看软删除的密钥保管库的属性|
|/locations/operationResults/read|检查长时间运行的操作的结果|
|/locations/deletedVaults/read|查看软删除的密钥保管库的属性|
|/locations/deletedVaults/purge/action|清除软删除的密钥保管库|

## <a name="microsoftlogic"></a>Microsoft.Logic

| 操作 | 说明 |
|---|---|
|/workflows/read|读取工作流。|
|/workflows/write|创建或更新工作流。|
|/workflows/delete|删除工作流。|
|/workflows/run/action|启动工作流的运行。|
|/workflows/disable/action|禁用工作流。|
|/workflows/enable/action|启用工作流。|
|/workflows/validate/action|验证工作流。|
|/workflows/move/action|将工作流从其现有订阅 ID、资源组和/或名称移到其他订阅 ID、资源组和/或名称。|
|/workflows/listSwagger/action|获取工作流 Swagger 定义。|
|/workflows/regenerateAccessKey/action|再生成访问密钥机密。|
|/workflows/listCallbackUrl/action|获取工作流的回调 URL。|
|/workflows/versions/read|读取工作流版本。|
|/workflows/versions/triggers/listCallbackUrl/action|获取触发器的回调 URL。|
|/workflows/runs/read|读取工作流运行。|
|/workflows/runs/cancel/action|取消工作流的运行。|
|/workflows/runs/actions/read|读取工作流运行操作。|
|/workflows/runs/operations/read|读取工作流运行操作状态。|
|/workflows/triggers/read|读取触发器。|
|/workflows/triggers/run/action|执行触发器。|
|/workflows/triggers/listCallbackUrl/action|获取触发器的回调 URL。|
|/workflows/triggers/histories/read|读取触发器历史记录。|
|/workflows/triggers/histories/resubmit/action|重新提交工作流触发器。|
|/workflows/accessKeys/read|读取访问密钥。|
|/workflows/accessKeys/write|创建或更新访问密钥。|
|/workflows/accessKeys/delete|删除访问密钥。|
|/workflows/accessKeys/list/action|列出访问密钥机密。|
|/workflows/accessKeys/regenerate/action|再生成访问密钥机密。|
|/locations/workflows/validate/action|验证工作流。|

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

| 操作 | 说明 |
|---|---|
|/register/action|注册机器学习 Web 服务资源提供程序的订阅，并启用 Web 服务的创建。|
|/webServices/action|为受支持的区域创建区域 Web 服务属性|
|/commitmentPlans/read|读取任何机器学习承诺计划|
|/commitmentPlans/write|创建或更新任何机器学习承诺计划|
|/commitmentPlans/delete|删除任何机器学习承诺计划|
|/commitmentPlans/join/action|加入任何机器学习承诺计划|
|/commitmentPlans/commitmentAssociations/read|读取任何机器学习承诺计划关联|
|/commitmentPlans/commitmentAssociations/move/action|移动任何机器学习承诺计划关联|
|/Workspaces/read|读取任何机器学习工作区|
|/Workspaces/write|创建或更新任何机器学习工作区|
|/Workspaces/delete|删除任何机器学习工作区|
|/Workspaces/listworkspacekeys/action|列出机器学习工作区的密钥|
|/Workspaces/resyncstoragekeys/action|重新同步针对机器学习工作区配置的存储帐户密钥|
|/webServices/read|读取任何机器学习 Web 服务|
|/webServices/write|创建或更新任何机器学习 Web 服务|
|/webServices/delete|删除任何机器学习 Web 服务|

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

| 操作 | 说明 |
|---|---|
|/agreements/offers/plans/read|返回给定应用商店项的协议|
|/agreements/offers/plans/sign/action|为给定应用商店项的协议签名|
|/agreements/offers/plans/cancel/action|取消给定应用商店项的协议|

## <a name="microsoftmedia"></a>Microsoft.Media

| 操作 | 说明 |
|---|---|
|/mediaservices/read||
|/mediaservices/write||
|/mediaservices/delete||
|/mediaservices/regenerateKey/action||
|/mediaservices/listKeys/action||
|/mediaservices/syncStorageKeys/action||

## <a name="microsoftnetwork"></a>Microsoft.Network

| 操作 | 说明 |
|---|---|
|/register/action|注册订阅|
|/unregister/action|取消注册订阅|
|/checkTrafficManagerNameAvailability/action|检查流量管理器相对 DNS 名称的可用性。|
|/dnszones/read|获取 JSON 格式的 DNS 区域。 区域属性包括 tags、etag、numberOfRecordSets 和 maxNumberOfRecordSets。 请注意，此命令不会检索区域中包含的记录集。|
|/dnszones/write|在资源组中创建或更新 DNS 区域。  用于更新 DNS 区域资源上的标记。 请注意，无法使用此命令在区域中创建或更新记录集。|
|/dnszones/delete|删除 JSON 格式的 DNS 区域。 区域属性包括 tags、etag、numberOfRecordSets 和 maxNumberOfRecordSets。|
|/dnszones/MX/read|获取 JSON 格式的、“MX”类型的记录集。 记录集包含记录列表以及 TTL、标记和 etag。|
|/dnszones/MX/write|在 DNS 区域中创建或更新“MX”类型的记录集。 指定的记录将替换记录集中的当前记录。|
|/dnszones/MX/delete|从 DNS 区域中删除具有给定名称的、类型为“MX”的记录集。|
|/dnszones/NS/read|获取 NS 类型的 DNS 记录集|
|/dnszones/NS/write|创建或更新 NS 类型的 DNS 记录集|
|/dnszones/NS/delete|删除 NS 类型的 DNS 记录集|
|/dnszones/AAAA/read|获取 JSON 格式的、“AAAA”类型的记录集。 记录集包含记录列表以及 TTL、标记和 etag。|
|/dnszones/AAAA/write|在 DNS 区域中创建或更新“AAAA”类型的记录集。 指定的记录将替换记录集中的当前记录。|
|/dnszones/AAAA/delete|从 DNS 区域中删除具有给定名称的、类型为“AAAA”的记录集。|
|/dnszones/CNAME/read|获取 JSON 格式的、“CNAME”类型的记录集。 记录集包含 TTL、标记和 etag。|
|/dnszones/CNAME/write|在 DNS 区域中创建或更新“CNAME”类型的记录集。 指定的记录将替换记录集中的当前记录。|
|/dnszones/CNAME/delete|从 DNS 区域中删除具有给定名称的、类型为“CNAME”的记录集。|
|/dnszones/SOA/read|获取 SOA 类型的 DNS 记录集|
|/dnszones/SOA/write|创建或更新 SOA 类型的 DNS 记录集|
|/dnszones/SRV/read|获取 JSON 格式的、“SRV”类型的记录集。 记录集包含记录列表以及 TTL、标记和 etag。|
|/dnszones/SRV/write|创建或更新 SRV 类型的记录集|
|/dnszones/SRV/delete|从 DNS 区域中删除具有给定名称的、类型为“SRV”的记录集。|
|/dnszones/PTR/read|获取 JSON 格式的、“PTR”类型的记录集。 记录集包含记录列表以及 TTL、标记和 etag。|
|/dnszones/PTR/write|在 DNS 区域中创建或更新“PTR”类型的记录集。 指定的记录将替换记录集中的当前记录。|
|/dnszones/PTR/delete|从 DNS 区域中删除具有给定名称的、类型为“PTR”的记录集。|
|/dnszones/A/read|获取 JSON 格式的、“A”类型的记录集。 记录集包含记录列表以及 TTL、标记和 etag。|
|/dnszones/A/write|在 DNS 区域中创建或更新“A”类型的记录集。 指定的记录将替换记录集中的当前记录。|
|/dnszones/A/delete|从 DNS 区域中删除具有给定名称的、类型为“A”的记录集。|
|/dnszones/TXT/read|获取 JSON 格式的、“TXT”类型的记录集。 记录集包含记录列表以及 TTL、标记和 etag。|
|/dnszones/TXT/write|在 DNS 区域中创建或更新“TXT”类型的记录集。 指定的记录将替换记录集中的当前记录。|
|/dnszones/TXT/delete|从 DNS 区域中删除具有给定名称的、类型为“TXT”的记录集。|
|/dnszones/recordsets/read|获取各种类型的 DNS 记录集|
|/networkInterfaces/read|获取网络接口定义。 |
|/networkInterfaces/write|创建网络接口，或更新现有的网络接口。 |
|/networkInterfaces/join/action|将虚拟机加入到网络接口|
|/networkInterfaces/delete|删除网络接口|
|/networkInterfaces/effectiveRouteTable/action|获取 VM 的网络接口上配置的路由表|
|/networkInterfaces/effectiveNetworkSecurityGroups/action|获取 VM 的网络接口上配置的网络安全组|
|/networkInterfaces/loadBalancers/read|获取网络接口所属的所有负载均衡器|
|/networkInterfaces/ipconfigurations/read|获取网络接口 IP 配置定义。 |
|/publicIPAddresses/read|获取公共 IP 地址定义。|
|/publicIPAddresses/write|创建公共 IP 地址，或更新现有的公共 IP 地址。 |
|/publicIPAddresses/delete|删除公共 IP 地址。|
|/publicIPAddresses/join/action|联接公共 IP 地址|
|/routeFilters/read|获取路由筛选器定义|
|/routeFilters/join/action|联接路由筛选器|
|/routeFilters/delete|删除路由筛选器定义|
|/routeFilters/write|创建路由筛选器，或更新现有的路由筛选器|
|/routeFilters/rules/read|获取路由筛选器规则定义|
|/routeFilters/rules/write|创建路由筛选器规则，或更新现有的路由筛选器规则|
|/routeFilters/rules/delete|删除路由筛选器规则定义|
|/networkWatchers/read|获取网络观察程序定义|
|/networkWatchers/write|创建网络观察程序，或更新现有的网络观察程序|
|/networkWatchers/delete|删除网络观察程序|
|/networkWatchers/configureFlowLog/action|配置目标资源的流日志记录。|
|/networkWatchers/ipFlowVerify/action|返回是允许还是拒绝与特定的目标相互发送数据包。|
|/networkWatchers/nextHop/action|对于指定的目标和目标 IP 地址，返回下一跃点类型和下一跃点 IP 地址。|
|/networkWatchers/queryFlowLogStatus/action|获取资源上流日志记录的状态。|
|/networkWatchers/queryTroubleshootResult/action|获取上次运行的或当前正在运行的故障排除操作的故障排除结果。|
|/networkWatchers/securityGroupView/action|查看 VM 上应用的已配置和有效的网络安全组规则。|
|/networkWatchers/topology/action|获取资源组中的资源及其关系的网络级视图。|
|/networkWatchers/troubleshoot/action|开始针对 Azure 中的网络资源进行故障排除。|
|/networkWatchers/packetCaptures/queryStatus/action|获取有关数据包捕获资源的属性和状态的信息。|
|/networkWatchers/packetCaptures/stop/action|停止正在运行的数据包捕获会话。|
|/networkWatchers/packetCaptures/read|获取数据包捕获定义|
|/networkWatchers/packetCaptures/write|创建数据包捕获|
|/networkWatchers/packetCaptures/delete|删除数据包捕获|
|/loadBalancers/read|获取负载均衡器定义|
|/loadBalancers/write|创建负载均衡器，或更新现有的负载均衡器|
|/loadBalancers/delete|删除负载均衡器|
|/loadBalancers/networkInterfaces/read|获取对负载均衡器下的所有网络接口的引用|
|/loadBalancers/loadBalancingRules/read|获取负载均衡器的负载均衡规则定义|
|/loadBalancers/backendAddressPools/read|获取负载均衡器的后端地址池定义|
|/loadBalancers/backendAddressPools/join/action|加入负载均衡器后端地址池|
|/loadBalancers/inboundNatPools/read|获取负载均衡器的入站 NAT 池定义|
|/loadBalancers/inboundNatPools/join/action|加入负载均衡器入站 NAT 池|
|/loadBalancers/inboundNatRules/read|获取负载均衡器的入站 NAT 规则定义|
|/loadBalancers/inboundNatRules/write|创建负载均衡器入站 NAT 规则，或更新现有的负载均衡器入站 NAT 规则|
|/loadBalancers/inboundNatRules/delete|删除负载均衡器入站 NAT 规则|
|/loadBalancers/inboundNatRules/join/action|加入负载均衡器入站 NAT 规则|
|/loadBalancers/outboundNatRules/read|获取负载均衡器的出站 NAT 规则定义|
|/loadBalancers/probes/read|获取负载均衡器探测|
|/loadBalancers/virtualMachines/read|获取对负载均衡器下的所有虚拟机的引用|
|/loadBalancers/frontendIPConfigurations/read|获取负载均衡器的前端 IP 配置定义|
|/trafficManagerGeographicHierarchies/read|获取流量管理器地理层次结构，其中包含可以配合地理流量路由方法使用的区域|
|/bgpServiceCommunities/read|获取 BGP 服务社区|
|/applicationGatewayAvailableWafRuleSets/read|获取应用程序网关可用的 WAF 规则集|
|/virtualNetworks/read|获取虚拟网络定义|
|/virtualNetworks/write|创建虚拟网络，或更新现有的虚拟网络|
|/virtualNetworks/delete|删除虚拟网络|
|/virtualNetworks/peer/action|在两个不同的虚拟网络之间建立对等互连|
|/virtualNetworks/virtualNetworkPeerings/read|获取虚拟网络对等互连定义|
|/virtualNetworks/virtualNetworkPeerings/write|创建虚拟网络对等互连，或更新现有的虚拟网络对等互连|
|/virtualNetworks/virtualNetworkPeerings/delete|删除虚拟网络对等互连|
|/virtualNetworks/subnets/read|获取虚拟网络子网定义|
|/virtualNetworks/subnets/write|创建虚拟网络子网，或更新现有的虚拟网络子网|
|/virtualNetworks/subnets/delete|删除虚拟网络子网|
|/virtualNetworks/subnets/join/action|加入虚拟网络|
|/virtualNetworks/subnets/joinViaServiceTunnel/action|将存储帐户或 SQL 数据库等资源加入到已启用服务隧道的子网。|
|/virtualNetworks/subnets/virtualMachines/read|获取对虚拟网络子网中的所有虚拟机的引用|
|/virtualNetworks/checkIpAddressAvailability/read|检查 IP 地址是否在指定的虚拟网络中可用|
|/virtualNetworks/virtualMachines/read|获取对虚拟网络中的所有虚拟机的引用|
|/expressRouteServiceProviders/read|获取 Express Route 服务提供商|
|/dnsoperationresults/read|获取 DNS 操作的结果|
|/localnetworkgateways/read|获取 LocalNetworkGateway|
|/localnetworkgateways/write|创建新的或更新现有的 LocalNetworkGateway|
|/localnetworkgateways/delete|删除 LocalNetworkGateway|
|/trafficManagerProfiles/read|获取流量管理器配置文件配置。 此配置包含 DNS 设置、流量路由设置、终结点监视设置，以及此流量管理器配置文件路由的终结点列表。|
|/trafficManagerProfiles/write|创建流量管理器配置文件，或修改现有流量管理器配置文件的配置。 这包括启用或禁用配置文件，以及修改 DNS 设置、流量路由设置或终结点监视设置。 可以添加、删除、启用或禁用流量管理器配置文件路由的终结点。|
|/trafficManagerProfiles/delete|删除流量管理器配置文件。 与流量管理器配置文件关联的所有设置都将丢失，该配置文件不再可用于路由流量。|
|/dnsoperationstatuses/read|获取 DNS 操作的状态 |
|/operations/read|获取可用操作|
|/expressRouteCircuits/read|获取 ExpressRouteCircuit|
|/expressRouteCircuits/write|创建新的或更新现有的 ExpressRouteCircuit|
|/expressRouteCircuits/delete|删除 ExpressRouteCircuit|
|/expressRouteCircuits/stats/read|获取 ExpressRouteCircuit 统计信息|
|/expressRouteCircuits/peerings/read|获取 ExpressRouteCircuit 对等互连|
|/expressRouteCircuits/peerings/write|创建新的或更新现有的 ExpressRouteCircuit 对等互连|
|/expressRouteCircuits/peerings/delete|删除 ExpressRouteCircuit 对等互连|
|/expressRouteCircuits/peerings/arpTables/action|获取 ExpressRouteCircuit 对等互连 ArpTable|
|/expressRouteCircuits/peerings/routeTables/action|获取 ExpressRouteCircuit 对等互连 RouteTable|
|/expressRouteCircuits/peerings/<br>routeTablesSummary/action|获取 ExpressRouteCircuit 对等互连 RouteTable 摘要|
|/expressRouteCircuits/peerings/stats/read|获取 ExpressRouteCircuit 对等互连统计信息|
|/expressRouteCircuits/authorizations/read|获取 ExpressRouteCircuit 授权|
|/expressRouteCircuits/authorizations/write|创建新的或更新现有的 ExpressRouteCircuit 授权|
|/expressRouteCircuits/authorizations/delete|删除 ExpressRouteCircuit 授权|
|/connections/read|获取 VirtualNetworkGatewayConnection|
|/connections/write|创建新的或更新现有的 VirtualNetworkGatewayConnection|
|/connections/delete|删除 VirtualNetworkGatewayConnection|
|/connections/sharedKey/read|获取 VirtualNetworkGatewayConnection SharedKey|
|/connections/sharedKey/write|创建新的或更新现有的 VirtualNetworkGatewayConnection SharedKey|
|/networkSecurityGroups/read|获取网络安全组定义|
|/networkSecurityGroups/write|创建网络安全组，或更新现有的网络安全组|
|/networkSecurityGroups/delete|删除网络安全组|
|/networkSecurityGroups/join/action|加入网络安全组|
|/networkSecurityGroups/defaultSecurityRules/read|获取默认的安全规则定义|
|/networkSecurityGroups/securityRules/read|获取安全规则定义|
|/networkSecurityGroups/securityRules/write|创建安全规则，或更新现有的安全规则|
|/networkSecurityGroups/securityRules/delete|删除安全规则|
|/applicationGateways/read|获取应用程序网关|
|/applicationGateways/write|创建应用程序网关，或更新应用程序网关|
|/applicationGateways/delete|删除应用程序网关|
|/applicationGateways/backendhealth/action|获取应用程序网关后端运行状况|
|/applicationGateways/start/action|启动应用程序网关|
|/applicationGateways/stop/action|停止应用程序网关|
|/applicationGateways/backendAddressPools/join/action|加入应用程序网关后端地址池|
|/routeTables/read|获取路由表定义|
|/routeTables/write|创建路由表，或更新现有的路由表|
|/routeTables/delete|删除路由表定义|
|/routeTables/join/action|加入路由表|
|/routeTables/routes/read|获取路由定义|
|/routeTables/routes/write|创建路由，或更新现有的路由|
|/routeTables/routes/delete|删除路由定义|
|/locations/operationResults/read|获取异步 POST 或 DELETE 操作的操作结果|
|/locations/checkDnsNameAvailability/read|检查 DNS 标签是否可在指定的位置使用|
|/locations/usages/read|获取资源用量指标|
|/locations/operations/read|获取表示异步操作状态的操作资源|

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

| 操作 | 说明 |
|---|---|
|/register/action|注册 NotifciationHubs 资源提供程序的订阅，并启用命名空间和 NotificationHubs 的创建|
|/CheckNamespaceAvailability/action|检查给定的命名空间资源名称是否在 NotificationHub 服务中可用。|
|/Namespaces/write|创建命名空间资源并更新其属性。 命名空间的标记和状态是可更新的属性。|
|/Namespaces/read|获取命名空间资源说明列表|
|/Namespaces/Delete|删除命名空间资源|
|/Namespaces/authorizationRules/action|获取命名空间授权规则说明列表。|
|/Namespaces/CheckNotificationHubAvailability/action|检查给定的 NotificationHub 名称是否在命名空间中可用。|
|/Namespaces/authorizationRules/write|创建命名空间级授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/Namespaces/authorizationRules/read|获取命名空间授权规则说明列表。|
|/Namespaces/authorizationRules/delete|删除命名空间授权规则。 无法删除默认的命名空间授权规则。 |
|/Namespaces/authorizationRules/listkeys/action|获取命名空间的连接字符串|
|/Namespaces/authorizationRules/regenerateKeys/action|命名空间授权规则再生成主密钥/辅助密钥，指定需要再生成的密钥|
|/Namespaces/NotificationHubs/write|创建通知中心并更新其属性。 其属性主要包括 PNS 凭据。 授权规则和 TTL|
|/Namespaces/NotificationHubs/read|获取通知中心资源说明列表|
|/Namespaces/NotificationHubs/Delete|删除通知中心资源|
|/Namespaces/NotificationHubs/authorizationRules/action|获取通知中心授权规则列表|
|/Namespaces/NotificationHubs/pnsCredentials/action|获取所有通知中心的 PNS 凭据。 这包括 WNS、MPNS、APNS、GCM 和百度凭据|
|/Namespaces/NotificationHubs/debugSend/action|发送测试推送通知。|
|/Namespaces/NotificationHubs/metricDefinitions/read|获取命名空间指标资源说明列表|
|/Namespaces/NotificationHubs/<br>authorizationRules/write|创建通知中心授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/Namespaces/NotificationHubs/<br>authorizationRules/read|获取通知中心授权规则列表|
|/Namespaces/NotificationHubs/<br>authorizationRules/delete|删除通知中心授权规则|
|/Namespaces/NotificationHubs/<br>authorizationRules/listkeys/action|获取通知中心的连接字符串|
|/Namespaces/NotificationHubs/<br>authorizationRules/regenerateKeys/action|通知中心授权规则再生成主密钥/辅助密钥，指定需要再生成的密钥|

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

| 操作 | 说明 |
|---|---|
|/register/action|将订阅注册到资源提供程序。|
|/linkTargets/read|列出不与 Azure 订阅关联的现有帐户。 若要将此 Azure 订阅链接到工作区，请使用此操作在“创建工作区”操作的客户 ID 属性中返回的客户 ID。|
|/workspaces/write|创建新的工作区，或者通过提供现有工作区中的客户 ID 链接到现有工作区。|
|/workspaces/read|获取现有工作区|
|/workspaces/delete|删除工作区。 如果该工作区在创建时已链接到现有工作区，则不会删除它链接到的工作区。|
|/workspaces/generateregistrationcertificate/action|为工作区生成注册证书。 此证书用于将 Microsoft System Center Operation Manager 连接到工作区。|
|/workspaces/sharedKeys/action|检索工作区的共享密钥。 这些密钥用于将 Microsoft Operational Insights 代理连接到工作区。|
|/workspaces/search/action|执行搜索查询|
|/workspaces/datasources/read|获取工作区下面的数据源。|
|/workspaces/datasources/write|在工作区下面创建/更新数据源。|
|/workspaces/datasources/delete|删除工作区下面的数据源。|
|/workspaces/managementGroups/read|获取已连接到此工作区的 System Center Operations Manager 管理组的名称和元数据。|
|/workspaces/schema/read|获取工作区的搜索架构。  搜索架构包含公开的字段及其类型。|
|/workspaces/usages/read|获取工作区的使用情况数据，包括工作区读取的数据量。|
|/workspaces/intelligencepacks/read|列出给定的工作区可见的所有智能包，并列出是为该工作区启用还是禁用了智能包。|
|/workspaces/intelligencepacks/enable/action|为给定的工作区启用智能包。|
|/workspaces/intelligencepacks/disable/action|为给定的工作区禁用智能包。|
|/workspaces/sharedKeys/read|检索工作区的共享密钥。 这些密钥用于将 Microsoft Operational Insights 代理连接到工作区。|
|/workspaces/savedSearches/read|获取保存的搜索查询|
|/workspaces/savedSearches/write|创建保存的搜索查询|
|/workspaces/savedSearches/delete|删除保存的搜索查询|
|/workspaces/storageinsightconfigs/write|创建新的存储配置。 这些配置用于从现有存储帐户中的某个位置提取数据。|
|/workspaces/storageinsightconfigs/read|获取存储配置。|
|/workspaces/storageinsightconfigs/delete|删除存储配置。 这会阻止 Microsoft Operational Insights 从存储帐户中读取数据。|
|/workspaces/configurationScopes/read|获取配置范围|
|/workspaces/configurationScopes/write|设置配置范围|
|/workspaces/configurationScopes/delete|删除配置范围|

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

| 操作 | 说明 |
|---|---|
|/register/action|将订阅注册到资源提供程序。|
|/solutions/write|创建新的 OMS 解决方案|
|/solutions/read|获取现有的 OMS 解决方案|
|/solutions/delete|删除现有的 OMS 解决方案|

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

| 操作 | 说明 |
|---|---|
|/Vaults/backupJobsExport/action|导出作业|
|/Vaults/write|“创建保管库”操作创建“vault”类型的 Azure 资源|
|/Vaults/read|“获取保管库”操作获取表示“vault”类型的 Azure 资源的对象|
|/Vaults/delete|“删除保管库”操作删除“vault”类型的指定 Azure 资源|
|/Vaults/refreshContainers/read|刷新容器列表|
|/Vaults/backupJobsExport/operationResults/read|返回导出作业操作的结果。|
|/Vaults/backupOperationResults/read|返回恢复服务保管库的备份操作结果。|
|/Vaults/monitoringAlerts/read|获取恢复服务保管库的警报。|
|/Vaults/monitoringAlerts/{uniqueAlertId}/read|获取警报的详细信息。|
|/Vaults/backupSecurityPIN/read|返回恢复服务保管库的安全 PIN 信息。|
|/vaults/replicationEvents/read|读取任何事件|
|/Vaults/backupProtectableItems/read|返回所有可保护项的列表。|
|/vaults/replicationFabrics/read|读取任何结构|
|/vaults/replicationFabrics/write|创建或更新任何结构|
|/vaults/replicationFabrics/remove/action|删除结构|
|/vaults/replicationFabrics/checkConsistency/action|检查结构的一致性|
|/vaults/replicationFabrics/delete|删除任何结构|
|/vaults/replicationFabrics/renewcertificate/action||
|/vaults/replicationFabrics/deployProcessServerImage/action|部署进程服务器映像|
|/vaults/replicationFabrics/reassociateGateway/action|重新关联网关|
|/vaults/replicationFabrics/replicationRecoveryServicesProviders/<br>read|读取任何恢复服务提供程序|
|/vaults/replicationFabrics/replicationRecoveryServicesProviders/<br>remove/action|删除恢复服务提供程序|
|/vaults/replicationFabrics/replicationRecoveryServicesProviders/<br>删除|删除任何恢复服务提供程序|
|/vaults/replicationFabrics/replicationRecoveryServicesProviders/<br>refreshProvider/action|刷新提供程序|
|/vaults/replicationFabrics/replicationStorageClassifications/read|读取任何存储分类|
|/vaults/replicationFabrics/replicationStorageClassifications/<br>replicationStorageClassificationMappings/read|读取任何存储分类映射|
|/vaults/replicationFabrics/replicationStorageClassifications/<br>replicationStorageClassificationMappings/write|创建或更新任何存储分类映射|
|/vaults/replicationFabrics/replicationStorageClassifications/<br>replicationStorageClassificationMappings/delete|删除任何存储分类映射|
|/vaults/replicationFabrics/replicationvCenters/read|读取任何作业|
|/vaults/replicationFabrics/replicationvCenters/write|创建或更新任何作业|
|/vaults/replicationFabrics/replicationvCenters/delete|删除任何作业|
|/vaults/replicationFabrics/replicationNetworks/read|读取任何网络|
|/vaults/replicationFabrics/replicationNetworks/<br>replicationNetworkMappings/read|读取任何网络映射|
|/vaults/replicationFabrics/replicationNetworks/<br>replicationNetworkMappings/write|创建或更新任何网络映射|
|/vaults/replicationFabrics/replicationNetworks/<br>replicationNetworkMappings/delete|删除任何网络映射|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>read|读取任何保护容器|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>discoverProtectableItem/action|发现可保护项|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>write|创建或更新任何保护容器|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>remove/action|删除保护容器|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>switchprotection/action|交换保护容器|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectableItems/read|读取任何可保护项|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectionContainerMappings/read|读取任何保护容器映射|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectionContainerMappings/write|创建或更新任何保护容器映射|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectionContainerMappings/remove/action|删除保护容器映射|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectionContainerMappings/delete|删除任何保护容器映射|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/read|读取任何受保护的项|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/write|创建或更新任何受保护的项|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/delete|删除任何受保护的项|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/remove/action|删除受保护的项|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/plannedFailover/action|计划内故障转移|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/unplannedFailover/action|故障转移|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/testFailover/action|测试故障转移|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/testFailoverCleanup/action|测试故障转移清理|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/failoverCommit/action|故障转移提交|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/reProtect/action|重新保护受保护的项|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/updateMobilityService/action|更新移动服务|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/repairReplication/action|修复复制|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/applyRecoveryPoint/action|应用还原点|
|/vaults/replicationFabrics/replicationProtectionContainers/<br>replicationProtectedItems/recoveryPoints/read|读取任何复制恢复点|
|/vaults/replicationPolicies/read|读取任何策略|
|/vaults/replicationPolicies/write|创建或更新任何策略|
|/vaults/replicationPolicies/delete|删除任何策略|
|/vaults/replicationRecoveryPlans/read|读取任何恢复计划|
|/vaults/replicationRecoveryPlans/write|创建或更新任何恢复计划|
|/vaults/replicationRecoveryPlans/delete|删除任何恢复计划|
|/vaults/replicationRecoveryPlans/plannedFailover/action|计划内故障转移恢复计划|
|/vaults/replicationRecoveryPlans/unplannedFailover/action|故障转移恢复计划|
|/vaults/replicationRecoveryPlans/testFailover/action|测试故障转移恢复计划|
|/vaults/replicationRecoveryPlans/testFailoverCleanup/action|测试故障转移清理恢复计划|
|/vaults/replicationRecoveryPlans/failoverCommit/action|故障转移提交恢复计划|
|/vaults/replicationRecoveryPlans/reProtect/action|重新保护恢复计划|
|/Vaults/extendedInformation/read|“获取扩展信息”操作获取表示“vault”类型的 Azure 资源的对象扩展信息|
|/Vaults/extendedInformation/write|“获取扩展信息”操作获取表示“vault”类型的 Azure 资源的对象扩展信息|
|/Vaults/extendedInformation/delete|“获取扩展信息”操作获取表示“vault”类型的 Azure 资源的对象扩展信息|
|/Vaults/backupManagementMetaData/read|返回恢复服务保管库的备份管理元数据。|
|/Vaults/backupProtectionContainers/read|返回属于订阅的所有容器|
|/Vaults/backupFabrics/operationResults/read|返回操作状态|
|/Vaults/backupFabrics/protectionContainers/read|返回所有已注册的容器|
|/Vaults/backupFabrics/protectionContainers/<br>operationResults/read|获取对保护容器执行的操作的结果。|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/read|返回受保护项的对象详细信息|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/write|创建备份受保护项|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/delete|删除受保护的项|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/backup/action|对受保护的项执行备份。|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/operationResults/read|获取对受保护项执行的操作的结果。|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/operationStatus/read|返回对受保护项执行的操作的状态。|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/recoveryPoints/read|获取受保护项的恢复点。|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/recoveryPoints/<br>restore/action|还原受保护项的恢复点。|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/recoveryPoints/<br>provisionInstantItemRecovery/action|预配受保护项的即时项恢复|
|/Vaults/backupFabrics/protectionContainers/<br>protectedItems/recoveryPoints/<br>revokeInstantItemRecovery/action|吊销受保护项的即时项恢复|
|/Vaults/usages/read|返回恢复服务保管库的使用情况详细信息。|
|/vaults/usages/read|读取任何保管库使用情况|
|/Vaults/certificates/write|“更新资源证书”操作更新资源/保管库凭据证书。|
|/Vaults/tokenInfo/read|返回恢复服务保管库的令牌信息。|
|/vaults/replicationAlertSettings/read|读取任何警报设置|
|/vaults/replicationAlertSettings/write|创建或更新任何警报设置|
|/Vaults/backupOperations/read|返回恢复服务保管库的备份操作状态。|
|/Vaults/storageConfig/read|返回恢复服务保管库的存储配置。|
|/Vaults/storageConfig/write|更新恢复服务保管库的存储配置。|
|/Vaults/backupUsageSummaries/read|返回恢复服务的受保护项和受保护服务器的摘要。|
|/Vaults/backupProtectedItems/read|返回所有受保护项的列表。|
|/Vaults/backupconfig/vaultconfig/read|返回恢复服务保管库的配置。|
|/Vaults/backupconfig/vaultconfig/write|更新恢复服务保管库的配置。|
|/Vaults/registeredIdentities/write|“注册服务容器”操作可用于向恢复服务注册容器。|
|/Vaults/registeredIdentities/read|“获取容器”操作可用于获取针对资源注册的容器。|
|/Vaults/registeredIdentities/delete|“取消注册容器”操作可用于取消注册容器。|
|/Vaults/registeredIdentities/operationResults/read|“获取操作结果”操作可用于获取异步提交的操作的操作状态和结果|
|/vaults/replicationJobs/read|读取任何作业|
|/vaults/replicationJobs/cancel/action|取消作业|
|/vaults/replicationJobs/restart/action|重新启动作业|
|/vaults/replicationJobs/resume/action|恢复作业|
|/Vaults/backupPolicies/read|返回所有保护策略|
|/Vaults/backupPolicies/write|创建保护策略|
|/Vaults/backupPolicies/delete|删除保护策略|
|/Vaults/backupPolicies/operationResults/read|获取策略操作的结果。|
|/Vaults/backupPolicies/operationStatus/read|获取策略操作的状态。|
|/Vaults/vaultTokens/read|“保管库令牌”操作可用于获取保管库级后端操作的保管库令牌。|
|/Vaults/monitoringConfigurations/notificationConfiguration/read|获取恢复服务保管库通知配置。|
|/Vaults/backupJobs/read|返回所有作业对象|
|/Vaults/backupJobs/cancel/action|取消作业|
|/Vaults/backupJobs/operationResults/read|返回作业操作的结果。|
|/locations/allocateStamp/action|AllocateStamp 是服务使用的内部操作|
|/locations/allocatedStamp/read|GetAllocatedStamp 是服务使用的内部操作|

## <a name="microsoftrelay"></a>Microsoft.Relay

| 操作 | 说明 |
|---|---|
|/checkNamespaceAvailability/action|检查给定订阅下的命名空间可用性。|
|/register/action|注册中继资源提供程序的订阅，并启用中继资源的创建|
|/namespaces/write|创建命名空间资源并更新其属性。 命名空间的标记和状态是可更新的属性。|
|/namespaces/read|获取命名空间资源说明列表|
|/namespaces/Delete|删除命名空间资源|
|/namespaces/authorizationRules/write|创建命名空间级授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/namespaces/authorizationRules/delete|删除命名空间授权规则。 无法删除默认的命名空间授权规则。 |
|/namespaces/authorizationRules/listkeys/action|获取命名空间的连接字符串|
|/namespaces/HybridConnections/write|创建或更新 HybridConnection 属性。|
|/namespaces/HybridConnections/read|获取 HybridConnection 资源说明列表|
|/namespaces/HybridConnections/Delete|用于删除 HybridConnection 资源的操作|
|/namespaces/HybridConnections/authorizationRules/write|创建 HybridConnection 授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/namespaces/HybridConnections/authorizationRules/delete|用于删除 HybridConnection 授权规则的操作|
|/namespaces/HybridConnections/authorizationRules/listkeys/action|获取 HybridConnection 的连接字符串|
|/namespaces/WcfRelays/write|创建或更新 WcfRelay 属性。|
|/namespaces/WcfRelays/read|获取 WcfRelay 资源说明列表|
|/namespaces/WcfRelays/Delete|用于删除 WcfRelay 资源的操作|
|/namespaces/WcfRelays/authorizationRules/write|创建 WcfRelay 授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/namespaces/WcfRelays/authorizationRules/delete|用于删除 WcfRelay 授权规则的操作|
|/namespaces/WcfRelays/authorizationRules/listkeys/action|获取 WcfRelay 的连接字符串|

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

| 操作 | 说明 |
|---|---|
|/AvailabilityStatuses/read|获取指定范围内所有资源的可用性状态|
|/AvailabilityStatuses/current/read|获取指定资源的可用性状态|

## <a name="microsoftresources"></a>Microsoft.Resources

| 操作 | 说明 |
|---|---|
|/checkResourceName/action|检查资源名称的有效性。|
|/providers/read|获取提供程序的列表。|
|/subscriptions/read|获取订阅的列表。|
|/subscriptions/operationresults/read|获取订阅操作结果。|
|/subscriptions/providers/read|获取或列出资源提供程序。|
|/subscriptions/tagNames/read|获取或列出订阅标记。|
|/subscriptions/tagNames/write|添加订阅标记。|
|/subscriptions/tagNames/delete|删除订阅标记。|
|/subscriptions/tagNames/tagValues/read|获取或列出订阅标记值。|
|/subscriptions/tagNames/tagValues/write|添加订阅标记值。|
|/subscriptions/tagNames/tagValues/delete|删除订阅标记值。|
|/subscriptions/resources/read|获取订阅的资源。|
|/subscriptions/resourceGroups/read|获取或列出资源组。|
|/subscriptions/resourceGroups/write|创建或更新资源组。|
|/subscriptions/resourceGroups/delete|删除资源组及其所有资源。|
|/subscriptions/resourceGroups/moveResources/action|将资源从一个资源组移到另一个资源组。|
|/subscriptions/resourceGroups/validateMoveResources/action|验证是否已将资源从一个资源组移到另一个资源组。|
|/subscriptions/resourcegroups/resources/read|获取资源组的资源。|
|/subscriptions/resourcegroups/deployments/read|获取或列出部署。|
|/subscriptions/resourcegroups/deployments/write|创建或更新部署。|
|/subscriptions/resourcegroups/deployments/operationstatuses/read|获取或列出部署操作状态。|
|/subscriptions/resourcegroups/deployments/operations/read|获取或列出部署操作。|
|/subscriptions/locations/read|获取支持的位置列表。|
|/links/read|获取或列出资源链接。|
|/links/write|创建或更新资源链接。|
|/links/delete|删除资源链接。|
|/tenants/read|获取租户的列表。|
|/resources/read|基于筛选器获取资源的列表。|
|/deployments/read|获取或列出部署。|
|/deployments/write|创建或更新部署。|
|/deployments/delete|删除部署。|
|/deployments/cancel/action|取消部署。|
|/deployments/validate/action|验证部署。|
|/deployments/operations/read|获取或列出部署操作。|

## <a name="microsoftscheduler"></a>Microsoft.Scheduler

| 操作 | 说明 |
|---|---|
|/jobcollections/read|获取作业集合|
|/jobcollections/write|创建或更新作业集合。|
|/jobcollections/delete|删除作业集合。|
|/jobcollections/enable/action|启用作业集合。|
|/jobcollections/disable/action|禁用作业集合。|
|/jobcollections/jobs/read|获取作业。|
|/jobcollections/jobs/write|创建或更新作业。|
|/jobcollections/jobs/delete|删除作业。|
|/jobcollections/jobs/run/action|运行作业。|
|/jobcollections/jobs/generateLogicAppDefinition/action|基于计划程序作业生成逻辑应用定义。|
|/jobcollections/jobs/jobhistories/read|获取作业历史记录。|

## <a name="microsoftsearch"></a>Microsoft.Search

| 操作 | 说明 |
|---|---|
|/register/action|注册搜索资源提供程序的订阅，并启用搜索服务的创建。|
|/checkNameAvailability/action|检查服务名称的可用性。|
|/searchServices/write|创建或更新搜索服务。|
|/searchServices/read|读取搜索服务。|
|/searchServices/delete|删除搜索服务。|
|/searchServices/start/action|启动搜索服务。|
|/searchServices/stop/action|停止搜索服务。|
|/searchServices/listAdminKeys/action|读取管理密钥。|
|/searchServices/regenerateAdminKey/action|再生成管理密钥。|
|/searchServices/createQueryKey/action|创建查询密钥。|
|/searchServices/queryKey/read|读取查询密钥。|
|/searchServices/queryKey/delete|删除查询密钥。|

## <a name="microsoftsecurity"></a>Microsoft.Security

| 操作 | 说明 |
|---|---|
|/jitNetworkAccessPolicies/read|获取实时网络访问策略|
|/jitNetworkAccessPolicies/write|创建新的或更新现有的实时网络访问策略|
|/jitNetworkAccessPolicies/initiate/action|启动实时网络访问策略|
|/securitySolutionsReferenceData/read|获取安全解决方案引用数据|
|/securityStatuses/read|获取 Azure 资源的安全运行状况|
|/webApplicationFirewalls/read|获取 Web 应用程序防火墙|
|/webApplicationFirewalls/write|创建新的或更新现有的 Web 应用程序防火墙|
|/webApplicationFirewalls/delete|删除 Web 应用程序防火墙|
|/securitySolutions/read|获取安全解决方案|
|/securitySolutions/write|创建新的或更新现有的安全解决方案|
|/securitySolutions/delete|删除安全解决方案|
|/tasks/read|获取所有可用的安全建议|
|/tasks/dismiss/action|关闭安全建议|
|/tasks/activate/action|激活安全建议|
|/policies/read|获取安全策略|
|/policies/write|更新安全策略|
|/applicationWhitelistings/read|获取应用程序允许列表|
|/applicationWhitelistings/write|创建新的或更新现有的应用程序允许列表|

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement

| 操作 | 说明 |
|---|---|
|/subscriptions/write|创建或更新订阅|
|/gateways/write|创建或更新网关|
|/gateways/delete|删除网关|
|/gateways/read|获取网关|
|/gateways/regenerateprofile/action|再生成网关配置文件|
|/gateways/upgradetolatest/action|将网关升级到最新版本|
|/nodes/write|创建或更新节点|
|/nodes/delete|删除节点|
|/nodes/read|获取节点|
|/sessions/write|创建或更新会话|
|/sessions/read|获取会话|
|/sessions/delete|删除会话|

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

| 操作 | 说明 |
|---|---|
|/checkNameAvailability/action|检查给定订阅下的命名空间可用性。|
|/register/action|注册 ServiceBus 资源提供程序的订阅，并启用 ServiceBus 资源的创建|
|/namespaces/write|创建命名空间资源并更新其属性。 命名空间的标记和状态是可更新的属性。|
|/namespaces/read|获取命名空间资源说明列表|
|/namespaces/Delete|删除命名空间资源|
|/namespaces/metricDefinitions/read|获取命名空间指标资源说明列表|
|/namespaces/authorizationRules/write|创建命名空间级授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/namespaces/authorizationRules/read|获取命名空间授权规则说明列表。|
|/namespaces/authorizationRules/delete|删除命名空间授权规则。 无法删除默认的命名空间授权规则。 |
|/namespaces/authorizationRules/listkeys/action|获取命名空间的连接字符串|
|/namespaces/authorizationRules/regenerateKeys/action|再生成资源的主密钥或辅助密钥|
|/namespaces/diagnosticSettings/read|获取命名空间诊断设置资源说明列表|
|/namespaces/diagnosticSettings/write|获取命名空间诊断设置资源说明列表|
|/namespaces/queues/write|创建或更新队列属性。|
|/namespaces/queues/read|获取队列资源说明列表|
|/namespaces/queues/Delete|用于删除队列资源的操作|
|/namespaces/queues/authorizationRules/write|创建队列授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/namespaces/queues/authorizationRules/read| 获取队列授权规则列表|
|/namespaces/queues/authorizationRules/delete|用于删除队列授权规则的操作|
|/namespaces/queues/authorizationRules/listkeys/action|获取队列的连接字符串|
|/namespaces/queues/authorizationRules/regenerateKeys/action|再生成资源的主密钥或辅助密钥|
|/namespaces/logDefinitions/read|获取命名空间日志资源说明列表|
|/namespaces/topics/write|创建或更新主题属性。|
|/namespaces/topics/read|获取主题资源说明列表|
|/namespaces/topics/Delete|用于删除主题资源的操作|
|/namespaces/topics/authorizationRules/write|创建主题授权规则并更新其属性。 可以更新授权规则访问权限、主密钥和辅助密钥。|
|/namespaces/topics/authorizationRules/read| 获取主题授权规则列表|
|/namespaces/topics/authorizationRules/delete|用于删除主题授权规则的操作|
|/namespaces/topics/authorizationRules/listkeys/action|获取主题的连接字符串|
|/namespaces/topics/authorizationRules/regenerateKeys/action|再生成资源的主密钥或辅助密钥|
|/namespaces/topics/subscriptions/write|创建或更新 TopicSubscription 属性。|
|/namespaces/topics/subscriptions/read|获取 TopicSubscription 资源说明列表|
|/namespaces/topics/subscriptions/Delete|用于删除 TopicSubscription 资源的操作|
|/namespaces/topics/subscriptions/rules/write|创建或更新规则属性。|
|/namespaces/topics/subscriptions/rules/read|获取规则资源说明列表|
|/namespaces/topics/subscriptions/rules/Delete|用于删除规则资源的操作|

## <a name="microsoftsql"></a>Microsoft.Sql

| 操作 | 说明 |
|---|---|
|/servers/read|返回订阅上的资源组中的服务器列表|
|/servers/write|创建新服务器，或者修改订阅上某个资源组中的现有服务器的属性|
|/servers/delete|删除服务器以及所有包含的数据库和弹性池|
|/servers/import/action|在服务器上创建新数据库，并部署 DacPac 包中的架构和数据|
|/servers/upgrade/action|启用最新版本的服务器上提供的新功能，并指定数据库版本转换映射|
|/servers/VulnerabilityAssessmentScans/action|执行漏洞评估服务器扫描|
|/servers/operationResults/read|该操作用于跟踪将服务器从低版本升级到高版本的进度|
|/servers/operationResults/delete|中止正在进行的服务器版本升级|
|/servers/securityAlertPolicies/read|检索在给定的服务器上配置的服务器威胁检测策略的详细信息|
|/servers/securityAlertPolicies/write|更改给定服务器的服务器威胁检测|
|/servers/securityAlertPolicies/operationResults/read|检索服务器威胁检测策略集操作的结果|
|/servers/administrators/read|检索服务器管理员详细信息|
|/servers/administrators/write|创建或更新服务器管理员|
|/servers/administrators/delete|从服务器中删除服务器管理员|
|/servers/recoverableDatabases/read|此操作用于实时数据库的灾难恢复，以便将数据库还原到最后一个已知正常的备份点。 它返回有关最后一个正常备份点的信息，但实际上不会还原数据库。|
|/servers/serviceObjectives/read|检索可在给定服务器上实现的服务级别目标（也称为性能层）的列表|
|/servers/firewallRules/read|检索服务器防火墙规则详细信息|
|/servers/firewallRules/write|创建或更新服务器防火墙规则，该规则控制允许连接到服务器的 IP 地址范围|
|/servers/firewallRules/delete|从服务器中删除防火墙规则|
|/servers/administratorOperationResults/read|检索服务器管理员操作结果|
|/servers/recommendedElasticPools/read|检索针对弹性数据库池的建议，以便根据历史资源利用率降低成本或提高性能|
|/servers/recommendedElasticPools/metrics/read|检索给定服务器的建议弹性数据库池的指标|
|/servers/recommendedElasticPools/databases/read|检索数据库，这些数据库应添加到为给定服务器建议的弹性数据库池|
|/servers/elasticPools/read|检索给定服务器上的弹性数据库池的详细信息|
|/servers/elasticPools/write|创建新的或更改现有的弹性数据库池属性|
|/servers/elasticPools/delete|删除现有的弹性数据库池|
|/servers/elasticPools/operationResults/read|检索有关给定弹性数据库池操作的详细信息|
|/servers/elasticPools/providers/Microsoft.Insights/<br>metricDefinitions/read|返回可用于弹性数据库池的指标类型|
|/servers/elasticPools/providers/Microsoft.Insights/<br>diagnosticSettings/read|获取资源的诊断设置|
|/servers/elasticPools/providers/Microsoft.Insights/<br>diagnosticSettings/write|创建或更新资源的诊断设置|
|/servers/elasticPools/metrics/read|返回弹性数据库池资源利用率指标|
|/servers/elasticPools/elasticPoolDatabaseActivity/read|检索属于弹性数据库池的给定数据库的活动和详细信息|
|/servers/elasticPools/advisors/read|返回可用于弹性池的顾问列表|
|/servers/elasticPools/advisors/write|在弹性池级别更新顾问的自动执行状态。|
|/servers/elasticPools/advisors/recommendedActions/read|返回弹性池的指定顾问的建议操作列表|
|/servers/elasticPools/advisors/recommendedActions/write|对弹性池应用建议的操作|
|/servers/elasticPools/elasticPoolActivity/read|检索给定弹性数据库池的活动和详细信息|
|/servers/elasticPools/databases/read|检索属于指定服务器上的弹性数据库池的数据库列表和详细信息|
|/servers/auditingPolicies/read|检索在给定的服务器上配置的默认服务器表审核策略的详细信息|
|/servers/auditingPolicies/write|更改给定服务器的默认服务器表审核|
|/servers/disasterRecoveryConfiguration/operationResults/read|获取灾难恢复配置操作结果|
|/servers/advisors/read|返回可用于服务器的顾问列表|
|/servers/advisors/write|在服务器级别更新顾问的自动执行状态。|
|/servers/advisors/recommendedActions/read|返回服务器的指定顾问的建议操作列表|
|/servers/advisors/recommendedActions/write|对服务器应用建议的操作|
|/servers/usages/read|返回服务器 DTU 配额，以及服务器中所有数据库当前消耗的 DTU|
|/servers/elasticPoolEstimates/read|返回已为此服务器创建的弹性池估计列表|
|/servers/elasticPoolEstimates/write|为提供的数据库列表创建新的弹性池估计|
|/servers/auditingSettings/read|检索在给定服务器上配置的服务器 Blob 审核策略的详细信息|
|/servers/auditingSettings/write|更改给定服务器的服务器 Blob 审核|
|/servers/auditingSettings/operationResults/read|检索服务器 Blob 审核策略集操作的结果|
|/servers/backupLongTermRetentionVaults/read|此操作用于获取备份长期保留保管库。 它返回有关已注册到此服务器的保管库的信息。|
|/servers/backupLongTermRetentionVaults/write|注册备份长期保留保管库|
|/servers/restorableDroppedDatabases/read|检索已在给定的服务器中删除，但仍包含在保留策略中的数据库的列表。 此操作返回数据库列表和关联的元数据，例如删除日期。|
|/servers/databases/read|返回订阅上的资源组中的服务器列表|
|/servers/databases/write|创建新服务器，或者修改订阅上某个资源组中的现有服务器的属性|
|/servers/databases/delete|删除服务器以及所有包含的数据库和弹性池|
|/servers/databases/export/action|在服务器上创建新数据库，并部署 DacPac 包中的架构和数据|
|/servers/databases/VulnerabilityAssessmentScans/action|执行漏洞评估数据库扫描。|
|/servers/databases/pause/action|暂停数据仓库版数据库|
|/servers/databases/resume/action|恢复数据仓库版数据库|
|/servers/databases/operationResults/read|该操作用于跟踪长时间运行的数据库操作（例如缩放）的进度。|
|/servers/databases/replicationLinks/read|返回为特定数据库建立的复制链接的详细信息|
|/servers/databases/replicationLinks/delete|强制终止复制关系，这可能会丢失数据|
|/servers/databases/replicationLinks/unlink/action|强制终止或者在与合作伙伴同步后终止复制关系|
|/servers/databases/replicationLinks/failover/action|从主节点同步所有更改后执行故障转移，使此数据库成为复制关系的主节点，并使远程主节点成为辅助节点|
|/servers/databases/replicationLinks/forceFailoverAllowDataLoss/action|立即故障转移（可能会丢失数据），使此数据库成为复制关系的主节点，并使远程主节点成为辅助节点|
|/servers/databases/replicationLinks/updateReplicationMode/action|将链接的复制模式更新为同步或异步模式|
|/servers/databases/replicationLinks/operationResults/read|获取对数据库复制链接长时间运行的操作的状态|
|/servers/databases/dataMaskingPolicies/read|检索在给定数据库上配置的数据掩码策略的详细信息|
|/servers/databases/dataMaskingPolicies/write|更改给定数据库的数据掩码策略|
|/servers/databases/dataMaskingPolicies/rules/read|检索在给定数据库上配置的数据掩码策略规则的详细信息|
|/servers/databases/dataMaskingPolicies/rules/write|更改给定数据库的数据掩码策略规则|
|/servers/databases/securityAlertPolicies/read|检索在给定数据库上配置的威胁检测策略的详细信息|
|/servers/databases/securityAlertPolicies/write|更改给定数据库的威胁检测策略|
|/servers/databases/providers/Microsoft.Insights/<br>metricDefinitions/read|返回可用于数据库的指标类型|
|/servers/databases/providers/Microsoft.Insights/<br>diagnosticSettings/read|获取资源的诊断设置|
|/servers/databases/providers/Microsoft.Insights/<br>diagnosticSettings/write|创建或更新资源的诊断设置|
|/servers/databases/providers/Microsoft.Insights/<br>logDefinitions/read|获取数据库的可用日志|
|/servers/databases/topQueries/read|返回所选查询在所选时间段内的聚合运行时统计信息|
|/servers/databases/topQueries/queryText/read|返回所选查询 ID 的 Transact-SQL 文本|
|/servers/databases/topQueries/statistics/read|返回所选查询在所选时间段内的聚合运行时统计信息|
|/servers/databases/connectionPolicies/read|检索在给定数据库上配置的连接策略的详细信息|
|/servers/databases/connectionPolicies/write|更改给定数据库的连接策略|
|/servers/databases/metrics/read|返回数据库资源利用率指标|
|/servers/databases/auditRecords/read|检索数据库 Blob 审核记录|
|/servers/databases/transparentDataEncryption/read|检索给定数据库的透明数据加密安全功能的状态和详细信息|
|/servers/databases/transparentDataEncryption/write|为给定的数据库启用或禁用透明数据加密|
|/servers/databases/transparentDataEncryption/operationResults/read|检索给定数据库的透明数据加密安全功能的状态和详细信息|
|/servers/databases/auditingPolicies/read|检索在给定数据库上配置的表审核策略的详细信息|
|/servers/databases/auditingPolicies/write|更改给定数据库的表审核策略|
|/servers/databases/dataWarehouseQueries/read|返回所选查询 ID 的数据仓库分布查询信息|
|/servers/databases/dataWarehouseQueries/<br>dataWarehouseQuerySteps/read|返回所选步骤 ID 的数据仓库查询的分布式查询步骤信息|
|/servers/databases/serviceTierAdvisors/read|返回有关根据查询执行统计信息扩展或缩减数据库的建议，以提高性能或降低成本|
|/servers/databases/advisors/read|返回可用于数据库的顾问列表|
|/servers/databases/advisors/write|在数据库级别更新顾问的自动执行状态。|
|/servers/databases/advisors/recommendedActions/read|返回数据库的指定顾问的建议操作列表|
|/servers/databases/advisors/recommendedActions/write|对数据库应用建议的操作|
|/servers/databases/usages/read|返回可以达到的最大数据库大小，以及数据当前占用的大小|
|/servers/databases/queryStore/read|返回数据库的当前 Query Store 设置值|
|/servers/databases/queryStore/write|更新数据库的 Query Store 设置|
|/servers/databases/auditingSettings/read|检索在给定数据库上配置的 Blob 审核策略的详细信息|
|/servers/databases/auditingSettings/write|更改给定数据库的 Blob 审核策略|
|/servers/databases/schemas/tables/recommendedIndexes/read|检索数据库上的索引建议列表|
|/servers/databases/schemas/tables/recommendedIndexes/write|应用索引建议|
|/servers/databases/schemas/tables/columns/read|检索表的列列表|
|/servers/databases/missingindexes/read|返回有关要创建、修改或删除的数据库索引的建议，以提高查询性能|
|/servers/databases/missingindexes/write|在特定的数据库中使用数据库索引建议|
|/servers/databases/importExportOperationResults/read|从存储帐户中的 DacPac 返回有关数据库导入或导出操作的详细信息|
|/servers/importExportOperationResults/read|从给定服务器上的存储帐户返回数据库导入操作的列表和详细信息|

## <a name="microsoftstorage"></a>Microsoft.Storage

| 操作 | 说明 |
|---|---|
|/register/action|注册存储资源提供程序的订阅，并启用存储帐户的创建。|
|/checknameavailability/read|检查帐户名称是否有效且未被使用。|
|/storageAccounts/write|使用指定的参数创建存储帐户、更新指定存储帐户的属性或标记，或者为其添加自定义域。|
|/storageAccounts/delete|删除现有的存储帐户。|
|/storageAccounts/listkeys/action|返回指定存储帐户的访问密钥。|
|/storageAccounts/regeneratekey/action|再生成指定存储帐户的访问密钥。|
|/storageAccounts/read|返回存储帐户的列表，或获取指定存储帐户的属性。|
|/storageAccounts/listAccountSas/action|返回指定存储帐户的帐户 SAS 令牌。|
|/storageAccounts/listServiceSas/action|存储服务 SAS 令牌|
|/storageAccounts/services/diagnosticSettings/write|创建/更新存储帐户诊断设置。|
|/skus/read|列出 Microsoft.Storage 支持的 SKU。|
|/usages/read|返回指定订阅中的资源的限制和当前使用计数|
|/operations/read|轮询异步操作的状态。|
|/locations/deleteVirtualNetworkOrSubnets/action|向 Microsoft.Storage 通知正在删除虚拟网络或子网|

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

| 操作 | 说明 |
|---|---|
|/managers/clearAlerts/action|清除与设备管理器关联的所有警报。|
|/managers/getActivationKey/action|获取设备管理器的激活密钥。|
|/managers/regenerateActivationKey/action|再生成设备管理器的激活密钥。|
|/managers/regenarateRegistationCertificate/action|再生成设备管理器的注册证书。|
|/managers/getEncryptionKey/action|获取设备管理器的加密密钥。|
|/managers/read|列出或获取设备管理器|
|/managers/delete|删除设备管理器|
|/managers/write|创建或更新设备管理器|
|/managers/configureDevice/action|配置设备|
|/managers/listActivationKey/action|获取 StorSimple Device Manager 的激活密钥。|
|/managers/listPublicEncryptionKey/action|列出 StorSimple Device Manager 的加密公钥。|
|/managers/listPrivateEncryptionKey/action|获取 StorSimple Device Manager 的加密私钥。|
|/managers/provisionCloudAppliance/action|创建新的云设备。|
|/Managers/write|“创建保管库”操作创建“vault”类型的 Azure 资源|
|/Managers/read|“获取保管库”操作获取表示“vault”类型的 Azure 资源的对象|
|/Managers/delete|“删除保管库”操作删除“vault”类型的指定 Azure 资源|
|/managers/storageAccountCredentials/write|创建或更新存储帐户凭据|
|/managers/storageAccountCredentials/read|列出或获取存储帐户凭据|
|/managers/storageAccountCredentials/delete|删除存储帐户凭据|
|/managers/storageAccountCredentials/listAccessKey/action|列出存储帐户凭据的访问密钥|
|/managers/accessControlRecords/read|列出或获取访问控制记录|
|/managers/accessControlRecords/write|创建或更新访问控制记录|
|/managers/accessControlRecords/delete|删除访问控制记录|
|/managers/metrics/read|列出或获取指标|
|/managers/bandwidthSettings/read|列出带宽设置（仅适用于 8000 系列）|
|/managers/bandwidthSettings/write|新建或更新带宽设置（仅适用于 8000 系列）|
|/managers/bandwidthSettings/delete|删除现有的带宽设置（仅适用于 8000 系列）|
|/Managers/extendedInformation/read|“获取扩展信息”操作获取表示“vault”类型的 Azure 资源的对象扩展信息|
|/Managers/extendedInformation/write|“获取扩展信息”操作获取表示“vault”类型的 Azure 资源的对象扩展信息|
|/Managers/extendedInformation/delete|“获取扩展信息”操作获取表示“vault”类型的 Azure 资源的对象扩展信息|
|/managers/alerts/read|列出或获取警报|
|/managers/storageDomains/read|列出或获取存储域|
|/managers/storageDomains/write|创建或更新存储域|
|/managers/storageDomains/delete|删除存储域|
|/managers/devices/scanForUpdates/action|扫描设备中的更新。|
|/managers/devices/download/action|下载设备的更新。|
|/managers/devices/install/action|在设备上安装更新。|
|/managers/devices/read|列出或获取设备|
|/managers/devices/write|创建或更新设备|
|/managers/devices/delete|删除设备|
|/managers/devices/deactivate/action|停用设备。|
|/managers/devices/publishSupportPackage/action|发布设备的支持包供 Microsoft 支持人员进行故障排除。|
|/managers/devices/failover/action|设备故障转移。|
|/managers/devices/sendTestAlertEmail/action|将测试警报电子邮件发送到配置的电子邮件收件人。|
|/managers/devices/installUpdates/action|在设备上安装更新|
|/managers/devices/listFailoverSets/action|列出现有设备的故障转移集。|
|/managers/devices/listFailoverTargets/action|列出设备的故障转移目标|
|/managers/devices/publicEncryptionKey/action|列出设备管理器的加密公钥|
|/managers/devices/hardwareComponentGroups/<br>read|列出硬件组件组|
|/managers/devices/hardwareComponentGroups/<br>changeControllerPowerState/action|更改硬件组件组的控制器电源状态|
|/managers/devices/metrics/read|列出或获取指标|
|/managers/devices/chapSettings/write|创建或更新 Chap 设置|
|/managers/devices/chapSettings/read|列出或获取 Chap 设置|
|/managers/devices/chapSettings/delete|删除 Chap 设置|
|/managers/devices/backupScheduleGroups/read|列出或获取备份计划组|
|/managers/devices/backupScheduleGroups/write|创建或更新备份计划组|
|/managers/devices/backupScheduleGroups/delete|删除备份计划组|
|/managers/devices/updateSummary/read|列出或获取更新摘要|
|/managers/devices/migrationSourceConfigurations/<br>import/action|导入要迁移的源配置|
|/managers/devices/migrationSourceConfigurations/<br>startMigrationEstimate/action|启动作业来估计迁移过程的持续时间。|
|/managers/devices/migrationSourceConfigurations/<br>startMigration/action|使用源配置开始迁移|
|/managers/devices/migrationSourceConfigurations/<br>confirmMigration/action|确认成功迁移并提交结果。|
|/managers/devices/migrationSourceConfigurations/<br>fetchMigrationEstimate/action|提取迁移估计作业的状态。|
|/managers/devices/migrationSourceConfigurations/<br>fetchMigrationStatus/action|提取迁移的状态。|
|/managers/devices/migrationSourceConfigurations/<br>fetchConfirmMigrationStatus/action|提取迁移的确认状态。|
|/managers/devices/alertSettings/read|列出或获取警报设置|
|/managers/devices/alertSettings/write|创建或更新警报设置|
|/managers/devices/networkSettings/read|列出或获取网络设置|
|/managers/devices/networkSettings/write|新建或更新网络设置|
|/managers/devices/jobs/read|列出或获取作业|
|/managers/devices/jobs/cancel/action|取消正在运行的作业|
|/managers/devices/metricsDefinitions/read|列出或获取指标定义|
|/managers/devices/volumeContainers/write|新建或更新卷容器（仅适用于 8000 系列）|
|/managers/devices/volumeContainers/read|列出卷容器（仅适用于 8000 系列）|
|/managers/devices/volumeContainers/delete|删除现有的卷容器（仅适用于 8000 系列）|
|/managers/devices/volumeContainers/listEncryptionKeys/action|列出卷容器的加密密钥|
|/managers/devices/volumeContainers/rolloverEncryptionKey/action|滚动更新卷容器的加密密钥|
|/managers/devices/volumeContainers/metrics/read|列出指标|
|/managers/devices/volumeContainers/volumes/read|列出卷|
|/managers/devices/volumeContainers/volumes/write|新建或更新卷|
|/managers/devices/volumeContainers/volumes/delete|删除现有的卷|
|/managers/devices/volumeContainers/volumes/metrics/read|列出指标|
|/managers/devices/volumeContainers/volumes/metricsDefinitions/read|列出指标定义|
|/managers/devices/volumeContainers/metricsDefinitions/read|列出指标定义|
|/managers/devices/iscsiservers/read|列出或获取 iSCSI 服务器|
|/managers/devices/iscsiservers/write|创建或更新 iSCSI 服务器|
|/managers/devices/iscsiservers/delete|删除 iSCSI 服务器|
|/managers/devices/iscsiservers/backup/action|备份 iSCSI 服务器。|
|/managers/devices/iscsiservers/metrics/read|列出或获取指标|
|/managers/devices/iscsiservers/disks/read|列出或获取磁盘|
|/managers/devices/iscsiservers/disks/write|创建或更新磁盘|
|/managers/devices/iscsiservers/disks/delete|删除磁盘|
|/managers/devices/iscsiservers/disks/metrics/read|列出或获取指标|
|/managers/devices/iscsiservers/disks/metricsDefinitions/read|列出或获取指标定义|
|/managers/devices/iscsiservers/metricsDefinitions/read|列出或获取指标定义|
|/managers/devices/backups/read|列出或获取备份集|
|/managers/devices/backups/delete|删除备份集|
|/managers/devices/backups/restore/action|从备份集还原所有卷。|
|/managers/devices/backups/elements/clone/action|使用备份元素克隆共享或卷。|
|/managers/devices/backupPolicies/write|新建或更新备份策略（仅适用于 8000 系列）|
|/managers/devices/backupPolicies/read|列出备份策略（仅适用于 8000 系列）|
|/managers/devices/backupPolicies/delete|删除现有的备份策略（仅适用于 8000 系列）|
|/managers/devices/backupPolicies/backup/action|执行手动备份以创建受策略保护的所有卷的按需备份。|
|/managers/devices/backupPolicies/schedules/write|新建或更新计划|
|/managers/devices/backupPolicies/schedules/read|列出计划|
|/managers/devices/backupPolicies/schedules/delete|删除现有的计划|
|/managers/devices/securitySettings/update/action|更新安全设置。|
|/managers/devices/securitySettings/read|列出安全设置|
|/managers/devices/securitySettings/<br>syncRemoteManagementCertificate/action|同步设备的远程管理证书。|
|/managers/devices/securitySettings/write|新建或更新安全设置|
|/managers/devices/fileservers/read|列出或获取文件服务器|
|/managers/devices/fileservers/write|创建或更新文件服务器|
|/managers/devices/fileservers/delete|删除文件服务器|
|/managers/devices/fileservers/backup/action|备份文件服务器。|
|/managers/devices/fileservers/metrics/read|列出或获取指标|
|/managers/devices/fileservers/shares/write|创建或更新共享|
|/managers/devices/fileservers/shares/read|列出或获取共享|
|/managers/devices/fileservers/shares/delete|删除共享|
|/managers/devices/fileservers/shares/metrics/read|列出或获取指标|
|/managers/devices/fileservers/shares/metricsDefinitions/read|列出或获取指标定义|
|/managers/devices/fileservers/metricsDefinitions/read|列出或获取指标定义|
|/managers/devices/timeSettings/read|列出或获取时间设置|
|/managers/devices/timeSettings/write|新建或更新时间设置|
|/Managers/certificates/write|“更新资源证书”操作更新资源/保管库凭据证书。|
|/managers/cloudApplianceConfigurations/read|列出云设备支持的配置|
|/managers/metricsDefinitions/read|列出或获取指标定义|
|/managers/encryptionSettings/read|列出或获取加密设置|

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

| 操作 | 说明 |
|---|---|
|/streamingjobs/Start/action|启动流分析作业|
|/streamingjobs/Stop/action|停止流分析作业|
|/streamingjobs/Read|读取流分析作业|
|/streamingjobs/Write|写入流分析作业|
|/streamingjobs/Delete|删除流分析作业|
|/streamingjobs/providers/Microsoft.Insights/metricDefinitions/read|获取 streamingjobs 的可用指标|
|/streamingjobs/providers/Microsoft.Insights/diagnosticSettings/read|读取诊断设置。|
|/streamingjobs/providers/Microsoft.Insights/diagnosticSettings/write|写入诊断设置。|
|/streamingjobs/providers/Microsoft.Insights/logDefinitions/read|获取 streamingjobs 的可用日志|
|/streamingjobs/transformations/Read|读取流分析作业转换|
|/streamingjobs/transformations/Write|写入流分析作业转换|
|/streamingjobs/transformations/Delete|删除流分析作业转换|
|/streamingjobs/inputs/Read|读取流分析作业输入|
|/streamingjobs/inputs/Write|写入流分析作业输入|
|/streamingjobs/inputs/Delete|删除流分析作业输入|
|/streamingjobs/outputs/Read|读取流分析作业输出|
|/streamingjobs/outputs/Write|写入流分析作业输出|
|/streamingjobs/outputs/Delete|删除流分析作业输出|

## <a name="microsoftsupport"></a>Microsoft.Support

| 操作 | 说明 |
|---|---|
|/register/action|注册到支持资源提供程序|
|/supportTickets/read|获取支持票证详细信息（包括状态、严重性、联系详细信息和通信），或获取各个订阅中的支持票证列表。|
|/supportTickets/write|创建或更新支持票证。 可以针对技术、计费、配额或订阅管理相关问题创建支持票证。 可以更新现有支持票证的严重性、联系详细信息和通信。|

## <a name="microsoftweb"></a>Microsoft.Web

| 操作 | 说明 |
|---|---|
|/unregister/action|取消注册订阅的 Microsoft.Web 资源提供程序。|
|/validate/action|验证。|
|/register/action|注册订阅的 Microsoft.Web 资源提供程序。|
|/hostingEnvironments/Read|获取应用服务环境的属性|
|/hostingEnvironments/Write|创建新的或更新现有的应用服务环境|
|/hostingEnvironments/Delete|删除应用服务环境|
|/hostingEnvironments/reboot/Action|重启应用服务环境中的所有计算机|
|/hostingenvironments/resume/action|恢复宿主环境。|
|/hostingenvironments/suspend/action|暂停宿主环境。|
|/hostingenvironments/metricdefinitions/read|获取宿主环境的指标定义。|
|/hostingEnvironments/workerPools/Read|获取应用服务环境中辅助角色池的属性|
|/hostingEnvironments/workerPools/Write|在应用服务环境中创建新的或更新现有的辅助角色池|
|/hostingenvironments/workerpools/metricdefinitions/read|获取宿主环境的辅助角色池指标定义。|
|/hostingenvironments/workerpools/metrics/read|获取宿主环境的辅助角色池指标。|
|/hostingenvironments/workerpools/skus/read|获取宿主环境的辅助角色池 SKU。|
|/hostingenvironments/workerpools/usages/read|获取宿主环境的辅助角色池使用情况。|
|/hostingenvironments/sites/read|获取宿主环境的 Web 应用。|
|/hostingenvironments/serverfarms/read|获取宿主环境的应用服务计划。|
|/hostingenvironments/usages/read|获取宿主环境使用情况。|
|/hostingenvironments/capacities/read|获取宿主环境容量。|
|/hostingenvironments/operations/read|获取宿主环境操作。|
|/hostingEnvironments/multiRolePools/Read|获取应用服务环境中前端池的属性|
|/hostingEnvironments/multiRolePools/Write|在应用服务环境中创建新的或更新现有的前端池|
|/hostingenvironments/multirolepools/metricdefinitions/read|获取宿主环境的多角色池指标定义。|
|/hostingenvironments/multirolepools/metrics/read|获取宿主环境的多角色池指标。|
|/hostingenvironments/multirolepools/skus/read|获取宿主环境的多角色池 SKU。|
|/hostingenvironments/multirolepools/usages/read|获取宿主环境的多角色池使用情况。|
|/hostingenvironments/diagnostics/read|获取宿主环境诊断。|
|/publishingusers/read|获取发布用户。|
|/publishingusers/write|更新发布用户。|
|/checknameavailability/read|检查资源名称是否可用。|
|/geoRegions/Read|获取地理区域的列表。|
|/sites/Read|获取 Web 应用的属性|
|/sites/Write|创建新的或更新现有的 Web 应用|
|/sites/Delete|删除现有的 Web 应用|
|/sites/backup/Action|创建新的 Web 应用备份|
|/sites/publishxml/Action|获取 Web 应用的发布配置文件 xml|
|/sites/publish/Action|发布 Web 应用|
|/sites/restart/Action|重启 Web 应用|
|/sites/start/Action|启动 Web 应用|
|/sites/stop/Action|停止 Web 应用|
|/sites/slotsswap/Action|交换 Web 应用部署槽|
|/sites/slotsdiffs/Action|获取 Web 应用与槽之间的配置差异|
|/sites/applySlotConfig/Action|将目标槽中的 Web 应用槽配置应用到当前 Web 应用|
|/sites/resetSlotConfig/Action|重置 Web 应用配置|
|/sites/functions/action|函数 Web 应用。|
|/sites/listsyncfunctiontriggerstatus/action|列出同步函数触发器状态 Web 应用。|
|/sites/networktrace/action|网络跟踪 Web 应用。|
|/sites/newpassword/action|Newpassword Web 应用。|
|/sites/sync/action|同步 Web 应用。|
|/sites/operationresults/read|获取 Web 应用操作结果。|
|/sites/webjobs/read|获取 Web 应用 Web 作业。|
|/sites/backup/read|获取 Web 应用备份。|
|/sites/backup/write|更新 Web 应用备份。|
|/sites/metricdefinitions/read|获取 Web 应用指标定义。|
|/sites/metrics/read|获取 Web 应用指标。|
|/sites/continuouswebjobs/delete|删除 Web 应用连续 Web 作业。|
|/sites/continuouswebjobs/read|获取 Web 应用连续 Web 作业。|
|/sites/continuouswebjobs/start/action|启动 Web 应用连续 Web 作业。|
|/sites/continuouswebjobs/stop/action|停止 Web 应用连续 Web 作业。|
|/sites/domainownershipidentifiers/read|获取 Web 应用域所有权标识符。|
|/sites/domainownershipidentifiers/write|更新 Web 应用域所有权标识符。|
|/sites/premieraddons/delete|删除 Web 应用高级加载项。|
|/sites/premieraddons/read|获取 Web 应用高级加载项。|
|/sites/premieraddons/write|更新 Web 应用高级加载项。|
|/sites/triggeredwebjobs/delete|删除 Web 应用触发的 Web 作业。|
|/sites/triggeredwebjobs/read|获取 Web 应用触发的 Web 作业。|
|/sites/triggeredwebjobs/run/action|运行 Web 应用触发的 Web 作业。|
|/sites/hostnamebindings/delete|删除 Web 应用主机名绑定。|
|/sites/hostnamebindings/read|获取 Web 应用主机名绑定。|
|/sites/hostnamebindings/write|更新 Web 应用主机名绑定。|
|/sites/virtualnetworkconnections/delete|删除 Web 应用虚拟网络连接。|
|/sites/virtualnetworkconnections/read|获取 Web 应用虚拟网络连接。|
|/sites/virtualnetworkconnections/write|更新 Web 应用虚拟网络连接。|
|/sites/virtualnetworkconnections/gateways/read|获取 Web 应用虚拟网络连接网关。|
|/sites/virtualnetworkconnections/gateways/write|更新 Web 应用虚拟网络连接网关。|
|/sites/publishxml/read|获取 Web 应用发布 XML。|
|/sites/hybridconnectionrelays/read|获取 Web 应用混合连接中继。|
|/sites/perfcounters/read|获取 Web 应用性能计数器。|
|/sites/usages/read|获取 Web 应用使用情况。|
|/sites/slots/Write|创建新的或更新现有的 Web 应用槽|
|/sites/slots/Delete|删除现有的 Web 应用槽|
|/sites/slots/backup/Action|创建新的 Web 应用槽备份。|
|/sites/slots/publishxml/Action|获取 Web 应用槽的发布配置文件 xml|
|/sites/slots/publish/Action|发布 Web 应用槽|
|/sites/slots/restart/Action|重启 Web 应用槽|
|/sites/slots/start/Action|启动 Web 应用槽|
|/sites/slots/stop/Action|停止 Web 应用槽|
|/sites/slots/slotsswap/Action|交换 Web 应用部署槽|
|/sites/slots/slotsdiffs/Action|获取 Web 应用与槽之间的配置差异|
|/sites/slots/applySlotConfig/Action|将目标槽中的 Web 应用槽配置应用到当前槽|
|/sites/slots/resetSlotConfig/Action|重置 Web 应用槽配置|
|/sites/slots/Read|获取 Web 应用部署槽的属性|
|/sites/slots/newpassword/action|Newpassword Web 应用槽。|
|/sites/slots/sync/action|同步 Web 应用槽。|
|/sites/slots/operationresults/read|获取 Web 应用槽操作结果。|
|/sites/slots/webjobs/read|获取 Web 应用槽 Web 作业。|
|/sites/slots/backup/write|更新 Web 应用槽备份。|
|/sites/slots/metricdefinitions/read|获取 Web 应用槽指标定义。|
|/sites/slots/metrics/read|获取 Web 应用槽指标。|
|/sites/slots/continuouswebjobs/delete|删除 Web 应用槽连续 Web 作业。|
|/sites/slots/continuouswebjobs/read|获取 Web 应用槽连续 Web 作业。|
|/sites/slots/continuouswebjobs/start/action|启动 Web 应用槽连续 Web 作业。|
|/sites/slots/continuouswebjobs/stop/action|停止 Web 应用槽连续 Web 作业。|
|/sites/slots/premieraddons/delete|删除 Web 应用槽高级加载项。|
|/sites/slots/premieraddons/read|获取 Web 应用槽高级加载项。|
|/sites/slots/premieraddons/write|更新 Web 应用槽高级加载项。|
|/sites/slots/triggeredwebjobs/delete|删除 Web 应用槽触发的 Web 作业。|
|/sites/slots/triggeredwebjobs/read|获取 Web 应用槽触发的 Web 作业。|
|/sites/slots/triggeredwebjobs/run/action|运行 Web 应用槽触发的 Web 作业。|
|/sites/slots/hostnamebindings/delete|删除 Web 应用槽主机名绑定。|
|/sites/slots/hostnamebindings/read|获取 Web 应用槽主机名绑定。|
|/sites/slots/hostnamebindings/write|更新 Web 应用槽主机名绑定。|
|/sites/slots/phplogging/read|获取 Web 应用槽 Phplogging。|
|/sites/slots/virtualnetworkconnections/delete|删除 Web 应用槽虚拟网络连接。|
|/sites/slots/virtualnetworkconnections/read|获取 Web 应用槽虚拟网络连接。|
|/sites/slots/virtualnetworkconnections/write|更新 Web 应用槽虚拟网络连接。|
|/sites/slots/virtualnetworkconnections/gateways/write|更新 Web 应用槽虚拟网络连接网关。|
|/sites/slots/usages/read|获取 Web 应用槽使用情况。|
|/sites/slots/hybridconnection/delete|删除 Web 应用槽混合连接。|
|/sites/slots/hybridconnection/read|获取 Web 应用槽混合连接。|
|/sites/slots/hybridconnection/write|更新 Web 应用槽混合连接。|
|/sites/slots/config/Read|获取 Web 应用槽的配置设置|
|/sites/slots/config/list/Action|列出 Web 应用槽的安全敏感设置，例如发布凭据、应用设置和连接字符串|
|/sites/slots/config/Write|更新 Web 应用槽的配置设置|
|/sites/slots/config/delete|删除 Web 应用槽配置。|
|/sites/slots/instances/read|获取 Web 应用槽实例。|
|/sites/slots/instances/processes/read|获取 Web 应用槽实例进程。|
|/sites/slots/instances/deployments/read|获取 Web 应用槽实例部署。|
|/sites/slots/sourcecontrols/Read|获取 Web 应用槽的源代码管理配置设置|
|/sites/slots/sourcecontrols/Write|更新 Web 应用槽的源代码管理配置设置|
|/sites/slots/sourcecontrols/Delete|删除 Web 应用槽的源代码管理配置设置|
|/sites/slots/restore/read|获取 Web 应用槽还原设置。|
|/sites/slots/analyzecustomhostname/read|获取 Web 应用槽分析自定义主机名。|
|/sites/slots/backups/Read|获取 Web 应用槽的备份属性|
|/sites/slots/backups/list/action|列出 Web 应用槽备份。|
|/sites/slots/backups/restore/action|还原 Web 应用槽备份。|
|/sites/slots/deployments/delete|删除 Web 应用槽部署。|
|/sites/slots/deployments/read|获取 Web 应用槽部署。|
|/sites/slots/deployments/write|更新 Web 应用槽部署。|
|/sites/slots/deployments/log/read|获取 Web 应用槽部署日志。|
|/sites/hybridconnection/delete|删除 Web 应用混合连接。|
|/sites/hybridconnection/read|获取 Web 应用混合连接。|
|/sites/hybridconnection/write|更新 Web 应用混合连接。|
|/sites/recommendationhistory/read|获取 Web 应用建议历史记录。|
|/sites/recommendations/Read|获取 Web 应用建议的列表。|
|/sites/recommendations/disable/action|禁用 Web 应用建议。|
|/sites/config/Read|获取 Web 应用配置设置|
|/sites/config/list/Action|列出 Web 应用的安全敏感设置，例如发布凭据、应用设置和连接字符串|
|/sites/config/Write|更新 Web 应用的配置设置|
|/sites/config/delete|删除 Web 应用配置。|
|/sites/instances/read|获取 Web 应用实例。|
|/sites/instances/processes/delete|删除 Web 应用实例进程。|
|/sites/instances/processes/read|获取 Web 应用实例进程。|
|/sites/instances/deployments/read|获取 Web 应用实例部署。|
|/sites/sourcecontrols/Read|获取 Web 应用的源代码管理配置设置|
|/sites/sourcecontrols/Write|更新 Web 应用的源代码管理配置设置|
|/sites/sourcecontrols/Delete|删除 Web 应用的源代码管理配置设置|
|/sites/restore/read|获取 Web 应用还原设置。|
|/sites/analyzecustomhostname/read|分析自定义主机名。|
|/sites/backups/Read|获取 Web 应用的备份属性|
|/sites/backups/list/action|列出 Web 应用备份。|
|/sites/backups/restore/action|还原 Web 应用备份。|
|/sites/snapshots/read|获取 Web 应用快照。|
|/sites/functions/delete|删除 Web 应用函数。|
|/sites/functions/listsecrets/action|列出机密 Web 应用函数。|
|/sites/functions/read|获取 Web 应用函数。|
|/sites/functions/write|更新 Web 应用函数。|
|/sites/deployments/delete|删除 Web 应用部署。|
|/sites/deployments/read|获取 Web 应用部署。|
|/sites/deployments/write|更新 Web 应用部署。|
|/sites/deployments/log/read|获取 Web 应用部署日志。|
|/sites/diagnostics/read|获取 Web 应用诊断。|
|/sites/diagnostics/workerprocessrecycle/read|获取 Web 应用诊断工作进程回收。|
|/sites/diagnostics/workeravailability/read|获取 Web 应用诊断 Workeravailability。|
|/sites/diagnostics/runtimeavailability/read|获取 Web 应用诊断运行时可用性。|
|/sites/diagnostics/cpuanalysis/read|获取 Web 应用 Cpuanalysis。|
|/sites/diagnostics/servicehealth/read|获取 Web 应用诊断服务运行状况。|
|/sites/diagnostics/frebanalysis/read|获取 Web 应用诊断 FREB 分析。|
|/availablestacks/read|获取可用堆栈。|
|/isusernameavailable/read|检查用户名是否可用。|
|/Microsoft.Web/apiManagementAccounts/<br>apis/Read|获取 API 列表。|
|/Microsoft.Web/apiManagementAccounts/<br>apis/Write|添加新的或更新现有的 API。|
|/Microsoft.Web/apiManagementAccounts/<br>apis/Delete|删除现有的 API。|
|/Microsoft.Web/apiManagementAccounts/<br>apis/connections/Read|获取连接列表。|
|/Microsoft.Web/apiManagementAccounts/<br>apis/connections/Write|保存新连接，或更新现有连接。|
|/Microsoft.Web/apiManagementAccounts/<br>apis/connections/Delete|删除现有连接。|
|/Microsoft.Web/apiManagementAccounts/<br>apis/connections/connectionAcls/Read|读取 ConnectionAcls|
|/Microsoft.Web/apiManagementAccounts/<br>apis/connections/connectionAcls/Write|添加或更新 ConnectionAcl|
|/Microsoft.Web/apiManagementAccounts/<br>apis/connections/connectionAcls/Delete|删除 ConnectionAcl|
|/Microsoft.Web/apiManagementAccounts/<br>apis/connectionAcls/Read|读取 API 的 ConnectionAcls|
|/Microsoft.Web/apiManagementAccounts/<br>apis/apiAcls/Read|读取 ConnectionAcls|
|/Microsoft.Web/apiManagementAccounts/<br>apis/apiAcls/Write|创建或更新 API ACL|
|/Microsoft.Web/apiManagementAccounts/<br>apis/apiAcls/Delete|删除 API ACL|
|/serverfarms/Read|获取应用服务计划的属性|
|/serverfarms/Write|创建新的或更新现有的应用服务计划。|
|/serverfarms/Delete|删除现有的应用服务计划|
|/serverfarms/restartSites/Action|重启应用服务计划中的所有 Web 应用|
|/serverfarms/operationresults/read|获取应用服务计划操作结果。|
|/serverfarms/capabilities/read|获取应用服务计划功能。|
|/serverfarms/metricdefinitions/read|获取应用服务计划指标定义。|
|/serverfarms/metrics/read|获取应用服务计划指标。|
|/serverfarms/hybridconnectionplanlimits/read|获取应用服务计划混合连接计划限制。|
|/serverfarms/virtualnetworkconnections/read|获取应用服务计划虚拟网络连接。|
|/serverfarms/virtualnetworkconnections/routes/delete|删除应用服务计划虚拟网络连接路由。|
|/serverfarms/virtualnetworkconnections/routes/read|获取应用服务计划虚拟网络连接路由。|
|/serverfarms/virtualnetworkconnections/routes/write|更新应用服务计划虚拟网络连接路由。|
|/serverfarms/virtualnetworkconnections/gateways/write|更新应用服务计划虚拟网络连接网关。|
|/serverfarms/firstpartyapps/settings/delete|删除应用服务计划第一方应用设置。|
|/serverfarms/firstpartyapps/settings/read|获取应用服务计划第一方应用设置。|
|/serverfarms/firstpartyapps/settings/write|更新应用服务计划第一方应用设置。|
|/serverfarms/sites/read|获取应用服务计划 Web 应用。|
|/serverfarms/workers/reboot/action|重启应用服务计划辅助角色。|
|/serverfarms/hybridconnectionrelays/read|获取应用服务计划混合连接中继。|
|/serverfarms/skus/read|获取应用服务计划 SKU。|
|/serverfarms/usages/read|获取应用服务计划使用情况。|
|/serverfarms/hybridconnectionnamespaces/relays/sites/read|获取应用服务计划混合连接命名空间中继 Web 应用。|
|/ishostnameavailable/read|检查主机名是否可用。|
|/connectionGateways/Read|获取连接网关列表。|
|/connectionGateways/Write|创建或更新连接网关。|
|/connectionGateways/Delete|删除连接网关。|
|/connectionGateways/Join/Action|加入连接网关。|
|/classicmobileservices/read|获取经典移动服务。|
|/skus/read|获取 SKU。|
|/certificates/Read|获取证书列表。|
|/certificates/Write|添加新的或更新现有的证书。|
|/certificates/Delete|删除现有证书。|
|/operations/read|获取操作。|
|/recommendations/Read|获取订阅的建议列表。|
|/ishostingenvironmentnameavailable/read|检查宿主环境名称是否可用。|
|/apiManagementAccounts/Read|获取 ApiManagementAccounts 的列表。|
|/apiManagementAccounts/Write|添加新的或更新现有的 ApiManagementAccount|
|/apiManagementAccounts/Delete|删除现有的 ApiManagementAccount|
|/apiManagementAccounts/connectionAcls/Read|获取连接 ACL 列表。|
|/apiManagementAccounts/apiAcls/Read|读取 ConnectionAcls|
|/connections/Read|获取连接列表。|
|/connections/Write|创建或更新连接。|
|/connections/Delete|删除连接。|
|/connections/Join/Action|加入连接。|
|/connections/confirmconsentcode/action|确认连接许可代码。|
|/connections/listconsentlinks/action|列出连接的许可链接。|
|/deploymentlocations/read|获取部署位置。|
|/sourcecontrols/read|获取源代码管理。|
|/sourcecontrols/write|更新源代码管理。|
|/managedhostingenvironments/read|获取托管的宿主环境。|
|/managedhostingenvironments/sites/read|获取托管的宿主环境的 Web 应用。|
|/managedhostingenvironments/serverfarms/read|获取托管的宿主环境的应用服务计划。|
|/locations/managedapis/read|获取位置托管 API。|
|/locations/apioperations/read|获取位置 API 操作。|
|/locations/connectiongatewayinstallations/read|获取位置连接网关安装。|
|/listSitesAssignedToHostName/Read|获取分配给主机名的站点名称。|

## <a name="next-steps"></a>后续步骤

- 了解如何[创建自定义角色](/documentation/articles/role-based-access-control-custom-roles/)。

- 查看[内置的 RBAC 角色](/documentation/articles/role-based-access-built-in-roles/)。

- 了解如何[按资源](/documentation/articles/role-based-access-control-configure/)管理访问权限分配

