# FastDFS 学习（1）—— 单机部署

## FastDFS

``` shell
# 安装编译所需的库和工具
sudo apt -y install libev-dev git gcc g++ make automake autoconf libtool pcre2-utils libpcre2-dev zlib1g zlib1g-dev openssl libssh-dev wget vim

# fastdfs 文件存储路径
sudo mkdir /home/dfs

cd /usr/local/src

# libfastcommon
sudo git clone https://github.com/happyfish100/libfastcommon.git --depth 1
cd libfastcommon/
sudo sh ./make.sh && sudo sh ./make.sh install

# FastDFS
cd ..
sudo git clone https://github.com/happyfish100/fastdfs.git --depth 1
cd fastdfs/
sudo sh ./make.sh
sudo sh ./make.sh install
sudo cp /usr/local/src/fastdfs/conf/http.conf /etc/fdfs/ #供nginx访问使用
sudo cp /usr/local/src/fastdfs/conf/mime.types /etc/fdfs/ #供nginx访问使用

# fastdfs-nginx-module
cd ..
sudo git clone https://github.com/happyfish100/fastdfs-nginx-module.git --depth 1
sudo cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs
```

## Nginx

``` shell
# dependencies
sudo apt install -y perl libperl-dev libgd3 libgd-dev libgeoip1 libgeoip-dev geoip-bin libxml2 libxml2-dev libxslt1.1 libxslt1-dev

# PCRE version 8.44
sudo wget https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.gz && sudo tar xzvf pcre-8.44.tar.gz
# zlib version 1.2.11
sudo wget https://www.zlib.net/zlib-1.2.11.tar.gz && sudo tar xzvf zlib-1.2.11.tar.gz
# openssl 1.1.1j
sudo wget https://www.openssl.org/source/openssl-1.1.1j.tar.gz && sudo tar xzvf openssl-1.1.1j.tar.gz
# nginx 1.18.0
sudo wget http://nginx.org/download/nginx-1.18.0.tar.gz && sudo tar -xzvf nginx-1.18.0.tar.gz
cd nginx-1.18.0

# man help
sudo cp man/nginx.8 /usr/share/man/man8
sudo gzip /usr/share/man/man8/nginx.8

# configure
sudo sh ./configure --prefix=/etc/nginx \
            --sbin-path=/usr/sbin/nginx \
            --modules-path=/usr/lib/nginx/modules \
            --conf-path=/etc/nginx/nginx.conf \
            --error-log-path=/var/log/nginx/error.log \
            --pid-path=/var/run/nginx.pid \
            --lock-path=/var/run/nginx.lock \
            --user=nginx \
            --group=nginx \
            --build=Ubuntu \
            --builddir=nginx-1.18.0 \
            --with-select_module \
            --with-poll_module \
            --with-threads \
            --with-file-aio \
            --with-http_ssl_module \
            --with-http_v2_module \
            --with-http_realip_module \
            --with-http_addition_module \
            --with-http_xslt_module=dynamic \
            --with-http_image_filter_module=dynamic \
            --with-http_geoip_module=dynamic \
            --with-http_sub_module \
            --with-http_dav_module \
            --with-http_flv_module \
            --with-http_mp4_module \
            --with-http_gunzip_module \
            --with-http_gzip_static_module \
            --with-http_auth_request_module \
            --with-http_random_index_module \
            --with-http_secure_link_module \
            --with-http_degradation_module \
            --with-http_slice_module \
            --with-http_stub_status_module \
            --with-http_perl_module=dynamic \
            --with-perl_modules_path=/usr/share/perl/5.30.0 \
            --with-perl=/usr/bin/perl \
            --http-log-path=/var/log/nginx/access.log \
            --http-client-body-temp-path=/var/cache/nginx/client_temp \
            --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
            --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
            --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
            --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
            --with-mail=dynamic \
            --with-mail_ssl_module \
            --with-stream=dynamic \
            --with-stream_ssl_module \
            --with-stream_realip_module \
            --with-stream_geoip_module=dynamic \
            --with-stream_ssl_preread_module \
            --with-compat \
            --with-pcre=../pcre-8.44 \
            --with-pcre-jit \
            --with-zlib=../zlib-1.2.11 \
            --with-openssl=../openssl-1.1.1j \
            --with-openssl-opt=no-nextprotoneg \
            --with-debug \
            --add-module=/usr/local/src/fastdfs-nginx-module/src/

# compile and install
sudo make && sudo make install

# add nginx user & group
sudo adduser --system --shell /bin/false --no-create-home --disabled-login --disabled-password --gecos "nginx user" --group nginx

# create folder
sudo mkdir -p /var/cache/nginx/client_temp /var/cache/nginx/fastcgi_temp /var/cache/nginx/proxy_temp /var/cache/nginx/scgi_temp /var/cache/nginx/uwsgi_temp
sudo chmod 700 /var/cache/nginx/*
sudo chown nginx:root /var/cache/nginx/*

# test nginx
sudo nginx -t
# output:
#nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
#nginx: configuration file /etc/nginx/nginx.conf test is successful
```

systemd:

``` shell
sudo vim /etc/systemd/system/nginx.service
```

copy following:
``` conf
[Unit]
Description=nginx - high performance web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

test systemd:

``` shell
sudo systemctl daemon-reload
sudo systemctl enable nginx.service
sudo systemctl start nginx.service
sudo systemctl status nginx.service
```

create directories & grant permissions:

``` shell
sudo mkdir /etc/nginx/{conf.d,snippets,sites-available,sites-enabled}
sudo chmod 640 /var/log/nginx/*
sudo chown nginx:adm /var/log/nginx/access.log /var/log/nginx/error.log
```

## FastDFS 单机配置

1. tracker

    ``` shell
    sudo vim /etc/fdfs/tracker.conf
    ```

    修改内容：

    ``` conf
    # 端口
    port = 22122
    # 存储日志和数据的根目录
    base_path=/home/dfs
    ```

2. storage

    ``` shell
    sudo vim /etc/fdfs/storage.conf
    ```

    修改内容：

    ``` conf
    # 数据和日志文件存储根目录
    base_path=/home/dfs
    # 第一个存储目录
    store_path0=/home/dfs
    # tracker服务器IP和端口，这里是单机部署所以就用本机的
    tracker_server=192.168.79.131:22122
    # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致)
    http.server_port=8888
    ```

## client 测试

``` shell
sudo vim /etc/fdfs/client.conf
```

修改内容：

``` conf
base_path=/home/dfs
tracker_server=192.168.79.131:22122

```

重启 tracker 和 storage：

``` shekll
sudo /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
sudo /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```

使用 fdfs_monitor 验证：

``` shell
sudo /usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```

输出如下（输出太长，省略部分）：

``` log
[2021-03-14 18:45:37] DEBUG - base_path=/home/dfs, connect_timeout=5, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=1, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

server_count=1, server_index=0

tracker server is 192.168.79.131:22122

group count: 1

Group 1:
group name = group1
```

测试：

``` shell
# 测试，有文件路径返回表示成功，如：group1/M00/00/00/wKhPg2BN6Z6AXk8WADVIAIo8asI.tar.gz
fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/zlib-1.2.11.tar.gz
```

## 配置 Nginx 访问

``` shell
sudo vim /etc/fdfs/mod_fastdfs.conf
```

修改如下：

``` conf
tracker_server=192.168.79.131:22122  #tracker服务器IP和端口
url_have_group_name=true
store_path0=/home/dfs
```

配置 nginx.config:

``` shell
sudo vim /etc/nginx/nginx.conf
```

添加 server 段如下：

``` conf
server {
    listen       8888;    ## 该端口为storage.conf中的http.server_port相同
    server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
```

reload nginx：

``` shell
sudo nginx -s reload
```

在测试章节中，我上传了名为 `group1/M00/00/00/wKhPg2BN6Z6AXk8WADVIAIo8asI.tar.gz` 的文件。

于是通过此链接测试 nginx 文件下载：`http://192.168.79.131:8888/group1/M00/00/00/wKhPg2BN6Z6AXk8WADVIAIo8asI.tar.gz`。

可以看到 FastDFS Nginx 模块是工作正常的：

![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210314190507.png)

## FastDFS 分布式部署

这部分是在工作机写的，IP 和系统（分布式用的 CentOS）有所不同可能造成和本篇内容的脱节，所以单独形成了一篇文章：

[]()

## 参考

- [Home · happyfish100/fastdfs Wiki](https://github.com/happyfish100/fastdfs/wiki)
- [How to Compile Nginx From Source on Ubuntu 20.04 LTS](How to Compile Nginx From Source on Ubuntu 20.04 LTS)
