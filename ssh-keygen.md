在运行 `ssh-keygen` 命令时，可以通过 `-f` 参数指定生成的 SSH Key 的文件名和存放路径。

例如，以下命令会在当前目录下生成一个名为 `my_ssh_key` 的 SSH Key：

```
ssh-keygen -t rsa -b 4096 -f my_ssh_key
```

其中，`-t rsa` 表示使用 RSA 算法生成 SSH Key，`-b 4096` 表示生成的 SSH Key 长度为 4096 位。

生成 SSH Key 后，可以使用以下命令将公钥复制到远程主机：

```
ssh-copy-id -i my_ssh_key.pub user@remote-host
```
或者
```
scp -P <port> <local-file> <username>@<remote-host>:<remote-path>
一般来说放在当前用户的~/.ssh
```

其中，`my_ssh_key.pub` 是公钥文件，`user` 是远程主机上的用户名，`remote-host` 是远程主机的地址或 IP。

然后把公钥文件配置到远程主机的~/.ssh/authorized_keys文件中
```
touch ~/.ssh/authorized_keys

cat ~/.ssh/my_ssh_key.pub >> ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys
```

本机里配置ssh连接的config
> vim ~/.ssh/config
```
Host utm
  HostName 192.168.11.22
  Port 12345
  User username
  IdentityFile ~/.ssh/id_rsa(私钥文件)
```

连接测试
```
ssh utm
```