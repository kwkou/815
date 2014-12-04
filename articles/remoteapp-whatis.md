<properties title="什么是 RemoteApp？" pageTitle="什么是 RemoteApp？" description="了解 RemoteApp。" metaKeywords="" services="" solutions="" documentationCenter="" authors="elizapo"  />

# 什么是 Azure RemoteApp？

RemoteApp 使您可以使通过 Azure 进行远程访问的程序，看起来就好像它们是在最终用户的本地计算机上运行一样。这些程序称为 RemoteApp 程序。RemoteApp 程序与客户端的桌面集成，而不是提供给远程桌面会话主机（RD 会话主机）服务器的桌面中的用户。RemoteApp 程序在其自己可调整大小的窗口中运行、可以在多个监视器之间拖动并在任务栏中有其自己的条目。如果用户在相同的 RD 会话主机服务器上运行多个 RemoteApp 程序，则 RemoteApp 程序将共享同一个远程桌面服务会话。

RemoteApp 在许多情况下可以降低复杂性并减少管理开销，其中包括：

-   分支机构，其中可能有有限的本地 IT 支持和有限的网络带宽。
-   用户需要远程访问程序的情况。
-   业务线 (LOB) 程序的部署，尤其是自定义 LOB 程序。
-   在某些环境中（如“公用办公桌”或“旅馆式办公”工作区），没有给用户分配计算机。
-   多个版本的程序的部署，特别是当本地安装多个版本时，可能会造成冲突。

与传统的远程桌面服务不同，Azure RemoteApp 在 Azure 管理门户中运行。您的用户通过门户访问程序，而您则通过管理员门户对程序和用户进行所有的管理。

有两种类型的 RemoteApp 部署：

-   在 Azure 云中托管云部署并存储程序的所有数据。
-   虽然混合部署在 Azure 云中托管，但此部署使用户能够访问存储在您的本地网络上的数据。

无论您选择哪种部署类型，在您创建该服务后，都会将您想要与您的用户共享的程序发布给最终用户源。这是您的用户通过 Azure 门户访问的可用程序的列表。请注意，该源会显示与您的订阅关联的所有服务的所有程序。

