# Dockerize Linked Connections

Repository with resources to dockerize Linked Connections.

## Setup

##### MongoDB
Build the MongoDB image and run it in a container.
```bash
docker build --tag buccaneer/lc-mongodb ./lc-mongodb
```
```bash
docker run -p 28001:27017 --name lc-mongodb-container buccaneer/lc-mongodb
```
Import the connections into the database.  Make sure to replace the --host with that of your docker machine or localhost if native.
```bash
gunzip -c belgianrailconnectionsOct2015.mongojsonstream.gz | mongoimport --db lc --collection connections --host 192.168.99.100 --port 28001
```
Ensure there is an index on departureTime. Make sure to replace the host with that of your docker machine or localhost if native.
```bash
mongo 192.168.99.100:28001
```
```bash
db.connections.ensureIndex({departureTime:1});
```
##### Query Server
Build the query server image and run it in a container.
```bash
docker build --tag buccaneer/lc-query-server ./lc-query-server
```
```bash
docker run -p 32777:8082 --name lc-query-server-container --link lc-mongodb-container buccaneer/lc-query-server
```

##### NGINX Cache
__!!__ First replace the IP address in lc-cache/conf/nginx.conf on line 49 (proxy pass) with that of your docker machine or localhost.

Build the nginx cache image and run it in a container.
```bash
docker build --tag buccaneer/lc-cache ./lc-cache
```
```bash
docker run -p 32778:8081 --name lc-cache-container --link lc-query-server-container buccaneer/lc-cache
```

##### Query mix generator & executor
Change directory to lc-query-mix then install dependencies.
```bash
cd lc-query-mix
```
```bash
npm install
```
Generate a query mix then execute it.
```bash
node ./bin/querymixgenerator.js > result.jsonstream
```
```bash
node ./bin/querymixexecutor.js < result.jsonstream
```

## Monitoring

Check for more instructions
https://www.brianchristner.io/how-to-setup-docker-monitoring/

##### InfluxDB
Run InfluxDB container
```bash
docker run -d -p 8083:8083 -p 8086:8086 --expose 8090 --expose 8099 --name lc-influx tutum/influxdb
```

Create an account with username 'root' and password 'root'.  

##### cAdvisor
Run cAdvisor container
```bash
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro --publish=8080:8080 --detach=true --link lc-influx:lc-influx --name=cadvisor google/cadvisor:latest -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=192.168.99.100:8086
```
##### Grafana
Run Grafana container
```bash
docker run -d -p 3000:3000 -e INFLUXDB_HOST=192.168.99.100 -e INFLUXDB_PORT=8086 -e INFLUXDB_NAME=cadvisor -e INFLUXDB_USER=root -e INFLUXDB_PASS=root --link lc-influx:lc-influx --name grafana grafana/grafana
```

[//]: #

   [npm]: <https://www.npmjs.com/>
   [node.js]: <https://nodejs.org/en/>
   [MongoDB]: <https://www.mongodb.org/>
