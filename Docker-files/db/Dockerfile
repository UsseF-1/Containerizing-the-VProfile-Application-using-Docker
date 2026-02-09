# syntax=docker/dockerfile:1
# VProfile - DB Image (MySQL + schema/data init)

FROM mysql:8.0.33

LABEL Project="vprofile" \
      Component="db" \
      Maintainer="your-name"

# Required environment for MySQL initialization
ENV MYSQL_ROOT_PASSWORD=vprodbpass
ENV MYSQL_DATABASE=accounts

# MySQL entrypoint will execute *.sql/*.sh from this directory on first start
ADD db_backup.sql /docker-entrypoint-initdb.d/db_backup.sql
