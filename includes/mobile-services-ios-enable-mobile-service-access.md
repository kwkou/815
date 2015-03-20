
将移动服务准备就绪后，您可以更新应用，以便在移动服务而不是本地集合中存储项。

1. 如果你尚未安装[移动服务 iOS SDK](https://go.microsoft.com/fwLink/p/?LinkID=266533)，请立即予以安装。安装之后，复制 WindowsAzureMobileServices.framework 目录并覆盖所下载项目中包含的 WindowsAzureMobileServices.framework 目录。这样，您所使用的就是最新的 Azure Mobile Services SDK。

2. 在 ToDoService.h 文件中，找到以下注释代码行：

        // TODO - create an MSClient proeprty

   	在此注释后，添加以下代码行：

        @property (nonatomic, strong)   MSClient *client;

   	这会创建一个属性，该属性表示与服务连接的 MSClient

3. 在 TodoService.m 文件中，找到以下注释代码行：

        // TODO - create an MSTable property for your items

   	在此注释后，在 @interface 声明中添加以下代码行：

        @property (nonatomic, strong)   MSTable *table;

   	这将会创建移动服务表的属性表示形式。

4. 在管理门户中单击"移动服务"，然后单击您刚刚创建的移动服务。

5. 单击"仪表板"选项卡并记下"站点 URL"中的值，然后单击"管理密钥"并记下"应用程序密钥"中的值。

   	![](./media/mobile-services-ios-enable-mobile-service-access/mobile-dashboard-tab.png)

  	从应用代码访问移动服务时，您需要使用这些值。

6. 返回 Xcode，打开 TodoService.m 并找到以下注释代码行：

        // Initialize the Mobile Service client with your URL and key.

    在此注释后，添加以下代码行：

        self.client = [MSClient clientWithApplicationURLString:@"APPURL" applicationKey:@"APPKEY"];

    这将会创建移动服务客户端的实例。

7. 将此代码中的 **APPURL** 和 **APPKEY** 值替换为你在执行步骤时从移动服务获取的 URL 和应用程序密钥 6.

8. 找到以下注释代码行：

        // Create an MSTable instance to allow us to work with the TodoItem table.

    在此注释后，添加以下代码行：

        self.table = [self.client tableWithName:@"TodoItem"];

    这将会创建 TodoItem 表实例。

9. 找到以下注释代码行：

 	    // Create a predicate that finds items where complete is false

    在此注释后，添加以下代码行：

        NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];

    这将会创建一个查询，用于返回尚未完成的所有任务。

10. 找到以下注释代码行：

        // Query the TodoItem table and update the items property with the results from the service

   	将该注释和后面的 **completion** 块调用替换为以下代码：

        [self.table readWhere:predicate completion:^(NSArray *results, NSInteger totalCount, NSError *error)
		{
		   self.items = [results mutableCopy];
           completion();
        }];

11. 找到 **addItem** 方法，并将方法的正文替换为以下代码：

        // Insert the item into the TodoItem table and add to the items array on completion
        [self.table insert:item completion:^(NSDictionary *result, NSError *error) {
            NSUInteger index = [items count];
            [(NSMutableArray *)items insertObject:item atIndex:index];

            // Let the caller know that we finished
            completion(index);
        }];

    此代码会将一个插入请求发送到移动服务。

12. 找到 **completeItem** 方法，并找到以下注释的代码行：

        // Update the item in the TodoItem table and remove from the items array on completion

    Replace the body of the method, from that point to the end of the method, with the following code:

        // Update the item in the TodoItem table and remove from the items array on completion
        [self.table update:mutable completion:^(NSDictionary *item, NSError *error) {

            // Get a fresh index in case the list has changed
            NSUInteger index = [items indexOfObjectIdenticalTo:mutable];

            [mutableItems removeObjectAtIndex:index];

            // Let the caller know that we have finished
            completion(index);
	    }];

   	此代码将删除标记为已完成的 TodoItem。

13. 在 TodoListController.m 中，找到 **onAdd** 方法，并用以下代码将其覆盖：

        (IBAction)onAdd:(id)sender
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


既然此应用已更新从而将移动服务用于后端存储，就可以针对移动服务测试该应用。
