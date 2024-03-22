=== Quick start with Kafka Connector via Docker
1. Start Docker
2. Navigate to the folder with Docker files (cd v1.latest)
3. Build the image:
docker-compose -p quickstart up -d --build
4. Login to the shell
docker exec -it shell /bin/bash
5. Create/update/delete connectors via rest api.
=== Commands for the Broker (we need to create a topic that we will use in the connector config)
List topics:
  kafka-topics --bootstrap-server broker:29092 --list
Create a topic with name nov_sink:
  kafka-topics --bootstrap-server broker:29092 --topic novsink --create --partitions 1 --replication-factor 1
Read messages from the beggining
  kafka-console-consumer --bootstrap-server broker:29092 --topic novsink --from-beginning
Publish a message to the topic
  kafka-console-producer --bootstrap-server broker:29092 --topic novsink
  {"_id": "01", "name":"Anna"}
  {"_id": "01", "name":"Binoy"}
Delete topic
kafka-topics.sh --bootstrap-server broker:29092 --delete --topic motion

=== API commands to configure the connectors:(terminal commands: docker exec -it shell /bin/bash)
list connectors:
  curl -X GET http://connect:8083/connectors
  curl -X GET http://connect:8083/connectors?expand=info
remove connector:
  curl -X DELETE http://connect:8083/connectors/mongo-sink
create connector:
  curl -X POST \
     -H "Content-Type: application/json" \
     --data '
     {"name": "mongo-sink",
      "config": {
         "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
         "connection.uri":"mongodb+srv://mongoadmin:passwordone@test.hvabt.mongodb.net/?retryWrites=true&w=majority",
         "database":"kafka",
         "collection":"sinkcoll",
         "topics":"novsink",
         "key.converter": "org.apache.kafka.connect.storage.StringConverter",
         "value.converter": "org.apache.kafka.connect.json.JsonConverter",
         "value.converter.schemas.enable": false,
         "errors.tolerance": "all",
         "errors.log.enable": true,
         "errors.log.include.message": true,
         "writemodel.strategy": "custom.sink.writemodel.ReplaceOneAddField"
         }
     }
     ' \
     http://connect:8083/connectors -w "\n"
check connector status:
  curl -X GET http://connect:8083/connectors/mongo-sink/status
  
  
Change the connector to use a custom write model strategy:
1. Create the java class. Build and extract the jar file.
2. Put the jar file in the docker image folder (v1.latest)
3. Modify the Dockerfile-MongoConnect (copy jar to the connect plugin folder) by adding the folowing lines
USER root
COPY mongo-kafkaconn-custom-v1.jar /usr/share/confluent-hub-components/mongodb-kafka-connect-mongodb/
4. Specify the write model strategy class in the connector settings:   
curl -X POST \
     -H "Content-Type: application/json" \
     --data '
     {"name": "mongo-sink",
      "config": {
         "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
         "connection.uri":"mongodb+srv://mongoadmin:M0n60_PWD@demo.2e57g.mongodb.net/?retryWrites=true&w=majority",
         "database":"kafka",
         "collection":"sinkcoll",
         "topics":"novsink",
         "key.converter": "org.apache.kafka.connect.storage.StringConverter",
         "value.converter": "org.apache.kafka.connect.json.JsonConverter",
         "value.converter.schemas.enable": false,
         "errors.tolerance": "all",
         "errors.log.enable": true,
         "errors.log.include.message": true,
         "writemodel.strategy": "custom.sink.writemodel.ReplaceUpsertWithoutNulls"
         }
     }
     ' \
     http://connect:8083/connectors -w "\n"