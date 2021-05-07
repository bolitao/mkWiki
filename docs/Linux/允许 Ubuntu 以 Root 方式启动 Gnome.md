# 允许 Ubuntu 以 Root 方式启动 Gnome

最近在捣鼓加密狗，厂商给的 C API 需要以 root 身份调用，我在其基础上用 JNA 封装了一层，方便 Java 调用，于是理所当然需要以 root 权限运行 IntelliJ 才能够调试加密狗。以上这是背景。

起初我尝试以 `sudo ./idea.sh` 来运行 IDEA，遗憾会闪退，GPU 啥的报错。只能换用 root 帐号登录 GUI 进行开发。

用 root 帐号登录 Gnome 时发现无法登录，不论输入什么都是密码错误，需要进行下配置：

编辑文件 `/etc/gdm3/custom.conf`：

``` shell
sudo vim /etc/gdm3/custom.conf
```

在 `[daemon]` 下添加如下 `AllowRoot=true`：

![CopyQ.HKXfHF.png](https://i.loli.net/2021/04/28/Xu1m9W5YVzMeoNH.png)

编辑文件 `/etc/gdm3/custom.conf`：

``` shell
sudo vim /etc/gdm3/custom.conf
```

注释以下行：

``` conf
auth   required        pam_succeed_if.so user != root quiet_success
```

![CopyQ.JBQmqN.png](https://i.loli.net/2021/04/28/ri7q8CIMvmxlPsp.png)

重启后就能以 root 登录 Gnome 了。
