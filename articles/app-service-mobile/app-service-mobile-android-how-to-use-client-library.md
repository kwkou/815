<properties
    pageTitle="如何使用用于 Android 的 Azure 移动应用 SDK | Azure"
    description="如何使用用于 Android 的 Azure 移动应用 SDK"
    services="app-service\mobile"
    documentationcenter="android"
    author="adrianhall"
    manager="adrianhall" />
<tags
    ms.assetid="5352d1e4-7685-4a11-aaf4-10bd2fa9f9fc"
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-android"
    ms.devlang="java"
    ms.topic="article"
    ms.date="04/25/2017"
    wacn.date="05/31/2017"
    ms.author="adrianha"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="46f35823e07db31b81401594ca194db058adf615"
    ms.lasthandoff="05/19/2017" />

# <a name="how-to-use-the-azure-mobile-apps-sdk-for-android"></a>如何使用用于 Android 的 Azure 移动应用 SDK

本指南说明如何使用用于移动应用的 Android 客户端 SDK 来实现常见方案，例如：

- 查询数据（插入、更新和删除）。
- 身份验证。
- 处理错误。
- 自定义客户端。

本指南侧重于客户端 Android SDK。  若要详细了解移动应用的服务器端 SDK，请参阅 [Work with .NET backend SDK][10]（使用 .NET 后端）或 [How to use the Node.js backend SDK][11]（如何使用 Node.js 后端 SDK）。

## <a name="reference-documentation"></a>参考文档

可以在 GitHub 上找到有关 Android 客户端库的 [Javadocs API 参考][12]。

## <a name="supported-platforms"></a>支持的平台

用于 Android 的 Azure 移动应用 SDK 支持手机和平板电脑外形规格的 API 级别 19 到 24（KitKat 到 Nougat）。  具体而言，身份验证利用通用 Web 框架方法收集凭据。  服务器流身份验证不适用于手表等小型设备。

## <a name="setup-and-prerequisites"></a>安装与先决条件

完成[移动应用快速入门](/documentation/articles/app-service-mobile-android-get-started/)教程。此任务可确保满足开发 Azure 移动应用的所有先决条件。快速入门还帮助配置帐户及创建第一个移动应用后端。

如果决定不完成快速入门教程，请完成以下任务：

- [创建移动应用后端][13]，搭配 Android 应用使用。
- 在 Android Studio 中，[更新 Gradle 生成文件](#gradle-build)。
- [启用 Internet 权限](#enable-internet)。

###<a name="gradle-build"></a>更新 Gradle 生成文件

更改以下两个 **build.gradle** 文件：

1. 将以下代码添加到 *buildscript* 标记内 *项目* 级别的 **build.gradle** 文件：

		buildscript {
		    repositories {
		        jcenter()
		    }
		}

2. 将以下代码添加到*依赖关系*标记内*模块应用*级别的 **build.gradle** 文件：

        compile 'com.microsoft.azure:azure-mobile-android:3.3.0'

    目前的最新版本为 3.3.0。 [bintray][14] 中列出了支持的版本。

###<a name="enable-internet"></a>启用 Internet 权限

若要访问 Azure，应用中必须已启用 Internet 权限。如果尚未启用，请将以下代码行添加到 **AndroidManifest.xml** 文件：

	<uses-permission android:name="android.permission.INTERNET" />

## <a name="create-a-client-connection"></a>创建客户端连接

Azure 移动应用为移动应用程序提供四项功能：

* 使用 Azure 移动应用服务进行数据访问和脱机同步。
* 调用使用 Azure 移动应用服务器 SDK 编写的自定义 API。
* 使用 Azure 应用服务身份验证和授权进行身份验证。
* 使用通知中心进行推送通知注册。

若要使用其中的每项功能，首先需要创建一个 `MobileServiceClient` 对象。  只应在移动客户端中创建一个 `MobileServiceClient` 对象（即，该客户端应该采用单一实例模式）。  若要创建 `MobileServiceClient` 对象，请执行以下操作：

    MobileServiceClient mClient = new MobileServiceClient(
        "<MobileAppUrl>",       // Replace with the Site URL
        this);                  // Your application Context

`<MobileAppUrl>` 是一个字符串，或者是指向移动后端的 URL 对象。  如果使用 Azure 应用服务来托管移动后端，请确保使用 URL 的安全 `https://` 版本。

客户端还需要能够访问活动或上下文 - 本示例中的 `this` 参数。  MobileServiceClient 构造应发生在 `AndroidManifest.xml` 文件中引用的活动的 `onCreate()` 方法内。

最佳做法是将服务器通信抽象化为其自身的（单一实例模式）类。  在本例中，应该传递构造函数中的活动，以便适当地配置服务。  例如：

    package com.example.appname.services;

    import android.content.Context;
    import com.microsoft.windowsazure.mobileservices.*;

    public AzureServiceAdapter {
        private String mMobileBackendUrl = "https://myappname.azurewebsites.cn";
        private Context mContext;
        private MobileServiceClient mClient;
        private static AzureServiceAdapter mInstance = null;

        private AzureServiceAdapter(Context context) {
            mContext = context;
            mClient = new MobileServiceClient(mMobileBackendUrl, mContext);
        }

        public static void Initialize(Context context) {
            if (mInstance == null) {
                mInstance = new AzureServiceAdapter(context);
            } else {
                throw new IllegalStateException("AzureServiceAdapter is already initialized");
            }
        }

        public static AzureServiceAdapter getInstance() {
            if (mInstance == null) {
                throw new IllegalStateException("AzureServiceAdapter is not initialized");
            }
            return mInstance;
        }

        public MobileServiceClient getClient() {
            return mClient;
        }

        // Place any public methods that operate on mClient here.
    }

现在，可以调用主活动的 `onCreate()` 方法中的 `AzureServiceAdapter.Initialize(this);`。  需要访问客户端的其他任何方法使用 `AzureServiceAdapter.getInstance();` 获取对服务适配器的引用。

## <a name="data-operations"></a>数据操作

Azure 移动应用 SDK 的核心作用是让你访问移动应用后端上的 SQL Azure 中存储的数据。  可以使用强类型化类（首选）或非类型化查询（不建议）访问此数据。  本部分重点介绍如何使用强类型化类。

### <a name="define-client-data-classes"></a>定义客户端数据类

若要访问 SQL Azure 表的数据，可定义对应于移动应用后端中的表的客户端数据类。 本主题中的示例采用名为 **MyDataTable** 的表，其中包含以下列：

* id
* text
* complete

相应的类型化客户端对象驻留在名为 **MyDataTable.java** 的文件中：

	public class ToDoItem {
		private String id;
		private String text;
		private Boolean complete;
	}

为添加的每个字段添加 getter 和 setter 方法。  如果 SQL Azure 表包含多个列，请将相应的字段添加到此类。  例如，如果 DTO（数据传输对象）包含整数 Priority 列，则可以添加此字段，以及其 getter 和 setter 方法：

	private Integer priority;

	/**
	* Returns the item priority
	*/
	public Integer getPriority() {
	return mPriority;
	}
	
	/**
	* Sets the item priority
	*
	* @param priority
	*            priority to set
	*/
	public final void setPriority(Integer priority) {
	mPriority = priority;
	}

若要了解如何在移动应用后端中创建更多表，请参阅[如何定义表控制器][15]（.NET 后端）或[使用动态架构定义表][16]（Node.js 后端）。

Azure 移动应用后端表定义了五个特殊字段，其中四个字段可用于客户端：

* `String id`：记录的全局唯一 ID。  最佳做法是将 ID 设置为 [UUID][17] 对象的字符串表示形式。
* `DateTimeOffset updatedAt`：上次更新日期/时间。  updatedAt 字段由服务器设置，永远不可由客户端代码设置。
* `DateTimeOffset createdAt`：对象的创建日期/时间。  createdAt 字段由服务器设置，永远不可由客户端代码设置。
* `byte[] version`：通常以字符串表示，版本也由服务器设置。
* `boolean deleted`：指示记录已被删除，但尚未清除。  不要使用 `deleted` 作为类中的属性。

`id` 字段是必填的。  `updatedAt` 字段和 `version` 字段用于脱机同步（分别用于增量同步和冲突解决）。  `createdAt` 字段是一个引用字段，客户端不会使用它。  名称是属性的“全局”名称且不可调整。  但是，可以使用 [gson][3] 库在对象与“全局”名称之间创建映射。  例如：

    package com.example.zumoappname;

    import com.microsoft.windowsazure.mobileservices.table.DateTimeOffset;

    public class ToDoItem
    {
        @com.google.gson.annotations.SerializedName("id")
        private String mId;
        public String getId() { return mId; }
        public final void setId(String id) { mId = id; }

        @com.google.gson.annotations.SerializedName("complete")
        private boolean mComplete;
        public boolean isComplete() { return mComplete; }
        public void setComplete(boolean complete) { mComplete = complete; }

        @com.google.gson.annotations.SerializedName("text")
        private String mText;
        public String getText() { return mText; }
        public final void setText(String text) { mText = text; }

        @com.google.gson.annotations.SerializedName("createdAt")
        private DateTimeOffset mCreatedAt;
        public DateTimeOffset getUpdatedAt() { return mCreatedAt; }
        protected DateTimeOffset setUpdatedAt(DateTimeOffset createdAt) { mCreatedAt = createdAt; }

        @com.google.gson.annotations.SerializedName("updatedAt")
        private DateTimeOffset mUpdatedAt;
        public DateTimeOffset getUpdatedAt() { return mUpdatedAt; }
        protected DateTimeOffset setUpdatedAt(DateTimeOffset updatedAt) { mUpdatedAt = updatedAt; }

        @com.google.gson.annotations.SerializedName("version")
        private String mVersion;
        public String getText() { return mVersion; }
        public final void setText(String version) { mVersion = version; }

        public ToDoItem() { }

        public ToDoItem(String id, String text) {
            this.setId(id);
            this.setText(text);
        }

        @Override
        public boolean equals(Object o) {
            return o instanceof ToDoItem && ((ToDoItem) o).mId == mId;
        }

        @Override
        public String toString() {
            return getText();
        }
    }

### <a name="create-a-table-reference"></a>创建表引用

若要访问表，请先通过对 [MobileServiceClient][9] 调用 **getTable** 方法来创建一个 [MobileServiceTable][8] 对象。  此方法有两个重载：

    public class MobileServiceClient {
        public <E> MobileServiceTable<E> getTable(Class<E> clazz);
        public <E> MobileServiceTable<E> getTable(String name, Class<E> clazz);
    }

在以下代码中， **mClient** 是对 MobileServiceClient 对象的引用。  如果类名称与表名称相同，则使用第一个重载，这也是快速入门中使用的重载：

    MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable(ToDoItem.class);

如果表名称与类名称不同，则使用第二个重载：第一个参数是表名称。

    MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable("ToDoItemBackup", ToDoItem.class);

## <a name="query"></a>查询后端表

首先，请获取表引用。  然后对表引用执行查询。  查询是以下元素的任意组合：

* `.where()` [筛选子句](#filtering)。
* `.orderBy()` [排序子句](#sorting)。
* `.select()` [字段选择子句](#selection)。
* [分页结果](#paging)的 `.skip()` 和 `.top()`。

子句必须按上述顺序提供。

### <a name="filter"></a>筛选结果

查询的一般形式为：

    List<MyDataTable> results = mDataTable
        // More filters here
        .execute()          // Returns a ListenableFuture<E>
        .get()              // Converts the async into a sync result

上面的示例返回所有结果（结果数上限为服务器设置的最大页面大小）。  `.execute()` 方法在后端上执行查询。  查询先转换为 [OData v3][19] 查询，然后再传输到移动应用后端。  移动应用后端收到查询后，会先将查询转换为 SQL 语句，然后在 SQL Azure 实例上执行该语句。  由于网络活动需要一段时间，因此 `.execute()` 方法将返回 [`ListenableFuture<E>`][18]。

### <a name="filtering"></a>筛选返回的数据

以下查询执行从 **complete** 等于 **false** 的 **ToDoItem** 表返回所有项目。

    List<ToDoItem> result = mToDoTable
        .where()
        .field("complete").eq(false)
        .execute()
        .get();

**mToDoTable** 是对前面创建的移动服务表的引用。

对表引用使用 **where** 方法调用定义筛选器。 然后，依次执行 **where** 方法、**field** 方法和用于指定逻辑谓词的方法。 可能的谓词方法包括 **eq**（等于）、**ne**（不等于）、**gt**（大于）、**ge**（大于或等于）、**lt**（小于）、**le**（小于或等于）。 使用这些方法可将数字和字符串字段与特定的值相比较。

可以按日期筛选。 使用以下方法可以比较整个日期字段或日期的某些部分：**year**、**month**、**day**、**hour**、**minute** 和 **second**。 以下示例针对 *截止日期* 等于 2013 的项添加一个筛选器。

    List<ToDoItem> results = MToDoTable
        .where()
        .year("due").eq(2013)
        .execute()
        .get();

以下方法支持对字符串字段使用复杂筛选器：**startsWith**、**endsWith**、**concat**、**subString**、**indexOf**、**replace**、**toLower**、**toUpper**、**trim** 和 **length**。 以下示例筛选 *text* 列以“PRI0”开头的表行。

    List<ToDoItem> results = mToDoTable
        .where()
        .startsWith("text", "PRI0")
        .execute()
        .get();

支持对数字字段使用以下运算符方法：**add**、**sub**、**mul**、**div**、**mod**、**floor**、**ceiling** 和 **round**。 以下示例筛选其中的 **duration** 为偶数的表行。

    List<ToDoItem> results = mToDoTable
        .where()
        .field("duration").mod(2).eq(0)
        .execute()
        .get();

可将谓词与以下逻辑方法相组合：**and**、**or** 和 **not**。 以下示例组合前述两个示例。

    List<ToDoItem> results = mToDoTable
        .where()
        .year("due").eq(2013).and().startsWith("text", "PRI0")
        .execute()
        .get();

组和嵌套逻辑运算符：

    List<ToDoItem> results = mToDoTable
        .where()
        .year("due").eq(2013)
        .and(
            startsWith("text", "PRI0")
            .or()
            .field("duration").gt(10)
        )
        .execute().get();

有关筛选操作的更详细介绍和示例，请参阅 [Exploring the richness of the Android client query model][20]（探索 Android 客户端查询模型的丰富功能）。

### <a name="sorting"></a>对返回的数据进行排序

以下代码返回 **ToDoItems** 表中的所有项，返回的结果已按 *text* 字段的升序排序。 *mToDoTable* 是对前面创建的后端表的引用：

    List<ToDoItem> results = mToDoTable
        .orderBy("text", QueryOrder.Ascending)
        .execute()
        .get();

**orderBy** 方法的第一个参数是与要排序的字段名称相同的字符串。 第二个参数使用 **QueryOrder** 枚举来指定是按升序还是按降序排序。  如果使用 ***where*** 方法筛选，则必须在调用 ***orderBy*** 方法之前调用 ***where*** 方法。

### <a name="selection"></a>选择特定列

以下代码演示了如何返回 **ToDoItems** 表中的所有项，但只显示 **complete** 和 **text** 字段。 **mToDoTable** 是对前面创建的后端表的引用。

    List<ToDoItemNarrow> result = mToDoTable
        .select("complete", "text")
        .execute()
        .get();

Select 函数的参数是要返回的表列的字符串名称。  **select** 方法需要接在 **where** 和 **orderBy** 等方法的后面。 它可以后接 **skip** 和 **top** 等分页方法。

### <a name="paging"></a>在页中返回数据

数据**始终**在页面中返回。  返回的最大记录数由服务器设置。  如果客户端请求更多记录，服务器将返回最大记录数。  默认情况下，服务器上的最大页面大小为 50 个记录。

第一个示例说明如何选择表中的前 5 个项。 该查询返回 **ToDoItems**表中的项。 **mToDoTable** 是对前面创建的后端表的引用：

    List<ToDoItem> result = mToDoTable
        .top(5)
        .execute()
        .get();

以下查询跳过前 5 个项，返回接下来的 5 个项：

    List<ToDoItem> result = mToDoTable
        .skip(5).top(5)
        .execute()
        .get();

如果想要获取表中的所有记录，请实现循环访问所有页面的代码：

    List<MyDataModel> results = new List<MyDataModel>();
    int nResults;
    do {
        int currentCount = results.size();
        List<MyDataModel> pagedResults = mDataTable
            .skip(currentCount).top(500)
            .execute().get();
        nResults = pagedResults.size();
        if (nResults > 0) {
            results.addAll(pagedResults);
        }
    } while (nResults > 0);

使用此方法请求所有记录会针对移动应用后端至少创建两个请求。

> [AZURE.TIP]
> 选择适当的页面大小可在执行请求时使用的内存量、带宽用量以及完全接收数据时产生的延迟之间进行平衡。  默认值（50 个记录）适用于所有设备。  如果以独占方式在较大的内存设备上运行，最高可将页面大小增大到 500。  我们发现，将页面大小增大到超过 500 条记录会导致出现不可接受的延迟，以及消耗大量内存的问题。

### <a name="chaining"></a>如何连接查询方法

用于查询后端表的方法是可以连接的。 通过链接查询方法，可以选择已排序并分页的筛选行的特定列。 可以创建复杂的逻辑筛选器。  每个查询方法都会返回一个查询对象。 若要结束方法序列并真正运行查询，可以调用 **execute** 方法。 例如：

    List<ToDoItem> results = mToDoTable
            .where()
            .year("due").eq(2013)
            .and(
                startsWith("text", "PRI0").or().field("duration").gt(10)
            )
            .orderBy(duration, QueryOrder.Ascending)
            .select("id", "complete", "text", "duration")
            .skip(200).top(100)
            .execute()
            .get();

链接的查询方法必须按照以下顺序排序：

1. 筛选 (**where**) 方法。
2. 排序 (**orderBy**) 方法。
3. 选择 (**select**) 方法。
4. 分页（**skip** 和 **top**）方法。

## <a name="binding"></a>将数据绑定到用户界面

数据绑定涉及到三个组件：

- 数据源
- 屏幕布局
- 将两者关联起来的适配器。

以下示例代码将移动应用 SQL Azure 表 **ToDoItem** 中的数据返回到一个数组中。此活动是数据应用程序的常见模式。数据库查询通常会返回行的集合，客户端在列表或数组中获取该集合。在此示例中，该数组就是数据源。

代码指定屏幕布局，用于定义设备中显示的数据视图。  这两者与适配器绑定在一起，在此代码中，适配器是 **ArrayAdapter&lt;ToDoItem&gt;** 类的扩展。

#### <a name="layout"></a>定义布局

布局由多个 XML 代码段定义。以某个现有布局为例，以下代码表示要在其中填充服务器数据的 **ListView**。

    <ListView
        android:id="@+id/listViewToDo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        tools:listitem="@layout/row_list_to_do" >
    </ListView>

在上述代码中， *listitem* 属性指定列表中单个行的布局 ID。 此代码指定复选框及其关联文本，并针对列表中的每个项进行一次实例化。 此布局不显示 **id** 字段，如果使用更复杂的布局，则会在屏幕中指定更多的字段。 以下代码摘自 **row_list_to_do.xml** 文件。

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

#### <a name="adapter"></a>定义适配器

由于视图的数据源是 **ToDoItem** 的数组，因此我们需要基于 **ArrayAdapter&lt;ToDoItem&gt;** 类子类化适配器。 此子类将使用 **row_list_to_do** 布局为每个 **ToDoItem** 生成一个视图。

在代码中，我们可以定义以下类作为 **ArrayAdapter&lt;E&gt;** 类的扩展：

    public class ToDoItemAdapter extends ArrayAdapter<ToDoItem> {
    }

替代适配器的 **getView** 方法。例如：

    @Override
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

        checkBox.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View arg0) {
                if (checkBox.isChecked()) {
                    checkBox.setEnabled(false);
                    if (mContext instanceof ToDoActivity) {
                        ToDoActivity activity = (ToDoActivity) mContext;
                        activity.checkItem(currentItem);
                    }
                }
            }
        });


		return row;
	}

在活动中创建此类的实例，如下所示：

	ToDoItemAdapter mAdapter;
	mAdapter = new ToDoItemAdapter(this, R.layout.row_list_to_do);

ToDoItemAdapter 构造函数的第二个参数是对布局的引用。我们现在可以实例化 **ListView** 并将适配器分配到 **ListView**。

	ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
	listViewToDo.setAdapter(mAdapter);

#### <a name="use-adapter"></a>使用适配器绑定到 UI

现在，你可以使用数据绑定了。下面的代码说明如何获取表中的项，以及使用返回的项填充本地适配器。

    public void showAll(View view) {
        AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>(){
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final List<ToDoItem> results = mToDoTable.execute().get();
                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            mAdapter.clear();
                            for (ToDoItem item : results) {
                                mAdapter.add(item);
                            }
                        }
                    });
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        };
		runAsyncTask(task);
    }

请在每次修改 **ToDoItem** 表时调用适配器。修改是逐条记录进行的，因此，要处理的是单个行而不是一个集合。插入项时，需要对适配器调用 **add** 方法；删除项时，需要调用 **remove** 方法。


可以在 [Android 快速入门项目][21]中找到完整示例。

## <a name="inserting"></a>将数据插入后端

实例化 *ToDoItem* 类的实例并设置其属性。

	ToDoItem item = new ToDoItem();
	item.text = "Test Program";
	item.complete = false;

然后，使用 **insert()** 插入对象：

    ToDoItem entity = mToDoTable
        .insert(item)       // Returns a ListenableFuture<ToDoItem>
        .get();

返回的实体将匹配插入后端表的数据，包括 ID 和后端上设置的任何其他值（例如 `createdAt`、`updatedAt` 和 `version` 字段）。

移动应用表需要名为 **id** 的主键列。默认情况下，此列是字符串。ID 列的默认值为 GUID。可以提供其他唯一的值，例如电子邮件地址或用户名。如果没有为插入的记录提供字符串 ID 值，后端会生成新的 GUID。

字符串 ID 值提供以下优势：

+ 无需往返访问数据库即可生成 ID。
+ 更方便地合并不同表或数据库中的记录。
+ ID 值能够更好地与应用程序的逻辑集成。

若要支持脱机同步， **必需** 提供字符串 ID 值。  将 ID 存储到后端数据库后，无法对它进行更改。

## <a name="updating"></a>更新移动应用中的数据

若要更新表中的数据，请将新对象传递给 **update()** 方法。

    mToDoTable
        .update(item)   // Returns a ListenableFuture<ToDoItem>
        .get();

在此示例中，*item* 是对 *ToDoItem* 表中某个行的引用，该表包含一些更改。
具有相同 **id** 的行会更新。

## <a name="deleting"></a>删除移动应用中的数据

以下代码演示如何通过指定数据对象来删除表中的数据。

    mToDoTable
        .delete(item);

也可以通过指定要删除的行的 **id** 字段来删除项。

	String myRowId = "2FA404AB-E458-44CD-BC1B-3BC847EF0902";
   	mToDoTable.delete(myRowId);

## <a name="lookup"></a>按 ID 查找特定项

使用 **lookUp()** 方法查找具有特定 **id** 字段的项：

	ToDoItem result = mToDoTable
						.lookUp("0380BAFB-BCFF-443C-B7D5-30199F730335")
						.get();

##<a name="untyped"></a>如何处理非类型化数据

使用非类型化编程模型可以准确控制 JSON 序列化。在某些常见方案中，可能会希望使用非类型化编程模型。例如，如果后端表中包含许多列，但只需要引用其中几个列。类型化模型需要在数据类中定义移动应用表的所有列。

用于访问数据的大多数 API 调用都与类型化编程调用类似。 主要差别在于，在非类型化模型中，要对 **MobileServiceJsonTable** 对象而不是 **MobileServiceTable** 对象调用方法。

### <a name="json_instance"></a>创建非类型化表的实例

与使用类型化模型相似，首先需要获取表引用，不过，此时该引用的是一个 **MobileServicesJsonTable** 对象。对客户端的实例调用 **getTable** 方法来获取引用：

    private MobileServiceJsonTable mJsonToDoTable;
	//...
    mJsonToDoTable = mClient.getTable("ToDoItem");

创建 **MobileServiceJsonTable** 的实例后，它就几乎具有与类型化编程模型所能使用的 API 相同的 API。在某些情况下，这些方法会采用非类型化参数，而不采用类型化参数。

### <a name="json_insert"></a>插入到非类型化的表
以下代码演示了如何执行插入。 第一步是创建属于 [gson][3] 库的 [JsonObject][1]。

	JsonObject jsonItem = new JsonObject();
	jsonItem.addProperty("text", "Wake up");
	jsonItem.addProperty("complete", false);

然后，使用 **insert()** 将非类型化对象插入表。

    JsonObject insertedItem = mJsonToDoTable
        .insert(jsonItem)
        .get();

如果需要获取所插入对象的 ID，请使用 **getAsJsonPrimitive()** 方法调用。

    String id = insertedItem.getAsJsonPrimitive("id").getAsString();

### <a name="json_delete"></a>从非类型化表中删除
以下代码演示了如何删除一个实例，在本例中，该实例就是我们在前一个 **insert** 示例中创建的 *JsonObject* 的实例。 该代码与类型化案例相同，但方法具有不同的签名，因为它引用了 **JsonObject**。

    mToDoTable.delete(insertedItem);

还可以使用某个实例的 ID 来直接删除该实例：

		 mToDoTable.delete(ID);

### <a name="json_get"></a>从非类型化表中返回所有行
以下代码演示了如何检索整个表。 由于使用的是 Json 数据表，你可以选择性地只检索某些表的列。

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
                            for (JsonElement item : results) {
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

类型化模型使用的相同筛选设置（筛选和分页方法）适用于非类型化模型。

## <a name="offline-sync"></a>实现脱机同步

Azure 移动应用客户端 SDK 还可使用 SQLite 数据库在本地存储服务器数据的副本，从而实现脱机数据同步。  无需建立移动连接即可针对脱机表执行操作。  脱机同步有助于提高恢复能力和性能，代价是用于解决冲突的逻辑变得更复杂。  Azure 移动应用客户端 SDK 实现以下功能：

* 增量同步：仅下载已更新的和新的记录，从而减少了带宽和内存消耗。
* 乐观并发：假设操作成功。  冲突解决推迟到在服务器上执行更新之后。
* 冲突解决：SDK 检测服务器上发生的有冲突更改，并提供挂钩来提醒用户。
* 软删除：将已删除的记录标记为已删除，使其他设备能够更新其脱机缓存。

### <a name="initialize-offline-sync"></a>初始化脱机同步

在使用每个脱机表之前，必须先在脱机缓存中定义该表。  通常，在创建客户端之后立即执行表定义：

    AsyncTask<Void, Void, Void> initializeStore(MobileServiceClient mClient)
        throws MobileServiceLocalStoreException, ExecutionException, InterruptedException
    {
        AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>() {
            @Override
            protected void doInBackground(Void... params) {
                try {
                    MobileServiceSyncContext syncContext = mClient.getSyncContext();
                    if (syncContext.isInitialized()) {
                        return null;
                    }
                    SQLiteLocalStore localStore = new SQLiteLocalStore(mClient.getContext(), "offlineStore", null, 1);

                    // Create a table definition.  As a best practice, store this with the model definition and return it via
                    // a static method
                    Map<String, ColumnDataType> toDoItemDefinition = new HashMap<String, ColumnDataType>();
                    toDoItemDefinition.put("id", ColumnDataType.String);
                    toDoItemDefinition.put("complete", ColumnDataType.Boolean);
                    toDoItemDefinition.put("text", ColumnDataType.String);
                    toDoItemDefinition.put("version", ColumnDataType.String);
                    toDoItemDefinition.put("updatedAt", ColumnDataType.DateTimeOffset);

                    // Now define the table in the local store
                    localStore.defineTable("ToDoItem", toDoItemDefinition);

                    // Specify a sync handler for conflict resolution
                    SimpleSyncHandler handler = new SimpleSyncHandler();

                    // Initialize the local store
                    syncContext.initialize(localStore, handler).get();
                } catch (final Exception e) {
                    createAndShowDialogFromTask(e, "Error");
                }
                return null;
            }
        };
        return runAsyncTask(task);
    }

### <a name="obtain-a-reference-to-the-offline-cache-table"></a>获取对脱机缓存表的引用

对于联机表，可以使用 `.getTable()`。  对于脱机表，可以使用 `.getSyncTable()`：

    MobileServiceTable<ToDoItem> mToDoTable = mClient.getSyncTable("ToDoItem", ToDoItem.class);

可用于联机表的所有方法（包括筛选、排序、分页、插入数据、更新数据和删除数据）同样适用于脱机表。

### <a name="synchronize-the-local-offline-cache"></a>同步本地脱机缓存

同步在应用的控制范围内。  下面是一个示例同步方法：

    private AsyncTask<Void, Void, Void> sync(MobileServiceClient mClient) {
        AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>(){
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    MobileServiceSyncContext syncContext = mClient.getSyncContext();
                    syncContext.push().get();
                    mToDoTable.pull(null, "todoitem").get();
                } catch (final Exception e) {
                    createAndShowDialogFromTask(e, "Error");
                }
                return null;
            }
        };
        return runAsyncTask(task);
    }

如果为 `.pull(query, queryname)` 方法提供了查询名称，则使用增量同步，以便仅返回自上次成功完成提取以来创建或更改的记录。

### <a name="handle-conflicts-during-offline-synchronization"></a>在脱机同步期间处理冲突

如果在执行 `.push()` 操作期间发生冲突，将引发 `MobileServiceConflictException`。   服务器发出的项嵌入在异常中，可以通过针对该异常执行 `.getItem()` 来检索该项。  通过针对 MobileServiceSyncContext 对象调用以下项来调整推送：

*  `.cancelAndDiscardItem()`
*  `.cancelAndUpdateItem()`
*  `.updateOperationAndItem()`

根据需要标记所有冲突后，可以再次调用 `.push()` 来解决所有冲突。

## <a name="custom-api"></a>调用自定义 API

自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

从 Android 客户端调用 **invokeApi** 方法，以调用自定义 API 终结点。 以下示例演示了如何调用名为 **completeAll** 的 API 终结点，从而返回名为 **MarkAllResult** 的集合类。

	public void completeItem(View view) {

	    ListenableFuture<MarkAllResult> result = mClient.invokeApi( "completeAll", MarkAllResult.class );

	    	Futures.addCallback(result, new FutureCallback<MarkAllResult>() {
	    		@Override
	    		public void onFailure(Throwable exc) {
	    			createAndShowDialog((Exception) exc, "Error");
	    		}

	    		@Override
	    		public void onSuccess(MarkAllResult result) {
	    			createAndShowDialog(result.getCount() + " item(s) marked as complete.", "Completed Items");
	                refreshItemsFromTable();
	    		}
	    	});
	    }

**invokeApi** 方法在客户端上调用，该客户端向新的自定义 API 发送 POST 请求。与任何错误相同，自定义 API 返回的结果也显示在消息对话框中。使用其他版本的 **invokeApi** 可以选择性地在请求正文中发送对象、指定 HTTP 方法，以及随请求一起发送查询参数。此外还提供了非类型化的 **invokeApi** 版本。

## <a name="authentication"></a>向应用添加身份验证

教程已详细说明如何添加这些功能。

应用服务支持使用各种外部标识提供者[对应用用户进行身份验证](/documentation/articles/app-service-mobile-android-get-started-users/)，这些提供者包括：Microsoft 帐户和 Azure Active Directory。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在后端中使用已经过身份验证的用户的标识来实施授权规则。

支持两种身份验证流: **服务器**流和**客户端**流。服务器流依赖于标识提供者的 Web 界面，因此可提供最简单的身份验证体验。无需其他 SDK 即可实现服务器流身份验证。服务器流身份验证不会与移动设备深度集成，因此仅建议用于概念验证方案。

客户端流依赖于标识提供者提供的 SDK，因此允许与设备特定的功能（例如单一登录）进行更深入的集成。例如，可以将 Facebook SDK 集成到移动应用程序中。移动客户端会切换到 Facebook 应用，并在换回移动应用前确认已登录。

在应用中启用身份验证需要执行以下四个步骤：

- 向标识提供者注册应用，以进行身份验证。
- 配置应用服务后端。
- 限制只有应用服务后端上经过身份验证的用户才能拥有表权限。
- 将身份验证代码添加到应用。

你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。还可以使用已经过身份验证的用户的 SID 来修改请求。有关详细信息，请查看[身份验证入门]和服务器 SDK 操作方法文档。

### <a name="caching"></a>身份验证：服务器流

以下代码使用 MicrosoftAccount 启动服务器流登录过程。  由于 MicrosoftAccount 的安全要求，需要指定其他配置：

    MobileServiceUser user = mClient.login(MobileServiceAuthenticationProvider.MicrosoftAccount, "{url_scheme_of_your_app}", MICROSOFTACCOUNT_LOGIN_REQUEST_CODE);

此外，请将以下方法添加到主活动类：

    // You can choose any unique number here to differentiate auth providers from each other. Note this is the same code at login() and onActivityResult().
    public static final int MICROSOFTACCOUNT_LOGIN_REQUEST_CODE = 1;

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        // When request completes
        if (resultCode == RESULT_OK) {
            // Check the request code matches the one we send in the login request
            if (requestCode == MICROSOFTACCOUNT_LOGIN_REQUEST_CODE) {
                MobileServiceActivityResult result = mClient.onActivityResult(data);
                if (result.isLoggedIn()) {
                    // login succeeded
                    createAndShowDialog(String.format("You are now logged in - %1$2s", mClient.getCurrentUser().getUserId()), "Success");
                    createTable();
                } else {
                    // login failed, check the error message
                    String errorMessage = result.getErrorMessage();
                    createAndShowDialog(errorMessage, "Error");
                }
            }
        }
    }

在主活动中定义的 `MICROSOFTACCOUNT_LOGIN_REQUEST_CODE` 用于 `login()` 方法，并在 `onActivityResult()` 方法中使用。  可以选择任意唯一编号，只要在 `login()` 方法和 `onActivityResult()` 方法中使用相同的编号即可。  如果将客户端代码抽象化为服务适配器（如前所示），应在服务适配器上调用相应的方法。

还需要为 customtabs 配置项目。  首先指定重定向 URL。  将以下代码片段添加到 `AndroidManifest.xml`：

    <activity android:name="com.microsoft.windowsazure.mobileservices.authentication.RedirectUrlActivity">
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="{url_scheme_of_your_app}" android:host="easyauth.callback"/>
        </intent-filter>
    </activity>

将 **redirectUriScheme** 添加到应用程序的 `build.gradle` 文件：

    android {
        buildTypes {
            release {
                // … …
                manifestPlaceholders = ['redirectUriScheme': '{url_scheme_of_your_app}://easyauth.callback']
            }
            debug {
                // … …
                manifestPlaceholders = ['redirectUriScheme': '{url_scheme_of_your_app}://easyauth.callback']
            }
        }
    }

最后，将 `com.android.support:customtabs:23.0.1` 添加到 `build.gradle` 文件中的依赖项列表：

    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])
        compile 'com.google.code.gson:gson:2.3'
        compile 'com.google.guava:guava:18.0'
        compile 'com.android.support:customtabs:23.0.1'
        compile 'com.squareup.okhttp:okhttp:2.5.0'
        compile 'com.microsoft.azure:azure-mobile-android:3.2.0@aar'
        compile 'com.microsoft.azure:azure-notifications-handler:1.0.1@jar'
    }

可以使用 **getUserId** 方法从 **MobileServiceUser** 获取已登录用户的 ID。 有关如何使用 Futures 调用异步登录 API 的示例，请参阅 [Get started with authentication]（身份验证入门）。

> [AZURE.WARNING]
> 所述的 URL 方案区分大小写。  请确保出现的所有 `{url_scheme_of_you_app}` 的大小写匹配。

### <a name="caching"></a>缓存身份验证令牌

缓存身份验证令牌需要将用户 ID 和身份验证令牌存储在设备本地。 下次启动应用时，只需检查缓存，如果这些值存在，则可以跳过登录过程，并使用这些数据重新进入客户端。 但是，这些数据是敏感的，为安全起见，应该以加密形式存储，以防手机失窃。  可以在 [缓存身份验证令牌][7]部分中了解有关缓存身份验证令牌的完整示例。

尝试使用过期的令牌时，会收到“401 未授权”  响应。 可以使用筛选器处理身份验证错误。  筛选器会截获对应用服务后端提出的请求。 筛选器代码会测试 401 响应，触发登录进程，然后恢复生成 401 响应的请求。

### <a name="refresh"></a>使用刷新令牌

Azure 应用服务身份验证和授权返回的令牌定义了一小时的生存期。  在此期限过后，必须重新验证用户的身份。  如果使用通过客户端流身份验证收到的长生存期令牌，则可以使用相同的令牌在 Azure 应用服务身份验证和授权中重新进行身份验证。  另外，还会生成一个具有新生存期的 Azure 应用服务令牌。

还可以将提供程序注册为使用刷新令牌。  刷新令牌不一定始终可用。  需要指定其他配置：

* 对于 **Azure Active Directory**，请为 Azure Active Directory 应用配置客户端机密。  配置 Azure Active Directory 身份验证时，请在 Azure 应用服务中指定客户端机密。  调用 `.login()` 时，请传递 `response_type=code id_token` 作为参数：

        HashMap<String, String> parameters = new HashMap<String, String>();
        parameters.put("response_type", "code id_token");
        MobileServiceUser user = mClient.login
            MobileServiceAuthenticationProvider.AzureActiveDirectory,
            "{url_scheme_of_your_app}",
            AAD_LOGIN_REQUEST_CODE,
            parameters);

* 对于 **Microsoft 帐户**，请选择 `wl.offline_access` 范围。

若要刷新令牌，请调用 `.refreshUser()`：

    MobileServiceUser user = mClient
        .refreshUser()  // async - returns a ListenableFuture<MobileServiceUser>
        .get();

最佳做法是创建一个筛选器用于检测来自服务器的 401 响应，并尝试刷新用户令牌。

## <a name="log-in-with-client-flow-authentication"></a>使用客户端流身份验证登录

使用客户端流身份验证登录的一般过程如下：

* 像配置服务器流身份验证一样配置 Azure 应用服务身份验证和授权。
* 集成用于身份验证的身份验证提供程序 SDK，以生成访问令牌。
* 按如下所示调用 `.login()` 方法：

        JSONObject payload = new JSONObject();
        payload.put("access_token", result.getAccessToken());
        ListenableFuture<MobileServiceUser> mLogin = mClient.login("{provider}", payload.toString());
        Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
            @Override
            public void onFailure(Throwable exc) {
                exc.printStackTrace();
            }
            @Override
            public void onSuccess(MobileServiceUser user) {
                Log.d(TAG, "Login Complete");
            }
        });

将 `onSuccess()` 方法替换为成功登录后要使用的任何代码。  `{provider}` 字符串是有效的提供程序：**aad** (Azure Active Directory) 或 **microsoftaccount** 。  如果已实现自定义身份验证，则还可以使用自定义身份验证提供程序标记。

### <a name="adal"></a>使用 Active Directory 身份验证库 (ADAL) 对用户进行身份验证

可以借助 Active Directory 身份验证库 (ADAL) 使用 Azure Active Directory 将用户登录到应用程序。 使用客户端流登录通常比使用 `loginAsync()` 方法更有利，因为它提供更直观的 UX 风格，并允许其他自定义。

1. 根据[如何为 Active Directory 登录配置应用服务][22]教程的说明，为 AAD 登录配置移动应用。 请务必完成注册本机客户端应用程序的可选步骤。
2. 通过修改 build.gradle 文件并包含以下定义来安装 ADAL：


	repositories {
	    mavenCentral()
	    flatDir {
	        dirs 'libs'
	    }
	    maven {
	        url "YourLocalMavenRepoPath\\.m2\\repository"
	    }
	}
	packagingOptions {
	    exclude 'META-INF/MSFTSIG.RSA'
	    exclude 'META-INF/MSFTSIG.SF'
	}
	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    compile('com.microsoft.aad:adal:1.1.1') {
	        exclude group: 'com.android.support'
	    } // Recent version is 1.1.1
	    compile 'com.android.support:support-v4:23.0.0'
	}

3. 将以下代码添加到应用程序并进行以下替换：

* 将 **INSERT-AUTHORITY-HERE** 替换为在其中预配应用程序的租户的名称。格式应为 https://login.chinacloudapi.cn/contoso.partner.onmschina.cn。

* 将 **INSERT-RESOURCE-ID-HERE** 替换移动应用后端的客户端 ID。 可以在门户中“Azure Active Directory 设置”下面的“高级”选项卡获取此客户端 ID。

* 将 **INSERT-CLIENT-ID-HERE** 替换为从本机客户端应用程序复制的客户端 ID。

* 将 **INSERT-REDIRECT-URI-HERE** 替换为站点的 _/.auth/login/done_ 终结点（使用 HTTPS 方案）。此值应类似于 _https://contoso.chinacloudsites.cn/.auth/login/done_ 。

		private AuthenticationContext mContext;
		private void authenticate() {
		String authority = "INSERT-AUTHORITY-HERE";
		String resourceId = "INSERT-RESOURCE-ID-HERE";
		String clientId = "INSERT-CLIENT-ID-HERE";
		String redirectUri = "INSERT-REDIRECT-URI-HERE";
		try {
		    mContext = new AuthenticationContext(this, authority, true);
		    mContext.acquireToken(this, resourceId, clientId, redirectUri, PromptBehavior.Auto, "", callback);
		} catch (Exception exc) {
		    exc.printStackTrace();
		}
		}
		private AuthenticationCallback<AuthenticationResult> callback = new AuthenticationCallback<AuthenticationResult>() {
		@Override
		public void onError(Exception exc) {
		    if (exc instanceof AuthenticationException) {
		        Log.d(TAG, "Cancelled");
		    } else {
		        Log.d(TAG, "Authentication error:" + exc.getMessage());
		    }
		}
		@Override
			public void onSuccess(AuthenticationResult result) {
		    if (result == null || result.getAccessToken() == null
		            || result.getAccessToken().isEmpty()) {
		        Log.d(TAG, "Token is empty");
		    } else {
		        try {
		            JSONObject payload = new JSONObject();
		            payload.put("access_token", result.getAccessToken());
		            ListenableFuture<MobileServiceUser> mLogin = mClient.login("aad", payload.toString());
		            Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
		                @Override
		                public void onFailure(Throwable exc) {
		                    exc.printStackTrace();
		                }
		                @Override
		                public void onSuccess(MobileServiceUser user) {
		            		Log.d(TAG, "Login Complete");
		                }
		            });
		        }
		        catch (Exception exc){
		            Log.d(TAG, "Authentication error:" + exc.getMessage());
		        }
		    }
		}
		};
		@Override
		protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		super.onActivityResult(requestCode, resultCode, data);
		if (mContext != null) {
		    mContext.onActivityResult(requestCode, resultCode, data);
		}
		}

## <a name="filters"></a> 调整客户端 - 服务器通信

客户端连接通常是使用 Android SDK 提供的底层 HTTP 库的基本 HTTP 连接。 有以下几个原因要更改：

* 希望使用备用 HTTP 库调整超时。
* 希望提供一个进度条。
* 希望添加自定义标头以支持 API 管理功能。
* 希望拦截失败的响应，以便可以重新认证。
* 希望将后端请求记录到分析服务。

### 使用备用 HTTP 库

在创建客户端引用后立即调用`.setAndroidHttpClientFactory()'方法。 例如，将连接超时设置为60秒（而不是默认的10秒）：

    mClient = new MobileServiceClient("https://myappname.azurewebsites.net");
    mClient.setAndroidHttpClientFactory(new OkHttpClientFactory() {
        @Override
        public OkHttpClient createOkHttpClient() {
            OkHttpClient client = new OkHttpClinet();
            client.setReadTimeout(60, TimeUnit.SECONDS);
            client.setWriteTimeout(60, TimeUnit.SECONDS);
            return client;
        }
    });

### <a name="implement-a-progress-filter"></a>实现进度筛选器

可以通过实现 `ServiceFilter` 来截获每个请求。  例如，以下代码将更新预先创建的进度栏：

    private class ProgressFilter implements ServiceFilter {
        @Override
        public ListenableFuture<ServiceFilterResponse> handleRequest(ServiceFilterRequest request, NextServiceFilterCallback next) {
            final SettableFuture<ServiceFilterResponse> resultFuture = SettableFuture.create();
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    if (mProgressBar != null) mProgressBar.setVisibility(ProgressBar.VISIBLE);
                }
            });

            ListenableFuture<ServiceFilterResponse> future = next.onNext(request);
            Futures.addCallback(future, new FutureCallback<ServiceFilterResponse>() {
                @Override
                public void onFailure(Throwable e) {
                    resultFuture.setException(e);
                }
                @Override
                public void onSuccess(ServiceFilterResponse response) {
                    runOnUiThread(new Runnable() {
                        @Override
                        pubic void run() {
                            if (mProgressBar != null)
                                mProgressBar.setVisibility(ProgressBar.GONE);
                        }
                    });
                    resultFuture.set(response);
                }
            });
            return resultFuture;
        }
    }

可按如下所示将此筛选器附加到客户端：

    mClient = new MobileServiceClient(applicationUrl).withFilter(new ProgressFilter());

### <a name="customize-request-headers"></a>自定义请求标头

使用以下 `ServiceFilter`，并像附加 `ProgressFilter` 一样附加筛选器：

    private class CustomHeaderFilter implements ServiceFilter {

        @Override
        public ListenableFuture<ServiceFilterResponse> handleRequest(
                    ServiceFilterRequest request,
                    NextServiceFilterCallback next) {

            runOnUiThread(new Runnable() {

                @Override
                public void run() {
                    request.addHeader("X-APIM-Router", "mobileBackend");
                }
            });

            SettableFuture<ServiceFilterResponse> result = SettableFuture.create();
            try {
                ServiceFilterResponse response = next.onNext(request).get();
                result.set(response);
            } catch (Exception exc) {
                result.setException(exc);
            }
        }
    }

### <a name="conversions"></a>配置自动序列化

可以使用 [gson][3] API，指定适用于每个列的转换策略。 在发送数据到 Azure 应用服务之前，Android 客户端库会在幕后使用 [gson][3] 将 Java 对象序列化为 JSON 数据。  下面的代码使用 **setFieldNamingStrategy()** 方法设置策略。 此示例删除初始字符（“m”），然后将每个字段名称的下一个字符小写。 例如，它会将“mId”变为“id”。  实现转换策略，减少在大多数字段中使用 `SerializedName()` 批注的需求。

    FieldNamingStrategy namingStrategy = new FieldNamingStrategy() {
        public String translateName(File field) {
            String name = field.getName();
            return Character.toLowerCase(name.charAt(1)) + name.substring(2);
        }
    }

    client.setGsonBuilder(
        MobileServiceClient
            .createMobileServiceGsonBuilder()
            .setFieldNamingStrategy(namingStategy)
    );

必须在使用 **MobileServiceClient** 创建移动客户端引用之前执行此代码。

<!-- URLs. -->
[Get started with Azure Mobile Apps]: /documentation/articles/app-service-mobile-android-get-started/
[ASCII control codes C0 and C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
[Mobile Services SDK for Android]: http://go.microsoft.com/fwlink/p/?LinkID=717033
[Azure 门户]: https://portal.azure.cn
[Azure 经典管理门户]: https://manage.windowsazure.cn/
[Get started with authentication]: /documentation/articles/app-service-mobile-android-get-started-users/
[身份验证入门]: /documentation/articles/app-service-mobile-android-get-started-users/
[2]: http://hashtagfail.com/post/44606137082/mobile-services-android-serialization-gson
[3]: http://go.microsoft.com/fwlink/p/?LinkId=290801
[4]: http://go.microsoft.com/fwlink/p/?LinkId=296840
[5]: /documentation/articles/app-service-mobile-android-get-started-push/
[6]: /documentation/articles/notification-hubs-push-notification-overview/#integration-with-app-service-mobile-apps
[7]: /documentation/articles/app-service-mobile-android-get-started-users/#cache-tokens
[8]: http://azure.github.io/azure-mobile-apps-android-client/com/microsoft/windowsazure/mobileservices/table/MobileServiceTable.html
[9]: http://azure.github.io/azure-mobile-apps-android-client/com/microsoft/windowsazure/mobileservices/MobileServiceClient.html
[10]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[11]: /documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/
[12]: http://azure.github.io/azure-mobile-apps-android-client/
[13]: /documentation/articles/app-service-mobile-android-get-started/#create-a-new-azure-mobile-app-backend
[14]: http://go.microsoft.com/fwlink/p/?LinkID=717034
[15]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#how-to-define-a-table-controller
[16]: /documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#TableOperations
[17]: https://developer.android.com/reference/java/util/UUID.html
[18]: https://github.com/google/guava/wiki/ListenableFutureExplained
[19]: http://www.odata.org/documentation/odata-version-3-0/
[20]: http://hashtagfail.com/post/46493261719/mobile-services-android-querying
[21]: https://github.com/Azure-Samples/azure-mobile-apps-android-quickstart
[22]: /documentation/articles/app-service-mobile-how-to-configure-active-directory-authentication/
[Future]: http://developer.android.com/reference/java/util/concurrent/Future.html
[AsyncTask]: http://developer.android.com/reference/android/os/AsyncTask.html

<!---HONumber=Mooncake_1114_2016-->