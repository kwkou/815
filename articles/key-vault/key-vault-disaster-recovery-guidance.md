<properties
    pageTitle="发生影响 Azure Key Vault 的 Azure 服务中断时应采取的措施 | Azure"
    description="了解发生影响 Azure Key Vault 的 Azure 服务中断时该怎么办。"
    services="key-vault"
    documentationcenter=""
    author="adamglick"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="19a9af63-3032-447b-9d1a-b0125f384edb"
    ms.service="key-vault"
    ms.workload="key-vault"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/27/2017"
    ms.author="sumedhb;aglick" />

# Azure 密钥保管库可用性和冗余
Azure 密钥保管库具有多层冗余功能，确保密钥和机密持续可供应用程序使用，即使服务的单个组件发生故障也是如此。

密钥保管库的内容将在区域中复制，并且会复制到至少 150 英里以外（但位于同一个地理位置）的次要区域。这可以保持密钥和机密的高持久性。

如果密钥保管库服务中的单个组件发生故障，则区域内的替代组件将接管任务，为请求提供服务，确保不会导致功能损失。你不需要采取任何措施即可触发此功能，这种机制会以透明的方式自动发生。

在整个 Azure 区域服务中断的情况下（这很少见），对该区域中的 Azure 密钥保管库发出的请求将自动路由（*故障转移*）到次要区域。当主要区域再次可用时，请求将路由回到（*故障回复*）主要区域。同样，不需要采取任何措施，因为这会自动发生。

应注意以下事项：

- 发生区域故障转移时，可能需要等待几分钟让服务故障转移。在此期间发出的请求在故障转移完成之前可能失败。
- 故障转移完成后，密钥保管库处于只读模式。在此模式下支持的请求包括：
  - 列出密钥保管库
  - 获取密钥保管库的属性
  - 列出机密
  - 获取机密
  - 列出密钥
  - 获取密钥的属性
  - 加密
  - 解密
  - 包装
  - 解包
  - 验证
  - 签名
  - 备份
- 故障回复之后，便支持所有请求类型（包括读取*和*写入请求）。

<!---HONumber=Mooncake_0220_2017-->
