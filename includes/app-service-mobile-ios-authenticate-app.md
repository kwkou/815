**Objective-C**：

1. 在 Mac 的 Xcode 中打开 _QSTodoListViewController.m_ 并添加以下方法。若未使用 microsoftaccount 作为标识提供者，请将 _microsoftaccount_ 更改为  _windowsazureactivedirectory_ 。

            - (void) loginAndGetData
            {
                MSClient *client = self.todoService.client;
                if (client.currentUser != nil) {
                    return;
                }
            
                [client loginWithProvider:@"microsoftaccount" controller:self animated:YES completion:^(MSUser *user, NSError *error) {
                    [self refresh];
                }];
            }


2. 按以下方式替换 _QSTodoListViewController.m_ 的 `viewDidLoad` 中的 `[self refresh]`：

        [self loginAndGetData];

3. 若要启动该应用程序，请按“运行”，然后再登录。当你登录时，你应能够查看 Todo 列表并进行更新。

**Swift**：

1. 在 Mac 的 Xcode 中打开 _ToDoTableViewController.swift_ 并添加以下方法。若未使用 microsoftaccount 作为标识提供者，请将 _microsoftaccount_ 更改为 _windowsazureactivedirectory_ 。
        
            func loginAndGetData() {
                
                guard let client = self.table?.client where client.currentUser == nil else {
                    return
                }
                
                client.loginWithProvider("microsoftaccount", controller: self, animated: true) { (user, error) in
                    self.refreshControl?.beginRefreshing()
                    self.onRefresh(self.refreshControl)
                }
            }


2. 删除 _ToDoTableViewController.swift_ 中 `viewDidLoad()` 末尾的 `self.refreshControl?.beginRefreshing()` 和 `self.onRefresh(self.refreshControl)` 行。在其位置上添加对 `loginAndGetData()` 的调用：

        loginAndGetData()

3. 若要启动该应用程序，请按“运行”，然后再登录。当你登录时，你应能够查看 Todo 列表并进行更新。

<!---HONumber=Mooncake_0919_2016-->