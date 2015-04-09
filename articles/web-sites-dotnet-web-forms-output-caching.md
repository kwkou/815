<properties linkid="video-center-detail" urlDisplayName="details" pageTitle="Video Center Details" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="How to Use ASP.NET Web Forms Output Caching with Azure Web Sites" authors="jroth" solutions="" manager="" editor="" />

# 如何将 ASP.NET Web 窗体输出缓存用于 Azure 网站

本主题介绍如何使用 Azure Cache Service（预览版）来支持 ASP.NET Web 窗体的 ASP.NET 页面输出缓存。页输出缓存是一种优化，它直接为特定时间段返回缓存的呈现页。它在比通常的更改更频繁访问某一页时会很有用。请注意，ASP.NET MVC 应用程序不支持页面输出缓存，这一点很重要。

缓存服务（预览版）提供了位于网站外部的分布式缓存服务。它可以让网站的所有实例都能访问页面的同一缓存呈现。

将缓存服务（预览版）用于页输出缓存的基本步骤包括：

-   [创建缓存。][创建缓存。]
-   [配置 ASP.NET 项目以使用 Azure 缓存。][配置 ASP.NET 项目以使用 Azure 缓存。]
-   [修改 web.config 文件。][修改 web.config 文件。]
-   [使用输出缓存临时返回某一页的已缓存版本。][使用输出缓存临时返回某一页的已缓存版本。]

## <span id="createcache"></span></a>创建缓存

1.  在 Azure 管理门户的底部，单击“新建”图标。

    ![“新建”图标][“新建”图标]

2.  选择**“数据服务”**、**“缓存”**，然后单击**“快速创建”**。

    ![“新建缓存”对话框][“新建缓存”对话框]

3.  在“终结点”文本框中为缓存键入唯一名称。然后为其他缓存属性选择相应的值，并单击“创建新缓存”。

4.  在管理门户中选择“缓存”图标以查看你所有的缓存服务终结点。

    ![“缓存”图标][“缓存”图标]

5.  然后你可以选择缓存服务终结点之一来查看其属性。下面各部分将使用“仪表板”选项卡中的设置为 ASP.NET 项目配置缓存。

## <span id="configureproject"></span></a>配置 ASP.NET 项目

1.  首先，确保你已经[安装了最新版本的][安装了最新版本的] **Azure SDK for .NET**。

2.  在 Visual Studio 中的“解决方案资源管理器”中右键单击 ASP.NET 项目，然后选择“管理 NuGet 包”。（如果你使用的是 WebMatrix，则改为单击工具栏上的 **NuGet** 按钮。

3.  在“联机搜索”编辑框中键入 **WindowsAzure.Caching**。

    ![“NuGet”对话框][“NuGet”对话框]

4.  选择“Azure 缓存”包，然后单击“安装”按钮。

## <span id="configurewebconfig"></span></a>修改 Web.Config 文件

除了为缓存生成程序集引用，NuGet 包还在 web.config 文件中添加存根项。要将缓存用于 ASP.NET 页面输出缓存，必须对 web.config 文件进行一些修改。

1.  打开 ASP.NET 项目的 **web.config** 文件。

2.  如果你有现有的 **caching** 和 **outputCache** 元素，请将它们注释掉（或者移除）。

3.  然后对 Azure Caching NuGet 包添加的 **caching** 元素取消注释。最终结果应类似于以下屏幕快照。

    ![输出配置][输出配置]

4.  接下来，找到 **dataCacheClients** 节。对 **securityProperties** 子元素取消注释。

    ![缓存配置][缓存配置]

5.  在 **autoDiscover** 元素中，将 **identifier** 特性设置为你的缓存的终结点 URL。要查找你的终结点 URL，请在 Azure 管理门户中转到缓存属性。在“仪表板”选项卡上，复制“速览”节中的“终结点 URL”值。

    ![端点 URL][端点 URL]

6.  在 **messageSecurity** 元素中，将 **authorizationInfo** 特性设置为你的缓存的访问密钥。要查找访问密钥，请在 Azure 管理门户中选择你的缓存。然后单击底部栏中的“管理密钥”图标。单击“主访问密钥”文本框旁边的复制按钮。

    ![管理密钥][管理密钥]

## <span id="useoutputcaching"></span></a>使用输出缓存

最后一步是配置你的 ASP.NET Web 窗体应用程序中的页面以使用输出缓存。可通过将 **OutputCache** 属性添加到 .ASPX 源的开头来完成该步骤。例如：

    <%@ OutputCache Duration="45" VaryByParam="*" %>

前面的示例缓存该页面四十五秒。因为 **VaryByParam** 设置为“\*”，所以，甚至在传递不同查询参数时，该缓存的页输出也不更改。但下面的示例为“UserId”参数的每个值都缓存该页面的一个不同版本：

    <%@ OutputCache Duration="45" VaryByParam="UserId" %>   

  [创建缓存。]: #createcache
  [配置 ASP.NET 项目以使用 Azure 缓存。]: #configureproject
  [修改 web.config 文件。]: #configurewebconfig
  [使用输出缓存临时返回某一页的已缓存版本。]: #useoutputcaching
  [“新建”图标]: ./media/web-sites-web-forms-output-caching/CacheScreenshot_NewButton.PNG
  [“新建缓存”对话框]: ./media/web-sites-web-forms-output-caching/CachingScreenshot_CreateOptions.PNG
  [“缓存”图标]: ./media/web-sites-web-forms-output-caching/CachingScreenshot_CacheIcon.PNG
  [安装了最新版本的]: http://www.windowsazure.cn/zh-cn/downloads/?sdk=net
  [“NuGet”对话框]: ./media/web-sites-web-forms-output-caching/CachingScreenshot_NuGet.PNG
  [输出配置]: ./media/web-sites-web-forms-output-caching/CachingScreenshot_OC_WebConfig.PNG
  [缓存配置]: ./media/web-sites-web-forms-output-caching/CachingScreenshot_CacheConfig.PNG
  [端点 URL]: ./media/web-sites-web-forms-output-caching/CachingScreenshot_EndpointURL.PNG
  [管理密钥]: ./media/web-sites-web-forms-output-caching/CachingScreenshot_ManageAccessKeys.PNG
