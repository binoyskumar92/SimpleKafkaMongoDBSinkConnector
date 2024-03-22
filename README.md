
# Kafka Connector Quick Start with Docker

This guide will help you set up a simple Kafka environment locally using Docker Compose to test the MongoDB sink connector.

## Quick Start Guide

### Setting Up

1. **Start Docker:** Ensure Docker is running on your machine.

2. **Build the Docker Image:**
   ```bash
   docker-compose -p quickstart up -d --build
   ```

3. **Access the Shell:**
   ```bash
   docker exec -it shell /bin/bash
   ```

4. **Manage Connectors:** Create, update, or delete connectors via REST API.

### Kafka Broker Commands

Begin by launching the Docker console and then open a terminal for the broker. Use these commands for interacting with the Kafka broker:

- **List Topics:**
  ```bash
  kafka-topics --bootstrap-server broker:29092 --list
  ```

- **Create a Topic (`demoTopic`):**
  ```bash
  kafka-topics --bootstrap-server broker:29092 --topic demoTopic --create --partitions 1 --replication-factor 1
  ```

- **Read Messages from the Beginning:**
  ```bash
  kafka-console-consumer --bootstrap-server broker:29092 --topic demoTopic --from-beginning
  ```

- **Publish a Message to the Topic:**
  ```bash
  kafka-console-producer --bootstrap-server broker:29092 --topic demoTopic
  # Then input your JSON message, for example:
  # {"_id": "01", "name":"Anna"}
  # {"_id": "02", "name":"Binoy"}
  ```

- **Delete a Topic:**
  ```bash
  kafka-topics --bootstrap-server broker:29092 --delete --topic demoTopic
  ```

### API Commands for Connector Configuration

These commands help you configure the connectors from the terminal:

- **Create Connector:**
  ```bash
  curl -X POST \
   -H "Content-Type: application/json" \
   --data '{
     "name": "mongo-sink",
     "config": {
       "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
       "connection.uri":"mongodb+srv://mongoadmin:passwordone@test.hvabt.mongodb.net/?retryWrites=true&w=majority",
       "database":"kafka",
       "collection":"sinkcoll",
       "topics":"demoTopic",
       "key.converter": "org.apache.kafka.connect.storage.StringConverter",
       "value.converter": "org.apache.kafka.connect.json.JsonConverter",
       "value.converter.schemas.enable": false,
       "errors.tolerance": "all",
       "errors.log.enable": true,
       "errors.log.include.message": true
     }
   }' \
   http://connect:8083/connectors -w "\n"
  ```

- **List Connectors:**
  ```bash
  curl -X GET http://connect:8083/connectors
  curl -X GET http://connect:8083/connectors?expand=info
  ```

- **Remove Connector:**
  ```bash
  curl -X DELETE http://connect:8083/connectors/mongo-sink
  ```
- **Check Connector Status:**
  ```bash
  curl -X GET http://connect:8083/connectors/mongo-sink/status
  ```
