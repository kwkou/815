

1. 打开项目文件 QSTodoListViewController.m，并在 **viewDidLoad** 方法中，删除用于将数据重新上载到表的以下代码：

        [self refresh];

2.	在 **viewDidLoad** 方法后，添加以下代码：  

        - (void)viewDidAppear:(BOOL)animated
        {
            MSClient *client = self.todoService.client;
            
            if (client.currentUser != nil) {
                return;
            }
            
            [client loginWithProvider:@"facebook" controller:self animated:YES completion:^(MSUser *user, NSError *error) {
                [self refresh];
            }];
        }

    <div class="dev-callout"><b>注意</b>
	<p>如果使用的标识提供程序不是 Facebook，请将传递给上述 <strong>loginWithProvider</strong> 的值更改为下列其中一项：<em>microsoftaccount</em>、<em>facebook</em>、<em>twitter</em>、<em>google</em> 或 <em>windowsazureactivedirectory</em>。</p>
    </div>
		
3. 按"运行"按钮以生成项目，在 iPhone 模拟器中启动应用，然后使用您选择的标识提供程序登录。

   	当您成功登录时，应用应该运行而不出现错误，您应该能够查询移动服务，并对数据进行更新。
