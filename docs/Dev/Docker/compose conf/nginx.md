``` conf
version: '3'

services:
  nginx:
    image: nginx:latest
    ports:
      - 1199:80
    volumes:
      - /root/apps/myapps/boliblog/boliblog-fe:/usr/share/nginx/html
      - /data/myapps/boliblog/nginx/nginx.conf:/etc/nginx/nginx.conf
    privileged: true
```

TODO:

1. deploy
2. proxy
3. ssl

- [https://codewithhugo.com/setting-up-express-and-redis-with-docker-compose/](https://codewithhugo.com/setting-up-express-and-redis-with-docker-compose/)