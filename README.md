# Secure Monitoring Solution in Docker

![Diagram](docs/diagram.png)

Docker Compose application for deploying [Grafana](https://grafana.com/), [InfluxDB](https://www.influxdata.com/products/influxdb-overview/) and [Traefik](https://containo.us/traefik/) in Docker containers.

The individual components are:

- **Grafana**: front-end for visualizing and querying data.

- **InfluxDB**: time-series database.

- **Traefik**: edge router/reverse proxy which will auto-generate and auto-renew TLS certificates using [Let's Encrypt](https://letsencrypt.org/). This means that all data sent to and from Grafana and InfluxDB will be encrypted.

## Prerequisites

- [Docker Engine](https://docs.docker.com/engine/install/ubuntu/)

- [Docker Compose](https://docs.docker.com/compose/install/)

## How to run locally

Deploy the application:

```bash
sudo docker-compose up
```

You can then access Grafana at [monitoring.docker.localhost](http://monitoring.docker.localhost). InfluxDB will be listening to port `8086` by default.

> Note that when accessing Grafana or InfluxDB that have been deployed locally, your browser and other apps will complain about invalid or self-signed TLS certificates. This is expected as localhost domains don't end with a valid top-level domain, and therefore Traefik won't attempt to request a certificate for them.

Stop a running application:

```bash
sudo docker-compose down
```

## General info

- Most settings that can be tweaked are provided in `.env`.

  - Make sure to set a secure password for Grafana and InfluxDB! This can also be managed with [Docker secrets](https://docs.docker.com/engine/swarm/secrets/).

- After the initial deployment, the containers are set to restart automatically if they stop e.g. on a machine reboot. They can be stopped completely using `docker-compose down`.

- Grafana will automatically be set up with InfluxDB as a data source (set up under `grafana/provisioning/datasources/influxdb.yml`).

- InfluxDB will run shell scripts in `docker-entrypoint-initdb.d` on startup.

- Traefik has been set up to redirect HTTP to HTTPS using [routers](https://doc.traefik.io/traefik/routing/routers/).

- If testing locally and an application which you want to send data to InfluxDB can't be set to ignore TLS certificates, change the `traefik.http.routers.influxdb-ssl.tls` label to `false` for the InfluxDB container inside `docker-compose.yml`.

## Deploying in production

- Change the `MONITORING_DOMAIN` environment variable to the domain where the application will be hosted.

- Set the `certificatesResolvers.lets-encrypt-ssl.acme.email` label in `docker-compose.yml` to a valid email.

- Uncomment the appropriate `CA_SERVER` environment variable in `.env` to use [Let's Encrypt's](https://letsencrypt.org/) production API.

    > There is a limit of 5 certificates per week from Let's Encrypt's production server as stated [here](https://letsencrypt.org/docs/rate-limits/). For more info on the Let's Encrypt staging environment and Traefik, check the note under this [Traefik docs page](https://docs.traefik.io/v2.0/user-guides/docker-compose/acme-tls/#setup).

- Deploy as you would locally, ideally in a detached state.

  ```bash
  sudo docker-compose up -d
  ```

## Useful commands

Check container logs

```bash
sudo docker container logs <CONTAINER NAME OR ID> [--follow]
```

Check where data is stored (Docker volumes)

```bash
$ sudo docker volume ls

DRIVER          VOLUME NAME
local           monitoring_grafana-lib
local           monitoring_influxdb-lib
local           monitoring_traefik-data
```

Attach to a container and use bash within it (useful for InfluxDB database maintenance)

```bash
sudo docker exec -it <CONTAINER NAME OR ID> /bin/bash
```

Start up the the InfluxDB CLI when attached to the InfluxDB docker container

```bash
influx --username <InfluxDB username> --password <InfluxDB password>
```

Check space used by Docker containers

```bash
sudo docker system df --verbose
```

## Links

- [How to backup and restore InfluxDB database from Docker containers](https://www.influxdata.com/blog/backuprestore-of-influxdb-fromto-docker-containers/)
