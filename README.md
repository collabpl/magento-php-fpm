# magento-php-fpm

Custom PHP-FPM Docker images for Magento 2, built from the official `php:X.X-fpm` base.
Includes a comprehensive set of PHP extensions required by Magento, PCOV for code
coverage, and [mhsendmail](https://github.com/mailhog/mhsendmail) for local email
interception via MailHog.

Available on:
- **Docker Hub**: [`collabpl/magento-php-fpm`](https://hub.docker.com/r/collabpl/magento-php-fpm)
- **GitLab Registry**: `gitlab-registry.collab.pl/collab/docker/magento-php-fpm`

## Available tags

| Tag | PHP | Base image |
|-----|-----|-----------|
| `8.3` | 8.3 | `php:8.3-fpm` |
| `8.4` | 8.4 | `php:8.4-fpm` |
| `8.5` | 8.5 | `php:8.5-fpm` |

## What's included

### PHP extensions

**Built-in**: `bcmath`, `bz2`, `calendar`, `exif`, `gd`, `gettext`, `gmp`, `intl`,
`mysqli`, `opcache`, `pdo_mysql`, `pspell`, `shmop`, `soap`, `sockets`, `sysvmsg`,
`sysvsem`, `sysvshm`, `tidy`, `xsl`, `zip`, `pcntl`, `ftp`, `ldap`

**PECL**: `gnupg`, `mailparse`, `msgpack`, `oauth`, `pcov`, `raphf`, `redis`,
`xmlrpc`, `yaml`, `libsodium`

### Other

- **mhsendmail** — MailHog sendmail drop-in, compiled from source during the build
- **www** user — UID/GID `1000`, owns `/var/www/html`
- **PHP-FPM pool** — dynamic, listening on `:9000`, logs to stdout/stderr

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PHP_MEMORY_LIMIT` | `2G` | `memory_limit` in php.ini |
| `PHP_VALIDATE_TIMESTAMPS` | `1` | `opcache.validate_timestamps` |
| `DEBUG` | `false` | Set to `true` to enable `bash -x` in the entrypoint |
| `MAGENTO_RUN_MODE` | `production` | Injected into the PHP-FPM pool as `MAGE_MODE` |
| `UPLOAD_MAX_FILESIZE` | `64M` | Max file upload size |
| `SENDMAIL_PATH` | `/dev/null` | Set to `/usr/local/bin/mhsendmail` for MailHog |
| `PHPRC` | `${MAGENTO_ROOT}/php.ini` | Path to the active php.ini |
| `PHP_EXTENSIONS` | (see Dockerfile) | Space-separated list of extensions enabled at startup |

## Usage

### Docker Hub

```bash
docker pull collabpl/magento-php-fpm:8.4
```

### GitLab Registry

```bash
docker pull gitlab-registry.collab.pl/collab/docker/magento-php-fpm:8.4
```

### docker-compose example

```yaml
services:
  php-fpm:
    image: collabpl/magento-php-fpm:8.4
    environment:
      PHP_MEMORY_LIMIT: "4G"
      MAGENTO_RUN_MODE: developer
      SENDMAIL_PATH: "/usr/local/bin/mhsendmail --smtp-addr=mailhog:1025"
    volumes:
      - ./src:/var/www/html
```

### With MailHog for local email testing

```yaml
services:
  php-fpm:
    image: collabpl/magento-php-fpm:8.4
    environment:
      SENDMAIL_PATH: "/usr/local/bin/mhsendmail --smtp-addr=mailhog:1025"

  mailhog:
    image: mailhog/mailhog
    ports:
      - "8025:8025"
```

## Repository structure

```
.
├── conf/etc/                  # shared PHP/FPM config (all versions)
│   ├── php-fpm.conf           # PHP-FPM pool config
│   ├── php-fpm.ini            # PHP settings (memory, opcache)
│   ├── mail.ini               # sendmail_path
│   ├── php-pcov.ini           # PCOV config
│   └── php-gnupg.ini          # gnupg extension
├── docker-entrypoint.sh       # shared entrypoint (all versions)
├── 8.3/
│   └── Dockerfile
├── 8.4/  (same structure)
├── 8.5/  (same structure)
├── .github/
│   └── workflows/             # GitHub Actions — build & push to Docker Hub
└── .gitlab-ci.yml             # GitLab CI — build via Kaniko
```

## CI/CD

**GitHub Actions** builds for both `linux/amd64` and `linux/arm64` and pushes to
Docker Hub on every push to `main` and on published releases. Requires
`DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` repository secrets.

**GitLab CI** uses the internal
[`kaniko-build`](https://gitlab.collab.pl/collab/components/kaniko-build)
component to build all three variants in parallel and push to the GitLab Container Registry.

## Adding a new PHP version

1. Create a new `{version}/` directory by copying an existing one.
2. Update the `FROM php:{version}-fpm` line in the Dockerfile.
3. Add the new tag to the `.gitlab-ci.yml` parallel matrix.
4. Create a new GitHub Actions workflow `.github/workflows/build-php-fpm-{version}.yml`.
5. Push — both pipelines will build and publish the new image automatically.

## License

MIT
