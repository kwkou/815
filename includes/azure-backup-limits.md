<properties
   pageTitle="Azure 备份限制表"
   description="描述 Azure 备份的系统限制。"
   services="backup"
   documentationCenter="NA"
   authors="Jim-Parker"
   manager="jwhit"
   editor="" />
<tags  ms.service="backup"  ms.date="07/01/2015 "wacn.date="10/03/2015" />


以下限制适用于 Azure 备份。

| 限制标识符 | 默认限制 |
|---|---|
|可针对每个保管库注册的服务器/计算机数量|50|
|存储在 Azure 保管库存储中的数据的数据源大小|最大 1.65 TB<sup>1</sup>|
|在每个 Azure 订阅中可以创建的备份保管库数量|25|
|每天计划备份的次数|对于 Windows Server/客户端为每天 3 个 <br/> 对于 SCDPM 为每天 2 个|
|可创建的恢复点数量|366<sup>2</sup>|
|将数据磁盘附加到 Azure 虚拟机进行备份|5|

- <sup>1</sup>1.65TB 限制不适用于 IaaS VM 备份。
- <sup>2</sup>你可以使用任意排列来实现此数字小于 366。

<!---HONumber=71-->