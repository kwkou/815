<properties linkid="" urlDisplayName="" pageTitle="Push notifications to users (Android ) | Mobile Dev Center" metaKeywords="" description="Learn how to use Mobile Services to push notifications to users of your Android app." metaCanonical="" services="" documentationCenter="Mobile" title="Push notifications to users by using Mobile Services" authors="ricksal" solutions="" manager="" editor="" />

# 使用移动服务向用户推送通知

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-android" title="Android" class="current">Android</a>
</div>

<div class="dev-onpage-left-content">
<p>本主题是<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-android">前面的推送通知教程</a>的引伸，其中添加了一个用于存储 Google Cloud Messaging (GCM) 注册 URI 的新表，你可以使用这些 URI 向 Android 应用程序的多个用户发送推送通知。在本教程中，每当在 ToDoList 表中完成插入后，执行一次更新就能生成发送到所有已注册设备的推送通知。在前面的教程中，通知只会发送到执行插入的设备。</p>
</div>

本教程将指导你完成在应用程序中更新推送通知的以下步骤：

1.  [创建 Registration 表][]
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

3.  为所有权限保留默认的“具有应用程序密钥的任何人”设置，在“表名”中键入 *Registration*，然后单击勾选按钮 。

    ![][2]

    此时将会创建用于存储注册 URI 的 "Registration" 表，你可以使用这些 URI 发送不同于项数据的推送通知。

接下来，你需要修改推送通知应用程序，以便将注册数据存储在此新表而不是 "TodoItem" 表中。

<a name="update-app"></a>
## 更新应用程序

1.  在 Eclipse 的“包资源管理器”中，右键单击包（在 `src` 节点下），单击“新建”，再单击“类” 。

2.  在“名称”中键入 `Registration`，然后单击“完成” 

    ![][3]

    此时将创建新的 Registration 类。

3.  打开 ToDoItem.java 文件并剪切以下代码：

        @com.google.gson.annotations.SerializedName("handle")
        private String mHandle;

        public String getHandle() {
        return mHandle;
        }

        public final void setHandle(String handle) {
        mHandle = handle;
        }

4.  将前一步骤中剪切的代码粘贴到前面创建的 "Registration" 类的正文中。

5.  将以下代码添加到 "Registration" 类：

        @com.google.gson.annotations.SerializedName("id")
        private int mId;

        /"
        * Returns the item id
         */
        public int getId() {
        return mId;
        }

        /"
        * Sets the item id
         * 
        * @param id :id to set
         */
        public final void setId(int id) {
        mId = id;
        }

6.  打开 "ToDoActivity.java" 文件，并在 `addItem` 方法中删除以下行：

        item.setHandle(MyHandler.getHandle());

7.  找到 `mClient` 属性，并将它替换为以下代码：

        /"
        * Mobile Service Client reference
         */
        private static MobileServiceClient mClient;

        /"
        * Returns the client reference
         */
        public static MobileServiceClient getClient() {
        return mClient;
        }

8.  在 "MyHandler" 文件中添加以下 import 语句：

        import android.util.Log;

        import com.microsoft.windowsazure.notifications.NotificationsHandler;

        import com.microsoft.windowsazure.mobileservices.MobileServiceClient;
        import com.microsoft.windowsazure.mobileservices.MobileServiceTable;
        import com.microsoft.windowsazure.mobileservices.ServiceFilterResponse;
        import com.microsoft.windowsazure.mobileservices.TableOperationCallback;

9.  将以下代码添加到 `onRegistered` 方法的末尾：

        MobileServiceClient client = ToDoActivity.getClient();
        MobileServiceTable<Registration> registrations = client.getTable(Registration.class);

        // Create a new Registration
        Registration registration = new Registration();
        registration.setHandle(gcmRegistrationId);

        // Insert the new Registration
        registrations.insert(registration, new TableOperationCallback<Registration>() {

        public void onCompleted(Registration entity, Exception exception, ServiceFilterResponse response) {

        if (exception != null) {
        Log.e("MyHandler", exception.getMessage());
        } else {
        Log.e("MyHandler", "Registration OK");
                }
            }
        });

现在，你的应用程序已更新，支持向用户发送推送通知。

<a name="update-scripts"></a>
## 更新服务器脚本

1.  在管理门户中，单击“数据”选项卡，然后单击“Registration”表 。

    ![][4]

2.  在“registration” 中，单击“脚本” 选项卡，然后选择“插入” 。

    ![][5]

    将显示当 "Registration" 表中发生插入时所调用的函数。

3.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        var registrationTable = tables.getTable('Registration');
        registrationTable
        .where({ handle:item.handle })
        .read({ success:insertRegistrationIfNotFound });
        function insertRegistrationIfNotFound(existingRegistrations) {
        if (existingRegistrations.length > 0) {
        request.respond(200, existingRegistrations[0]);
        } else {
        request.execute();
                }
            }
        }

    此脚本将检查 "Registration" 表中是否存在具有相同 URI 的现有注册。仅当未找到匹配的注册时，插入操作才会继续。这可以防止出现重复的注册记录。

4.  单击“TodoItem” ，再单击“脚本”并选择“插入”。 

	![][6]

5.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        request.execute({
        success:function() {
        // Write to the response and then send the notification in the background
        request.respond();
        sendNotifications(item.text);

                }
            });

        function sendNotifications(item_text) {
        var registrationTable = tables.getTable('Registration');
        registrationTable.read({
        success:function(registrations) {
        registrations.forEach(function(registration) {
        push.gcm.send(registration.handle, item_text, {
        success:function(response) {
        console.log('Push notification sent:', response);
        }, error:function(error) {
        console.log('Error sending push notification:', error);
                            }
                        });
                    });
                }
            });
        }

    }

    此插入脚本会向 "Registration" 表中存储的所有注册发送推送通知（包括已插入项的文本）。

<a name="test-app"></a>
## 测试应用程序

1.  在 Eclipse 的“运行” 菜单中，单击“运行”以启动应用程序 。

2.  在应用程序中键入有意义的文本（例如 *A new Mobile Services task*），然后单击“添加” 按钮。

3.  屏幕下方将短暂出现一个黑色的通知框。

    ![][7]

你已成功完成本教程。

## 后续步骤

演示推送通知操作基础知识的教程到此结束。建议你了解有关以下移动服务主题的详细信息：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

-   [如何使用适用于移动服务的 Android 客户端库][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows Phone]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-android "Android"
  [前面的推送通知教程]: /zh-cn/develop/mobile/tutorials/get-started-with-push-android
  [创建 Registration 表]: #create-table
  [更新应用程序]: #update-app
  [更新服务器脚本]: #update-scripts
  [验证推送通知行为]: #test-app
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-android-push-notifications-app-users/mobile-services-selection.png
  [1]: ./media/mobile-services-android-push-notifications-app-users/mobile-create-table.png
  [2]: ./media/mobile-services-android-push-notifications-app-users/mobile-create-registration-table.png
  [3]: ./media/mobile-services-android-push-notifications-app-users/mobile-create-registration-class.png
  [4]: ./media/mobile-services-android-push-notifications-app-users/mobile-portal-data-tables-registration.png
  [5]: ./media/mobile-services-android-push-notifications-app-users/mobile-insert-script-registration.png
  [6]: ./media/mobile-services-android-push-notifications-app-users/mobile-insert-script-push2.png
  [7]: ./media/mobile-services-android-push-notifications-app-users/mobile-push-icon.png
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-android
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-android
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [如何使用适用于移动服务的 Android 客户端库]: /zh-cn/develop/mobile/how-to-guides/work-with-android-client-library
