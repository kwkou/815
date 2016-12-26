
1. 在 Mac 上访问 [Azure 门户预览]。依次单击“浏览全部”>“移动应用”> 刚创建的后端。在移动应用设置中，依次单击“快速入门”>“iOS (Objective-C)”。若更爱 Swift，则改为单击“快速入门”>“iOS (Swift)”。在“下载并运行 iOS 项目”下单击“下载”。这会下载完整的 Xcode 项目，用于预配置要连接到后端的应用。使用 Xcode 打开项目。

2. 按“运行”按钮生成项目，并在 iOS 模拟器中启动应用。
3. 在应用中键入有意义的文本（例如 *Complete the tutorial*），然后单击加号 (**+**) 图标。这会将一个 POST 请求发送到之前部署的 Azure 后端。请求中的后端数据插入到 TodoItem SQL 表中，并将新存储项的相关信息返回到移动应用中。移动应用会在列表中显示此数据。

   ![在 iOS 上运行的快速启动应用](./media/app-service-mobile-ios-quickstart/mobile-quickstart-startup-ios.png)  


[Azure 门户预览]: https://portal.azure.cn/

<!---HONumber=Mooncake_1219_2016-->