## 安装 Windows 应用商店应用程序的存储客户端

若要使用 SAS 将图像上载到 Blob 存储，必须先添加 NuGet 包，该包用于安装 Windows 应用商店应用程序的存储客户端库。

1. 在 Visual Studio 的“解决方案资源管理器”中，右键单击项目名称，然后选择“管理 NuGet 包”。

2. 在左窗格中，选择“联机”类别，搜索 `WindowsAzure.Storage`，在“Azure 存储空间”包上单击“安装”，然后接受许可协议。

  	![](./media/mobile-services-windows-universal-dotnet-upload-to-blob-storage/mobile-add-storage-nuget-package-dotnet.png)

  	随即会将 Azure 存储服务的客户端库添加到项目。

接下来，你要更新快速入门应用程序以捕获和上载图像。

## 更新快速启动客户端应用以捕获和上载图像

1. 在 Visual Studio 中，打开 Windows 应用程序项目的 Package.appxmanifest 文件，并在“功能”选项卡中启用“网络摄像机”和“麦克风”功能。

   	![](./media/mobile-services-windows-universal-dotnet-upload-to-blob-storage/mobile-app-manifest-camera.png)
 
   	这样可以确保你的应用能够使用连接到计算机的相机。应用第一次运行时，将请求用户允许对相机进行访问。

2. 对 Windows Phone 应用程序项目重复上述步骤。
 
3. 在 Windows 应用程序项目中，打开 MainPage.xaml 文件，并将第一个 **QuickStartTask** 元素之后紧邻的 **StackPanel** 元素替换为以下代码：

		<StackPanel Orientation="Horizontal" Margin="72,0,0,0">
            <TextBox Name="TextInput" Margin="5" MaxHeight="40" MinWidth="300"></TextBox>
            <AppBarButton Label="Photo" Icon="Camera" Name="ButtonCapture" Click="ButtonCapture_Click" />
            <AppBarButton Label="Upload" Icon="Upload" Name="ButtonSave" Click="ButtonSave_Click"/>
        </StackPanel>
        <Grid Name="captureGrid" Margin="62,0,0,0" Visibility="Collapsed" >
            <Grid.RowDefinitions>
                <RowDefinition Height="*" />
                <RowDefinition Height="Auto" />
            </Grid.RowDefinitions>
            <CaptureElement x:Name="previewElement" Grid.Row="0" Tapped="previewElement_Tapped" />
            <Image Name="imagePreview" Grid.Row="0"  />
            <StackPanel Name="captureButtons" Orientation="Horizontal" Grid.Row="1">
                <TextBlock Name="TextCapture" />
                <AppBarButton Label="Retake" Icon="Redo" Name="ButtonRetake" Click="ButtonCapture_Click" />
                <AppBarButton Label="Cancel" Icon="Cancel" Name="ButtonCancel" Click="ButtonCancel_Click" />
            </StackPanel>
        </Grid>

2. 将 **DataTemplate** 中的 **StackPanel** 元素替换为以下代码：

        <StackPanel Orientation="Vertical">
            <CheckBox Name="CheckBoxComplete" IsChecked="{Binding Complete, Mode=TwoWay}" 
                        Checked="CheckBoxComplete_Checked" Content="{Binding Text}" 
                        Margin="10,5" VerticalAlignment="Center"/>
            <Image Name="ImageUpload" Source="{Binding ImageUri, Mode=OneWay}"
                    MaxHeight="250"/>
        </StackPanel> 

   	这样可将图像添加到 **ItemTemplate**，并将其绑定源设置为 Blob 存储服务中已上载图像的 URI。

3. 在 Windows Phone 应用程序项目中，打开 MainPage.xaml 文件，并将 **ButtonSave** 元素替换为以下代码：

        <StackPanel Grid.Row ="1" Grid.Column="1"  Orientation="Horizontal">
            <AppBarButton Label="Photo" Icon="Camera" Name="ButtonCapture" 
                          Click="ButtonCapture_Click" Height="78" Width="62" />
            <AppBarButton Label="Upload" Icon="Upload" Name="ButtonSave" 
                          Click="ButtonSave_Click"/>
        </StackPanel>
        <Grid Grid.Row="2" Name="captureGrid" Grid.RowSpan="3" Grid.ColumnSpan="2" 
              Canvas.ZIndex="99" Background="{ThemeResource ApplicationPageBackgroundThemeBrush}" 
              Visibility="Collapsed">
            <Grid.RowDefinitions>
                <RowDefinition Height="*" />
                <RowDefinition Height="Auto" />
            </Grid.RowDefinitions>
            <CaptureElement Grid.Row="0" x:Name="previewElement" Tapped="previewElement_Tapped" />                    
            <Image Grid.Row="0" Name="imagePreview" Visibility="Collapsed" />
            <StackPanel Grid.Row="1" Name="captureButtons" 
                        Orientation="Horizontal" Visibility="Collapsed">
                <TextBlock Name="TextCapture" VerticalAlignment="Bottom" />
                <AppBarButton Label="Retake" Icon="Redo" Name="ButtonRetake" 
                              Click="ButtonRetake_Click" />
                <AppBarButton Label="Cancel" Icon="Cancel" Name="ButtonCancel" 
                              Click="ButtonCancel_Click" />
            </StackPanel>
        </Grid>

2. 将 **DataTemplate** 中的 **StackPanel** 元素替换为以下代码：

        <StackPanel Orientation="Vertical">
            <CheckBox Name="CheckBoxComplete" IsChecked="{Binding Complete, Mode=TwoWay}" 
                      Checked="CheckBoxComplete_Checked" Content="{Binding Text}" 
                      Margin="10,5" VerticalAlignment="Center"/>
            <Image Name="ImageUpload" Source="{Binding ImageUri, Mode=OneWay}" 
                   MaxHeight="150"/>
        </StackPanel>

4. 在共享的 DataModel 文件夹中，打开 TodoItem.cs 项目文件并将以下属性添加到 TodoItem 类：

        [JsonProperty(PropertyName = "containerName")]
        public string ContainerName { get; set; }
		
        [JsonProperty(PropertyName = "resourceName")]
        public string ResourceName { get; set; }
		
        [JsonProperty(PropertyName = "sasQueryString")]
        public string SasQueryString { get; set; }
		
        [JsonProperty(PropertyName = "imageUri")]
        public string ImageUri { get; set; } 

3. 打开共享的 MainPage.xaml.cs 项目文件并添加以下 **using** 语句：
	
		using Windows.Media.Capture;
		using Windows.Media.MediaProperties;
		using Windows.Storage;
		using Windows.UI.Xaml.Input;
		using Microsoft.WindowsAzure.Storage.Auth;
		using Microsoft.WindowsAzure.Storage.Blob;
		using Windows.UI.Xaml.Media.Imaging;

5. 在 MainPage 类中，添加以下代码：

        // Use a StorageFile to hold the captured image for upload.
        StorageFile media = null;
        MediaCapture cameraCapture;
        bool IsCaptureInProgress;

        private async Task CaptureImage()
        {
            // Capture a new photo or video from the device.
            cameraCapture = new MediaCapture();
            cameraCapture.Failed += cameraCapture_Failed;

            // Initialize the camera for capture.
            await cameraCapture.InitializeAsync();

            #if WINDOWS_PHONE_APP
            cameraCapture.SetPreviewRotation(VideoRotation.Clockwise90Degrees);
            cameraCapture.SetRecordRotation(VideoRotation.Clockwise90Degrees);
            #endif

            captureGrid.Visibility = Windows.UI.Xaml.Visibility.Visible;
            previewElement.Visibility = Windows.UI.Xaml.Visibility.Visible;
            previewElement.Source = cameraCapture;
            await cameraCapture.StartPreviewAsync();
        }

        private async void previewElement_Tapped(object sender, TappedRoutedEventArgs e)
        {
            // Block multiple taps.
            if (!IsCaptureInProgress)
            {                
                IsCaptureInProgress = true;

                // Create the temporary storage file.
                media = await ApplicationData.Current.LocalFolder
                    .CreateFileAsync("capture_file.jpg", CreationCollisionOption.ReplaceExisting);

                // Take the picture and store it locally as a JPEG.
                await cameraCapture.CapturePhotoToStorageFileAsync(
                    ImageEncodingProperties.CreateJpeg(), media);

                captureButtons.Visibility = Visibility.Visible;

				// Use the stored image as the preview source.
                BitmapImage tempBitmap = new BitmapImage(new Uri(media.Path));
                imagePreview.Source = tempBitmap;
                imagePreview.Visibility = Visibility.Visible;
                previewElement.Visibility = Visibility.Collapsed;
                IsCaptureInProgress = false;
            }
        }

        private async void cameraCapture_Failed(MediaCapture sender, MediaCaptureFailedEventArgs errorEventArgs)
        {
            // It's safest to return this back onto the UI thread to show the message dialog.
            MessageDialog dialog = new MessageDialog(errorEventArgs.Message);
            await this.Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, 
                async () => { await dialog.ShowAsync(); });            
        }

        private async void ButtonCapture_Click(object sender, RoutedEventArgs e)
        {
            // Prevent multiple initializations.
            ButtonCapture.IsEnabled = false;

            await CaptureImage();
        }

        private async void ButtonRetake_Click(object sender, Windows.UI.Xaml.RoutedEventArgs e)
        {
            // Reset the capture and then start again.
            await ResetCaptureAsync();
            await CaptureImage();
        }

        private async void ButtonCancel_Click(object sender, Windows.UI.Xaml.RoutedEventArgs e)
        {
            await ResetCaptureAsync();
        }

        private async Task ResetCaptureAsync()
        {
            captureGrid.Visibility = Windows.UI.Xaml.Visibility.Collapsed;
            imagePreview.Visibility = Visibility.Collapsed;
            captureButtons.Visibility = Visibility.Collapsed;
            previewElement.Source = null;
            ButtonCapture.IsEnabled = true;

            // Make sure we stop the preview and release resources.
            await cameraCapture.StopPreviewAsync();
            cameraCapture.Dispose();
            media = null;
        }

  	这段代码显示用于捕获图像的 UI，并将图像保存到存储文件中。

6. 将现有的 `InsertTodoItem` 方法替换为以下代码：
 
        private async Task InsertTodoItem(TodoItem todoItem)
        {
            string errorString = string.Empty;

            if (media != null)
            {
                // Set blob properties of TodoItem.
                todoItem.ContainerName = "todoitemimages";

                // Use a unigue GUID to avoid collisions.
                todoItem.ResourceName = Guid.NewGuid().ToString();
            }

            // Send the item to be inserted. When blob properties are set this
            // generates an SAS in the response.
            await todoTable.InsertAsync(todoItem);

            // If we have a returned SAS, then upload the blob.
            if (!string.IsNullOrEmpty(todoItem.SasQueryString))
            {
                // Get the URI generated that contains the SAS 
                // and extract the storage credentials.
                StorageCredentials cred = new StorageCredentials(todoItem.SasQueryString);
                var imageUri = new Uri(todoItem.ImageUri);

                // Instantiate a Blob store container based on the info in the returned item.
                CloudBlobContainer container = new CloudBlobContainer(
                    new Uri(string.Format("https://{0}/{1}",
                        imageUri.Host, todoItem.ContainerName)), cred);

                // Get the new image as a stream.
                using (var inputStream = await media.OpenReadAsync())
                {
                    // Upload the new image as a BLOB from the stream.
                    CloudBlockBlob blobFromSASCredential =
                        container.GetBlockBlobReference(todoItem.ResourceName);
                    await blobFromSASCredential.UploadFromStreamAsync(inputStream);
                }

                // When you request an SAS at the container-level instead of the blob-level,
                // you are able to upload multiple streams using the same container credentials.

                await ResetCaptureAsync();
            }

            // Add the new item to the collection.
            items.Add(todoItem);
        }

	这段代码可向移动服务发送插入新 TodoItem 的请求。响应包含 SAS，然后可将其用于将图像从本地存储上载到 Azure Blob 存储。已上载的图像的 URL 用于数据绑定。

最后一个步骤是测试应用程序的两个版本并验证从两个设备上载是否成功。
		
## <a name="test"></a>测试在你的应用程序中上载图像

1. 在 Visual Studio 中，请确保 Windows 项目被设置为默认项目，然后按 F5 键运行应用程序。

2. 在“插入 TodoItem”下的文本框中输入文本，然后单击“照片”。

3. 单击或点击以拍摄照片，然后单击“上载”插入新项并上载图像。

	![](./media/mobile-services-windows-universal-dotnet-upload-to-blob-storage/mobile-quickstart-blob-appbar2.png)

4. 新项和已上载图像都显示在列表视图中。

	![](./media/mobile-services-windows-universal-dotnet-upload-to-blob-storage/mobile-quickstart-blob-ie.png)

   	>[AZURE.NOTE]新项的 *imageUri* 属性绑定到**图像**控件时，将从 Blob 存储自动下载图像。

5. 停止此应用程序并重新启动该应用程序的 Windows Phone 项目版本。

	显示以前上载的图像。

6. 和前面一样，在文本框中输入一些文本，然后单击“照片”。

   	![](./media/mobile-services-windows-universal-dotnet-upload-to-blob-storage/mobile-upload-blob-app-view-wp8.png)

3. 单击预览以拍摄照片，然后单击“上载”插入新项并上载图像。

	![](./media/mobile-services-windows-universal-dotnet-upload-to-blob-storage/mobile-upload-blob-app-view-final-wp8.png)

你已完成上载图像教程。

<!---HONumber=74-->