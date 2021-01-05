## mirrors

`sources.list`:

``` shell
nano /etc/apt/sources.list
```

content:

``` shell
deb http://mirrors.ustc.edu.cn/debian/ buster main
deb-src http://mirrors.ustc.edu.cn/debian/ buster main

deb https://mirrors.tuna.tsinghua.edu.cn/debian-security/ buster/updates main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security/ buster/updates main contrib non-free

# buster-updates, previously known as 'volatile'
deb http://mirrors.ustc.edu.cn/debian/ buster-updates main contrib non-free
deb-src http://mirrors.ustc.edu.cn/debian/ buster-updates main contrib non-free
```

`openmediavault.list`:

``` shell
nano /etc/apt/sources.list.d/openmediavault.list
```

content:

``` shell
deb https://mirrors.bfsu.edu.cn/OpenMediaVault/public/ usul main
deb https://mirrors.bfsu.edu.cn/OpenMediaVault/public/ usul-proposed main
deb https://mirrors.bfsu.edu.cn/OpenMediaVault/public/ usul partner
```

## omv extras

install (may use a proxy):

``` shell
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash 
```

> ref: [omv-extras.org](https://omv-extras.org/)

mirrors:

`nano /etc/apt/sources.list.d/omvextras.list`

``` shell
deb https://mirrors.bfsu.edu.cn/OpenMediaVault/openmediavault-plugin-developers/usul buster main
deb [arch=amd64] https://download.docker.com/linux/debian buster stable
deb http://linux.teamviewer.com/deb stable main
deb https://mirrors.bfsu.edu.cn/OpenMediaVault/openmediavault-plugin-developers/usul-testing buster main
deb https://mirrors.bfsu.edu.cn/OpenMediaVault/openmediavault-plugin-developers/usul-extras buster main
```

## docker proxy

``` shell
sudo mkdir -p /etc/systemd/system/docker.service.d
```

``` shell
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf
```

content:

``` conf
[Service]
Environment="HTTP_PROXY=http://192.168.79.1:10809"
Environment="HTTPS_PROXY=http://192.168.79.1:10809"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
```

restart:

``` shell
sudo systemctl daemon-reload
sudo systemctl show --property=Environment docker # check
sudo systemctl restart docker
```

> ref: [Control Docker with systemd | Docker Documentation](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)