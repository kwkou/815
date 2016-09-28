在传播域名记录后，必须将其与 Web 应用关联。通过以下步骤使用 Web 浏览器启用域名。

> [AZURE.NOTE] 在之前的步骤中创建的 CNAME 记录通过 DNS 系统向外传播可能需要一段时间。在 CNAME 传播之前，无法将域名添加到 Web 应用。如果使用的是 A 记录，则在上一步中创建了 **awverify** CNAME 传播后，才可将 A 记录域名添加到 Web 应用。
>
> 可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 等服务验证该 CNAME 是否可用。

1. 在浏览器中，打开 [Azure 门户预览](https://portal.azure.cn)。

2. 在“Web 应用”选项卡中，单击 Web 应用的名称，选择“设置”，然后选择“自定义域和 SSL”。

	![](./media/custom-dns-web-site/dncmntask-cname-6.png)

3. 在“自定义域和 SSL”边栏选项卡中，单击“自带外部域”。

	![](./media/custom-dns-web-site/dncmntask-cname-7.png)

4. 使用“域名”文本框输入要与此 Web 应用相关联的域名。

	![](./media/custom-dns-web-site/dncmntask-cname-8.png)

5. 单击“保存”以保存域名配置。

	完成配置后，自定义域名将在 Web 应用的“域名”部分列出。

此时，你应该可以在浏览器中输入自定义域名，并查看它是否成功地将你转至 Web 应用。

<!---HONumber=Mooncake_0919_2016-->