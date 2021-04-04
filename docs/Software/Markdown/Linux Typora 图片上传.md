# Linux Typora 图片上传

Typora 在 Windows 和 MacOS 下能够正常调用 PicGo 来上传图片，不过在 Linux 各发行版上是不行的，所以需要借助 PicGo Core 进行上传。

~~ 安装 core 库： ~~

~~
``` shell
cnpm install -g picgo
```
~~
~~ 创建 picgo 配置文件：~~

core：直接在 Typora 下载。

core 配置文件内容：

``` json
{
  "picBed": {
    "current": "github",
    "uploader": "github",
    "proxy": "http://127.0.0.1:10809",
    "github": {
      "repo": "bolitao/PicRepository",
      "token": "xx",
      "path": "img/",
      "customUrl": "https://cdn.jsdelivr.net/gh/bolitao/PicRepository",
      "branch": "master"
      }
  },
  "picgoPlugins": {
    "picgo-plugin-github": true
  }
}
```
