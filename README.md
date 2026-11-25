# Inception Project

*This project has been created as part of the 42 curriculum by smoon.*

## Description

**Inception** is a Docker containerization project that deploys a complete WordPress infrastructure using Docker Compose. The project demonstrates containerization best practices by orchestrating three core services:

- **Nginx**: Reverse proxy and web server handling HTTPS traffic
- **WordPress**: PHP application running on PHP-FPM
- **MariaDB**: Relational database for WordPress persistence

The primary goal is to understand Docker containerization, networking, data persistence, and the orchestration of multi-service applications. This project showcases how modern applications are deployed in isolated, reproducible environments that can be easily scaled and managed.

## Configuration

For detailed setup instructions and commands, refer to the documentation:

- **Users**: See [USER_DOC.md](USER_DOC.md) for how to start, stop, and access the application
- **Developers**: See [DEV_DOC.md](DEV_DOC.md) for prerequisites, configuration, and development workflows

Configuration example: Copy `srcs/.env.example` to `srcs/.env` and customize as needed.

## Quick setup

Create `srcs/.env` using `srcs/.env.example`

Run `make`

## Project Architecture & Docker Usage

### Project Structure

```
srcs/
├── docker-compose.yml          # Main orchestration file
└── requirements/
    ├── mariadb/                # MariaDB service
    │   ├── Dockerfile
    │   ├── conf/
    │   │   └── 50-server.cnf
    │   └── tools/
    │       └── init_db.sh
    ├── nginx/                  # Nginx reverse proxy
    │   ├── Dockerfile
    │   └── conf/
    │       ├── nginx.conf
    │       └── server.conf
    └── wordpress/              # WordPress application
        ├── Dockerfile
        ├── conf/
        │   ├── wp-config.php
        │   └── www.conf
        └── tools/
            └── wp_setup.sh
```

### Design Choices

1. **Docker Volumes with Bind Mounts**: Persistent data is stored in host directories (`~/data/`) and mounted into containers using bind mounts acting within named volumes, allowing data to survive container lifecycle changes and data to be easily accessed by the user.

2. **Docker Compose Orchestration**: A single `docker-compose.yml` file manages all services, their dependencies, networking, and configuration.

3. **Environment Variables**: Sensitive and configurable data (database credentials, WordPress settings, SSL paths) are managed via `.env` file, allowing easy configuration without changing code.

4. **Docker Network (Bridge)**: All services communicate through a custom bridge network (`inception-network`), providing:
   - Service discovery via container names
   - Isolation from the host network
   - Easy container-to-container communication

5. **HTTPS/SSL Support**: Nginx is configured with SSL certificates, enabling secure communication with the application.

6. **Service Dependencies**: The compose file defines dependency relationships (nginx depends on wordpress, wordpress depends on mariadb), ensuring services start in the correct order.

## Technical Comparisons

### Virtual Machines vs Docker

| Aspect | Virtual Machines | Docker |
|--------|-----------------|--------|
| **Overhead** | Heavy (full OS per VM) | Lightweight (shared kernel) |
| **Startup Time** | Minutes | Seconds |
| **Resource Usage** | ~1-2GB per VM | ~10-50MB per container |
| **Isolation** | Complete (separate kernel) | Process-level (shared kernel) |
| **Use Case** | Full OS customization needed | Application packaging & deployment |
| **Portability** | VM files are large & platform-specific | Container images are portable |

**Choice**: Docker was selected for Inception because it provides sufficient isolation for development/testing while being lightweight and portable—ideal for multi-service application deployment.

### Secrets vs Environment Variables

| Aspect | Environment Variables | Docker Secrets |
|--------|----------------------|-----------------|
| **Visibility** | Visible in container `env` and process `ps` | Not visible in container environment |
| **Rotation** | Requires container restart | Can change without restart |
| **Storage** | In `.env` file or passed at runtime | In `/run/secrets` (swarm/k8s) |
| **Scope** | Container-wide access | File-based, must be explicitly read |
| **Complexity** | Simple, no extra setup | Requires Docker Swarm or Kubernetes |

**Choice**: This project uses **environment variables** (.env file) for simplicity and because it's sufficient for development/testing. In production, Docker Secrets (with Swarm) or Kubernetes Secrets would be preferred for better security and secrets rotation.

### Docker Network vs Host Network

| Aspect | Bridge Network (Custom) | Host Network |
|--------|-------------------------|-------------|
| **Isolation** | Services isolated from host | Services directly on host network |
| **Service Discovery** | Container name resolution | Must use IP addresses directly |
| **Port Mapping** | Required to expose ports | Ports directly accessible |
| **Security** | Contained to custom network | Exposed to host network directly |
| **Performance** | Slight overhead from virtual interface | Minimal overhead |

**Choice**: **Bridge network** (`inception-network`) was chosen because it:
- Provides service discovery (containers communicate via service names)
- Isolates services from the host network
- Allows explicit port mapping for control
- Is the standard for production deployments

### Docker Volumes vs Bind Mounts

| Aspect | Named Volumes | Bind Mounts |
|--------|---------------|------------|
| **Performance** | Optimized for containers | Dependent on host filesystem |
| **Portability** | Docker-managed, portable | Path-dependent, less portable |
| **Permissions** | Docker handles permissions | Host filesystem permissions apply |
| **Backup** | Via Docker commands | Manual or script-based |
| **Ease of Use** | Simple, no host path management | More control, but requires setup |
| **Host Access** | Limited direct access | Full filesystem access |

**Choice**: **Bind mounts** were chosen for Inception because:
- Direct access to persistent data on the host (`~/data/`) for backup/inspection
- Fine-grained control over storage location
- Easier debugging during development
- Explicit visibility of where data lives

For production, named volumes would be preferred for better portability and Docker management.

## Resources

### Docker & Containerization

- [Docker Official Documentation](https://docs.docker.com/)
- [University of Helsinki - DevOps with Docker course](https://courses.mooc.fi/org/uh-cs/courses/devops-with-docker)

### WordPress & PHP

- [WordPress Official Documentation](https://wordpress.org/documentation/)
- [WordPress with PHP-FPM Setup](https://wordpress.org/support/article/how-to-install-wordpress/)
- [PHP-FPM Configuration](https://www.php.net/manual/en/install.fpm.php)

### Nginx & MariaDB

- [Nginx Official Documentation](https://nginx.org/en/docs/)
- [MariaDB Official Documentation](https://mariadb.com/documentation/)
- [Bogdan Caraman: install Nginx & MariaDB](https://blog.bogdancaraman.com/install-nginx-mariadb-php-fpm-debian-13-cli/)


### Tasks Supported by AI

**Documentation Structure & Content**: AI helped organize and structure the README, USER_DOC and DEV_DOC to meet project requirements, including writing clear descriptions of Docker concepts and design choices and set

**Debugging**: AI was used to identify bugs in set up e.g. misaligned variable names

