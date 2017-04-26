<properties
    pageTitle="使用 Azure 媒体服务管理编码的速度和并发 | Azure"
    description="本文简要概述了如何使用 Azure 媒体服务管理编码作业/任务的速度和并发。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="676313f8-a158-4e3a-a99b-2c29a341ecc9"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/10/2017"
    wacn.date="04/24/2017"
    ms.author="juliako"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="a31b0be1a3923fa7d559b5375ab2bd93717be3fe"
    ms.lasthandoff="04/14/2017" />

#  <a name="manage-speed-and-concurrency-of-your-encoding"></a>管理编码的速度和并发

本文简要概述了如何管理编码作业/任务的速度和并发。

## <a name="overview"></a>概述

在媒体服务中，**保留单位类型**决定了编码处理任务的处理速度。 可以在以下保留单位类型中进行选择：**S1**、**S2** 或 **S3**。 例如，与 **S1** 保留单位类型相比，使用 **S2** 保留单位类型时，同一编码作业运行速度更快。 [缩放编码单位](/documentation/articles/media-services-scale-media-processing-overview/)主题显示了一个表，该表可帮助你在不同的编码速度之间进行选择时做出决定。

除了指定保留单位类型之外，你还可以为帐户预配**保留单位**。 设置的保留单位数决定了给定帐户中可并发处理的媒体任务数。 例如，如果帐户具有 5 个保留单元，则只要有任务要处理，就可以同时运行 5 个媒体任务。 其余任务将排队等待，运行的任务完成后才选择它们以按顺序进行处理。 如果帐户未预配任何保留单位，则按顺序选择任务进行处理。 在这种情况下，完成一个任务和开始下一个任务之间的等待时间将取决于系统中资源的可用性。

有关介绍如何缩放编码单位的详细信息和示例，请参阅[此](/documentation/articles/media-services-scale-media-processing-overview/)主题。

## <a name="next-step"></a>后续步骤

[缩放编码单位](/documentation/articles/media-services-scale-media-processing-overview/)