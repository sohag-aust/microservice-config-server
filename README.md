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

