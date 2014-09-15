### 通过 Composer 安装

1.  [安装 Git][]。

    **说明**

    在 Windows 上，你还需要向你的 PATH 环境变量中添加 Git 可执行文件。

2.  在你的项目的根目录中创建一个名为 **composer.json** 的文件并向其中添加以下代码：

        {
        "require": {
        "microsoft/windowsazure": "*"
            },          
        "repositories": [
                {
        "type":"pear",
        "url":"http://pear.php.net"
                }
            ],
        "minimum-stability":"dev"
        }

3.  将 **[composer.phar][]** 下载到项目根目录中。

4.  打开命令提示符并在项目根目录中执行该文件

        php composer.phar install

### 手动安装

若要手动下载并安装用于 Azure 的 PHP 客户端库，请执行以下步骤：

1.  下载包含 [GitHub][] 中的库的 .zip 存档。或者，复制现有存储库并将其克隆到你的本地计算机。（后一种选择需要一个 GitHub 帐户并要求已在本地安装 Git。）

    **说明**

    用于 Azure 的 PHP 客户端库依赖于 [HTTP\_Request2][]、[Mail\_mime][] 和 [Mail\_mimeDecode][] PEAR 包。若要解决这些依赖关系，建议使用 [PEAR 包管理器][]安装这些包。

2.  将下载的存档的 `WindowsAzure` 目录复制到你的应用程序目录结构中。

有关安装用于 Azure 的 PHP 客户端库的更多信息（包括安装为 PEAR 包的信息），请参阅[下载 Azure SDK for PHP][]。

  [安装 Git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
  [composer.phar]: http://getcomposer.org/composer.phar
  [GitHub]: http://go.microsoft.com/fwlink/?LinkId=252719
  [HTTP\_Request2]: http://pear.php.net/package/HTTP_Request2
  [Mail\_mime]: http://pear.php.net/package/Mail_mime
  [Mail\_mimeDecode]: http://pear.php.net/package/Mail_mimeDecode
  [PEAR 包管理器]: http://pear.php.net/manual/en/installation.php
  [下载 Azure SDK for PHP]: ../php-download-sdk/
