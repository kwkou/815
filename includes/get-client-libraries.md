###通过 Composer 安装

1. [安装 Git][install-git]请注意，在 Windows 上，你还必须向 PATH 环境变量添加 Git 可执行文件。 

2. 在您的项目的根目录中创建一个名为 **composer.json** 的文件并向其添加以下代码：

	```
	{
	    "repositories": [
	        {
	            "type": "pear",
	            "url": "http://pear.php.net"
	        }
	    ],
	    "require": {
	        "pear-pear.php.net/mail_mime" : "*",
	        "pear-pear.php.net/http_request2" : "*",
	        "pear-pear.php.net/mail_mimedecode" : "*",
	        "microsoft/windowsazure": "*"
	    }
	}
	```

3. 将 **[composer.phar][composer-phar]** 下载到你的项目根目录中。

4. 打开命令提示符并在项目根目录中执行以下命令

	```
	php composer.phar install
	```

###手动安装

若要手动下载并安装用于 Azure 的 PHP 客户端库，请执行以下步骤：

> [AZURE.NOTE]用于 Azure 的 PHP 客户端库依赖于 **HTTP_Request2**、**Mail_mime** 和 **Mail_mimeDecode** PEAR 包。若要处理这些依赖关系，建议使用 **PEAR 包管理器**安装这些包
 
1. 下载包含 [GitHub][php-sdk-github] 中的库的 .zip 存档。或者，复制现有存储库并将其克隆到您的本地计算机。后一种选择需要一个 GitHub 帐户并要求已在本地安装 Git。
	
2. 将已下载存档的 `WindowsAzure` 目录复制到你的应用程序目录结构中。

有关安装用于 Azure 的 PHP 客户端库的详细信息（包括安装为 PEAR 包的信息），请参阅[下载 Azure SDK for PHP][download-SDK-PHP]。


[php-sdk-github]: http://go.microsoft.com/fwlink/?LinkId=252719
[install-git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[download-SDK-PHP]: /documentation/articles/php-download-sdk/
[composer-phar]: http://getcomposer.org/composer.phar

<!---HONumber=82-->