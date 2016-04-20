FROM nginx
ENV TZ=Europe/Brussels
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN mkdir -p /data/nginx/cache
RUN rm /etc/nginx/conf.d/default.conf
COPY conf /etc/nginx