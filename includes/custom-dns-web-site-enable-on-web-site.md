在传播域名记录后，必须将其与 Web 应用关联。通过以下步骤使用 Web 浏览器启用域名。

> [AZURE.NOTE] 在之前的步骤中创建的 TXT 记录通过 DNS 系统向外传播可能需要一段时间。在 TXT 记录传播之前，无法将域名添加到 Web 应用。如果使用的是 A 记录，则上一步创建的 TXT 记录传播后才可将 A 记录域名添加到 Web 应用。
>
> 可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 等服务验证该 TXT 记录是否可用。

1. 在浏览器中，打开 [Azure 门户](https://portal.azure.cn)。

2. 在“Web 应用”选项卡中，单击 Web 应用的名称，然后选择“自定义域”

	![](./media/custom-dns-web-site/dncmntask-cname-6.png)

3. 在“自定义域”边栏选项卡上，单击“添加主机名”。
	
4. 使用“主机名”文本框输入要与此 Web 应用相关联的域名。

	![](./media/custom-dns-web-site/add-custom-domain.png)

6.  单击“验证”。

7.  单击“验证”时，Azure 将启动域验证工作流。这将检查域的所有权和主机名的可用性，并报告成功或错误详情（附带解决错误的说明性指南）。

此时，你应该可以在浏览器中输入自定义域名，并查看它是否成功地将你转至 Web 应用。

<!---HONumber=Mooncake_0926_2016-->