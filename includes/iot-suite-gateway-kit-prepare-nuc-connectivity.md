## <a name="prepare-your-intel-nuc"></a>准备 Intel NUC

若要完成硬件设置，需执行以下操作：

- 将 Intel NUC 连接到工具包中提供的电源。
- 使用以太网电缆将 Intel NUC 连接到网络。

现在已完成 Intel NUC 网关设备的硬件设置。

### <a name="sign-in-and-access-the-terminal"></a>登录并访问终端

可以通过两个选项来访问 Intel NUC 上的终端环境：

- 如果已将键盘和显示器连接到 Intel NUC，则可直接访问 shell。 默认凭据是用户名 **root** 和密码 **root**。

- 从台式机使用 SSH 访问 Intel NUC 上的 shell。

#### <a name="sign-in-with-ssh"></a>使用 SSH 登录

若要使用 SSH 进行登录，需要 Intel NUC 的 IP 地址。 如果已将键盘和显示器连接到 Intel NUC，可以使用 `ifconfig` 命令来查找 IP 地址。 或者，连接到路由器以列出网络上设备的地址。

使用用户名 **root** 和密码 **root** 登录。

#### <a name="optional-share-a-folder-on-your-intel-nuc"></a>可选：共享 Intel NUC 上的文件夹

（可选）可以将 Intel NUC 上的文件夹与桌面环境共享。 共享文件夹之后，即可使用偏好的桌面文本编辑器（例如 [Visual Studio Code](https://code.visualstudio.com/) 或 [Sublime Text](http://www.sublimetext.com/)）在 Intel NUC 上编辑文件，而不必使用 `nano` 或 `vi`。

若要与 Windows 共享文件夹，请在 Intel NUC 上配置 Samba 服务器。 也可将 Intel NUC 上的 SFTP 服务器与台式机上的 SFTP 客户端配合使用。