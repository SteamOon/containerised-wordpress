FROM	debian:bookworm-slim

# Install mariadb & remove package list
RUN	apt-get update && apt-get install -y \
	--no-install-recommends --no-install-suggests mariadb-server \
	&& rm -rf /var/lib/apt/lists/* 

# Copy conf
COPY	conf/50-server.cnf /etc/mysql/mariadb.conf.d/
COPY	tools/init_db.sh /bin/

# Set permissions
RUN	chmod +x /bin/init_db.sh

EXPOSE	3306

CMD ["init_db.sh", "mysqld_safe"]
