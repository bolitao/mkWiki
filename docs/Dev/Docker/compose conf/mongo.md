``` yml
version: '3.1'

services:
  mongo:
    image: mongo:latest
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: bolitao
      MONGO_INITDB_ROOT_PASSWORD: bolitao
    volumes:
      - /data/mongo/db:/data/db # data path
      - /data/mongo/log:/var/log/mongodb # log path
    ports:
      - 27017:27017

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: bolitao
      ME_CONFIG_MONGODB_ADMINPASSWORD: bolitao
```

> REF:
> 
> 1. [mongo - Docker Hub](https://hub.docker.com/_/mongo)
> 2. [docker-compose构建mongodb容器实例_feinifi的博客-CSDN博客](https://blog.csdn.net/feinifi/article/details/105098829)
