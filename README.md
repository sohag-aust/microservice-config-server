### Reading Config properties from class path for microservices
    spring:
      application:
        name: "configserver"
      profiles:
        active: native
      cloud:
        config:
          server:
            native:
            search-locations: "classpath:/config"

### Read account service configuration from config server application
    http://localhost:8071/<ACCOUNT_SERVICE_NAME>/<PROPERTIES_BASED_ON_SERVER>
    http://localhost:8071/accounts/prod


### Added a encrypt secret key (a random key), which can encrypt and decrypt plain text.
    In the Config Server Application we have /encrypt and /decrypt endpoints (POST method) to encrypt and decrypt
    text messages

    so, suppose we want to encrypt our email: abc@gmail.com, then hit /encrypt post API
    then, put the value in the config git repository, 
    Question ?? how git repo understand the encrypted email is encrypted or plain text ?
    Ans :: adding {cipher} before the value
    like previously email properties was
        email: abc@gmail.com
    now it will be
        email: {cipher}<ENCRYPTED_VALUE_OF_abc@gmail.com>


### Refresh Configuration at Runtime using Spring Cloud Bus

    These comes into picture, because in actuator refresh api, we need to run the refresh API for all the microservices, which is 
    difficult. if we have 500 microservices, then we need to run the refresh API for every micro services

    So, in the Spring Cloud Bus, I need to run the busrefresh api for only 1 microservice, and it will led to changes for all the microservices
    
    1. Firstly install and run rabbitmq docker image for rabbitmq mq related task and management related tasks
    sudo docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.12-management

    2. Secondly, add the following maven dependency for cloud bus

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>

    3. Add actuator management dependency to account, loans, and cards microservices for cloud
    currently i have enabled it in these microservices using 
        management:
          endpoints:
            web:
              exposure:
                include: "*"

    4. Add rabbitmq related config to these microservices for spring cloud bus
    5. After doing the upper steps, restart all the microservices and config server application
    6. Now, change in the config repo for the microservices,
        suppose change cards microservices prod properties, 
        so verifying it by calling
        http://localhost:8071/cards/prod
        so, here without restarting this config server we can read updated configurations

    7. Now, if we want to get the updated properties from cards microservices,
        we need to hit /busrefresh api from any microservice, so that the effect can be into other microservices
        suppose, if i changed the configurations for all the 3 microservices, then if i hit /busrefresh post api
        from 1 microservice, then it will reresh the other microservices as well.