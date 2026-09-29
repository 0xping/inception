# Inception

A WordPress site served over HTTPS by three containers I built from plain Debian images, wired together with Docker Compose. System administration project from 1337 (42 Network).

## What it does
- **NGINX** is the only public entry point: port 443, **TLS 1.3 only**, and it forwards `.php` requests to WordPress over FastCGI.
- **WordPress + PHP-FPM 7.4** listens on port 9000 inside the network. It's installed and configured on container start with WP-CLI: downloads core, writes `wp-config.php`, creates the admin and an author user.
- **MariaDB** creates the database and user on startup, then runs `mysqld` in the foreground.
- Each service has its own Dockerfile, based on `debian:bullseye`. No prebuilt official images.
- A private bridge network (`inception-net`) and named volumes for the database and site files, so data survives container rebuilds. The volumes bind to hardcoded host paths, `/home/aait-lfd/data/db` and `/home/aait-lfd/data/wordpress`.
- NGINX generates a self-signed TLS certificate on container start if one isn't already present.
- `restart: always` on every service.
- All credentials come from an `.env` file that isn't in the repo.

## How it works
```
browser ──443/TLS──▶ nginx ──FastCGI :9000──▶ wordpress (php-fpm) ──3306──▶ mariadb
                       │                           │                          │
                       └──── volume: wordpress ────┘                  volume: db
```

## Build & run
Create `srcs/.env` with `LOGIN`, `CERTS`, `CERTS_KEY`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `WP_TITLE`, `WP_ADMIN_NAME`, `WP_ADMIN_PASSWORD`, `WP_ADMIN_EMAIL`, `WORDPRESS_USER`, `WORDPRESS_USER_EMAIL` and `WORDPRESS_USER_PASSWORD`. Add `127.0.0.1 <LOGIN>.42.fr` to `/etc/hosts` (the site is configured with that URL). Then:

```sh
make        # build images and start the stack
```
Warning: `make fclean` removes **every** Docker container, image, network and volume on the machine, not just this project's. Run it in a VM.

`make re` runs `fclean` first, then `all`: the same warning applies.

## What I learned
- How NGINX talks to PHP-FPM over FastCGI, and why the web server and the app runtime are separate processes.
- Writing Dockerfiles from a bare OS image, and keeping one process in the foreground so the container stays alive (PID 1).
- Container networking and service discovery by name (`wordpress:9000`, `dbhost=mariadb`).
- Named volumes versus the container lifecycle, and keeping secrets out of images with an env file.
