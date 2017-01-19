<properties 
	pageTitle="C# SDK 调用认知服务时提示 Key 无效" 
	description="C# SDK 调用认知服务时提示 Key 无效" 
	service="microsoft.cognitiveservices"
	resource="cognitiveservices"
	authors=""
	displayOrder=""
	selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Cognitive Services, C# SDK, Key"​
    cloudEnvironments="MoonCake" 
/>
<tags 
	ms.service="cognitive-services-aog"
	ms.date="" 
	wacn.date="01/12/2017"
/>
# C# SDK 调用认知服务时提示 Key 无效

## **问题描述**

使用 C# 的 SDK 无法调用认知服务。

## **问题现象**

确保在 China Azure 的管理门户中获取的 key 准确无误后，使用该 key 调用认知服务，却提示 key 无效。

经过分析，发现是因为 SDK 调用的认知服务指向了 Global 版本。

## **解决方法**

通过以下 2 步解决该问题：

1.	下载最新版本 C# 对应的SDK。
2.	修改初始化人脸识别对象的方法，添加指向中国版认知服务的 endpoint。

		faceServiceClient = new FaceServiceClient("<key>", "https://api.cognitive.azure.cn/face/v1.0");





