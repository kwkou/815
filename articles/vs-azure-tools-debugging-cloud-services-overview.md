<properties
    pageTitle="调试 Azure 云服务的选项 | Azure"
    description="调试 Azure 云服务"
    services="visual-studio-online"
    documentationcenter="n/a"
    author="TomArcher"
    manager="douge"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="80755da7-8350-4f5c-97ce-2962beabb36d"
    ms.service="visual-studio-online"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="multiple"
    ms.workload="na"
    ms.date="03/18/2017"
    wacn.date="04/17/2017"
    ms.author="tarcher"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="aba6791f49c6827a78bf6d91a88417d7bae9000a"
    ms.lasthandoff="04/07/2017" />

# <a name="learn-the-various-ways-to-debug-an-azure-cloud-service"></a>了解调试 Azure 云服务的各种方法
本文提供了调试 Azure 云服务的各种方法的链接。 

## <a name="debugging-a-cloud-service-in-visual-studio"></a>在 Visual Studio 中调试云服务
使用 Azure 计算模拟器调试本地计算机上的云服务可为您节省时间和金钱。 部署某个服务之前在本地对其进行调试可以提高可靠性和性能，且不会产生计算时间的相关费用。 但是，仅当你在 Azure 中运行云服务时，某些错误才可能会出现。 仅在 Azure 中运行云服务时才会发生的错误可以通过在发布服务时启用远程调试，然后将调试器附加到角色实例来进行调试。 有关详细信息，请参阅 [Debug your cloud service on your local computer](/documentation/articles/vs-azure-tools-debug-cloud-services-virtual-machines/#debug-your-cloud-service-on-your-local-computer/)（在本地计算机上调试云服务）。

## <a name="using-azure-diagnostics"></a>使用 Azure 诊断 
可使用 Azure 诊断以记录在角色内运行的代码的详细信息，角色是否在开发环境或 Azure 中运行。 有关详细信息，请参阅[在 Azure 云服务中启用 Azure 诊断](/documentation/articles/cloud-services-dotnet-diagnostics/)。

## <a name="using-intellitrace"></a>使用 IntelliTrace 
如果使用 Visual Studio Enterprise 来编写以 .NET Framework 4.5 为目标的角色，从 Visual Studio 部署 Azure 云服务时，可以启用 IntelliTrace。 IntelliTrace 提供一个日志，可将该日志与 Visual Studio 一起使用以调试应用程序，就如同应用程序在 Azure 中运行一样。 有关详细信息，请参阅 [Debugging a published cloud service with IntelliTrace and Visual Studio](http://go.microsoft.com/fwlink/p/?LinkId=623016)（使用 IntelliTrace 和 Visual Studio 调试已发布的云服务）。

## <a name="remote-debugging"></a>远程调试 
从 Visual Studio 部署云服务时，可以在云服务上启用远程调试。 如果选择为部署的项目启用远程调试，将在运行每个角色实例的虚拟机上安装远程调试服务。 这些服务（如 `msvsmon.exe`）不影响性能，也不额外收费。 有关详细信息，请参阅 [Debug a cloud service in Azure](/documentation/articles/vs-azure-tools-debug-cloud-services-virtual-machines/#debug-a-cloud-service-in-azure/)（调试 Azure 中的云服务）。

## <a name="next-steps"></a>后续步骤
- [在 Visual Studio 中调试 Azure 云服务或 VM](/documentation/articles/vs-azure-tools-debug-cloud-services-virtual-machines/) - 了解如何调试 Azure 云服务的详细信息。

<!-- Update_Description: wording update -->