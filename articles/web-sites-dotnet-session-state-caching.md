<properties linkid="video-center-index" urlDisplayName="index" pageTitle="Use ASP.NET session state with Azure Web Sites" metaKeywords="azure cache service session state" description="Learn how to use the Azure Cache Service to support ASP.NET session state caching." metaCanonical="" services="cache" documentationCenter=".NET" title="How to Use ASP.NET Session State with Azure Web Sites" authors="jroth" solutions="" manager="" editor="mollybos" />

# 如何将 ASP.NET 会话状态用于 Azure 网站

本主题介绍如何使用 Azure Cache Service（预览版）来支持 ASP.NET Web 窗体的 ASP.NET 会话状态缓存。

没有外部提供程序，会话状态以进程内的形式存储在托管网站的 Web 服务器上。对于 Azure 网站，进程内会话状态有两个问题。首先，对于具有多个实例的网站，在一个实例上存储的会话状态无法被其他实例访问。因为用户请求可以路由到任何实例，所以，不确保会话信息就在那里。第二，配置中的任何更改都可能导致网站运行在完全不同的服务器上。

缓存服务（预览版）提供了位于网站外部的分布式缓存服务。这解决了与进程中会话状态有关的问题。有关如何使用会话状态的详细信息，请参阅 [ASP.NET 会话状态概述][ASP.NET 会话状态概述]。

将缓存服务（预览版）用于会话状态缓存的基本步骤包括：

-   [创建缓存。][创建缓存。]
-   [配置 ASP.NET 项目以使用 Azure 缓存。][配置 ASP.NET 项目以使用 Azure 缓存。]
-   [修改 web.config 文件。][修改 web.config 文件。]
-   [使用会话对象存储和检索缓存项。][使用会话对象存储和检索缓存项。]

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

除了为缓存生成程序集引用，NuGet 包还在 web.config 文件中添加存根项。要将缓存用于会话状态，必须对 web.config 文件进行一些修改。

1.  打开 ASP.NET 项目的 **web.config** 文件。

2.  找到现有的 **sessionState** 元素，将它注释掉。

3.  然后对 Azure Caching NuGet 包添加的 **sessionState** 元素取消注释。最终结果应类似于以下屏幕快照。

    [SessionStateConfig][SessionStateConfig]

4.  接下来，找到 **dataCacheClients** 节。对 **securityProperties** 子元素取消注释。

    ![缓存配置][缓存配置]

5.  在 **autoDiscover** 元素中，将 **identifier** 特性设置为你的缓存的终结点 URL。要查找你的终结点 URL，请在 Azure 管理门户中转到缓存属性。在“仪表板”选项卡上，复制“速览”节中的“终结点 URL”值。

    ![端点 URL][端点 URL]

6.  在 **messageSecurity** 元素中，将 **authorizationInfo** 特性设置为你的缓存的访问密钥。要查找访问密钥，请在 Azure 管理门户中选择你的缓存。然后单击底部栏中的“管理密钥”图标。单击“主访问密钥”文本框旁边的复制按钮。

    ![管理密钥][管理密钥]

## <span id="usesessionobject"></span></a>在代码中使用 Session 对象

最后一步是开始在 ASP.NET 代码中使用 Session 对象。通过使用 **Session.Add** 方法将对象添加到会话状态。此方法使用键值对在会话状态缓存中存储项。

    string strValue = "yourvalue";
    Session.Add("yourkey", strValue);

下面的代码从会话状态中检索该值。

    object objValue = Session["yourkey"];
    if (objValue != null)
       strValue = (string)obj;  

有关如何使用 ASP.NET 会话状态的详细信息，请参阅 [ASP.NET 会话状态概述][ASP.NET 会话状态概述]。

  [ASP.NET 会话状态概述]: http://msdn.microsoft.com/zh-cn/library/ms178581.aspx
  [创建缓存。]: #createcache
  [配置 ASP.NET 项目以使用 Azure 缓存。]: #configureproject
  [修改 web.config 文件。]: #configurewebconfig
  [使用会话对象存储和检索缓存项。]: #usesessionobject
  [“新建”图标]: ./media/web-sites-dotnet-session-state-caching/CacheScreenshot_NewButton.png
  [“新建缓存”对话框]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CreateOptions.png
  [“缓存”图标]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CacheIcon.png
  [安装了最新版本的]: http://www.windowsazure.cn/zh-cn/downloads/?sdk=net
  [“NuGet”对话框]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_NuGet.png
  [缓存配置]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CacheConfig.png
  [端点 URL]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_EndpointURL.png
  [管理密钥]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_ManageAccessKeys.png
