
本教程基于 [GetStartedWithMobileServices 应用](http://go.microsoft.com/fwlink/p/?LinkID=510826)而编写，后者是 Visual Studio 2013 中的通用 Windows 应用项目。此应用的 UI 与移动服务快速入门生成的应用相同，不过，前者的一些新增项存储在本地内存中。 

1. 从[开发人员代码样本站点]下载 GetStartedWithMobileServices 样本应用的 C# 版本。 

2. 在 Visual Studio 2013 中，打开下载的项目，然后检查在 GetStartedWithData.Shared 项目文件夹中找到的 MainPage.xaml.cs 文件。

   	请注意添加的 **TodoItem** 对象存储在内存中的 **ObservableCollection&lt;TodoItem&gt;** 内。

3. 按 **F5** 键以重新构建项目并启动此应用。

4. 在应用的"插入 TodoItem"中键入一些文本，然后单击"保存"。

   	![](./media/mobile-services-windows-universal-dotnet-download-project/mobile-quickstart-startup.png) 

   	请注意将显示保存的文本。

5. 右键单击 Windows Phone 8.1 项目，单击**设为启动项目**，然后按 **F5** 以启动 Windows Phone Store 应用。  

	![](./media/mobile-services-windows-universal-dotnet-download-project/mobile-quickstart-startup-wp8.png)

6. 重复步骤 3 和 4 以验证该样本的行为方式是否相同。
