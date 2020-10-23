# mac

## nvm
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.36.0/install.sh | bash
```
如提示以下错误
```
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
```

通过 `https://ipaddress.com` 找到最合适的 `ip`，然后，添加到 `/etc/hosts`

```
199.232.68.133	raw.githubusercontent.com
199.232.68.133	githubusercontent.com
```

如提示以下错误
```
=> xcode-select: note: no developer tools were found at '/Applications/Xcode.app', requesting install. Choose an option in the dialog to download the command line developer tools.
```
表示需要安装 `xcode`

在 `app store` 里安装 `xcode`，然后，`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.36.0/install.sh | bash`

关闭终端后，再次打开一个终端，`nvm --version` 查看是否安装成功

如果遇到 `zsh: command not found: nvm`，没有 `.bash_profile` 则创建一个 `.bash_profile` 文件

```
# 进入当前用户的home目录
$ cd ~/
# 创建. bash_profile文件
$ touch .bash_profile 
# 创建. bash_profile文件
$ open .bash_profile
```

在后面添加一下配置内容
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

然后，保存文件，使之生效
```
source .bash_profile
```

```nvm --version```
输出版本信息，则表明安装成功，但是，环境变量添加后，每次打开都需要重新 `source .bash_profile` 才能使nvm生效，否则依然提示 `zsh: command not found: nvm`

如果没有 `.zshrc`，则需要创建该文件，然后，添加配置内容
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```
然后，执行 `source .zshrc`，重启terminal，再次执行 `nvm --version`，会有信息打印出来

## nginx

安装 `brew`

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

安装 `nginx`
```
brew install nginx
```

然后会输出以下内容
```
==> Pouring openssl@1.1-1.1.1h.catalina.bottle.tar.gz
==> Caveats
A CA file has been bootstrapped using certificates from the system
keychain. To add additional certificates, place .pem files in
  /usr/local/etc/openssl@1.1/certs

and run
  /usr/local/opt/openssl@1.1/bin/c_rehash

openssl@1.1 is keg-only, which means it was not symlinked into /usr/local,
because macOS provides LibreSSL.

If you need to have openssl@1.1 first in your PATH run:
  echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> ~/.zshrc

For compilers to find openssl@1.1 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"

==> Summary
🍺  /usr/local/Cellar/openssl@1.1/1.1.1h: 8,067 files, 18.5MB
==> Installing nginx dependency: pcre
==> Pouring pcre-8.44.catalina.bottle.tar.gz
🍺  /usr/local/Cellar/pcre/8.44: 204 files, 5.5MB
==> Installing nginx
==> Pouring nginx-1.19.3.catalina.bottle.tar.gz
==> Caveats
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
==> Summary
🍺  /usr/local/Cellar/nginx/1.19.3: 25 files, 2.2MB
==> Caveats
==> openssl@1.1
A CA file has been bootstrapped using certificates from the system
keychain. To add additional certificates, place .pem files in
  /usr/local/etc/openssl@1.1/certs

and run
  /usr/local/opt/openssl@1.1/bin/c_rehash

openssl@1.1 is keg-only, which means it was not symlinked into /usr/local,
because macOS provides LibreSSL.

If you need to have openssl@1.1 first in your PATH run:
  echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> ~/.zshrc

For compilers to find openssl@1.1 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"

==> nginx
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx

```

测试是否安装成功
```
nginx -v
```

有版本信息打印出来则表示成功


## node

```nvm ls```

会打印如下信息
```
iojs -> N/A (default)
node -> stable (-> N/A) (default)
unstable -> N/A (default)
```

```
brew install node
```

默认会安装最新的node

`nvm ls`会打印如下信息内容
```
->      v15.0.1
default -> node (-> v15.0.1)
node -> stable (-> v15.0.1) (default)
stable -> 15.0 (-> v15.0.1) (default)
iojs -> N/A (default)
unstable -> N/A (default)
lts/* -> lts/erbium (-> N/A)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.22.1 (-> N/A)
lts/erbium -> v12.19.0 (-> N/A)
```

如果没有安装过别的版本的 node，执行了 `nvm uninstall node` 会提示以下错误信息
```
nvm: Cannot uninstall currently-active node version, v15.0.1 (inferred from node).
```

安装另外一个版本的之后，执行，`nvm uninstall v15.0.1`，然后执行 `nvm ls`，打印如下消息内容
```
->     v12.19.0
default -> node (-> v12.19.0)
node -> stable (-> v12.19.0) (default)
stable -> 12.19 (-> v12.19.0) (default)
iojs -> N/A (default)
unstable -> N/A (default)
lts/* -> lts/erbium (-> v12.19.0)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.22.1 (-> N/A)
lts/erbium -> v12.19.0
```

执行 `nvm ls-remote`，可以打印出更多的版本信息列表
