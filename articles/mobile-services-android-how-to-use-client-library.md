<properties linkid="develop-mobile-how-to-guides/work-with-android-client-library" urlDisplayName="Android Client Library" pageTitle="Working with the Mobile Services Android Client Library" metaKeywords="" description="Learn how to use an Android client for Azure Mobile Services." metaCanonical="" services="" documentationCenter="Mobile" title="How to use the Android client library for Mobile Services" authors="ricksal" solutions="" manager="" editor="" />

# 如何使用适用于移动服务的 Android 客户端库

<div class="dev-center-tutorial-selector sublanding"> 
  <a href="/zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/" title=".NET Framework">.NET Framework</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-html-js-client/" title="HTML/JavaScript">HTML/JavaScript</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-ios-client-library/" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-android-client-library/" title="Android" class="current">Android</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-xamarin-client-library/" title="Xamarin">Xamarin</a>
</div>

本指南说明如何使用适用于 Azure 移动服务的 Android 客户端执行常见任务。所述的任务包括：查询数据；插入、更新和删除数据；对用户进行身份验证；处理错误；自定义客户端。如果你是第一次使用移动服务，最好先完成[移动服务快速入门][]。快速入门教程可帮助你配置帐户并创建第一个移动服务。

这些示例用 Java 编写，需要安装[移动服务 SDK][]。本教程还要求安装 [Android SDK][]，其中包含 Eclipse 集成开发环境 (IDE) 和 Android 开发人员工具 (ADT) 插件。移动服务 SDK 支持 Android 2.2 或更高版本，但我们建议针对 Android 4.2 或更高版本生成应用程序。

## 目录

-   [什么是移动服务][]
-   [概念][]
-   [安装与先决条件][]
-   [如何：创建移动服务客户端][]
-   [如何：创建表引用][]

    -   [API 结构][]
-   [如何：从移动服务查询数据][]

    -   [筛选返回的数据][]
    -   [为返回的数据排序][]
    -   [在页中返回数据][]
    -   [选择特定的列][]
    -   [如何：连接查询方法][]
-   [如何：在移动服务中插入数据][]
-   [如何：在移动服务中更新数据][]
-   [如何：在移动服务中删除数据][]
-   [如何：查找特定的项][]
-   [如何：处理非类型化数据][]
-   [如何：将数据绑定到用户界面][]

    -   [如何：定义布局][]
    -   [如何：定义适配器][]
    -   [如何：使用适配器][]
-   [如何：对用户进行身份验证][]

    -   [缓存身份验证令牌][]
-   [如何：处理错误][]
-   [如何：自定义客户端][]

    -   [自定义请求标头][]
    -   [自定义序列化][]
-   [后续步骤][]

[WACOM.INCLUDE [mobile-services-concepts][]]

<a name="setup"></a>
## 安装安装与先决条件

假设你已创建一个移动服务和一个表。有关详细信息，请参阅[创建表][]。在本主题使用的代码中，我们假设表的名称为 *ToDoItem*，其中包含以下列：

<ul>
<li>id</li>
<li>text</li>
<li>complete</li>
<li>due</li>
<li>duration</li>
</ul>

相应的类型化客户端对象如下：

    public class TodoItem
    private String id;
    private String text;
    private Boolean complete;
    private Date due
    private Integer duration;
    }

启用动态架构后，Azure 移动服务将基于 insert 或 update 请求中的对象自动生成新列。有关详细信息，请参阅[动态架构][]。

<a name="create-client"></a>
## 创建移动服务客户端如何：创建移动服务客户端

以下代码将创建用于访问移动服务的 [MobileServiceClient][] 对象。

            MobileServiceClient mClient = new MobileServiceClient(
    "MobileServiceUrl", // Replace with the above Site URL
    "AppKey",           // replace with the Application Key 
    this)

在上面的代码中，请将 `MobileServiceUrl` 和 `AppKey` 依次替换为移动服务 URL 和应用程序密钥。在 Azure 管理门户中选择你的移动服务，然后单击*“仪表板”*即可获取这两个值。

<a name="instantiating"></a>
## 创建表引用如何：创建表引用

在移动服务中查询或修改数据的最简单方法就是使用*类型化编程模型*，因为 Java 是强类型化语言（稍后我们将会介绍*类型化*模型）。在客户端与移动服务之间发送数据时，此模型将使用 [gson][] 库向 JSON 提供无缝序列化和反序列化：开发人员无需执行任何操作，框架能够处理一切。

查询或修改数据所要执行的第一项操作就是通过对 ["MobileServiceClient"][MobileServiceClient] 调用 "getTable" 方法来创建一个 [MobileServiceTable][] 对象。下面是此方法的两个重载：

    public class MobileServiceClient {
    public <E> MobileServiceTable<E> getTable(Class<E> clazz);
    public <E> MobileServiceTable<E> getTable(String name, Class<E> clazz);
    }

在以下代码中，*mClient* 是对移动服务客户端的引用。

如果类名称与表名称相同，则使用[第一个重载][]：

        MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable(ToDoItem.class);

如果表名称与类型名称不同，则使用[第二个重载][]：

        MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable("ToDoItemBackup", ToDoItem.class);

<a name="api"></a>
### API 结构

移动服务表操作使用异步回调模型。涉及查询和操作（例如插入、更新和删除）的方法全都包含一个用作回调对象的参数。此对象始终包含 "OnCompleted" 方法。"onCompleted" 方法包含一个用作 "Exception" 对象的参数，你可以测试该对象，以确定方法调用是否成功。如果 "Exception" 对象为 null，则表示成功，否则，"Exception" 对象将描述失败原因。

存在多个不同的回调对象，要使用其中的哪个对象取决于你是要查询、修改还是删除数据。*onCompleted* 方法的参数根据该方法所属的回调对象的不同而异。

<a name="querying"></a>
## 查询数据如何：从移动服务查询数据

本部分介绍如何向移动服务发出查询。其中的小节介绍了排序、筛选和分页等不同操作。最后，我们将介绍如何将这些操作连接起来。

以下代码将返回 *ToDoItem* 表中的所有项。

        mToDoTable.execute(new TableQueryCallback<ToDoItem>() {
    public void onCompleted(List<ToDoItem> result, int count,
    Exception exception, ServiceFilterResponse response) {
    if (exception == null) {
    for (ToDoItem item :result) {
    Log.i(TAG, "Read object with ID " + item.id);  
                        }
                    }
                }
            });

类似于这样的查询将使用 ["TableQueryCallback\<E\>"][] 回调对象。

*result* 参数返回查询后生成的结果集，*exception* 测试的成功分支中的代码显示了如何分析各个行。

<a name="filtering"></a>
### 如何：筛选返回的数据

以下代码将返回 *complete* 字段等于 *false* 的 *ToDoItem* 表中的所有项。*mToDoTable* 是对前面创建的移动服务表的引用。

        mToDoTable.where().field("complete").eq(false)
    .execute(new TableQueryCallback<ToDoItem>() {
    public void onCompleted(List<ToDoItem> result, 
    int count, 
    Exception exception,
    ServiceFilterResponse response) {
    if (exception == null) {
    for (ToDoItem item :result) {
    Log.i(TAG, "Read object with ID " + item.id);  
                    }
                } 
            }
        });

通过对表引用执行 ["where"][] 方法调用来启动筛选器。然后，依次执行 ["field"][] 方法调用和用于指定逻辑谓词的方法调用。可能的谓词方法包括 ["eq"][]、["ne"][]、["gt"][]、["ge"][]、["lt"][]、["le"][]，等等。

执行这些操作便足以将数字和字符串字段与特定值进行比较。不过，你还可以执行其他许多操作。

例如，你可以按日期筛选。你可以比较整个日期字段，或者使用 ["year"][]、["month"][]、["day"][]、["hour"][]、["minute"][] 和 ["second"][] 等方法比较日期的一部分。以下代码片段将会针对“截止日期”等于 2013 的项添加一个筛选器"。

        mToDoTable.where().year("due").eq(2013)

你可以使用 ["startsWith"][]、["endsWith"][]、["concat"][]、["subString"][]、["indexOf"][]、["replace"][]、["toLower"][]、["toUpper"][]、["trim"][] 和 ["length"][] 等方法对字符串字段运行各种复杂筛选器。以下代码片段将会筛选 *text* 列以“PRI0”开头的表行。

        mToDoTable.where().startsWith("text", "PRI0")

还允许使用 ["add"][]、["sub"][]、["mul"][]、["div"][]、["mod"][]、["floor"][]、["ceiling"][] 和 ["round"][] 等方法对数字字段运行各种更复杂的筛选器。以下代码片段将会筛选其中的 *duration* 为偶数的表行。

        mToDoTable.where().field("duration").mod(2).eq(0)

你可以使用 ["and"][]、["or"][] 和 ["not"][] 等方法来组合谓词。以下代码片段将组合上面的两个示例。

        mToDoTable.where().year("due").eq(2013).and().startsWith("text", "PRI0")

你可以按照以下代码片段所示来组合与嵌套逻辑运算符：

        mToDoTable.where()
    .year("due").eq(2013)
    .and
    (startsWith("text", "PRI0").or().field("duration").gt(10))

有关筛选操作的更详细介绍和示例，请参阅[了解移动服务 Android 客户端查询模型的丰富功能][]。


<a name="sorting"></a>
### 如何：为返回的数据排序

以下代码将返回 *ToDoItems* 表中的所有项，返回的结果已按 *text* 字段的升序排序。*mToDoTable* 是对前面创建的移动服务表的引用。

        mToDoTable.orderBy("text", QueryOrder.Ascending)
    .execute(new TableQueryCallback<ToDoItem>() { 
    /* same implementation as above */ 
            }); 

["orderBy"][] 方法的第一个参数是与要排序的字段名称相同的字符串。

第二个参数使用 ["QueryOrder"][] 枚举来指定是按升序还是按降序排序。

请注意，如果你使用 "*where"* 方法筛选，则必须在调用 "*orderBy"* 方法之前调用 "*where"* 方法。

<a name="paging"></a>
### 如何：在页中返回数据

第一个示例演示了如何选择表中的前 5 个项。该查询将返回 *ToDoItems* 表中的项。*mToDoTable* 是对前面创建的移动服务表的引用。

        mToDoTable.top(5)
    .execute(new TableQueryCallback<ToDoItem>() {   
    public void onCompleted(List<ToDoItem> result, 
    int count,
    Exception exception, 
    ServiceFilterResponse response) {
    if (exception == null) {
    for (ToDoItem item :result) {
    Log.i(TAG, "Read object with ID " + item.id);  
                        }
                    } 
                }
            });

接下来，我们定义一个查询，以跳过前 5 个项，返回后 5 个项。

        mToDoTable.skip(5).top(5)
    .execute(new TableQueryCallback<ToDoItem>() {   
    // implement onCompleted logic here
            });

<a name="selecting"></a>
### 如何：选择特定的列

以下代码演示如何返回 *ToDoItems* 表中的所有项，但只显示 *complete* 和 *text* 字段。*mToDoTable* 是对前面创建的移动服务表的引用。

        mToDoTable.select("complete", "text")
    .execute(new TableQueryCallback<ToDoItem>() { 
    /* same implementation as above */ 
            }); 

在这里，select 函数的参数是要返回的表列的字符串名称。

["select"][] 方法需接在 ["where"][1] 和 ["orderBy"][2] 等方法（如果存在）的后面。它可以后接 ["top"][] 等方法。

<a name="chaining"></a>
### 如何：连接查询方法

可以连接用于查询移动服务表的方法。这样，你便可以执行多种操作，例如，选择已排序并分页的筛选行的特定列。你可以创建相当复杂的逻辑筛选器。

这种操作的工作原理是通过使用的查询方法返回 ["MobileServiceQuery\<T\>"][] 对象，随之又对这些对象调用更多的方法。若要结束方法序列并真正运行查询，你可以调用 ["execute"][] 方法。

在以下代码示例中，*mToDoTable* 是对移动服务 *ToDoItem* 表的引用。

        mToDoTable.where().year("due").eq(2013)
    .and().startsWith("text", "PRI0")
    .or().field("duration").gt(10)
    .select("id", "complete", "text", "duration")
    .orderBy(duration, QueryOrder.Ascending).top(20)                
    .execute(new TableQueryCallback<ToDoItem>() { 
    /* code to execute */ 
                });

将方法链接在一起时，最重要的是 *where* 方法和谓词必须出现在最前面。然后，你就可以按照最符合应用程序需求的顺序调用后续方法。

<a name="inserting"></a>
## 插入数据如何：在移动服务中插入数据

以下代码演示了如何在表中插入新行。

首先，实例化 *ToDoItem* 类的实例并设置该实例的属性。

        ToDoItem mToDoItem = new ToDoItem();
    mToDoItem.text = "Test Program";
    mToDoItem.complete = false;
    mToDoItem.duration = 5; 
        

接下来，调用 ["insert"][] 方法。

        mToDoTable.insert(mToDoItem, new TableOperationCallback<ToDoItem>() {
    public void onCompleted(ToDoItem entity, 
    Exception exception, 
    ServiceFilterResponse response) {   
    if (exception == null) {
    Log.i(TAG, "Read object with ID " + entity.id);  
                } 
            }
        });

对于 "insert" 操作，回调对象为 ["TableOperationCallback\<ToDoItem\>"][]。

"onCompleted" 方法的 entity 参数包含新插入的对象。成功代码显示了如何访问所插入行的 *id*。

移动服务支持为表 ID 使用唯一的自定义字符串值。这样，应用程序便可为移动服务表的 ID 列使用自定义值（如电子邮件地址或用户名）。例如，如果你想要根据电子邮件地址识别每条记录，可以使用以下 JSON 对象。

        ToDoItem mToDoItem = new ToDoItem();
    mToDoItem.id = "myemail@mydomain.com";
    mToDoItem.text = "Test Program";
    mToDoItem.complete = false;
    mToDoItem.duration = 5; 

如果将新记录插入到表时未提供字符串 ID 值，移动服务将为 ID 生成唯一值。

支持字符串 ID 为开发人员带来了以下优势

-   无需往返访问数据库即可生成 ID。
-   更方便地合并不同表或数据库中的记录。
-   ID 值能够更好地与应用程序的逻辑相集成。

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

-   控制字符：[0x0000-0x001F] 和 [0x007F-0x009F]。有关详细信息，请参阅 [ASCII 控制代码 C0 和 C1][]。
-   可打印字符："""(0x0022)、"+" (0x002B)、"/" (0x002F)、"?"(0x003F)、"\\" (0x005C)、"\`" (0x0060)
-   ID“.”和“..”

也可以为表使用整数 ID。若要使用整数 ID，必须使用 `mobile table create` 命令并结合 `--integerId` 选项创建表。应在适用于 Azure 的命令行界面 (CLI) 中使用此命令。有关使用 CLI 的详细信息，请参阅[用于管理移动服务表的 CLI][]。

<a name="updating"></a>
## 更新数据如何：在移动服务中更新数据

以下代码演示了如何更新表中的数据。在此示例中，*mToDoItem* 是对 *ToDoItem* 表中某个项的引用，我们将更新该项的 *duration* 属性。

        mToDoItem.duration = 5;
    mToDoTable.update(mToDoItem, new TableOperationCallback<ToDoItem>() {
    public void onCompleted(ToDoItem entity, 
    Exception exception, 
    ServiceFilterResponse response) {
    if (exception == null) {
    Log.i(TAG, "Read object with ID " + entity.id);  
                } 
            }
        });

请注意，回调对象和 *onCompleted* 方法的参数与执行插入时使用的相同。

<a name="deleting"></a>
## 删除数据如何：在移动服务中删除数据

以下代码演示了如何删除表中的数据。该代码使用对 ToDoItem 表中某个现有项的引用（在本例中为 *mToDoItem*）来删除该项。

        mToDoTable.delete(mToDoItem, new TableDeleteCallback() {
    public void onCompleted(Exception exception,
    ServiceFilterResponse response) {
    if (exception == null) {
    Log.i(TAG, "Object deleted");
                }
            }
        });

请注意，在执行 *delete* 时，回调对象为 ["TableDeleteCallback"][]，"onCompleted" 方法略有不同，因为它不返回表行。

以下代码演示了执行删除操作的另一种方法。该代码通过指定要删除的行的 ID 字段值（假设等于 "37BBF396-11F0-4B39-85C8-B319C729AF6D"）来删除 ToDoItem 表中的现有项。

        mToDoTable.delete("37BBF396-11F0-4B39-85C8-B319C729AF6D", new TableDeleteCallback() {
    public void onCompleted(Exception exception, 
    ServiceFilterResponse response) {
    if (exception == null) {
    Log.i(TAG, "Object deleted");
                }
            }
        });

<a name="lookup"></a>
## 查找数据如何：查找特定的项

有时，你需要按 *id* 查找特定的项，这一点不像查询，因为查询通常会返回满足某些条件的项集合。以下代码演示了如何查找 *id* 等于 "37BBF396-11F0-4B39-85C8-B319C729AF6D" 的项。

        mToDoTable.lookUp("37BBF396-11F0-4B39-85C8-B319C729AF6D", new TableOperationCallback<ToDoItem>() {
    public void onCompleted(item entity, Exception exception,
    ServiceFilterResponse response) {
    if (exception == null) {
    Log.i(TAG, "Read object with ID " + entity.id);    
                }
            }
        });

<a name="untyped"></a>
## 处理非类型化数据如何：处理非类型化数据

使用非类型化编程模型可以全面控制 JSON 序列化，在某些情况下，你可能想要使用该模型。例如，你的移动服务表包含大量的列，而你只需要引用其中的某些列。使用类型化模型需要在数据类中定义移动服务表的所有列。但如果使用非类型化模型，只需定义你要使用的列。

与使用类型化模型相似，首先需要获取表引用，不过，此时该引用的是一个 [MobileServicesJsonTable][] 对象。对移动服务客户端的实例调用 [getTable()][] 方法可获取该引用。

你可以使用此方法的以下重载，该重载用于处理基于 JSON 的非类型化编程模型：

        public class MobileServiceClient {
    public MobileServiceJsonTable getTable(String name);
        }

用于访问数据的大多数 API 调用都与类型化编程调用类似。主要差别在于，在非类型化模型中，你要对 "MobileServiceJsonTable" 对象而不是 "MobileServiceTable" 对象调用方法。回调对象和 "onCompleted" 方法的用法与在类型化模型中十分相似。

<a name="json_instance"></a>
### 如何：创建非类型化表的实例

创建移动服务客户端的实例（在此处为 *mClient* 变量）后，接下来请使用以下代码创建 "MobileServiceJsonTable" 的实例。

        MobileServiceJsonTable mTable = mClient.getTable("ToDoItem");

创建 "MobileServiceJsonTable" 的实例后，便可以对该实例调用使用类型化编程模型所能调用的几乎所有方法。但是，在某些情况下，这些方法采用非类型化参数，如以下示例所示。

<a name="json_insert"></a>
### 如何：插入到非类型化表中

以下代码演示了如何执行插入。第一步是创建属于 [gson][] 库的一部分的 ["JsonObject"][]。

        JsonObject task = new JsonObject();
    task.addProperty("text", "Wake up");
    task.addProperty("complete", false);
    task.addProperty("duration", 5);

下一步是插入对象。传递给 ["insert"][3] 方法的回调函数是 ["TableJsonOperationCallback"][] 类的实例。请注意，*onCompleted* 方法的第一个参数是 JsonObject。

        mTable.insert(task, new TableJsonOperationCallback() {
    public void onCompleted(JsonObject jsonObject, 
    Exception exception,
    ServiceFilterResponse response) {
    if (exception == null) {
    Log.i(TAG, "Object inserted with ID " + 
    jsonObject.getAsJsonPrimitive("id").getAsString());
                }
            }
        });

请注意，可使用以下方法调用获取所插入对象的 ID：

                jsonObject.getAsJsonPrimitive("id").getAsInt());

<a name="json_delete"></a>
### 如何：从非类型化表中删除数据

以下代码演示了如何删除一个实例，在本例中，该实例就是我们在前一个 *insert* 示例中创建的 "JsonObject" 的实例。请注意，回调对象 "TableDeleteCallback" 是类型化编程模型中使用的同一个对象，它的 "onCompleted" 方法的签名不同于 "insert" 示例中使用的签名。

        mTable.delete(task, new TableDeleteCallback() {
    public void onCompleted(Exception exception, 
    ServiceFilterResponse response) {
    if (exception == null) {
    Log.i(TAG, "Object deleted");
                }
            }
        });

还可以使用某个实例的 ID 来直接删除该实例：

        mTable.delete(task.getAsJsonPrimitive("id").getAsString(), ...)

<a name="json_get"></a>
### 如何：返回非类型化表中的所有行

以下代码演示了如何检索整个表。请注意，非类型化编程模型使用不同的回调对象：["TableJsonQueryCallback"][]。

        mTable.execute(new TableJsonQueryCallback() {
    public void onCompleted(JsonElement result, 
    int count, 
    Exception exception,
    ServiceFilterResponse response) {
    if (exception == null) {
    JsonArray results = result.getAsJsonArray();
    for(JsonElement item :results){
    Log.i(TAG, "Read object with ID " + 
    item.getAsJsonObject().getAsJsonPrimitive("id").getAsInt());
                    }
                }
            }
        });

你可以通过连接与类型化编程模型中所用方法同名的方法来执行筛选、排序和分页。

<a name="binding"></a>
## 绑定数据如何：将数据绑定到用户界面

数据绑定涉及到三个组件：

-   数据源
-   屏幕布局
-   将两者绑定起来的适配器

在下面的示例代码中，我们会将移动服务表 *ToDoItem* 中的数据返回到一个数组中。这是数据应用程序经常使用的一种模式：数据库查询通常会返回行的集合，客户将在列表或数组中获取该集合。在此示例中，该数组就是数据源。

代码将指定屏幕布局，用于定义设备中显示的数据视图。

数据源和屏幕布局通过适配器绑定在一起，在此代码中，该适配器是 *ArrayAdapter\<ToDoItem\>* 类的扩展。

<a name="layout"></a>
### 如何：定义布局

布局由多个 XML 代码段定义。以某个现有布局为例，我们假设以下代码表示了要在其中填充服务器数据的 "ListView"。

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
        
<a name="adapter"></a>
### 如何：定义适配器

由于此处视图的数据源是一个 *ToDoItem* 数组，因此我们需要基于 *ArrayAdapter\<ToDoItem\>* 类子类化适配器。此子类将使用 *row\_list\_to\_do* 布局为每个 *ToDoItem* 生成一个视图。

在代码中，我们可以定义以下类作为 *ArrayAdapter\<E\>* 类的扩展：

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

请注意，ToDoItemAdapter 构造函数的第二个参数是对布局的引用。在该构造函数的调用后面添加以下代码，以便先获取对 "ListView" 的引用，然后调用 *setAdapter*，使该视图将自身配置为使用刚创建的适配器：

        ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
    listViewToDo.setAdapter(mAdapter);

<a name="use-adapter"></a>
### 如何：使用适配器

现在，你可以使用数据绑定了。以下代码演示了如何获取移动服务表中的项，清除适配器，然后调用适配器的 *add* 方法以在表中填充返回的项。

        mToDoTable.execute(new TableQueryCallback<ToDoItem>() {
    public void onCompleted(List<ToDoItem> result, int count, Exception exception, ServiceFilterResponse response) {
    if (exception == null) {
    mAdapter.clear();
    for (ToDoItem item :result) {
    mAdapter.add(item);
                    }
                } 
            }
        });

每次修改 *ToDoItem* 表后，也必须调用该适配器（如果你想要显示执行修改操作后的结果）。由于修改是按记录完成的，因此要处理的是单个行而不是一个集合。插入项时，需要对适配器调用 *add* 方法；删除项时，需要调用 *remove* 方法。

<a name="authentication"></a>
## 身份验证如何：对用户进行身份验证

移动服务支持使用各种外部标识提供者对应用程序用户进行身份验证和授权，这些提供者包括：Facebook、Google、Microsoft 帐户、Twitter 和 Azure Active Directory。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅[身份验证入门][]。

支持两种身份验证流：*服务器*流和*客户端*流。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能（例如单一登录）进行更深入的集成。

在应用程序中启用身份验证需要执行以下三个步骤：

1.  注册你的应用程序以使用提供者进行身份验证，然后配置移动服务
2.  将表权限限制给已经过身份验证的用户
3.  向应用程序添加身份验证代码

移动服务支持使用以下现有标识提供者对用户进行身份验证：

-   Microsoft 帐户
-   Facebook
-   Twitter
-   Google
-   Azure Active Directory

你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。还可以使用已经过身份验证的用户的 ID 来修改请求。

前两个任务可使用 [Azure 管理门户][]来完成。有关详细信息，请参阅[身份验证入门][]。

<a name="caching"></a>
### 如何：向应用程序添加身份验证代码

1.  将以下 import 语句添加到应用程序的活动文件。

        import com.microsoft.windowsazure.mobileservices.MobileServiceUser;
        import com.microsoft.windowsazure.mobileservices.MobileServiceAuthenticationProvider;
        import com.microsoft.windowsazure.mobileservices.UserAuthenticationCallback;

2.  在活动类的 "onCreate" 方法中，在创建 `MobileServiceClient` 对象的代码后面添加以下代码行：我们假设对 `MobileServiceClient` 对象的引用为 *mClient*。

            // Login using the Google provider.
        mClient.login(MobileServiceAuthenticationProvider.Google,
        new UserAuthenticationCallback() {
        @Override
        public void onCompleted(MobileServiceUser user,
        Exception exception, ServiceFilterResponse response) {  
        if (exception == null) {
        /* User now logged in, you can get their identity via user.getUserId() */ 
        } else {
        /* Login error */
                            }
                        }
                    });

    此代码将使用 Google 登录对用户进行身份验证。此时将出现一个对话框，其中显示了已经过身份验证的用户的 ID。如果未正常完成身份验证，你将无法继续操作。

    <div class="dev-callout"><b>说明</b>

    <p>如果使用的标识提供者不是 Google，请将传递给上述 <b>login</b> 方法的值更改为下列其中一项：<i>MicrosoftAccount</i>、<i>Facebook</i>、<i>Twitter</i> 或 <i>WindowsAzureActiveDirectory</i>。</p>
	</div>

3.  运行应用程序时，请使用选择的标识提供者登录。

<a name="caching"></a>
### 如何：缓存身份验证令牌

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
    // Login using the provider.
    mClient.login(MobileServiceAuthenticationProvider.MicrosoftAccount,
    new UserAuthenticationCallback() {
    @Override
    public void onCompleted(MobileServiceUser user,
    Exception exception, ServiceFilterResponse response) {
    if (exception == null) {
    createTable();
    cacheUser(mClient.getCurrentUser());
    } else {
    createAndShowDialog("You must log in. Login Required", "Error");
                            }
                        }
                    });
        }
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

如果令牌过期会发生什么情况呢？在这种情况下，如果你尝试使用它来建立连接，将会收到“401 未授权”响应"。此时，用户必须登录以获取新令牌。使用筛选器可以截获对移动服务的调用以及来自移动服务的响应，因此不需要在应用程序中调用移动服务的每个位置编写代码来处理这种情况。此时，筛选器代码将测试 401 响应，根据需要触发登录进程，然后恢复生成 401 响应的请求。

<a name="errors"></a>
## 错误处理如何：处理错误

[此处][]提供了一个执行验证并处理各种错误的示例，该示例通过出错时返回异常的服务器脚本以及用于处理异常的客户端代码来实施验证。

处理错误的另一种方法是提供*全局*错误处理程序。该示例中访问移动服务表的代码涉及到三个不同的回调对象：

-   "TableQueryCallback" / "TableQueryJsonCallback"
-   "TableOperationCallback" / "TableJsonOperationCallback"
-   "TableDeleteCallback"

其中的每个对象都有一个 "OnCompleted" 方法，该方法的第二个参数为 "java.lang.Exception" 对象。你可以子类化这些回调对象，并通过实现自己的 "onCompleted" 方法来检查 exception 参数是否为 null。如果为 null，则表示未出错，你只需调用 "super.OnCompleted()"。

如果 "Exception" 对象不为 null，请执行某种常用的错误处理方法，以显示有关错误的更详细信息。以下代码段演示了显示更多详细信息的一种方法。

        String msg = exception.getCause().getMessage();

现在，开发人员可以使用其子类化回调，而不必考虑异常检查，因为系统将在一个中心位置 (\#2) 为回调的所有实例处理该异常。

<a name="customizing"></a>
## 自定义客户端如何：自定义客户端

<a name="headers"></a>
### 如何：自定义请求标头

你可能需要将一个自定义标头附加到每个传出请求。按如下所示配置 ServiceFilter 可以实现此目的：

        client = client.withFilter(new ServiceFilter() {
        
    @Override
    public void handleRequest(ServiceFilterRequest request,
    NextServiceFilterCallback nextServiceFilterCallback,
    ServiceFilterResponseCallback responseCallback) {
    request.addHeader("My-Header", "Value");      
    nextServiceFilterCallback.onNext(request, responseCallback);
            }
        });

<a name="serialization"></a>
### 如何：自定义序列化

默认情况下，移动服务假设服务器上与客户端上的表名称、列名称和数据类型都完全匹配。但是，在许多情况下，服务器上和客户端上的名称并不匹配。现举一例：你要更改某个现有的客户端，使其使用 Azure 移动服务而不是竞争者的产品。

此时，你可能需要执行类似于下面的自定义操作：

-   
    使移动服务表中使用的列名称与你在客户端中使用的名称不匹配
-   使用一个移动服务表，其名称不同于该表在客户端中映射到的类
-   启用属性自动大写
-   向对象添加复杂属性

<a name="columns"></a>
### 如何：映射不同的客户端名称和服务器名称

假设你的 Java 客户端代码为 *ToDoItem* 对象属性使用了类似于下面的标准 Java 样式名称：

<ul>
<li>mId</li>
<li>mText</li>
<li>mComplete</li>
<li>mDuration</li>

</ul>

则你必须将客户端名称序列化为与服务器上 *ToDoItem* 表的列名称匹配的 JSON 名称。以下代码利用 [gson][] 库来执行此操作。

    @com.google.gson.annotations.SerializedName("text")
    private String mText;

    @com.google.gson.annotations.SerializedName("id")
    private int mId;

    @com.google.gson.annotations.SerializedName("complete")
    private boolean mComplete;

    @com.google.gson.annotations.SerializedName("duration")
    private String mDuration;

<a name="table"></a>
### 如何：在客户端与移动服务之间映射不同的表名称

如以下代码所示，只需使用
[getTable()][第二个重载] 函数的重写之一，就能轻松地将客户端表名称映射为不同的移动服务表名称。

        mToDoTable = mClient.getTable("ToDoItemBackup", ToDoItem.class);

<a name="conversions"></a>
### 如何：自动执行列名称映射

如前一部分中所示，映射只包含几个列的简短表的列名称并不复杂。但是，如果表包含大量的列（例如 20 或 30 个列），则我们可以调用 [gson][] API 并指定要应用到每个列的转换策略，这样就无需批注每一个列名称。

为此，我们需要使用 [gson][] 库，Android 客户端库在幕后使用该库将 Java 对象序列化为要发送到 Azure 移动服务的 JSON 数据。

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

<a name="complex"></a>
### 如何：将对象或数组属性存储到表中

到目前为止，我们的所有序列化示例都使用了可轻松序列化成 JSON 和移动服务表的基元类型（例如整数和字符串）。假设我们要将一个不能自动序列化成 JSON 和表的复杂对象添加到客户端类型。例如，我们要将一个字符串数组添加到客户端对象。此时，我们需要指定如何执行序列化，以及如何将数组存储到移动服务表中。

若要查看有关如何执行此操作的示例，请阅读博客文章[在移动服务 Android 客户端中使用][][gson][] 库自定义序列化。

每当我们要使用一个不能自动序列化成 JSON 和移动服务表的复杂对象时，就可以使用此常规方法。

<a name="next-steps"></a>
## 后续步骤

[][]<http://dl.windowsazure.com/androiddocs/com/microsoft/windowsazure/mobileservices/package-summary.html>提供了 Android 客户端 API 的 Javadocs 参考

  [.NET Framework]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/ ".NET Framework"
  [HTML/JavaScript]: /zh-cn/develop/mobile/how-to-guides/work-with-html-js-client/ "HTML/JavaScript"
  [iOS]: /zh-cn/develop/mobile/how-to-guides/work-with-ios-client-library/ "iOS"
  [Android]: /zh-cn/develop/mobile/how-to-guides/work-with-android-client-library/ "Android"
  [Xamarin]: /zh-cn/develop/mobile/how-to-guides/work-with-xamarin-client-library/ "Xamarin"
  [移动服务快速入门]: /zh-cn/develop/mobile/tutorials/get-started-android/
  [移动服务 SDK]: http://go.microsoft.com/fwlink/p/?linkid=280126
  [Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=280125&clcid=0x409
  [什么是移动服务]: #what-is
  [概念]: #concepts
  [安装与先决条件]: #setup
  [如何：创建移动服务客户端]: #create-client
  [如何：创建表引用]: #instantiating
  [API 结构]: #api
  [如何：从移动服务查询数据]: #querying
  [筛选返回的数据]: #filtering
  [为返回的数据排序]: #sorting
  [在页中返回数据]: #paging
  [选择特定的列]: #selecting
  [如何：连接查询方法]: #chaining
  [如何：在移动服务中插入数据]: #inserting
  [如何：在移动服务中更新数据]: #updating
  [如何：在移动服务中删除数据]: #deleting
  [如何：查找特定的项]: #lookup
  [如何：处理非类型化数据]: #untyped
  [如何：将数据绑定到用户界面]: #binding
  [如何：定义布局]: #layout
  [如何：定义适配器]: #adapter
  [如何：使用适配器]: #use-adapter
  [如何：对用户进行身份验证]: #authentication
  [缓存身份验证令牌]: #caching
  [如何：处理错误]: #errors
  [如何：自定义客户端]: #customizing
  [自定义请求标头]: #headers
  [自定义序列化]: #serialization
  [后续步骤]: #next-steps
  [mobile-services-concepts]: ../includes/mobile-services-concepts.md
  [创建表]: http://go.microsoft.com/fwlink/p/?LinkId=298592
  [动态架构]: http://go.microsoft.com/fwlink/p/?LinkId=296271
  [MobileServiceClient]: http://dl.windowsazure.com/androiddocs/com/microsoft/windowsazure/mobileservices/MobileServiceClient.html
  [gson]: %20http://go.microsoft.com/fwlink/p/?LinkId=290801
  [MobileServiceTable]: http://go.microsoft.com/fwlink/p/?LinkId=296835
  [第一个重载]: http://go.microsoft.com/fwlink/p/?LinkId=296839
  [第二个重载]: http://go.microsoft.com/fwlink/p/?LinkId=296840
  ["TableQueryCallback\<E\>"]: http://go.microsoft.com/fwlink/p/?LinkId=296849
  ["where"]: http://go.microsoft.com/fwlink/p/?LinkId=296867
  ["field"]: http://go.microsoft.com/fwlink/p/?LinkId=296869
  ["eq"]: http://go.microsoft.com/fwlink/p/?LinkId=298461
  ["ne"]: http://go.microsoft.com/fwlink/p/?LinkId=298462
  ["gt"]: http://go.microsoft.com/fwlink/p/?LinkId=298463
  ["ge"]: http://go.microsoft.com/fwlink/p/?LinkId=298464
  ["lt"]: http://go.microsoft.com/fwlink/p/?LinkId=298465
  ["le"]: http://go.microsoft.com/fwlink/p/?LinkId=298466
  ["year"]: http://go.microsoft.com/fwlink/p/?LinkId=298467
  ["month"]: http://go.microsoft.com/fwlink/p/?LinkId=298468
  ["day"]: http://go.microsoft.com/fwlink/p/?LinkId=298469
  ["hour"]: http://go.microsoft.com/fwlink/p/?LinkId=298470
  ["minute"]: http://go.microsoft.com/fwlink/p/?LinkId=298471
  ["second"]: http://go.microsoft.com/fwlink/p/?LinkId=298472
  ["startsWith"]: http://go.microsoft.com/fwlink/p/?LinkId=298473
  ["endsWith"]: http://go.microsoft.com/fwlink/p/?LinkId=298474
  ["concat"]: http://go.microsoft.com/fwlink/p/?LinkId=298475
  ["subString"]: http://go.microsoft.com/fwlink/p/?LinkId=298477
  ["indexOf"]: http://go.microsoft.com/fwlink/p/?LinkId=298488
  ["replace"]: http://go.microsoft.com/fwlink/p/?LinkId=298491
  ["toLower"]: http://go.microsoft.com/fwlink/p/?LinkId=298492
  ["toUpper"]: http://go.microsoft.com/fwlink/p/?LinkId=298493
  ["trim"]: http://go.microsoft.com/fwlink/p/?LinkId=298495
  ["length"]: http://go.microsoft.com/fwlink/p/?LinkId=298496
  ["add"]: http://go.microsoft.com/fwlink/p/?LinkId=298497
  ["sub"]: http://go.microsoft.com/fwlink/p/?LinkId=298499
  ["mul"]: http://go.microsoft.com/fwlink/p/?LinkId=298500
  ["div"]: http://go.microsoft.com/fwlink/p/?LinkId=298502
  ["mod"]: http://go.microsoft.com/fwlink/p/?LinkId=298503
  ["floor"]: http://go.microsoft.com/fwlink/p/?LinkId=298505
  ["ceiling"]: http://go.microsoft.com/fwlink/p/?LinkId=298506
  ["round"]: http://go.microsoft.com/fwlink/p/?LinkId=298507
  ["and"]: http://go.microsoft.com/fwlink/p/?LinkId=298512
  ["or"]: http://go.microsoft.com/fwlink/p/?LinkId=298514
  ["not"]: http://go.microsoft.com/fwlink/p/?LinkId=298515
  [了解移动服务 Android 客户端查询模型的丰富功能]: http://hashtagfail.com/post/46493261719/mobile-services-android-querying
  ["orderBy"]: http://go.microsoft.com/fwlink/p/?LinkId=298519
  ["QueryOrder"]: http://go.microsoft.com/fwlink/p/?LinkId=298521
  ["select"]: http://go.microsoft.com/fwlink/p/?LinkId=290689
  [1]: http://go.microsoft.com/fwlink/p/?LinkId=296296
  [2]: http://go.microsoft.com/fwlink/p/?LinkId=296313
  ["top"]: http://go.microsoft.com/fwlink/p/?LinkId=298731
  ["MobileServiceQuery\<T\>"]: http://go.microsoft.com/fwlink/p/?LinkId=298551
  ["execute"]: http://go.microsoft.com/fwlink/p/?LinkId=298554
  ["insert"]: http://go.microsoft.com/fwlink/p/?LinkId=296862
  ["TableOperationCallback\<ToDoItem\>"]: http://go.microsoft.com/fwlink/p/?LinkId=296865
  [ASCII 控制代码 C0 和 C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
  [用于管理移动服务表的 CLI]: http://www.windowsazure.cn/zh-cn/manage/linux/other-resources/command-line-tools/#Mobile_Tables
  ["TableDeleteCallback"]: http://go.microsoft.com/fwlink/p/?LinkId=296858
  [MobileServicesJsonTable]: http://go.microsoft.com/fwlink/p/?LinkId=298733
  [getTable()]: http://go.microsoft.com/fwlink/p/?LinkId=298734
  ["JsonObject"]: http://google-gson.googlecode.com/svn/trunk/gson/docs/javadocs/com/google/gson/JsonObject.html
  [3]: http://go.microsoft.com/fwlink/p/?LinkId=298535
  ["TableJsonOperationCallback"]: http://go.microsoft.com/fwlink/p/?LinkId=298532
  ["TableJsonQueryCallback"]: http://go.microsoft.com/fwlink/p/?LinkId=298543
  [身份验证入门]: http://go.microsoft.com/fwlink/p/?LinkId=296316
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [此处]: https://www.windowsazure.com/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet/
  [在移动服务 Android 客户端中使用]: http://hashtagfail.com/post/44606137082/mobile-services-android-serialization-gson
  []: http://go.microsoft.com/fwlink/p/?LinkId=298735 "此处"
