重装了下笔点的 Ubunt，发现 RescueTime 的 Launcher 的启动器图标竟然没了，手动添加下吧：

``` shell
sudo vim /usr/share/applications/Rescuetime.desktop # 创建 .desktop 文件
```

内容如下：

``` desktop
[Desktop Entry]
Encoding=UTF-8
Name=RescueTime
Exec=/usr/bin/rescuetime
#Icon=/path/to/eclipse/icon
Type=Application
Categories=Others;
```
