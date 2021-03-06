


# kafka-stack-docker-compose



## Single Zookeeper / Single Kafka

This configuration fits most development requirements.

 - Zookeeper will be available at `$DOCKER_HOST_IP:2181`
 - Kafka will be available at `$DOCKER_HOST_IP:9092`


Run with:
```
docker-compose -f zk-single-kafka-single.yml up
docker-compose -f zk-single-kafka-single.yml down
```

## Single Zookeeper / Multiple Kafka

If you want to have two brokers and experiment with replication / fault-tolerance.

- Zookeeper will be available at `$DOCKER_HOST_IP:2181`
- Kafka will be available at `$DOCKER_HOST_IP:9092,$DOCKER_HOST_IP:9093,$DOCKER_HOST_IP:9094`


Run with:
```
docker-compose -f zk-single-kafka-multiple.yml up
docker-compose -f zk-single-kafka-multiple.yml down
```

## Multiple Zookeeper / Single Kafka

If you want to have three zookeeper and experiment with zookeeper fault-tolerance.

- Zookeeper will be available at `$DOCKER_HOST_IP:2181,$DOCKER_HOST_IP:2182,$DOCKER_HOST_IP:2183`
- Kafka will be available at `$DOCKER_HOST_IP:9092`

Run with:
```
docker-compose -f zk-multiple-kafka-single.yml up
docker-compose -f zk-multiple-kafka-single.yml down
```


## Multiple Zookeeper / Multiple Kafka

If you want to have three zookeeper and two kafka brokers to experiment with production setup.

- Zookeeper will be available at `$DOCKER_HOST_IP:2181,$DOCKER_HOST_IP:2182,$DOCKER_HOST_IP:2183`
- Kafka will be available at `$DOCKER_HOST_IP:9092,$DOCKER_HOST_IP:9093,$DOCKER_HOST_IP:9094`

Run with:
```
docker-compose -f zk-multiple-kafka-multiple.yml up
docker-compose -f zk-multiple-kafka-multiple.yml down
```


## Full stack

 - Single Zookeeper: `$DOCKER_HOST_IP:2181`
 - Single Kafka: `$DOCKER_HOST_IP:9092`
 - Kafka Schema Registry: `$DOCKER_HOST_IP:8081`
 - Kafka Schema Registry UI: `$DOCKER_HOST_IP:8001`
 - Kafka Rest Proxy: `$DOCKER_HOST_IP:8082`
 - Kafka Topics UI: `$DOCKER_HOST_IP:8000`
 - Kafka Connect: `$DOCKER_HOST_IP:8083`
 - Kafka Connect UI: `$DOCKER_HOST_IP:8003`
 - Zoonavigator Web: `$DOCKER_HOST_IP:8004`


 Run with:
 ```
 docker-compose -f full-stack.yml up
 docker-compose -f full-stack.yml down
 ```
 
 Run KSQL CLI:
 ```
docker-compose -f full-stack.yml exec ksql-cli ksql http://ksql-server:8088
  ```
  
 To Create streams using KSQL 
 
 ```
 CREATE STREAM testTopic1 \
  (orderid VARCHAR, \
   tenant VARCHAR) \
  WITH (KAFKA_TOPIC='testTopic1', \
        VALUE_FORMAT='JSON');
        
 CREATE STREAM testTopic1_2 \
      (totalAmount STRUCT< \
            amount DOUBLE >   , \
       orderid VARCHAR, \
       tenant VARCHAR) \
WITH (KAFKA_TOPIC='testTopic1', VALUE_FORMAT='JSON');       
 ```

For this we have to create a testTopic1 and have a JSON as below

 ```
 {"orderid":"o123","tenant":"ecommerce","totalAmount":{"amount":150.0}}
  ```
  
 To get orders per 5 minutes
 ```
 CREATE TABLE ORDERS_AGG AS 
  SELECT 
  		WINDOWSTART() AS WINDOW_START_TS, 
         COUNT(*),tenant AS ORDER_COUNT
    FROM testTopic1 
           WINDOW TUMBLING (SIZE 5 MINUTES)
           group by tenant;

 ```
  
  To get order total or revenue per 5 minutes
  ```
  CREATE TABLE ORDERS_AGG_3 AS 
  SELECT 
  		WINDOWSTART() AS WINDOW_START_TS, 
         SUM(totalAmount->amount)
         AS SUM
         ,tenant AS TENANT
    FROM testTopic1_2 
           WINDOW TUMBLING (SIZE 5 MINUTES)
           group by tenant;
  ```
