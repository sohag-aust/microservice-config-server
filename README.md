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


### Refresh config at runtime using Spring Cloud Bus & Spring Cloud Config monitor
    
    This comes into picture because, in the upper process we need to hit at lease /busrefresh api to refresh the configuration
    to all the 3 microservices.
    
    But in the following approach, we don't need to hit any of API or don't need to refresh the configuration
    If, config changes then it will effect all the 3 microservices.

    1. Firstly, need to add config monitor dependency to pom.xml to the configserver
    2. Add management configuration to application.yml in configserver
    3. Add rabbitmq properties to application.yml in configserver
    4. create webhook for tracking configuration changes in config repository
        goto settings -> click webhooks option -> create new webhook
        so eikhane akta url dite hobe jeta amr config server e changes gula assign kore dibe
        like: https://localhost:8071/monitor
        ei monitor api ta cloud monitor dependency provide kore.
        but github kivabe amr localhost chinbe ? jodi kono server e application thakto tahole server er public IP diye dile kaj hoto
        kintu, as my application is running in localhost so how can i solve this issue ?
        using : hookdeck.com website.
        goto : https://console.hookdeck.com/
        
        then, goto install section for linux installation,
        follow the commands, 
        1. firslty download tar.gz, then extract using tar command
        2. secondly move the hookdeck to /usr/local/bin directory using command : sudo cp hookdeck /usr/local/bin
        3. then, run the hookdeck login command from the console.hookdeck.com
                like : hookdeck login --cli-key 45lh8lvgd5c3ak2uftjufoz3ao9uovqlrqky2er7t961j8kuyb
        4. Listen hookdeck port like: hookdeck listen [port] Source
        5. question will arrive : What path should the events be forwarded to (ie: /webhooks)?
            ans:: /monitor
        6. question will arrive : What's your connection label (ie: My API)?
            ans:: localhost
        7. Then, there will be a url , which we will paste in the webhook section in the github config-properties repo

### HookDeck Login again and URL

    1. Goto console.hookdeck.com
    2. clock add destination option
    3. copy login cli, like : hookdeck login --cli-key 45lh8lvgd5c3ak2uftjufoz3ao9uovqlrqky2er7t961j8kuyb
    4. goto /usr/local/bin/ of local machine directory
    5. try with the hookdeck login command
    6. if getting error like : Verifying credentials... unexpected http status code: 401 Unauthorized
    7. run : hookdeck logout command
    8. try with login command again
    9. Listen port for configserver using hookdeck command : hookdeck listen 8071 Source
    10. Then add /monitor url
    11. add localhost
    12. After that, there will be a event url like :  https://hkdk.events/s6xkqpa7vvx2ia
    13. copy it and paste it into webhook section of my config properties github repo, if there is already one, then just edit and update

### Section-7 :: MySql docker compose container

    1. Firstly create new docker images for all services with tag s7
        like: configserver:s7, accounts:s7, loans:s7, cards:s7

    2. Then, chaange docker compose with mysql container related changes