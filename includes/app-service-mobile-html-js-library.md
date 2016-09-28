##<a name="create-client"></a>创建客户端连接

通过创建 `WindowsAzure.MobileServiceClient` 对象创建客户端连接。将 `appUrl` 替换为到移动应用的 URL。

```
var client = WindowsAzure.MobileServiceClient(appUrl);
```

##<a name="table-reference"></a>使用表格

若要访问或更新数据，请创建到后端表的引用。将 `tableName` 替换为表名称

```
var table = client.getTable(tableName);
```

拥有表格引用后，可进一步使用表格：

* [查询表](#querying)
  * [筛选数据](#table-filter)
  * [分页浏览数据](#table-paging)
  * [排序数据](#sorting-data)
* [插入数据](#inserting)
* [修改数据](#modifying)
* [删除数据](#deleting)

###<a name="querying"></a>如何：查询表格引用

拥有表格引用后，可用其查询服务器上的数据。查询使用了“类 LINQ”语言。
若要返回表中的所有数据，请使用以下项：

```
/**
 * Process the results that are received by a call to table.read()
 *
 * @param {Object} results the results as a pseudo-array
 * @param {int} results.length the length of the results array
 * @param {Object} results[] the individual results
 */
function success(results) {
   var numItemsRead = results.length;

   for (var i = 0 ; i < results.length ; i++) {
       var row = results[i];
       // Each row is an object - the properties are the columns
   }
}

function failure(error) {
    throw new Error('Error loading data: ', error);
}

table
    .read()
    .then(success, failure);
```

随结果调用 success 函数。请勿在 success 函数中使用 `for (var i in results)`，因为这会在使用其他查询函数（如 `.includeTotalCount()`）时迭代结果中所含的信息。

有关查询语法的详细信息，请参阅[查询对象文档]。

####<a name="table-filter"></a>在服务器上筛选数据

可在表格引用上使用 `where` 子句：

```
table
    .where({ userId: user.userId, complete: false })
    .read()
    .then(success, failure);
```

也可使用筛选对象的函数。该情况下，`this` 变量被分配给经过筛选的当前对象。以下示例在功能上等效于上个示例：

```
function filterByUserId(currentUserId) {
    return this.userId === currentUserId && this.complete === false;
}

table
    .where(filterByUserId, user.userId)
    .read()
    .then(success, failure);
```

####<a name="table-paging"></a>分页浏览数据

利用 take() 和 skip() 方法。例如，如想要将表拆分为 100 行记录：

```
var totalCount = 0, pages = 0;

// Step 1 - get the total number of records
table.includeTotalCount().take(0).read(function (results) {
    totalCount = results.totalCount;
    pages = Math.floor(totalCount/100) + 1;
    loadPage(0);
}, failure);

function loadPage(pageNum) {
    let skip = pageNum * 100;
    table.skip(skip).take(100).read(function (results) {
        for (var i = 0 ; i < results.length ; i++) {
            var row = results[i];
            // Process each row
        }
    }
}
```

`.includeTotalCount()` 方法用于将 totalCount 字段添加到结果对象。如果不分页，totalCount 字段会填充要返回的记录总数。

然后可使用页变量和某些 UI 按钮提供页列表；使用 loadPage() 为每页加载新记录。应实现某种形式的缓存，加快已加载记录的访问速度。


####<a name="sorting-data"></a>如何：返回排序后的数据

使用 .orderBy() 或 .orderByDescending() 查询方法：

```
table
    .orderBy('name')
    .read()
    .then(success, failure);
```

有关 Query 对象的详细信息，请参阅[查询对象文档]。

###<a name="inserting"></a>如何：插入数据

使用相应日期创建 JavaScript 对象并异步调用 table.insert()：

```
var newItem = {
    name: 'My Name',
    signupDate: new Date()
};

table
    .insert(newItem)
    .done(function (insertedItem) {
        var id = insertedItem.id;
    }, failure);
```

成功插入后，插入项随同步操作所需的其他字段一并返回。应使用此信息自行更新缓存，便于后续更新。

请注意，Azure 移动应用 Node.js Server SDK 支持用于开发的动态架构。对于动态架构，表格架构随时更新，使得只需在插入或更新操作进行指定即可向表格添加列。建议先关闭动态架构，再将应用程序迁移到生产。

###<a name="modifying"></a>如何：修改数据

类似于 .insert() 方法，应创建 Update 对象，然后调用 .update()。Update 对象必须包含要更新的记录的 ID，此 ID 在读取记录或调用 .insert() 时获取。

```
var updateItem = {
    id: '7163bc7a-70b2-4dde-98e9-8818969611bd',
    name: 'My New Name'
};

table
    .update(updateItem)
    .done(function (updatedItem) {
        // You can now update your cached copy
    }, failure);
```

###<a name="deleting"></a>如何：删除数据

调用 .del() 方法删除记录。在对象引用中传递 ID：

```
table
    .del({ id: '7163bc7a-70b2-4dde-98e9-8818969611bd' })
    .done(function () {
        // Record is now deleted - update your cache
    }, failure);
```

<!---HONumber=Mooncake_0919_2016-->