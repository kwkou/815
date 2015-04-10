<properties urlDisplayName="移动分析入门" pageTitle="移动分析入门 | 移动开发人员中心" metaKeywords="" description="移动分析入门。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Get Started with Mobile Analytics" authors="mahender" manager="dwrede"/>

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-multiple" ms.devlang="multiple" ms.topic="article" ms.date="10/10/2014" ms.author="mahender" />

# 移动分析入门 (Capptain)

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-mobile-analytics" title="iOS" class="current">iOS</a>
</div>

在本教程中，你将使用 [Capptain] 将移动分析功能添加到你的应用程序。使用移动分析，可以确定用户如何与你的应用程序进行交互以及哪些部分驱动大多数活动。若要开始获取这些数据，你将使用 Capptain SDK 来装备你的应用程序。


>[AZURE.NOTE] Microsoft 拥有的 Capptain.com 为 Azure 移动服务标准层客户的每月最多 100,000 个活动用户免费提供了移动应用程序的分析功能。若要利用此服务，请在 mobileservices@microsoft.com 上与我们联系，以便提供进一步的说明。下面的教程总结了 Capptain.com 特性和功能，并提供了有关如何使用这些特性和功能的说明。


本教程将指导你完成以下基本步骤：

1. [初始化 Capptain SDK]
2. [重载 UIViewController]

本教程需要以下项：

* [Capptain] 帐户
* [移动服务标准层]应用程序

## <a name="initialize"></a>初始化 Capptain SDK

1. 导航到 Capptain 中注册的你的应用程序的"应用程序详细信息"页。选择 SDK 选项卡并下载程序包。

2. 在 XCode 中，通过右键单击你的项目并选择"将文件添加到..."将 Capptain SDK 添加到你的项目选择 CapptainSDK 文件夹。

3. 选择你的项目。在"生成阶段"选项卡下，选择"将二进制文件链接到库"并添加以下框架：
    * AdSupport.framework - 将链接设为可选
    * SystemConfiguration.framework
    * CoreTelephony.framework
    * CFNetwork.framework
    * CoreLocation.framework
    * libxml2.dylib

    >[AZURE.NOTE] 可以删除 AdSupport 框架。Capptain 需要此框架来收集 IDFA。但是，可以禁用 IDFA 收集以符合有关此 ID 的 Apple 策略。

4. 在应用程序委托实现文件中，导入 Capptain 代理。


        #import "CapptainAgent.h"


5. 在 `applicationDidFinishLaunching:` 或 `application:didFinishLaunchingWithOptions:` 内，通过为 `registerapp:identifiedby:` 传递应用程序 ID 和 SDK 密钥初始化 Capptain 代理。

         - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
        {
          [...]
          [CapptainAgent registerApp:@"YOUR_APPID" identifiedBy:@"YOUR_SDK_KEY"];
          [...]
        }

## <a name="instrument"></a>重载 UIViewController

1. 在你的项目中找到 `UIViewController` 的每个子级并确保每个子级改为从 `CapptainViewController` 继承。

        #import <UIKit/UIKit.h>
        #import "CapptainViewController.h"

        // formerly @interface Tab1ViewController : UIViewController<UITextFieldDelegate>
        @interface Tab1ViewController : CapptainViewController<UITextFieldDelegate> {
          UITextField* myTextField1;
          UITextField* myTextField2;
        }

        @property (nonatomic, retain) IBOutlet UITextField* myTextField1;
        @property (nonatomic, retain) IBOutlet UITextField* myTextField2;

2. 在你的项目中找到 `UITableViewController` 的每个子级并确保每个子级改为从 `CapptainTableViewController` 继承。

    你的应用程序现已配置为将分析数据发送到 Capptain。

## 后续步骤
在 [http://www.capptain.com](http://www.capptain.com) 了解有关 Capptain 可为你的应用程序做什么的详细信息

<!-- Anchors. -->
[初始化 Capptain SDK]: #initialize
[重载 UIViewController]: #instrument


<!-- URLs. -->
[Capptain]: http://www.capptain.com
[移动服务标准层]: /zh-cn/pricing/details/mobile-services/
