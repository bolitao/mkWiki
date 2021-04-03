# Filebeat 安装和使用

debian 系安装：

```sh
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.12.0-amd64.deb
sudo dpkg -i filebeat-7.12.0-amd64.deb
```

安装完后去看看 `/etc/filebeat/filebeat.reference.yml` 文件，里面有丰富的样例和注释：

![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository@master/img/20210326153025.png)