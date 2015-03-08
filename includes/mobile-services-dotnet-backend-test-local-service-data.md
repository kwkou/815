在本部分中，你将要使用 Visual Studio 在 IIS Express 中的开发工作站上本地托管移动服务。然后，你将要测试应用程序和后端服务。


1. 在 Visual Studio 中，按 F7 键或者在**构建**菜单中单击**构建解决方案**，以同时构建 Windows Store 应用和移动服务。在 Visual Studio 的输出窗口中确认是否已生成这两个项目且未出错

2. 在 Visual Studio 中，按 F5 键或者在**调试**菜单中单击**启动调试**，以运行应用并在 IIS Express 中本地托管移动服务。 

 
3. 输入新 todoitem 的文本。然后单击**保存**。这样便会在 IIS Express 本地托管的移动服务所创建的数据库中插入一个新的 todoItem。 

    ![](./media/mobile-services-dotnet-backend-test-local-service-data/new-local-todoitem.png)

4. 单击某个项对应的复选框可将它标记为已完成。

    ![](./media/mobile-services-dotnet-backend-test-local-service-data/local-item-checked.png)

5. 在 Visual Studio 中，打开 Server Explorer 并展开"数据连接"即可查看数据库中针对后端服务创建的更改。右键单击 **MS_TableConnectionString** 下的 TodoItems 表，然后单击**显示表数据**

    ![](./media/mobile-services-dotnet-backend-test-local-service-data/vs-show-local-table-data.png)
