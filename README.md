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