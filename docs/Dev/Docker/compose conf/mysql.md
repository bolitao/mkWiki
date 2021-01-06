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

TODO:

- 主从
