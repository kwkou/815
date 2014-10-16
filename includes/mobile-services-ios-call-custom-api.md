<a name="update-app"></a>
## 更新应用程序更新应用程序以调用自定义 API

1.  创建一个按钮，让你能够单击它调用自定义 API。将 "Round Rect Button" 从位于“实用程序”窗格底部的“对象库” 拖出， 将其放置在文本字段的下方或旁边。双击以添加文本“全部”。 

    这样可以添加新按钮“全部” 。

2.  打开 "QSTodoService.m" 代码文件，找到 `refreshDataOnSuccess` 方法并确认它包含以下代码：

        - (void)refreshDataOnSuccess:(QSCompletionBlock)completion
        {          
        // Create a predicate that finds items where complete is false
        NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];

        // Query the TodoItem table and update the items property with the results from the service
        [self.table readWithPredicate:predicate completion:^(NSArray *results, NSInteger totalCount, NSError *error)
            {
        [self logErrorIfNotNil:error];

        items = [results mutableCopy];

        // Let the caller know that we finished
        completion();
            }];                                 
        }

    这样可以筛选项，使得查询不返回已完成的项。

3.  现在，我们要将此对象连接到视图控制器源代码。按 "Ctrl" 并单击新的“全部” 按钮，将鼠标拖动到 "QSTodoListViewController.h" 中的 `@end` 之前。将对象连接到 "QSTodoListViewController" 中的名为 `onCompleteAll` 的新操作 。Xcode 将自动在 `@end` 行之前插入以下行：

           - (IBAction)onCompleteAll:(id)sender;

4.  `onCompleteAll` 方法的目的是处理新按钮的 Click 事件。它调用新的 `completeAll` 方法，我们会将该方法添加到自定义类，该类再向新的自定义 API 发送 POST 请求。与任何错误相同，自定义 API 返回的结果也显示在消息对话框中。编辑 "QSTodoListViewController.m"，在 `@end` 行之前添加以下实现：

           - (IBAction)onCompleteAll:(id)sender {
        [self.todoService completeAll:^(id result, NSHTTPURLResponse* response, NSError* error)
             {
        if (error)
                 {
        NSString* errorMessage = @"There was a problem! ";
        errorMessage = [errorMessage stringByAppendingString:[error localizedDescription]];
        UIAlertView* myAlert = [[UIAlertView alloc]
        initWithTitle:@"Error!"
        message:errorMessage
        delegate:nil
        cancelButtonTitle:@"Okay"
        otherButtonTitles:nil];
        [myAlert show];
        [self refresh];
        } else {
        NSString* successMessage = [NSString stringWithFormat:@"%d items marked as complete", [[result objectForKey:@"count"] integerValue]];                   
        UIAlertView* myAlert = [[UIAlertView alloc]
        initWithTitle:@"Success!"
        message:successMessage
        delegate:nil
        cancelButtonTitle:@"Okay"
        otherButtonTitles:nil];
        [myAlert show];
        [self refresh];
                 }
             }];
           }

5.  请注意，上面的代码引用了新方法 `completeAll`，该方法尚未在 "QSTodoService" 中定义。编辑 "QSTodoService.h" 并在 `@end` 行之前添加以下代码行：

        - (void) completeAll:(MSAPIBlock)completion;

6.  在 "QSTodoService.m" 中的 `@end` 行之前添加 `completeAll` 的相应实现。iOS 与 JavaScript 的相似之处在于它不支持任意类型的 JSON 序列化。因此，它还具有非常简单的 API，可用于调用自定义 API，包含 `invokeAPI` 方法。

        - (void) completeAll:(MSAPIBlock)completion
        {
        [self.client
        invokeAPI:@"completeall"
        body:nil
        HTTPMethod:@"POST"
        parameters:nil
        headers:nil
        completion:completion ];
        }

<a name="test-app"></a>
## 测试应用程序

1.  在 Xcode 中，选择要部署到的模拟器（iPhone 或 iPad），按“运行”  按钮（或 "Command+R" 键）以重新生成项目并启动应用程序。这样可以执行你的 Windows Azure 移动服务客户端，它使用 iOS SDK 构建，可查询来自你的移动服务的项。

2.  在文本字段中键入文本，然后单击 "+" 按钮。此时会将一个新项作为 insert 发送到移动服务。

3.  重复上述步骤，直至你将多个项添加到列表。

4.  点击“全部”按钮。 此时会显示一个警报框，指示标记为完成的项的数量，并再次执行筛选查询，将所有项从列表中清除。

    ![][0]

  [0]: ./media/mobile-services-ios-call-custom-api/mobile-custom-api-ios-completed.png
