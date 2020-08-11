# abc-icommerce-guideline

# Introduction:
   - Due to time limitation, I implement a prototype system to demo micro-service pattern to adapt basic requirement from NAB instead of MVP or POC
   - The source code is implemented base on below tech stack please reference at Tech Stack
   > Spring cloud framework [AOP, cloud stream kafka, filter etc.]  
   > Netflix OSS [Eureka, zuul, hystrix, ribbon]
   > Feign client
   > Kafka, message broker
   > H2 for data storage
   > Lombok
   > Junit 5
   > Jacoco
   > Implement Global exception handler
   - Should have sometime in future (mentioned in design doc):
   > EHcache
   > Redis
   > Elasticsearch
   > Blob object (MinIO, S3, Az Blob etc.)
   > ELK stack
   > Prometheus
    

# Work station setup:
- Install Kafka: 
> Download kafka from [here](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.3.1/kafka_2.11-2.3.1.tgz)
extract kafka
```sh
tar -xzf kafka_2.11-x.x.x.tgz
cd kafka_2.11-x.x.x
```
OSX/Linux
```sh
> bin/zookeeper-server-start.sh config/zookeeper.properties
> bin/kafka-server-start.sh config/server.properties
```


