FROM wordpress:php8.3-apache

RUN apt-get update && \
  apt-get install -y --no-install-recommends libxml2-dev && \
  docker-php-ext-install soap && \
  rm -rf /var/lib/apt/lists/*
