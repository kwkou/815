<properties
    pageTitle="池创建事件 - Azure | Azure"
    caps.latest.revision="4"
    author="tamram"
    manager="timlt" />
<tags
    ms.custom=""
    ms.date="2017-02-01"
    wacn.date="05/15/2017"
    ms.prod="azure"
    ms.reviewer=""
    ms.service="batch"
    ms.suite=""
    ms.tgt_pltfrm=""
    ms.topic="reference"
    ms.assetid="8473bab4-6c16-4ee2-8d78-22fa3ca87487"
    ms.author="tamram"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="fa47cd86758122a0f63b27268f6a582da0fe0689"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="pool-create-event"></a>池创建事件
池创建事件日志正文

## <a name="remarks"></a>备注
 池创建后，会发出此事件。 日志内容将公开有关池的常规信息。 请注意，如果池的目标大小大于 0 个计算节点，则池调整大小启动事件将在此事件之后立即发生。

 以下示例显示了使用 CloudServiceConfiguration 属性所创建池的池创建事件的正文。

    {
        "id": "myPool1",
        "displayName": "Production Pool",
        "vmSize": "Small",
        "cloudServiceConfiguration": {
            "osFamily": "3",
            "targetOsVersion": "*"
        },
        "networkConfiguration": {
            "subnetId": " "
        },
        "resizeTimeout": "300000",
        "targetDedicated": 2,
        "maxTasksPerNode": 1,
        "vmFillType": "Spread",
        "enableAutoScale": false,
        "enableInterNodeCommunication": false,
        "isAutoPool": false
    }

|元素|类型|说明|
|-------------|----------|-----------|
|id|String|池的 id。|
|displayName|String|池的显示名称。|
|vmSize|String|池中虚拟机的大小。 池中所有虚拟机的大小相同。 <br/><br/> 有关云服务池（使用 cloudServiceConfiguration 创建的池）的虚拟机可用大小的信息，请参阅[云服务的大小](/documentation/articles/cloud-services-sizes-specs/)。 批处理支持除 `ExtraSmall` 以外的所有云服务 VM 大小。<br/><br/> 有关使用虚拟机应用商店中映像的池（使用 virtualMachineConfiguration 创建的池）的可用 VM 大小的信息，请参阅[虚拟机的大小](/documentation/articles/virtual-machines-linux-sizes/) (Linux) 或[虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes/) (Windows)。 Batch 支持除 `STANDARD_A0` 和高级存储大小（`STANDARD_GS`、`STANDARD_DS` 和 `STANDARD_DSV2` 系列）以外所有的 Azure VM 大小。|
|[cloudServiceConfiguration](#bk_csconf)|复杂类型|池的云服务配置。|
|[virtualMachineConfiguration](#bk_vmconf)|复杂类型|池的虚拟机配置。|
|[networkConfiguration](#bk_netconf)|复杂类型|池的网络配置。|
|resizeTimeout|时间|为针对池的上一次调整大小操作指定的将计算节点分配到池的超时。  （创建池时的初始大小计为一次调整大小。）|
|targetDedicated|Int32|池请求的计算节点数。|
|enableAutoScale|Bool|指定池大小是否随时间自动调整。|
|enableInterNodeCommunication|Bool|指定是否针对节点间的直接通信设置池。|
|isAutoPool|Bool|指定是否已通过作业的 AutoPool 机制创建池。|
|maxTasksPerNode|Int32|可在池中单个计算节点上并发运行的任务的最大数目。|
|vmFillType|String|定义批处理服务如何在池中不同的计算节点之间分配任务。 有效值为 Spread 或 Pack。|

###  <a name="bk_csconf"></a> cloudServiceConfiguration

|元素名称|类型|说明|
|------------------|----------|-----------|
|osFamily|String|要安装在池中虚拟机上的 Azure 来宾 OS 系列。<br /><br /> 可能的值包括：<br /><br /> **2** - OS 系列 2，等效于 Windows Server 2008 R2 SP1。<br /><br /> **3** - OS 系列 3，等效于Windows Server 2012。<br /><br /> **4** - OS 系列 4，等效于 Windows Server 2012 R2。<br /><br /> 有关详细信息，请参阅 [Azure 来宾 OS 版本](/documentation/articles/cloud-services-guestos-update-matrix/#releases/)。|
|targetOSVersion|String|要安装在池中虚拟机上的 Azure 来宾 OS 版本。<br /><br /> 默认值为 **\***，用于指定特定系列的最新操作系统版本。<br /><br /> 有关其他允许的值的信息，请参阅 [Azure 来宾 OS 版本](/documentation/articles/cloud-services-guestos-update-matrix/#releases/)。|

###  <a name="bk_vmconf"></a> virtualMachineConfiguration

|元素名称|类型|说明|
|------------------|----------|-----------|
|[imageReference](#bk_imgref)|复杂类型|指定关于要使用的平台或应用商店映像的信息。|
|nodeAgentSKUId|String|在计算节点上预配的批处理节点代理的 SKU。|
|[windowsConfiguration](#bk_winconf)|复杂类型|指定虚拟机上的 Windows 操作系统设置。 如果 imageReference 引用 Linux OS 映像，则不能指定此属性。|

###  <a name="bk_imgref"></a> imageReference

|元素名称|类型|说明|
|------------------|----------|-----------|
|发布者|String|映像的发布者。|
|offer|String|映像的产品/服务。|
|sku|String|映像的 SKU。|
|版本|String|映像的版本。|

###  <a name="bk_winconf"></a> windowsConfiguration

|元素名称|类型|说明|
|------------------|----------|-----------|
|enableAutomaticUpdates|布尔|指示是否对虚拟机启用自动更新。 如果未指定此属性，则默认值为 true。|

###  <a name="bk_netconf"></a> networkConfiguration

|元素名称|类型|说明|
|------------------|--------------|----------|
|subnetId|String|指定创建池计算节点的子网的资源标识符。|

