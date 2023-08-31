FROM nginx:1.21.0

ENV UID=1001 
ENV GID=1001 
ENV USER=nginx
ENV GROUP=nginx

EXPOSE 8000
WORKDIR /app

## add permissions for nginx user
RUN usermod -u ${UID} ${USER} && groupmod -g ${GID} ${GROUP} && \
    chown -R ${USER}:${GROUP} /app && chmod -R 755 /app && \
    chown -R ${USER}:${GROUP} /var/cache/nginx && \
    chown -R ${USER}:${GROUP} /var/log/nginx

COPY conf.d/default.conf /etc/nginx/conf.d/default.conf
COPY /app /app

USER nginx



