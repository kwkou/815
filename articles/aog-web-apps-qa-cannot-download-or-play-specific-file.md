<properties
	pageTitle="web 应用中如何下载或播放特殊类型的文件"
	description="web 应用中如何下载或播放特殊类型的文件"
	service=""
	resource="webapps"
	authors=""
	displayOrder=""
	selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Web Apps, MIME"​
    cloudEnvironments="MoonCake" />
<tags
	ms.service="app-service-web-aog"
	ms.date=""
	wacn.date="1/20/2017" />
# web 应用中如何下载或播放特殊类型的文件

## **问题描述**

在使用 Azure Web 应用时，处理特殊类型的文件会提示无法下载或者无法播放。

## **解决办法**

IIS 服务默认不提供对于某些特殊文件格式的支持，例如：`.apk`，需要在 `Web.config` 中增加对于该类型的 MIME 声明,如下所示：

	<configuration>
		<system.webServer>
			<staticContent>
				<mimeMap fileExtension=”.apk” mimeType=”application/vnd.android” />
			</staticContent>
		</system.webServer>
	</configuration>

## **参考文档**

其它类型的特殊文件类型的 MIME 声明请参考下面的链接：<br>
- [MIME Types in IIS](https://msdn.microsoft.com/en-us/library/bb742440.aspx)<br>
- [Adding Mime Types to your Windows Azure Web Site](https://blogs.iis.net/richma/adding-mime-types-to-your-windows-azure-web-site)

