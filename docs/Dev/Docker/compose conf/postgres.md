``` yml
version: '3.1'

services:
  db:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: bolitao
      POSTGRES_PASSWORD: bolitao
    volumes:
      - /data/postgres/db:/var/lib/postgresql/data/
    ports:
      - 5432:5432
```
