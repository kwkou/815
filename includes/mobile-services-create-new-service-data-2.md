为了能够在新移动服务中存储应用数据，必须先在关联的 SQL 数据库实例中创建一个新表。

1. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 单击“数据”选项卡，然后单击“+创建”。

   	此时将显示“创建新表”对话框。

3. 在“表名”中键入 _TodoItem_，然后单击勾选按钮。

	此时将创建一个设置了默认权限的新存储表 **TodoItem**。这意味着，具有应用程序密钥（随应用一起分发）的任何用户都可以访问和更改表中的数据。

	>[AZURE.NOTE] 移动服务快速入门中使用了相同的表名。 但是，每个表是在特定于给定移动服务的架构中创建的。 这是为了防止当多个移动服务使用同一数据库时发生数据冲突。

4. 单击新的 **TodoItem** 表，然后验证是否不存在任何数据行。

5. 单击“列”选项卡。验证系统是否已自动为您创建了以下默认列：
	
	<table border="1" cellpadding="10">
<tr>
<th>列名</th>
<th>类型</th>
<th>索引</th>
</tr>
<tr>
<td>id</td>
<td>字符串</td>
<td>已编入索引</td>
</tr>
<tr>
<td>__createdAt</td>
<td>日期</td>
<td>已编入索引</td>
</tr>
<tr>
<td>__updatedAt</td>
<td>日期</td>
<td><font color="transparent">-</font></td>
</tr>
<tr>
<td>__version</td>
<td>时间戳 (MSSQL)</td>
<td><font color="transparent">-</font></td>
</tr> 	
</table> 	
这是对移动服务中的表的最低要求。

>[AZURE.NOTE]如果在移动服务中启用了动态架构，则通过插入或更新操作向移动服务发送 JSON 对象时，将自动创建新列。

现在，您可以将新移动服务用作应用的数据存储。

<!---HONumber=Mooncake_0118_2016-->