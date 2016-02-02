
现在，让我们更新应用，以便在 Azure 移动服务而不是本地内存集合中存储项。

* 在 **TodoService.h** 中找到以下行：

```
// TODO - create an MSClient property
```

将此注释替换为以下行。这会创建一个属性，该属性表示与服务连接的 `MSClient`。

```
@property (nonatomic, strong)   MSClient *client;
```


* 在 **TodoService.m** 中找到以下行：

```
// TODO - create an MSTable property for your items
```

将此注释替换为 `@interface` 声明中的以下行。这将会创建移动服务表的属性表示形式。

```
@property (nonatomic, strong)   MSTable *table;
```


* 在 [Azure 经典门户](https://manage.windowsazure.cn/)中单击“移动服务”，然后单击该移动服务。单击“仪表板”选项卡，并记下“站点 URL”。然后单击“管理密钥”，并记下“应用程序密钥”。从应用代码访问移动服务时，您需要使用这些值。


* 在 **TodoService.m** 中找到以下行：

```
// Initialize the Mobile Service client with your URL and key.
```

在此注释后，添加以下代码行。将 `APPURL` 和 `APPKEY` 替换为你在上一步获取的站点 URL 和应用程序密钥。

```
self.client = [MSClient clientWithApplicationURLString:@"APPURL" applicationKey:@"APPKEY"];
```


* 在 **TodoService.m** 中找到以下行：

```
// Create an MSTable instance to allow us to work with the TodoItem table.
```

将此注释替换为以下行。这将会创建 TodoItem 表实例。

```
self.table = [self.client tableWithName:@"TodoItem"];
```


* 在 **TodoService.m** 中找到以下行：

```
// Create a predicate that finds items where complete is false
```

将此注释替换为以下行。这将会创建一个查询，用于返回尚未完成的所有任务。

```
NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
```


* 找到以下行：

```
// Query the TodoItem table and update the items property with the results from the service
```

将该注释和后面的 **completion** 块调用替换为以下代码：

```
[self.table readWhere:predicate completion:^(NSArray *results, NSInteger totalCount, NSError *error)
{
self.items = [results mutableCopy];
   completion();
}];
```


* 找到 **addItem** 方法，并将其正文替换为以下代码。此代码会将一个插入请求发送到移动服务。

```

        // Insert the item into the TodoItem table and add to the items array on completion
        [self.table insert:item completion:^(NSDictionary *result, NSError *error) {
            NSUInteger index = [items count];
            [(NSMutableArray *)items insertObject:item atIndex:index];

            // Let the caller know that we finished
            completion(index);
        }];
```


* 找到 **completeItem** 方法，并找到以下注释的代码行：

```
// Update the item in the TodoItem table and remove from the items array on completion
```

在该方法的正文中，将从该位置到方法末尾的部分替换为以下代码。此代码将删除标记为已完成的 todo 项。

```
// Update the item in the TodoItem table and remove from the items array on completion
[self.table update:mutable completion:^(NSDictionary *item, NSError *error) {

            // Get a fresh index in case the list has changed
            NSUInteger index = [items indexOfObjectIdenticalTo:mutable];

            [mutableItems removeObjectAtIndex:index];

    // Let the caller know that we have finished
    completion(index);
}];
```


* 在 TodoListController.m 中，找到 **onAdd** 方法，并用以下代码将其覆盖：

```
- (IBAction)onAdd:(id)sender
{
    if (itemText.text.length  == 0)
    {
        return;
    }

          NSDictionary *item = @{ @"text" : itemText.text, @"complete" : @NO };
          UITableView *view = self.tableView;
          [self.todoService addItem:item completion:^(NSUInteger index)
          {
              NSIndexPath *indexPath = [NSIndexPath indexPathForRow:index inSection:0];
              [view insertRowsAtIndexPaths:@[ indexPath ]
                          withRowAnimation:UITableViewRowAnimationTop];
          }];

    itemText.text = @"";
}
```

<!---HONumber=Mooncake_0118_2016-->