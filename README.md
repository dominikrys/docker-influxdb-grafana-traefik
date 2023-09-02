# Secure Metrics Solution in Docker

![Diagram](docs/diagram.png)

Docker Compose application for deploying [InfluxDB](https://www.influxdata.com/products/influxdb-overview/), [Grafana](https://grafana.com/), and [Traefik](https://containo.us/traefik/).

The individual components are:

- **InfluxDB**: time-series database.

- **Grafana**: front-end for visualizing and querying data in InfluxDB.

- **Traefik**: edge router/reverse proxy which will auto-generate and auto-renew TLS certificates using [Let's Encrypt](https://letsencrypt.org/). Traefik makes it so that data sent to and from Grafana and InfluxDB will be encrypted.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)

## How to Run Locally

Deploy the containers:

```bash
docker-compose up
```

Grafana will be accessible at [metrics.docker.localhost](http://metrics.docker.localhost) and InfluxDB at [metrics.docker.localhost:8086](http://metrics.docker.localhost:8086). Use the credentials in [.env](.env) to log in to Grafana. To write data to InfluxDB, refer to [the InfluxDB docs](https://docs.influxdata.com/influxdb/v2.7/write-data/).

Stop running containers:

```bash
docker-compose down
```

## Notes

- When accessing Grafana or InfluxDB that have been deployed locally, your browser and other apps may show warnings about invalid or self-signed TLS certificates. This is expected as localhost domains don't end with a valid top-level domain, so Traefik won't attempt to request a certificate for them.

- Grafana is accessible from the HTTP and HTTPS ports (`80` and `443` respectively), with redirection from HTTP to HTTPS handled using [Traefik routers](https://doc.traefik.io/traefik/routing/routers/).

- Grafana will automatically be set up with InfluxDB as a data source (set up under `grafana/provisioning/datasources/influxdb.yml`).

- InfluxDB will run shell scripts in `docker-entrypoint-initdb.d` on startup.

- If you're testing locally, and an application which you want to use to send data to InfluxDB can't be set to ignore TLS certificates, change the `traefik.http.routers.influxdb-ssl.tls` label to `false` for the InfluxDB container inside `docker-compose.yml`.

## Deploying in Production

- Set a secure password for Grafana and InfluxDB.

- Change the `METRICS_DOMAIN` environment variable in [`.env`](./.env) to the domain where the application will be hosted.

- Set the `LETS_ENCRYPT_EMAIL` environment variable in [`.env`](./.env) to a valid email that you wish to receive emails about [certificates issues to](https://cert-manager.io/docs/configuration/acme/#creating-a-basic-acme-issuer).

- Uncomment the appropriate `CA_SERVER` environment variable in [`.env`](./.env) to use [Let's Encrypt's](https://letsencrypt.org/) production API.

  > There is a limit of 5 certificates per week from Let's Encrypt's production server as stated [here](https://letsencrypt.org/docs/rate-limits/). For more info on the Let's Encrypt staging environment and Traefik, check the note under this [Traefik docs page](https://docs.traefik.io/v2.0/user-guides/docker-compose/acme-tls/#setup).

## Useful Commands

Check container logs

```bash
sudo docker container logs <CONTAINER NAME OR ID> [--follow]
```

Attach to a container and use bash within it (useful for InfluxDB database maintenance)

```bash
sudo docker exec -it <CONTAINER NAME OR ID> bash
```

Start up the InfluxDB CLI when attached to the InfluxDB docker container

```bash
influx --username <InfluxDB username> --password <InfluxDB password>
```

Delete running containers and their volumes

```bash
docker-compose down
```

Run containers in the background

```bash
docker-compose up -d
```
