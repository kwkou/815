<properties linkid="develop-dotnet-website-with-webmatrix" urlDisplayName="Website with WebMatrix" pageTitle=".NET web site with WebMatrix - Azure tutorials" metaKeywords="WebMatrix Azure, WebMatrix Azure, Azure web site WebMatrix, Azure website WebMatrix, Web Matrix Azure, WebMatrix Azure" description="Learn how to develop and deploy an Azure web site with WebMatrix." metaCanonical="" services="web-sites" documentationCenter=".NET" title="Develop and deploy a web site with Microsoft WebMatrix" authors="" solutions="" manager="" editor="" />

# 使用 Microsoft WebMatrix 开发和部署网站

本指南介绍如何使用 Microsoft WebMatrix 创建网站并将其部署到 Azure。你将使用 WebMatrix 网站模板中的一个示例应用程序。

你将了解到以下内容：

-   如何在 WebMatrix 中登录 Azure
-   如何将内置模板和 WebMatrix 配合使用来创建网站
-   如何将自定义的网站直接从 WebMatrix 中部署到 Azure。

[WACOM.INCLUDE [create-account-and-websites-note][create-account-and-websites-note]]

## 登录 Azure

1.  启动 WebMatrix。
2.  如果这是你首次使用 WebMatrix 3，系统将会提示你登录 Azure。否则，你可以单击“登录”按钮，然后选择“添加帐户”。选择使用你的 Microsoft 帐户“登录”。

    ![添加帐户][添加帐户]

3.  如果你已注册某一 Azure 帐户，则可以使用你的 Microsoft 帐户登录：

    ![登录 Azure][登录 Azure]

## 使用 Azure 的内置模板创建网站

1.  在开始屏幕上，单击“新建”按钮，然后选择“模板库”以便从模板库创建一个新网站：

    ![从模板库创建新网站][从模板库创建新网站]

2.  模板库将显示可本地运行或在 Azure 上运行的可用模板的列表。从模板列表中选择“面包店”，输入“bakerysample”作为“网站名称”，然后单击“下一步”。

    ![从模板创建网站][从模板创建网站]

3.  如果你已登录到 Azure 中，现在将具有该选项以便为你的本地网站创建一个 Azure 网站。选择一个独有名称，然后选择你希望要创建你的网站的数据中心：

    ![在 Azure 中创建网站][在 Azure 中创建网站]

    在 WebMatrix 生成网站完毕后，将显示 WebMatrix IDE：

    ![WebMatrix IDE][WebMatrix IDE]

## 测试网站

面包店示例中有一个模拟的订单窗体，该窗体向你提供的 Windows Live Hotmail 帐户发送一封电子邮件及所订购的商品。

1.  在 WebMatrix 的左侧导航窗格中，展开“面包店示例”文件夹。

    ![][]

2.  通过双击文件名打开 *Order.cshtml* 页。

    ![][1]

3.  查找内容为“//SMTP Configuration for Hotmail”的注释。

    ![][2]

4.  将以下行中的值更改为你自己的电子邮件提供商：

        WebMail.SmtpServer = "smtp.live.com";
        WebMail.SmtpPort  = 25;
        WebMail.EnableSsl = true; 

        //Enter your Hotmail credentials for UserName/Password and a "From" address for the e-mail
        WebMail.UserName = "";
        WebMail.Password = "";
        WebMail.From = "";

    将 WebMail.SmtpServer 的值更改为你在正常情况下用于发送电子邮件的电子邮件服务器的名称。然后，填写用户名和密码的值。将 From 属性设置为你的电子邮件地址。

5.  在 WebMatrix 功能区上，单击“运行”以测试网站。

    ![][3]

6.  在某个产品上单击“立即订购”，然后将订单发给你自己。

7.  检查你的电子邮件，确保获得了订单确认。如果发送电子邮件遇到困难，请参阅《ASP.NET 网页 (Razor) 故障排除指南》中的[有关发送电子邮件的问题][有关发送电子邮件的问题]。

## 将自定义的网站从 WebMatrix 部署到 Azure

1.  在 WebMatrix 中，从“主页”(Home) 功能区单击“发布”(Publish)，为网站显示“发布预览”(Publish Preview) 对话框。

    ![WebMatrix 发布预览][WebMatrix 发布预览]

2.  单击以选中 bakery.sdf 旁的复选框，然后单击“继续”。发布完毕后，将在 WebMatrix IDE 底部显示 Azure 上更新网站的 URL。

    ![发布完毕][发布完毕]

3.  单击该链接以在浏览器中打开该网站：

    ![面包店示例网站][面包店示例网站]

    还可通过单击“网站”以显示订阅的所有网站，在 Azure 门户中找到网站的 URL。每个网站的 URL 显示在“网站”页上的“URL”列中。

## 修改该网站并将其重新发布到 Azure 网站

可使用 WebMatrix 修改该网站并将其重新发布到 Azure 网站。在以下过程中，你将添加一个复选框以指示订单是赠品。

1.  打开 *Order.cshtml* 页。

2.  找到“shiping”类窗体定义。紧接 \<li\> 块后插入以下代码。

        <li class="gift">
            <div class="fieldcontainer" data-role="fieldcontain">
                <label for="isGift">This is a gift</label>           
                <div>@Html.CheckBox("isGift")</div>
            </div>
        </li>

    ![][4]

3.  在文件中找到“var shipping = Request["orderShipping"];”行，然后紧接其后插入下面这行代码。

        var gift = Request["isGift"];

4.  在以下代码段中紧接确认配送地址不为空的代码块后。

        if(gift != null) {
            body += "This is a gift." + "<br/>";
        }

    你的 *order.cshtml* 文件应与下图类似。

    ![][5]

5.  保存该文件并在本地运行网站，然后向你自己发送一个测试订单。确保测试这个新复选框。

    ![][6]

6.  通过在“主页”功能区上单击“发布”，重新发布该网站。

7.  在“发布预览”对话框中，确保选中 Order.cshtml 并单击“继续”。

8.  单击该链接以在浏览器中打开网站并测试 Azure 网站上的更新。

# 后续步骤

你已了解如何创建网站并将网站从 WebMatrix 部署到 Azure。若要了解有关 WebMatrix 的详细信息，请参阅下列资源：

-   [WebMatrix for Azure][WebMatrix for Azure]

-   [WebMatrix 网站][WebMatrix 网站]

  [create-account-and-websites-note]: ../includes/create-account-and-websites-note.md
  [添加帐户]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-add-account.png
  [登录 Azure]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-sign-in.png
  [从模板库创建新网站]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-site-from-template.png
  [从模板创建网站]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-site-from-template-details.png
  [在 Azure 中创建网站]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-site-from-template-azure.png
  [WebMatrix IDE]: ./media/web-sites-dotnet-using-webmatrix/howtowebmatrixide.png
  []: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-1.png
  [1]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-2.png
  [2]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-3.png
  [3]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-4.png
  [有关发送电子邮件的问题]: http://go.microsoft.com/fwlink/?LinkId=253001#email
  [WebMatrix 发布预览]: ./media/web-sites-dotnet-using-webmatrix/howtopublishpreview.png
  [发布完毕]: ./media/web-sites-dotnet-using-webmatrix/howtopublished2.png
  [面包店示例网站]: ./media/web-sites-dotnet-using-webmatrix/howtobakerysamplesite.png
  [4]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-5.png
  [5]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-6.png
  [6]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-7.png
  [WebMatrix for Azure]: http://go.microsoft.com/fwlink/?LinkID=253622&clcid=0x409
  [WebMatrix 网站]: http://www.microsoft.com/click/services/Redirect2.ashx?CR_CC=200106398
