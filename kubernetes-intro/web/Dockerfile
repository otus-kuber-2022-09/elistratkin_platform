FROM nginx:1.19.6

COPY ./context/app.conf /etc/nginx/conf.d/
COPY ./context/homework.html /app/

RUN usermod -u 1001 www-data \
    && chown -R www-data:www-data /app

EXPOSE 8000