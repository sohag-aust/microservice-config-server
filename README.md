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