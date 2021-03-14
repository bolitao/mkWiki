### 集群部署

准备三台服务器，其 IP 为 192.168.15.130、192.168.15.131、192.168.15.132（简记为 130、131、132）。三台服务器均已安装 FastDFS tracker、FastDFS storage、FastDFS client 和 Tengine。

1. 修改三台机器的 tracker 配置：

   ``` shell
   sudo vim /etc/fdfs/tracker.conf
   
   # 修改以下内容
   port=22122  # tracker 服务器端口
   base_path=/home/dfs  # 存储日志和数据的根目录
   ```

2. 修改三台机器的 storage 配置：

   ``` shell
   sudo vim /etc/fdfs/storage.conf
   
   # 修改以下配置
   tracker_server = 192.168.15.130:22122
   tracker_server = 192.168.15.131:22122
   tracker_server = 192.168.15.132:22122
   port=23000
   base_path=/home/dfs
   store_path0=/home/dfs
   http.server_port=8888
   ```

   重启 tracker  和 storage 服务，检查是否 storage 服务器是否登记到 tracker：

   ``` shell
   sudo /etc/init.d/fdfs_trackerd restart # 重启 tracker 服务
   sudo /etc/init.d/fdfs_storaged restart # 重启 storage 服务
   
   sudo /usr/bin/fdfs_monitor /etc/fdfs/storage.conf # 检查服务状态
   # 以下是输出，省略部分非关键信息
   ...
   tracker server is 192.168.15.132:22122
   
   group count: 1
   
   Group 1:
   group name = group1
   ...
           Storage 1:
                   id = 192.168.15.130
                   ip_addr = 192.168.15.130  ACTIVE
                   http domain =
                   version = 6.06
                   join time = 2020-07-14 10:08:26
                   up time = 2020-07-14 15:57:34
   ...
           Storage 2:
                   id = 192.168.15.131
                   ip_addr = 192.168.15.131  ACTIVE
                   http domain =
                   version = 6.06
   ...
           Storage 3:
                   id = 192.168.15.132
                   ip_addr = 192.168.15.132  ACTIVE
                   http domain =
                   version = 6.06
   ```

   可以看到三台存储服务器都处于激活状态，多运行几次 `fdfs_monitor` 命令会发现 `tracker server` 有所变换，即多个 tracker 的切换功能也是生效的。

3. 测试 client，修改 client 配置文件：

   ``` shell
   sudo vim /etc/fdfs/client.conf
   
   # 需要修改的内容如下：
   base_path=/home/moe/dfs
   tracker_server = 192.168.15.130:22122
   tracker_server = 192.168.15.131:22122
   tracker_server = 192.168.15.132:22122
   ```

   测试 client 文件上传：

   ``` shell
   fdfs_upload_file /etc/fdfs/client.conf /home/bolitao/apps/screenfetch/screenfetch-dev
   # 返回上传的文件位置和文件名：
   group1/M00/00/00/wKgPgl8Nau-AcXEFAAPBDqCqwtI9997099
   ```

   由于三台 storage 处于同一 group，组内文件互备，所以可以在三台存储服务器中发现上传的文件：

   ``` shell
   ls -l /home/dfs/data/00/00 # 列出文件
   # 以下是命令输出：
   -rw-r--r-- 1 root root 246030 7月  14 16:21 wKgPgl8Nau-AcXEFAAPBDqCqwtI9997099
   ```

4. 配置 nginx 访问，修改服务器的 `/etc/fdfs/mod_fastdfs.conf` 文件：

   ``` shell
   tracker_server = 192.168.15.130:22122
   tracker_server = 192.168.15.131:22122
   tracker_server = 192.168.15.132:22122
   url_have_group_name=true
   store_path0=/home/dfs
   ```

   在 nginx 中添加以下 server：

   ``` nginx
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

   重启三台 nginx 后，可在三台服务器下载文件，比如访问 `http://192.168.15.130:8888/group1/M00/00/00/wKgPgl8Nau-AcXEFAAPBDqCqwtI9997099`（或其他两个 host），会弹出下载对话框：

   ![image-20200714163319953](https://cdn.jsdelivr.net/gh/bolitao/PicRepository@master/img/image-20200714163319953.png)

> 备注：这里配置了三台服务器既作 tracker 服务器也作存储服务器。实际运用时更多情况是将 tracker 和 storage 划分，tracker 用作负载均衡和调度，storage 用来存储，如下图所示：
>
> ![11](https://cdn.jsdelivr.net/gh/bolitao/PicRepository@master/img/11.png)

### 基本操作

- tracker 服务管理：

  ``` shell
  sudo /etc/init.d/fdfs_trackerd start # 启动 tracker 服务
  sudo /etc/init.d/fdfs_trackerd restart # 重启动 tracker 服务
  sudo /etc/init.d/fdfs_trackerd stop # 停止 tracker 服务
  sudo chkconfig fdfs_trackerd on # 自启动 tracker 服务
  ```

- storage 服务管理：

  ``` shell
  sudo /etc/init.d/fdfs_storaged start # 启动 storage 服务
  sudo /etc/init.d/fdfs_storaged restart # 重启 storage 服务
  sudo /etc/init.d/fdfs_storaged stop # 停止 storage 服务
  sudo chkconfig fdfs_storaged on # 自启动 storage 服务
  ```

- 检查集群（会输出所有服务器的信息）：

  ``` shell
  sudo /usr/bin/fdfs_monitor /etc/fdfs/storage.conf
  # 以下是命令输出：
  [sudo] bolitao 的密码：
  [2020-07-14 10:24:49] DEBUG - base_path=/home/dfs, connect_timeout=5, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=1, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0
  
  server_count=1, server_index=0
  
  tracker server is 192.168.188.130:22122
  
  group count: 1
  
  Group 1:
  group name = group1
  disk total space = 48,096 MB
  disk free space = 44,515 MB
  trunk free space = 0 MB
  storage server count = 1
  active server count = 1
  storage server port = 23000
  storage HTTP port = 8888
  store path count = 1
  subdir count per path = 256
  current write server index = 0
  current trunk file id = 0
  ...
  ```

- 上传：使用 `fdfs_upload_file` 命令，命令跟的第一个参数表示客户端配置文件，第二个参数表示要上传的文件，比如：

  ``` shell
  sudo fdfs_upload_file /etc/fdfs/client.conf /home/bolitao/apps/apache-tomcat-9.0.37/BUILDING.txt
  ```

- 下载：使用 `fdfs_download_file` 命令进行下载，第一个参数是客户端配置文件，第二个参数是要下载的文件，比如：

  ``` shell
  fdfs_download_file /etc/fdfs/client.conf group1/M00/00/00/wKi8gl8NE5yAFQc7AABKJvC0Vqw094.txt
  ```

- 删除：使用 `fdfs_delete_file` 命令进行文件删除，第一个参数是配置文件，第二个参数是要删除的文件，比如：

  ``` shell
  fdfs_delete_file /etc/fdfs/client.conf group1/M00/00/00/wKi8gl8NE5yAFQc7AABKJvC0Vqw094.txt
  
  # 再次下载该文件
  fdfs_download_file /etc/fdfs/client.conf 
  group1/M00/00/00/wKi8gl8NE5yAFQc7AABKJvC0Vqw094.txt
  # 提示文件不存在
  [2020-07-14 10:39:16] ERROR - file: tracker_proto.c, line: 50, server: 192.168.188.130:23000, response status 2 != 0
  [2020-07-14 10:39:16] ERROR - file: ../client/storage_client.c, line: 598, fdfs_recv_header fail, result: 2
  download file fail, error no: 2, error info: No such file or directory
  ```

- 设置多个组：

  1. 修改 `/etc/fdfs/storage.conf` 文件中：

     - `store_path_count` 的值
     - 新增 `store_path`，比如 `store_path1`、`store_path2`

  2. 修改 `mod_fastdfs.conf` 文件：

     - `store_path_count` 的值

     - `store_pathx` 的值

     - 仿照 group 添加更多 group，比如：

       ``` shell
       [group1]
       group_name=group1
       storage_server_port=23000
       store_path_count=3
       store_path0=/home/dfs/data/group1/M00
       store_path1=/home/dfs/data/group1/M01
       
       [group2]
       group_name=group2
       storage_server_port=23000
       store_path_count=3
       store_path0=/home/dfs/data/group2/M00
       store_path1=/home/dfs/data/group2/M01
       ```

  3. 修改 nginx 反向代理配置：

     ``` nginx
     location ~/group[0-9]/M00 {
         root /home/dfs/data/group1
         ngx_fastdfs_module;
     }
     
     location ~/group[0-9]/M01 {
         root /home/dfs/data/group2
         ngx_fastdfs_module;
     }
     ```
