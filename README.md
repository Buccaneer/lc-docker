--MONGODB
docker build --tag buccaneer/lc-mongodb .
docker run -p 28001:27017 --name lc-mongodb-container buccaneer/lc-mongodb

--LOCAL
gunzip -c belgianrailconnectionsOct2015.mongojsonstream.gz | mongoimport --db lc --collection connections --host 192.168.99.100 --port 28001

--QUERY SERVER
docker build --tag buccaneer/lc-query-server .
docker run -p 32777:8080 --name lc-query-server-container --link lc-mongodb-container buccaneer/lc-query-server

--NGINX CACHE
!!CHANGE IP IN conf/nginx.conf

docker build --tag buccaneer/lc-cache .
docker run -p 32778:8081 --name lc-cache-container --link lc-query-server-container buccaneer/lc-cache

--RUN
node ./bin/querymixgenerator.js > result.jsonstream
node ./bin/querymixexecutor.js < result.jsonstream
