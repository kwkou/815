

1. 在 Mac 上启动“Keychain Access”。在左侧导航栏的“类别”下打开“我的证书”。找到在上一节中下载的 SSL 证书，并公开其内容。仅选择证书（不选择私钥），并[将其导出](https://support.apple.com/kb/PH20122?locale=en_US)。
2. 在 [Azure 门户](https://portal.azure.cn/)中，单击“浏览全部”>“应用服务”，然后单击移动应用后端。在“设置”下，单击“应用服务推送”，然后单击通知中心名称。转到“Apple 推送通知服务”>“上传证书”。上传 .p12 文件，选择正确的**模式**（具体取决于之前的客户端 SSL 证书是生产还是沙盒）。保存所有更改。

现在，服务已配置为在 iOS 上使用通知中心。

[1]: ./media/app-service-mobile-apns-configure-push/mobile-push-notification-hub.png

<!---HONumber=Mooncake_0116_2017-->