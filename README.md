# abc-icommerce-guideline

# Introduction:
   - Due to time limitation, I implement a prototype system to demo micro-service pattern to adapt the basic requirement from NAB instead of MVP or POC
   - As there is no discussion during development so that I did a lot of assumptions as below:
        - Have filtered API
        - Whenever inventory product update, original data will be move to history table and update
        - Have audit service receive audit sync from kafka message
        - Customer can log in by social authenticate not implemented by mentioned in design doc
        - Please follow below scenario to play around with system API
            - Admin create product by inventory service with these api start with "/inventory/product" (POST/PUT/DELETE/GET)
            - Then user can create a basket and create some basket details by shopping cart service with API like "/cart/basket" or "/cart/basket-detail", while add basket detail with particular product id and quantity, shopping cart service will call inventory service for checking availability production quantity
            - Final user and submit basket id to order service for the checkout, while doing checking order service will call shopping cart service to get basket info data then call inventory service for order, Finally call shopping cart service to close the basket
            NOTE every user will have only one opening basket with status = true, then will be false if user place the order before user can do a new basket. Detail API will be mentioned at the section "API play around".
                  
   - The source code is implemented base on below tech stack please reference at Tech Stack
   > Spring cloud framework [AOP, cloud stream kafka, request filter etc.]
>
   > Netflix OSS [Eureka, zuul, hystrix, ribbon]
>
   > Feign client
>
   > Kafka, message broker
>
   > H2 for data storage
>
   > Lombok
>
   > Junit 5
>
   > Jacoco
>
   > Implement Global exception handler
>
   > Implement Global request header filter
>
   > Validation check
>
   > Many design pattern like template, DI, 
> 
   - Should have sometime in future (mentioned in design doc):
> Should implement Atomic transaction by apply event driven or cloud stream kafka Stream (instead of cloud stream kafka only), compensation transaction in exception case
>
> Should add swagger for API description
>
> Use flyway for database schema versioning
>   
> EHcache
>
> Redis
>
> Elasticsearch
>
> Blob object (MinIO, S3, Az Blob etc.)
>
> ELK stack
>
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

- working machine need have JDK 8 and Maven 3.6.0
Assumption: those software has already installed
- checkout source code from the links below:
https://github.com/tuantruong2206/abc-icommerce-common-lib.git
https://github.com/tuantruong2206/abc-icommerce-config-repo.git
https://github.com/tuantruong2206/abc-icommerce-config-server.git
https://github.com/tuantruong2206/abc-icommerce-discovery-service.git
https://github.com/tuantruong2206/abc-icommerce-api-gw.git
https://github.com/tuantruong2206/abc-icommerce-audit-service.git
https://github.com/tuantruong2206/abc-icommerce-inventory-service.git
https://github.com/tuantruong2206/abc-icommerce-shopping-cart-service.git
https://github.com/tuantruong2206/abc-icommerce-order-service.git

- Need to go to each cloned dir and then run command: mvn clean install

- Start services as following order of the checkout repo list:
```sh
> mvn spring-boot:run -Drun.profiles=local -DskipTests
```

- service port mapping:
>| Service name  | Port | Service id                          | H2 data dir                  |
>
>|---------------|------|-------------------------------------|------------------------------|
>
>| API gateway   | 8082 | nab-icommerce-api-gw                | NA                           |
>
>| Order         | 8030 | nab-icommerce-order-service         | ./data/order-service         |
>
>| Shopping cart | 8040 | nab-icommerce-shopping-cart-service | ./data/shopping-cart-service |
>
>| Audit         | 8050 | nab-icommerce-audit-service         | ./data/audit-service         |
>
>| Inventory     | 8060 | nab-icommerce-inventory-service     | ./data/inventory-service     |
>
>| config        | 8888 |                                     |                              |
>
>| Discovery     | 8761 |                                     |                              |
- Data access:
    - After successfully starting services, we can access to its own db by this convention
    ```sh
    http://localhost:<service port>/h2-console
    user:sa
    password: password
    datadir:jdbc:h2:./data/shopping-cart-service
    ```
    - you can verify and see above services
- API play around:
    - Please strictly follow above scenario to play around with the system. Here below is API detail:
    -  Inventory service: I did data validation on this service, global exception handling as well as auditing
        - CRUD and data validation check as below:
            - Create a product with error return product name is null
            ```sh
             curl --location --request POST 'http://localhost:8082/inventory/product' \
             --header 'Content-Type: application/json' \
             --header 'HEADER_USER: Brian.truong' \
             --data-raw '{
                 "price": 150,
                 "color": "black",
                 "description": "Apple iphone 2020",
                 "quantity": 10
             }'
            ```
            - Create a product successfully
            ```sh
          curl --location --request POST 'http://localhost:8082/inventory/product' \
          --header 'Content-Type: application/json' \
          --header 'HEADER_USER: Brian.truong' \
          --data-raw '{
              "name": "iphone xs max 100",
              "price": 150,
              "color": "black",
              "description": "Apple iphone 2020 100",
              "quantity": 100
          }'
            ```
            - Edit the product with an error return PLEASE change id follow your data
            ```sh
            curl --location --request PUT 'http://localhost:8082/inventory/product' \
                        --header 'Content-Type: application/json' \
                        --header 'HEADER_USER: Brian.truong' \
                        --data-raw '{
                            "id": 2,
                                "name": "iphone xs max 29",
                                "description": "Apple iphone 2020 29",
                                "price": 1,
                                "color": "silver"
                        }'
            ```
            
            - Edit the product PLEASE change id follow your data
            ```sh
          curl --location --request PUT 'http://localhost:8082/inventory/product' \
          --header 'Content-Type: application/json' \
          --header 'HEADER_USER: Brian.truong' \
          --data-raw '{
              "id": 2,
                  "name": "iphone xs max 29",
                  "description": "Apple iphone 2020 29",
                  "price": 10,
                  "quantity": 100,
                  "color": "silver"
          }'
            ```
            - Delete the product PLEASE change id follow your data
            ```sh
          curl --location --request DELETE 'localhost:8082/inventory/product/6' \
          --header 'HEADER_USER: Brian.truong'
            ```
    -  shopping cart service: This service is created for the demo of interaction between service to service communication by feign client, Should have compensation transaction
        - We need create a basket first by
            ```sh
          curl --location --request POST 'http://localhost:8082/cart/basket' \
          --header 'Content-Type: application/json' \
          --header 'HEADER_USER: Brian.truong' \
          --data-raw '{
              "userid": "Brian.truong1"
          }'
          ```
        - Then call basket detail to put product to basket by API Note change prodId follow your prod and basketId, while call this API, it will check inventory service for product quantity then put to basket
        PLEASE change prodId, belongToBasketId follow your data
            ```sh
          curl --location --request POST 'http://localhost:8082/cart/basket-detail' \
          --header 'Content-Type: application/json' \
          --header 'HEADER_USER: Brian.truong' \
          --data-raw '{
              "prodId": 2,
              "belongToBasketId": 20,
              "quantity": 1
          }'
          ```
    - Order service: this service is created for the demo of interaction between 3 services by feign clients, Should have compensation transaction 
        - We have basket with detail basket then call order API for checking out, order service will call shopping cart API to get basket info for order. Then call inventory service for order. finally call shopping cart service to close basket
        - Please change userid amd basketId properly       
            ```sh
          curl --location --request POST 'http://localhost:8082/order' \
          --header 'Content-Type: application/json' \
          --header 'HEADER_USER: Brian.truong' \
          --data-raw '{
              "userid": "Brian.truong1",
              "amount": 152,
              "basketId": 16
          }'
          ```
           
 

