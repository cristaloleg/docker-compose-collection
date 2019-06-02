# docker-compose Collection

- Postgres:
```yaml
services:
  db-postgres:
    image: postgres:11.3
    container_name: postgres_container
    restart: unless-stopped
    tty: true
    ports:
      - "5432:5432"
    environment:
      - PGHOST=localhost
      - PGDATABASE=postgres
      - PGUSER=postgres
      - POSTGRES_PASSWORD=
    volumes:
      - my-db-postgres:/var/lib/postgresql/data

volumes:
  my-db-postgres:
```

- MariaDB
```yaml
services:
  db-mariadb:
    image: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: example
      MYSQL_USER: example
      MYSQL_PASSWORD: example
      MYSQL_ALLOW_EMPTY_PASSWORD: yes # no
      MYSQL_RANDOM_ROOT_PASSWORD: yes # <not set>
```

- Mongo
```yaml
services:
  db-mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_DATABASE: db
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example

  db-mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
```

- MySQL
```yaml
services:
  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE:
      MYSQL_USER:
      MYSQL_PASSWORD:
      MYSQL_ALLOW_EMPTY_PASSWORD: 
      MYSQL_RANDOM_ROOT_PASSWORD: 
      MYSQL_ONETIME_PASSWORD: 
```

- Nginx
```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    environment:
      - NGINX_HOST=foobar.com
      - NGINX_PORT=80
    volumes:
      - ./mysite.template:/etc/nginx/conf.d/mysite.template
    # command: [nginx-debug, '-g', 'daemon off;']
```