Fronend Dockerfile
------------------

FROM nginx:alpine
 
# Copy Angular build output into nginx container
COPY  dist/hrms/browser/ /usr/share/nginx/html
 
# Set permissions
RUN chmod -R 755 /usr/share/nginx/html
 
# Copy Nginx config  into nginx container
COPY default.conf /etc/nginx/conf.d/default.conf
 
# EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
 

Backend DOCKERFILE
-----------------------------
FROM eclipse-temurin:17-jdk-alpine
# Set the
WORKDIR /app
# Copy the jar file into the container
COPY target/hrms-0.0.1-SNAPSHOT.jar hrms-0.0.1-SNAPSHOT.jar
# Make the port available to the outside world
EXPOSE 8080

# Run the jar file
CMD ["java", "-jar", "hrms-0.0.1-SNAPSHOT.jar"]

Defualt.cong
------------------
server {
    listen 80;
    server_name _;

   # Serve Angular application
    root /usr/share/nginx/html;
    index index.html;

   location / {
        try_files $uri $uri/ /index.html;
    }

   # Proxy API requests to Spring Boot service
    location /api/ {
        proxy_pass http://<mention -service-name from yml-file (Backend-service)>:<mention port of same service>;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

   # Error handling for Angular routes
    error_page 404 /index.html;
    location = /index.html {
        allow all;
    } 
}

docker-compose.yml
-----------------

version: "3.8"
 
services:
  mysql-db:
    image: mysql:8.0
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: pipdb
      MYSQL_USER: pipuser
      MYSQL_PASSWORD: pip123
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
 
  <Backend-container-name>:
    image: <img-name>:v1
    restart: always
    depends_on:
      mysql-db:
        condition: service_healthy
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql-db:3306/pipdb
      SPRING_DATASOURCE_USERNAME: pipuser
      SPRING_DATASOURCE_PASSWORD: pip123
    ports:
      - "8080:8080"
 
  <Frontend-container-name>:
    image: <img-name>:v1
    restart: always
    ports:
      - "80:80"
    depends_on:
      - <Backen-container-name>
 
volumes:
  mysql-data:
