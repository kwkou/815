<properties 
	pageTitle="使用移动服务 Android 客户端库" 
	description="了解如何使用适用于 Azure 移动服务的 Android 客户端。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="erikre"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="04/07/2016"
	wacn.date="05/23/2016"/>


# 如何使用适用于移动服务的 Android 客户端库

[AZURE.INCLUDE [mobile-services-selector-client-library](../includes/mobile-services-selector-client-library.md)]
 
本指南说明如何使用适用于 Azure 移动服务的 Android 客户端执行常见任务。所述的任务包括：查询数据；插入、更新和删除数据；对用户进行身份验证；处理错误；自定义客户端。

如果你移动服务的新手，应该先完成[移动服务入门]教程。成功完成该教程可确保你会安装 Android Studio；该软件可帮助你配置帐户并创建第一个移动服务，安装支持 Android 2.2 或更高版本的移动服务 SDK，但我们建议你针对 Android 4.2 或更高版本进行生成。

<!-- 可以在[此处](http://go.microsoft.com/fwlink/p/?LinkId=298735)找到有关 Android 客户端库的 Javadocs API 参考。-->

[AZURE.INCLUDE [mobile-services-concepts](../includes/mobile-services-concepts.md)]

##<a name="setup"></a>安装与先决条件

假设你已创建一个移动服务和一个表。有关详细信息，请参阅[创建表](http://go.microsoft.com/fwlink/p/?LinkId=298592)。在本主题使用的代码中，我们假设表的名称为 *ToDoItem*，其中包含以下列：

- ID
- text
- complete

相应的类型化客户端对象如下：

	public class ToDoItem {
		private String id;
		private String text;
		private Boolean complete;
	}
	
启用动态架构后，Azure 移动服务将基于 insert 或 update 请求中的对象自动生成新列。有关详细信息，请参阅[动态架构](https://msdn.microsoft.com/zh-cn/library/azure/jj193175.aspx)。

##<a name="create-client"></a>如何创建移动服务客户端
以下代码将创建用于访问移动服务的 MobileServiceClient 对象。代码会进入在 *AndroidManifest.xml* 中指定为 **MAIN** 操作和 **LAUNCHER** 类别的 Activity 类的 `onCreate` 方法。

		MobileServiceClient mClient = new MobileServiceClient(
				"MobileServiceUrl", // Replace with the above Site URL
				"AppKey", 			// replace with the Application Key 
				this)

在上面的代码中，请将 `MobileServiceUrl` 和 `AppKey` 依次替换为移动服务 URL 和应用程序密钥。在 Azure 经典门户中选择你的移动服务，然后单击“仪表板”即可获取这两个值。

## <a name="instantiating"></a>如何创建表引用

在移动服务中查询或修改数据的最简单方法就是使用类型化编程模型，因为 Java 是强类型化语言（稍后我们将会介绍非类型化模型）。在客户端和移动服务之间发送数据时，此模型使用 [gson](http://go.microsoft.com/fwlink/p/?LinkId=290801) 库提供对 JSON 的无缝序列化和反序列化：开发人员无需执行任何操作，该框架将处理一切。

查询或修改数据所要执行的第一项操作就是通过对 **MobileServiceClient** 调用 **getTable** 方法来创建一个 **MobileServiceTable** 对象。下面是此方法的两个重载：

	public class MobileServiceClient {
	    public <E> MobileServiceTable<E> getTable(Class<E> clazz);
	    public <E> MobileServiceTable<E> getTable(String name, Class<E> clazz);
	}

在以下代码中，*mClient* 是对移动服务客户端的引用。

如果类名称与表名称相同，则使用**第一个重载**：

		MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable(ToDoItem.class);


如果表名称与类型名称不同，则使用**第二个重载**。

		MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable("ToDoItemBackup", ToDoItem.class);

## <a name="api"></a>API 结构
 
从 2.0 版客户端库开始，移动服务表操作将在所有异步作业（例如涉及查询的方法）和操作（例如插入、更新和删除）中使用 [Future](http://developer.android.com/reference/java/util/concurrent/Future.html) 和 [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 对象。这就可以更方便地（在后台线程上）执行多个操作，而无需处理多个嵌套回调。


## <a name="querying"></a>如何从移动服务查询数据

本部分介绍如何向移动服务发出查询。其中的小节介绍了排序、筛选和分页等不同操作。最后，我们将介绍如何将这些操作连接起来。

### <a name="showAll"></a>如何返回表中的所有项

以下代码将返回 *ToDoItem* 表中的所有项。它会通过将项添加到适配器在 UI 中显示这些项。此代码类似于[移动服务快速入门]教程中的内容。

		new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {

					final MobileServiceList<ToDoItem> result = mToDoTable.execute().get();
                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            mAdapter.clear();
						for (ToDoItem item : result) {
                                mAdapter.add(item);
					}
				}
			});
               } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
               }
               return result;
            }
		}.execute();


与此类似的查询使用 [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 对象。

*result* 变量返回查询的结果，`mToDoTable.execute().get()` 语句后面的代码演示如何显示单个行。


### <a name="filtering"></a>如何筛选返回的数据

以下代码将返回 *complete* 字段等于 *false* 的 *ToDoItem* 表中的所有项。*mToDoTable* 是对前面创建的移动服务表的引用。

        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final MobileServiceList<ToDoItem> result = 
						mToDoTable.where().field("complete").eq(false).execute().get();
					for (ToDoItem item : result) {
                		Log.i(TAG, "Read object with ID " + item.id);  
					}
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
				} 
                return null;
			}
        }.execute();



通过对表引用执行 **where** 方法调用来启动筛选器。然后，依次执行 **field** 方法调用和用于指定逻辑谓词的方法调用。可能的谓词方法包括 **eq**、**ne**、**gt**、**ge**、**lt**、**le** 等。

执行这些操作便足以将数字和字符串字段与特定值进行比较。不过，你还可以执行其他许多操作。

例如，你可以按日期筛选。你可以比较整个日期字段，或者使用 **year**、**month**、**day**、**hour**、**minute**和**second**等方法比较日期的一部分。以下代码片段将会针对“截止日期”等于 2013 的项添加一个筛选器。

		mToDoTable.where().year("due").eq(2013).execute().get();

你可以使用 **startsWith**、**endsWith**、**concat**、**subString**、**indexOf**、**replace**、**toLower**、**toUpper**、**trim**和**length**等方法对字符串字段运行各种复杂筛选器。以下代码片段将会筛选 *text* 列以“PRI0”开头的表行。

		mToDoTable.where().startsWith("text", "PRI0").execute().get();

还允许使用  **add**、**sub**、**mul**、**div**、**mod**、**floor**、**ceiling**和**round** 等方法对数字字段运行各种更复杂的筛选器。以下代码片段将会筛选其中的 *duration* 为偶数的表行。

		mToDoTable.where().field("duration").mod(2).eq(0).execute().get();


你可以使用 **and**、**or**和**not** 等方法来组合谓词。以下代码片段将组合上面的两个示例。

		mToDoTable.where().year("due").eq(2013).and().startsWith("text", "PRI0")
					.execute().get();

你可以按照以下代码片段所示来组合与嵌套逻辑运算符：

		mToDoTable.where()
					.year("due").eq(2013)
						.and
					(startsWith("text", "PRI0").or().field("duration").gt(10))
					.execute().get();

有关筛选操作的更详细介绍和示例，请参阅[了解移动服务 Android 客户端查询模型的丰富功能](http://hashtagfail.com/post/46493261719/mobile-services-android-querying)。

### <a name="sorting"></a>如何为返回的数据排序

以下代码将返回 *ToDoItems* 表中的所有项，返回的结果已按 *text* 字段的升序排序。*mToDoTable* 是对前面创建的移动服务表的引用。

		mToDoTable.orderBy("text", QueryOrder.Ascending).execute().get();

**orderBy** 方法的第一个参数是与要排序的字段名称相同的字符串。

第二个参数使用 **QueryOrder** 枚举来指定是按升序还是按降序排序。

请注意，如果你使用 ***where*** 方法筛选，则必须在调用 ***orderBy*** 方法之前调用 ***where*** 方法。

### <a name="paging"></a>如何在页中返回数据

第一个示例演示了如何选择表中的前 5 个项。该查询将返回 *ToDoItems* 表中的项。*mToDoTable* 是对前面创建的移动服务表的引用。

       final MobileServiceList<ToDoItem> result = mToDoTable.top(5).execute().get();


接下来，我们定义一个查询，以跳过前 5 个项，返回后 5 个项。

		mToDoTable.skip(5).top(5).execute().get();


### <a name="selecting"></a>如何选择特定的列

以下代码演示如何返回 *ToDoItems* 表中的所有项，但只显示 *complete* 和 *text* 字段。*mToDoTable* 是对前面创建的移动服务表的引用。

		mToDoTable.select("complete", "text").execute().get();

	
在这里，select 函数的参数是要返回的表列的字符串名称。

**select** 方法需接在 **where** 和 **orderBy** 等方法（如果存在）的后面。它可以后接 **top** 等方法。

### <a name="chaining"></a>如何连接查询方法 

可以连接用于查询移动服务表的方法。这样，你便可以执行多种操作，例如，选择已排序并分页的筛选行的特定列。你可以创建相当复杂的逻辑筛选器。

这种操作的工作原理是通过使用的查询方法返回 **MobileServiceQuery&lt;T&gt;** 对象，随之又对这些对象调用更多的方法。若要结束方法序列并真正运行查询，你可以调用 **execute** 方法。

在以下代码示例中，*mToDoTable* 是对移动服务 *ToDoItem* 表的引用。

		mToDoTable.where().year("due").eq(2013)
						.and().startsWith("text", "PRI0")
						.or().field("duration").gt(10)
					.select("id", "complete", "text", "duration")
					.orderBy(duration, QueryOrder.Ascending).top(20)				
					.execute().get();

将方法链接在一起时，最重要的是 *where* 方法和谓词必须出现在最前面。然后，你就可以按照最符合应用程序需求的顺序调用后续方法。


## <a name="inserting"></a>如何在移动服务中插入数据

以下代码演示了如何在表中插入新行。

首先，实例化 *ToDoItem* 类的实例并设置该实例的属性。

		ToDoItem mToDoItem = new ToDoItem();
		mToDoItem.text = "Test Program";
		mToDoItem.complete = false;
		
 接着执行以下代码：

		// Insert the new item
	    new AsyncTask<Void, Void, Void>() {
	
	        @Override
	        protected Void doInBackground(Void... params) {
	            try {
	                mToDoTable.insert(item).get();
	                if (!item.isComplete()) {
	                    runOnUiThread(new Runnable() {
	                        public void run() {
	                            mAdapter.add(item);
			}
		});
	                }
	            } catch (Exception exception) {
	                createAndShowDialog(exception, "Error");
	            }
	            return null;
	        }
	    }.execute();


此代码将插入新项，并将其添加到适配器以便在 UI 中显示。

移动服务支持为表 ID 使用唯一的自定义字符串值。这样，应用程序便可为移动服务表的 ID 列使用自定义值（如电子邮件地址或用户名）。例如，如果你想要根据电子邮件地址识别每条记录，可以使用以下 JSON 对象。

		ToDoItem mToDoItem = new ToDoItem();
		mToDoItem.id = "myemail@mydomain.com";
		mToDoItem.text = "Test Program";
		mToDoItem.complete = false;

如果将新记录插入到表时未提供字符串 ID 值，移动服务将为 ID 生成唯一值。

支持字符串 ID 为开发人员带来了以下优势

+ 无需往返访问数据库即可生成 ID。
+ 更方便地合并不同表或数据库中的记录。
+ ID 值能够更好地与应用程序的逻辑相集成。

你也可以使用服务器脚本来设置 ID 值。下面的脚本示例将生成一个自定义 GUID 并将其分配给新记录的 ID。此 ID 类似于你未传入记录的 ID 值时，移动服务生成的 ID 值。

	//Example of generating an id. This is not required since Mobile Services
	//will generate an id if one is not passed in.
	item.id = item.id || newGuid();
	request.execute();

	function newGuid() {
		var pad4 = function(str) { return "0000".substring(str.length) + str; };
		var hex4 = function () { return pad4(Math.floor(Math.random() * 0x10000 /* 65536 */ ).toString(16)); };
		return (hex4() + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + hex4() + hex4());
	}


如果应用程序提供了某个 ID 的值，移动服务将按原样存储该值，包括前导和尾随空格。不会从值中裁剪掉空格。

`id` 的值必须唯一，并且不能包含以下集中的字符：

+ 控制字符：[0x0000-0x001F] 和 [0x007F-0x009F]。有关详细信息，请参阅 [ASCII 控制代码 C0 和 C1]。
+  可打印字符：**"**(0x0022), **+** (0x002B), **/** (0x002F), **?** (0x003F), **\** (0x005C), **`** (0x0060)
+  ID“.”和“..”

也可以为表使用整数 ID。若要使用整数 ID，必须使用 `mobile table create` 命令并结合 `--integerId` 选项创建表。应在适用于 Azure 的命令行界面 (CLI) 中使用此命令。有关使用 CLI 的详细信息，请参阅 [用于管理移动服务表的 CLI]。


##<a name="updating"></a>如何在移动服务中更新数据

以下代码演示了如何更新表中的数据。在此示例中，*item* 是对 *ToDoItem* 表中某个行的引用，该表包含一些更改。以下方法会更新表和 UI 适配器。

	private void updateItem(final ToDoItem item) {
	    if (mClient == null) {
	        return;
				} 
	
	    new AsyncTask<Void, Void, Void>() {
	
	        @Override
	        protected Void doInBackground(Void... params) {
	            try {
	                mToDoTable.update(item).get();
	                runOnUiThread(new Runnable() {
	                    public void run() {
	                        if (item.isComplete()) {
	                            mAdapter.remove(item);
	                        }
	                        refreshItemsFromTable();
			}
		});
	            } catch (Exception exception) {
	                createAndShowDialog(exception, "Error");
	            }
	            return null;
	        }
	    }.execute();
	}

## <a name="deleting"></a>如何在移动服务中删除数据

以下代码演示了如何删除表中的数据。该代码会从 ToDoItem 表中将 UI 上已选中“已完成”复选框的现有项删除。

	public void checkItem(final ToDoItem item) {
		if (mClient == null) {
			return;
		}

		// Set the item as completed and update it in the table
		item.setComplete(true);

		new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {
                    mToDoTable.delete(item);
                    runOnUiThread(new Runnable() {
                        public void run() {
                            if (item.isComplete()) {
                                mAdapter.remove(item);
		        }
                            refreshItemsFromTable();
		    }
		});
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
	}


以下代码演示了执行删除操作的另一种方法。该代码通过指定要删除的行的 ID 字段值（假设等于 "2FA404AB-E458-44CD-BC1B-3BC847EF0902"）来删除 ToDoItem 表中的现有项。在实际的应用程序中，你会以某种方式获取 ID，并将它作为变量传入。此处为了简化测试，你可以在 Azure 经典门户中转到你的服务，单击“数据”并复制你要测试的 ID。

    public void deleteItem(View view) {

        final String ID = "2FA404AB-E458-44CD-BC1B-3BC847EF0902";
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {
                    mToDoTable.delete(ID);
                    runOnUiThread(new Runnable() {
                        public void run() {
                            refreshItemsFromTable();
		    }
		});
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
    }

##<a name="lookup"></a>如何查找特定的项
有时，你需要按 *id* 查找特定的项，这一点不像查询，因为查询通常会返回满足某些条件的项集合。以下代码演示了如何执行此作业，此处假设 *id* 值为 `0380BAFB-BCFF-443C-B7D5-30199F730335`。在实际的应用程序中，你会以某种方式获取 ID，并将它作为变量传入。此处为了简化测试，你可以在 Azure 经典门户中转到你的服务，单击“数据”选项卡并复制你要测试的 ID。

    /**
     * Lookup specific item from table and UI
     */
    public void lookup(View view) {

        final String ID = "0380BAFB-BCFF-443C-B7D5-30199F730335";
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final ToDoItem result = mToDoTable.lookUp(ID).get();
                    runOnUiThread(new Runnable() {
                        public void run() {
                            mAdapter.clear();
                            mAdapter.add(result);
		    }
		});
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
    }

## <a name="untyped"></a>如何处理非类型化数据

使用非类型化编程模型可以全面控制 JSON 序列化，在某些情况下，你可能想要使用该模型。例如，你的移动服务表包含大量的列，而你只需要引用其中的某些列。使用类型化模型需要在数据类中定义移动服务表的所有列。但如果使用非类型化模型，只需定义你要使用的列。

用于访问数据的大多数 API 调用都与类型化编程调用类似。主要差别在于，在非类型化模型中，你要对 **MobileServiceJsonTable** 对象而不是 **MobileServiceTable** 对象调用方法。


### <a name="json_instance"></a>如何创建非类型化表的实例

与使用类型化模型相似，首先需要获取表引用，不过，此时该引用的是一个 **MobileServicesJsonTable** 对象。对移动服务客户端的实例调用 **getTable()** 方法可获取该引用。

首先定义变量：

    /**
     * Mobile Service Json Table used to access untyped data
     */
    private MobileServiceJsonTable mJsonToDoTable;



在 **onCreate** 方法中创建移动服务客户端的实例（在此处为 *mClient* 变量）后，接下来请使用以下代码创建 **MobileServiceJsonTable** 的实例。


	// Get the Mobile Service Json Table to use
	mJsonToDoTable = mClient.getTable("ToDoItem");

创建 **MobileServiceJsonTable** 的实例后，便可以对该实例调用使用类型化编程模型所能调用的几乎所有方法。但是，在某些情况下，这些方法采用非类型化参数，如以下示例所示。

### <a name="json_insert"></a>如何插入到非类型化表中

以下代码演示了如何执行插入。第一步是创建属于 <a href=" http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> 库的一部分的 [**JsonObject**](http://google-gson.googlecode.com/svn/trunk/gson/docs/javadocs/com/google/gson/JsonObject.html)。

		JsonObject item = new JsonObject();
		item.addProperty("text", "Wake up");
		item.addProperty("complete", false);

下一步是插入对象。传递给 **insert** 方法的回调函数是 **TableJsonOperationCallback** 类的实例。注意，*insert* 方法的参数如何成为 JsonObject。
		 
        // Insert the new item
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {
                    mJsonToDoTable.insert(item).get();
                    refreshItemsFromTable();
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
		        }
                return null;
		    }
        }.execute();


如果需要获取所插入对象的ID，请使用此方法调用：

	jsonObject.getAsJsonPrimitive("id").getAsInt());


### <a name="json_delete"></a>如何从非类型化表中删除数据

以下代码演示了如何删除一个实例，在本例中，该实例就是我们在前一个 *insert* 示例中创建的 **JsonObject** 的实例。请注意该代码与类型化案例相同，但方法具有不同的签名，因为它引用了 **JsonObject**。


    mToDoTable.delete(item);


还可以使用某个实例的 ID 来直接删除该实例：
		
	mToDoTable.delete(ID);



### <a name="json_get"></a>如何返回非类型化表中的所有行

以下代码演示了如何检索整个表。由于使用的是 Json 数据表，你可以选择性地只检索某些表的列。

    public void showAllUntyped(View view) {
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final JsonElement result = mJsonToDoTable.execute().get();
                    final JsonArray results = result.getAsJsonArray();
                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            mAdapter.clear();
		            for(JsonElement item : results){
                                String ID = item.getAsJsonObject().getAsJsonPrimitive("id").getAsString();
                                String mText = item.getAsJsonObject().getAsJsonPrimitive("text").getAsString();
                                Boolean mComplete = item.getAsJsonObject().getAsJsonPrimitive("complete").getAsBoolean();
                                ToDoItem mToDoItem = new ToDoItem();
                                mToDoItem.setId(ID);
                                mToDoItem.setText(mText);
                                mToDoItem.setComplete(mComplete);
                                mAdapter.add(mToDoItem);
		        }
		    }
		});
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
    }

你可以通过连接与类型化编程模型中所用方法同名的方法来执行筛选、排序和分页。


## <a name="binding"></a>如何将数据绑定到用户界面

数据绑定涉及到三个组件：

- 数据源
- 屏幕布局
- 将两者绑定起来的适配器

在以下示例代码中，我们会将移动服务表 *ToDoItem* 中的数据返回到一个数组中。这是数据应用程序经常使用的一种模式：数据库查询通常会返回行的集合，客户将在列表或数组中获取该集合。在此示例中，该数组就是数据源。

代码将指定屏幕布局，用于定义设备中显示的数据视图。

数据源和屏幕布局通过适配器绑定在一起，在此代码中，该适配器是 *ArrayAdapter&lt;ToDoItem&gt;* 类的扩展。

### <a name="layout"></a>如何定义布局
 
布局由多个 XML 代码段定义。以某个现有布局为例，我们假设以下代码表示了要在其中填充服务器数据的 **ListView**。

	    <ListView
	        android:id="@+id/listViewToDo"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        tools:listitem="@layout/row_list_to_do" >
	    </ListView>
	

在上面的代码中，*listitem* 属性指定列表中单个行的布局 ID。以下代码指定了一个复选框及其关联的文本。这些元素将会针对列表中的每个项实例化一次。如果使用更复杂的布局，则会在屏幕中指定更多的字段。以下代码摘自 *row\_list\_to\_do.xml* 文件。

		<?xml version="1.0" encoding="utf-8"?>
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		    android:layout_width="match_parent"
		    android:layout_height="match_parent"
		    android:orientation="horizontal">		    
		    <CheckBox
		        android:id="@+id/checkToDoItem"
		        android:layout_width="wrap_content"
		        android:layout_height="wrap_content"
		        android:text="@string/checkbox_text" />
		</LinearLayout>
		

### <a name="adapter"></a>如何定义适配器
	
由于此处视图的数据源是一个 *ToDoItem* 数组，因此我们需要基于 *ArrayAdapter&lt;ToDoItem&gt;* 类子类化适配器。此子类将使用 *row_list_to_do* 布局为每个 *ToDoItem* 生成一个视图。

在代码中，我们可以定义以下类作为 *ArrayAdapter&lt;E&gt;* 类的扩展：

		public class ToDoItemAdapter extends ArrayAdapter<ToDoItem> {


必须重写适配器的 *getView* 方法。以下示例代码演示了如何执行此操作：具体的代码根据应用程序而定。

	public View getView(int position, View convertView, ViewGroup parent) {
		View row = convertView;

		final ToDoItem currentItem = getItem(position);

		if (row == null) {
			LayoutInflater inflater = ((Activity) mContext).getLayoutInflater();
			row = inflater.inflate(R.layout.row_list_to_do, parent, false);
		}

		row.setTag(currentItem);

		final CheckBox checkBox = (CheckBox) row.findViewById(R.id.checkToDoItem);
		checkBox.setText(currentItem.getText());
		checkBox.setChecked(false);
		checkBox.setEnabled(true);

		return row;
	}

在活动中创建此类的实例，如下所示：

		ToDoItemAdapter mAdapter;
		mAdapter = new ToDoItemAdapter(this, R.layout.row_list_to_do);

请注意，ToDoItemAdapter 构造函数的第二个参数是对布局的引用。在该构造函数的调用后面添加以下代码，以便先获取对 **ListView** 的引用，然后调用 *setAdapter*，使该视图将自身配置为使用刚创建的适配器：

		ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
		listViewToDo.setAdapter(mAdapter);


### <a name="use-adapter"></a>如何使用适配器

现在，你可以使用数据绑定了。以下代码演示了如何获取移动服务表中的项，清除适配器，然后调用适配器的 *add* 方法以在表中填充返回的项。

    public void showAll(View view) {
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final MobileServiceList<ToDoItem> result = mToDoTable.execute().get();
                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
					mAdapter.clear();
					for (ToDoItem item : result) {
						mAdapter.add(item);
					}
				} 
		});
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
    }

每次修改 *ToDoItem* 表后，也必须调用该适配器（如果你想要显示执行修改操作后的结果）。由于修改是按记录完成的，因此要处理的是单个行而不是一个集合。插入项时，需要对适配器调用 *add* 方法；删除项时，需要调用 *remove* 方法。

##<a name="custom-api"></a>如何：调用自定义 API

自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。有关如何在移动服务中创建自定义 API 的示例，请参阅[如何：定义自定义 API 终结点](/documentation/articles/mobile-services-dotnet-backend-define-custom-api/)。

[AZURE.INCLUDE [mobile-services-android-call-custom-api](../includes/mobile-services-android-call-custom-api.md)]


##<a name="authentication"></a>如何对用户进行身份验证

移动服务支持使用各种外部标识提供者对应用用户进行身份验证和授权，这些提供者包括：Microsoft 帐户和 Azure Active Directory。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在后端中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅[身份验证入门](/documentation/articles/mobile-services-android-get-started-users/)。

支持两种身份验证流: 服务器流和客户端流。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能（例如单一登录）进行更深入的集成。

在应用程序中启用身份验证需要执行以下三个步骤：

- 注册你的应用程序以使用提供者进行身份验证，然后配置移动服务
- 将表权限限制给已经过身份验证的用户
- 向应用程序添加身份验证代码


移动服务支持使用以下现有标识提供者对用户进行身份验证：

- Microsoft 帐户
- Azure Active Directory

你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。还可以使用已经过身份验证的用户的 ID 来修改请求。

前两个任务可使用 [Azure 管理门户](https://manage.windowsazure.cn/)来完成。有关详细信息，请参阅[身份验证入门](/documentation/articles/mobile-services-android-get-started-users/)。

### <a name="caching"></a>如何向应用程序添加身份验证代码

1.  将以下 import 语句添加到应用程序的活动文件。

		import java.util.concurrent.ExecutionException;
		import java.util.concurrent.atomic.AtomicBoolean;

		import android.content.Context;
		import android.content.SharedPreferences;
		import android.content.SharedPreferences.Editor;

		import com.microsoft.windowsazure.mobileservices.authentication.MobileServiceAuthenticationProvider;
		import com.microsoft.windowsazure.mobileservices.authentication.MobileServiceUser;

2. 在活动类的 **onCreate** 方法中，在创建 `MobileServiceClient` 对象的代码后面添加以下代码行：我们假设对 `MobileServiceClient` 对象的引用为 *mClient*。
	
			// Login using the Google provider.
	    
		ListenableFuture<MobileServiceUser> mLogin = mClient.login(MobileServiceAuthenticationProvider.Google);

    	Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
						@Override
    		public void onFailure(Throwable exc) {
    			createAndShowDialog((Exception) exc, "Error");
							}
    		@Override
    		public void onSuccess(MobileServiceUser user) {
    			createAndShowDialog(String.format(
                        "You are now logged in - %1$2s",
                        user.getUserId()), "Success");
    			createTable();	
						}
					});

    此代码将使用 Google 登录对用户进行身份验证。此时将出现一个对话框，其中显示了已经过身份验证的用户的 ID。如果未正常完成身份验证，你将无法继续操作。

    > [AZURE.NOTE]如果使用的标识提供程序不是 Google，请将传递给上述 **login** 方法的值更改为下列其中一项：_MicrosoftAccount_或 _WindowsAzureActiveDirectory_。


3. 运行应用程序时，请使用选择的标识提供者登录。


### <a name="caching"></a>如何缓存身份验证令牌

本部分说明如何缓存身份验证令牌。执行此操作的目的是避免令牌仍然有效且应用程序处于“休眠”状态时用户必须再次完成身份验证。

缓存身份验证令牌需要将用户 ID 和身份验证令牌存储在设备本地。下一次启动应用程序时，你只需检查缓存，如果这些值存在，则你可以跳过登录过程，并使用这些数据重新进入客户端。但是，这些数据是敏感的，为安全起见，应该以加密形式存储，以防手机失窃。

以下代码段演示了如何获取 Microsoft 帐户登录的令牌。该令牌已缓存，以后如果找到了缓存，将重新加载该令牌。

	private void authenticate() {
		if (LoadCache())
		{
			createTable();
		}
		else
		{
			    // Login using the Google provider.    
				ListenableFuture<MobileServiceUser> mLogin = mClient.login(MobileServiceAuthenticationProvider.Google);
		
		    	Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
		                @Override
		    		public void onFailure(Throwable exc) {
		                        createAndShowDialog("You must log in. Login Required", "Error");
		                    }
		    		@Override
		    		public void onSuccess(MobileServiceUser user) {
		    			createAndShowDialog(String.format(
		                        "You are now logged in - %1$2s",
		                        user.getUserId()), "Success");
		    			cacheUserToken(mClient.getCurrentUser());
		    			createTable();	
		                }
		    	});		}
		}


	private boolean LoadCache()
	{
		SharedPreferences prefs = getSharedPreferences("temp", Context.MODE_PRIVATE);
		String tmp1 = prefs.getString("tmp1", "undefined"); 
		if (tmp1 == "undefined")
			return false;
		String tmp2 = prefs.getString("tmp2", "undefined"); 
		if (tmp2 == "undefined")
			return false;
		MobileServiceUser user = new MobileServiceUser(tmp1);
		user.setAuthenticationToken(tmp2);
		mClient.setCurrentUser(user);		
		return true;
	}


	private void cacheUser(MobileServiceUser user)
	{
		SharedPreferences prefs = getSharedPreferences("temp", Context.MODE_PRIVATE);
		Editor editor = prefs.edit();
		editor.putString("tmp1", user.getUserId());
		editor.putString("tmp2", user.getAuthenticationToken());
		editor.commit();
	}


如果令牌过期会发生什么情况呢？ 在这种情况下，如果你尝试使用它来建立连接，将会收到“401 未授权”响应。此时，用户必须登录以获取新令牌。使用筛选器可以截获对移动服务的调用以及来自移动服务的响应，因此不需要在应用程序中调用移动服务的每个位置编写代码来处理这种情况。此时，筛选器代码将测试 401 响应，根据需要触发登录进程，然后恢复生成 401 响应的请求。


## <a name="customizing"></a>如何自定义客户端

你可以通过多种方法自定义移动服务客户端的默认行为。

### <a name="headers"></a>如何自定义请求标头

你可能需要将一个自定义标头附加到每个传出请求。按如下所示配置 **ServiceFilter** 可以实现此目的：

	private class CustomHeaderFilter implements ServiceFilter {

        @Override
        public ListenableFuture<ServiceFilterResponse> handleRequest(
                	ServiceFilterRequest request, 
					NextServiceFilterCallback next) {

            runOnUiThread(new Runnable() {
		
		    @Override
                public void run() {
	        		request.addHeader("My-Header", "Value");	                }
		});

            SettableFuture<ServiceFilterResponse> result = SettableFuture.create();
            try {
                ServiceFilterResponse response = next.onNext(request).get();
                result.set(response);
            } catch (Exception exc) {
                result.setException(exc);
            }
        }

### <a name="serialization"></a>如何自定义序列化

默认情况下，移动服务假设服务器上与客户端上的表名称、列名称和数据类型都完全匹配。但是，在许多情况下，服务器上和客户端上的名称并不匹配。现举一例：你要更改某个现有的客户端，使其使用移动服务而不是竞争者的产品。

此时，你可能需要执行类似于下面的自定义操作：

- 使移动服务表中使用的列名称与你在客户端中使用的名称不匹配
- 使用一个移动服务表，其名称不同于该表在客户端中映射到的类
- 启用属性自动大写
- 向对象添加复杂属性

### <a name="columns"></a>如何映射不同的客户端名称和服务器名称

假设你的 Java 客户端代码为 *ToDoItem* 对象属性使用了类似于下面的标准 Java 样式名称。

- mId
- mText
- mComplete
- mDuration


则你必须将客户端名称序列化为与服务器上 *ToDoItem* 表的列名称匹配的 JSON 名称。以下代码利用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> 库来执行此操作。

	@com.google.gson.annotations.SerializedName("text")
	private String mText;

	@com.google.gson.annotations.SerializedName("id")
	private int mId;

	@com.google.gson.annotations.SerializedName("complete")
	private boolean mComplete;
 
	@com.google.gson.annotations.SerializedName("duration")
	private String mDuration;

### <a name="table"></a>如何在客户端与移动服务之间映射不同的表名称

如以下代码所示，只需使用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=296840" target="_blank">getTable()</a> 函数的重写之一，就能轻松地将客户端表名称映射为不同的移动服务表名称。

		mToDoTable = mClient.getTable("ToDoItemBackup", ToDoItem.class);


### <a name="conversions"></a>如何自动执行列名称映射

如前一部分中所示，映射只包含几个列的简短表的列名称并不复杂。但是，如果表包含大量的列（例如 20 或 30 个列），则我们可以调用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> API 并指定要应用到每个列的转换策略，这样就无需批注每一个列名称。

为此，我们需要使用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> 库，Android 客户端库在幕后使用该库将 Java 对象序列化为要发送到 Azure 移动服务的 JSON 数据。

以下代码使用 *setFieldNamingStrategy()* 方法，我们在其中定义了 *FieldNamingStrategy()* 方法。此方法指定删除初始字符（“m”），然后将每个字段名称的下一个字符小写。此代码还启用了输出 JSON 的整齐打印。

	client.setGsonBuilder(
	    MobileServiceClient
	    .createMobileServiceGsonBuilder()
	    .setFieldNamingStrategy(new FieldNamingStrategy() {
	        public String translateName(Field field) {
	            String name = field.getName();
	            return Character.toLowerCase(name.charAt(1))
	                + name.substring(2);
	            }
	        })
	        .setPrettyPrinting());
	


必须在对移动服务客户端对象执行任何方法调用之前执行此代码。

### <a name="complex"></a>如何将对象或数组属性存储到表中 

到目前为止，我们的所有序列化示例都使用了可轻松序列化成 JSON 和移动服务表的基元类型（例如整数和字符串）。假设我们要将一个不能自动序列化成 JSON 和表的复杂对象添加到客户端类型。例如，我们要将一个字符串数组添加到客户端对象。此时，我们需要指定如何执行序列化，以及如何将数组存储到移动服务表中。

若要查看有关如何执行此操作的示例，请阅读博客文章<a href="http://hashtagfail.com/post/44606137082/mobile-services-android-serialization-gson" target="_blank">在移动服务 Android 客户端中使用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> 库自定义序列化</a>。

每当我们要使用一个不能自动序列化成 JSON 和移动服务表的复杂对象时，就可以使用此常规方法。

<!-- Anchors. -->

[What is Mobile Services]: #what-is
[Concepts]: #concepts
[How to: Create the Mobile Services client]: #create-client
[How to: Create a table reference]: #instantiating
[The API structure]: #api
[How to: Query data from a mobile service]: #querying
[Return all Items]: #showAll
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[How to: Concatenate query methods]: #chaining
[How to: Bind data to the user interface]: #binding
[How to: Define the layout]: #layout
[How to: Define the adapter]: #adapter
[How to: Use the adapter]: #use-adapter
[How to: Insert data into a mobile service]: #inserting
[How to: update data in a mobile service]: #updating
[How to: Delete data in a mobile service]: #deleting
[How to: Look up a specific item]: #lookup
[How to: Work with untyped data]: #untyped
[How to: Authenticate users]: #authentication
[Cache authentication tokens]: #caching
[How to: Handle errors]: #errors
[How to: Design unit tests]: #tests
[How to: Customize the client]: #customizing
[Customize request headers]: #headers
[Customize serialization]: #serialization
[Next Steps]: #next-steps
[Setup and Prerequisites]: #setup

<!-- Images. -->



<!-- URLs. -->
[移动服务入门]: /documentation/articles/mobile-services-android-get-started/
[移动服务快速入门]: /documentation/articles/mobile-services-android-get-started/
[ASCII 控制代码 C0 和 C1]: http://zh.wikipedia.org/wiki/Data_link_escape_character#C1_set

<!---HONumber=Mooncake_0118_2016-->