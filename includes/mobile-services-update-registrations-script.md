

1. 在 [Azure 经典门户](https://manage.windowsazure.cn/)中，单击“数据”选项卡，然后单击“Registrations”表。 

	![](./media/mobile-services-update-registrations-script/mobile-portal-data-tables-registrations.png)

2. 在“Registrations”中，单击“脚本”选项卡，然后选择“插入”。
   
	![](./media/mobile-services-update-registrations-script/mobile-insert-script-registrations.png)

	将显示当 **Registrations** 表中发生插入时所调用的函数。

3. 将 insert 函数替换为以下代码，然后单击“保存”：

		function insert(item, user, request) {
			var registrationTable = tables.getTable('Registrations');
			registrationTable
				.where({ handle: item.handle })
				.read({ success: insertChannelIfNotFound });
	        function insertChannelIfNotFound(existingRegistrations) {
        	    if (existingRegistrations.length > 0) {
            	    request.respond(200, existingRegistrations[0]);
        	    } else {
            	    request.execute();
        	    }
    	    }
	    }

   这样可以注册一个新的插入脚本，该脚本将注册信息存储在新表中。

<!---HONumber=Mooncake_0118_2016-->