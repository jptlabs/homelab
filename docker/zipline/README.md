# Zipline Setup Guide

> [!NOTE]
> For a more-in-depth guide read the official documentation [here](https://github.com/diced/zipline).
>

## Installing with Docker Compose

```yml
services:
  postgresql:
    image: postgres:16
    restart: unless-stopped
    env_file:
      - .env
    environment:
      POSTGRES_USER: ${POSTGRESQL_USER:-zipline}
      POSTGRES_PASSWORD: ${POSTGRESQL_PASSWORD:?POSTGRESSQL_PASSWORD is required}
      POSTGRES_DB: ${POSTGRESQL_DB:-zipline}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD', 'pg_isready', '-U', 'zipline']
      interval: 10s
      timeout: 5s
      retries: 5

  zipline:
    image: ghcr.io/diced/zipline
    ports:
      - '3000:3000' # Change this as needed
    env_file:
      - .env
    environment:
      - DATABASE_URL=postgres://${POSTGRESQL_USER:-zipline}:${POSTGRESQL_PASSWORD}@postgresql:5432/${POSTGRESQL_DB:-zipline}
    depends_on:
      - postgresql
    volumes:
      - '/path/to/your/share:/zipline/uploads' # The path to your smb or nfs share
      - './public:/zipline/public'
      - './themes:/zipline/themes'

volumes:
  pgdata:
```

Create a docker-compose.yml file and paste the contents in, or run this command:

```bash
wget https://raw.githubusercontent.com/jptlabs/homelab/main/docker/zipline/docker-compose.yml
```

### Generating Secrets

```bash
echo "POSTGRESQL_PASSWORD=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" > .env
echo "CORE_SECRET=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" >> .env
```

Without these variables zipline won't run.

### Starting Zipline

Now all you have to do is run this command to start the server:

```bash
docker compose up -d
```

### Updating Zipline

Pull the latest version of the image:

```bash
docker compose pull
```

Recreate and restart the container:

```bash
docker compose up -d
```

Clean up old images:

```bash
docker image prune -f
```
