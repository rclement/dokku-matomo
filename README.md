# Matomo on Dokku

Deploy your own instance of [Matomo](https://matomo.org) on
[Dokku](https://github.com/dokku/dokku)!

This setup is makes use of the great ready-to-use
[Matomo Docker image](https://github.com/crazy-max/docker-matomo) from @crazy-max,
which does all of the heavy-lifting to properly deploy Matomo without too much headache.

Current Matomo version: 3.5.1

# Setup

## Requirements

Be sure to properly setup a [Dokku](https://github.com/dokku/dokku) instance.

The following Dokku plugins need to be installed:

- [dokku-mariadb](https://github.com/dokku/dokku-mariadb)
- [dokku-letsencrypt](https://github.com/dokku/dokku-letsencrypt)

## App and database

1. Create the `matomo` app:

```
    dokku apps:create matomo
```

2. Create the mariadb database:

```
    dokku mariadb:create matomo
    dokku mariadb:link matomo matomo
```

## Setup app

1. Persistent storage:

First, from the host:

```
    sudo mkdir -p /var/lib/dokku/data/storage/matomo
    sudo chown -R 32767 /var/lib/dokku/data/storage/matomo
```

Then, remotely:

```
    dokku storage:mount /var/lib/dokku/data/storage/matomo:/data
```

2. Configuration:

```
    dokku config:set --no-restart TZ="Europe\/Paris"
    dokku config:set --no-restart LOG_LEVEL=WARN
    dokku config:set --no-restart MEMORY_LIMIT=256M
    dokku config:set --no-restart UPLOAD_MAX_SIZE=16M
    dokku config:set --no-restart OPCACHE_MEM_SIZE=128
    dokku config:set --no-restart SSMTP_HOST=smtp.example.com
    dokku config:set --no-restart SSMTP_PORT=587
    dokku config:set --no-restart SSMTP_HOSTNAME=matomo.dokku-apps.me
    dokku config:set --no-restart SSMTP_USER=user@example.com
    dokku config:set --no-restart SSMTP_PASSWORD=
    dokku config:set --no-restart SSMTP_TLS=YES
    dokku config:set --no-restart ARCHIVE_OPTIONS="--concurrent-requests-per-website=3"
    dokku config:set --no-restart CRON_GEOIP="0\ 2\ \*\ \*\ \*"
    dokku config:set --no-restart CRON_ARCHIVE="0\ \*\ \*\ \*\ \*"
```

3. Fix proxy ports:

```
    dokku proxy:ports-add http:80:80
    dokku proxy:ports-remove http:80:5000
```

4. Domain name (optional):

```
    dokku domains:set matomo.dokku-apps.me
```

5. HTTPS support (optional, but highly recommended when possible):

```
    dokku config:set --no-restart matomo DOKKU_LETSENCRYPT_EMAIL=user@example.com
    dokku letsencrypt matomo
```

# Deploy

1. Clone this repository:

```
    git clone https://github.com/rclement/dokku-matomo.git
```

2. Setup Dokku git remote (with your defined domain):

```
    git remote add dokku dokku@example.com:matomo
```

3. Push `matomo`:

```
    git push dokku master
```

# Run

`matomo` should be running at `http://matomo.dokku-apps.me` or any domain previously specified.
Also, archiving and GeoIP update cron tasks will be running in background.

# Post-install configuration

Some extra steps might be required after the deployment:

## Installation wizard

The following features can be configured during the first run:

- Database config (use the configuration from `dokku config:get matomo DATABASE_URL`)
- Initial user config
- Initial site config

## Archiving

- Disable "Archive reports when viewed from the browser"

## GeoIP2

- Replace all variables "MM_*" with "GEOIP_*"
- Enable "GeoIP (HTTP Server Module)"

## Advanced configuration

If needed, the Matomo configuration file is located at `/var/lib/dokku/data/storage/matomo/config/config.ini.php`
and can be manually edited.
