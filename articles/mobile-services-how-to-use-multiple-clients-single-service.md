<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="Considerations for supporting multiple clients from a single mobile service" authors="krisragh" solutions="" manager="" editor="" />

# 有关从单个移动服务支持多个客户端的注意事项

利用 Azure 移动服务支持移动应用程序开发的主要优势之一在于，能够使用单个后端服务来支持多个客户端平台上的应用程序。移动服务为所有主要设备平台提供了本机客户端库。有关详细信息，请参阅[教程和资源][]。

尽管移动服务可让你使用单个后端服务轻松地在多个客户端平台之间迁移本机应用程序，但还有一些注意事项需要你的关注。本主题提供有关如何在所有客户端平台之间运行推送通知的指导。另外，还介绍了如何解决在 Windows 应用商店应用程序和 Windows Phone 应用程序中使用 Microsoft 帐户进行客户端定向的单一登录时遇到的问题。最后，本主题还介绍了有关在 Visual Studio 项目中重用代码的一些最佳实践。

## 推送通知

本部分介绍从移动服务向多个平台上的客户端应用程序发送推送通知的两种方法。

### 利用通知中心

Azure 通知中心是一个 Azure 服务和可伸缩的解决方案，用于将推送通知从移动服务（或任何后端）发送到所有主要设备平台上的客户端应用程序。有关详细信息，请参阅 [Azure 通知中心][]。

通知中心提供了一致且统一的基础结构让你创建和管理设备注册，以及将推送通知发送到所有主要设备平台上运行的设备。有关详细信息，请参阅[通知中心入门][]。通知中心支持平台特定的模板注册，这样，你便可以使用单个 API 调用向任何已注册平台上运行的应用程序发送通知。有关详细信息，请参阅[向用户发送跨平台通知][]。

通知中心是从移动服务向多个客户端平台发送通知的推荐解决方案。

### 使用通用注册表和平台标识符

如果你选择不使用通知中心，仍可以通过定义一个通用设备注册机制，来支持从移动服务向多个客户端发送推送通知。使用单个表来存储受支持平台的推送通知服务所用的设备特定信息。“"推送通知入门"”教程（[Windows 应用商店 C\#][]/[Windows 应用商店 JavaScript][]/[Windows Phone][]/[iOS][]/[Android][]）使用 "Registrations" 表，并在名为 *handle* 的列中存储通道 URI (Windows)、设备标记 (iOS) 或句柄 (Android)。

<div class="dev-callout"><b>说明</b>

<p>当你使用 Visual Studio 2013 中的“添加推送通知”向导将推送通知添加到 Windows 应用商店应用程序时，该向导将自动创建名为 <b>Channel</b> 的表来存储通道 URI。“Visual Studio 2012 中的推送通知入门”教程（<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet-vs2012">Windows 应用商店 C\#</a>/<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-js-vs2012">Windows 应用商店 JavaScript</a>）说明了如何使用 <b>Registrations</b> 表启用推送通知。</p>
</div>

若要在这单个注册表中支持多个客户端，请在表中包含 *platform* 列，在该列中，此字段设置为在注册期间插入行的客户端的平台。例如，以下 "Registrations" 类定义（摘自某个 Windows 应用商店 C\# 或 Windows Phone 应用程序）会将客户端 *ChannelUri* 字段映射到 Registrations 表中的 *handle* 列：

        public class Registrations
        {
    [JsonProperty(PropertyName = "platform")]           
    public string Platform { 
    get
                {
    return "windowsstore";
    // return "windowsphone";
                }
    set {}
            }
            
    public string Id { get; set; }
        
    [JsonProperty(PropertyName = "handle")]
    public string ChannelUri { get; set; }
        }

请注意，在此 Windows 设备上，*Platform* 字段始终返回 `windowsstore`（或 `windowsphone`）。如果启用了动态架构（默认设置），移动服务将在 Registrations 表中添加一个 platform 列（如果尚不存在）。有关详细信息，请参阅[动态架构][]。

在发送通知的服务器端脚本中，使用该 platform 字段来确定要对[推送对象][]调用的平台特定函数。以下脚本是摘自“"推送通知入门"”教程（[Windows 应用商店 C\#][]/[Windows 应用商店 JavaScript][]/[Windows Phone][]/[iOS][]/[Android][]）的、用于启用多个客户端平台的服务器脚本的修改版：

        function insert(item, user, request) {
    request.execute({
    success:function() {
    request.respond();
    sendNotifications();
                }
            });
        
    function sendNotifications() {
    var registrationsTable = tables.getTable('Registrations');
    registrationsTable.read({
    success:function(registrations) {
    registrations.forEach(function(registration) {
    if (registration.platform === 'winstore') {
    push.wns.sendToastText04(registration.handle, {
    text1:item.text
                                }, {
    success:pushCompleted
                                });
    } else if (registration.platform === 'winphone') {
    push.mpns.sendFlipTile(registration.handle, {
    title:item.text
                                }, {
    success:pushCompleted
                                });
    } else if (registration.platform === 'ios') {
    push.apns.send(registration.handle, {
    alert:"Toast:" + item.text,
    payload: {
    inAppMessage:item.text
                                    }
                                });
    } else if (registration.platform === 'android') {
    push.gcm.send(registration.handle, item.text, {
    success:pushCompleted
                                });
    } else {
    console.error("Unknown push notification platform");
                            }
                        });
                    }
                });
            }
        
    function pushCompleted(pushResponse) {
    console.log("Sent push:", pushResponse);
            }
        }

## Windows 应用程序注册

若要同时在 Windows 应用商店应用程序和 Windows Phone 应用程序中使用 Microsoft 帐户进行单一登录客户端身份验证，必须先在 Windows 应用商店仪表板上注册 Windows 应用商店应用程序。这是因为，在为 Windows Phone 创建 Live Connect 注册后，无法为 Windows 应用商店创建这种注册。有关如何执行此操作的详细信息，请阅读主题“"使用 Live Connect 单一登录对 Windows 应用商店应用程序进行身份验证"”（[Windows 应用商店][]/[Windows Phone][3]）。

## 在 Visual Studio 项目中重用代码的最佳实践

使用可移植类库可以编写和生成能够在多个平台（例如 Windows 应用商店和 Windows Phone）上工作的托管程序集。你可以创建包含你想要在多个项目间共享的代码的类，然后从不同类型的项目引用这些类。

你可以使用 .NET Framework 可移植类库来实现模型-视图-视图模型 (MVVM) 模式，并在多个平台之间共享程序集。你可以在 Visual Studio 2012 中的可移植类库项目内实现模型和视图模型类，然后创建针对不同平台自定义的视图。例如，在平台之间共用的模型代码能够以平台未知的方式从 Azure 移动服务等源中检索数据。MSDN 库提供了[概述和示例][]、[API 差异][]介绍、有关[使用可移植类库实现 MVVM 模式][]的示例、其他[说明性指导][]，以及有关在可移植类库项目中[管理资源][]的信息。

  [教程和资源]: /zh-cn/develop/mobile/resources/
  [Azure 通知中心]: /zh-cn/develop/net/how-to-guides/service-bus-notification-hubs/
  [通知中心入门]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet/
  [向用户发送跨平台通知]: /zh-cn/manage/services/notification-hubs/notify-users-xplat-mobile-services/
  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet-vs2012/
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-push-js-vs2012/
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/get-started-with-push-wp8/
  [iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-push-ios/
  [Android]: /zh-cn/develop/mobile/tutorials/get-started-with-push-android/
  [1]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet-vs2012
  [2]: /zh-cn/develop/mobile/tutorials/get-started-with-push-js-vs2012
  [动态架构]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj193175.aspx
  [推送对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554217.aspx
  [Windows 应用商店]: /zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet/
  [3]: /zh-cn/develop/mobile/tutorials/single-sign-on-wp8/
  [概述和示例]: http://msdn.microsoft.com/zh-cn/library/gg597391(v=vs.110)
  [API 差异]: http://msdn.microsoft.com/zh-cn/library/gg597392(v=vs.110)
  [使用可移植类库实现 MVVM 模式]: http://msdn.microsoft.com/zh-cn/library/hh563947(v=vs.110)
  [说明性指导]: http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/jj714086(v=vs.105).aspx
  [管理资源]: http://msdn.microsoft.com/zh-cn/library/hh871422(v=vs.110)
