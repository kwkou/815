<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="How to Use Media Services" authors="" solutions="" manager="" editor="" />

如何使用 Media Services
=======================

本指南说明如何开始使用 Azure Media Services 进行编程。本指南由几篇文章组成，并提供 Media Services 的技术概述、针对 Media Services 配置 Azure 帐户的步骤、开发设置指南，以及介绍如何完成典型编程任务的主题。演示的方案包括：上载资产、加密资产、为资产编码和交付资产。示例以 C\# 编写，并使用 Media Services SDK for .NET。有关 Azure Media Services 的详细信息，请参阅[后续步骤](#next-steps)主题。

也可以使用基于 OData 的 REST API 为 Media Services 编程。通过 .NET 语言或其他编程语言对 Media Services 执行 REST API 调用便可生成应用程序。有关使用 Media Services REST API 进行编程的完整文档系列，请参阅[使用 Azure Media Services REST API 生成应用程序](http://go.microsoft.com/fwlink/?linkid=252967)。

若要使用 Media Services REST API 或 Media Services SDK 开始编程，请先根据[针对 Media Services 设置 Azure 帐户](#setup-account)部分中所述，启用用于 Media Services 的 Azure 帐户。

[此处](http://msdn.microsoft.com/zh-cn/library/hh973613.aspx)提供了最新的 Media Services SDK 文档。

什么是 Media Services？
-----------------------

Azure Media Services 构成了一个可扩展的媒体平台，它在 Azure 中集成了 Microsoft Media Platform 和第三方媒体组件的精华。Media Services 在云中提供一个媒体管道，让行业合作伙伴扩展或取代组件技术。ISV 和媒体提供商可以使用 Media Services 来生成端到端媒体解决方案。本概述主题将介绍 Media Services 的一般体系结构和常见开发方案。

下图展示了 Media Services 的基本体系结构。

![Media Services 体系结构](./media/media-services-dotnet-how-to-use/wams-01.png)

### Media Services 功能支持

当前版本的 Media Services 提供以下功能集，用于在云中开发媒体应用程序。

-   **引入**。引入操作可将资产插入到系统，例如，先将其上载并加密，然后再将其放入 Azure 存储空间。Media Services 允许与合作伙伴组件相集成，以提供快速 UDP（用户数据报协议）上载解决方案。
-   **编码**。编码操作包括编码、变换和转换媒体资产。你可以使用 Azure 媒体编码器在云中运行编码任务。编码选项包括：
    -   使用 Azure 媒体编码器并操作一系列标准编解码器和格式，包括行业领先的 IIS 平滑流式处理和 MP4，以及将相关格式转换为 Apple HTTP 实时流。
    -   转换整个库或单个文件，并获取对输入和输出的全面控制权。
    -   受支持的文件类型、格式和编解码器众多（请参阅 [Media Services 支持的文件类型](http://msdn.microsoft.com/en-us/library/hh973634)）。
    -   支持的格式转换。Media Services 允许你将 ISO MP4 (.mp4) 转换为平滑流式处理文件格式 (PIFF 1.3)（.ismv；.isma）。还可以将平滑流式处理文件格式 (PIFF) 转换为 Apple HTTP 实时流（.msu8、.ts）。
-   **保护**。保护内容意味着对实时流内容或点播内容进行加密，以安全地进行传输、存储和交付。Media Services 提供 DRM 技术感知的解决方案来保护内容。当前支持的 DRM 技术包括 Microsoft PlayReady 和 MPEG 通用加密。将来会提供对其他 DRM 技术的支持。
-   **流式处理**。流式处理内容涉及到将实时内容或点播内容发送到客户端，或者让你从云中下载特定的媒体文件。Media Services 提供格式感知的解决方案来流式处理内容。Media Services 针对平滑流式处理、Apple HTTP 实时流和 MP4 格式提供流式来源支持。将来会提供对其他格式的支持。你也可以使用第三方 CDN（启用相应的选项即可扩展为支持数百万个用户）无缝交付流式处理内容。

### Media Services 开发方案

Media Services 支持下表中所述的多种常见媒体开发方案。

<table data-morhtml="true" border="2" cellspacing="0" cellpadding="5" style="border: 2px solid #000000;">
  <thead data-morhtml="true">
    <tr data-morhtml="true">
<th data-morhtml="true">方案</th>
<th data-morhtml="true">说明</th>
    </tr>
  </thead>
  <tbody data-morhtml="true">
    <tr data-morhtml="true">
<td data-morhtml="true">生成端到端工作流</td>
<td data-morhtml="true">完全在云中生成综合性的媒体工作流。从上载媒体到分发内容，Media Services 提供一系列组件，将这些组件相结合可以处理特定的应用工作流。当前功能包括上载、存储、编码、格式转换、内容保护和点播流交付。</td>
    </tr>
    <tr data-morhtml="true">
<td data-morhtml="true">生成混合工作流</td>
<td data-morhtml="true">你可以将 Media Services 与现有工具和过程相集成。例如，在现场为内容编码，再将其上载到 Media Services 以转码为多种格式，然后通过 Azure CDN 或第三方 CDN 交付内容。可以通过标准 REST API 单独调用 Media Services，以将它与外部应用程序和服务相集成。</td>
    </tr>
    <tr data-morhtml="true">
<td data-morhtml="true">为媒体播放器提供云支持</td>
<td data-morhtml="true">你可以跨多个设备（包括 iOS、Android 和 Windows 设备）与平台创建、管理和交付媒体。</td>
    </tr>
  </tbody>
</table>

### Media Services 客户端开发

使用 SDK 和播放器框架扩展 Media Services 解决方案的功能范围，以生成媒体客户端应用程序。开发人员可以使用这些客户端来生成 Media Services 应用程序，在各种设备和平台上提供引人入胜的用户体验。根据目标设备，你可以选择 Microsoft 和第三方合作伙伴提供的多种 SDK 和播放器框架。

下面提供了可用客户端 SDK 和播放器框架的列表。有关此处所列和其他已计划的 SDK 与播放器框架及其支持的功能的详细信息，请参阅 Media Services 论坛中的 [Media Services 客户端开发](http://social.msdn.microsoft.com/Forums/zh-cn/MediaServices/thread/e9092ec6-2dfc-44cb-adce-1dc935309d2a)。

#### Mac 和 PC 客户端支持

对于 PC 和 Mac，你可以使用 Microsoft Silverlight 或 Adobe Open Source Media Framework 来针对性地设计流式处理体验。

-   [适用于 Silverlight 的平滑流式处理客户端](http://www.microsoft.com/zh-cn/download/details.aspx?id=29940)
-   [Microsoft Media Platform：适用于 Silverlight 的播放器框架](http://smf.codeplex.com/documentation)
-   [适用于 OSMF 2.0 的平滑流式处理插件](http://go.microsoft.com/fwlink/?LinkId=275022)。有关如何使用此插件的信息，请参阅[如何使用适用于 Adobe Open Source Media Framework 的平滑流式处理插件](http://go.microsoft.com/fwlink/?LinkId=275034)。

#### Windows 8 应用程序

对于 Windows 8，你可以使用支持的任一开发语言和构造（例如 HTML、Javascript、XAML、C\# 和 C+）生成 Windows 应用商店应用程序。

-   [适用于 Windows 8 的平滑流式处理客户端 SDK](http://go.microsoft.com/fwlink/?LinkID=246146)。有关如何使用此 SDK 创建 Windows 应用商店应用程序的详细信息，请参阅[如何生成平滑流式处理 Windows 应用商店应用程序](http://go.microsoft.com/fwlink/?LinkId=271647)。有关如何使用 HTML5 语言创建平滑流式处理播放器的信息，请参阅[演练：生成你的第一个 HTML5 平滑流式处理播放器](http://msdn.microsoft.com/en-us/library/jj573656(v=vs.90).aspx)。

-   [Microsoft Media Platform：适用于 Windows 8 Windows 应用商店应用程序的播放器框架](http://playerframework.codeplex.com/wikipage?title=Player%20Framework%20for%20Windows%208%20Metro%20Style%20Apps&referringTitle=Home)

#### Xbox

Xbox 支持可使用平滑流式处理内容的 Xbox LIVE 应用程序。Xbox LIVE 应用程序开发工具包 (ADK) 包含：

-   适用于 Xbox LIVE ADK 的平滑流式处理客户端
-   Microsoft Media Platform：适用于 Xbox LIVE ADK 的播放器框架

#### 嵌入式设备或专用设备

联网的电视机、机顶盒、蓝光播放机、智能电视机顶盒等设备，以及带有自定义应用程序开发框架和自定义媒体管道的移动设备。Microsoft 提供以下可购买许可的移植工具包，并允许合作伙伴为平台移植平滑流式处理播放功能。

-   [平滑流式处理客户端移植工具包](http://www.microsoft.com/zh-cn/mediaplatform/sspk.aspx)
-   [Microsoft PlayReady 设备移植工具包](http://www.microsoft.com/PlayReady/Licensing/device_technology.mspx)

#### Windows Phone

Microsoft 提供可用于生成 Windows Phone 版高级视频应用程序的 SDK。

-   [适用于 Silverlight 的平滑流式处理客户端](http://www.microsoft.com/zh-cn/download/details.aspx?id=29940)
-   [Microsoft Media Platform：适用于 Silverlight 的播放器框架](http://smf.codeplex.com/documentation)

#### iOS 设备

对于 iPhone、iPod 和 iPad 等 iOS 设备，Microsoft 随附了一个 SDK，让你针对这些平台生成应用程序，以交付高级视频内容：适用于具有 PlayReady 功能的 iOS 设备的平滑流式处理 SDK。该 SDK 仅向许可接受方提供，有关详细信息，请[向 Microsoft 发送电子邮件](mailto:askdrm@microsoft.com)。有关 iOS 开发的信息，请参阅 [iOS 开发人员中心](https://developer.apple.com/devcenter/ios/index.action)。

#### Android 设备

有许多 Microsoft 合作伙伴针对 Android 平台提供 SDK，用于在 Android 设备上添加播放平滑流式处理内容的功能。有关这些合作伙伴的更多详细信息，请[向 Microsoft 发送电子邮件](mailto:sspkinfo@microsoft.com?subject=Partner%20SDKs%20for%20Android%20Devices)。

后续步骤
--------

了解 Media Services 的概况后，请转到[针对 Media Services 设置计算机](http://go.microsoft.com/fwlink/?LinkId=301751)主题。

