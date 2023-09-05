# Matrix

> Work in Progress
{.is-info}


Create `docker-compose` file for your matrix

```bash
nano docker-compose.yml
```

and add following content

```yaml
version: '3.3'

services:
  app:
    image: matrixdotorg/synapse
    restart: always
    ports:
      - 8008:8008
    volumes:
      - /var/docker_data/matrix:/data
```

Configure it

```bash
docker run -it --rm -v /var/docker_data/matrix:/data -e SYNAPSE_SERVER_NAME=matrix.YOUR_DOMAIN -e SYNAPSE_REPORT_STATS=yes matrixdotorg/synapse:latest generate
```

Start Server

```bash
docker-compose up -d
```

`TODO` add configuration for Traefik