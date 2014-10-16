## <a name="update-app"></a>更新应用程序以调用自定义 API

1.  我们将在现有按钮旁边添加一个标记为“Complete All”（完成全部）的按钮，并将两个按钮都下移一行。在 Eclipse 中，打开你的快速启动项目中的 *res\\layout\\activity\_to\_do.xml* 文件，找到包含以下名称的 **Button** 元素的 **LinearLayout** 元素：`buttonAddToDo`. 复制 **LinearLayout** 并将其紧邻着粘贴在原始元素之后。将 **Button** 元素从第一个 **LinearLayout** 中删除。

2.  在第二个 **LinearLayout** 中，删除 **EditText** 元素，并将以下代码紧邻着添加到现有 **Button** 元素之后：

        <Button
            android:id="@+id/buttonCompleteItem"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="completeItem"
            android:text="@string/complete_button_text" />

    这样可将新按钮添加到页面中的单独一行上，位于现有按钮旁边。

3.  第二个 **LinearLayout** 现在如下所示：

         <LinearLayout
            android:layout_width="match_parent" 
            android:layout_height="wrap_content" 
            android:background="#71BCFA"
            android:padding="6dip"  >
            <Button
                android:id="@+id/buttonAddToDo"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:onClick="addItem"
                android:text="@string/add_button_text" />
            <Button
                android:id="@+id/buttonCompleteItem"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:onClick="completeItem"
                android:text="@string/complete_button_text" />
        </LinearLayout>

4.  打开 res\\values\\string.xml 文件并添加以下代码行：

        <string name="complete_button_text">Complete All</string>

5.  在包资源管理器中，右键单击 *src* 文件夹中的项目名称 (`com.example.{your projects name}`)，选择“新建”，然后选择“类”。在对话框的类名称字段中输入 **MarkAllResult**，选择“确定”，并使用以下代码替代生成的类定义：

        import com.google.gson.annotations.SerializedName;

        public class MarkAllResult {
            @SerializedName("count")
            public int mCount;

            public int getCount() {
                return mCount;
            }

            public void setCount(int count) {
                    this.mCount = count;
            }
        }

    此类用于保存自定义 API 返回的行计数值。

6.  找到 **ToDoActivity.java** 文件中的 **refreshItemsFromTable** 方法，并确认第一行代码的开头如下所示：

        mToDoTable.where().field("complete").eq(false)

    这样可以筛选项，使得查询不返回已完成的项。

7.  确认 **ToDoActivity.java** 正在导入文件开头的导入块中的 ApiOperationCallback：

    import com.microsoft.windowsazure.mobileservices.ApiOperationCallback;

8.  在 **ToDoActivity.java** 文件中，添加以下方法：

        public void completeItem(View view) {
            mClient.invokeApi("completeAll", MarkAllResult.class, new ApiOperationCallback<MarkAllResult>() {
                @Override
                public void onCompleted(MarkAllResult result, Exception error, ServiceFilterResponse response) {
                    if (error != null) {
                        createAndShowDialog(error, "Error");
                    } else {
                        createAndShowDialog(result.getCount() + " item(s) marked as complete.", "Completed Items");
                        refreshItemsFromTable();
                    }
                }
            });
        }

    此方法可处理新按钮的 **Click** 事件。**invokeApi** 方法在客户端上调用，该客户端向新的自定义 API 发送 POST 请求。与任何错误相同，自定义 API 返回的结果也显示在消息对话框中。

## 测试应用程序

1.  在“运行”菜单中，单击“运行”，以便在 Android 模拟器中启动项目。

    这将执行使用 Android SDK 构建的应用程序，该应用程序使用客户端库发送一个查询，该查询从你的移动服务返回项目。

2.  在应用程序的“插入 TodoItem”中键入一些文本，然后单击“添加”。

3.  重复上述步骤，直至你将多个 ToDo 项添加到列表。

4.  单击“Complete All”（完成全部）按钮。

    ![][]

    此时会显示一个消息框，指示标记为完成的多个项，并再次执行筛选查询，将所有项从列表中清除。

  []: ./media/mobile-services-android-call-custom-api/mobile-custom-api-android-completed.png
