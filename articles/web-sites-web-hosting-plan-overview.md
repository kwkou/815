<properties pageTitle="What is a web hosting plan?" description="Web hosting plan overview" title="What is a web hosting plan?" services="web-sites" authors="adamab" />

# 什么是 Web 宿主计划？

Web 宿主计划表示一组可在你的网站间共享的功能和容量。Web 宿主计划支持数种定价级别（例如，免费、共享、基本和标准），其中每种级别都有其自己的功能。同一订阅、资源组和地理位置中的网站可以共享一个 Web 宿主计划。

## Web 宿主计划中的功能

每个定价级别（例如，免费、共享、基本和标准）都有其自己的配套功能。[转到此处][转到此处]以了解最新功能和定价信息。

下面是一些有关 Web 宿主计划和功能的有用提示：

-   你可以随时更改 Web 宿主计划的定价级别，不会造成任何停机。
-   同一订阅、位置和资源组中的网站可以一起共享一个 Web 宿主计划。
-   通过确定 Web 宿主计划目标，可以实现自动伸缩这样的功能。如果你想自动伸缩单个网站，则应将一个 Web 宿主计划专用于该网站。

## Web 宿主计划和容量

免费和共享级别的 Web 宿主计划为网站提供共享的基础结构，这意味着你的网站可与其他客户的网站共享资源。

基本和标准级别的 Web 宿主计划为网站提供专用的基础结构，这意味着只有你选择与此计划关联的一个或多个网站才会在那些资源上运行。在这个级别，你可以配置 Web 宿主计划来使用一个或多个虚拟机实例。支持的实例包括小型、中型和大型。这些虚拟机是代表你托管的，这意味着你不需要再为操作系统更新或任何类似的事情操心。

有关共享（预览版）级别的注意事项。对于除“共享”以外的其他所有级别，你只需根据级别以及选择的容量按一个价格为 Web 宿主计划支付费用，使用该计划的各个网站都不需要再支付其他费用。共享的 Web 宿主计划则有所不同。由于共享了基础结构，该计划中的每个网站都会被单独收取费用。

### Web 宿主计划和全新 Azure 预览版门户

利用全新 Azure 预览版门户，你可以将现有的或新的网站与 Web 宿主计划关联起来。事实上，所有现有的网站均已根据其订阅、地理位置和当前定价级别自动分配到了 Web 宿主计划。

在创建新网站时，门户会询问你要将该网站与哪个 Web 宿主计划关联。你可以创建新 Web 宿主计划，或选择现有计划。若要使用现有计划，你的新网站必须与现有计划存在于同一订阅、地理位置和资源组中。在创建新的空网站时，Azure 会默认使用最近用过的 Web 宿主计划来帮助你充分利用已经创建的计划。在创建带有数据库的网站时，将无法使用重复利用现有计划的选项。

通过使用左边菜单栏上的“浏览”按钮，然后单击屏幕上出现的活动窗格右上角的“所有”，你可以看到你所有订阅的所有 Web 宿主计划。

![][]
![][1]

你也可以通过查看网站叶片顶部出现的资源组的图形表现方式，了解到每一个网站关联了哪个 Web 宿主计划。

![][2]

单击计划将启动一个叶片，让你管理你的 Web 宿主计划。[详细了解如何管理 Web 宿主计划][详细了解如何管理 Web 宿主计划]。

![][3]

### 后续步骤

若要开始使用 Azure，请参阅 [Microsoft Azure 免费试用版][Microsoft Azure 免费试用版]。

<!-- Images. -->

  [转到此处]: http://azure.microsoft.com/zh-cn/pricing/details/web-sites/
  []: ./media/web-sites-web-hosting-plan-overview/browse-everything.png
  [1]: ./media/web-sites-web-hosting-plan-overview/browse-web-hosting-plans.png
  [2]: ./media/web-sites-web-hosting-plan-overview/web-hosting-plan-resource-map.png
  [详细了解如何管理 Web 宿主计划]: http://azure.microsoft.com/zh-cn/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/
  [3]: ./media/web-sites-web-hosting-plan-overview/web-hosting-plan-blade.png
  [Microsoft Azure 免费试用版]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
