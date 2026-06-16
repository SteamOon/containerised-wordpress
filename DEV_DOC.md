# Developer documentation

## 1) Prerequisites
- Install Docker and Docker Compose (or Docker Compose V2 via `docker compose`).
- `make`, `git`, and `sudo` access (some cleanup commands use `sudo`).

## 2) Configuration files and secrets
- Copy the example env file and edit values:

```bash
cp .env.example .env
# edit .env with DB / WP values
```

- Main Compose file: [srcs/docker-compose.yml](srcs/docker-compose.yml)
- WordPress configuration template: [srcs/requirements/wordpress/conf/wp-config.php](srcs/requirements/wordpress/conf/wp-config.php)

## 3) Build and launch (Makefile)
- Build images and prepare host data directories:

```bash
make build
```

- Start the whole stack in detached mode:

```bash
make up
```

- Stop the stack:

```bash
make down
```

- Rebuild from scratch and remove data:

```bash
make fclean
make all
```

## 4) Useful container and volume commands
- View running containers:

```bash
docker compose -f srcs/docker-compose.yml ps
```

- Follow logs:

```bash
docker compose -f srcs/docker-compose.yml logs -f
```

- Enter a running container (example `wordpress` service):

```bash
docker compose -f srcs/docker-compose.yml exec wordpress /bin/bash
```

- Remove stopped containers and unused images:

```bash
docker system prune -af
docker volume prune -f
```

## 5) Where project data is stored and persistence
- The Makefile creates host directories for persistent data. By default the Makefile sets `DATA_DIR = /home/$(USER)/data` where `USER` is defined at the top of the Makefile (for example `/home/steven/data`). The two directories created are:

- `/home/<user>/data/database` — MariaDB data files (database persistence)
- `/home/<user>/data/wordpress_files` — WordPress uploads and site files

- The compose file mounts these host folders (and/or docker volumes) into the containers so data survives container rebuilds. To move or back up persistent data, stop the stack and copy these host directories.

## 6) Configuration and setup files (per service)

This section explains the purpose of the main configuration and setup files for each container. See the referenced files under `srcs/requirements/` for implementation details.

- **Docker Compose & environment**
	- `srcs/docker-compose.yml`: orchestrates the services, networks, volumes, build contexts, `env_file` references and service dependencies.
	- `srcs/.env.example`: example environment file. Copy to `srcs/.env` or use your own `.env` to provide `DB_*`, `WP_*`, and SSL variables consumed by services.

- **MariaDB**
	- `srcs/requirements/mariadb/Dockerfile`: builds the MariaDB image and sets up any container-specific packaging or entrypoint steps.
	- `srcs/requirements/mariadb/conf/50-server.cnf`: custom MariaDB configuration (connection limits, timeouts, etc.). This file is mounted or copied into the image to override default MySQL/MariaDB settings.
	- `srcs/requirements/mariadb/tools/init_db.sh`: initialization script run at container startup to create the database, users, and apply initial grants if required. The script reads credentials from environment variables (`DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_PASS_ROOT`).
	- Data volume: the compose file mounts the `database` volume to `/var/lib/mysql` so database files persist on the host.

- **WordPress (PHP / PHP-FPM)**
	- `srcs/requirements/wordpress/Dockerfile`: builds the PHP + PHP-FPM image, installs PHP extensions, and prepares the webroot.
	- `srcs/requirements/wordpress/conf/wp-config.php`: WordPress configuration template that reads database credentials and other settings (DB_HOST, DB_USER, DB_PASSWORD, WP salts and URLs) from environment variables. The build or startup step copies or generates a working `wp-config.php` in `/var/www/inception`.
	- `srcs/requirements/wordpress/conf/www.conf`: PHP-FPM pool configuration (user/group, process manager settings, socket/port configuration).
	- `srcs/requirements/wordpress/tools/wp_setup.sh`: helper script using WP-CLI to install WordPress core, create the admin user, set options, and perform any post-install tasks. It relies on the environment variables declared in `srcs/.env`.
	- Volume: `wordpress_files` is mounted to `/var/www/inception` so themes, uploads and plugins persist.

- **Nginx**
	- `srcs/requirements/nginx/Dockerfile`: builds the nginx image and can include certificate generation steps or copy configuration files.
	- `srcs/requirements/nginx/conf/nginx.conf`: main nginx configuration (events, http blocks, include directives).
	- `srcs/requirements/nginx/conf/server.conf`: site-specific server block that defines `server_name`, SSL configuration (paths passed via build args / env), document root (`/var/www/inception`), and proxying rules to PHP-FPM where applicable.
	- SSL build args: the compose file passes `CERT_FOLDER`, `CERTIFICATE`, `KEY`, and other metadata as build args so `nginx` can be built with the expected certificate paths.

Notes:
- The services rely on environment variables declared in `srcs/.env` (or your own `.env`) to configure runtime credentials and behavior.
- For development the project uses bind mounts to host directories (`~/data/*`) for easy inspection and backups. In production you may prefer named volumes or external managed storage.


