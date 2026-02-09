# syntax=docker/dockerfile:1
# VProfile - Web Image (Nginx reverse proxy)

FROM nginx:latest

LABEL Project="vprofile" \
      Component="web" \
      Maintainer="your-name"

# Remove default config and add our own
RUN rm -f /etc/nginx/conf.d/default.conf
COPY nginxvproapp.conf /etc/nginx/conf.d/nginxvproapp.conf
