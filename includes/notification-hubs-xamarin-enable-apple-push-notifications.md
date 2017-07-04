若要通过 Apple Push Notification 服务 (APNS) 为应用注册推送通知，必须在 Apple 开发人员门户上为项目创建推送证书、应用 ID 并预配配置文件。

应用可以使用应用 ID 中包含的配置设置来发送和接收推送通知。 这些设置包括应用发送和接收推送通知时，在 APNS 上进行身份验证所需的推送通知证书。

有关这些概念的详细信息，请参阅 [Apple Push Notification 服务](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1)文档。

## <a name="generate-the-certificate-signing-request-file-for-the-push-certificate"></a>为推送证书生成证书签名请求文件
这些步骤将引导你完成创建证书签名请求的过程。此请求将生成用于 APNS 的推送证书。

1. 在 Mac 上，运行 Keychain Access 工具。 可以从启动板上的“Utilities”或“Other”文件夹中打开该工具。

2. 选择“Keychain Access”，展开“Certificate Assistant”（证书助理），然后选择“Request a Certificate from a Certificate Authority...”（从证书颁发机构请求证书...）。

      ![请求证书](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-request-cert-from-ca.png)

3. 选择“User Email Address”（用户电子邮件地址）和“Common Name”（公用名）。

4. 确保已选择“Saved to disk”（保存到磁盘），然后选择“Continue”（继续）。 将“CA Email Address”（CA 电子邮件地址）字段留空，因为不需要此信息。

      ![证书信息](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-csr-info.png)

5. 在“Save As”（另存为）中为证书签名请求 (CSR) 文件键入一个名称。
6. 在“Where”（位置）中选择位置，然后选择“Save”（保存）。

      ![保存证书签名请求](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-save-csr.png)

      此操作会将 CSR 文件保存到选定位置。 （默认位置为桌面。）请记住为此文件选择的位置。

## <a name="register-your-app-for-push-notifications"></a>为推送通知注册应用程序
在 Apple 中为应用程序创建新的显式应用 ID，然后对其进行配置以便推送通知。  

1. 导航到 Apple 开发人员中心的 [iOS 预配门户](http://go.microsoft.com/fwlink/p/?LinkId=272456)，然后使用 Apple ID 登录。

2. 选择“Identifiers”（标识符），然后选择“App IDs”（应用 ID）。

3. 选择 **+** 符号注册新应用。

      ![注册新应用](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-ios-appids.png)

4. 更新新应用的以下三个字段，然后选择“Continue”（继续）：

   - **Name**（名称）：在“App ID Description”（应用 ID 说明）部分，为应用键入一个描述性名称。

   - **Bundle Identifier**（捆绑标识符）：在“Explicit App ID”（显式应用 ID）部分，使用 [App Distribution Guide](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/ConfiguringYourApp/ConfiguringYourApp.html#//apple_ref/doc/uid/TP40012582-CH28-SW8)（应用分发指南）中所述的 `<Organization Identifier>.<Product Name>` 格式输入“Bundle Identifier”（捆绑标识符）。 此标识符必须与应用的 XCode、Xamarin 或 Cordova 项目中使用的标识符匹配。

   - **Push Notifications**（推送通知）：在“App Services”（应用服务）部分选中“Push Notifications”（推送通知）选项。

     ![注册应用 ID](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-new-appid-info.png)

5. 在“Confirm your App ID”（确认应用 ID）屏幕上检查设置。 确认后，选择“Register”（注册）。

6. 提交新应用 ID 后，将会看到“注册完成”屏幕。 选择“完成” 。

7. 在开发人员中心的“App IDs”（应用 ID）下，找到创建的应用 ID，然后选择它所在的行。 选择应用 ID 行会显示应用详细信息。 然后选择底部的“Edit”（编辑）按钮。

8. 滚动到屏幕底部，然后选择“Development SSL Certificate”（开发 SSL 证书）部分的“Create Certificate...”（创建证书...）按钮。

      ![开发推送 SSL 证书](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-appid-create-cert.png)

    将显示“Add iOS Certificate”（添加 iOS 证书）助手。

    > [AZURE.NOTE]
    > 本教程使用开发证书。 注册生产证书的过程与此相同。 请确保在发送通知时使用相同的证书类型。
    >

9. 选择“Choose File”（选择文件），然后浏览到推送证书 CSR 保存到的位置。 然后选择“Generate”（生成）。

      ![生成证书](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-appid-cert-choose-csr.png)

10. 门户创建证书后，请选择“Download”（下载）按钮。

      ![证书准备就绪](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-appid-download-cert.png)

       随后将会下载签名证书并将其保存到计算机上的 Downloads 文件夹。

      ![下载文件夹](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-cert-downloaded.png)

    > [AZURE.NOTE]
    > 默认情况下，下载的文件（开发证书）名为 **aps_development.cer**。
    >
    >
11. 双击下载的推送证书 **aps_development.cer**。 将在 Keychain 中安装新证书，如以下屏幕截图所示：

       ![Keychain Access](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-cert-in-keychain.png)

    > [AZURE.NOTE]
    > 证书中的名称可能不同，但带有 **Apple Development iOS Push Services** 前缀。
        >
       >
12. 在 **Keychain Access** 中，选择在“Certificates”（证书）类别中创建的新推送证书。 选择“Export”（导出），为文件命名，选择“.p12”格式，然后选择“Save”（保存）。

    记住导出的 .p12 推送证书的文件名和位置。 稍后要在 Azure 经典管理门户中将它上传，以便通过它来启用 APNS 身份验证。 如果 **.p12** 格式选项不可用，可能需重新启动 Keychain Access。

## <a name="create-a-provisioning-profile-for-the-app"></a>为应用程序创建配置文件
1. 返回 <a href="http://go.microsoft.com/fwlink/p/?LinkId=272456" target="_blank">iOS 预配门户</a>，选择“Provisioning Profiles”（预配配置文件），选择“All”（全部），然后选择 **+** 按钮创建配置文件。 此时会启动“Add iOS Provisiong Profile”（添加 iOS 预配配置文件）工具。

      ![预配配置文件](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-new-provisioning-profile.png)

2. 在“Development”（开发）下，选择“iOS App Development”（iOS 应用开发）作为预配配置文件类型，然后选择“Continue”（继续）。

3. 在“App ID”（应用 ID）下拉列表中选择创建的应用 ID，然后选择“Continue”（继续）。

      ![选择应用 ID](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-select-appid-for-provisioning.png)

4. 在“Select certificates”（选择证书）屏幕中，选择用于代码签名的开发证书，然后选择“Continue”（继续）。 这是一个签名证书，而不是创建的推送证书。

       ![Signing certificate](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-provisioning-select-cert.png)

5. 选择用于测试的“Devices”（设备），然后选择“Continue”（继续）。

     ![添加预配配置文件](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-provisioning-select-devices.png)

6. 最后，在“Profile Name”（配置文件名称）中为配置文件选取一个名称，然后选择“Generate”（生成）。

      ![为配置文件命名](./media/notification-hubs-xamarin-enable-apple-push-notifications/notification-hubs-provisioning-name-profile.png)