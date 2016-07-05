如果需要域，你可以直接在 [Azure 经典管理门户](https://manage.windowsazure.cn)上购买域。请使用以下步骤来购买域名，然后将其分配给 Web 应用。

1. 在你的浏览器中，打开 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2. 在“Web 应用”选项卡中，单击 Web 应用的名称，选择“设置”，然后选择“自定义域和 SSL”。

	![](./media/custom-dns-web-site/dncmntask-cname-6.png)

3. 在“自定义域和 SSL”边栏选项卡中，单击“购买域”。

	![](./media/custom-dns-web-site/dncmntask-cname-buydomains-1.png)

4. 在“购买域”边栏选项卡中，使用文本框输入你想要购买的域名。将直接在文本框下方显示建议的可用域。选择你想要购买什么域。

  ![](./media/custom-dns-web-site/dncmntask-cname-buydomains-2.png)

5. 单击“联系人信息”，然后填充域的联系人信息表单。

  ![](./media/custom-dns-web-site/dncmntask-cname-buydomains-3.png)

6. 单击“购买域”边栏选项卡上的“选择”，然后你就会在“购买确认”边栏选项卡上看到购买信息。如果你接受法律条款，请单击“购买”，然后就会提交你的订单，你可以在“通知”中监视购买流程。

  ![](./media/custom-dns-web-site/dncmntask-cname-buydomains-4.png)

  ![](./media/custom-dns-web-site/dncmntask-cname-buydomains-5.png)

7. 如果你成功订购了域，则可管理该域并将其分配给你的 Web 应用。单击域右侧的“...”。然后，你可以**删除**、**取消**或**管理主机名**。单击“管理主机名”，然后，我们就可以在“主机名绑定”边栏选项卡中将子域绑定到我们的 Web 应用。

	![](./media/custom-dns-web-site/dncmntask-cname-buydomains-6.png)

	配置完成后，自定义域名将会列在 Web 应用的“分配到站点的主机名”部分。

此时，你应该可以在浏览器中输入自定义域名，并查看它是否成功地将你转至 Web 应用。

<!---HONumber=64-->