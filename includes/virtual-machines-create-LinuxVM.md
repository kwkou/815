<!-- need to be verified -->


1. 使用[从 Azure CLI 连接到 Azure](/documentation/articles/xplat-cli-connect/)中列出的步骤登录到 Azure 订阅。

2. 确保使用经典部署模式，如下所示：

    ```azurecli
    azure config mode asm
    ```

3. 从可用映像中找出你想要加载的 Linux 映像，如下所示：

   ```azurecli   
    azure vm image list | grep "Linux"
    ```
   
    In a Windows command-prompt window, use **find** instead of grep.
   
4. 使用 `azure vm create`通过上一列表中的 Linux 映像创建 VM。此步骤将创建云服务和存储帐户。你还可通过 `-c` 选项将此 VM 连接到现有云服务。使用 `-e` 选项创建 SSH 终结点以登录到 Linux 虚拟机。以下示例使用位于 `China North` 的 `Ubuntu-14_04_3-LTS` 映像创建名为 `myVM` 的 VM，并添加用户名 `ops`：
   
    ```azurecli
    azure vm create myVM \
        b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20160516-en-us-30GB \
        -g ops -p P@ssw0rd! -z "Small" -e -l "China North"
    ```

    输出类似于以下示例：

    ```azurecli
    info:    Executing command vm create
    + Looking up image b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20160516-en-us-30GB
    + Looking up cloud service
    info:    cloud service myVM not found.
    + Creating cloud service
    + Retrieving storage accounts
    + Creating VM
    info:    vm create command OK
    ```
   
   > [AZURE.NOTE]
   对于 Linux 虚拟机，必须在 `vm create` 中提供 `-e` 选项。在创建虚拟机后便无法启用 SSH。有关 SSH 的详细信息，请参阅[如何在 Azure 中将 SSH 用于 Linux](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)。

5. 可以通过使用 `azure vm show` 命令验证 VM 的属性。以下示例列出名为 `myVM` 的 VM 的信息：

    ```azurecli   
    azure vm show myVM
    ```

6. 使用 `azure vm start` 命令启动 VM，如下所示：

    ```azurecli
    azure vm start myVM
    ```

## 后续步骤
有关所有这些 Azure CLI 虚拟机命令的详细信息，请参阅[使用经典部署 API 的 Azure CLI](/documentation/articles/virtual-machines-command-line-tools/)。

<!---HONumber=Mooncake_1212_2016-->