<properties 
	pageTitle="在本地 VMWare 站点之间部署" 
	description="Azure Site Recovery 中的 InMage Scout 可以处理本地 VMWare 站点之间的复制、故障转移和恢复。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="rayne-wiselman" 
	manager="jwhit" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.date="05/29/2015" 
	wacn.date="10/03/2015"/>


# 在本地 VMWare 站点之间部署


##概述

Azure Site Recovery 中的 InMage Scout 在本地 VMWare 站点之间提供实时复制。Azure Site Recovery 服务订阅中已随附 InMage Scout。


## 先决条件

- **Azure 帐户** — 你需要一个 [Microsoft Azure](http://www.windowsazure.cn/) 帐户。你可以从[试用版](/pricing/1rmb-trial)开始。


##步骤 1：创建保管库并下载 InMage Scout

1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击“数据服务”>“恢复服务”>“Site Recovery 保管库”。
3. 单击“新建”>“快速创建”。
4. 在“名称”中，输入一个友好名称以标识此保管库。
5. 在“区域”中，为保管库选择地理区域。若要查看受支持的区域，请参阅 [Azure Site Recovery 价格详细信息](http://www.windowsazure.cn/home/features/site-recovery/#price)中的“地域可用性”。

<P>检查状态栏以确认保管库已成功创建。保管库将以“活动”状态列在主要的“恢复服务”页上。</P>

##步骤 2：配置保管库

1. 单击“创建保管库”。
2. 在“恢复服务”页中，单击保管库以打开“快速启动”页。
3. 在下拉列表中，选择“两个本地 VMWare 站点之间”。
4. 下载 InMage Scout。
5. 使用随产品下载的 InMage Scout 文档在两个 VMWare 站点之间设置复制。


##后续步骤

请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)上发布你的任何问题。

<!---HONumber=71-->