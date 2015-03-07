<properties linkid="develop-mobile-tutorials-dotnet-backend-get-started-with-data-javascript-vs2013" urlDisplayName="Get Started with Data" pageTitle="数据处理入门 (Windows 应用商店) | 移动开发人员中心" metaKeywords="" description="了解如何开始使用移动服务来利用 Windows 应用商店应用程序中的数据。" metaCanonical="" services="" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="wesmc" solutions="" manager="" editor="" />



# 将移动服务添加到现有应用程序

[WACOM.INCLUDE [mobile-services-selector-get-started-data-legacy](../includes/mobile-services-selector-get-started-data-legacy.md)]

本主题说明如何使用 Azure 移动服务作为 Windows 应用商店应用程序的后端数据源。在本教程中，你将要为某个应用程序（该应用程序在内存中存储数据）下载一个 Visual Studio 2013 项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，并查看运行该应用程序时对数据所做的更改。

在本教程中创建的移动服务是一个 .NET 后端移动服务。借助 .NET 后端，你可以对移动服务中的服务器端业务逻辑使用 .NET 语言和 Visual Studio，并且可以在本地计算机上运行和调试移动服务。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 JavaScript 后端版本。

>[WACOM.NOTE]本主题说明如何将 Azure 移动服务添加到 Windows 应用商店项目。你可以使用 Visual Studio 2013 工具将相同的 .NET 后端移动服务添加到一个通用的 Windows 应用程序项目。有关详细信息，请参阅本教程的[通用 Windows 应用程序版本](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-universal-javascript-get-started-data)。

本教程将指导你完成以下基本步骤：

1. [下载 Windows 应用商店应用程序项目]
2. [创建新的移动服务]
3. [在本地下载移动服务]
4. [更新 Windows 应用商店应用程序以使用移动服务]
5. [针对本地托管的服务测试 Windows 应用商店应用程序]
6. [将移动服务发布到 Azure]
7. [针对 Azure 中托管的服务测试 Windows 应用商店应用程序]

若要完成本教程，你需要以下各项：

* 有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](/zh-cn/pricing/1rmb-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.cn%2Fzh-cn%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-store-javascript-get-started-data%2F)。
* <a href="https://go.microsoft.com/fwLink/p/?LinkID=257546" target="_blank">Visual Studio Professional 2013</a>。可以使用免费试用版。

## <a name="download-app"></a>下载 GetStartedWithData 项目

本教程以 [GetStartedWithMobileServices 应用程序][开发人员代码示例站点]（Visual Studio 2013 中的一个 Windows 应用商店应用程序项目）为基础。此应用程序的 UI 与移动服务快速入门中生成的应用程序类似，不过，前者的一些新增项本地存储在内存中。 

1. 从[开发人员代码示例站点]下载 GetStartedWithMobileServices 示例应用程序的 JavaScript 版本。 

2. 右键单击 Visual Studio，然后单击**"以管理员身份运行"**，以使用管理特权运行 Visual Studio 2013。

3. 在 Visual Studio 2013 中，打开已下载的项目。在解决方案资源管理器中展开 js 文件夹，并检查 default.js 文件。

   	请注意添加的 **TodoItem** 对象存储在内存中的  `WinJS.Binding.List` 内。

4. 按 **F5** 键重新生成项目并启动应用程序。

5. 在应用程序中**"插入 TodoItem"**下面的框内键入一些文本，然后单击**"保存"**。

   	![][0]  

   	可以看到，保存的文本已显示在**"查询和更新数据"**下的第二列中。

## <a name="create-service"></a>创建新的移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

## <a name="download-the-service-locally"></a>下载移动服务项目并将其添加到解决方案

1. 在 [Azure 管理门户]中，单击新建的移动服务或者其云图标选项卡，以转到概述页。

    ![][2]

2. 单击**"Windows 应用商店"**平台。在**"入门"**部分下，展开**"连接现有 Windows 应用商店应用程序"**，然后单击**"下载"**按钮以下载移动服务的个性化初学者项目。 

    ![][3]

3. 向下滚动到**"入门"**部分底部的标题为**"将服务发布到云"**的步骤。单击以下屏幕快照中所示的链接，下载你刚刚下载的移动服务的发布配置文件。

    > [WACOM.NOTE] 请将该文件保存在安全位置，因为它包含与你的 Azure 帐户相关的敏感信息。在根据本教程后面的内容发布移动服务后，你将要删除此文件。 

    ![][5]


4. 解压缩你下载的个性化服务初学者项目。将 zip 文件中的文件夹复制到"数据处理入门"解决方案文件 (.sln) 所在的同一个 **JavaScript** 目录。这样可以方便 NuGet 包管理器将所有程序包保持同步。 

    ![][26]

5. 在 Visual Studio 的解决方案资源管理器中，右键单击"数据处理入门"Windows 应用商店应用程序对应的解决方案。单击**"添加"**，然后单击**"现有项目"**。

    ![][4]

6. 在"添加现有项目"对话框中，导航到你已移到 **JavaScript** 目录中的移动服务项目文件夹。在服务子目录中选择 C# 项目文件 (.csproj)。单击**"打开"**将该项目添加到你的解决方案。

    ![][6]

7. 在 Visual Studio 的解决方案资源管理器中，右键单击你刚添加的服务项目，然后单击**"生成"**以验证该项目是否能够生成且不出错。在生成期间，NuGet 包管理器可能需要还原项目中引用的某些 NuGet 包。

    ![][20]

8. 再次右键单击该服务项目。这一次请单击**"调试"**上下文菜单下的**"启动新实例"**。

    ![][21]

    Visual Studio 将打开服务的默认网页。你可以单击**"立即尝试"**以从默认网页测试移动服务中的方法。

    ![][22]

    默认情况下，Visual Studio 在 IIS Express 本地托管你的移动服务。在任务栏中右键单击 IIS Express 的任务栏图标即可看到此信息。

    ![][23]


# <a name="update-app"></a>更新 Windows 应用商店应用程序以使用移动服务

在本部分中，你将要更新 Windows 应用商店应用程序，以将移动服务用作应用程序的后端服务。


1. 在 Visual Studio 的解决方案资源管理器中，右键单击 Windows 应用商店应用程序项目，然后单击**"管理 NuGet 包"**。

    ![][7]

2. 在"管理 NuGet 包"对话框中，搜索联机程序包集合中的 **WindowsAzure.MobileServices.WinJS**，并单击它以安装 Azure 移动服务 Nuget 包。然后关闭该对话框。

    ![][8]

3. 返回到 Azure 管理门户中的移动服务概述页，找到标签为**"连接你的应用程序并存储服务中的数据"**的步骤。单击**"JavaScript"**以将它选作你的语言，然后复制用于创建  `MobileServiceClient` 的代码段。

    ![][9]

4. 在 Visual Studio 的解决方案资源管理器中，展开 **js** 文件夹并打开 default.js。将复制的代码段粘贴到紧靠在  `todoItems` 变量定义前面的  `app.onactivated` 事件处理程序中。请注意，该代码段包含一个已注释掉的构造函数，该构造函数用于通过应用程序密钥连接到 Azure 中的移动服务。在后面的步骤中，你将要取消注释此构造函数。

    ![][10]


5. 在 default.js 中，将  `app.onactiviated` 事件处理程序的余下代码替换为以下代码，以便定义  `todoItems` 以及要通过移动服务进行测试的操作。然后，保存文件。

    此代码借助适用于 JavaScript 的移动服务 SDK 使应用程序将其数据存储在服务提供的表中，而不是本地存储在内存中。三个主要方法为  `insertTodoItem`、 `refreshTodoItems` 和  `updateCheckedTodoItem`。通过这三个方法，你可以使用 Azure 中的表异步插入、查询和更新数据集合。

            var todoTable = client.getTable('TodoItem');
            var todoItems = new WinJS.Binding.List();
            var insertTodoItem = function (todoItem) {
                // This code inserts a new TodoItem into the database. 
                // Mobile Services assigns each item an id and the 
                // item is added to the Binding List
                todoTable.insert(todoItem).done(function (item) {
                    todoItems.push(item);
                });
            };
            var refreshTodoItems = function () {
                // This code refreshes the entries in the list view 
                // by querying the TodoItems table.
                todoTable.where()
                    .read()
                    .done(function (results) {
                        todoItems = new WinJS.Binding.List(results);
                        listItems.winControl.itemDataSource = todoItems.dataSource;
                    });
            };
            var updateCheckedTodoItem = function (todoItem) {
                // This code updates the database. 
                // When the MobileService responds, 
                // the item is removed from the list 
                todoTable.update(todoItem).done(function (item) {
                    todoItems.splice(todoItems.indexOf(item), 1);
                });
            };
            buttonSave.addEventListener("click", function () {
                insertTodoItem({
                    text: textInput.value,
                    complete: false
                });
            });
            buttonRefresh.addEventListener("click", function () {
                refreshTodoItems();
            });
            listItems.addEventListener("change", function (eventArgs) {
                var todoItem = eventArgs.target.dataContext.backingData;
                todoItem.complete = eventArgs.target.checked;
                updateCheckedTodoItem(todoItem);
            });
            refreshTodoItems();

6. 在 Visual Studio 的解决方案资源管理器中打开 default.html 文件。在该文件的顶部，为 MobileServices.js 添加新的 WinJS 脚本引用。然后，保存文件。

        <script type="text/javascript" src="/js/MobileServices.js"></script>

    ![][19]

## <a name="test-locally-hosted"></a>针对本地托管的服务测试 Windows 应用商店应用程序

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service-data](../includes/mobile-services-dotnet-backend-test-local-service-data.md)]

## <a name="publish-mobile-service"></a>将移动服务发布到 Azure

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]


## <a name="test-azure-hosted"></a>测试已发布到 Azure 的移动服务

1. 在 Visual Studio 的解决方案资源管理器中，展开 **js** 文件夹并打开 default.js。注释掉功能如下的代码：创建与本地托管移动服务建立连接的  `MobileServiceClient`。取消注释功能如下的代码：创建与 Azure 中的服务建立连接的  `MobileServiceClient`。保存对该文件所做的更改。

        //var client = new WindowsAzure.MobileServiceClient(
        //          "http://localhost:59226"
        //);
        // Use this constructor instead after publishing to the cloud
        var client = new WindowsAzure.MobileServiceClient(
            "https://todolist.preview.azure-mobile-preview.net/",
            "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");


2. 在 Visual Studio 中，按 F5 键或者在"调试"菜单中单击**"启动调试"**。这样，便会使用运行应用程序之前发生的更改重新生成 Windows 应用商店应用程序，以连接到 Azure 中远程托管的移动服务。 

    ![][12]


3. 输入一些新的 todoitem，然后单击**"保存"**以保存每个 todoitem。单击复选框以完成其中的某些新项。每个新的 todoItem 将在你前面通过 Azure 管理门户为移动服务配置的 SQL 数据库中存储并更新。 

    ![][16]

    你可以重新启动应用程序，以查看更改是否已持久保存在 Azure 中的数据库内。你还可以使用 Azure 管理门户或者 Visual Studio 的 SQL Server 对象资源管理器来检查数据库。后面两个步骤将使用 Azure 管理门户来查看数据库中的更改。

4. 在 Azure 管理门户中，单击与你的移动服务关联的数据库对应的"管理"。

    ![][17]

5. 在管理门户中，执行一个查询以查看 Windows 应用商店应用程序所做的更改。你的查询应类似于以下查询，不过，请使用你的数据库名称而不是  `todolist`。

        SELECT * FROM [todolist].[todoitems]

    ![][18]

**数据处理入门**教程到此结束。

## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使 Windows 应用商店应用程序处理移动服务中的数据的基础知识。接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

* [使用脚本验证和修改数据]
  <br/>详细了解如何使用移动服务中的服务器脚本验证和更改从应用程序发送的数据。

* [使用分页优化查询]
  <br/>了解如何使用查询中的分页控制单个请求中处理的数据量。

完成数据系列教程后，请试着学习下列教程之一：

* [身份验证入门]
  <br/>了解如何对应用程序用户进行身份验证。

* [推送通知入门]
  <br/>了解如何向应用程序发送一条非常简单的推送通知。

* [移动服务 .NET 操作方法概念性参考]
  <br/>详细了解如何将移动服务与 HTML 和 JavaScript 一起使用。
  
<!-- Anchors. -->

[下载 Windows 应用商店应用程序项目]: #download-app
[创建新的移动服务]: #create-service
[在本地下载移动服务]: #download-the-service-locally
[更新 Windows 应用商店应用程序以使用移动服务]: #update-app
[针对本地托管的服务测试 Windows 应用商店应用程序]: #test-locally-hosted
[将移动服务发布到 Azure]: #publish-mobile-service
[针对 Azure 中托管的服务测试 Windows 应用商店应用程序]: #test-azure-hosted
[Next Steps]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/app-view.png
[1]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/mobile-data-sample-download-javascript-vs13.png
[2]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/mobile-service-overview-page.png
[3]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/download-service-project.png
[4]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/add-service-project-to-solution.png
[5]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/download-publishing-profile.png
[6]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/add-existing-project-dialog.png
[7]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-manage-nuget-packages.png
[8]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/manage-nuget-packages.png
[9]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/copy-mobileserviceclient-snippet.png
[10]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-pasted-mobileserviceclient.png
[11]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-build-solution.png
[12]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-run-solution.png

[16]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/azure-items.png
[17]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/manage-sql-azure-database.png
[18]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/sql-azure-query.png
[19]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-mobileservices-script-reference.png
[20]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-build-service-project.png
[21]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-start-debug-service-project.png
[22]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/service-welcome-page.png
[23]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/iis-express-tray.png

[26]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/copy-service-and-packages-folder.png

<!-- URLs. -->
[使用脚本验证和修改数据]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-validate-modify-data-server-scripts
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-add-paging-data
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push/


[Azure 管理门户]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[移动服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkId=257545
[开发人员代码示例站点]:  http://go.microsoft.com/fwlink/p/?LinkId=328660
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library/
[MobileServiceClient 类]: http://go.microsoft.com/fwlink/p/?LinkId=302030
