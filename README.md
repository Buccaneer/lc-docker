# Dockerize Linked Connections

Repository with resources to dockerize Linked Connections.

## 1. Setup

### 1.1 MongoDB
#### Build the MongoDB image and run it in a container
```bash
docker build --tag buccaneer/lc-mongodb ./lc-mongodb
```
```bash
docker run -p 28001:27017 --name lc-mongodb-container buccaneer/lc-mongodb
```
#### Import the connections into the database

##### Linux
```bash
gunzip -c belgianrailconnectionsOct2015.mongojsonstream.gz | mongoimport --db lc --collection connections --port 28001
```
##### Windows
__!!__ Make sure the host is that of your docker machine.
```bash
gunzip -c belgianrailconnectionsOct2015.mongojsonstream.gz | mongoimport --db lc --collection connections --host 192.168.99.100 --port 28001
```
#### Ensure there is an index on departureTime

##### Linux
```bash
mongo lc --port 28001
```
##### Windows
__!!__ Make sure the host is that of your docker machine.
```bash
mongo 192.168.99.100:28001/lc
```
##### Mongo shell
```bash
db.connections.ensureIndex({departureTime:1});
quit();
```

### 1.2 Query server
#### Build the query server image and run it in a container
```bash
docker build --tag buccaneer/lc-query-server ./lc-query-server
```
```bash
docker run -p 8082:8082 --name lc-query-server-container --link lc-mongodb-container buccaneer/lc-query-server
```

### 1.3 Linked Connections server
#### Build the lc server image and run it in a container
```bash
docker build --tag buccaneer/lc-server ./lc-server
```
```bash
docker run -p 8084:8084 --name lc-server-container --link lc-mongodb-container buccaneer/lc-server
```

### 1.4 NGINX Cache
__!!__ Copy __lc-cache/conf-linux__ or __lc-cache/conf-windows__ folder as __lc-cache/conf__ to use the correct configuration.
#### Build the nginx cache image and run it in a container.
```bash
docker build --tag buccaneer/lc-cache ./lc-cache
```
```bash
docker run -p 8081:8081 --name lc-cache-container --link lc-query-server-container buccaneer/lc-cache
```

### 1.5 Query mix generator & executor
#### Change directory to lc-query-mix then install dependencies.
```bash
cd lc-query-mix
```
```bash
npm install
```
#### Generate a query mix then execute it.
```bash
node ./bin/querymixgenerator.js > result.jsonstream
```
```bash
node ./bin/querymixexecutor.js < result.jsonstream
```

## 2. Monitoring

For additional instructions check:

https://www.brianchristner.io/how-to-setup-docker-monitoring/

### 2.1 InfluxDB
Run InfluxDB container
```bash
docker run -d -p 8083:8083 -p 8086:8086 --expose 8090 --expose 8099 --name lc-influx tutum/influxdb
```

Create an account with username 'root' and password 'root'.  

### 2.2 cAdvisor
Run cAdvisor container
```bash
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro --publish=8080:8080 --detach=true --link lc-influx:lc-influx --name=cadvisor google/cadvisor:latest -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=192.168.99.100:8086
```
### 2.3 Grafana
Run Grafana container
```bash
docker run -d -p 3000:3000 -e INFLUXDB_HOST=192.168.99.100 -e INFLUXDB_PORT=8086 -e INFLUXDB_NAME=cadvisor -e INFLUXDB_USER=root -e INFLUXDB_PASS=root --link lc-influx:lc-influx --name grafana grafana/grafana
```

[//]: #

   [npm]: <https://www.npmjs.com/>
   [node.js]: <https://nodejs.org/en/>
   [MongoDB]: <https://www.mongodb.org/>
