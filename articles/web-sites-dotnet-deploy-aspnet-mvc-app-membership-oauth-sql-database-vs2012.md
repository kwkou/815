<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title=" OAuth" authors="riande" solutions="" manager="wpickett" editor="mollybos" />

# 使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 应用部署到 Azure 网站

***由 [Rick Anderson][Rick Anderson] 和 Tom Dykstra 撰写。上次更新时间：2013 年 10 月 15 日。***

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/develop/net/tutorials/web-site-with-sql-database/" title="Visual Studio 2013">Visual Studio 2013</a><a href="/zh-cn/develop/net/tutorials/web-site-with-sql-database-vs2012/" title="Visual Studio 2012" class="current">Visual Studio 2012</a></div>

<div class="dev-callout"><strong>说明</strong><p>已提供<a href="/zh-cn/develop/net/tutorials/web-site-with-sql-database/">本教程的较新版本</a>。如果您想要使用 Visual Studio 2012，则仍可以执行此版本，但较新版本更加易于执行。</p></div>

本教程演示如何构建安全的 ASP.NET MVC 4 Web 应用程序，以便用户能够使用 Facebook、Yahoo 和 Google 凭据进行登录。您还会将应用程序部署到 Azure。

您可以免费注册一个 Azure 帐户，而且，如果您还没有 Visual Studio 2012，则此 SDK 会自动安装 Visual Studio 2012 for Web Express。您可以免费开始针对 Azure 进行开发。

本教程假定您之前未使用过 Azure。完成本教程之后，您将能够在云中启动并运行安全的数据驱动的 Web 应用程序，以及使用云数据库。

您将了解到以下内容：

-   如何通过安装 Azure SDK 来让您的计算机可以进行 Azure 开发。
-   如何创建安全的 ASP.NET MVC 4 项目并将其发布到 Azure 网站。
-   如何使用 OAuth 和 ASP.NET 成员资格数据库保护您的应用程序。
-   如何将成员资格数据库部署到 Azure。
-   如何使用 SQL Database 在 Azure 中存储数据。
-   如何使用 Visual Studio 更新和管理成员资格数据库。

您将生成一个简单的联系人列表 Web 应用程序，该应用程序基于 ASP.NET MVC 4 构建并使用 ADO.NET Entity Framework 进行数据库访问。下图演示了完整应用程序的登录页面：

![登录页面][登录页面]

[WACOM.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

在本教程中：

-   [设置开发环境][设置开发环境]
-   [设置 Azure 环境][设置 Azure 环境]
-   [创建 ASP.NET MVC 4 应用程序][创建 ASP.NET MVC 4 应用程序]
-   [将应用程序部署到 Azure][将应用程序部署到 Azure]
-   [向应用程序添加数据库][向应用程序添加数据库]
-   [添加 OAuth 提供程序][添加 OAuth 提供程序]
-   [向成员资格数据库中添加角色][向成员资格数据库中添加角色]
-   [创建数据部署脚本][创建数据部署脚本]
-   [将应用部署到 Azure][将应用部署到 Azure]
-   [更新成员资格数据库][更新成员资格数据库]
-   [后续步骤][后续步骤]

## <a name="bkmk_setupdevenv"></a>设置开发环境

首先，通过安装适用于 .NET Framework 的 Azure SDK 来设置开发环境。

1.  若要安装 Azure SDK for .NET，请单击以下链接。如果您尚未安装 Visual Studio 2012，可单击该链接安装它。本教程需要 Visual Studio 2012。
    [Azure SDK for Visual Studio 2012][Azure SDK for Visual Studio 2012]
2.  当提示您运行或保存可执行安装文件时，单击“运行”。
3.  在“Web 平台安装程序”窗口中，单击“安装”，然后进行安装。

![Web 平台安装程序 – Azure SDK for .NET][Web 平台安装程序 – Azure SDK for .NET]

安装完成后，您便做好了开发准备工作。

## <a name="bkmk_setupwindowsazure"></a>设置 Azure 环境

接下来，通过创建 Azure 网站和 SQL Database 来设置 Azure 环境。

### 在 Azure 中创建网站和 SQL 数据库

您的 Azure 网站将在共享宿主环境中运行，这意味着它将在与其他 Azure 客户端共享的虚拟机 (VM) 上运行。共享宿主环境是一种在云中开始工作的低成本方式。稍后，如果您的 Web 流量增加，则应用程序可进行扩展，通过在专用 VM 上运行来满足需要。如果您需要一个更复杂的体系结构，则可迁移到 Azure 云服务。云服务在您可根据自己的需求进行配置的专用 VM 上运行。

Azure SQL Database 是根据 SQL Server 技术构建的基于云的关系数据库服务。可以与 SQL Server 一起使用的工具和应用程序也可用于 SQL Database。

1.  在 [Azure 管理门户][Azure 管理门户]中，单击左侧选项卡中的“网站”，然后单击“新建”。

![管理门户中的“新建”按钮][管理门户中的“新建”按钮]

1.  单击“自定义创建”。

    ![管理门户中的“与数据库一起创建”链接][管理门户中的“与数据库一起创建”链接]

“新网站 – 自定义创建”向导将打开。

1.  在该向导的“新建网站”步骤中，在“URL”框中输入将用作您的应用程序的唯一 URL 的字符串。完整的 URL 将包含您在此处输入的内容和您在文本框旁边看到的后缀。图中显示“contactmgr2”，但可能已有人使用了该 URL，因此必须另外选择一个。

    ![管理门户中的“与数据库一起创建”链接][1]

2.  在“数据库”下拉列表中，选择“创建新的 SQL 数据库”。

3.  在“区域”下拉列表中，选择您为网站所选的同一区域。此设置指定将运行您的 VM 的数据中心。在“数据库连接字符串名称”中，输入 *connectionString1*。

    ![“新建网站 - 与数据库一起创建”向导的“创建新网站”步骤][1]

4.  单击对话框底部的向右箭头。该向导将前进到“数据库设置”步骤。

5.  在“名称”框中，输入 *ContactDB*。

6.  在“服务器”框中，选择“新建 SQL Database 服务器”。或者，如果您之前创建了 SQL Server 数据库，则可从下拉列表控件中选择 SQL Server。

7.  输入管理员“登录名”和“密码”。如果您选择了“新建 SQL Database 服务器”，则在此处不要输入现有名称和密码。您应输入新的名称和密码，您现在定义的名称和密码将在您以后访问数据库时使用。如果您选择了之前创建的 SQL Server，系统将提示您输入之前创建的 SQL Server 帐户名称的密码。在本教程中，我们不选中“高级”框。您可以使用“高级”框设置数据库大小（默认为 1 GB，但您可以将其增加到 150 GB）和排序规则。

8.  单击对话框底部的复选标记以指示您已完成操作。

    ![“新建网站 - 与数据库一起创建”向导的“数据库设置”步骤][“新建网站 - 与数据库一起创建”向导的“数据库设置”步骤]

    下图指示使用的是现有 SQL Server 和登录名。
    ![“新建网站 – 与数据库一起创建”向导的“数据库设置”步骤][“新建网站 – 与数据库一起创建”向导的“数据库设置”步骤]

    管理门户返回到“网站”页面，“状态”列显示正在创建网站。稍后（通常不到一分钟），“状态”列会显示已成功创建网站。在左侧的导航栏中，您的帐户中拥有的网站的数量将会显示在“网站”图标旁边，而数据库的数量将会显示在“SQL Database”图标旁边。

## <a name="bkmk_createmvc4app"></a>创建 ASP.NET MVC 4 应用程序

您已经创建了一个 Azure 网站，但其中还没有内容。下一步将创建要发布到 Azure 的 Visual Studio Web 应用程序项目。

### 创建项目

1.  启动 Visual Studio 2012。
2.  在“文件”菜单中，单击“新建项目”。
3.  在“新建项目”对话框中，展开“Visual C#”并在“已安装的模板”下选择“Web”，然后选择“ASP.NET MVC 4 Web 应用程序。保持默认值 **.NET Framework 4.5**。将该应用程序命名为 **ContactManager**，然后单击“确定”。

    ![“新建项目”对话框][“新建项目”对话框]

4.  在“新建 ASP.NET MVC 4 项目”对话框中，选择“Internet 应用程序”模板。保留默认值 Razor“视图引擎”，然后单击“确定”。

    ![“新建 ASP.NET MVC 4 项目”对话框][“新建 ASP.NET MVC 4 项目”对话框]

### 设置页眉和页脚

1.  在“解决方案资源管理器”中，展开 Views\\Shared 文件夹，然后打开 \*\_Layout.cshtml\* 文件。

    ![解决方案资源管理器中的 \_Layout.cshtml][解决方案资源管理器中的 \_Layout.cshtml]

2.  用“联系人管理器”替换每一处“我的 ASP.NET MVC 应用程序”。
3.  用“CM 演示”替换“在此处显示您的徽标”。

### 在本地运行应用程序

1.  按 Ctrl+F5 运行应用程序。应用程序主页将显示在默认浏览器中。
    ![“待办事项列表”主页][“待办事项列表”主页]

这就是您创建将要部署到 Azure 的应用程序目前所需的全部操作。稍后您将添加数据库功能。

## <a name="bkmk_deploytowindowsazure1"></a>将应用程序部署到 Azure

1.  在浏览器中，打开[Azure 管理门户][2]。

2.  在“网站”选项卡中，单击先前创建的网站的名称。

    ![管理门户的“网站”选项卡中的“联系人管理器”应用程序][管理门户的“网站”选项卡中的“联系人管理器”应用程序]

3.  在窗口右侧，单击“下载发布配置文件”。

    ![“快速启动”选项卡和“下载发布配置文件”按钮][“快速启动”选项卡和“下载发布配置文件”按钮]

    此步骤将下载一个文件，其中包含将应用程序部署到您的网站所需的全部设置。您将此文件导入到 Visual Studio 中，这样您就不必手动输入此信息。

4.  将 .*publishsettings* 文件保存到您可以从 Visual Studio 访问的文件夹中。

    ![保存 .publishsettings 文件][保存 .publishsettings 文件]

    [WACOM.INCLUDE [publishsettingsfilewarningchunk](../includes/publishsettingsfilewarningchunk.md)]

5.  在 Visual Studio 中，在“解决方案资源管理器”中右键单击该项目，从上下文菜单中选择“发布”。

    ![项目上下文菜单中的“发布”][项目上下文菜单中的“发布”]

    “发布 Web”向导将打开。

6.  在“发布 Web”向导的“配置文件”选项卡中，单击“导入”。

    ![导入发布设置][导入发布设置]

    “导入发布配置文件”对话框随即出现。

7.  如果您之前未在 Visual Studio 中添加 Azure 订阅，请执行下列步骤。在这些步骤中，您将添加订阅，以便“从 Azure 网站导入”下的下拉列表中包含您的网站。

    a. 在“导入发布配置文件”对话框中，单击“添加 Azure 订阅”。

    ![添加 Windows Azure 订阅][添加 Windows Azure 订阅]

    b. 在“导入 Azure 订阅”对话框中，单击“下载订阅文件”。

    ![下载订阅][下载订阅]

    c. 在浏览器窗口中，保存 *.publishsettings* 文件。

    ![下载发布文件][下载发布文件]

    > [WACOM.NOTE]
    > .publishsettings 文件中包含您的凭据（未编码），这些凭据用来管理您的 Azure 订阅和服务。确保此文件安全的最佳做法是，将其暂时存储在您的源目录的外部（例如存储在 Libraries\\Documents 文件夹中），然后在完成导入后将其删除。获得了 publishsettings 文件访问权的恶意用户可以编辑、创建和删除您的 Azure 服务。

    d. 在“导入 Azure 订阅”对话框中，单击“浏览”并导航到 *.publishsettings* 文件。

    ![下载订阅][下载订阅]

    e. 单击“导入”。

    ![导入][导入]

8.  在“导入发布配置文件”对话框中，选择“从 Azure 网站导入”，再从下拉列表中选择您的网站，然后单击“确定”。

    ![导入发布配置文件][导入发布配置文件]

    您创建的应用程序现在在云中运行。下次部署该应用程序时，仅会部署已更改（或新的）文件。

    ![在 Azure 中运行的“待办事项列表”主页][在 Azure 中运行的“待办事项列表”主页]

## <a name="bkmk_addadatabase"></a>向应用程序添加数据库

接下来，您将更新 MVC 应用程序以添加显示和更新联系人以及在数据库中存储数据的功能。应用程序将使用 Entity Framework 创建数据库并读取和更新数据库中的数据。

### 为联系人添加数据模型类

首先，使用代码创建一个简单的数据模型。

1.  在“解决方案资源管理器”中，右键单击 Models 文件夹，单击“添加”，然后单击“类”。

![Models 文件夹上下文菜单中的“添加类”][Models 文件夹上下文菜单中的“添加类”]

1.  在“添加新项”对话框中，将新的类文件命名为 *Contact.cs*，然后单击“添加”。

![“添加新项”对话框][“添加新项”对话框]

1.  将 Contacts.cs 文件的内容替换为以下代码。

        using System.ComponentModel.DataAnnotations;
        using System.Globalization;
        namespace ContactManager.Models
        {
            public class Contact
            {
                public int ContactId { get; set; }
                public string Name { get; set; }
                public string Address { get; set; }
                public string City { get; set; }
                public string State { get; set; }
                public string Zip { get; set; }
                [DataType(DataType.EmailAddress)]
                public string Email { get; set; }
            }
        }

**Contacts** 类定义您将为每个联系人存储的数据以及数据库需要的主键 *ContactID*。

### 创建使应用程序用户可以使用联系人的网页

ASP.NET MVC 基架功能可以自动生成用于执行创建、读取、更新和删除 (CRUD) 操作的代码。

## <a name="bkmk_addcontroller"></a>为数据添加控制器和视图

1.  生成项目 **(Ctrl+Shift+B)**。（在使用基架机制前必须生成项目。）
2.  在“解决方案资源管理器”中，右键单击“Controllers”文件夹，然后单击“添加”，再单击“控制器”。

    ![Controllers 文件夹上下文菜单中的“添加控制器”][Controllers 文件夹上下文菜单中的“添加控制器”]

3.  在“添加控制器”对话框中，输入“HomeController”作为控制器名称。
4.  将“基架选项”模板设置为“包含读/写操作和视图的 MVC 控制器(使用 Entity Framework)”。
5.  选择“联系人”作为模型类和“\<新建数据上下文...\>”作为您的数据上下文类。

    ![“添加控制器”对话框][“添加控制器”对话框]

6.  在“新建数据上下文”对话框中，接受默认值 *ContactManager.Models.ContactManagerContext*。
    ![“添加控制器”对话框][3]

7.  单击“确定”，然后单击“添加控制器”对话框中的“添加”。
8.  在“添加控制器”覆盖对话框中，确保选中所有选项，然后单击“确定”。

    ![“添加控制器”消息框][“添加控制器”消息框]

Visual Studio 将创建一个控制器方法并为 **Contact** 对象的 CRUD 数据库操作创建视图。

## 启用迁移、创建数据库、添加示例数据和数据初始值设定项

接下来的任务是启用[代码优先迁移][代码优先迁移] 功能以便基于您创建的数据模型创建数据库。

1.  在“工具”菜单中，依次选择“库程序包管理器”和“程序包管理器控制台”。

    ![“工具”菜单中的“程序包管理器控制台”][“工具”菜单中的“程序包管理器控制台”]

2.  在“程序包管理器控制台”窗口中，输入以下命令：

        enable-migrations -ContextTypeName ContactManagerContext

    ![enable-migrations][enable-migrations]
    您必须指定上下文类型名称 (**ContactManagerContext**)，因为项目包含两个 [DbContext][DbContext] 派生类，即我们刚添加的 **ContactManagerContext** 和用于成员资格数据库的 **UsersContext**。**ContactManagerContext** 类由 Visual Studio 支架向导添加。

    **enable-migrations** 命令将创建一个 *Migrations* 文件夹，并在该文件夹中放入一个可编辑以配置 Migrations 的 *Configuration.cs* 文件。

3.  在“程序包管理器控制台”窗口中，输入以下命令：

        add-migration Initial

    **add-migration Initial** 命令将在创建数据库的 *Migrations* 文件夹中生成一个名为 **\<date\_stamp\>Initial** 的文件。第一个参数 (**Initial**) 是任意参数并将用于创建文件名称。您可以在“解决方案资源管理器”中查看新的类文件。

    在 **Initial** 类中，**Up** 方法用于创建 Contacts 表，而 **Down** 方法（在您想要返回到以前的状态时使用）用于删除该表。

4.  打开 *Migrations\\Configuration.cs* 文件。
5.  添加以下命名空间。

         using ContactManager.Models;

6.  将 *Seed* 方法替换为以下代码：

        protected override void Seed(ContactManager.Models.ContactManagerContext context)
        {
            context.Contacts.AddOrUpdate(p => p.Name,
               new Contact
               {
                   Name = "Debra Garcia",
                   Address = "1234 Main St",
                   City = "Redmond",
                   State = "WA",
                   Zip = "10999",
                   Email = "debra@example.com",
               },
                new Contact
                {
                    Name = "Thorsten Weinrich",
                    Address = "5678 1st Ave W",
                    City = "Redmond",
                    State = "WA",
                    Zip = "10999",
                    Email = "thorsten@example.com",
                },
                new Contact
                {
                    Name = "Yuhong Li",
                    Address = "9012 State st",
                    City = "Redmond",
                    State = "WA",
                    Zip = "10999",
                    Email = "yuhong@example.com",
                },
                new Contact
                {
                    Name = "Jon Orton",
                    Address = "3456 Maple St",
                    City = "Redmond",
                    State = "WA",
                    Zip = "10999",
                    Email = "jon@example.com",
                },
                new Contact
                {
                    Name = "Diliana Alexieva-Bosseva",
                    Address = "7890 2nd Ave E",
                    City = "Redmond",
                    State = "WA",
                    Zip = "10999",
                    Email = "diliana@example.com",
                }
                );
        }

    上面这段代码将用联系信息初始化数据库。有关对数据库进行种子设定的更多信息，请参见[对 Entity Framework (EF) 数据库进行种子设定和调试][对 Entity Framework (EF) 数据库进行种子设定和调试]。

7.  在“程序包管理器控制台”中输入以下命令：

        update-database

    ![“程序包管理器控制台”命令][“程序包管理器控制台”命令]

    **update-database** 用于运行将创建数据库的初始迁移。默认情况下，将以 SQL Server Express LocalDB 数据库的形式创建数据库。（除非您已安装 SQL Server Express，在这种情况下，将使用 SQL Server Express 实例创建数据库。）

8.  按 Ctrl+F5 运行应用程序。

应用程序将显示种子数据并提供编辑、详细信息和删除链接。

![数据的 MVC 视图][数据的 MVC 视图]

## <a name="addOauth"></a><span class="short-header">OAuth</span>添加 OAuth 提供程序

[OAuth][OAuth] 是一种开放协议，允许以一种简单而标准的方法从 Web、移动和桌面应用程序进行安全授权。ASP.NET MVC Internet 模板使用 OAuth 公开将 Facebook、Twitter、Google、Yahoo 和 Microsoft 作为身份验证提供程序。虽然本教程仅使用 Facebook、Google 和 Yahoo 作为身份验证提供程序，但您可轻松修改代码以使用其中任何提供程序。实施其他提供程序的步骤与您将在本教程中看到的步骤非常类似。

除了身份验证外，本教程还将使用角色实施授权。只有您添加到 canEdit 角色中的用户将能创建、编辑或删除联系人。

## 使用外部提供程序进行注册

若要使用某些外部提供程序的凭据对用户进行身份验证，您必须使用相关提供程序注册您的网站并获取密钥和连接密钥。Google 和 Yahoo 不需要您注册和获取密钥。

本教程不提供使用这些提供程序进行注册所必须执行的全部步骤。这些步骤通常比较简单。若要成功注册您的网站，请按照这些网站上提供的说明操作。若要开始注册您的网站，请查看下列门户的开发人员网站：

-   [Facebook][Facebook]
-   [Microsoft][Microsoft]
-   [Twitter][Twitter]

导航到 [https://developers.facebook.com/apps][https://developers.facebook.com/apps] 页面并进行登录（如有必要）。单击“以开发人员身份进行注册”按钮并完成注册过程。在完成注册后，请单击“新建应用程序”。输入应用程序的名称。您无需输入应用程序命名空间。

![新建 FB 应用程序][新建 FB 应用程序]

在“应用程序域”中输入 localhost 并在“网站 URL”中输入 http://localhost/。对“沙盒模式”单击“启用”，然后单击“保存更改”。

您将需要“应用程序 ID”和“应用程序密码”以在此应用程序中实现 OAuth。
![新建 FB 应用程序][4]

## 创建测试用户

在“设置”下的左窗格中，单击“开发人员角色”。单击“测试用户”行（而不是“测试人员”行）中的“创建”链接。

![FB 测试人员][FB 测试人员]

单击“修改”链接以获取测试用户电子邮件（将用于登录到应用程序）。单击“查看更多”链接，然后单击“编辑”以设置测试用户密码。

## 从提供程序中添加应用程序 ID 和密码

打开 *App\_Start\\AuthConfig.cs* 文件。删除 *RegisterFacebookClient* 方法中的注释字符并添加应用程序 ID 和应用程序密码。使用您获取的值，下面显示的值将不起作用。删除 *OAuthWebSecurity.RegisterGoogleClient* 调用中的注释字符，并添加如下所示的 *OAuthWebSecurity.RegisterYahooClient*。Google 和 Yahoo 提供程序不需要您注册和获取密钥。
警告：确保您的应用程序 ID 和密码安全可靠。拥有您的应用程序 ID 和密码的恶意用户可以假扮为您的应用程序。

     public static void RegisterAuth()
        {
            OAuthWebSecurity.RegisterFacebookClient(
                appId: "enter numeric key here",
                appSecret: "enter numeric secret here");

            OAuthWebSecurity.RegisterGoogleClient();
            OAuthWebSecurity.RegisterYahooClient();
        }

1.  运行应用程序并单击“登录”链接。
2.  单击“Facebook”按钮。
3.  输入您的 Facebook 凭据或某一测试用户凭据。
4.  单击“确定”以允许应用程序访问您的 Facebook 资源。
5.  您将重定向到“注册”页。如果您使用测试帐户登录，则可将“用户名”更改为更短的内容，例如“Bill FB 测试”。单击“注册”按钮，这会将用户名和电子邮件别名保存到成员资格数据库。
6.  注册其他用户。目前登录系统中的一个漏洞阻止您以其他用户身份使用同一提供程序进行注销和登录（也就是说，您无法注销您的 Facebook 帐户且无法使用其他 Facebook 帐户重新登录）。为解决此问题，请使用其他浏览器导航到该站点并注册其他用户。将向管理员角色中添加一名用户，并且该用户将拥有应用程序的编辑访问权限，而其他用户将只拥有站点上非编辑方法的访问权限。匿名用户将只能访问主页。
7.  使用其他提供程序注册其他用户。
8.  **可选**：您还可以创建与某个提供程序没有关联的本地帐户。在本教程后面，我们将删除创建本地帐户的功能。要创建本地帐户，请单击“注销”链接（如果您已登录），然后单击“注册链接”。出于管理目的，您可能希望创建一个与任何外部身份验证提供程序都没有关联的本地帐户。

## <a name="mbrDB"></a><span class="short-header">成员资格数据库</span>向成员资格数据库添加角色

在本节中，您会将 *canEdit* 角色添加到成员资格数据库。只有具有 canEdit 角色的用户才能编辑数据。最佳做法是按照角色可以执行的操作命名这些角色，因此 *canEdit* 优于名为 *admin* 的角色。在您的应用程序升级后，您可以添加新角色，例如 *canDeleteMembers*，而不是 *superAdmin*。

1.  在“视图”菜单中，单击“服务器资源管理器”。

2.  在“服务器资源管理器”中，展开 **DefaultConnection**，然后展开“表”。

3.  右键单击 **UserProfile**，然后单击“显示表数据”。

    ![显示表数据][显示表数据]

4.  记录将拥有 canEdit 角色的用户的 **UserId**。在下图中，**UserId** 为 2 的用户 *ricka* 将拥有站点的 canEdit 角色。

    ![用户 ID][用户 ID]

5.  右键单击“webpages\_Roles”，然后单击“显示表数据”。
6.  在 **RoleName** 单元格中输入 **canEdit**。如果这是您第一次添加角色，**RoleId** 将为 1。记录 RoleID。确保不存在尾随空格字符，角色表中的“canEdit”与控制器代码中的“canEdit”将不匹配。

    ![roleID][roleID]

7.  右键单击“webpages UsersInRoles”，然后单击“显示表数据”。输入希望向其授予 *canEdit* 访问权限的用户的 **UserId** 以及 **RoleId**。

    ![用户角色 ID 表][用户角色 ID 表]

**Webpages\_OAuthMembership** 表包含 OAuth 提供程序、提供程序 UserID 和每个已注册的 OAuth 用户的 UserID。<!-- Don't replace "-" with "_" or it won't validate -->**webpages-Membership** 表包含 ASP.NET 成员资格表。您可以使用注册链接将用户添加到此表。最好添加与 Facebook 或其他第三方授权提供程序没有关联的具有 *canEdit* 角色的用户，这样您就可以始终拥有 *canEdit* 访问权限，即使在第三方授权提供程序不可用时亦如此。在本教程后面，我们将禁用 ASP.NET 成员身份注册。

## 通过 Authorize 属性保护应用程序

在本节中，我们将应用 [Authorize][Authorize] 特性限制对操作方法的访问。匿名用户将只能查看主页。已注册用户将能查看联系人详细信息、关于页面和联系人页面。只有具有 *canEdit* 角色的用户才能访问可更改数据的操作方法。

1.  向应用程序中添加 [Authorize][Authorize] 筛选器和 [RequireHttps][RequireHttps] 筛选器。替代方法是向每个控制器中添加 [Authorize][Authorize] 和 [RequireHttps][RequireHttps] 属性，但最安全的做法是将这些属性应用于整个应用程序。通过全局添加这两个属性，您添加的每个新控制器和操作方法都将自动受到保护，您将无需记住应用它们。有关更多信息，请参见[保护 ASP.NET MVC 4 应用程序和新 AllowAnonymous 属性][保护 ASP.NET MVC 4 应用程序和新 AllowAnonymous 属性]。打开 *App\_Start\\FilterConfig.cs* 文件并将 *RegisterGlobalFilters* 方法替换为以下内容。

        public static void
        RegisterGlobalFilters(GlobalFilterCollection filters)
        {
            filters.Add(new HandleErrorAttribute());
            filters.Add(new System.Web.Mvc.AuthorizeAttribute());
            filters.Add(new RequireHttpsAttribute());
        }

2.  向 **Index** 方法中添加 [AllowAnonymous][保护 ASP.NET MVC 4 应用程序和新 AllowAnonymous 属性] 属性。[AllowAnonymous][保护 ASP.NET MVC 4 应用程序和新 AllowAnonymous 属性] 属性使您能够将您要选择取消授权的方法加入白名单。
3.  向可更改数据（创建、编辑、删除）的 Get 和 Post 方法中添加 [Authorize(Roles = "canEdit")]。
4.  添加 *About* 和 *Contact* 方法。下面显示了一部分已完成代码。

        public class HomeController : Controller
        {
            private ContactManagerContext db = new ContactManagerContext();
            [AllowAnonymous]
            public ActionResult Index()
            {
                return View(db.Contacts.ToList());
            }

            public ActionResult About()
            {
                return View();
            }

            public ActionResult Contact()
            {
                return View();
            }

            [Authorize(Roles = "canEdit")]
            public ActionResult Create()
            {
                return View();
            }
            // Methods moved and omitted for clarity.
        }

5.  取消 ASP.NET 成员身份注册。项目中当前的 ASP.NET 成员身份注册不提供密码重置支持，并且不会验证用户是否正在注册（例如，使用 [CAPTCHA][CAPTCHA]）。在使用某个第三方提供程序验证用户身份后，该用户即可进行注册。在 AccountController 中，从 GET 和 POST *Register* 方法中删除 *[AllowAnonymous]*。这将防止机器人和匿名用户进行注册。
6.  在 *Views\\Shared\_LoginPartial.cshtml* 中，删除“注册”操作链接。
7.  启用 SSL。在“解决方案资源管理器”中，单击 **ContactManager** 项目，然后单击 F4 以显示属性对话框。将“已启用 SSL”更改为 true。复制 **SSL URL**。

    ![启用 SSL][启用 SSL]

8.  在“解决方案资源管理器”中，右键单击“Contact Manager”项目，然后单击“属性”。
9.  在左侧选项卡中，单击 **Web**。
10. 将“项目 URL”更改为使用“SSL URL”。
11. 单击“创建虚拟目录”。

    ![启用 SSL][5]

12. 按 Ctrl+F5 运行应用程序。浏览器将显示一个证书警告。对于我们的应用程序，您可以安全地单击链接“继续浏览此网站”。确认只有具有 *canEdit* 角色的用户可以更改数据。确认匿名用户只能查看主页。

    ![认证警告][认证警告]

    ![认证警告][6]

Azure 网站包含有效的安全证书，因此在部署到 Azure 时，将不会显示此警告。

## <a name="ppd"></a><span class="short-header">准备数据库</span>创建数据部署脚本

</p>
成员资格数据库并不受 Entity Framework Code First 的管理，因此您无法使用迁移部署该数据库。我们将使用 [dbDacFx][dbDacFx] 提供程序部署数据库架构，并且我们将配置发布配置文件以运行会将初始成员资格数据插入成员表中的脚本。

本教程将使用 SQL Server Management Studio (SSMS) 创建数据部署脚本。

从 [Microsoft SQL Server 2012 Express 下载中心][Microsoft SQL Server 2012 Express 下载中心]安装 SSMS：

-   [ENU\x64\SQLManagementStudio\_x64\_ENU.exe][ENU\x64\SQLManagementStudio\_x64\_ENU.exe]（适用于 64 位系统）。
-   [ENU\x86\SQLManagementStudio\_x86\_ENU.exe][ENU\x86\SQLManagementStudio\_x86\_ENU.exe]（适用于 32 位系统）。

如果您为系统选择了错误的安装程序，安装将失败，您可以尝试另一个安装程序。

（请注意，这是一个 600 MB 的下载。可能需要较长时间进行安装且可能需要重新启动您的计算机。）

在 SQL Server 安装中心的第一页上，单击“全新 SQL Server 独立安装或向现有安装添加功能”，然后按照说明操作，接受默认选项。下图显示安装 SSMS 的步骤。

![SQL 安装][SQL 安装]

### 创建开发数据库脚本

1.  运行 SSMS。
2.  在“连接到服务器”对话框中，输入 *(localdb)\\v11.0* 作为服务器名称，保持“身份验证”设置为“Windows 身份验证”，然后单击“连接”。如果您已安装 SQL Express，请输入 **.\\SQLEXPRESS**。

    ![“连接到服务器”对话框][“连接到服务器”对话框]

3.  在“对象资源管理器”窗口中，展开“数据库”，右键单击“aspnet-ContactManager”，单击“任务”，然后单击“生成脚本”。

    ![生成脚本][生成脚本]

4.  在“生成和发布脚本”对话框中，单击“设置脚本编写选项”。
    您可以跳过“选择对象”步骤，因为默认值为对整个数据库和所有数据库对象进行脚本编写，而这正是您需要的。

5.  单击“高级”。

    ![设置脚本编写选项][设置脚本编写选项]

6.  在“高级脚本编写选项”对话框中，向下滚动到“要编写脚本的数据的类型”，然后单击下拉列表中的“仅数据”选项。（有关下一步，请参见下图。）

7.  将“编写 USE DATABASE 脚本”更改为“False”。USE 语句对于 Azure SQL Database 无效，且在测试环境中部署到 SQL Server Express 时不需要这些语句。

    ![设置脚本编写选项][7]

8.  单击“确定”。
9.  在“生成和发布脚本”对话框中，“文件名”框指定将创建脚本的位置。更改您的解决方案文件夹（包含您的 *Contacts.sln* 文件的文件夹）的路径并将文件名更改为 *aspnet-data-membership.sql*。
10. 单击“下一步”转到“摘要”选项卡，然后再次单击“下一步”以创建脚本。

    ![保存或发布][保存或发布]

11. 单击“完成”。

## <a name="bkmk_deploytowindowsazure11"></a>将应用部署到 Azure

1.  打开应用程序根 *Web.config* 文件。查找 *DefaultConnection* 标记，然后将其复制并粘贴到 *DefaultConnection* 标记行下。重命名已复制元素 *DefaultConnectionDeploy*。您将需要此连接字符串以在成员资格数据库中部署用户数据。
    ![3 个连接字符串][3 个连接字符串]

2.  构建应用程序。
3.  在 Visual Studio 中，在“解决方案资源管理器”中右键单击该项目，从上下文菜单中选择“发布”。

    ![项目上下文菜单中的“发布”][8]

“发布 Web”向导将打开。

1.  单击“设置”选项卡。单击“v”图标为 **ContactManagerContext** 和 **DefaultConnectionDeploy** 选择“远程连接字符串”。列出的三个数据库全部将使用相同的连接字符串。**ContactManagerContext** 数据库用于存储联系人，**DefaultConnectionDeploy** 仅用于将用户帐户数据部署到成员资格数据库，而 **UsersContext** 数据库是成员资格数据库。

    ![设置][设置]

2.  在 **ContactManagerContext** 下，选中“执行 Code First 迁移”。

    ![设置][9]

3.  在 **DefaultConnectionDeploy** 下，选中“更新数据库”，然后单击“配置数据库更新”链接。
4.  单击“添加 SQL 脚本”链接，然后导航到 *aspnet-data-membership.sql* 文件。您只需执行此操作一次。下次部署取时可取消选中“更新数据库”，因为您无需向成员资格表中添加用户数据。

    ![添加 sql][添加 sql]

5.  单击“发布”。
6.  导航到 [https://developers.facebook.com/apps][https://developers.facebook.com/apps] 页面，并将“应用程序域”和“站点 URL”设置更改为 Azure URL。
7.  测试应用程序。确认只有具有 *canEdit* 角色的用户可以更改数据。确认匿名用户只能查看主页。确认经过身份验证的用户可以导航到不会更改数据的所有链接。
8.  下次发布应用程序时，请确保取消选中 **DefaultConnectionDeploy** 下的“更新数据库”。

    ![设置][9]

## <a name="ppd2"></a><span class="short-header">更新数据库</span>更新成员资格数据库

在将站点部署到 Azure 且您拥有更多已注册用户后，您可能希望使其中一些用户成为 *canEdit* 角色的成员。在本节中，我们将使用 Visual Studio 连接到 SQL 数据库并将用户添加到 *canEdit* 角色。

![设置][9]

1.  在“解决方案资源管理器”中，右键单击该项目并单击“发布”。
    ![发布][发布]

2.  单击“设置”选项卡。
3.  复制连接字符串。例如，本示例中使用的连接字符串为：
    Data Source=tcp:d015leqjqx.database.chinacloudapi.cn,1433;
    Initial Catalog=ContactDB2;User Id=ricka0@d015lxyze;Password=xyzxyz
4.  关闭“发布”对话框。
5.  在“视图”菜单中，单击“服务器资源管理器”。

6.  单击“连接到数据库”图标。

    ![发布][10]

7.  如果提示您选择数据源，请单击“Microsoft SQL Server”。
    ![发布][11]

8.  复制并粘贴“服务器名称”，该名称以 *tcp* 开头（参见下图）。
9.  单击“使用 SQL Server 身份验证”
10. 输入您的“用户名”和“密码”，它们位于您所复制的连接字符串中。
11. 输入数据库名称（ContactDB，或者如果您没有将其命名为 ContactDB，则为数据库中“Initial Catalog=”后的字符串。）如果出现错误对话框，请参见下一节。
12. 单击“测试连接”。如果出现错误对话框，请参见下一节。
    ![“添加连接”对话框][“添加连接”对话框]

## 无法打开服务器登录错误

如果出现提示“无法打开服务器”的错误对话框，您将需要将 IP 地址添加到允许的 IP 中。

![防火墙错误][防火墙错误]

1.  在 Azure 门户中的左侧选项卡中选择 **SQL Databases**。

    ![选择 SQL][选择 SQL]

2.  选择您要打开的数据库。

3.  单击“为此 IP 地址设置 Azure 防火墙规则”链接。

    ![防火墙规则][防火墙规则]

4.  当系统提示您“现有防火墙规则中不包括当前 IP 地址 xxx.xxx.xxx.xxx。是否要更新防火墙规则？”，请单击“是”。添加此地址通常还不够，您将需要添加一系列 IP 地址。

## 添加一系列允许的 IP 地址

1.  在 Azure 门户中，单击“SQL Databases”。
2.  单击托管您的数据库的“服务器”。

    ![数据库服务器][数据库服务器]

3.  单击页面顶部的“配置”。
4.  添加规则名称、起始和结束 IP 地址。
    ![IP 范围][IP 范围]

5.  在页面底部，单击“保存”。
6.  现在您可以使用前面所示的步骤编辑成员资格数据库。

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

本教程和示例应用程序由 [Rick Anderson][12] (Twitter [@RickAndMSFT][Rick Anderson]) 在 Tom Dykstra、Tom FitzMacken 和 Barry Dorrans (Twitter [@blowdart][@blowdart]) 的帮助下编写。

请提供有关您喜欢的内容或者您希望看到改善的内容的反馈，不仅关于教程本身，也关于它所演示的产品。您的反馈将帮助我们确定优先改进哪些方面。我们特别希望确定大家对于对配置和部署成员资格数据库的流程进行更多自动化的兴趣有多大。

若要获取丰富多彩的 Facebook、Google 和 Yahoo 登录按钮，请参阅博文[在 ASP.NET MVC 4 中自定义外部登录按钮][在 ASP.NET MVC 4 中自定义外部登录按钮]。有关使用 Windows 身份验证的信息，请参见以下内容：

-   [Azure 身份验证][Azure 身份验证]
-   [如何使用 ASP.NET MVC 创建 Intranet 站点][如何使用 ASP.NET MVC 创建 Intranet 站点]

在 Azure 应用程序中存储数据的另一种方法是使用 Azure 存储，该服务提供 Blob 和表形式的非关系型数据存储。以下链接提供有关 ASP.NET MVC 和 Window Azure 的更多信息。

-   [使用存储表、队列和 Blob 的 .NET 多层应用程序][使用存储表、队列和 Blob 的 .NET 多层应用程序]。
-   [ASP.NET MVC 4 简介][ASP.NET MVC 4 简介]
-   [使用 MVC 的 Entity Framework 入门][使用 MVC 的 Entity Framework 入门]
-   [OAuth 2.0 与登录][OAuth 2.0 与登录]

您已经了解如何将 Web 应用程序部署到 Azure 网站。若要了解有关如何配置、管理和缩放 Azure 网站的更多信息，请参见[常见任务][常见任务]页中的操作方法主题。

若要了解如何调试 Azure 网站，请参阅[在 Visual Studio 中对 Azure 网站进行故障排除][在 Visual Studio 中对 Azure 网站进行故障排除]。

若要了解如何将应用程序部署到 Azure 云服务，请参阅[本教程中的云服务版本][本教程中的云服务版本]和[使用 Azure 开发 Web 应用程序][使用 Azure 开发 Web 应用程序]。选择在 Azure 云服务中而不是在 Azure 网站中运行 ASP.NET Web 应用程序有以下一些原因：

-   您想要运行应用程序的 Web 服务器上的管理员权限。
-   您想要使用远程桌面连接来访问运行应用程序的 Web 服务器。
-   您的应用程序是多层的，并且您想要在多个虚拟服务器（Web 服务器和辅助服务器）上分布工作。

有关 SQL Database 和 Azure 存储的详细信息，请参阅 [Azure 上的数据存储产品/服务][Azure 上的数据存储产品/服务]。

若要了解有关如何使用 SQL Database 的详细信息，请参阅[在 ASP.NET 数据访问内容映射中使用 Azure SQL Database][在 ASP.NET 数据访问内容映射中使用 Azure SQL Database]。

若要了解有关 Entity Framework 和代码优先迁移的更多信息，请参见以下资源：

-   [使用 MVC 的 Entity Framework 入门][使用 MVC 的 Entity Framework 入门]
-   [代码优先迁移][13]

<!-- bookmarks --> <!-- links --> <!-- links from Tom's hopefully no collisions --> <!-- images-->

  [Rick Anderson]: https://twitter.com/RickAndMSFT
  [登录页面]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxb.png
  [设置开发环境]: #bkmk_setupdevenv
  [设置 Azure 环境]: #bkmk_setupwindowsazure
  [创建 ASP.NET MVC 4 应用程序]: #bkmk_createmvc4app
  [将应用程序部署到 Azure]: #bkmk_deploytowindowsazure1
  [向应用程序添加数据库]: #bkmk_addadatabase
  [添加 OAuth 提供程序]: #addOauth
  [向成员资格数据库中添加角色]: #mbrDB
  [创建数据部署脚本]: #ppd
  [将应用部署到 Azure]: #bkmk_deploytowindowsazure11
  [更新成员资格数据库]: #ppd2
  [后续步骤]: #nextsteps
  [Azure SDK for Visual Studio 2012]: http://go.microsoft.com/fwlink/?LinkId=254364
  [Web 平台安装程序 – Azure SDK for .NET]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/WebPIAzureSdk20NetVS12.png
  [Azure 管理门户]: https://manage.windowsazure.cn
  [管理门户中的“新建”按钮]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxWSnew2.png
  [管理门户中的“与数据库一起创建”链接]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxCreateWSwithDB.png
  [1]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxCreateWSwithDB_2.png
  [“新建网站 - 与数据库一起创建”向导的“数据库设置”步骤]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-setup-azure-site-004.png
  [“新建网站 – 与数据库一起创建”向导的“数据库设置”步骤]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxPrevDB.png
  [“新建项目”对话框]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-createapp-002.png
  [“新建 ASP.NET MVC 4 项目”对话框]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxb2.png
  [解决方案资源管理器中的 \_Layout.cshtml]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-createapp-004.png
  [“待办事项列表”主页]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxa.png
  [2]: http://manage.windowsazure.cn "门户"
  [管理门户的“网站”选项卡中的“联系人管理器”应用程序]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-setup-azure-site-006.png
  [“快速启动”选项卡和“下载发布配置文件”按钮]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-deploy1-download-profile.png
  [保存 .publishsettings 文件]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-deploy1-save-profile.png
  [项目上下文菜单中的“发布”]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/PublishVSSolution.png
  [导入发布设置]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/ImportPublishSettings.png
  [添加 Windows Azure 订阅]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rzAddWAsub.png
  [下载订阅]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rzDownLoad.png
  [下载发布文件]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rzDown2.png
  [导入]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rzImp.png
  [导入发布配置文件]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/ImportPublishProfile.png
  [在 Azure 中运行的“待办事项列表”主页]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/newapp005.png
  [Models 文件夹上下文菜单中的“添加类”]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-adddatabase-001.png
  [“添加新项”对话框]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-adddatabase-002.png
  [Controllers 文件夹上下文菜单中的“添加控制器”]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-controller-add-context-menu.png
  [“添加控制器”对话框]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-controller-add-controller-dialog.png
  [3]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxNewCtx.png
  [“添加控制器”消息框]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxOverwrite.png
  [代码优先迁移]: http://msdn.microsoft.com/library/hh770484.aspx
  [“工具”菜单中的“程序包管理器控制台”]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-migrations-package-manager-menu.png
  [enable-migrations]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxE.png
  [DbContext]: http://msdn.microsoft.com/zh-cn/library/system.data.entity.dbcontext(v=VS.103).aspx
  [对 Entity Framework (EF) 数据库进行种子设定和调试]: http://blogs.msdn.com/b/rickandy/archive/2013/02/12/seeding-and-debugging-entity-framework-ef-dbs.aspx
  [“程序包管理器控制台”命令]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-migrations-package-manager-console.png
  [数据的 MVC 视图]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx2.png
  [OAuth]: http://oauth.net/ "http://oauth.net/"
  [Facebook]: http://developers.facebook.com/
  [Microsoft]: http://go.microsoft.com/fwlink/?LinkID=144070
  [Twitter]: http://dev.twitter.com/
  [https://developers.facebook.com/apps]: https://developers.facebook.com/apps/
  [新建 FB 应用程序]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxFBapp.png
  [4]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxFB.png
  [FB 测试人员]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxFBt.png
  [显示表数据]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxSTD.png
  [用户 ID]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxUid.png
  [roleID]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxRoleID.png
  [用户角色 ID 表]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxUR.png
  [Authorize]: http://msdn.microsoft.com/zh-cn/library/system.web.mvc.authorizeattribute(v=vs.100).aspx
  [RequireHttps]: http://msdn.microsoft.com/zh-cn/library/system.web.mvc.requirehttpsattribute(v=vs.108).aspx
  [保护 ASP.NET MVC 4 应用程序和新 AllowAnonymous 属性]: http://blogs.msdn.com/b/rickandy/archive/2012/03/23/securing-your-asp-net-mvc-4-app-and-the-new-allowanonymous-attribute.aspx
  [CAPTCHA]: http://www.asp.net/web-pages/tutorials/security/16-adding-security-and-membership
  [启用 SSL]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxSSL.png
  [5]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxS2.png
  [认证警告]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxNOT.png
  [6]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxNOT2.png
  [dbDacFx]: http://msdn.microsoft.com/zh-cn/library/dd394698.aspx
  [Microsoft SQL Server 2012 Express 下载中心]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29062
  [SQL 安装]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxSS.png
  [“连接到服务器”对话框]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxC2S.png
  [生成脚本]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxGenScripts.png
  [设置脚本编写选项]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx11.png
  [7]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxAdv.png
  [保存或发布]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx1.png
  [3 个连接字符串]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxd.png
  [8]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/dntutmobile-deploy1-publish-001.png
  [设置]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxD2.png
  [9]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxSettings.png
  [添加 sql]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxAddSQL2.png
  [发布]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxP.png
  [10]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rxc.png
  [11]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx3.png
  [“添加连接”对话框]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx4.png
  [防火墙错误]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx5.png
  [选择 SQL]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx6.png
  [防火墙规则]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx7.png
  [数据库服务器]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx8.png
  [IP 范围]: ./media/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database-vs2012/rx9.png
  [12]: http://blogs.msdn.com/b/rickandy/
  [@blowdart]: https://twitter.com/blowdart
  [在 ASP.NET MVC 4 中自定义外部登录按钮]: http://www.beabigrockstar.com/customizing-external-login-buttons-in-asp-net-mvc-4/
  [Azure 身份验证]: http://www.asp.net/vnext/overview/fall-2012-update/windows-azure-authentication
  [如何使用 ASP.NET MVC 创建 Intranet 站点]: http://msdn.microsoft.com/zh-cn/library/gg703322(v=vs.98).aspx
  [使用存储表、队列和 Blob 的 .NET 多层应用程序]: http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-dotnet-multi-tier-app-storage-1-overview/
  [ASP.NET MVC 4 简介]: http://www.asp.net/mvc/tutorials/mvc-4/getting-started-with-aspnet-mvc4/intro-to-aspnet-mvc-4
  [使用 MVC 的 Entity Framework 入门]: http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application
  [OAuth 2.0 与登录]: http://blogs.msdn.com/b/vbertocci/archive/2013/01/02/oauth-2-0-and-sign-in.aspx
  [常见任务]: http://www.windowsazure.com/zh-cn/develop/net/common-tasks/
  [在 Visual Studio 中对 Azure 网站进行故障排除]: /zh-cn/develop/net/tutorials/troubleshoot-web-sites-in-visual-studio/
  [本教程中的云服务版本]: http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/
  [使用 Azure 开发 Web 应用程序]: http://msdn.microsoft.com/zh-cn/library/Hh674484
  [Azure 上的数据存储产品/服务]: http://social.technet.microsoft.com/wiki/contents/articles/data-storage-offerings-on-the-windows-azure-platform.aspx
  [在 ASP.NET 数据访问内容映射中使用 Azure SQL Database]: http://go.microsoft.com/fwlink/p/?LinkId=282414#ssdb
  [13]: http://msdn.microsoft.com/zh-cn/library/hh770484
