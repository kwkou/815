
本教程的最后一个阶段是生成和运行您的新应用。

1. 浏览到您保存压缩项目文件的位置，在计算机上展开文件，并在 Visual Studio 中打开解决方案文件。

2. 按 **F5** 键以重新构建项目并启动此应用。

3. 在应用中的**"插入 TodoItem"**中键入有意义的文本（例如  *Complete the tutorial*），然后单击**保存**。

   	这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在应用的第二列中。

4. （可选）在通用 Windows 解决方案中，将默认启动项目更改为其他应用，然后再次运行该应用。

	请注意，在应用启动之后会从移动服务加载上一步中保存的数据。
 
4. 返回管理门户，单击**数据**选项卡，然后单击 **TodoItem** 表。

   	![](./media/mobile-services-javascript-backend-run-app/mobile-data-tab.png)

   	这使您可以浏览此应用插入表中的数据。

   	![](./media/mobile-services-javascript-backend-run-app/mobile-data-browse.png)
