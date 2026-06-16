FROM debian:bookworm-slim

EXPOSE 443

# install nginx & openssl
RUN	apt-get update && apt-get install -y \
	--no-install-recommends --no-install-suggests nginx \
	openssl \
	&& rm -rf /var/lib/apt/lists/*

ARG	CERT_FOLDER CERTIFICATE KEY COUNTRY STATE LOCALITY ORGANIZATION UNIT COMMON_NAME

RUN	mkdir -p ${CERT_FOLDER} && \
	openssl req -newkey rsa:4096 -x509 -sha256 -days 365 -nodes \
   	-out ${CERTIFICATE} \
   	-keyout ${KEY} \
   	-subj "/C=${COUNTRY}/O=${ORGANIZATION}/OU=${UNIT}/CN=${COMMON_NAME}"

COPY	conf/nginx.conf	/etc/nginx/
COPY	conf/server.conf	/etc/nginx/conf.d/

RUN	echo "\tserver_name ${COMMON_NAME};\n\
	\tssl_certificate ${CERTIFICATE};\n\
	\tssl_certificate_key ${KEY};\n\
	}" >> /etc/nginx/conf.d/server.conf

# set user ID and group ID to the same value

RUN	mkdir -p /var/www/ \
	&& chown -R www-data:www-data /var/www/ \
	&& usermod -u 33 www-data && groupmod -g 33 www-data

#CMD	bash
CMD	["nginx", "-g", "daemon off;"]
