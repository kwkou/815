<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN image processing"
    metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, Cache refresh, content prefetching, log download, cache rules, CDN help files, CDN technical files, CDN"
    description="Learn how to use advanced features of the Azure CDN Management Portal to manage CDN endpoints"
    metaCanonical=""
    services="cdn"
    documentationCenter=".NET"
    authors="v-jijes"
    solutions=""
    manager=""
    editor="" />
<tags
    ms.service="cdn_en"
    ms.author="v-jijes"
    ms.topic="article"
    ms.date="5/8/2017"
    wacn.date="5/8/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-image-processing/)
- [English](/documentation/articles/cdn-enus-image-processing/)

# <a name="azure-cdn-"></a>Azure CDN photo processing


## <a name=""></a>Overview

Azure CDN Image Processing is a reliable, secure, and affordable image processing service provided by Azure CDN Services. Its functions include scaling, cropping, rotating, sharpening, blurring, pipeline, watermark brightness and contrast, format conversion, and getting image information. It can flexibly adapt to a variety of requirements including different device sizes, image display styles for different pages, and anti-theft watermarks.

Azure CDN Image Processing allows you to use Azure CDN Services to obtain processed versions of images from any location and any device at any time.

## <a name=""></a>Service Details

Azure CDN Image Processing was introduced as a value-added function for Azure CDN Services, so in order to use Azure CDN Image Processing, you must first create a CDN accelerated node with the acceleration type “image processing.” **The image service cannot be used as a standalone Azure service**.

There is currently no graphic user interface (GUI) for the Azure CDN image service. After you have created the corresponding CDN accelerated node, directly use the CDN accelerated domain and add image processing parameters to control the dimensions, crop mode, and quality of the image to access the processed version of the image.

## <a name=""></a>Limitations

1. Format conversion supports the following formats: JPG, PNG, BMP, WEBP, and GIF
2. The original image file must not be larger than 20MB
3. The maximum dimensions for processed images is 4,096 x 4,096 pixels, and no side may be longer than 4,096*4 pixels.

## <a name=""></a>Service Creation Process

CDN nodes with the “image processing” acceleration type can only be created from [Azure Portal Preview](https://portal.azure.cn/)

### <a name="1--cdn-profile"></a>1. Create a CDN Profile

![][1]

If images are to be stored in Azure Storage, you must ensure that the Azure subscription selected in this step is the same as the subscription used to create the Azure Storage account in Step 1. 
For the “Price Tier,” select “S1” as shown in the image above. S1 includes the new “image processing” acceleration type.

### <a name="2--cdn-"></a>2. Create a CDN accelerated node with the image processing acceleration type

![][2]

As shown in the image above, choose “image processing acceleration” for the acceleration type. Next, choose the corresponding source station type for your source station.

Once the CDN accelerated node has been configured, you can CNAME the custom domain name to the Azure CDN platform.
Once the CNAME comes into effect, you can start processing images. All original images and processed images will subsequently be accelerated by the CDN.

### <a name="3-"></a>3. Verify that an image is accessible

Before you start doing any specific processing of images, you should verify that all the configurations you’ve created so far have come into effect (if you stored the original image in an account that is not an Azure Storage account, please skip directly to 3.3; if the original image is stored in an Azure Storage account, please refer to 3.1, 3.2 and 3.3):

#### <a name="31--azure-storage--azure-storage-"></a>3.1 Create an Azure Storage account and select the Azure Storage account as the source station.

To find out how to create an Azure Storage account, see [Create a Storage Account](/documentation/articles/storage-create-storage-account/).

You can also use an existing Azure Storage account instead of creating a new one.

#### <a name="32-"></a>3.2 Upload the original image

Once you have created an Azure Storage account, you can upload the original image into the blob.

There are two methods of uploading:

- [Using Azure Portal Preview](https://portal.azure.cn)

- [Using the Azure Storage API](/documentation/articles/storage-dotnet-how-to-use-blobs/)

- [Azure Storage Explorer](http://storageexplorer.com/)

#### <a name="33-"></a>3.3 Access verification

1. Assuming the access link for the your original image is ` http://yourstorageaccount.blob.core.chinacloudapi.cn/container_name/img_name.jpg ` (first, confirm that this link is accessible, i.e. the corresponding container and blob files are publicly accessible).

2. If the CDN-accelerated access link for the original image (taking the custom domain name used in the image above as an example) ` http://imgprocess.yourcompany.cn/container_name/img_name.jpg ` can be accessed, you can be sure that all the configurations are in effect.

<a name=""></a>Image Processing
--

## <a name=""></a>Image Processing Access Rules

Access is provided using the URL below:

    http://your_CDN_custom_domain/container_name/image_object?basic=<处理字符串>

**your\_CDN\_custom\_domain**: The custom domain name you used to create the image processing acceleration-type CDN node, e.g. `imgprocess.yourcompany.cn`

**container_name**: The name of the Azure Storage container where the original image is located

**image_object**: The name of the original image for processing, e.g. `image.jpg`

A specific explanation of `处理字符串` is provided below.

## <a name=""></a>Image Processing Syntax

### <a name=""></a>Detailed definitions for processing strings

`处理字符串` A combination of parameters in a form such as `{值}{参数关键字}`, wherein all the parameters are linked by `_`.

To perform format conversion, `Process String` must have the ending `.{目标格式扩展名}`.

### <a name=""></a>Supported basic image operations (parameters)

| Operation name | Syntax | Notes |
|:----------|:-------------------------------------------------------------|:-------------------------------------|
| Scale and crop | <p>`{Percentage}p`, `{Height}h`, `{Width}w`, `{ScaleType}e`, `{Cut}c`, `{HandleIfLarger}l`, `{Red}-{Green}-{Blue}bgc`. If you wish to use the `{Red}-{Green}-{Blue}bgc` parameter, it must be used together with the `{ScaleType}e`, `{Height}h` and `{Width}w` parameters. See the table below for details</p> ||
| Advanced crop | <p>`{x}-{y}-{Width}-{Height}a`, within which `{x}` and `{y}` respectively define the horizontal and vertical coordinates of the crop start position, `{Width}` and `{Height}` respectively define the width and height of the crop region in the range 1-4096. </p>| <p>A crop start position outside the range of the original image will cause a 400 `Bad Request`, while a crop that is beyond the original image range or where the specified width and height are zero will crop up to the end position of the original image </p>|
| Region crop | <p>`{Width}x{Height}-{Position}`rc, in which `{Width}` and `{Height}` respectively define the width and height of the crop region in the range 0-4096, and the crop will be performed by the method defined by `{Position}`; see the table below for the implications of different values</p>| <p>Setting width/height values larger than the original dimensions will return the original image, while an empty `Height` or `Width` value or a value of zero indicates that the height/width of the original image will be used</p> |
| Inscribed circle | <p>`{Radius}-{Type}ci`, within which `{Radius}` specifies the radius of the inscribed circle, `{Type}`: an integer with a value of 0 or 1, where 1 means that the image returned will be cropped to the smallest square shape that can accommodate the inscribed circle; otherwise it will not be cropped</p> ||
| Rounded rectangle | <p>`{Radius}-2ci`, within which `{Radius}` specifies the radius of the rounded corners</p> | The rounded corner radius cannot be larger than half the length of the shortest side of the original image |
| Indexed crop |<p> `{length}x-{index}ic` or `{length}y-{index}ic`, respectively correspond to performing a slice based on the horizontal coordinates or the vertical coordinates. This returns an image composed of the first several such slices, within which `{Length}` is the width of the slice and `{index}` is the sequence number of the slices returned</p> | The slice sequence numbers start from zero, while the original image will be returned if the number is larger than the slice setting for the original image |
| Rotate |<p> `{Degree}r`, within which `{Degree}` represents the degrees of clockwise rotation performed on the image</p> ||
| Sharpen | <p>`{Degree}sh`, within which `{Degree}` is a non-negative integer that specifies the degree of sharpening</p> ||
| Blur |<p> `{Radius}-{Sigma}bl`, within which `{Radius}` is the blur radius and `{Sigma}` is the blur’s standard deviation</p> ||
| Brightness and contrast | <p>`{Brightness}b` and `{Contrast}d`, wherein `{Brightness}` is an integer that may be negative and indicates the adjustment in brightness relative to the original image, while `{Contrast}` is an integer that may be negative and indicates the adjustment in contrast relative to the original image</p> ||
| Image quality | <p>`{Degree}q`, within which `{Degree}` is an integer value indicating the image quality in the range 0-100</p> | <p>Only valid when the output format is `jpg/jpeg`.</p> |
| Format conversion | <p>`.{format}`, within which `{format}` is the target format</p> | <p>The following formats are currently supported: `jpg`, `jpeg`, `png`, `webp` and `gif`</p> |
| Progressive display | <p>`{Setting}pr`, within which `{Setting}` is an integer in the range 0-1, and the image will be progressively presented to the user as it is loaded when this value is 1</p> | <p>Only valid when the output format is `jpg/jpeg`. </p>|
| Image information | <p>`info`, returns the image details in JSON format; see the corresponding subsection below</p> | Cannot be used together with other parameters |
| Image EXIF information |<p> `exif`, returns the image’s EXIF information in JSON format; see the API examples below for details </p>| Cannot be used together with other parameters |
| Image main color | <p>`imageAve`, returns the image main color information in JSON format; see the corresponding subsection below</p> | Cannot be used together with other parameters |
| Pipeline | <p>`{Process1}`&#124;`{Process2}`, within which `Process1` and `Process2` are both `处理字符串` and the image is first processed using the operation process defined by `Process1`, then processed using the operation process defined by `Process2`.</p>| <p>`Process` may be any of the parameters above except `图片信息``EXIF 信息` and `图片主色调`.</p> |
| Watermark | See the corresponding subsection below | Cannot coexist with other parameters in the same section of the pipeline |

### <a name=""></a>Scale and crop

#### <a name=""></a>Parameter

| Name | Description |
| :---- | :---------------------------------------------------------------------------------------------- |
| <p>`{Percentage}`</p> | An integer with a value in the range 1-1,000 that indicates the processing performed on the width/height |
| <p>`{Height}`, `{Width}`</p> | The height value and width value of the required image, which must be a non-negative integer that does not exceed the dimension limits |
| <p>`{ScaleType}`</p> | An integer that defines the scale mode; see the table below for details |
| <p>`{Cut}`</p> | An integer with a value of 0 or 1, wherein no other operations are performed when it is the default value 0, but the original image is automatically cropped to specified dimensions when the value is 1 |
|<p> `{HandleIfLarger}`</p> | An integer with a value of 0 or 1, wherein the default value 1 permits the image to be scaled up to dimensions larger than the original dimensions, while a value of 0 means that the original image dimensions are used when the specified dimensions are larger than the original image |
| <p>`{Red}`, `{Green}` and `{Blue}`</p> | Integer values in the range 0-255, which respectively represent the red, green, and blue channel values filled in in the RGB color space |

__Note:__

* _The results of processing in any situation must not make the image dimensions larger than the limits. See the `限制`section_ in `简介` for specific limits.

* _`Percentage`May be used together with `Height` or `Width`; the image will be scaled to a size where the specified width and height are multiplied by a specified percentage_ 

* _, `Height` and `Width` can all exist independently, and any of the values that are missing will default to the original image dimensions_

*  _and `Percentage` may exist independently, and the image will be scaled to a size where the original dimensions are multiplied by a specific percentage_

*  _and `Cut` only take effect when they coexist with `Height` and `Width`_

* The background color  _must be used together with `{ScaleType}` 4_

#### <a name=""></a>Scale mode

| Value | Presentation |
| :---- | :--------------------------------------------------------------------------------------- |
| 0 or undefined | Locked aspect ratio and long side prioritized |
| 1 | Locked aspect ratio and short side prioritized |
| 2 | Forced width/height |
| 4 | Locked aspect ratio and short side prioritized, and fill in empty regions with a specified color after scaling (automatically filled in white if the color is not specified). |

**Note:** The concepts of `长边` and `短边` here are determined based on the **ratio**of `原始长度 / 目标长度`.

### <a name=""></a>Image information response body

Includes at least the JSON objects for the following strings:

| Name | Content |
|:--- | :--------------------------------- | 
| Width | Image width in pixels |
| Height | Image height in pixels |
| Size | Image file size in bytes |
| MimeType | Original image file type |

Response body example

    {
        "Width": 221,
        "Height": 284,
        "Size": 119540,
        "AveColor": null,
        "MimeType": "Jpeg"
    }

### <a name=""></a>Image main color response header

Includes at least the JSON objects for the following strings:

| Name | Content |
|:--- | :--------------------------------- | 
| RGB | The image’s main colors indicated by a hexadecimal value in the RGB color space |

#### <a name=""></a>Response body example

JSON

    {
        "RGB": "273B2A"
    }

### <a name="-"></a>Example (updates ongoing)

Assuming ` http://imgprocess.yourcompany.cn/container_name/img_name.jpg ` is the original image URL that has undergone Azure CDN acceleration.

![][3]

#### <a name="1--60"></a>1. Scale the image down to 60% of the original size:

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=60p

![][4]

#### <a name="2--20"></a>2. Process the original image with rounded corners (with a corner radius of 20):

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=20-2ci

![][5]

#### <a name="3--60-20"></a>3. First, scale the image down to 60% of the original size, then process the image with rounded corners (with a corner radius of 20):

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=60p_20-2ci 

![][6]

#### <a name="4-"></a>4. Get the image information:

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=info

JSON

    {
        "Width": 800,
        "Height": 400,
        "Size": 87341,
        "AveColor": null,
        "MimeType": "Jpeg"
    }

#### <a name="5--exif-"></a>5. Get the image’s EXIF information:

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=exif

JSON

    {
        "Orientation": 1,
        "XResolution": 72.0,
        "YResolution": 72.0,
        "ResolutionUnit": 2,
        "Software": "Adobe Photoshop CS5 Windows",
        "DateTime": "2014:04:15 16:43:14",
        "ColorSpace": 1,
        "PixelXDimension": 800,
        "PixelYDimension": 400
    }

#### <a name="6-"></a>6. Get the image’s main colors:

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=imageAve

JSON

    {
        "RGB": "0296C8"
    }

### <a name=""></a>Watermark operations

The watermark function’s `处理字符串` is a query string that may include the following fields:

| Name | Description |
|:----|:--------------------------------------------------------------------------------------------------------------|
| watermark | Integer that defines the watermark type, value in the range 1-3; see the table below for detailed explanations |
| p | Integer that defines the watermark position, value in the range 1-9; see the table below for detailed explanations |
| s | Integer that defines the watermark transparency, value in the range 0-100 |
| text | <p>String that has been `Base64`-encoded and defines the watermark content</p> |
| type | <p>String that has been `Base64`-encoded and defines the name of the font used for the watermark text; see the table below for supported fonts</p> |
| color | <p>Six-digit hexadecimal number in the 8-bit RGB color space that has been `Base64` encoded and defines the watermark text color. For example `白色`, with a value of `ffffff`, `Base64` after encoding is `ZmZmZmZm`. </p>|
| size | Positive integer that defines the watermark text font size |
| object | <p>String that has been `Base64`-encoded and defines the filename for the image used in the image watermark</p> |

**Notes** 

* The `watermark` field is mandatory and must be the first field in the query string
* Images used in image watermarks must be stored in the same Azure Storage account as the original image
* Base64 encoding conversion can be directly processed using online tools, such as [base64encode](https://www.base64encode.org/)
* The file content and filename distinguish between upper and lower case, so pay attention to capitalization when performing base64 transcoding

#### <a name=""></a>Watermark type

| Value | Definition |
| :---- | :---- |
| 1 | Image watermark |
| 2 | Text watermark |

#### <a name=""></a>Watermark location

| Value | Definition |
| :---- | :---- |
| 1 | Top left |
| 2 | Top |
| 3 | Top right |
| 4 | Left side |
| 5 | Center |
| 6 | Right side |
| 7 | Bottom left |
| 8 | Bottom |
| 9 | Bottom right |

#### <a name=""></a>Supported fonts

| Name | Definition | Base64 encoding |
|:------  |:--------|:----------------------|
| dengxian | Sans Serif | ZGVuZ3hpYW4= |
| fangsong| Fang Song | ZmFuZ3Nvbmc= |
| fangzhengshuti | FZShuTi | ZmFuZ3poZW5nc2h1dGk= | 
| fangzhengyaoti | FZYaoTi | ZmFuZ3poZW5neWFvdGk= |
| kaiti | Kaiti | a2FpdGk= |
| lishu | Lishu | bGlzaHU= |
| microsoftyahei | Microsoft YaHei | bWljcm9zb2Z0eWFoZWk= | 
| microsoftyaheiui | Microsoft YaHei UI | bWljcm9zb2Z0eWFoZWl1aQ== |
| xinsongti | NSimSun | eGluc29uZ3Rp |
| heiti | Heiti | aGVpdGk= |
| songti | SimSun | c29uZ3Rp |
| huawencaiyun | STCaiyun | aHVhd2VuY2FpeXVu |
| huawenfangsong | STFangsong | aHVhd2VuZmFuZ3Nvbmc= |
| huawenhupo | STHupo | aHVhd2VuaHVwbw== |
| huawenkaiti | STKaiti | aHVhd2Vua2FpdGk= |
| huawenlishu | STLiti | aHVhd2VubGlzaHU= |
| huawensongti | STSong | aHVhd2Vuc29uZ3Rp |
| huawenxihei | STXihei | aHVhd2VueGloZWk= |
| huawenxingkai | STXingkai | aHVhd2VueGluZ2thaQ== |
| huawenxinwei | STXinwei | aHVhd2VueGlud2Vp |
| huawenzhongsong | STZhongsong | aHVhd2Vuemhvbmdzb25n |
| youyuan | Simyou | eW91eXVhbg== |

#### <a name=""></a>Sample

Assuming the watermark image file exists and uses `container_name/watermark.jpg`, the corresponding Base64 encoding for it is `Y29udGFpbmVyX25hbWUvd2F0ZXJtYXJrLmpwZw==`

![][9]

#### <a name="1-"></a>1. Add a watermark image to the original image:

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=watermark=1;p=3;s=50;object=Y29udGFpbmVyX25hbWUvd2F0ZXJtYXJrLmpwZw==

![][7]

#### <a name="2--60-20"></a>2. The original image is first scaled down to 60% of the original size, then processed with rounded corners (with a corner radius of 20), and finally an image watermark is added (note that this uses the pipeline **|** operator):

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=60p_20-2ci|watermark=1;p=3;s=50;object=Y29udGFpbmVyX25hbWUvd2F0ZXJtYXJrLmpwZw==

![][8]

#### <a name="3--60-20-azure-china-cdn"></a>3. The original image is first scaled down to 60% of the original size, then processed with rounded corners (with a corner radius of 20), and finally an image watermark is added (note that this uses the pipeline **|** operator), the text content is “Azure China CDN” and the font is “Microsoft Yahei”:

    http://imgprocess.yourcompany.cn/container_name/img_name.jpg?basic=60p_20-2ci|watermark=2;p=3;s=50;text=QXp1cmUgQ2hpbmEgQ0RO;type=bWljcm9zb2Z0eWFoZWk=

![][10]

<!--Image references-->
[1]: ./media/cdn-image-processing/imgp01.png
[2]: ./media/cdn-image-processing/imgp02.png
[3]: ./media/cdn-image-processing/azure.jpg
[4]: ./media/cdn-image-processing/60p.jpg
[5]: ./media/cdn-image-processing/20ci.jpg
[6]: ./media/cdn-image-processing/60p_20ci.jpg
[7]: ./media/cdn-image-processing/w1.jpg
[8]: ./media/cdn-image-processing/w2.jpg
[9]: ./media/cdn-image-processing/cdn_watermark.png
[10]: ./media/cdn-image-processing/w3.jpg

<!--HONumber=May17_HO3-->