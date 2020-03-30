## Linux用户新增

### 1、判断用户是否存在

```shell
cat /etc/passwd user
```

如果存在用户，可以使用userdel -r user删除用户

### 2、新增用户

```shell
useradd -d /home/sftpuser -s /sbin/nologin sftpuser
```

### 3、设置密码

```shell
passwd sftpuser
```

### 4、修改/etc/ssh/sshd_config配置

```shell
Match User sftpuser
ChrootDirectory [指定路径]
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp
```

### 5、重启sshd服务

```shell
service sshd restart
```

