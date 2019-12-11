# docker-compose collection

## Contents

- [Database](#database)
- [Logs](#logs)
- [Messaging](#messaging)
- [Metrics](#metrics)
- [Proxy](#proxy)
- [Services](#services)
- [Storage](#storage)
- [Search](#search)

### Database

- [Aerospike](#Aerospike)
- [Clickhouse](#Clickhouse)
- [MariaDB](#MariaDB)
- [Mongo](#Mongo)
- [MySQL](#MySQL)
- [Postgres](#Postgres)

#### Aerospike
```yaml
services:
  db-aerospike:
    image: aerospike/aerospike-server:4.5.2.2
    ports:
      - "3000:3000"
```

#### Clickhouse
```yaml
services:
  db-clickhouse:
    image: yandex/clickhouse-server
    ports:
      - "8123:8123"
      - "9000:9000"
      - "9009:9009"
```

#### MariaDB
```yaml
services:
  db-mariadb:
    image: mariadb
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: example
      MYSQL_USER: example
      MYSQL_PASSWORD: example
      MYSQL_ALLOW_EMPTY_PASSWORD: 'no'
      MYSQL_RANDOM_ROOT_PASSWORD: 'no'
    volumes:
      - mariadb:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db-mariadb
      MYSQL_USER: example
      MYSQL_PASSWORD: example

volumes:
  mariadb:
```

#### Mongo
```yaml
services:
  db-mongo:
    image: mongo
    restart: unless-stopped
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_DATABASE: db
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - mongo:/data/db

  db-mongo-express:
    image: mongo-express
    restart: unless-stopped
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: db-mongo
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example

volumes:
  mongo:
```

#### MySQL
```yaml
services:
  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE:
      MYSQL_USER:
      MYSQL_PASSWORD:
      MYSQL_ALLOW_EMPTY_PASSWORD:
      MYSQL_RANDOM_ROOT_PASSWORD:
      MYSQL_ONETIME_PASSWORD:
    volumes:
      - mysql:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db

volumes:
  mysql:
```

#### Postgres
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

### Logs
### Messaging

- [NATS](#NATS)
- [RabbitMQ](#RabbitMQ)

#### NATS
```yaml
services:
  nats:
    image: nats:1.4.1
    ports:
      - "4222:4222"
```   
 
#### RabbitMQ
```yaml
services:
  rabbitmq:
    image: rabbitmq:3.7-management-alpine
    restart: unless-stopped
    ports:
      - "5672:5672" # queue connection
      - "15672:15672" # management UI
    environment:
      - RABBITMQ_DEFAULT_USER=
      - RABBITMQ_DEFAULT_PASS=
      - RABBITMQ_DEFAULT_VHOST=/
    volumes:
      - rabbitmq:/var/lib/rabbitmq
    hostname: rabbitmq

volumes:
  rabbitmq:
```

### Metrics

- [Grafana](#Grafana)

#### Grafana
```yaml
  grafana:
    container_name: grafana
    image: grafana/grafana:6.2.1
    ports:
      - "3000:3000"
    volumes:
      - grafanadata:/var/lib/grafana
    networks:
      - metrics_net
    restart: always

volumes:
  grafanadata: {}
networks:
  metrics_net:
```

### Proxy

- [Envoy](#Envoy)
- [Nginx](#Nginx)

#### Envoy
```yaml
  proxy-envoy:
    build:
      context: .
      dockerfile: envoyproxy/envoy
    volumes:
      - ./envoy.yaml:/etc/envoy.yaml
    networks:
      - envoymesh
    expose:
      - "80"
      - "8001"
    ports:
      - "8000:80"
      - "8001:8001"

networks:
  envoymesh: {}
```

#### Nginx
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

### Services

- [Consul](#Consul)
- [Zookeeper](#Zookeeper)

#### Consul
```yaml
  consul:
    image: consul
    command: "agent -dev -bind=0.0.0.0 -client=0.0.0.0 -server -ui -bootstrap -config-file=/config/consul.json -enable-script-checks"
    ports:
      - 8500:8500
    dns_search: .
    volumes:
      - ./consul.json:/config/consul.json
```

#### Zookeeper
```yaml
services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
      ZOO_TICK_TIME: 2000
      ZOO_INIT_LIMIT: 5
      ZOO_SYNC_LIMIT: 2
      ZOO_MAX_CLIENT_CNXNS: 60
      ZOO_STANDALONE_ENABLED: true
      ZOO_AUTOPURGE_PURGEINTERVAL: 0
      ZOO_AUTOPURGE_SNAPRETAINCOUNT: 3

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888
```

### Storage

- [Memcached](#Memcached)
- [Redis](#Redis)

#### Memcached
```yaml
services:
  memcached:
    image: memcached:1-alpine
    restart: unless-stopped
    ports:
      - "11211:11211"
```

#### Redis
```yaml
services:
  redis:
    image: redis:5-alpine
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis:/data
    command: redis-server --appendonly yes

volumes:
  redis:
```

### Search

- [ELK Stack: Elasticsearch, Logstash, Kibana](#ELK)

#### ELK
```yaml

# Required on Linux:
# sudo sysctl -w vm.max_map_count=262144
# sudo sh -c 'echo "vm.max_map_count=262144" >> /etc/sysctl.conf'
# https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html

services:
  elk:
    image: sebp/elk:683
    restart: unless-stopped
    ports:
      - "5601:5601" # Elasticsearch
      - "9200:9200" # Kibana
    volumes:
      - es:/usr/share/elasticsearch/data

volumes:
  es:
```
