

 打开项目文件 QSTodoListViewController.m，并在 **viewDidLoad** 方法中，删除用于将数据重新上载到表的以下代码：

```
        - (void) loginAndGetData
        {
            MSClient *client = self.todoService.client;
            
            if (client.currentUser != nil) {
                return;
            }
            
            [client loginWithProvider:@"facebook" controller:self animated:YES completion:^(MSUser *user, NSError *error) {
                [self refresh];
            }];
        }
```

* 将 `viewDidLoad` 中的 `[self refresh]` 替换为以下代码：

```
        [self loginAndGetData];
```

* 若要启动该应用程序，请按“运行”，然后再登录。当你登录时，你应能够查看 Todo 列表并进行更新。

<!---HONumber=74-->