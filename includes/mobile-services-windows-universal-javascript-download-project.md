
本教程基于 [GetStartedWithMobileServices](http://go.microsoft.com/fwlink/p/?LinkID=510826) 应用而编写，后者是 Visual Studio 2013 中的通用 Windows 应用项目。此应用的 UI 与移动服务快速入门生成的应用相同，不过，前者的一些新增项存储在本地内存中。

1. 从 [开发人员代码样本站点] 下载 GetStartedWithMobileServices 样本应用的 JavaScript 版本。 

3. 在 Visual Studio 2013 中，打开下载的项目，在 Solution Explorer 中展开 js 文件夹，并检查 default.js 文件。

   	请注意，添加的 **TodoItem** 对象存储在内存中的 `WinJS.Binding.List` 内。

4. 按 **F5** 键以重新构建项目并启动此应用。

4. 在应用的“插入 TodoItem”中键入一些文本，然后单击“保存”。

   	![](./media/mobile-services-windows-universal-dotnet-download-project/mobile-quickstart-startup.png)

   	请注意将显示保存的文本。

5. 右键单击 Windows Phone 8.1 项目，单击“设为启动项目”，然后按 **F5** 以启动 Windows Phone Store 应用。

	![](./media/mobile-services-windows-universal-dotnet-download-project/mobile-quickstart-startup-wp8.png)

6. 重复步骤 3 和 4 以验证该样本的行为方式是否相同。

<!---HONumber=74-->