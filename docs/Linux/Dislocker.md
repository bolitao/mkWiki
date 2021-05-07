# Linux 下读写 bitlocker 设备

安装 dislocker：

``` shell
yay -S dislocker
```

创建两个文件夹，一个为 dislocker 挂载位置，一个为 mount 挂载位置：

``` shell
sudo mkdir /media/bitlocker /media/mount
```

使用 dislocker 挂载 bitlocker 分区：

``` shell
# 只读:
sudo dislocker -r -V /dev/nvme0n1p4 -upw -- /media/bitlocker

# 读写：
# sudo dislocker -V /dev/nvme0n1p4 -upw -- /media/bitlocker
```

以上命令中 `/dev/nvme0n1p4` 为 bitlocker 分区，**`pw`** 为密码。

挂载 dislocker 文件到为实际可用分区：

``` shell
sudo -i
# 只读：
mount -r -o loop /media/bitlocker/dislocker-file /media/mount

# 读写：
# mount -o loop /media/bitlocker/dislocker-file /media/mount
```
