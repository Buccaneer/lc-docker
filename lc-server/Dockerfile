FROM node:5

ENV TZ=Europe/Brussels
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

COPY ./server /usr/src/app
COPY config.json /usr/src/app/
RUN npm install
CMD node bin/server.js -c config.json
EXPOSE 8084
