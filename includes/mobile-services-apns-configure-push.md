
* 遵循[在服务器上安装客户端 SSL 签名标识](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/ConfiguringPushNotifications/ConfiguringPushNotifications.html#//apple_ref/doc/uid/TP40012582-CH32-SW15)中的步骤，将你在前一步骤中下载的证书导出到 .p12 文件。

* 在 Azure 门户中，单击“移动服务”> 你的应用 >“推送”选项卡 >“Apple 推送通知设置”>“上载”。上载 .p12 文件，确保选择正确的“模式”（选择“沙盒”或“生产”，具体取决于生成的客户端 SSL 证书是“开发”还是“分发”。） 现在，你的移动服务已配置为在 iOS 上使用通知中心！

<!---HONumber=71-->