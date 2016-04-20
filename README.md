# Dockerize Linked Connections

Repository with resources to dockerize Linked Connections.

## Setup

##### MongoDB
Build
```bash
docker build --tag buccaneer/lc-mongodb ./lc-mongodb
```
Run
```bash
docker run -p 28001:27017 --name lc-mongodb-container buccaneer/lc-mongodb
```
Import connections
```bash
gunzip -c belgianrailconnectionsOct2015.mongojsonstream.gz | mongoimport --db lc --collection connections --host 192.168.99.100 --port 28001
```
Ensure there is an index on departureTime
```mongodb
db.connections.ensureIndex({departureTime:1});
```
##### Query Server
Build
```bash
docker build --tag buccaneer/lc-query-server ./lc-query-server
```
Run
```bash
docker run -p 32777:8082 --name lc-query-server-container --link lc-mongodb-container buccaneer/lc-query-server
```

##### NGINX Cache
!!CHANGE IP IN conf/nginx.conf
Build
```bash
docker build --tag buccaneer/lc-cache ./lc-cache
```
Run
```bash
> docker run -p 32778:8081 --name lc-cache-container --link lc-query-server-container buccaneer/lc-cache
```

##### Query mix generator & executor
Generate query mix
```bash
node ./lc-query-mix/bin/querymixgenerator.js > result.jsonstream
```
Execute the mix
```bash
node ./lc-query-mix/bin/querymixexecutor.js < result.jsonstream
```

## Monitoring

Check for more instructions
https://www.brianchristner.io/how-to-setup-docker-monitoring/

##### InfluxDB
Run InfluxDB container
```bash
docker run -d -p 8083:8083 -p 8086:8086 --expose 8090 --expose 8099 --name lc-influx tutum/influxdb
```

Create account with username 'root' and password 'root'.  

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
