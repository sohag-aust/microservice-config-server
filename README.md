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

### Proxy configuration in /etc/nginx/sites-enabled
    upstream config_servers {
        server localhost:8071;
        server localhost:9071;
    }

    server {
        listen 5000;
	server_name localhost;
        
        location / {
            proxy_pass http://config_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }


    then restart nginx server : nginx -s reload

    then start config server in 2 port
    => steps : 
    1. create jar
    2. run jar : java -jar <jar_name> --server.port=8071 and java -jar <jar_name> --server.port=9071