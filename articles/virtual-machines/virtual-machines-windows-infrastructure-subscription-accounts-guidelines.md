<properties
	pageTitle="订阅和帐户准则 | Azure"
	description="了解 Azure 中订阅和帐户的关键设计和实施准则。"
	documentationCenter=""
	services="virtual-machines-windows"
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/17/2017"
	wacn.date="04/27/2017"
	ms.author="iainfou"/>

# 订阅和帐户准则

[AZURE.INCLUDE [virtual-machines-windows-infrastructure-guidelines-intro](../../includes/virtual-machines-windows-infrastructure-guidelines-intro.md)]

本文重点介绍如何随环境和用户群的增长实行订阅和帐户管理。


## 订阅和帐户的实施准则

决策：

- 需要使用怎样的订阅和帐户组合来托管 IT 工作负荷或基础结构？
- 如何细分层次结构以适应组织的需要？

任务：

- 定义逻辑组织层次结构，以便从订阅级别进行管理。
- 根据此逻辑层次结构，定义所需帐户和每个帐户下的订阅。
- 按照命名约定来创建订阅和帐户集。


## 订阅和帐户

若要使用 Azure，需要一个或多个 Azure 订阅。虚拟机 (VM) 或虚拟网络等资源存在于这些订阅中。

- 企业客户通常具有企业许可登记表，该表是层次结构中的最顶层资源并与一个或多个帐户相关联。
- 对于没有企业许可登记表的使用者和客户，最顶层资源是帐户。
- 订阅关联到帐户，并且每个帐户可以有一个或多个订阅。Azure 在订阅级别记录计费信息。

由于帐户/订阅关系上只能存在两个层次结构级别，因此根据计费要求调整帐户和订阅的命名约定至关重要。例如，一家全球性公司在使用 Azure 时，可以选择每个区域建立一个帐户，并在区域级别管理订阅：

![](./media/virtual-machines-common-infrastructure-service-guidelines/sub01.png)

例如，可以使用以下结构：

![](./media/virtual-machines-common-infrastructure-service-guidelines/sub02.png)  


如果某一区域决定将多个订阅关联到一个特定组，则命名约定应引入相应方法，对额外的帐户或订阅名称数据进行编码。此组织允许传递计费数据，以在计费报告期间生成新的层次结构级别：

![](./media/virtual-machines-common-infrastructure-service-guidelines/sub03.png)  


该组织可以如下所示：

![](./media/virtual-machines-common-infrastructure-service-guidelines/sub04.png)  


我们通过可下载的文件为企业协议中的单个帐户或所有帐户提供详细的计费信息。


## <a name="next-steps"></a>后续步骤

[AZURE.INCLUDE [virtual-machines-windows-infrastructure-guidelines-next-steps](../../includes/virtual-machines-windows-infrastructure-guidelines-next-steps.md)]

<!---HONumber=Mooncake_Quality_Review_1215_2016-->