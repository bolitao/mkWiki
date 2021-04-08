# IDE 的各种小问题

## GNOME 下 Jetbrains IDE 无法使用 `CTRL + ALT + LEFT/RIGHT` 进行历史浏览跳转

和 GNOME 键位冲突导致的，使用下列命令屏蔽 GNOME 左右移动工作区的快捷键：

``` shell
gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-left "[]"
gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-right "[]"
```

恢复：

``` shell
gsettings reset org.gnome.desktop.wm.keybindings switch-to-workspace-left
gsettings reset org.gnome.desktop.wm.keybindings switch-to-workspace-right
```
