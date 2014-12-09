<properties linkid="develop-dotnet-rest-service-using-web-api" urlDisplayName="REST service using Web API" pageTitle="使用 Web API 的 .NET REST 服务 &mdash; Azure 教程" metaKeywords="Azure tutorial web site, ASP.NET API web site, Azure VS" description="本教程将向您介绍如何通过使用 Visual Studio 将使用 ASP.NET Web API 的应用程序部署到 Azure 网站。" metaCanonical="" services="web-sites" documentationCenter=".NET" title="使用 ASP.NET Web API 和 SQL 数据库的 REST 服务" authors="riande" solutions="" manager="" editor="" />

# 使用 ASP.NET Web API 和 SQL 数据库的 REST 服务

***由 [Rick Anderson][Rick Anderson] 和 Tom Dykstra 撰写。2014 年 3 月更新***

本教程将向您介绍如何使用 Visual Studio 2013 或 Visual Studio 2013 for Web Express 中的“发布 Web”向导将 ASP.NET Web 应用程序部署到 Azure 网站。（如果您喜欢使用 Visual Studio 2012，请参见[本教程的上一版本][本教程的上一版本]。）

您可以免费注册一个 Azure 帐户，而且，如果您还没有 Visual Studio 2013，则此 SDK 会自动安装 Visual Studio 2013 for Web Express。这样您就能够完全免费地开始针对 Azure 进行开发了。

本教程假定您之前未使用过 Azure。完成本教程之后，您将能够在云中启动并运行简单的 Web 应用程序。

您将了解到以下内容：

-   如何通过安装 Azure SDK 以支持计算机进行 Azure 开发。
-   如何创建 Visual Studio ASP.NET MVC 5 项目并将其发布到 Azure 网站。
-   如何使用 ASP.NET Web API 实现 Restful API 调用。
-   如何使用 SQL 数据库在 Azure 中存储数据。
-   如何将应用程序更新发布到 Azure。

您将生成一个简单的联系人列表 Web 应用程序，该应用程序基于 ASP.NET MVC 5 构建并使用 ADO.NET Entity Framework 进行数据库访问。下图演示了完整的应用程序：

![网站的屏幕快照][网站的屏幕快照]
本教程的内容：

-   [设置开发环境][设置开发环境]
-   [设置 Azure 环境][设置 Azure 环境]
-   [创建 ASP.NET MVC 5 应用程序][创建 ASP.NET MVC 5 应用程序]
-   [将应用程序部署到 Azure][将应用程序部署到 Azure]
-   [向应用程序添加数据库][向应用程序添加数据库]
-   [为数据添加控制器和视图][为数据添加控制器和视图]
-   [添加 Web API Restful 接口][添加 Web API Restful 接口]
-   [添加 XSRF 保护][添加 XSRF 保护]
-   [将应用程序更新发布到 Azure 和 SQL 数据库][将应用程序更新发布到 Azure 和 SQL 数据库]

<a name="bkmk_setupdevenv"></a>
<!-- the next line produces the "Set up the development environment" section as see at http://www.windowsazure.com/zh-cn/documentation/articles/web-sites-dotnet-get-started/ -->
[WACOM.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## <a name="bkmk_setupwindowsazure"></a> 设置 Azure 环境

接下来，通过创建 Azure 网站和 SQL Database 来设置 Azure 环境。

### 在 Azure 中创建网站和 SQL 数据库

下一步是创建您的应用程序将要使用的 Azure 网站和 SQL 数据库。

您的 Azure 网站将在共享宿主环境中运行，这意味着它将在与其他 Azure 客户端共享的虚拟机 (VM) 上运行。共享宿主环境是一种在云中开始工作的低成本方式。稍后，如果您的 Web 流量增加，则应用程序可进行扩展，通过在专用 VM 上运行来满足需要。如果您需要一个更复杂的体系结构，则可迁移到 Azure 云服务。云服务在您可根据自己的需求进行配置的专用 VM 上运行。

SQL 数据库是根据 SQL Server 技术构建的基于云的关系数据库服务。可以与 SQL Server 一起使用的工具和应用程序也可用于 SQL Database。

1.  在 [Azure 管理门户][Azure 管理门户] 中，单击左侧选项卡中的“网站”，然后单击“新建”。

2.  单击“自定义创建”。

    ![管理门户中的“与数据库一起创建”链接][管理门户中的“与数据库一起创建”链接]

    “新网站 - 自定义创建”向导将打开。

3.  在该向导的“新建网站”步骤中，在“URL”框中输入将用作您的应用程序的唯一 URL 的字符串。完整的 URL 由您在此处输入的字符串以及您在文本框下面看到的后缀组成。图中显示的是 "contactmgr11"，但该 URL 可能已被占用，因此您必须选择另一个 URL。

4.  在“区域”下拉列表中，选择离您最近的区域。

5.  在“数据库”下拉列表中，选择“创建免费的 20 MB SQL 数据库”。 **** ****

    ![“新建网站 - 与数据库一起创建”向导的“创建新网站”步骤][“新建网站 - 与数据库一起创建”向导的“创建新网站”步骤]

6.  单击对话框底部的向右箭头。

    该向导将前进到“数据库设置”步骤。

7.  在“名称”框中，输入 *ContactDB*。

8.  在“服务器”框中，选择“新建 SQL 数据库服务器”。 **** **** 或者，如果您之前创建了 SQL Server 数据库，则可从下拉列表控件中选择 SQL Server。

9.  单击对话框底部的向右箭头。

10. 输入管理员“登录名”和“密码”。如果您选择了“新建 SQL 数据库服务器”，则在此处不要输入现有名称和密码。您应输入新的名称和密码，现在定义的名称和密码将在您以后访问数据库时使用。 **** 如果您选择了之前创建的 SQL Server，系统将提示您输入之前创建的 SQL Server 帐户名称的密码。在本教程中，我们不选中“高级”框。您可以使用“高级”框设置数据库大小（默认为 1 GB，但您可以将其增加到 150 GB）和排序规则。

11. 单击对话框底部的复选标记以指示您已完成操作。

    ![“新建网站 - 与数据库一起创建”向导的“数据库设置”步骤][“新建网站 - 与数据库一起创建”向导的“数据库设置”步骤]

    下图显示了使用现有 SQL Server 和登录名。

    ![“新建网站 - 与数据库一起创建”向导的“数据库设置”步骤][1]

    管理门户返回到“网站”页面，“状态”列显示正在创建网站。稍后（通常不到一分钟），“状态”列会显示已成功创建网站。在左侧的导航栏中，您的帐户中拥有的网站的数量将会显示在“网站”图标旁边，而数据库的数量将会显示在“SQL 数据库”图标旁边。

<!-- [Web Sites page of Management Portal, web site created][setup009] -->

## <a name="bkmk_createmvc4app"></a> 创建 ASP.NET MVC 5 应用程序

您已经创建了一个 Azure 网站，但其中还没有内容。下一步将创建要发布到 Azure 的 Visual Studio Web 应用程序。

### 创建项目

1.  启动 Visual Studio 2013。
2.  在“文件”菜单中，单击“新建项目”。
3.  在“新建项目”对话框中，展开“Visual C#”并选择“Web”，然后选择“ASP.NET MVC 5 Web 应用程序。保持默认值 **.NET Framework 4.5**。将应用程序命名为 **ContactManager**，然后单击“确定”。
    ![“新建项目”对话框][“新建项目”对话框]
4.  在“新建 ASP.NET 项目”对话框中，选择 **MVC** 模板，选中 **Web API**，然后单击“更改身份验证”。

    ![“新建 ASP.NET 项目”对话框][“新建 ASP.NET 项目”对话框]

5.  在“更改身份验证”对话框中，单击“无身份验证”，然后单击“确定”。

    ![无身份验证][无身份验证]

    您要创建的示例应用程序将没有需要用户登录的功能。有关如何实现身份验证和授权功能的信息，请参见本教程末尾的[后续步骤][后续步骤]一节。

6.  在“新建 ASP.NET 项目”对话框中，单击“确定”。

    ![“新建 ASP.NET 项目”对话框][2]

### 设置页眉和页脚

1.  在“解决方案资源管理器”中，展开 *Views\\Shared* 文件夹并打开 \*\_Layout.cshtml\* 文件。

    ![解决方案资源管理器中的 \_Layout.cshtml][解决方案资源管理器中的 \_Layout.cshtml]

2.  将 \*\_Layout.cshtml\* 文件的内容替换为以下代码。

        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="utf-8" />
            <title>@ViewBag.Title - Contact Manager</title>
            <link href="~/favicon.ico" rel="shortcut icon" type="image/x-icon" />
            <meta name="viewport" content="width=device-width" />
            @Styles.Render("~/Content/css")
            @Scripts.Render("~/bundles/modernizr")
        </head>
        <body>
            <header>
                <div class="content-wrapper">
                    <div class="float-left">
                        <p class="site-title">@Html.ActionLink("Contact Manager", "Index", "Home")</p>
                    </div>
                </div>
            </header>
            <div id="body">
                @RenderSection("featured", required: false)
                <section class="content-wrapper main-content clear-fix">
                    @RenderBody()
                </section>
            </div>
            <footer>
                <div class="content-wrapper">
                    <div class="float-left">
                        <p>&copy; @DateTime.Now.Year - Contact Manager</p>
                    </div>
                </div>
            </footer>
            @Scripts.Render("~/bundles/jquery")
            @RenderSection("scripts", required: false)
        </body>
        </html>

上面的标记会将应用程序名称从 "My ASP.NET App" 更改为 "Contact Manager"，并移除到“主页”、“关于”以及“联系人”的链接。

### 在本地运行应用程序

1.  按 CTRL+F5 运行应用程序。
    应用程序主页显示在默认浏览器中。
    ![待办事项列表主页][待办事项列表主页]

这就是您创建将要部署到 Azure 的应用程序目前所需的全部操作。稍后您将添加数据库功能。

## <a name="bkmk_deploytowindowsazure1"></a>将应用程序部署到 Azure

1.  在 Visual Studio 中，在“解决方案资源管理器”中右键单击该项目，从上下文菜单中选择“发布”。

    ![项目上下文菜单中的“发布”][项目上下文菜单中的“发布”]

    “发布 Web”向导将打开。

2.  在“发布 Web”向导的“配置文件”选项卡中，单击“导入”。

    ![导入发布设置][导入发布设置]

    “导入发布配置文件”对话框随即出现。

3.  选择“从 Azure 网站导入”。如果之前未进行过此操作，则必须首先登录。单击“登录”。输入与您的订阅关联的用户并按步骤登录。

    ![登录][登录]

    从下拉列表中选择您的网站，然后单击“确定”。

    ![导入发布配置文件][导入发布配置文件]

4.  在“连接”选项卡中，单击“验证连接”以确保设置正确。

    ![验证连接][验证连接]

5.  在连接通过验证后，“验证连接”按钮旁边会出现一个绿色复选标记。

    ![“连接”选项卡中的连接成功图标和“下一步”按钮][“连接”选项卡中的连接成功图标和“下一步”按钮]

6.  单击“下一步”。

    ![“设置”选项卡][“设置”选项卡]

    您可以接受此选项卡上的默认设置。您将要部署“发布”生成配置，因此不需要在目标服务器上删除文件、预编译应用程序或排除 App\_Data 文件夹中的文件。如果您希望在实时 Azure 站点上进行调试，则需要部署调试配置（不是发布）。请参见本教程结尾部分的[后续步骤][后续步骤]。

7.  在“预览”选项卡中，单击“开始预览”。

    该选项卡会显示将复制到服务器的文件的列表。显示预览并不是发布应用程序所必需的，但它是一个需要了解的很有用的功能。在本例中，您不需要对显示的文件列表执行任何操作。下次发布时，只有已更改的文件会显示在预览列表中。

    ![“预览”选项卡中的“开始预览”按钮][“预览”选项卡中的“开始预览”按钮]

8.  单击“发布”。

    Visual Studio 开始执行将文件复制到 Azure 服务器的过程。“输出”窗口将显示已执行的部署操作并报告已成功完成部署。

9.  默认浏览器会自动打开，并指向所部署站点的 URL。

    您创建的应用程序现在在云中运行。

    ![在 Azure 中运行的待办事项列表主页][在 Azure 中运行的待办事项列表主页]

## <a name="bkmk_addadatabase"></a> 向应用程序添加数据库

接下来，您将更新 MVC 应用程序以添加显示和更新联系人以及在数据库中存储数据的功能。应用程序将使用 Entity Framework 创建数据库并读取和更新数据库中的数据。

### 为联系人添加数据模型类

首先，使用代码创建一个简单的数据模型。

1.  在“解决方案资源管理器”中，右键单击 Models 文件夹，单击“添加”，然后单击“类”。

    ![Models 文件夹上下文菜单中的“添加类”][Models 文件夹上下文菜单中的“添加类”]

2.  在“添加新项”对话框中，将新的类文件命名为 *Contact.cs*，然后单击“添加”。

    ![“添加新项”对话框][“添加新项”对话框]

3.  将 Contacts.cs 文件的内容替换为以下代码。

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
                public string Email { get; set; }
                public string Twitter { get; set; }
                public string Self
                {
                    get { return string.Format(CultureInfo.CurrentCulture,
                         "api/contacts/{0}", this.ContactId); }
                    set { }
                }
            }
        }

**Contacts** 类定义您将为每个联系人存储的数据以及数据库需要的主键 ContactID。您可以在本教程结尾部分的[后续步骤][后续步骤]中了解到更多有关数据模型的信息。

### 创建使应用程序用户可以使用联系人的网页

ASP.NET MVC 基架功能可以自动生成用于执行创建、读取、更新和删除 (CRUD) 操作的代码。

## <a name="bkmk_addcontroller"></a>为数据添加控制器和视图

1.  在“解决方案资源管理器”中，展开 Controllers 文件夹。

2.  生成项目 **(Ctrl+Shift+B)**。（在使用基架机制前必须生成项目。）

3.  右键单击 Controllers 文件夹并单击“添加”，然后单击“控制器”。

    ![Controllers 文件夹上下文菜单中的“添加控制器”][Controllers 文件夹上下文菜单中的“添加控制器”]

4.  在“添加基架”对话框中，选择“包含视图的 MVC 控制器（使用 Entity Framework）”并单击“添加”。

![添加控制器][添加控制器]

1.  将控制器名设置为 **HomeController**。选择“联系人”作为模型类。单击“新建数据上下文”按钮并接受默认的 "ContactManager.Models.ContactManagerContext" 为“新的数据上下文类型”。单击“添加”。

    ![“添加控制器”对话框][“添加控制器”对话框]

    一个提示对话框出现：“名为 HomeController 的文件已存在。是否希望将其替换？”。单击“是”。我们正在覆盖使用新项目创建的主控制器。我们将为联系人列表使用新的主控制器。

    Visual Studio 将创建一个控制器方法并为 **Contact** 对象的 CRUD 数据库操作创建视图。

## 启用迁移、创建数据库、添加示例数据和数据初始值设定项

接下来的任务是启用[代码优先迁移][代码优先迁移] 功能以便基于您创建的数据模型创建数据库。

1.  在“工具”菜单中，依次选择“库程序包管理器”和“程序包管理器控制台”。

    ![“工具”菜单中的“程序包管理器控制台”][“工具”菜单中的“程序包管理器控制台”]

2.  在“程序包管理器控制台”窗口中，输入以下命令：

        enable-migrations 

    **enable-migrations** 命令将创建一个 *Migrations* 文件夹，并在该文件夹中放入一个可编辑以配置 Migrations 的 *Configuration.cs* 文件。

3.  在“程序包管理器控制台”窗口中，输入以下命令：

        add-migration Initial

    **add-migration Initial** 命令将生成一个创建数据库的名为 **\<date\_stamp\>Initial** 的类。第一个参数 (*Initial*) 是任意参数并将用于创建文件名称。您可以在“解决方案资源管理器”中查看新的类文件。

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
                   Twitter = "debra_example"
               },
                new Contact
                {
                    Name = "Thorsten Weinrich",
                    Address = "5678 1st Ave W",
                    City = "Redmond",
                    State = "WA",
                    Zip = "10999",
                    Email = "thorsten@example.com",
                    Twitter = "thorsten_example"
                },
                new Contact
                {
                    Name = "Yuhong Li",
                    Address = "9012 State st",
                    City = "Redmond",
                    State = "WA",
                    Zip = "10999",
                    Email = "yuhong@example.com",
                    Twitter = "yuhong_example"
                },
                new Contact
                {
                    Name = "Jon Orton",
                    Address = "3456 Maple St",
                    City = "Redmond",
                    State = "WA",
                    Zip = "10999",
                    Email = "jon@example.com",
                    Twitter = "jon_example"
                },
                new Contact
                {
                    Name = "Diliana Alexieva-Bosseva",
                    Address = "7890 2nd Ave E",
                    City = "Redmond",
                    State = "WA",
                    Zip = "10999",
                    Email = "diliana@example.com",
                    Twitter = "diliana_example"
                }
                );
        }

    上面这段代码将用联系人信息初始化数据库。有关对数据库进行种子设定的更多信息，请参见[调试 Entity Framework (EF) 数据库][调试 Entity Framework (EF) 数据库]。

7.  在“程序包管理器控制台”中输入以下命令：

        update-database

    ![“程序包管理器控制台”命令][“程序包管理器控制台”命令]

    **update-database** 用于运行将创建数据库的初始迁移。默认情况下，将以 SQL Server Express LocalDB 数据库的形式创建数据库。

8.  按 Ctrl+F5 运行应用程序。

应用程序将显示种子数据并提供编辑、详细信息和删除链接。

![数据的 MVC 视图][数据的 MVC 视图]

## <a name="bkmk_addview"></a>编辑视图

1.  打开 *Views\\Home\\Index.cshtml* 文件。在下一步中，我们将生成的标记替换为使用 [jQuery][jQuery] 和 [Knockout.js][Knockout.js] 的代码。此新代码将使用 Web API 和 JSON 检索联系人列表，然后使用 knockout.js 将联系人数据绑定至 UI。有关更多信息，请参见本教程结尾部分的[后续步骤][后续步骤]。

2.  将文件的内容替换为以下代码。

        @model IEnumerable<ContactManager.Models.Contact>
        @{
            ViewBag.Title = "Home";
        }
        @section Scripts {
            @Scripts.Render("~/bundles/knockout")
            <script type="text/javascript">
                function ContactsViewModel() {
                    var self = this;
                    self.contacts = ko.observableArray([]);
                    self.addContact = function () {
                        $.post("api/contacts",
                            $("#addContact").serialize(),
                            function (value) {
                                self.contacts.push(value);
                            },
                            "json");
                    }
                    self.removeContact = function (contact) {
                        $.ajax({
                            type: "DELETE",
                            url: contact.Self,
                            success: function () {
                                self.contacts.remove(contact);
                            }
                        });
                    }

                    $.getJSON("api/contacts", function (data) {
                        self.contacts(data);
                    });
                }
                ko.applyBindings(new ContactsViewModel());  
        </script>
        }
        <ul id="contacts" data-bind="foreach: contacts">
            <li class="ui-widget-content ui-corner-all">
                <h1 data-bind="text: Name" class="ui-widget-header"></h1>
                <div><span data-bind="text: $data.Address || 'Address?'"></span></div>
                <div>
                    <span data-bind="text: $data.City || 'City?'"></span>,
                    <span data-bind="text: $data.State || 'State?'"></span>
                    <span data-bind="text: $data.Zip || 'Zip?'"></span>
                </div>
                <div data-bind="if: $data.Email"><a data-bind="attr: { href: 'mailto:' + Email }, text: Email"></a></div>
                <div data-bind="ifnot: $data.Email"><span>Email?</span></div>
                <div data-bind="if: $data.Twitter"><a data-bind="attr: { href: 'http://twitter.com/' + Twitter }, text: '@@' + Twitter"></a></div>
                <div data-bind="ifnot: $data.Twitter"><span>Twitter?</span></div>
                <p><a data-bind="attr: { href: Self }, click: $root.removeContact" class="removeContact ui-state-default ui-corner-all">Remove</a></p>
            </li>
        </ul>
        <form id="addContact" data-bind="submit: addContact">
            <fieldset>
                <legend>Add New Contact</legend>
                <ol>
                    <li>
                        <label for="Name">Name</label>
                        <input type="text" name="Name" />
                    </li>
                    <li>
                        <label for="Address">Address</label>
                        <input type="text" name="Address" >
                    </li>
                    <li>
                        <label for="City">City</label>
                        <input type="text" name="City" />
                    </li>
                    <li>
                        <label for="State">State</label>
                        <input type="text" name="State" />
                    </li>
                    <li>
                        <label for="Zip">Zip</label>
                        <input type="text" name="Zip" />
                    </li>
                    <li>
                        <label for="Email">E-mail</label>
                        <input type="text" name="Email" />
                    </li>
                    <li>
                        <label for="Twitter">Twitter</label>
                        <input type="text" name="Twitter" />
                    </li>
                </ol>
                <input type="submit" value="Add" />
            </fieldset>
        </form>

3.  右键单击 Content 文件夹并单击“添加”，然后单击“新建项...”。

    ![在 Content 文件夹上下文菜单中添加样式表][在 Content 文件夹上下文菜单中添加样式表]

4.  在“添加新项”对话框中，在右上的搜索框中输入 **Style**，然后选择“样式表”。
    ![添加新项对话框][添加新项对话框]

5.  将文件命名为 *Contacts.css* 并单击“添加”。将文件的内容替换为以下代码。

        .column {
            float: left;
            width: 50%;
            padding: 0;
            margin: 5px 0;
        }
        form ol {
            list-style-type: none;
            padding: 0;
            margin: 0;
        }
        form li {
            padding: 1px;
            margin: 3px;
        }
        form input[type="text"] {
            width: 100%;
        }
        #addContact {
            width: 300px;
            float: left;
            width:30%;
        }
        #contacts {
            list-style-type: none;
            margin: 0;
            padding: 0;
            float:left;
            width: 70%;
        }
        #contacts li {
            margin: 3px 3px 3px 0;
            padding: 1px;
            float: left;
            width: 300px;
            text-align: center;
            background-image: none;
            background-color: #F5F5F5;
        }
        #contacts li h1
        {
            padding: 0;
            margin: 0;
            background-image: none;
            background-color: Orange;
            color: White;
            font-family: Trebuchet MS, Tahoma, Verdana, Arial, sans-serif;
        }
        .removeContact, .viewImage
        {
            padding: 3px;
            text-decoration: none;
        }

    该样式表将用作联系人管理器应用程序的布局、颜色和样式。

6.  打开 *App\_Start\\BundleConfig.cs* 文件。

7.  添加以下代码以注册 [Knockout][Knockout] 插件。

        bundles.Add(new ScriptBundle("~/bundles/knockout").Include(
                    "~/Scripts/knockout-{version}.js"));

    此示例使用 knockout 来简化处理屏幕模板的动态 JavaScript 代码。

8.  修改 contents/css 条目以注册 *contacts.css* 样式表。将以下行

                 bundles.Add(new StyleBundle("~/Content/css").Include(
                   "~/Content/bootstrap.css",
                   "~/Content/site.css"));

    更改为：

        bundles.Add(new StyleBundle("~/Content/css").Include(
                   "~/Content/bootstrap.css",
                   "~/Content/contacts.css",
                   "~/Content/site.css"));

9.  在“程序包管理器控制台”中运行以下命令以安装 Knockout。

    Install-Package knockoutjs

## <a name="bkmk_addwebapi"></a>为 Web API Restful 接口添加控制器

1.  在“解决方案资源管理器”中，右键单击 Controllers，然后依次单击“添加”和“控制器....”。

2.  在“添加基架”对话框中，进入“包含操作的 Web API 2 控制器（使用 Entity Framework）”并单击“添加”。

    ![添加 API 控制器][添加 API 控制器]

3.  在“添加控制器”对话框中，输入 "ContactsController" 作为您的控制器名称。为“模型类”选择 "Contact (ContactManager.Models)"。保留“数据上下文类”的默认值。

4.  单击“添加”。

### 在本地运行应用程序

1.  按 Ctrl+F5 运行应用程序。

    ![索引页面][网站的屏幕快照]

2.  输入联系人信息并单击“添加”。该应用程序将返回主页并显示刚才输入的联系人信息。

    ![包含待办事项列表项的索引页面][包含待办事项列表项的索引页面]

3.  在浏览器中，将 **/api/contacts** 追加到 URL。

    生成的 URL 类似于 http://localhost:1234/api/contacts。添加的 RESTful Web API 将返回存储的联系人。Firefox 和 Chrome 将以 XML 格式显示数据。

    ![包含待办事项列表项的索引页面][3]

    IE 将提示您打开或保存联系人。

    ![Web API 保存对话框][Web API 保存对话框]

    您可以在记事本或浏览器中打开返回的联系人。

    此输出可由另一个应用程序（如移动 Web 页面或应用程序）使用。

    ![Web API 保存对话框][4]

    **安全警告**：此时，您的应用程序是不安全的，而且容易受到 CSRF 攻击。本教程稍后部分将将解决这一漏洞。有关更多信息，请参见[防止跨站点请求伪造 (CSRF) 攻击][防止跨站点请求伪造 (CSRF) 攻击]。

## <a name="xsrf"></a><span class="short-header">XSRF</span>添加 XSRF 保护

跨站点请求伪造（也称为 XSRF 或 CSRF）是一种针对 Web 托管型应用程序的攻击，恶意网站凭此可以影响客户端浏览器与受该浏览器信任的网站之间的交互。这些攻击出现的原因可能是 Web 浏览器针对每一个对网站的请求自动发送身份验证令牌。典型示例是身份验证 cookie，如 ASP.NET 的表单身份验证票证。然而，使用任何持久身份验证（如 Windows Authentication、Basic 等）的网站也可能成为受攻击目标。

XSRF 攻击不同于网络钓鱼攻击。网络钓鱼攻击需要与受害者进行交互。在网络钓鱼攻击中，恶意网站将仿冒目标网站，受到欺骗的受害者会向攻击者提供敏感信息。在 XSRF 攻击中，通常不必与受害者进行交互。相反，浏览器自动向目标网站发送所有相关 cookie 为攻击者提供了可乘之机。

有关更多信息，请参见[打开 Web 应用程序安全性项目][打开 Web 应用程序安全性项目] (OWASP) [XSRF][XSRF]。

1.  在“解决方案资源管理器”中，右键单击 **ContactManager** 项目并单击“添加”，然后单击“类”。

2.  将文件命名为 *ValidateHttpAntiForgeryTokenAttribute.cs* 并添加以下代码：

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Net;
        using System.Net.Http;
        using System.Web.Helpers;
        using System.Web.Http.Controllers;
        using System.Web.Http.Filters;
        using System.Web.Mvc;
        namespace ContactManager.Filters
        {
            public class ValidateHttpAntiForgeryTokenAttribute : AuthorizationFilterAttribute
            {
                public override void OnAuthorization(HttpActionContext actionContext)
                {
                    HttpRequestMessage request = actionContext.ControllerContext.Request;
                    try
                    {
                        if (IsAjaxRequest(request))
                        {
                            ValidateRequestHeader(request);
                        }
                        else
                        {
                            AntiForgery.Validate();
                        }
                    }
                    catch (HttpAntiForgeryException e)
                    {
                        actionContext.Response = request.CreateErrorResponse(HttpStatusCode.Forbidden, e);
                    }
                }
                private bool IsAjaxRequest(HttpRequestMessage request)
                {
                    IEnumerable<string> xRequestedWithHeaders;
                    if (request.Headers.TryGetValues("X-Requested-With", out xRequestedWithHeaders))
                    {
                        string headerValue = xRequestedWithHeaders.FirstOrDefault();
                        if (!String.IsNullOrEmpty(headerValue))
                        {
                            return String.Equals(headerValue, "XMLHttpRequest", StringComparison.OrdinalIgnoreCase);
                        }
                    }
                    return false;
                }
                private void ValidateRequestHeader(HttpRequestMessage request)
                {
                    string cookieToken = String.Empty;
                    string formToken = String.Empty;
                    IEnumerable<string> tokenHeaders;
                    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
                    {
                        string tokenValue = tokenHeaders.FirstOrDefault();
                        if (!String.IsNullOrEmpty(tokenValue))
                        {
                            string[] tokens = tokenValue.Split(':');
                            if (tokens.Length == 2)
                            {
                                cookieToken = tokens[0].Trim();
                                formToken = tokens[1].Trim();
                            }
                        }
                    }
                    AntiForgery.Validate(cookieToken, formToken);
                }
            }
        }

3.  将以下 *using* 语句添加到联系人控制器以便您可以访问 **[ValidateHttpAntiForgeryToken]** 属性。

    using ContactManager.Filters;

4.  将 **[ValidateHttpAntiForgeryToken]** 属性添加到 **ContactsController** 的 Post 方法以保护其免受 XSRF 威胁。将其添加到 "PutContact"、"PostContact" 和 **DeleteContact** 操作方法。

    [ValidateHttpAntiForgeryToken]
     public IHttpActionResult PutContact(int id, Contact contact)
     {

5.  更新 *Views\\Home\\Index.cshtml* 文件的 *Scripts* 部分以包含代码，从而获取 XSRF 令牌。

         @section Scripts {
            @Scripts.Render("~/bundles/knockout")
            <script type="text/javascript">
                @functions{
                   public string TokenHeaderValue()
                   {
                      string cookieToken, formToken;
                      AntiForgery.GetTokens(null, out cookieToken, out formToken);
                      return cookieToken + ":" + formToken;                
                   }
                }

               function ContactsViewModel() {
                  var self = this;
                  self.contacts = ko.observableArray([]);
                  self.addContact = function () {

                     $.ajax({
                        type: "post",
                        url: "api/contacts",
                        data: $("#addContact").serialize(),
                        dataType: "json",
                        success: function (value) {
                           self.contacts.push(value);
                        },
                        headers: {
                           'RequestVerificationToken': '@TokenHeaderValue()'
                        }
                     });

                  }
                  self.removeContact = function (contact) {
                     $.ajax({
                        type: "DELETE",
                        url: contact.Self,
                        success: function () {
                           self.contacts.remove(contact);
                        },
                        headers: {
                           'RequestVerificationToken': '@TokenHeaderValue()'
                        }

                     });
                  }

                  $.getJSON("api/contacts", function (data) {
                     self.contacts(data);
                  });
               }
               ko.applyBindings(new ContactsViewModel());
            </script>

## <a name="bkmk_deploydatabaseupdate"></a>将应用程序更新发布到 Azure 和 SQL 数据库

若要发布应用程序，可重复之前遵循的过程。

1.  在“解决方案资源管理器”中，右键单击项目并选择“发布”。

    ![发布][发布]

2.  单击“设置”选项卡。

3.  在 **ContactsManagerContext(ContactsManagerContext)** 之下，单击 **v** 图标将远程连接字符串更改为联系人数据库的连接字符串。单击 **ContactDB**。

    ![设置][设置]

4.  选中“执行代码优先迁移（应用程序启动时运行）”复选框。

5.  单击“下一步”，然后单击“预览”。Visual Studio 将显示一个需要添加或更新的文件列表。

6.  单击“发布”。
    部署完成后，浏览器将打开该应用程序的主页。

    ![没有任何联系人的索引页面][网站的屏幕快照]

    Visual Studio 发布过程自动将部署的 *Web.config* 文件中的连接字符串配置为指向 SQL 数据库。其还配置了代码优先迁移，以在部署后应用程序首次访问数据库时自动将数据库升级至最新版本。

    由于这种配置，通过运行之前创建的 **Initial** 类中的代码，代码优先创建了数据库。其会在应用程序在部署后首次尝试访问数据库时进行。

7.  在本地运行应用程序时按之前所做输入一个联系人以验证数据库部署是否成功。

当看到输入的项得到了保存并显示在联系人管理器页面，您就可以知晓其已存储在数据库中。

![包含联系人的索引页面][包含待办事项列表项的索引页面]

该应用程序现在是运行在云中，使用 SQL 数据库存储其数据。在 Azure 中测试应用程序完成后，将其删除。该应用程序是公开的并且没有限制访问的机制。

## <a name="nextsteps"></a> 后续步骤

实际的应用程序需要身份验证和授权，您可以使用成员资格数据库实现此目的。教程[使用 OAuth、成员资格以及 SQL 数据库部署安全的 ASP.NET MVC 应用程序][使用 OAuth、成员资格以及 SQL 数据库部署安全的 ASP.NET MVC 应用程序]基于本教程，其中介绍了如何部署包含成员资格数据库的 Web 应用程序。

另一种在 Azure 应用程序中存储数据的方法是使用 Azure 存储，该方法以 Blob 和表的形式提供非关系数据存储。以下链接提供了更多有关 Web API、ASP.NET MVC 以及 Window Azure 的信息。

-   [使用 MVC 的 Entity Framework 入门][使用 MVC 的 Entity Framework 入门]
-   [ASP.NET MVC 5 简介][ASP.NET MVC 5 简介]
-   [您的第一个 ASP.NET Web API][您的第一个 ASP.NET Web API]
-   [调试 WAWS][调试 WAWS]

本教程和示例应用程序由 [Rick Anderson][5] (Twitter [@RickAndMSFT][Rick Anderson]) 在 Tom Dykstra 和 Barry Dorrans (Twitter [@blowdart][@blowdart]) 的帮助下编写完成。

请提供有关您喜欢的内容或者您希望看到改善的内容的反馈，不仅关于教程本身，也关于它所演示的产品。您的反馈将帮助我们确定优先改进哪些方面。我们特别希望确定大家对于对配置和部署成员资格数据库的流程进行更多自动化的兴趣有多大。

<!-- bookmarks --> <!-- links --> <!-- images-->

  [Rick Anderson]: https://twitter.com/RickAndMSFT
  [本教程的上一版本]: /zh-cn/develop/net/tutorials/get-started-vs2012/
  [网站的屏幕快照]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobil-intro-finished-web-app.png
  [设置开发环境]: #bkmk_setupdevenv
  [设置 Azure 环境]: #bkmk_setupwindowsazure
  [创建 ASP.NET MVC 5 应用程序]: #bkmk_createmvc4app
  [将应用程序部署到 Azure]: #bkmk_deploytowindowsazure1
  [向应用程序添加数据库]: #bkmk_addadatabase
  [为数据添加控制器和视图]: #bkmk_addcontroller
  [添加 Web API Restful 接口]: #bkmk_addwebapi
  [添加 XSRF 保护]: #xsrf
  [将应用程序更新发布到 Azure 和 SQL 数据库]: #bkmk_deploydatabaseupdate
  [Azure 管理门户]: https://manage.windowsazure.cn
  [管理门户中的“与数据库一起创建”链接]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rr6.PNG
  [“新建网站 - 与数据库一起创建”向导的“创建新网站”步骤]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rrCWS.png
  [“新建网站 - 与数据库一起创建”向导的“数据库设置”步骤]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-setup-azure-site-004.png
  [1]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rxPrevDB.png
  [“新建项目”对话框]: /media/devcenter/dotnet/dntutmobile-createapp-002.png
  [“新建 ASP.NET 项目”对话框]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rt3.PNG
  [无身份验证]: ./media/web-sites-dotnet-get-started-vs2013/GS13noauth.png
  [后续步骤]: #nextsteps
  [2]: ./media/web-sites-dotnet-get-started-vs2013/GS13newaspnetprojdb.png
  [解决方案资源管理器中的 \_Layout.cshtml]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-createapp-004.png
  [待办事项列表主页]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rr5.PNG
  [项目上下文菜单中的“发布”]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/PublishVSSolution.png
  [导入发布设置]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/ImportPublishSettings.png
  [登录]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rr7.png
  [导入发布配置文件]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rr8.png
  [验证连接]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/ValidateConnection.png
  [“连接”选项卡中的连接成功图标和“下一步”按钮]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-deploy1-publish-005.png
  [“设置”选项卡]: ./media/web-sites-dotnet-get-started-vs2013/GS13SettingsTab.png
  [“预览”选项卡中的“开始预览”按钮]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-deploy1-publish-007.png
  [在 Azure 中运行的待办事项列表主页]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rxz2.png
  [Models 文件夹上下文菜单中的“添加类”]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-adddatabase-001.png
  [“添加新项”对话框]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-adddatabase-002.png
  [Controllers 文件夹上下文菜单中的“添加控制器”]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-controller-add-context-menu.png
  [添加控制器]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rrAC.PNG
  [“添加控制器”对话框]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rr9.PNG
  [代码优先迁移]: http://curah.microsoft.com/55220
  [“工具”菜单中的“程序包管理器控制台”]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-migrations-package-manager-menu.png
  [调试 Entity Framework (EF) 数据库]: http://blogs.msdn.com/b/rickandy/archive/2013/02/12/seeding-and-debugging-entity-framework-ef-dbs.aspx
  [“程序包管理器控制台”命令]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-migrations-package-manager-console.png
  [数据的 MVC 视图]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rxz3.png
  [jQuery]: http://jquery.com/
  [Knockout.js]: http://knockoutjs.com/
  [在 Content 文件夹上下文菜单中添加样式表]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-controller-add-contents-context-menu.png
  [添加新项对话框]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rxStyle.png
  [Knockout]: http://knockoutjs.com/index.html "KO"
  [添加 API 控制器]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rt1.PNG
  [包含待办事项列表项的索引页面]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-webapi-added-contact.png
  [3]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rxFFchrome.png
  [Web API 保存对话框]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-webapi-save-returned-contacts.png
  [4]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/dntutmobile-webapi-contacts-in-notepad.png
  [防止跨站点请求伪造 (CSRF) 攻击]: http://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-(csrf)-attacks
  [打开 Web 应用程序安全性项目]: https://www.owasp.org/index.php/Main_Page
  [XSRF]: https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)
  [发布]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rxP.png
  [设置]: ./media/web-sites-dotnet-rest-service-aspnet-api-sql-database/rt5.png
  [使用 OAuth、成员资格以及 SQL 数据库部署安全的 ASP.NET MVC 应用程序]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/
  [使用 MVC 的 Entity Framework 入门]: http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application
  [ASP.NET MVC 5 简介]: http://www.asp.net/mvc/tutorials/mvc-5/introduction/getting-started
  [您的第一个 ASP.NET Web API]: http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api
  [调试 WAWS]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio/
  [5]: http://blogs.msdn.com/b/rickandy/
  [@blowdart]: https://twitter.com/blowdart
