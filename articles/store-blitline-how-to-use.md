<properties 
	pageTitle="如何使用 Blitline 处理图像 - Azure 功能指南" 
	description="了解如何使用 Blitline 服务在 Azure 应用程序中处理图像。" 
	services="" 
	documentationCenter=".net" 
	authors="blitline-dev" 
	manager="jason@blitline.com" 
	editor="jason@blitline.com"/>

<tags 
	ms.service="multiple" 
	ms.date="12/09/2014" 
	wacn.date="10/3/2015"/>






# 如何将 Blitline 与 Azure 和 Azure 存储一起使用

本指南将说明如何访问 Blitline 服务以及如何将作业提交到 Blitline。

## 目录

[什么是 Blitline？][] [Blitline 无法实现的操作][] [创建 Blitline 帐户][] [如何创建 Blitline 作业][] [ 如何将图像保存到 Azure 存储空间][] [后续步骤][]

## <a id="whatis"></a>什么是 Blitline？

Blitline 是一项基于云的图像处理服务，此服务提供了企业级图像处理，其成本仅为客户自行进行图像处理所产生的成本的一小部分。

实际上，我们已反复地执行过图像处理，对于所有网站，通常是从头开始重新执行此操作。我们之所以认识到这一点，是因为我们也已执行此操作上百万次。于是我们认为也许该让所有人都了解这一点了。我们知道如何快速而高效地进行图像处理，同时为每个人节省时间。

有关详细信息，请参阅 [http://www.blitline.com](http://www.blitline.com)。

## <a id="whatisnot"></a>Blitline 无法实现的操作...

为了阐明 Blitline 的用处，不妨先介绍 Blitline 无法实现的操作。

- Blitline 没有用于上载图像的 HTML 小组件。你必须公开图像或具有 Blitline 可获得的受限权限。

- Blitline 不会像 Aviary.com 一样进行实时图像处理

- Blitline 不接受图像上载，你无法将图像直接推送到 Blitline。你必须将图像推送到 Azure 存储或 Blitline 支持的其他位置，然后告知 Blitline 可获取该图像的位置。

- Blitline 会执行高度并行处理，而不执行任何同步处理。这意味着，你必须向我们提供 postback\_url，并且我们会告诉你何时处理完毕。

## <a id="createaccount"></a>创建 Blitline 帐户

[AZURE.INCLUDE [blitline-signup](../includes/blitline-signup.md)]

## <a id="createjob"></a>如何创建 Blitline 作业

Blitline 使用 JSON 定义要对图像执行的操作。此 JSON 由几个简单的字段构成。

最简单的示例如下所示：

	    json : '{
       "application_id": "MY_APP_ID",
       "src" : "http://cdn.blitline.com/filters/boys.jpeg",
       "functions" : [ {
           "name": "resize_to_fit",
           "params" : { "width": 240, "height": 140 },
           "save" : { "image_identifier" : "external_sample_1" }
       } ]
    }'

此处的 JSON 将采用“src”图像“...boys.jpeg”，然后将图像的大小调整为 240x140。

可以在 Azure 上的“连接信息”或“管理”选项卡中找到应用程序 ID。此 ID 是可供你用来在 Blitline 上运行作业的机密标识符。

“save”参数标识有关你在我们处理图像后要将图像放置到的位置的信息。在此常见示例中，我们没有定义该位置。如果未定义位置，Blitline 会将图像本地（临时）存储在一个唯一的云位置。你将能够在创建 Blitline 时从 Blitline 返回的 JSON 中获取该位置。“image”标识符是必需的，在标识此特定的保持图像时将返回此标识符。

可以在以下位置查找有关我们支持的*函数*的详细信息：<http://www.blitline.com/docs/functions>

还可以在以下位置查找有关作业选项的文档：<http://www.blitline.com/docs/api>

获得 JSON 后，您只需将其 **POST** 到 `http://api.blitline.com/jobs`

你将找回与下面类似的 JSON：

    {
     "results":
         {"images":
            [{
              "image_identifier":"external_sample_1",
              "s3_url":"https://s3.amazonaws.com/dev.blitline/2011110722/YOUR_APP_ID/CK3f0xBF_2bV6wf7gEZE8w.jpg"
            }],
          "job_id":"4eb8c9f72a50ee2a9900002f"
         }
    }


这意味着，Blitline 已接收您的请求，它已将该请求置于处理队列中，可在图像完成后在以下位置找到该图像：****https://s3.amazonaws.com/dev.blitline/2011110722/YOUR\_APP\_ID/CK3f0xBF_2bV6wf7gEZE8w.jpg**

## <a id="saveazure"></a>如何将图像保存到 Azure 存储帐户

如果你拥有 Azure 存储帐户，则可以让 Blitline 轻松地将处理过的图像推送到你的 Azure 容器中。通过添加“azure\_destination”，可定义 Blitline 将推送到的位置和相关权限。

下面是一个示例：

    job : '{
      "application_id": "YOUR_APP_ID",
      "src" : "http://www.google.com/logos/2011/houdini11-hp.jpg",
         "functions" : [{
         "name": "blur",
         "save" : {
             "image_identifier" : "YOUR_IMAGE_IDENTIFIER",
             "azure_destination" : {
                 "account_name" : "YOUR_AZURE_CONTAINER_NAME",
                 "shared_access_signature" : "SAS_THAT_GIVES_BLITLINE_PERMISSION_TO_WRITE_THIS_OBJECT_TO_CONTAINER",
               }
           }
         }]
       }'


通过自行填写大写的值，可以将此 JSON 提交到 http://api.blitline.com/job，并且将使用模糊滤镜处理“src”图像，然后将该图像推送到您的 Azure 目标。

<h3>请注意：</h3>

SAS 必须包含完整 SAS URL，包括目标文件的文件名。

示例：

    http://blitline.blob.core.windows.net/sample/image.jpg?sr=b&sv=2012-02-12&st=2013-04-12T03%3A18%3A30Z&se=2013-04-12T04%3A18%3A30Z&sp=w&sig=Bte2hkkbwTT2sqlkkKLop2asByrE0sIfeesOwj7jNA5o%3D


您还可以在[此处](http://www.blitline.com/docs/azure_storage)参阅最新版本 Blitline 的 Azure 存储空间文档。


## <a id="nextsteps"></a>后续步骤

访问 blitline.com 可参阅有关我们所有其他功能的信息：

* Blitline API 终结点文档 <http://www.blitline.com/docs/api>
* Blitline API 函数 <http://www.blitline.com/docs/functions>
* Blitline API 示例 <http://www.blitline.com/docs/examples>
* 第三方 Nuget 库 <http://nuget.org/packages/Blitline.Net>


  [后续步骤]: #nextsteps
  [什么是 Blitline？]: #whatis
  [Blitline 无法实现的操作]: #whatisnot
  [创建 Blitline 帐户]: #createaccount
  [如何创建 Blitline 作业]: #createjob
  [ 如何将图像保存到 Azure 存储空间]: #saveazure

<!---HONumber=71-->