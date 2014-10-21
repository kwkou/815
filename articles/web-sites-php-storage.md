<properties linkid="develop-php-website-with-storage" urlDisplayName="Web w/ Storage" pageTitle="PHP web site with table storage - Azure tutorial" metaKeywords="Azure table storage PHP, Azure PHP website, Azure PHP web site, Azure PHP tutorial, Azure PHP example" description="This tutorial shows you how to create a PHP website and use the Azure Tables storage service in the back-end." metaCanonical="" services="web-sites,storage" documentationCenter="PHP" title="Create a PHP Web Site using Azure Storage" authors="" solutions="" manager="" editor="" />

# 使用 Azure 存储空间创建 PHP 网站

本教程演示如何创建 PHP 网站并在后端使用 Azure 表存储服务。本教程假定你已在计算机上安装 [PHP][PHP] 和 Web 服务器。本教程中的说明适用于任何操作系统，包括 Windows、Mac 和 Linux。完成本指南后，你将拥有一个在 Azure 中运行的并访问表存储服务的 PHP 网站。

你将了解到以下内容：

-   如何安装 Azure 客户端库并将其包含在你的应用程序中。
-   如何使用用于创建表以及用于创建、查询和删除表实体的客户端库。
-   如何创建 Azure 存储帐户并设置应用程序以使用该帐户。
-   如何创建 Azure 网站并使用 Git 部署到该网站

你将在 PHP 中构建一个简单的 Tasklist Web 应用程序。以下是已完成应用程序的屏幕快照：

![Azure PHP 网站][Azure PHP 网站]

[WACOM.INCLUDE [create-account-and-websites-note][create-account-and-websites-note]]

## 安装 Azure 客户端库

若要通过 Composer 安装 Azure 的 PHP 客户端库，请执行下列步骤：

1.  [安装 Git][安装 Git]

    > [WACOM.NOTE]
    > 在 Windows 上，你还需要向你的 PATH 环境变量中添加 Git 可执行文件。

2.  在你的项目的根目录中创建一个名为 **composer.json** 的文件并向其中添加以下代码：

        {
            "require": {
                "microsoft/windowsazure": "*"
            },          
            "repositories": [
                {
                    "type": "pear",
                    "url": "http://pear.php.net"
                }
            ],
            "minimum-stability": "dev"
        }

3.  将 **[composer.phar][composer.phar]** 下载到项目根目录中。

4.  打开命令提示符并在项目根目录中执行该文件

        php composer.phar install

## 开始使用客户端库

在使用库时，必须先执行四个基本步骤，然后才能调用 Azure API。你将创建一个执行这些步骤的初始化脚本。

-   创建一个名为 **init.php** 的文件。

-   首先，包含 autoloader 脚本：

        require_once 'vendor\autoload.php'; 

-   包含你将使用的命名空间。

    若要创建任何 Azure 服务客户端，你将需要使用 **ServicesBuilder** 类：

        use WindowsAzure\Common\ServicesBuilder;

    若要捕获由任何 API 调用生成的异常，你需要 **ServiceException** 类：

        use WindowsAzure\Common\ServiceException;

-   若要实例化服务客户端，你还需要有效的连接字符串。表服务连接字符串的格式为：

    对于访问实时服务：

        DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]

    对于访问模拟器存储：

        UseDevelopmentStorage=true

-   使用 `ServicesBuilder::createTableService` 工厂方法可实例化表服务调用周围的包装。

        $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    `$tableRestProxy` 包含可对 Azure 表进行的每个 REST 调用的方法。

## 创建表

你必须先为数据创建一个容器（即表），然后才能存储数据。

-   创建一个名为 **createtable.php** 的文件。

-   首先，包含你刚刚创建的初始化脚本。你将在访问 Azure 的每个文件中包含此脚本：

        <?php
        require_once "init.php";

-   然后，调用传入表名称中的 *createTable*。与其他 NoSQL 表存储相似，Azure 表不需要架构。

        try {
            $tableRestProxy->createTable('tasks');
        }
        catch(ServiceException $e){
            $code = $e->getCode();
            $error_message = $e->getMessage();
            echo $code.": ".$error_message."<br />";
        }
        ?>

    可在此处找到错误代码和消息：[][]<http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx></a>

## 查询表

Tasklist 应用程序的主页应列出所有现有任务并允许插入新任务。

-   创建一个名为 **index.php** 的文件并插入将构成页标题的以下 HTML 和 PHP 代码：

        <html>
        <head>
            <title>Index</title>
            <style type="text/css">
                body { background-color: #fff; border-top: solid 10px #000;
                    color: #333; font-size: .85em; margin: 20; padding: 20;
                    font-family: "Segoe UI", Verdana, Helvetica, Sans-Serif;
                }
                h1, h2, h3,{ color: #000; margin-bottom: 0; padding-bottom: 0; }
                h1 { font-size: 2em; }
                h2 { font-size: 1.75em; }
                h3 { font-size: 1.2em; }
                table { margin-top: 0.75em; }
                th { font-size: 1.2em; text-align: left; border: none; padding-left: 0; }
                td { padding: 0.25em 2em 0.25em 0em; border: 0 none; }
            </style>
        </head>
        <body>
        <h1>My ToDo List <font color="grey" size="5">(powered by PHP and Azure Tables) </font></h1>
        <?php       
        require_once "init.php";

-   若要在 Azure 表中查询存储在*任务* 表中的**所有实例**，可以调用仅传递表名称的 *queryEntities* 方法。在下面的“更新实体”一节中，你还将了解如何传递查询特定实体的筛选器。

        try {
            $result = $tableRestProxy->queryEntities('tasks');
        }
        catch(ServiceException $e){
            $code = $e->getCode();
            $error_message = $e->getMessage();
            echo $code.": ".$error_message."<br />";
        }

-   对结果集中的实体进行迭代：

        $entities = $result->getEntities();

        for ($i = 0; $i < count($entities); $i++) {

-   在获取`Entity`后，用于读取数据的模型为 `Entity->getPropertyValue('[name]')`:

            if ($i == 0) {
                echo "<table border='1'>
                <tr>
                    <td>Name</td>
                    <td>Category</td>
                    <td>Date</td>
                    <td>Mark Complete?</td>
                    <td>Delete?</td>
                </tr>";
            }
            echo "
                <tr>
                    <td>".$entities[$i]->getPropertyValue('name')."</td>
                    <td>".$entities[$i]->getPropertyValue('category')."</td>
                    <td>".$entities[$i]->getPropertyValue('date')."</td>";
                    if ($entities[$i]->getPropertyValue('complete') == false)
                        echo "<td><a href='markitem.php?complete=true&pk=".$entities[$i]->getPartitionKey()."&rk=".$entities[$i]->getRowKey()."'>Mark Complete</a></td>";
                    else
                        echo "<td><a href='markitem.php?complete=false&pk=".$entities[$i]->getPartitionKey()."&rk=".$entities[$i]->getRowKey()."'>Unmark Complete</a></td>";
                    echo "
                    <td><a href='deleteitem.php?pk=".$entities[$i]->getPartitionKey()."&rk=".$entities[$i]->getRowKey()."'>Delete</a></td>
                </tr>";
        }

        if ($i > 0)
            echo "</table>";
        else
            echo "<h3>No items on list.</h3>";
        ?>

-   最后，你必须插入将数据注入任务插入脚本中的表单并完成 HTML：

            <hr/>
            <form action="additem.php" method="post">
                <table border="1">
                    <tr>
                        <td>Item Name: </td>
                        <td><input name="itemname" type="textbox"/></td>
                    </tr>
                    <tr>
                        <td>Category: </td>
                        <td><input name="category" type="textbox"/></td>
                    </tr>
                    <tr>
                        <td>Date: </td>
                        <td><input name="date" type="textbox"/></td>
                    </tr>
                </table>
                <input type="submit" value="Add item"/>
            </form>
        </body>
        </html>

## 将实体插入表中

你的应用程序现在可以读取表中存储的所有项目。由于最初没有任何项目，因此我们将添加一个将数据写入数据库的函数。

-   创建一个名为 **additem.php** 的文件。

-   将以下内容添加到该文件：

        <?php       
        require_once "init.php";        
        use WindowsAzure\Table\Models\Entity;
        use WindowsAzure\Table\Models\EdmType;      

-   插入实体的第一步是实例化`Entity` 对象并设置该对象的属性：

        $entity = new Entity();
        $entity->setPartitionKey('p1');
        $entity->setRowKey((string) microtime(true));
        $entity->addProperty('name', EdmType::STRING, $_POST['itemname']);
        $entity->addProperty('category', EdmType::STRING, $_POST['category']);
        $entity->addProperty('date', EdmType::STRING, $_POST['date']);
        $entity->addProperty('complete', EdmType::BOOLEAN, false);

-   然后，可以将刚创建的`$entity` 传递到`insertEntity` 方法：

        try{
            $tableRestProxy->insertEntity('tasks', $entity);
        }
        catch(ServiceException $e){
            $code = $e->getCode();
            $error_message = $e->getMessage();
            echo $code.": ".$error_message."<br />";
        }

-   最后，使页面在插入实体后返回主页：

        header('Location: index.php');      
        ?>

## 更新实体

任务列表应用程序能够将项目标记为完成，也能够取消标记项目。主页将传入实体的 *RowKey* 和 *PartitionKey* 以及目标状态（marked==1，unmarked==0）。

-   创建一个名为 **markitem.php** 的文件并添加初始化部分

        <?php       
        require_once "init.php";

-   更新实体的第一步是从表中获取实体：

        $result = $tableRestProxy->queryEntities('tasks', 'PartitionKey eq \''.$_GET['pk'].'\' and RowKey eq \''.$_GET['rk'].'\'');     
        $entities = $result->getEntities();     
        $entity = $entities[0];

    如你所见，传入查询筛选器的格式为`Key eq 'Value'`。有关查询语法的完整说明，请参阅[此处][此处]。

-   然后，你可以更改任何属性：

        $entity->setPropertyValue('complete', ($_GET['complete'] == 'true') ? true : false);

-   而`updateEntity` 方法则会执行更新：

        try{
            $result = $tableRestProxy->updateEntity('tasks', $entity);
        }
        catch(ServiceException $e){
            $code = $e->getCode();
            $error_message = $e->getMessage();
            echo $code.": ".$error_message."<br />";
        }

-   使页面在插入实体后返回主页：

        header('Location: index.php');      
        ?>

## 删除实体

可通过调用一次`deleteItem` 来删除项。传入值为 **PartitionKey** 和 **RowKey**，二者共同构成了实体的主密钥。创建一个名为 **deleteitem.php** 的文件并插入以下代码：

        <?php
        
        require_once "init.php";        
        $tableRestProxy->deleteEntity('tasks', $_GET['pk'], $_GET['rk']);       
        header('Location: index.php');
        
        ?>

## 创建 Azure 存储帐户

若要让应用程序将数据存储在云中，你需要先在 Azure 中创建一个存储帐户，然后将正确的身份验证信息传递到 *Configuration* 类。

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  单击该门户左下角的“+ 新建”图标。

    ![创建新的 Azure 网站][创建新的 Azure 网站]

3.  依次单击**“数据服务”**、**“存储”**、**“快速创建”**。

    ![自定义创建新的网站][自定义创建新的网站]

    输入“URL”的值，并在“区域”下拉菜单中为你的网站选择数据中心。单击对话框底部的“创建存储帐户”按钮。

    ![填写网站详细信息][填写网站详细信息]

    创建存储帐户后，将显示文本“已成功完成存储帐户‘[NAME]’的创建”。

4.  确保已选定“存储”选项卡，然后从列表中选择刚创建的存储帐户。

5.  单击底部应用程序栏上的“管理密钥”。

    ![选择“管理密钥”][选择“管理密钥”]

6.  记下所创建的存储帐户的名称和主密钥。

    ![选择“管理密钥”][1]

7.  打开 **init.php** 并将`[YOUR_STORAGE_ACCOUNT_NAME]` 和 `[YOUR_STORAGE_ACCOUNT_KEY]` 替换为你在上一步中记下的帐户名和密钥。保存文件。

## 创建 Azure 网站并设置 Git 发布

按照以下步骤创建 Azure 网站：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  单击该门户左下角的“+ 新建”图标。

    ![创建新的 Azure 网站][创建新的 Azure 网站]

3.  依次单击“计算”、“网站”和“快速创建”。

    ![自定义创建新的网站][2]

    输入“URL”的值，并在“区域”下拉菜单中为你的网站选择数据中心。单击对话框底部的“创建新网站”按钮。

    ![填写网站详细信息][3]

    创建网站后，你会看到文本“网站‘[SITENAME]’创建已成功完成”。现在，你可以启用 Git 发布。

4.  单击网站列表中显示的网站的名称以打开该网站的“快速启动”仪表板。

    ![打开网站仪表板][打开网站仪表板]

5.  在“快速启动”页的右下角，选择“从源代码管理设置部署”。

    ![设置 Git 发布][设置 Git 发布]

6.  在系统询问“你的源代码在哪里?”时，选择“本地 Git 存储库”，然后单击箭头。

    ![你的源代码在哪里][你的源代码在哪里]

7.  若要启用 Git 发布，你必须提供用户名和密码。记下你创建的用户名和密码。（如果你之前已设置 Git 存储库，则将跳过此步骤。）

    ![创建发布凭据][创建发布凭据]

    设置存储库需要花费几秒钟的时间。

8.  一旦 Git 存储库准备就绪，就会显示有关要用于设置本地存储库并将文件推送到 Azure 的 Git 命令的说明。

    ![为网站创建存储库后返回的 Git 部署说明。][为网站创建存储库后返回的 Git 部署说明。]

    记下说明，因为将在下一节中用到这些说明来发布应用程序。

## 发布应用程序

若要使用 Git 发布应用程序，请执行下列步骤。

1.  打开应用程序的根的下方的 **vendor/microsoft/windowsazure** 文件夹并删除以下文件和文件夹：

    -   .git
    -   .gitattributes
    -   .gitignore

    当 Composer 包管理器下载 Azure 客户端库及其依赖项时，该管理器将通过克隆其所在的 GitHub 存储库来执行此操作。在下一步中，将通过在应用程序的根文件夹的外部创建存储库来通过 Git 部署应用程序。Git 将忽略客户端库所在的子存储库，除非已删除存储库特定的文件。

2.  打开 GitBash（或终端，如果 Git 在你的`PATH`中），将目录更改为应用程序的根目录，并运行以下命令（**注意：**这些步骤与在“创建 Azure 网站并设置 Git 发布”一节的结尾标明的步骤相同）：

        git init
        git add .
        git commit -m "initial commit"
        git remote add azure [URL for remote repository]
        git push azure master

    系统将提示你输入之前创建的密码。

3.  浏览到 **http://[你的网站域]/createtable.php** 以创建应用程序的表。
4.  浏览到 **http://[你的网站域]/index.php** 以开始使用应用程序。

发布应用程序之后，你可以开始对其进行更改并使用 Git 发布所做的更改。

## 发布对应用程序所做的更改

若要发布对应用程序所做的更改，请执行下列步骤：

1.  本地对应用程序进行更改。
2.  打开 GitBash（或终端，如果 Git 在你的`PATH`中），将目录更改为应用程序的根目录，并运行以下命令：

        git add .
        git commit -m "comment describing changes"
        git push azure master

    系统将提示你输入之前创建的密码。

3.  浏览到 **http://[你的网站域]/index.php** 以查看你的更改。

  [PHP]: http://www.php.net/manual/en/install.php
  [Azure PHP 网站]: ./media/web-sites-php-storage/ws-storage-app.png
  [create-account-and-websites-note]: ../includes/create-account-and-websites-note.md
  [安装 Git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
  [composer.phar]: http://getcomposer.org/composer.phar
  []: http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
  [此处]: http://msdn.microsoft.com/zh-cn/library/azure/dd894031.aspx
  [Azure 管理门户]: https://manage.windowsazure.cn
  [创建新的 Azure 网站]: ./media/web-sites-php-storage/new_website.jpg
  [自定义创建新的网站]: ./media/web-sites-php-storage/storage-quick-create.png
  [填写网站详细信息]: ./media/web-sites-php-storage/storage-quick-create-details.png
  [选择“管理密钥”]: ./media/web-sites-php-storage/storage-manage-keys.png
  [1]: ./media/web-sites-php-storage/storage-access-keys.png
  [2]: ./media/web-sites-php-storage/website-quick-create.png
  [3]: ./media/web-sites-php-storage/website-quick-create-details.png
  [打开网站仪表板]: ./media/web-sites-php-storage/go_to_dashboard.png
  [设置 Git 发布]: ./media/web-sites-php-storage/setup_git_publishing.png
  [你的源代码在哪里]: ./media/web-sites-php-storage/where_is_code.png
  [创建发布凭据]: ./media/web-sites-php-storage/git-deployment-credentials.png
  [为网站创建存储库后返回的 Git 部署说明。]: ./media/web-sites-php-storage/git-instructions.png
  [http://[你的]: http://[your
