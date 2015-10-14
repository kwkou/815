
接下来，需要更改您注册接收通知的时间，以确保尝试注册之前用户已经过身份验证。


1. 在 Eclipse 中的 Package Explorer 内，打开 ToDoActivity.java 文件，并找到  `onCreate` 方法。将以下代码从  `onCreate` 方法移到  `createTable` 方法的开头。

        NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);

      `authenticate` 方法完成时，将调用  `createTable` 方法。整个  `createTable` 方法应类似以下内容。

        private void createTable() {
        
            NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);
        
            // Get the Mobile Service Table instance to use
            mToDoTable = mClient.getTable(ToDoItem.class);
            
            mTextNewToDo = (EditText) findViewById(R.id.textNewToDo);
            
            // Create an adapter to bind the items with the view
            mAdapter = new ToDoItemAdapter(this, R.layout.row_list_to_do);
            ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
            listViewToDo.setAdapter(mAdapter);
            
            // Load the items from the Mobile Service
            refreshItemsFromTable();
        }	

<!---HONumber=71-->