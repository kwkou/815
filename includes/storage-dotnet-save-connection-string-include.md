### 将存储帐户凭据保存到 app.config 文件

接下来，将你的凭据保存到项目的 app.config 文件中。编辑 app.config 文件，使其看起来类似于下面的示例，将 `myaccount` 替换为你的存储帐户名称，并将 `mykey` 替换为你的存储帐户密钥。

	<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
	    <startup>
	        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
	    </startup>
	    <appSettings>
	        <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=StorageAccountKeyEndingIn==;EndpointSuffix=core.chinacloudapi.cn" />
	    </appSettings>
	</configuration>

<!---HONumber=Mooncake_0405_2016-->