
1. 在 [Azure 门户预览](https://portal.azure.cn/)中，依次单击“浏览”>“应用服务”，然后单击移动应用后端 >“所有设置”，然后在“移动”下，单击“推送”。

2. 在“推送”通知服务中，单击 “Windows (WNS)”，输入“安全密钥”（客户端机密）和从 Live 服务站点获取的“包 SID”，然后单击“保存”。

    ![设置门户中的 GCM API 密钥](./media/app-service-mobile-configure-wns/mobile-push-wns-credentials.png)

移动应用后端现已配置为使用 WNS 将推送通知发送到使用其通知中心的 Windows 应用。

<!---HONumber=Mooncake_0919_2016-->