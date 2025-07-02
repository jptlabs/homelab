# Jellyfin setup guide with hardware accelerated transcoding

> [!NOTE]
> For a more-in-depth guide read the official documentation [here](https://jellyfin.org/docs/general/installation/container/).
>

## Installing with Docker Compose

```yml
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    network_mode: 'host'
    devices:
      - /dev/dri:/dev/dri # For hw accelerated transcoding with a gpu
    volumes:
      - /docker_volumes/jellyfin/config:/config
      - /docker_volumes/jellyfin/cache:/cache
      - type: bind
        source: /path/to/your/media
        target: /media
        read_only: false
      # - type: bind      
      #   source: /path/to/your/music
      #   target: /music
      #   read_only: false
      # Optional - if you choose to use jellyfin as a music server as well, jellyfin needs rw access to create playlists
    restart: 'unless-stopped'
 #   environment:
#      - JELLYFIN_PublishedServerUrl=https://host.domain.tld
    # Optional - may be necessary for docker healthcheck to pass if running in host network mode
    extra_hosts:
      - 'host.docker.internal:host-gateway'
```

Create a docker-compose.yml file and paste the contents in, or run this command:

```bash
wget https://raw.githubusercontent.com/jptlabs/homelab/main/docker/jellyfin/docker-compose.yml
```

Configure ports, volumes, or anything else as needed.

> [!NOTE]
> Make sure that you change `/path/to/your/media` to wherever your media is stored. Jellyfin may need rw access if you want trickplay images.
> 

### Starting Jellyfin

Now all you have to do is run this command to start the server:

```bash
docker compose up -d
```

You should now be able to access jellyfin through `http://localhost:8096` or the port you specified.

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
# HTTPS Proxy for Jellyfin with HTTP to HTTPS redirection
http:
  middlewares:
    https-redirectscheme:
      redirectScheme:
        scheme: https
        permanent: true
  routers:
    jellyfin-redirect:
      entryPoints:
        - "http"
      rule: "Host(`jellyfin.domain.tld`)"
      middlewares:
        - https-redirectscheme
      service: noop@internal # Dummy service to handle redirection
    jellyfin:
      entryPoints:
        - "https"
      rule: "Host(`jellyfin.domain.tld`)"
      tls: {}
      service: jellyfin
  services:
    jellyfin:
      loadBalancer:
        servers:
          - url: "http://hostip:8096"
        passHostHeader: true
```

Alternatively you can use [labels](https://doc.traefik.io/traefik/providers/docker/#routing-configuration-with-labels) if jellyfin runs on the same instance as traefik.