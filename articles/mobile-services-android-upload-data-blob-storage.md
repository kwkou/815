<properties 
	pageTitle="使用移动服务将图像上载到 Blob 存储 (Android) | 移动服务" 
	description="了解如何使用移动服务将图像上载到 Azure 存储空间以及从 Android 应用访问图像。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="erikre"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="04/11/2016"
	wacn.date="06/13/2016"/>

# 从 Android 设备将图像上载到 Azure 存储空间

[AZURE.INCLUDE [mobile-services-selector-upload-data-blob-storage](../includes/mobile-services-selector-upload-data-blob-storage.md)]

本主题演示如何使 Android Azure 移动服务应用能够将图像上载到 Azure 存储空间。

移动服务使用 SQL 数据库存储数据。但是，在 Azure 存储空间中存储二进制大型对象 (BLOB) 数据更有效。在本教程中，你将使移动服务快速入门应用能够使用 Android 相机拍照，并将图像上载到 Azure 存储空间。


## 入门所需操作

在开始本教程之前，必须先完成移动服务快速入门：[移动服务入门]。

本教程还需要以下项：

+ [Azure 存储帐户](/documentation/articles/storage-create-storage-account/)
+ 带相机的 Android 设备

## 应用的工作方式

上载照片图像是一个多步骤过程：

- 首先你拍照，然后在 SQL 数据库中插入一个 TodoItem 行，其中包含 Azure 存储空间使用的新元数据字段。
- 新的移动服务 SQL **插入**脚本要求 Azure 存储空间提供共享访问签名 (SAS)。
- 该脚本将 SAS 和 blob 的 URI 返回给客户端。
- 客户端使用 SAS 和 blob URI 上载照片。

那么，SAS 是什么？

在客户端应用内部存储将数据上载到 Azure 存储服务程序所需的凭据是不安全的。你应将这些凭据存储在移动服务中，并使用它们来生成用于授权上载新图像的共享访问签名 (SAS)。移动服务会向客户端应用安全返回 SAS（一个具有 5 分钟到期期限的凭据）。然后，应用程序将使用此临时凭据来上载图像。有关详细信息，请参阅[共享访问签名，第 1 部分：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)

## 代码示例
[此处](https://github.com/Azure/mobile-services-samples/tree/master/UploadImages)提供了此应用的已完成客户端源代码部分。若要运行此代码，你必须先完成本教程的移动服务后端部分。

## 在 Azure 管理门户中更新已注册的插入脚本

[AZURE.INCLUDE [mobile-services-configure-blob-storage](../includes/mobile-services-configure-blob-storage.md)]


## 更新快速入门应用以捕获和上载图像

### 引用 Azure 存储空间 Android 客户端库

1. 若要添加对库的引用，请在“应用”> **build.gradle** 文件中，向 `dependencies` 节添加以下行：

		compile 'com.microsoft.azure.android:azure-storage-android:0.6.0@aar'


2. 将 `minSdkVersion` 值更改为 15（相机 API 所需的值）。

3. 按“将项目与 Gradle 文件同步”图标。

### 更新相机和存储的清单文件

1. 通过向 **AndroidManifest.xml** 添加以下行，指定你的应用需要让相机可用：

	    <uses-feature android:name="android.hardware.camera"
	        android:required="true" />

2. 通过向 **AndroidManifest.xml** 添加以下行，指定你的应用需要写入到外部存储的权限：

	    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

	请注意，Android 外部存储不一定是 SD 卡：有关详细信息，请参阅[保存文件](http://developer.android.com/training/basics/data-storage/files.html)。

### 更新新用户界面的资源文件

1. 通过向 *values* 目录中的 **strings.xml** 文件添加以下内容，为新按钮添加标题：

	    <string name="preview_button_text">Take Photo</string>
	    <string name="upload_button_text">Upload</string>

2. 在 **res => layout** 文件夹中的 **activity\_to\_do.xml** 文件中，在“添加”按钮的现有代码前添加以下代码。

         <Button
             android:id="@+id/buttonPreview"
             android:layout_width="120dip"
             android:layout_height="wrap_content"
             android:onClick="takePicture"
             android:text="@string/preview_button_text" />

3. 将“添加”按钮元素替换为以下代码：

         <Button
             android:id="@+id/buttonUpload"
             android:layout_width="100dip"
             android:layout_height="wrap_content"
             android:onClick="uploadPhoto"
             android:text="@string/upload_button_text" />


### 添加照片拍摄的代码

1. 在 **ToDoActivity.java** 中添加以下代码以创建一个具有唯一名称的**文件**对象。

		// Create a File object for storing the photo
	    private File createImageFile() throws IOException {
	        // Create an image file name
	        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
	        String imageFileName = "JPEG_" + timeStamp + "_";
	        File storageDir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES);
	        File image = File.createTempFile(
	                imageFileName,  /* prefix */
	                ".jpg",         /* suffix */
	                storageDir      /* directory */
	        );
	        return image;
	    }

2. 添加以下代码以启动 Android 相机应用。然后可以拍照，如果有照片看起来不错，请按“保存”，这会将其存储在你刚创建的文件中。

		// Run an Intent to start up the Android camera
	    static final int REQUEST_TAKE_PHOTO = 1;
	    public Uri mPhotoFileUri = null;
	    public File mPhotoFile = null;
		
	    public void takePicture(View view) {
	        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
	        // Ensure that there's a camera activity to handle the intent
	        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
	            // Create the File where the photo should go
	            try {
	                mPhotoFile = createImageFile();
	            } catch (IOException ex) {
	                // Error occurred while creating the File
	                //
	            }
	            // Continue only if the File was successfully created
	            if (mPhotoFile != null) {
	                mPhotoFileUri = Uri.fromFile(mPhotoFile);
	                takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, mPhotoFileUri);
	                startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
	            }
	        }
	    }

### 添加代码以将照片文件上载到 blob 存储中


1. 首先，我们通过向 **ToDoItem.java** 添加以下代码，将一些新的元数据字段添加到 `ToDoItem` 对象中。

		/**
	     *  imageUri - points to location in storage where photo will go
	     */
	    @com.google.gson.annotations.SerializedName("imageUri")
	    private String mImageUri;
	
	    /**
	     * Returns the item ImageUri
	     */
	    public String getImageUri() {
	        return mImageUri;
	    }
	
	    /**
	     * Sets the item ImageUri
	     *
	     * @param ImageUri
	     *            Uri to set
	     */
	    public final void setImageUri(String ImageUri) {
	        mImageUri = ImageUri;
	    }
	
	    /**
	     * ContainerName - like a directory, holds blobs
	     */
	    @com.google.gson.annotations.SerializedName("containerName")
	    private String mContainerName;
	
	    /**
	     * Returns the item ContainerName
	     */
	    public String getContainerName() {
	        return mContainerName;
	    }
	
	    /**
	     * Sets the item ContainerName
	     *
	     * @param ContainerName
	     *            Uri to set
	     */
	    public final void setContainerName(String ContainerName) {
	        mContainerName = ContainerName;
	    }
	
	    /**
	     *  ResourceName
	     */
	    @com.google.gson.annotations.SerializedName("resourceName")
	    private String mResourceName;
	
	    /**
	     * Returns the item ResourceName
	     */
	    public String getResourceName() {
	        return mResourceName;
	    }
	
	    /**
	     * Sets the item ResourceName
	     *
	     * @param ResourceName
	     *            Uri to set
	     */
	    public final void setResourceName(String ResourceName) {
	        mResourceName = ResourceName;
	    }
	
	    /**
	     *  SasQueryString - permission to write to storage
	     */
	    @com.google.gson.annotations.SerializedName("sasQueryString")
	    private String mSasQueryString;
	
	    /**
	     * Returns the item SasQueryString
	     */
	    public String getSasQueryString() {
	        return mSasQueryString;
	    }
	
	    /**
	     * Sets the item SasQueryString
	     *
	     * @param SasQueryString
	     *            Uri to set
	     */
	    public final void setSasQueryString(String SasQueryString) {
	        mSasQueryString = SasQueryString;
	    }

2. 使用以下代码替换 0 参数构造函数：

	    /**
	     * ToDoItem constructor
	     */
	    public ToDoItem() {
	        mContainerName = "";
	        mResourceName = "";
	        mImageUri = "";
	        mSasQueryString = "";
	    }

3. 使用以下代码替换多参数构造函数：

	    /**
	     * Initializes a new ToDoItem
	     *
	     * @param text
	     *            The item text
	     * @param id
	     *            The item id
	     */
	    public ToDoItem(String text,
	                    String id,
	                    String containerName,
	                    String resourceName,
	                    String imageUri,
	                    String sasQueryString) {
	        this.setText(text);
	        this.setId(id);
	        this.setContainerName(containerName);
	        this.setResourceName(resourceName);
	        this.setImageUri(imageUri);
	        this.setSasQueryString(sasQueryString);
	    }


4. 在 **ToDoActivity.java** 文件中，将 **ToDoActivity.java** 中的 **addItem** 方法替换为以下代码，以便上载图像。

	    /**
	     * Add a new item
	     *
	     * @param view
	     *            The view that originated the call
	     */
	    public void uploadPhoto(View view) {
	        if (mClient == null) {
	            return;
	        }
	
	        // Create a new item
	        final ToDoItem item = new ToDoItem();
	
	        item.setText(mTextNewToDo.getText().toString());
	        item.setComplete(false);
	        item.setContainerName("todoitemimages");
	
	        // Use a unigue GUID to avoid collisions.
	        UUID uuid = UUID.randomUUID();
	        String uuidInString = uuid.toString();
	        item.setResourceName(uuidInString);
	
	        // Send the item to be inserted. When blob properties are set this
	        // generates an SAS in the response.
	        AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>(){
	            @Override
	            protected Void doInBackground(Void... params) {
	                try {
		                    final ToDoItem entity = addItemInTable(item);
		
		                    // If we have a returned SAS, then upload the blob.
		                    if (entity.getSasQueryString() != null) {
		
	                       // Get the URI generated that contains the SAS
	                        // and extract the storage credentials.
	                        StorageCredentials cred = 
								new StorageCredentialsSharedAccessSignature(entity.getSasQueryString());
	                        URI imageUri = new URI(entity.getImageUri());
	
	                        // Upload the new image as a BLOB from a stream.
	                        CloudBlockBlob blobFromSASCredential =
	                                new CloudBlockBlob(imageUri, cred);
	
	                        blobFromSASCredential.uploadFromFile(mPhotoFileUri.getPath());
  	                    }
	
	                    runOnUiThread(new Runnable() {
	                        @Override
	                        public void run() {
	                            if(!entity.isComplete()){
	                                mAdapter.add(entity);
	                            }
	                        }
	                    });
	                } catch (final Exception e) {
	                    createAndShowDialogFromTask(e, "Error");
	                }
	                return null;
	            }
	        };
	
	        runAsyncTask(task);
	
	        mTextNewToDo.setText("");
	    }
	

这段代码可向移动服务发送插入新 TodoItem 的请求。响应包含 SAS，然后可将其用于将图像从本地存储上载到 Azure 存储空间中的 Blob。


## 测试上载图像 

1. 在 Android Studio 中按“运行”。在对话框中，选择要使用的设备。

2. 当应用 UI 出现时，在标记为“添加 ToDo 项”的文本框中输入文本。

3. 按“拍照”。当相机应用启动时，拍摄一张照片。按复选标记以接受该照片。

4. 按“上载”。注意 ToDoItem 如何像往常一样已添加到列表中。

5. 在 Azure 经典门户中，转到你的存储帐户，按“容器”选项卡，然后在列表中按你的容器的名称。

6. 此时将显示已上载的 blob 文件列表。选择其中一个，然后按“下载”。

7. 现在，你上载的图像将显示在浏览器窗口中。


## <a name="next-steps"></a>后续步骤

现在，你已能够通过将移动服务与 Blob 服务集成安全地上载图片，请查看一些其他的后端服务和集成主题：

+ [使用 SendGrid 从移动服务发送电子邮件]
 
	了解如何使用 SendGrid 电子邮件服务为你的移动服务添加电子邮件功能。本主题演示如何添加服务器端脚本，以使用 SendGrid 发送电子邮件。

+ [移动服务服务器脚本参考]

	参考使用服务器脚本执行服务器端任务，并与其他 Azure 组件和外部资源集成的主题。
  
 
<!-- Anchors. -->
[Install the Storage Client library]: #install-storage-client
[Update the client app to capture images]: #add-select-images
[Update the insert script to generate an SAS]: #update-scripts
[Upload images to test the app]: #test
[Next Steps]: #next-steps

<!-- Images. -->

[2]: ./media/mobile-services-windows-store-dotnet-upload-data-blob-storage/mobile-add-storage-nuget-package-dotnet.png


<!-- URLs. -->
[使用 SendGrid 从移动服务发送电子邮件]: /documentation/articles/store-sendgrid-mobile-services-send-email-scripts/

[Send push notifications to Windows Store apps using Service Bus from a .NET back-end]: http://go.microsoft.com/fwlink/?LinkId=277073&clcid=0x409
[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts/
[移动服务入门]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/
[Azure classic portal]: https://manage.windowsazure.cn/
[How To Create a Storage Account]: /documentation/articles/storage-create-storage-account/
[Azure Storage Client library for Store apps]: http://go.microsoft.com/fwlink/p/?LinkId=276866

[App settings]: http://msdn.microsoft.com/zh-cn/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7
 

<!---HONumber=Mooncake_0118_2016-->