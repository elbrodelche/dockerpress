# DockerPress: WordPress Local Development Stack

A Docker Compose based WordPress development environment with:

- WordPress (custom image with PHP 8.3 + SOAP extension)
- MySQL 8.4 (LTS)
- phpMyAdmin
- WP-CLI
- Composer

This repository is optimized for local development on macOS/Linux.

## Stack Versions

Current pinned images and build base:

- WordPress base image: `wordpress:php8.3-apache` (built from [wordpress/Dockerfile](wordpress/Dockerfile))
- WP-CLI: `wordpress:cli-php8.3`
- MySQL: `mysql:8.4`
- phpMyAdmin: `phpmyadmin:5-apache`
- Composer: `composer:2`

## Service Map

- `wp`: main WordPress app on `${IP}:80`
- `db`: MySQL database on `${IP}:3306`
- `pma`: phpMyAdmin UI on `${IP}:8080`
- `wpcli`: WP-CLI one-off commands (no published port)
- `composer`: Composer one-off commands (no published port)

## Prerequisites

- Docker Engine installed and running
- Docker Compose plugin installed (`docker compose`)
- Git (optional, but recommended)

Verify:

```bash
docker --version
docker compose version
```

## Project Structure

- [docker-compose.yml](docker-compose.yml): all services and volumes
- [wordpress/Dockerfile](wordpress/Dockerfile): WordPress image customization
- [config/php.conf.ini](config/php.conf.ini): PHP overrides for uploads/memory/timeouts
- [cli/export.sh](cli/export.sh): exports DB dump to `wp-data/`
- [cli/setup-hosts-file.sh](cli/setup-hosts-file.sh): add/remove local hostnames
- [cli/create-cert.sh](cli/create-cert.sh): create a local TLS cert
- [cli/trust-cert.sh](cli/trust-cert.sh): trust cert in macOS keychain

## Quick Start (New Project)

- Clone the repository and enter it.

```bash
git clone <your-repo-url> dockerpress
cd dockerpress
```

- Create your environment file.

```bash
cp .env.dist .env
```

- Edit `.env` with secure credentials.

```dotenv
IP=127.0.0.1
DB_ROOT_PASSWORD=change-this-root-password
DB_NAME=wordpress
DB_USER=wordpress
DB_PASSWORD=change-this-app-password
```

- Start the stack.

```bash
docker compose up -d --build
```

- Open the app:

- WordPress: `http://127.0.0.1`
- phpMyAdmin: `http://127.0.0.1:8080`

- Complete WordPress installation in the browser.

## Environment Variables

Required values in `.env`:

- `IP`: bind address for published ports (default `127.0.0.1`)
- `DB_ROOT_PASSWORD`: MySQL root password (admin only)
- `DB_NAME`: application database name
- `DB_USER`: non-root WordPress DB user
- `DB_PASSWORD`: non-root WordPress DB password

Recommendation:

- Keep `IP=127.0.0.1` for local-only exposure.
- Use long random passwords for `DB_ROOT_PASSWORD` and `DB_PASSWORD`.

## Common Operations

### Start

```bash
docker compose up -d
```

### Stop

```bash
docker compose stop
```

### Restart

```bash
docker compose restart
```

### View Logs

```bash
docker compose logs -f
```

### Rebuild WordPress Image

Use this after changing [wordpress/Dockerfile](wordpress/Dockerfile):

```bash
docker compose build wp
docker compose up -d
```

### Remove Containers (Keep DB Volume)

```bash
docker compose down
```

### Remove Containers and DB Volume (Destructive)

```bash
docker compose down -v
```

## WP-CLI Usage

Run commands through the `wpcli` service:

```bash
docker compose run --rm wpcli plugin list
docker compose run --rm wpcli theme list
```

Install WordPress from CLI example:

```bash
docker compose run --rm wpcli core install \
  --url=http://localhost \
  --title="Local WordPress" \
  --admin_user=admin \
  --admin_email=admin@example.com
```

Optional shell alias:

```bash
alias wp='docker compose run --rm wpcli'
```

Then:

```bash
wp plugin list
```

## Composer Usage

Run composer in the app directory (`wp-app`):

```bash
docker compose run --rm composer --version
docker compose run --rm composer require wpackagist-plugin/query-monitor
```

## Database Backup and Restore

### Export (Backup)

Creates a SQL dump in `wp-data/` using [cli/export.sh](cli/export.sh):

```bash
./cli/export.sh
```

### Restore

- Stop stack and clear existing DB volume:

```bash
docker compose down -v
```

- Place your `.sql` file into `wp-data/`.

- Start again:

```bash
docker compose up -d
```

MySQL imports files in `wp-data/` during initialization.

## Using Existing WordPress Source

1. Place your existing WordPress files in `wp-app/`.
2. Place your DB dump in `wp-data/`.
3. Start stack:

```bash
docker compose up -d
```

If needed, force URLs in [wp-app/wp-config.php](wp-app/wp-config.php):

```php
define('WP_HOME', 'http://wp-app.local');
define('WP_SITEURL', 'http://wp-app.local');
```

## Local Domain and HTTPS Helpers (macOS)

Scripts are in [cli](cli).

### Add/Remove Host Entry

Interactive script:

```bash
./cli/setup-hosts-file.sh
```

### Create Local Certificate

Run from `cli/` directory because cert output paths are relative:

```bash
cd cli
./create-cert.sh
```

This creates cert/key files in `certs/`.

### Trust Certificate in macOS Keychain

Also run from `cli/` directory:

```bash
cd cli
./trust-cert.sh
```

## Security Notes

- Images are pinned to major/minor tags for predictable behavior.
- For maximum reproducibility, pin image digests in [docker-compose.yml](docker-compose.yml).
- Do not commit `.env` with real passwords.
- Keep Docker Desktop / Engine and images updated regularly:

```bash
docker compose pull --ignore-buildable
docker compose build wp
docker compose up -d
```

## Troubleshooting

### Containers do not start

```bash
docker compose ps
docker compose logs -f db
docker compose logs -f wp
```

### Port already in use

Change `IP` in `.env` or stop the conflicting service.

### Database connection errors in WordPress

Check:

- `.env` credentials (`DB_NAME`, `DB_USER`, `DB_PASSWORD`)
- db health status
- WordPress logs

```bash
docker compose ps
docker compose logs -f db
docker compose logs -f wp
```

### phpMyAdmin login fails

- Host: `db`
- User: `root` (or `DB_USER`)
- Password: from `.env`

### Clean reset

Warning: this removes DB data.

```bash
docker compose down -v
rm -rf wp-app
mkdir -p wp-app wp-data
docker compose up -d --build
```

## Development Tips

- Keep plugins/themes in version control in your host project and mount them into `wp-app/wp-content`.
- Use WP-CLI for repeatable setup steps.
- Use separate `.env` values per project if you run multiple stacks.

## License

See [LICENSE](LICENSE).
