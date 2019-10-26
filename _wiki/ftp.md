---
layout: wiki
title: ftp server and client
categories: Tools
description: vsftp usage
keywords: ftp
---

### Introduction

ftp 是文件传输协议 file transfer protocol 的缩写，现在较为安全的 ftp 工具是 [vsftpd](https://security.appspot.com/vsftpd.html)，它针对安全性、性能和稳定性都做了优化，可以较好的防范一些安全性的漏洞，本文仅针对基于 vsftpd 的 ftp 服务器和客户端访问进行了简要描述。

### Reference

[如何在 Ubuntu 18.04 上为用户目录设置 vsftpd](https://www.howtoing.com/how-to-set-up-vsftpd-for-a-user-s-directory-on-ubuntu-18-04)

### Install

首先安装 vsftpd

```shell
$ sudo apt-get install vsftpd
```

安装完之后备份配置文件 vsftpd.conf

```shell
$ sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

### Prepare User

添加用来访问 ftp 服务器的用户，**这里请尽量使用 adduser 命令而不是 useradd**，因为会自动在 /home 目录下生成对应的用户文件夹

```shell
$ sudo adduser share
```

之后根据提示来设置密码和相关信息

当用户被限制在特定目录时，FTP通常更安全。 vsftpd 用 chroot 完成了这个要求。 为本地用户启用 chroot，默认情况下它们仅限于其主目录。 但是，由于 vsftpd 保护目录的方式，用户不能写入。 这对于只应通过FTP连接的新用户来说很好，但如果现有用户也具有shell访问权限，则可能需要写入其主文件夹。

在这个例子中，不是从主目录中删除写权限，而是创建一个 ftp 目录作为 chroot 和一个可写 files 目录来保存实际文件。

创建 ftp 文件夹

```shell
$ sudo mkdir /home/share/ftp
```

设置所有权

```shell
$ sudo chown nobody:nogroup /home/share/ftp
```

删除写权限

```shell
$ sudo chmod a-w /home/share/ftp
```

验证权限

```shell
$ sudo ls -la /home/share/ftp
Outputtotal 8
4 dr-xr-xr-x  2 nobody nogroup 4096 Aug 24 21:29 .
4 drwxr-xr-x  3 share  share   4096 Aug 24 21:29 ..
```

接下来创建文件上传目录并为用户分配所有权

```shell
$ sudo mkdir /home/share/ftp/files
$ sudo chown share:share /home/share/ftp/files
```

对 ftp 目录的权限检查应返回以下内容：

```shell
$ sudo ls -la /home/share/ftp
Outputtotal 12
dr-xr-xr-x 3 nobody nogroup 4096 Aug 26 14:01 .
drwxr-xr-x 3 share  share   4096 Aug 26 13:59 ..
drwxr-xr-x 2 share  share   4096 Aug 26 14:01 files
```

最后，让我们添加一个 test.txt 文件，以便在测试时使用

```shell
$ echo "vsftpd test file" | sudo tee /home/share/ftp/files/test.txt
```

现在我们已经保护了 ftp 目录并允许用户访问 files 目录，让我们修改我们的配置。

### Configure

我们计划允许具有本地shell帐户的单个用户与FTP连接。 这两个关键设置已在 vsftpd.conf 设置。 首先打开配置文件，验证配置中的设置是否与以下设置相匹配

```shell
$ sudo vim /etc/vsftpd.conf
```

/etc/vsftpd.conf文件

```shell
# Allow anonymous FTP? (Disabled by default).
anonymous_enable=NO
#
# Uncomment this to allow local users to log in.
local_enable=YES
```

接下来，让我们通过取消注释`write_enable`设置来允许用户上传文件

/etc/vsftpd.conf

```shell
write_enable=YES
```

我们还将取消注释 chroot 以防止 FTP 连接的用户访问目录树之外的任何文件或命令

/etc/vsftpd.conf

```shell
chroot_local_user=YES
```

我们还添加一个`user_sub_token`以在`local_root directory`路径中插入用户名，这样我们的配置将适用于此用户和任何其他未来用户。 在文件中的任何位置添加这些设置

/etc/vsftpd.conf

```shell
user_sub_token=$USER
local_root=/home/$USER/ftp
```

要根据具体情况允许FTP访问，让我们设置配置，以便用户只有在明确添加到列表时才能访问，而不是默认情况下：

/etc/vsftpd.conf文件

```shell
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
```

userlist_deny 切换逻辑：当它设置为 YES，列表中的用户被拒绝 FTP 访问。 当它设置为 NO，只允许列表中的用户访问。

完成更改后，保存文件并退出编辑器。

最后，让我们将用户添加到`/etc/vsftpd.userlist`。 使用 *-a* 标志追加到文件

```shell
$ echo "sammy" | sudo tee -a /etc/vsftpd.userlist
```

检查它是否按预期添加

```shell
$ cat /etc/vsftpd.userlist
share
```

重新启动守护程序以加载配置更改

```shell
$ sudo systemctl restart vsftpd
```

配置到位后，继续测试 FTP 访问。

### Test

我们已将服务器配置为仅允许用户 share 通过 FTP 连接。 让我们确保它按预期工作。

**匿名用户应该无法连接**：我们已禁用匿名访问。 让我们通过尝试匿名连接来测试它。 如果我们的配置设置正确，则应拒绝匿名用户的权限。 

```
ftp -p {your ftp server ip}
OutputConnected to {your ftp server ip}.
220 (vsFTPd 3.0.3)
Name ({your ftp server ip}:default): anonymous
530 Permission denied.
ftp: Login failed.
ftp>
```

关闭连接：

```
bye
```

**除了 share 之外的用户应该无法连接**：接下来，让我们尝试连接 sudo 用户。 他们也应该被拒绝访问，并且应该在他们被允许输入密码之前发生：

```
ftp -p {your ftp server ip}
OutputConnected to {your ftp server ip}.
220 (vsFTPd 3.0.3)
Name ({your ftp server ip}:default): sudo_user
530 Permission denied.
ftp: Login failed.
ftp>
```

关闭连接：

```
bye
```

**用户 share 应该能够连接，读取和写入文件**：让我们确保我们的指定用户可以连接

```
ftp -p {your ftp server ip}
OutputConnected to {your ftp server ip}.
220 (vsFTPd 3.0.3)
Name ({your ftp server ip}:default): share
331 Please specify the password.
Password: your_user's_password
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

让我们切换到`files`目录并使用`get`命令将我们之前创建的测试文件传输到本地机器：

```
cd files
get test.txt
Output227 Entering Passive Mode (203,0,113,0,169,12).
150 Opening BINARY mode data connection for test.txt (16 bytes).
226 Transfer complete.
16 bytes received in 0.0101 seconds (1588 bytes/s)
ftp>
```

接下来，让我们使用新名称上传文件以测试写入权限：

```
put test.txt upload.txt
Output227 Entering Passive Mode (203,0,113,0,164,71).
150 Ok to send data.
226 Transfer complete.
16 bytes sent in 0.000894 seconds (17897 bytes/s)
```

关闭连接：

```
bye
```