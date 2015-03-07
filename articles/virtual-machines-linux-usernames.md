<properties linkid="services-linux-user-names" urlDisplayName="User Names in Linux" pageTitle="在 Azure 上选择 Linux 的用户名" metaKeywords="" description="了解如何在 Azure 中选择 Linux 虚拟机的用户名。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Selecting User Names for Linux on Azure" authors="" solutions="" manager="" editor="" />





#选择 Azure # 上 linux 的用户名

当创建 Linux 虚拟机时，则可以选择其用户名，某一名称或接受默认值， *azureuser*。在大多数情况下这个新用户并不存在于基本映像，并且在设置过程中创建。如果在基的 VM 映像上存在的用户，然后 Azure Linux 代理只需配置的密码 （和/或 SSH 密钥） 为该用户根据在创建 VM 时指定的信息。

**但是**，Linux 定义了一套不应使用的用户名。设置过程将**失败**如果您尝试设置使用现有的系统用户，作为具有 UID 0-99 的用户定义的 Linux 虚拟机。一个典型示例是`root`用户，具有 UID 0。

 - 另请参见：[Linux 标准基本的用户 ID 范围](http://refspecs.linuxfoundation.org/LSB_4.1.0/LSB-Core-generic/LSB-Core-generic/uidrange.html)

以下是应避免使用时设置 Linux 虚拟机的用户名。我们建议您**不使用这些用户名**因为设置过程中可能会失败。


## openSUSE
- abrt
- adm
- audio
- bin
- bin
- cdrom
- cgred
- daemon
- dbus
- dialout
- dip
- disk
- floppy
- ftp
- games
- gopher
- haldaemon
- halt
- kmem
- lock
- lp
- mail
- man
- mem
- nfsnobody
- nobody
- ntp
- operator
- oprofile
- postdrop
- postfix
- qpidd
- root
- rpc
- rpcuser
- saslauth
- shutdown
- slocate
- sshd
- stapdev
- stapusr
- sync
- sys
- tape
- test
- tcpdump
- tty
- users
- utempter
- utmp
- uucp
- vcsa
- video
- wheel


## SLES
- audio
- bin
- cdrom
- console
- daemon
- dialout
- disk
- floppy
- ftp
- ftp
- games
- haldaemon
- kmem
- lp
- lp
- mail
- maildrop
- man
- messagebus
- modem
- news
- news
- nobody
- nogroup
- polkituser
- postfix
- public
- root
- shadow
- sshd
- sys
- test
- trusted
- tty
- users
- utmp
- uucp
- uuidd
- video
- wheel
- www
- wwwrun
- xok

 
## CentOS
- abrt
- adm
- audio
- bin
- cdrom
- cgred
- daemon
- dbus
- dialout
- dip
- disk
- floppy
- ftp
- ftp
- games
- gopher
- haldaemon
- halt
- kmem
- lock
- lp
- mail
- man
- mem
- nfsnobody
- nobody
- ntp
- operator
- oprofile
- postdrop
- postfix
- qpidd
- root
- rpc
- rpcuser
- saslauth
- shutdown
- slocate
- sshd
- stapdev
- stapusr
- sync
- sys
- tape
- test
- tcpdump
- tty
- users
- utempter
- utmp
- uucp
- vcsa
- video
- wheel


## Ubuntu
- adm
- admin
- audio
- backup
- bin
- cdrom
- crontab
- daemon
- dialout
- dip
- disk
- fax
- floppy
- fuse
- games
- gnats
- irc
- kmem
- landscape
- libuuid
- list
- lp
- mail
- man
- messagebus
- mlocate
- netdev
- news
- nobody
- nogroup
- operator
- plugdev
- proxy
- root
- sasl
- shadow
- src
- ssh
- sshd
- staff
- sudo
- sync
- sys
- syslog
- tape
- tty
- users
- utmp
- uucp
- video
- voice
- whoopsie
- www-data

