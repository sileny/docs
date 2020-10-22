# permission

1、前提：已安装ssh

2、检查SSH公钥

```
cd ~/.ssh
```

3、生成SSH公钥

```
C:\Users\dear>ssh-keygen -t rsa -C "sunsilently@outlook.com"
```

4、添加 `SSH` 公钥到 `github`

打开github，找到账户里面添加 `SSH`，把 `id_rsa.pub` 内容复制到 `key` 里面


5 测试是否生效

使用下面的命令测试

```
ssh -T git@github.com
```

D:\code>ssh -T git@github.com
The authenticity of host 'github.com (13.250.177.223)' can't be established.<br/>
RSA key fingerprint is SHA256:nThbg6kXUpJxWGl7E1IGOCspRomTxd8ARxLviKw6E5SY.<br/>
Are you sure you want to continue connecting (yes/no)? **yes**<br/>
Warning: Permanently added 'github.com,13.250.177.223' (RSA) to the list of known hosts.<br/>
Enter passphrase for key 'C:\Users\dear/.ssh/id_rsa':<br/>
Hi sileny! You've successfully authenticated, but GitHub does not provide shell access.


