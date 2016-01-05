<properties
   pageTitle="Azure 备份限制表"
   description="描述 Azure 备份的系统限制。"
   services="backup"
   documentationCenter="NA"
   authors="Jim-Parker"
   manager="jwhit"
   editor="" />
<tags
   ms.service="backup"
   ms.date="10/05/2015"
   wacn.date="12/31/2015" />


以下限制适用于 Azure 备份。

| 限制标识符 | 默认限制 |
|---|---|
|可针对每个保管库注册的服务器/计算机数量|对于 Windows Server/客户端/SCDPM 为 50 个 <br/> 对于 IaaS VM 为 200 个|
|存储在 Azure 保管库存储中的数据的数据源大小|最大 54400 GB<sup>1</sup>|
|在每个 Azure 订阅中可以创建的备份保管库数量|25|
|每天计划备份的次数|对于 Windows Server/客户端为每天 3 次 <br/> 对于 SCDPM 为每天 2 次 <br/> 对于 IaaS VM 为每天 1 次|
|将数据磁盘附加到 Azure 虚拟机进行备份|16|

- <sup>1</sup>54400 GB 限制不适用于 IaaS VM 备份。

<!---HONumber=Mooncake_1207_2015-->