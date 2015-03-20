<properties linkid="develop-mobile-tutorials-add-paging-to-data-ios" urlDisplayName="为数据添加分页" pageTitle="为数据添加分页 (iOS) | 移动开发人员中心" metaKeywords="" description="了解如何使用分页来管理从移动服务返回给 iOS 应用程序的数据量。" metaCanonical="" services="" documentationCenter="Mobile" title="Refine Mobile Services queries with paging" authors="" solutions="" manager="" editor="" />



# 使用分页优化移动服务查询

[WACOM.INCLUDE [mobile-services-selector-add-paging-data](../includes/mobile-services-selector-add-paging-data.md)]

本主题说明如何使用分页来管理从 Azure 移动服务返回给 iOS 应用程序的数据量。在本教程中，你将在客户端上使用 **fetchLimit** 和 **fetchOffset** 查询属性来请求特定的数据"页"。

> [AZURE.NOTE] 为了防止移动设备客户端上发生数据溢出，移动服务实施了自动页限制，该限制默认为每个响应中最多 50 个项。通过指定页大小，你最多可以在响应中显式请求 1,000 个项。

本教程以前一教程[数据处理入门]中的步骤和示例应用程序为基础。在开始学习本教程之前，最起码需要先完成数据处理系列中的第一篇教程，即[数据处理入门]。

1. 在 Xcode 中，打开你在完成[数据处理入门]教程时修改的项目。

2. 按**"运行"**按钮 (Command + R) 生成项目并启动应用程序，在文本框中输入文本，然后单击加号 (**+**) 图标。

3. 重复以上步骤至少三次，因此你将在 TodoItem 表中存储三个以上的项。

4. 打开 QSTodoService.m 文件并找到以下方法：

        - (void)refreshDataOnSuccess:(QSCompletionBlock)completion

   	将整个方法的正文替换为以下代码。

        // Create a predicate that finds active items in which complete is false
        NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];

        // Retrieve the MSTable's MSQuery instance with the predicate you just created.
        MSQuery * query = [self.table queryWithPredicate:predicate];

        query.includeTotalCount = YES; // Request the total item count

        // Start with the first item, and retrieve only three items
        query.fetchOffset = 0;
        query.fetchLimit = 3;

        // Invoke the MSQuery instance directly, rather than using the MSTable helper methods.
        [query readWithCompletion:^(NSArray *results, NSInteger totalCount, NSError *error) {

            [self logErrorIfNotNil:error];
            if (!error)
            {
                // Log total count.
                NSLog(@"Total item count: %@",[NSString stringWithFormat:@"%zd", (ssize_t) totalCount]);
            }

            items = [results mutableCopy];

            // Let the caller know that we finished
            completion();
        }];

   	此查询将返回未标记为已完成的前面三个项。

5. 重新生成并启动应用程序。

    请注意，仅显示 TodoItem 表的前三个结果。

7. 找到以下代码行以再次更新 **refreshDataOnSuccess** 方法：

        query.fetchOffset = 0;

   	这次请将 **query.fetchOffset** 值设置为 3。

   	此查询将跳过前三个结果，返回其后的三个结果。实际上这是数据的第二"页"，其页大小为三个项。

    > [AZURE.NOTE] 本教程为 **fetchOffset** 和 **fetchLimit** 属性设置了硬编码分页值，因此使用的是简化的方案。在实际应用中，您可以对页导航控件或类似的 UI 使用类似于上面的查询，让用户导航到上一页和下一页。你还可以设置 **query.includeTotalCount = YES** 以获取服务器上所有可用项的总数，以及分页的数据。

## <a name="next-steps"> </a>后续步骤

演示移动服务中数据处理基础知识的系列教程到此结束。建议你了解有关以下移动服务主题的详细信息：

* [身份验证入门]
  <br/>了解如何使用 Windows 帐户对应用程序的用户进行身份验证。

<!--
* [推送通知入门]
  <br/>了解如何将非常简单的推送通知发送到你的应用程序。
-->

<!-- Anchors. -->

[后续步骤]:#next-steps

<!-- Images. -->


<!-- URLs. -->
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-ios
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-ios
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-ios
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-ios

[管理门户]: https://manage.windowsazure.cn/
