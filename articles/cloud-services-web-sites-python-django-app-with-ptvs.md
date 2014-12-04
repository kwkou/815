<properties linkid="develop-python-tutorials-django-with-python-tools-for-visual-studio" urlDisplayName="Django with Python Tools for Visual Studio 2.0" pageTitle="使用 Python Tools for Visual Studio 2.0 创建 Django 应用程序" metaKeywords="" description="了解如何使用 Python Tools for Visual Studio 创建一个 Django 应用程序，该应用程序在 SQL Database 或 MySQL 数据库实例中存储数据并且可被部署到某一网站或云服务。" metaCanonical="" services="web-sites,cloud-services" documentationCenter="Python" title="使用 Python Tools 2.0 for Visual Studio 创建 Django 应用程序" authors="" solutions="" manager="" editor="" />

# 使用 Python Tools 2.0 for Visual Studio 创建 Django 应用程序

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>在本教程中，我们将使用 Python Tools 2.0 for Visual Studio 创建一个简单的 Django 应用程序。该应用程序将允许用户对轮询投票。首先我们将使用本地 sqlite3 数据库，然后移到 Azure 上的 SQL Server 或 MySQL 数据库。我们将演示如何实现 Django 管理界面，并使用该界面向我们的数据库添加轮询。我们还将使用在 Visual Studio 中集成的 Django shell。最后，我们将应用程序部署到一个 Azure 网站和 Azure 云服务。</p>
<p>如果你更愿意观看视频，右侧的视频片段提供了与本教程相同的步骤。</p>
</div>

<div class="dev-onpage-video-wrapper"><a href="http://www.youtube.com/watch?v=wkqjafvvU5w" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/python/videos/django-tutorial-180x120.png') !important;" href="http://www.youtube.com/watch?v=wkqjafvvU5w" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">33:08</span></div>

</div>

本教程重点介绍 Python Tools for Visual Studio 和 Python Tools for Azure。有关在本教程中内置的 Django 和轮询应用程序的更详细信息，请参阅 <https://www.djangoproject.com/>。

[WACOM.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 要求

若要完成本教程，你将需要

-   [Python Tools 2.0 for Visual Studio][Python Tools 2.0 for Visual Studio]
-   [Python 2.7（32 位）][Python 2.7（32 位）]
-   Visual Studio 和 Azure SDK：
-   带有 Azure SDK 2.1 的 VS 2010 Pro 或更高版本
-   带有 Azure SDK 2.1、2.2 或更高版本的 VS 2012 Pro 或更高版本
-   带有 Azure SDK 2.2 或更高版本的 VS 2013 Pro 或更高版本
-   免费的 VS 2013 集成外壳。**Azure SDK 不支持集成外壳**。你可以使用免费的集成外壳在本地开发、调试和运行 Django 应用程序，但不能使用 Visual Studio 发布到 Azure 网站或云服务。

使用 Web 平台安装程序安装 Azure SDK。这将安装 SDK、模拟器以及 Azure Tools for Visual Studio。在 Web 平台安装程序中，搜索 **Azure SDK for .NET** 并且选择针对你的 Visual Studio 版本的 SDK 的支持版本之一。

**注意：**为了使用 Azure 网站或云服务成功运行此应用程序，你将需要使用来自 [python.org][Python 2.7（32 位）] 的正式 CPython 2.7 分发版本。其他分发版本可能能够使用，但不会受到官方支持。

## 下载现有项目

如果你想要跳到本教程的后面部分，你可以[下载此项目的源代码][下载此项目的源代码]。

Sqlite3 数据库已创建，使用的是下面的超级用户凭据：

    Username: tutorial
    Password: azure

下载不包括虚拟环境。你应按照[创建虚拟环境][创建虚拟环境]部分的步骤创建一个。完成此操作后，你的项目即可用于[调试][调试]部分。

## 创建项目

Python Tools for Visual Studio 支持 Python 虚拟环境。我们将创建一个 Django 项目并且使用虚拟环境安装我们的依赖项。这是用于设置发布到 Azure 网站或云服务的项目的建议方法。

1.  依次打开 Visual Studio、“文件”/“新建项目”，找到名称为“教程”的 Django 应用程序。

    ![新建项目][新建项目]

**注意：**在解决方案资源管理器中的“引用”下，你将看到针对 Django 1.4 的节点。该节点是用于 Azure 云服务部署的，用来在目标计算机上安装 Python 和 Django。不要从引用节点下删除对 Django 1.4 的引用。因为我们在使用虚拟环境并且在其中安装我们自己的 Django 包，所以，将使用安装在我们的虚拟环境中的 Django 包。

## <a name="creating-a-virtual-environment"></a>创建虚拟环境

我们将使用虚拟环境安装我们的依赖项。这是适合任何 Python 应用程序的好做法，在发布到 Azure 时是必需的。

1.  新建虚拟环境。在解决方案资源管理器中，右键单击“Python 环境”，然后选择“添加虚拟环境”。

    ![添加虚拟环境][添加虚拟环境]

2.  选择 Python 2.7 作为基础 Python 解释程序并且接受默认名称 **env**。如果你尚未安装，PTVS 将安装 pip 和/或 virtualenv。

3.  右键单击 **env** 和“安装 Python 包”：**django**

    ![安装 Django][安装 Django]

4.  Django 具有大量文件，因此将占用一些时间进行安装。可在输出窗口中查看安装进度。

    ![安装 Django 输出][安装 Django 输出]

    **注意：**在相当少见的情况下，你可能会在输出窗口中看到失败。如果出现此情况，请检查错误是否与清理相关。有时候清理将失败，但安装仍是成功的（在输出窗口中向上滚动将会确认这一点）。这是因为 PTVS 会获取一个针对新创建的临时文件/文件夹的锁，这将阻止 pip 清理步骤删除它们。

5.  右键单击 **env** 和“安装 Python 包”：**pytz**（可选但建议使用，django 使用它来提供时区支持）

## 验证虚拟环境

1.  我们必须确保所有内容正确安装。使用 **F5** 或 **CTRL+F5** 启动网站。这将启动 django 部署服务器并且启动你的 Web 浏览器。你应该看到以下页面：

    ![Django Web 浏览器][Django Web 浏览器]

## 创建轮询应用程序

在本节中，我们将添加一个应用程序以便处理轮询中的投票。

一个 Django 项目可以有多个应用程序。在本教程中，我们的项目名称是“教程”并且与 Visual Studio 项目相对应。我们正添加的应用程序的名称是“轮询”，并且将是我们的项目节点下的文件夹。

1.  依次选择“项目节点”、“添加-\>Django 应用程序”，名称为“轮询”。这将为该应用程序创建一个文件夹，其中包含针对常用应用程序文件的 boilerplate 代码。

    ![添加 Django 应用程序][添加 Django 应用程序]

2.  在 **tutorial/settings.py** 中，将以下内容添加到 **INSTALLED\_APPS**：

        'polls',

3.  并且取消对 **INSTALLED\_APPS** 的注释：

        'django.contrib.admin',

4.  使用以下代码替换 **tutorial/urls.py**：

        from django.conf.urls import patterns, include, url

        from django.contrib import admin
        admin.autodiscover()

        urlpatterns = patterns('',
            url(r'^', include('polls.urls', namespace="polls")),
            url(r'^admin/', include(admin.site.urls)),
        )

5.  使用以下代码替换 **polls/models.py**：

        from django.db import models

        class Poll(models.Model):
            question = models.CharField(max_length=200)
            pub_date = models.DateTimeField('date published')

            def __unicode__(self):
                return self.question

        class Choice(models.Model):
            poll = models.ForeignKey(Poll)
            choice_text = models.CharField(max_length=200)
            votes = models.IntegerField(default=0)

            def __unicode__(self):
                return self.choice_text

6.  使用以下代码替换 **polls/views.py**：

        from django.shortcuts import get_object_or_404, render
        from django.http import HttpResponseRedirect
        from django.core.urlresolvers import reverse
        from polls.models import Choice, Poll

        def vote(request, poll_id):
            p = get_object_or_404(Poll, pk=poll_id)
            try:
                selected_choice = p.choice_set.get(pk=request.POST['choice'])
            except (KeyError, Choice.DoesNotExist):
                # Redisplay the poll voting form.
                return render(request, 'polls/detail.html', {
                    'poll': p,
                    'error_message': "You didn't select a choice.",
                })
            else:
                selected_choice.votes += 1
                selected_choice.save()
                # Always return an HttpResponseRedirect after successfully dealing
                # with POST data. This prevents data from being posted twice if a
                # user hits the Back button.
                return HttpResponseRedirect(reverse('polls:results', args=(p.id,)))

7.  添加具有以下代码的新的 Python 文件 **polls/urls.py**：

        from django.conf.urls import patterns, url
        from django.views.generic import DetailView, ListView
        from polls.models import Poll

        urlpatterns = patterns('',
            url(r'^$',
                ListView.as_view(
                    queryset=Poll.objects.order_by('-pub_date')[:5],
                    context_object_name='latest_poll_list',
                    template_name='polls/index.html'),
                name='index'),
            url(r'^(?P<pk>\d+)/$',
                DetailView.as_view(
                    model=Poll,
                    template_name='polls/detail.html'),
                name='detail'),
            url(r'^(?P<pk>\d+)/results/$',
                DetailView.as_view(
                    model=Poll,
                    template_name='polls/results.html'),
                name='results'),
            url(r'^(?P<poll_id>\d+)/vote/$', 'polls.views.vote', name='vote'),
        )

8.  使用以下代码创建一个新项 **polls/admin.py**：

        from django.contrib import admin
        from polls.models import Choice, Poll

        class ChoiceInline(admin.TabularInline):
            model = Choice
            extra = 3

        class PollAdmin(admin.ModelAdmin):
            fieldsets = [
                (None,               {'fields': ['question']}),
                ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
            ]
            inlines = [ChoiceInline]
            list_display = ('question', 'pub_date')
            list_filter = ['pub_date']
            search_fields = ['question']
            date_hierarchy = 'pub_date'

        admin.site.register(Poll, PollAdmin)

9.  在 **polls/templates** 文件夹下，创建名为 **polls** 的一个新文件夹。

10. 使用拖放或剪切/粘贴将文件 **polls/templates/index.html** 移到 **polls/templates/polls** 文件夹下。

11. 使用以下标记替换 **polls/templates/polls/index.html**：

        <html>
        <head></head>
        <body>
        {% if latest_poll_list %}
            <ul>
            {% for poll in latest_poll_list %}
                <li><a href="{% url 'polls:detail' poll.id %}">{{ poll.question }}</a></li>
            {% endfor %}
            </ul>
        {% else %}
            <p>No polls are available.</p>
        {% endif %}
        </body>
        </html>

12. 使用以下代码创建新的 Django HTML 模板 **polls/templates/polls/detail.html**：

        <html>
        <head></head>
        <body>
        <h1>{{ poll.question }}</h1>
        {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
        <form action="{% url 'polls:vote' poll.id %}" method="post">
        {% csrf_token %}
        {% for choice in poll.choice_set.all %}
            <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
            <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
        {% endfor %}
        <input type="submit" value="Vote" />
        </form>
        </body>
        </html>

13. 使用以下代码创建新的 Django HTML 模板 **polls/templates/polls/results.html**：

        <html>
        <head></head>
        <body>
        <h1>{{ poll.question }}</h1>
        <ul>
        {% for choice in poll.choice_set.all %}
            <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
        {% endfor %}
        </ul>
        <a href="{% url 'polls:detail' poll.id %}">Vote again?</a>
        </body>
        </html>

14. 你现在应该在你的项目中拥有下列文件：

    ![解决方案资源管理器][解决方案资源管理器]

## 在本地创建 sqlite3 数据库

我们的 Web 应用程序几乎可供使用了，但我们首先需要配置数据库。为了在本地测试我们的网站，我们将创建 sqlite3 数据库。这是一种非常轻型的数据库，不要求任何附加安装。将在项目文件夹中创建数据库文件。

1.  在 **tutorial/settings.py** 中，将以下导入添加到该文件的顶部：

        from os import path

2.  在该文件的顶部旁在导入后添加以下定义：

        PROJECT_ROOT = path.dirname(path.abspath(path.dirname(__file__)))

3.  将 DATABASES 部分更改为以下代码：

        DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.sqlite3',
                'NAME': path.join(PROJECT_ROOT, 'db.sqlite3'),
                'USER': '',
                'PASSWORD': '',
                'HOST': '',
                'PORT': '',
            }
        }

4.  右键单击项目节点并且选择“Django-\>Django 同步数据库”。Django 管理交互式窗口将出现。因为该数据库尚不存在，系统将提示你创建管理员凭据。输入用户名和密码。电子邮件是可选的。

    ![Django 同步数据库][Django 同步数据库]

5.  使用 F5 或 CTRL-F5 启动网站。这将启动 django 部署服务器并且启动你的 Web 浏览器。该网站的根 url 将显示轮询索引，但在该数据库中尚没有任何索引。

    ![Web 浏览器][Web 浏览器]

6.  导航到 **http://localhost:{port}/admin**。你可以从开发服务器控制台窗口获取端口号。使用你在前面的步骤中创建的凭据登录。

    ![添加轮询][添加轮询]

7.  使用管理界面添加一个或两个轮询。不要将太多时间花在向本地数据库添加轮询上。我们在后面将切换到云数据库并且在那时将重新填充该数据库。

    ![轮询索引][轮询索引]

8.  导航到 **http://localhost:{port}/**。你将看到已添加的轮询的索引。

    ![][0]

9.  单击某一轮询可转到投票页。

    ![轮询详细信息][轮询详细信息]

10. 提交你的投票，并且你将被重定向到结果页，从中应该会看到投票计数递增。

    ![轮询结果][轮询结果]

## 使用样式表和其他静态文件

在本节中，我们将更新我们网站的外观以便使用样式表。将以不同方式处理静态文件（例如样式表），因此，将静态文件存储于正确的位置十分重要。

1.  在 **tutorial/settings.py** 中，找到 **STATIC\_ROOT** 的赋值并将其更改为：

        STATIC_ROOT = path.join(PROJECT_ROOT, 'static').replace('\','/')

2.  在 **polls** 文件夹下，创建名为 **static** 的一个新文件夹。

3.  在 **polls/static** 文件夹下，创建名为 **polls** 的一个新文件夹。

4.  在 **polls/static/polls** 文件夹下，创建名为 **images** 的一个新文件夹。

5.  添加具有以下代码的一个新样式表文件 **polls/static/polls/style.css**：

        body {
            color: darkblue;
            background: white url("images/background.jpg");
        }

6.  将一个现有图像添加到 **polls/static/polls/images** 文件夹中，并且将其命名为 **background.jpg**。

7.  你现在应该在你的项目中拥有下列文件：

    ![解决方案资源管理器][1]

8.  编辑所有模板的标头以便使用以下标记引用样式表：

        <head>
        {% load staticfiles %}
        <link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />
        </head>

9.  再次运行该网站。索引、轮询和结果页都将使用我们创建的样式表，并且具有深蓝色文本和背景图像。

    ![轮询详细信息][2]

## <a name="debugging"></a>调试

Python Tools for Visual Studio 具有用于调试 Django 模板的特别支持。

1.  打开 **polls/templates/polls/index.html** 并且使用 **F9** 在该行上放置一个断点：

        {% if latest_poll_list %}

2.  使用 **F5** 启动调试。Visual Studio 将在该模板文件上中断。

3.  从“调试-\>窗口-\>本地”打开“本地窗口”，你将会看到 **latest\_poll\_list** 变量和其值。

4.  你可以按 **F10**，像在常规 Python 代码中一样分步调试。在 for 循环内，你可以查看 **poll** 的值：

    ![调试][3]

## 在 Azure 上创建数据库

现在，我们已验证了我们的轮询应用程序可在本地工作，接下来我们将使用在 Azure 上托管的数据库。

在下面的两个部分中，我们将介绍如何使用 SQL 数据库和 MySQL 数据库。这两者都是托管服务。

另一个选项是创建虚拟机并安装数据库服务器。有关在 Azure Linux VM 上设置 MySQL 的说明，请参阅[此处][此处]。

**注意：**可以在 Azure 上使用 sqlite3 数据库（仅限开发目的，我们不建议在生产中使用它）。你将需要将 **db.sqlite3** 文件添加到你的项目中，以便将该数据库连同你的 django 应用程序一起部署。

### SQL Database

在本节中，我们将在 Azure 上创建一个 SQL 数据库，将必需的包添加到我们的虚拟环境中，以及更改我们的设置以便使用这个新数据库。

1.  在 Microsoft Azure 管理门户中，选择“SQL DATABASES”。

2.  首先创建用于托管该数据库的服务器。选择“服务器”和“添加”。

3.  在新创建的服务器的“配置”选项卡中，你将看到显示的当前客户端 IP 地址。在该地址旁边，单击“添加到允许的 IP 地址”。

    **注意：**有时候 Azure 不会正确检测到客户端 IP 地址。如果在同步数据库时你收到错误消息，则应从错误消息中复制/粘贴 IP 地址并且将其添加到允许的 IP 地址中。

4.  接下来，我们将创建数据库。在“数据库”选项卡中，从底部栏上单击“添加”。

5.  在 Visual Studio 中，我们将从 Django 访问 SQL Server 数据库所需的包安装到我们的虚拟环境中。

6.  右键单击 **env** 和“安装 Python 包”：使用 **easy\_install** 的 **pyodbc**。

7.  右键单击 **env** 和“安装 Python 包”：使用 **pip** 的 **django-pyodbc-azure**。

8.  编辑 **tutorial/settings.py** 并且将 **DATABASES** 定义更改为以下内容，将 **NAME**、**USER**、**PASSWORD** 和 **HOST** 替换为在 ClearDB 控制面板中列出的值：

        DATABASES = {
            'default': {
                'ENGINE': 'sql_server.pyodbc',
                'NAME': '<database name>',
                'USER': '<user name>@<server name>',
                'PASSWORD': '<user password>',
                'HOST': '<server name>.database.windows.net',
                'PORT': '',
                'OPTIONS': {
                    'driver': 'SQL Server Native Client 11.0',
                    'MARS_Connection': True,
                },
            }
        }

    你将需要确保使用已安装在你的计算机上的驱动程序。从开始菜单/屏幕打开“管理工具”，然后选择“ODBC 数据源(32 位)”。这些驱动程序将在“驱动程序”选项卡下列出。

    当在某一 Azure 网站上运行时，“SQL Server Native Client 10.0”和“SQL Server Native Client 11.0”都将起作用。

    当在某一 Azure 云服务上运行时，只有“SQL Server Native Client 11.0”起作用。

9.  同步数据库并且创建管理凭据，就像我们为本地 sqlite3 数据库做的一样。

### MySQL 数据库

在本节中，我们将在 Azure 上创建一个 MySQL 数据库，将必需的包添加到我们的虚拟环境中，以及更改我们的设置以便使用这个新数据库。

在 Azure 应用商店中，你可以将不同的服务（包括 MySQL 数据库）添加到你的帐户中。我们可以创建一个免费试用帐户，并且小数据库在某个大小以下是免费的。

1.  在 Azure 管理门户中，依次选择**“新建”**-\>**“应用商店”**-\>**“应用服务”**-\>**“ClearDB MySQL 数据库”**。创建具有免费计划的数据库。

2.  接下来，我们将从 Django 访问 MySQL 数据库所需的包安装到我们的虚拟环境中。右键单击 **env** 和“安装 Python 包”：使用 **easy\_install** 的 **mysql-python**。

3.  编辑 **tutorial/settings.py** 并且将 **DATABASES** 定义更改为以下内容，将 **NAME**、**USER**、**PASSWORD** 和 **HOST** 替换为在 ClearDB 控制面板中列出的值：

        DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.mysql',
                'NAME': '<database name>',
                'USER': '<user name>',
                'PASSWORD': '<user password>',
                'HOST': '<host url>',
                'PORT': '',
            }
        }

4.  同步数据库并且创建管理凭据，就像我们为本地 sqlite3 数据库做的一样。

## 使用 Django Shell

1.  右键单击项目节点并且选择“Django-\>打开 Django Shell”。

2.  在此交互式窗口中，我们可以使用我们的模型访问数据库。输入以下代码以便向该数据库添加轮询：

        from polls.models import Poll, Choice
        from django.utils import timezone

        p = Poll(question="Favorite way to host Django on Azure?", pub_date=timezone.now())
        p.save()

        p.choice_set.create(choice_text='Web Site', votes=0)
        p.choice_set.create(choice_text='Cloud Service', votes=0)
        p.choice_set.create(choice_text='Virtual Machine', votes=0)

    ![Django Shell 添加轮询][Django Shell 添加轮询]

3.  模型的静态分析提供完整 API 的有限视图。在交互式窗口中，你将获取针对活动对象的 IntelliSense，因此，这是浏览 API 的一个很棒的方法。下面是在交互式窗口中尝试执行的一些操作：

        # all poll objects
        Poll.objects.all()

        # primary key for poll
        p.id

        # all choice objects for the poll
        p.choice_set.all()

        # get object by primary key
        Poll.objects.get(pk=1)

    ![Django Shell 查询轮询][Django Shell 查询轮询]

4.  启动网站。你应该看到我们使用 Django shell 添加的轮询。

## 发布到 Azure

现在我们的数据库位于 Azure 上，接下来的步骤是在 Azure 上托管该网站本身。

Azure 有几个用于托管 Django 应用程序的选项：

-   [网站][网站]
-   [云服务][云服务]
-   [虚拟机][虚拟机]

Python Tools for Visual Studio 具有向 Azure 网站和云服务进行发布的功能。在接下来的两个部分中将对此进行说明，你可以选择只完成其中一个部分或者两个部分都完成。

在这两种情况下，PTVS 将负责为你配置 IIS，而如果你的项目中尚无 web.config 文件，你将需要生成该文件。只要你在 settings.py 中设置了 STATIC\_ROOT，就将自动收集静态文件 (manage.py collectstatic)。

在虚拟机中托管 Django 超出了本教程的论述范围。该过程涉及使用你期望的操作系统（Windows 或 Linux）创建虚拟机、安装 Python 以及手动部署 Django 应用程序。

### Azure 网站

1.  首先我们将需要创建网站。使用 Azure 管理门户，依次单击**“新建”**-\>**“计算”**-\>**“网站”**-\>**“快速创建”**。选择任何可用名称。

2.  在创建后，下载网站的发布配置文件。

    ![网站下载配置文件][网站下载配置文件]

3.  在 Visual Studio 中，右键单击项目节点，然后选择“发布”。

    ![网站发布][网站发布]

4.  导入你先前下载的网站发布配置文件。

5.  接受默认值，然后单击“发布”以便开始发布。

6.  在完成发布后，Web 浏览器将打开到已发布的网站。

    ![网站浏览器][网站浏览器]

### Azure 云服务

1.  右键单击项目节点并且选择“添加 Azure 云服务项目”或“转换 -\> 转换为 Azure 云服务项目”（根据你的 Visual Studio 版本，你看到的可能与此不同）。这将向解决方案添加一个新项目，该项目具有 .Azure 后缀。此新项目在解决方案中标记为启动项目。

    **注意：**如果你在项目上下文菜单中未看到用于创建 Azure 云服务项目的命令，则需要安装 Azure Tools for Visual Studio。它们作为 Azure SDK for .NET 的一部分安装。请参见本教程开始处的要求部分。

    ![解决方案资源管理器][4]

#### 在 Azure 模拟器中运行

1.  你将需要“以管理员身份重新启动 Visual Studio”，以便能够在计算模拟器中运行。

2.  使用 **F5** 开始调试，应用程序将在计算模拟器中运行和部署。请验证管理界面有效并且你可以对轮询投票。

    ![计算模拟器][计算模拟器]

3.  如果你不想继续以管理员身份运行，现在可以重新启动 Visual Studio。

#### 发布到 Azure 云服务

1.  接下来你将云服务发布到 Azure。右键单击 **tutorial.Azure** 云服务项目，然后选择“发布”。

    **注意：**请确保在云服务项目上选择“发布”。这将启动“发布 Azure 应用程序”对话框，以便发布到 Azure 云服务。如果你在 Django 项目上选择“发布”，将启动用于发布到 Azure 网站的“发布 Web”对话框。

    ![云服务发布][云服务发布]

2.  你将需要导入你的 Azure 订阅文件。单击[登录以下载凭据][登录以下载凭据]链接以便从 Azure 门户下载它。

3.  在“设置”页的云服务组合框中选择“新建”，以便创建一个新的云服务。可以使用可用的任何名称。

    ![云服务设置][云服务设置]

4.  接受默认设置，然后单击“发布”。此操作所用的时间将会比发布到网站长，因为它需要为云服务设置虚拟机。可在“Azure 活动日志”窗口中查看进度：

    ![云服务发布][5]

5.  在该操作完成后，在“Azure 活动日志”窗口中单击网站 URL 打开一个 Web 浏览器。

    ![云服务浏览器][云服务浏览器]

## 结束语

在本教程中，我们使用 [Python Tools for Visual Studio][Python Tools 2.0 for Visual Studio] 开发了一个 Django 应用程序。我们使用了 3 个不同的数据库：sqlite3、SQL Server 和 MySQL 数据库。最后，我们将应用程序发布到了 Azure 网站和云服务。

  [Python Tools 2.0 for Visual Studio]: http://pytools.codeplex.com
  [Python 2.7（32 位）]: http://www.python.org/download/
  [下载此项目的源代码]: http://download-codeplex.sec.s-msft.com/Download?ProjectName=pytools&DownloadId=783376
  [创建虚拟环境]: #creating-a-virtual-environment
  [调试]: #debugging
  [新建项目]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-001-new-project.png
  [添加虚拟环境]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-002-add-virtual-env.png
  [安装 Django]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-003-install-django.png
  [安装 Django 输出]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-004-install-django-output.png
  [Django Web 浏览器]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-004b-itworked.png
  [添加 Django 应用程序]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-005-add-django-app.png
  [解决方案资源管理器]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-006-solution-explorer.png
  [Django 同步数据库]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-007-sqlite3.png
  [Web 浏览器]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-008-dev-server.png
  [添加轮询]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-009-admin-login.png
  [轮询索引]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-009-admin-add-poll.png
  [0]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-010-index.png
  [轮询详细信息]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-011-detail.png
  [轮询结果]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-012-results.png
  [1]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-013-solution-explorer.png
  [2]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-014-detail-styled.png
  [3]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-015-debugging.png
  [此处]: http://www.windowsazure.com/zh-cn/manage/linux/common-tasks/mysql-on-a-linux-vm/
  [Django Shell 添加轮询]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-018-shell-add-poll.png
  [Django Shell 查询轮询]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-019-shell-query.png
  [网站]: http://www.windowsazure.com/zh-cn/services/web-sites/
  [云服务]: http://www.windowsazure.com/zh-cn/services/cloud-services/
  [虚拟机]: http://www.windowsazure.com/zh-cn/services/virtual-machines/
  [网站下载配置文件]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-020-website-download-profile.png
  [网站发布]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-020-website-publish.png
  [网站浏览器]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-020-website.png
  [4]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-021-cloudservice-solution-explorer.png
  [计算模拟器]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-021-cloudservice-emulator.png
  [云服务发布]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-021-cloudservice-publish.png
  [登录以下载凭据]: https://manage.windowsazure.cn/publishsettings/index?client=vs&schemaversion=2.0
  [云服务设置]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-021-cloudservice-settings.png
  [5]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-021-cloudservice-publish-progress.png
  [云服务浏览器]: ./media/cloud-services-python-create-deploy-django-app/django-tutorial-021-cloudservice.png
