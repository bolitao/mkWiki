# Git 相关疑难杂症

## git gpg sign 提示 gpg No secret key

在 Windows 下，可能是 git 没找到 gpg 路径（我遇到的情况是这种），在 `~/.gitconfig` 添加如下行可解决：

``` conf
[gpg]
    program = C:\\Users\\username\\scoop\\apps\\gnupg\\current\\bin\\gpg.exe
```

## git 必要的几个配置

``` conf
[user]
    name = Boli Tao
    email = boli.tao@outlook.com
    signingkey = 8864AF040E6BACA1
[http]
    proxy = http://127.0.0.1:10809
[commit]
    gpgsign = true
```
