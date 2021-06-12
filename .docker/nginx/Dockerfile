FROM nginx:1.18-alpine

ENV USERNAME www-app

# --------------------------------------------------------------------------------------------------

# CREATE APPLICATION USER

RUN addgroup -g 1000 ${USERNAME} \
    && adduser -G ${USERNAME} -u 1000 -h /var/www -s /bin/bash -S ${USERNAME}

# --------------------------------------------------------------------------------------------------

RUN touch /var/run/nginx.pid

# --------------------------------------------------------------------------------------------------

RUN apk add --no-cache \
    curl \
    nss-tools \
    openssl

# --------------------------------------------------------------------------------------------------

# SETUP SSL CERTIFICATION

RUN mkdir /etc/nginx/certs && echo -e "\n\n\n\n\n\n\n" | openssl req \
    -x509 -nodes \
    -days 365 \
    -newkey rsa:2048 \
    -keyout /etc/nginx/certs/nginx.key \
    -out /etc/nginx/certs/nginx.crt

# Install mkcert. See: https://github.com/FiloSottile/mkcert
RUN (cd /usr/local/bin/ \
    && curl -L https://github.com/FiloSottile/mkcert/releases/download/v1.4.1/mkcert-v1.4.1-linux-amd64 -o mkcert \
    && chmod +x mkcert)

# --------------------------------------------------------------------------------------------------

# ADD CONF FILES

COPY ./.docker/nginx/conf/nginx.conf /etc/nginx/
COPY ./.docker/nginx/conf/default.conf /etc/nginx/conf.d/default.conf

# --------------------------------------------------------------------------------------------------

# MIRROR VOLUMES ON NGINX AND PHP CONTAINERS

RUN mkdir -p /etc/nginx/html /var/www/html /sock \
    && chown -R ${USERNAME}:${USERNAME} /etc/nginx /var/www /var/cache/nginx /var/run/nginx.pid /sock

# --------------------------------------------------------------------------------------------------

# FINAL TOUCHES

EXPOSE 8000 8443
USER ${USERNAME}:${USERNAME}
VOLUME /var/www
WORKDIR /var/www/html
