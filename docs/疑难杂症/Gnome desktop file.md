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

WPS Example: 

``` desktop
[Desktop Entry]
Comment=Use WPS Writer to office work.
Comment[zh_CN]=使用 WPS 2019进行办公
Exec=/usr/bin/wps %F
GenericName=WPS
GenericName[zh_CN]=WPS 2019
Name=WPS 2019
Name[zh_CN]=WPS 2019
StartupNotify=false
Terminal=false
Type=Application
Categories=Office;WordProcessor;Qt;
X-DBUS-ServiceName=
X-DBUS-StartupType=
X-KDE-SubstituteUID=false
X-KDE-Username=
Icon=wps-office2019-kprometheus
InitialPreference=3
StartupWMClass=wpsoffice
```
