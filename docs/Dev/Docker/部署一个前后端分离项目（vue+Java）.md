## java Dockerfile

``` Dockerfile
FROM java:8
EXPOSE 8899
ADD boliblog-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java", "-jar", "/app.jar", "--spring.profiles.active=prod"]
```

## docker-compose

``` yml
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
  mysql:
    image: mysql:latest
    ports:
      - 3307:3306
    environment:
      MYSQL_ROOT_PASSWORD: 'bolitao'
  redis:
    image: redis:latest
    ports:
      - 6389:6379
    volumes:
      - /data/myapps/boliblog/redis/redis.conf:/usr/local/etc/redis/redis.conf
      - /data/myapps/boliblog/redis/datadir:/data/
      - /data/myapps/boliblog/redis/logs:/logs/
    command:
      - /usr/local/etc/redis/redis.conf
  boliblog:
    image: boliblog:latest
    build: .
    ports:
      - 8899:8899
    depends_on:
      - mysql
      - redis
```
