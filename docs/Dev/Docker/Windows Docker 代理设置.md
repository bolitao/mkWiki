在使用 Docker for Windows 时，设置本地代理（127.0.0.1:10809）无法拉取镜像：

``` shell
Unable to find image 'docker/getting-started:latest' locally
docker: Error response from daemon: Get https://registry-1.docker.io/v2/: proxyconnect tcp: dial tcp 127.0.0.1:10809: connect: connection refused.
See 'docker run --help'.
```

原因是我使用的 Docker 后端基于 WSL 2，其代理不能设置为 `127.0.0.1` 或是 `localhost`，而应该设置为 Windows 的 WSL 网卡地址，比如：

``` powershell
❯ ipconfig

# output:
以太网适配器 vEthernet (WSL):

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::a99e:b411:2b0c:1ad6%47
   IPv4 地址 . . . . . . . . . . . . : 172.30.112.1
   子网掩码  . . . . . . . . . . . . : 255.255.240.0
   默认网关. . . . . . . . . . . . . :
```

将代理设置为 `172.30.112.1:10809` 就好了。

``` shell
docker pull mysql
Using default tag: latest
latest: Pulling from library/mysql
6ec7b7d162b2: Pull complete
fedd960d3481: Pull complete
7ab947313861: Pull complete
64f92f19e638: Pull complete
3e80b17bff96: Pull complete
014e976799f9: Pull complete
59ae84fee1b3: Pull complete
ffe10de703ea: Pull complete
657af6d90c83: Pull complete
98bfb480322c: Pull complete
9f2c4202ac29: Pull complete
a369b92bfc99: Pull complete
Digest: sha256:365e891b22abd3336d65baefc475b4a9a1e29a01a7b6b5be04367fcc9f373bb7
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest
```
