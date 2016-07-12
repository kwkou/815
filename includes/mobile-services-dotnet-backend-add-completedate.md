在本节中，我们将通过添加一个名为 **CompleteDate** 的新时间戳字段，修改数据库的模型。此字段将记录上一次完成 Todo 项的时间。实体框架将使用派生自 [DropCreateDatabaseIfModelChanges](http://go.microsoft.com/fwlink/?LinkId=394621) 的默认数据库初始程序类，基于我们的模型更改来更新数据库。 

1. 在 Visual Studio 的解决方案资源管理器中，展开 todolist 服务项目中的 **App\_Start** 文件夹。打开 WebApiConfig.cs 文件。

2. 请注意，在 WebApiConfig.cs 文件中，默认数据库初始值设定项类是从  `DropCreateDatabaseIfModelChanges` 类派生的。这意味着，对该模型的任何更改将导致表被删除并重新创建，以适应新模型。因此表中的数据将丢失，并且表将重新植入。修改数据库初始值设定项的 Seed 方法，使得种子数据如下所示，然后保存 WebApiConfig.cs 文件。

    >[AZURE.NOTE]使用默认数据库初始值设定项时，只要实体框架在代码优先模型定义中检测到数据模型更改，它就会删除并重新创建数据库。若要进行此数据模型更改并维护数据库中的现有数据，必须使用代码优先迁移。有关更多信息，请参阅[如何使用代码优先迁移来更新数据模型](/zh-cn/documentation/articles/mobile-services-dotnet-backend-how-to-use-code-first-migrations/)。

        List<TodoItem> todoItems = new List<TodoItem>
        {
          new TodoItem { Id = "1", Text = "First seed item", Complete = false },
          new TodoItem { Id = "2", Text = "Second seed item", Complete = false },
        };
     

3. 在 Visual Studio 的解决方案资源管理器中，展开 todolist 服务项目中的 **DataObjects** 文件夹。打开 TodoItem.cs 文件并更新 TodoItem 类以包括 CompleteDate 字段，如下所示。然后保存 TodoItem.cs 文件。

        public class TodoItem : EntityData
        {
          public string Text { get; set; }
          public bool Complete { get; set; }
          public System.DateTime? CompleteDate { get; set; }
        }

4. 在 Visual Studio 的解决方案资源管理器中，展开 todolist 服务项目中的 **Controllers** 文件夹。打开 TodoItemController.cs 文件并更新  `PatchTodoItem` 方法，这样当 **Complete** 属性从 false 更改为 true 时，该方法将会设置 **CompleteDate**。然后保存 TodoItemController.cs 文件。

        public Task<TodoItem> PatchTodoItem(string id, Delta<TodoItem> patch)
        {
            // If complete is being set to true and was false, set CompleteDate
            if ((patch.GetEntity().Complete == true) &&
                (GetTodoItem(id).Queryable.Single().Complete == false))
            {
                patch.TrySetPropertyValue("CompleteDate", System.DateTime.Now);
            }
            return UpdateAsync(id, patch);
        }


5. 重新生成 todolist .NET 后端服务项目，并确认没有生成错误。 

接下来，您将更新客户端应用，以显示新的 **CompleteDate** 数据。
<!---HONumber=74-->