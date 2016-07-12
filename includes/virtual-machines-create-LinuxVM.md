1. 使用[从 Azure CLI 连接到 Azure](/documentation/articles/xplat-cli-connect/) 中列出的步骤登录到你的 Azure 订阅。

2. 请使用以下命令你确保处于服务管理模式下：

        azure config mode asm

3. 从可用映像中找出你想要加载的 Linux 映像：

        azure vm image list | grep "Linux"

   在 Windows 命令提示符窗口中，使用 find 而不是 grep。

4. 利用上述列表中的 Linux 映像，使用 `azure vm create` 创建新的虚拟机。此步骤将创建新的云服务以及新的存储帐户。你还可通过 `-c` 选项将此虚拟机连接到现有云服务。其还通过 `-e` 选项创建 SSH 终结点以登录到 Linux 虚拟机。

        ~$ azure vm create "MyTestVM" 250d269906be4694a10aee49a3385f2d__suse-opensuse-13.2-v20160302 "adminUser" -z "Small" -e -l "China North"
        info:    Executing command vm create
        + Looking up image 250d269906be4694a10aee49a3385f2d__suse-opensuse-13.2-v20160302
        Enter VM 'adminUser' password:*********
        Confirm password: *********
        + Looking up cloud service
        info:    cloud service MyTestVM not found.
        + Creating cloud service
        + Retrieving storage accounts
        + Creating a new storage account 'mytestvm1437604756125'
        + Creating VM
        info:    vm create command OK

    >[AZURE.NOTE]对于 Linux 虚拟机，你必须提供在 `vm create` 中提供 `-e` 选项；不能在创建虚拟机后启用 SSH。有关 SSH 的详细信息，请参阅[如何在 Azure 中将 SSH 用于 Linux](/documentation/articles/virtual-machines-linux-ssh-from-linux/)。

    请注意，映像 *250d269906be4694a10aee49a3385f2d__suse-opensuse-13.2-v20160302* 是我们在上述步骤中从映像列表中选择的映像。*MyTestVM* 是我们的新虚拟机的名称，*adminUser* 是我们将 SSH 用于虚拟机的用户名。你可以根据你的要求来替换这些变量。有关此命令的更多详细信息，请访问[使用 Azure 服务管理的 Azure CLI](/documentation/articles/virtual-machines-command-line-tools/)。

5. 新创建的 Linux 虚拟机将显示在由以下命令生成的列表中：

        azure vm list

6. 可以通过使用以下命令来验证虚拟机的属性：

        azure vm show MyTestVM

7. 使用 `azure vm start` 命令即可启动新创建的虚拟机。

有关所有这些 Azure CLI 虚拟机命令的详细信息，请参阅[使用服务管理 API 的 Azure CLI](/documentation/articles/virtual-machines-command-line-tools/)。

<!---HONumber=Mooncake_1221_2015-->