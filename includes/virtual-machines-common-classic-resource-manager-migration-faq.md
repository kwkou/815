# <a name="frequently-asked-questions-about-classic-to-azure-resource-manager-migration"></a>有关从经典部署模型迁移到 Azure Resource Manager 部署模型的常见问题

## <a name="does-this-migration-plan-affect-any-of-my-existing-services-or-applications-that-run-on-azure-virtual-machines"></a>此迁移计划是否影响 Azure 虚拟机上运行的任何现有服务或应用程序？ 

不会。 VM（经典）是公开上市的完全受支持的服务。 你可以继续使用这些资源来拓展你在 Azure 上的足迹。

## <a name="what-happens-to-my-vms-if-i-dont-plan-on-migrating-in-the-near-future"></a>如果我近期不打算迁移，我的 VM 会出现什么情况？ 

我们近期不会淘汰现有的经典 API 和资源模型。 我们想要通过 Resource Manager 部署模型中提供的高级功能，让迁移变得简单。 强烈建议查看 Resource Manager 下 IaaS 包含的[一些改进](/documentation/articles/resource-manager-deployment-model/)。

## <a name="what-does-this-migration-plan-mean-for-my-existing-tooling"></a>对于我现有的工具而言，此迁移计划有何意义？ 

将工具更新为 Resource Manager 部署模型，是必须在迁移计划中考虑的最重要的更改之一。

## <a name="how-long-will-the-management-plane-downtime-be"></a>管理平面的停机时间持续多久？ 

这取决于迁移的资源数量。 对于较小型部署（几十个 VM），整个迁移过程应该不超过一小时。 如果是大规模部署（数百个 VM），迁移过程可能需要花费几个小时。

## <a name="can-i-roll-back-after-my-migrating-resources-are-committed-in-resource-manager"></a>在 Resource Manager 中提交迁移的资源之后，是否还可以回滚？ 

只要资源处于准备就绪状态，就可以中止迁移。 但是，在通过提交操作成功迁移资源之后，就不支持回滚。

## <a name="can-i-roll-back-my-migration-if-the-commit-operation-fails"></a>提交操作失败时，是否可以回滚迁移？ 

如果提交操作失败，就无法中止迁移。 包括提交操作在内的所有迁移操作都是幂等的。 因此，建议在片刻之后重试操作。 如果仍遇到错误，请创建支持票证，或在 [VM 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=WAVirtualMachinesforWindows)上创建标记为 ClassicIaaSMigration 的论坛帖子。

## <a name="do-i-have-to-buy-another-express-route-circuit-if-i-have-to-use-iaas-under-resource-manager"></a>如果我必须使用 Resource Manager 下的 IaaS，是否必须购买其他 ExpressRoute 线路？ 

不会。 我们近期实现了[将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-move/)。 如果你已有 ExpressRoute 线路，则不需要购买新的线路。

## <a name="what-if-i-had-configured-role-based-access-control-policies-for-my-classic-iaas-resources"></a>如果我已经为经典 IaaS 资源配置基于角色的访问控制策略，该怎么办？ 

在迁移期间，资源从经典资源转换为 Resource Manager 资源。 因此，建议计划需要在迁移之后进行的 RBAC 策略更新。

## <a name="what-if-im-using-azure-site-recovery-or-azure-backup-today"></a>如果我目前正在使用 Azure Site Recovery 或 Azure 备份，该怎么办？ 

若要迁移允许进行备份的虚拟机，请参阅[已经在备份保管库中备份了经典 VM。想要将 VM 从经典模型迁移到 Resource Manager 模型，如何在恢复服务保管库中备份它们？](/documentation/articles/backup-azure-backup-ibiza-faq/)

## <a name="can-i-validate-my-subscription-or-resources-to-see-if-theyre-capable-of-migration"></a>我是否可以验证订阅或资源，以查看其是否能够迁移？ 

是的。 在平台支持的迁移选项中，准备迁移的第一个步骤，就是验证资源是否能够进行迁移。 如果验证操作失败，用户会收到包含无法完成迁移的所有原因的消息。

## <a name="what-happens-if-i-run-into-a-quota-error-while-preparing-the-iaas-resources-for-migration"></a>如果我在准备要迁移的 IaaS 资源时遇到配额错误，会发生什么情况？ 

建议你中止迁移，然后记录支持请求，以在要迁移 VM 的区域中增加配额。 配额请求经过批准后，可以重新开始执行迁移步骤。

## <a name="how-do-i-report-an-issue"></a>如何报告问题？ 

请在 [VM 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=WAVirtualMachinesforWindows)上使用关键字 ClassicIaaSMigration 发布有关迁移的问题和疑惑。 建议你将所有问题都发布在此论坛上。 如果你有支持协定，也欢迎你记录支持票证。

## <a name="what-if-i-dont-like-the-names-of-the-resources-that-the-platform-chose-during-migration"></a>如果我不喜欢平台在迁移期间选择的资源名称，该怎么做？ 

在经典部署模型中为其显式提供名称的所有资源都会在迁移期间得到保留。 在某些情况下，会创建新资源。 例如：为每个 VM 创建网络接口。 目前无法控制迁移期间所创建的这些新资源的名称。 请在 [Azure 反馈论坛](http://feedback.azure.com)上针对此功能进行投票。

## <a name="can-i-migrate-expressroute-circuits-used-across-subscriptions-with-authorization-links"></a>是否可以使用授权链接迁移跨订阅使用的 ExpressRoute 线路？ 

不停机无法自动迁移使用跨订阅授权链接的 ExpressRoute 线路。 我们提供了有关如何使用手动步骤迁移这些线路的指南。 有关步骤和详细信息，请参阅[将 ExpressRoute 线路和关联的虚拟网络从经典部署模型迁移到 Resource Manager 部署模型](/documentation/articles/expressroute-migration-classic-resource-manager/)。

## <a name="i-got-a-message-vm-is-reporting-the-overall-agent-status-as-not-ready-hence-the-vm-cannot-be-migrated-ensure-that-the-vm-agent-is-reporting-overall-agent-status-as-ready-or-vm-contains-extension-whose-status-is-not-being-reported-from-the-vm-hence-this-vm-cannot-be-migrated-"></a>我收到一条消息，指出*“VM 报告总体代理状态为‘未就绪’。因此，此 VM 无法迁移。请确保 VM 代理报告总体代理状态为‘就绪’”*或*“VM 包含 VM 未报告其状态的扩展。 因此，此 VM 无法迁移。” *

当 VM 未建立到 Internet 的出站连接时，将收到此消息。 VM 代理使用出站连接访问 Azure 存储帐户，每隔五分钟更新一次代理状态。