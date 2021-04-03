
``` bash
sudo mkdir -p /etc/systemd/system/docker.service.d
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

``` bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

check:

``` bash
sudo systemctl show --property=Environment docker
```

``` bash
# output:
Environment=HTTP_PROXY=http://192.168.79.1:10809 HTTPS_PROXY=http://192.168.79.1:10809 NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp
```
