

在 Azure 中使用映像来提供包含操作系统的新虚拟机。一个映像也可能包含一个或多个数据磁盘。映像可从多个源获取：

-	Azure 在应用商店和库中提供映像。可以找到最新的 Windows Server 和 Linux 操作系统分发版。一些映像还包含应用程序，如 SQL Server。MSDN 权益和 MSDN 即付即用订户有权访问其他映像。
-	开源社区通过 VM 仓库提供映像。
-	你还可以通过捕获现有的 Azure 虚拟机用作映像或上载映像来在 Azure 中存储和使用自己的映像。

## 关于 VM 映像和 OS 映像

可以在 Azure 中使用两种类型的映像：*VM 映像*和 *OS 映像*。VM 映像包括在创建该映像时附加到虚拟机的操作系统和所有磁盘。VM 映像是较新类型的映像。在引入 VM 映像之前，Azure 中的映像只能包含一个通用操作系统，而不能包含任何附加磁盘。只包含一个通用操作系统的 VM 映像基本上与原始类型的映像（即，OS 映像）相同。

你可以基于 Azure 中的虚拟机或复制和上载的在别处运行的虚拟机，创建你自己的映像。如果要使用映像来创建多个虚拟机，则需要通用化该虚拟机以将其用作映像。若要创建 Windows Server 映像，请在上载 .vhd 文件之前在服务器上运行 Sysprep 命令对其进行通用化。有关 Sysprep 的详细信息，请参阅 [How to Use Sysprep: An Introduction](https://technet.microsoft.com/zh-cn/library/bb457073.aspx)（如何使用 Sysprep：简介）和 [Sysprep Support for Server Roles](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)（服务器角色的 Sysprep 支持）。在运行 Sysprep 之前备份 VM。创建 Linux 映像的过程因分发版而异。通常，需要运行一组特定于分发版的命令，以及运行 Azure Linux 代理。

<!---HONumber=Mooncake_1010_2016-->