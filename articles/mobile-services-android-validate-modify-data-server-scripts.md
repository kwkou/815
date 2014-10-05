<properties linkid="develop-mobile-tutorials-validate-modify-and-augment-data-android" urlDisplayName="Validate Data - Android" pageTitle="Use server scripts to validate and modify data (Android) | Mobile Dev Center" metaKeywords="" description="Learn how to validate and modify data sent using server scripts from your Android app." metaCanonical="" services="" documentationCenter="Mobile" title="Validate and modify data in Mobile Services by using server scripts" authors="ricksal" solutions="" manager="" editor="" />

# 使用服务器脚本在移动服务中验证和修改数据

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet" title="Windows Store C#">Windows Store C#</a><a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-js" title="Windows Store JavaScript">Windows Store JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-android" title="Android" class="current">Android</a><a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-html" title="HTML">HTML</a><a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-android" title="Xamarin.Android">Xamarin.Android</a>
</div>

本主题说明如何在 Azure 移动服务中利用服务器脚本。你可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。在本教程中，你将要定义并注册用于验证和修改数据的服务器脚本。由于服务器端脚本的行为往往会影响到客户端，因此你还要更新 Android 应用程序以利用这些新行为。

本教程将指导你完成以下基本步骤：

1.  [添加字符串长度验证][]
2.  [更新客户端以支持验证][]
3.  [在插入操作中添加时间戳][]
4.  [更新客户端以显示时间戳][]

本教程以前一教程[数据处理入门][]中的步骤和示例应用程序为基础。在开始本教程之前，必须先完成[数据处理入门][]。

<a name="string-length-validation"></a>
## 添加验证

验证用户提交的数据的长度总不失为一种良好做法。首先，你要注册一个脚本，用于验证发送到移动服务的字符串数据长度，并拒绝过长（在本例中为 10 个字符以上）的字符串。

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][]

2.  单击“数据” 选项卡，然后单击 TodoItem  表。

    ![][1]

3.  单击“脚本”，然后选择“插入”操作 。

    ![][2]

4.  将现有脚本替换为以下函数，然后单击“保存” 。

        function insert(item, user, request) {
        if (item.text.length > 10) {
        request.respond(statusCodes.BAD_REQUEST, 'Text length must be 10 characters or less.');
        } else {
        request.execute();
            }
        }

    此脚本将检查 "text" 属性的长度，如果该长度超过 10 个字符，则发送错误响应。如果未超过 10 个字符，将调用 "execute" 方法以完成插入。

    <div class="dev-callout">
    <b>说明</b>

    <p>在“脚本”选项卡中，依次单击“清除”和“保存”可以删除某个已注册的脚本 。</p>
	</div>

<a name="update-client-validation"></a>
## 更新客户端

移动服务会验证数据和发送错误响应，而你则需要验证你的应用程序是否能够正确处理验证后生成的错误响应。

1.  在 Eclipse 中，打开你在完成[数据处理入门][]教程后创建的项目。

2.  在 ToDoActivity.java 文件中，找到 "addItem" 方法，并将对 createAndShowDialog 方法的调用替换为以下代码：

        createAndShowDialog(exception.getCause().getMessage(), "Error");

    此时将显示移动服务返回的错误消息。

3.  在“运行”菜单中单击“运行”以启动应用程序，在文本框中键入 10 个字符以上的文本，然后单击“添加”按钮 。

可以看到，错误已被处理，并且已向用户显示了错误消息。

<a name="add-timestamp"></a>
## 添加时间戳

前面的任务验证了某个插入操作，并已接受或拒绝该操作。现在，你要使用一个服务器脚本来更新已插入的数据，该脚本将在插入对象之前向其添加一个时间戳属性。

"说明"

此处演示的 "createdAt" 时间戳属性目前是多余的。移动服务会自动为每个表创建一个 "\_\_createdAt" 系统属性。

1.  在[管理门户][Azure 管理门户]中的“脚本”选项卡上，将当前的 "Insert" 脚本替换为以下函数，然后单击“保存” 。

        function insert(item, user, request) {
        if (item.text.length > 10) {
        request.respond(statusCodes.BAD_REQUEST, 'Text length must be under 10');
        } else {
        item.createdAt = new Date();
        request.execute();
            }
        }

    在通过调用 "request"."execute" 插入对象之前，此函数会将一个新的 "createdAt" 时间戳属性添加到该对象，从而扩展前面的 insert 脚本。

    <div class="dev-callout"><b>说明</b>

    <p>首次运行此 insert 脚本时，必须启用动态架构。启用动态架构后，移动服务在首次执行时会自动将 <b>createdAt</b> 列添加到 <b>TodoItem</b> 表。默认情况下，将为新的移动服务启用动态架构，发布应用程序之前应该禁用动态架构。</p>
	</div>

2.  在“运行”菜单中单击“运行”以启动应用程序，在文本框中键入文本（长度小于 10 个字符），然后单击“添加” 。

    你会发现，新的时间戳并未显示在应用程序 UI 中。

3.  返回管理门户，在“todoitem”表中单击“浏览”选项卡 。

    可以看到，现在出现了一个“createdAt” 列，并且插入的新项带有一个时间戳值。

接下来，需要更新 Android 应用程序以显示这个新列。

<a name="update-client-timestamp"></a>
## 再次更新客户端

移动服务客户端将忽略响应中的、无法序列化成已定义类型上的属性的所有数据。最后一步是更新客户端以显示这些新数据。

1.  在包资源管理器中打开 ToDoItem.java 文件，然后添加以下 "import" 语句：

        import java.util.Date;

2.  将以下代码添加到 "TodoItem" 类中的 private 字段定义：

        /"
        * Timestamp of the item inserted by the service.
         */
        @com.google.gson.annotations.SerializedName("createdAt")
        private Date mCreatedAt;

    <div class="dev-callout"><b>说明</b>

    <p><code>SerializedName</code> 批注告知客户端要将应用程序中的新 <code>mCreatedAt</code> 属性映射到 TodoItem 表中定义的 <code>createdAt</code> 列，该列具有不同的名称。如果你使用此批注，应用程序在对象上使用的属性名称可以不同于 SQL Database 中的列名称。如果不使用此批注，则会因大小写不同而发生错误。</p>
	</div>

3.  将以下方法添加到 ToDoItem 类，以获取并设置新的 mCreatedAt 属性：

        /"
        * Sets the timestamp.
         * 
        * @param date
        *            timestamp to set
         */
        public final void setCreatedAt(Date date) {
        mCreatedAt = date;
        }

        /"
        * Returns the timestamp.
         */
        public Date getCreatedAt() {
        return mCreatedAt;
        }

4.  在包资源管理器中打开 ToDoItemAdapter.java 文件，然后添加以下 "import" 语句：

        import java.text.DateFormat;

5.  在 GetView 方法中添加以下代码：

        String createdAtText = "";
        if (currentItem.getCreatedAt() != null){
        DateFormat formatter = DateFormat.getDateInstance(DateFormat.SHORT);
        createdAtText = formatter.format(currentItem.getCreatedAt());
        }

    这样，当存在时间戳值时，就会生成带格式的日期字符串。

6.  找到代码 `checkBox.setText(currentItem.getText());` 并将该代码行替换为以下代码：

        checkBox.setText(currentItem.getText() + " " + createdAtText);

    这会在要显示的项后面追加时间戳日期。

7.  在“运行” 菜单中，单击“运行”以启动应用程序 。

    可以看到，只显示了你在更新 insert 脚本后插入的项的时间戳。

8.  将现有的 "RefreshItemsFromTable" 方法替换为以下代码：

        private void refreshItemsFromTable() {

        mToDoTable.where().field("complete").eq(false).and().field("createdAt").ne((String)null)
        .execute(new TableQueryCallback<ToDoItem>() {

        public void onCompleted(List<ToDoItem> result, int count,
        Exception exception, ServiceFilterResponse response) {
        if (exception == null) {
        mAdapter.clear();

        for (ToDoItem item :result) {
        mAdapter.add(item);
                                }

        } else {
        createAndShowDialog(exception, "Error");
                            }
                        }
                    });
        }

    此方法将更新查询，以便同时筛选掉没有时间戳值的项。

9.  在“运行” 菜单中，单击“运行”以启动应用程序 。

    可以看到，创建的不带时间戳值的所有项已从 UI 中消失。

现在，你已完成了这篇数据处理教程。

<a name="next-steps"> </a>
## 后续步骤

现在你已完成本教程，建议你继续学习数据系列中的最后一篇教程：[使用分页优化查询][]。

在为用户授权以及发送推送通知时，也可以使用服务器脚本。有关详细信息，请参阅以下教程：

-   [使用脚本为用户授权][]
    了解如何基于某个已经过身份验证的用户的 ID 筛选数据。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-html "HTML"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-android "Xamarin.Android"
  [添加字符串长度验证]: #string-length-validation
  [更新客户端以支持验证]: #update-client-validation
  [在插入操作中添加时间戳]: #add-timestamp
  [更新客户端以显示时间戳]: #update-client-timestamp
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-android
  [Azure 管理门户]: https://manage.windowsazure.cn/
  []: ./media/mobile-services-android-validate-modify-data-server-scripts/mobile-services-selection.png
  [1]: ./media/mobile-services-android-validate-modify-data-server-scripts/mobile-portal-data-tables.png
  [2]: ./media/mobile-services-android-validate-modify-data-server-scripts/mobile-insert-script-users.png
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-android
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-android
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-android
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
