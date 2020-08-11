# abc-icommerce-guideline

# Introduction:
   - Due to time limitation, I implement a prototype system to demo micro-service pattern to adapt basic requirement from NAB instead of MVP or POC
   - The source code is implemented base on below tech stack please reference at Tech Stack
   > Spring cloud framework [AOP, cloud stream kafka, filter etc.]
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
   > Many design pattern
> 
   - Should have sometime in future (mentioned in design doc):
> Should implement Atomic transaction by apply event driven or cloud stream kafka Stream (instead of cloud stream kafka only)
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
    - API gateway: 8082
    - Order service(./data/order-service, nab-icommerce-order-service): 8030 
    - Shopping cart service (./data/shopping-cart-service, nab-icommerce-shopping-cart-service): 8040
    - Audit service (./data/audit-service, nab-icommerce-audit-service): 8050
    - Inventory service (./data/inventory-service, nab-inventory-service): 8060
    - Config service: 8888
    - Discovery service: 8761
    NOTE: (data-dir in h2, eureka-service-name)
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
    - Because there is a limitation of time, the prototype is not completely good enough for exception handling. This project is to demo purpose only. It shows such as microservice, internal service to service communication, kafka event, global exception handling etc.
    - The context to test API is as below scenario (my ASSUMPTION):
        - Admin create product by inventory/product api (POST/PUT/DELETE/GET)
        - Then user can create basket and create basket detail by API, while add basket detail with particular product, shopping cart service will call inventory service for checking avaiability production quantity
        - Final user and submit basket id to order service for checkout, while doing checking order service will call shopping cart server to get basket info then call inventory service for order, Finally call shopping cart service to close basket
    - Please strictly follow above scenario to play around with the system. Here below is API detail
    -  Inventory service: I did data validation on this service
        - CRUD and data validation check as below:
            - Create product with error return product name is null
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
            - Create product successfully
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
            - Edit product with error return PLEASE change id follow your data
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
            
            - Edit product PLEASE change id follow your data
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
            - Delete product PLEASE change id follow your data
            ```sh
          curl --location --request DELETE 'localhost:8082/inventory/product/6' \
          --header 'HEADER_USER: Brian.truong'
            ```
    -  shopping cart service:
        - We need create basket first by below api
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
    - Order service:
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
           
 

