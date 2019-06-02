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
