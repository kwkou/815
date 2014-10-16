<properties linkid="develop-net-tutorials-push-notifications-to-users-wp8" urlDisplayName="Push Notifications to Users (WP8)" pageTitle="Push notifications to users (Windows Phone) | Mobile Dev Center" metaKeywords="" description="Learn how to use Mobile Services to push notifications to users of your Windows Phone app." metaCanonical="" services="" documentationCenter="" title="Push notifications to users by using Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 使用移动服务向用户推送通知

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-wp8" title="Windows Phone" class="current">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-android" title="Android">Android</a>
</div>

本主题是[前面的推送通知教程][]的引伸，其中添加了一个用于存储 Microsoft 推送通知服务 (MPNS) 通道 URI 的新表。然后，可以使用这些通道向 Windows Phone 8 应用程序的用户发送推送通知。

本教程将指导你完成在应用程序中更新推送通知的以下步骤：

1.  [创建 Channel 表][]
2.  [更新应用程序][]
3.  [更新服务器脚本][]
4.  [验证推送通知行为][]

本教程是在移动服务快速入门和前面的[推送通知入门][前面的推送通知教程]教程的基础上制作的。在开始本教程之前，必须先完成[推送通知入门][前面的推送通知教程]。

<a name="create-table"></a>
## 创建新表

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][]

2.  单击“数据”选项卡 ，然后单击“创建” 。

    ![][1]

    此时将显示“创建新表” 对话框。

3.  为所有权限保留默认的“具有应用程序密钥的任何人” 设置，在“表名” 中键入 *Channel*，然后单击勾选按钮。

    ![][2]

    此时将会创建用于存储通道 URI 的 "Channel" 表，你可以使用这些 URI 发送不同于项数据的推送通知。

接下来，你需要修改推送通知应用程序，以便将数据存储在此新表而不是 "TodoItem" 表中。

<a name="update-app"></a>
## 更新应用程序

1.  在 Visual Studio 2012 Express for Windows Phone 中，打开[推送通知入门][前面的推送通知教程]教程中的项目，打开文件 MainPage.xaml.cs，并从 "TodoItem" 类中删除 "Channel" 属性。该类现在应如下所示：

        public class TodoItem
        {
        public int Id { get; set; }

        [JsonProperty(PropertyName = "text")]
        public string Text { get; set; }

        [JsonProperty(PropertyName = "complete")]
        public bool Complete { get; set; }
        }

2.  将 "ButtonSave\_Click" 事件处理程序方法替换为此方法的原始版本，如下所示：

        private void ButtonSave_Click(object sender, RoutedEventArgs e)
        {
        var todoItem = new TodoItem { Text = TextInput.Text };
        InsertTodoItem(todoItem);
        }

3.  添加以下用于创建新的 "Channel" 类的代码：

        public class Channel
        {
        public int Id { get; set; }

        [JsonProperty(PropertyName = "uri")]
        public string Uri { get; set; }
        }

4.  打开文件 App.xaml.cs，并将 "AcquirePushChannel" 方法替换为以下代码：

        private void AcquirePushChannel()
        {
        CurrentChannel = HttpNotificationChannel.Find("MyPushChannel");

        if (CurrentChannel == null)
            {
        CurrentChannel = new HttpNotificationChannel("MyPushChannel");
        CurrentChannel.Open();
        CurrentChannel.BindToShellTile();
            }

        IMobileServiceTable<Channel> channelTable = App.MobileService.GetTable<Channel>();
        var channel = new Channel { Uri = CurrentChannel.ChannelUri.ToString() };
        channelTable.InsertAsync(channel);
        }

    此代码向 Channel 表中插入一个通道。

<a name="update-scripts"></a>
## 更新服务器脚本

1.  在管理门户中，单击“数据” 选项卡，然后单击 "Channel" 表。

    ![][3]

2.  在“通道” 中，单击“脚本” 选项卡，然后选择“插入” 。

    ![][4]

    将显示当 "Channel" 表中发生插入时所调用的函数。

3.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        var channelTable = tables.getTable('Channel');
        channelTable
        .where({ uri:item.uri })
        .read({ success:insertChannelIfNotFound });
        function insertChannelIfNotFound(existingChannels) {
        if (existingChannels.length > 0) {
        request.respond(200, existingChannels[0]);
        } else {
        request.execute();
                    }
                }
            }

    此脚本将检查 "Channel" 表中是否存在具有相同 URI 的现有通道。仅当未找到匹配的通道时，插入操作才会继续。这可以防止出现重复的通道记录。

4.  单击“TodoItem” ，再单击“脚本”并选择“插入”。 

    ![][5]

5.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        request.execute({
        success:function() {
        request.respond();
        sendNotifications();
                }
            });

        function sendNotifications() {
        var channelTable = tables.getTable('Channel');
        channelTable.read({
        success:function(channels) {
        channels.forEach(function(channel) {
        push.mpns.sendFlipTile(channel.uri, {
        title:item.text
                            }, {
        success:function(pushResponse) {
        console.log("Sent push:", pushResponse);
                                }
                            });
                        });
                    }
                });
            }
        }

    此插入脚本会向 "Channel" 表中存储的所有通道发送推送通知（包括已插入项的文本）。

<a name="test-app"></a>
## 测试应用程序

1.  在 Visual Studio 中，选择“生成” 菜单上的“部署解决方案” 。

2.  点击名为 "TodoList" 或 "hello push" 的磁贴以启动应用程序。

    ![][6]

3.  在应用程序的文本框内输入文本“hello push again”，然后单击“保存” 。

    ![][7]

    此时会将一个插入请求发送到移动服务，以存储添加的项。

4.  按“开始”按钮 以返回到“开始”菜单。

    ![][8]

    请注意，应用程序已收到该推送通知，并且磁贴的标题现在是 "hello push"。

5.  （可选）同时在两个设备上运行应用程序，并重复前一步骤。

    通知将发送到所有正在运行的应用程序实例。

## 后续步骤

演示推送通知操作基础知识的教程到此结束。建议你了解有关以下移动服务主题的详细信息：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

  [Windows Phone]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-android "Android"
  [前面的推送通知教程]: /zh-cn/develop/mobile/tutorials/get-started-with-push-wp8
  [创建 Channel 表]: #create-table
  [更新应用程序]: #update-app
  [更新服务器脚本]: #update-scripts
  [验证推送通知行为]: #test-app
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-services-selection.png
  [1]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-create-table.png
  [2]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-create-channel-table.png
  [3]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-portal-data-tables-channel.png
  [4]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-insert-script-channel.png
  [5]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-insert-script-push2.png
  [6]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-quickstart-push4-wp8.png
  [7]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-quickstart-push5-wp8.png
  [8]: ./media/mobile-services-windows-phone-push-notifications-app-users/mobile-quickstart-push6-wp8.png
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-wp8
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-wp8
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/p/?LinkId=262293
