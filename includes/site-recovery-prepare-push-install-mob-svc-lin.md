### 准备在 Linux 服务器上执行推送安装

1. 确保 Linux 计算机与进程服务器之间已建立网络连接。
2. 创建可供进程服务器用来访问计算机的帐户。（该帐户应是源 Linux 服务器上的 **root** 用户，仅用于推送安装/更新）。
3. 确保源 Linux 服务器上的 `/etc/hosts` 文件包含用于将本地主机名映射到所有网络适配器关联的 IP 地址的条目。
4. 在要复制的计算机上安装最新的 openssh、openssh-server 和 openssl 包。
5. 确保 SSH 已启用且正在端口 22 上运行。
6. 在 sshd\_config 文件中启用 SFTP 子系统与密码身份验证，如下所示：
  - 以 **root** 身份登录。
  - 在文件 /etc/ssh/sshd\_config 中，找到以 **PasswordAuthentication** 开头的行。
  - 取消注释该行，并将值从“no”更改为“yes”。
  6.4 找到以“Subsystem”开头的行，并取消注释该行。

     ![Linux](./media/site-recovery-prepare-push-install-mob-svc-lin/mobility2.png)  


7. 添加在 CSPSConfigtool 中创建的帐户。

    - 登录到配置服务器。
    - 打开 **cspsconfigtool.exe**。（桌面上有该工具的快捷方式，也可以在 %ProgramData%\\home\\svsystems\\bin 文件夹中找到它）
    - 在“管理帐户”选项卡中，单击“添加帐户”。
    - 添加已创建的帐户。添加帐户后，在为计算机启用复制时，需要提供这些凭据。

<!---HONumber=Mooncake_0327_2017-->