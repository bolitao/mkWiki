``` yml
version: '3.1'

services:
  db:
    image: mysql:latest
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: bolitao
      MYSQL_USER: 'bolitao'
      MYSQL_PASSWORD: 'bolitao'
      TZ: Asia/Shanghai
      character-set-server: utf8mb4
      collation-server: utf8mb4_unicode_ci
    volumes:
      - /data/mysql/db:/var/lib/mysql
    ports:
      - 3306:3306
    expose:
      - 3306

  adminer:
    image: adminer
    restart: always
    ports:
      - 8089:8080
```

grant:

``` sql
GRANT ALL PRIVILEGES ON *.* TO 'bolitao'@'%' WITH GRANT OPTION;
```

> **PS**
> **一定要指定**时区**环境变量，不然用 `NOW()`、`SYSDATE()` 等内置 MySQL 方法会出现非预期结果。``**

TODO:

- 主从
