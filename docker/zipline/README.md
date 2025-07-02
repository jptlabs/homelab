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
      - /docker_volumes/zipline/pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD', 'pg_isready', '-U', 'zipline']
      interval: 10s
      timeout: 5s
      retries: 5

  zipline:
    image: ghcr.io/diced/zipline:latest
    restart: unless-stopped
    ports:
      - '3000:3000'
    env_file:
      - .env
    environment:
      - DATABASE_URL=postgres://${POSTGRESQL_USER:-zipline}:${POSTGRESQL_PASSWORD}@postgresql:5432/${POSTGRESQL_DB:-zipline}
    depends_on:
      - postgresql
    volumes:
      - '/path/to/your/share:/zipline/uploads'
      - '/docker_volumes/zipline/public:/zipline/public'
      - '/docker_volumes/zipline/themes:/zipline/themes'

```

Create a docker-compose.yml file and paste the contents in, or run this command:

```bash
wget https://raw.githubusercontent.com/jptlabs/homelab/main/docker/zipline/docker-compose.yml
```

Configure ports, volumes, or anything else as needed.

> [!NOTE]
> Make sure that you change `/path/to/your/share` to wherever you want uploads to be stored. This can be a local folder or an smb/nfs share.
> 

### Generating Secrets

Running this command will generate the .env file.

```bash
echo "POSTGRESQL_PASSWORD=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" > .env
echo "CORE_SECRET=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" >> .env
```

Without these variables zipline won't run. Alternatively you can create these values yourself.

### Starting Zipline

Now all you have to do is run this command to start the server:

```bash
docker compose up -d
```

You should now be able to access zipline through `http://localhost:3000` or the port you specified.

## Updating with Docker Compose

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

## Traefik configuration

```yaml
# HTTPS Proxy for Zipline with HTTP to HTTPS redirection
http:
  middlewares:
    https-redirectscheme:
      redirectScheme:
        scheme: https
        permanent: true
  routers:
    zipline-redirect:
      entryPoints:
        - "http"
      rule: "Host(`zipline.domain.tld`)"
      middlewares:
        - https-redirectscheme
      service: noop@internal # Dummy service to handle redirection
    zipline:
      entryPoints:
        - "https"
      rule: "Host(`zipline.domain.tld`)"
      tls: {}
      service: zipline
  services:
    zipline:
      loadBalancer:
        servers:
          - url: "http://hostip:3000"
        passHostHeader: true
```

Alternatively you can use [labels](https://doc.traefik.io/traefik/providers/docker/#routing-configuration-with-labels) if Zipline runs on the same instance as Traefik.