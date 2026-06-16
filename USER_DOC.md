# User documentation

## Services provided
- WordPress application (PHP + php-fpm)
- MariaDB database
- Nginx web server (reverse proxy / static files)

## Start and stop the project
- Build images and create host data directories:

```bash
make build
```

- Start services in detached mode:

```bash
make up
```

- Stop services:

```bash
make down
```

- Full wipe (containers, images, volumes and project data):

```bash
make fclean
```

Notes: `make` targets use the Compose file at [srcs/docker-compose.yml](srcs/docker-compose.yml).

## Accessing the website and admin panel
- Open the site in a browser at the address and port configured in [srcs/docker-compose.yml](srcs/docker-compose.yml). By default the stack exposes the web service through the host network ports defined in the compose file (typically `http://localhost/`).
- WordPress admin panel is available at `/wp-admin` (for example `http://localhost/wp-admin`).

## Locating and managing credentials
- Environment variables and runtime configuration are defined in the project root files `.env` and `.env.example`. Copy `.env.example` to `.env` and fill values before launching.
- Secrets (if any) are stored under the `secrets/` directory at the repository root — check there for any private files your deployment expects.
- Database credentials used by WordPress are set via environment variables referenced by `wp-config.php` (see [srcs/requirements/wordpress/conf/wp-config.php](srcs/requirements/wordpress/conf/wp-config.php)).

## Checking services are running correctly
- View container status:

```bash
docker compose -f srcs/docker-compose.yml ps
```

- Follow logs for a specific service (replace `service_name`):

```bash
docker compose -f srcs/docker-compose.yml logs -f service_name
```

- Check Nginx/HTTP response:

```bash
curl -I http://localhost/
```

- Check WordPress with WP-CLI inside the php container:

```bash
docker exec -it mariadb /bin/bash
mysql -u your_user -p db_name
```

