

#### Step 1: Create Dockerfiles for All Microservices

Create a `Dockerfile` in the root of each microservice directory (e.g., `order-microservice`, `product-microservice`, etc.) with the following content:

```dockerfile
# Dockerfile for each microservice
FROM openjdk:11-jre-slim

WORKDIR /app

COPY target/*.jar app.jar

EXPOSE <service-port>  # Replace with the specific port for each service

ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Replace `<service-port>`** with:
- **Order Service**: `8763`
- **Product Service**: `8762`
- **Payment Service**: `8764`
- **User Service**: `8765`
- **API Gateway**: `8769`

#### Step 2: Build Docker Images

Navigate to each microservice directory and run the following commands to build Docker images.

```bash
# Build Order Service
cd order-microservice
mvn clean install -DskipTests=true
docker build -t order-service-image .

# Build Product Service
cd ../product-microservice
mvn clean install -DskipTests=true
docker build -t product-service-image .

# Build Payment Service
cd ../payment-microservice
mvn clean install -DskipTests=true
docker build -t payment-service-image .

# Build User Service
cd ../user-microservice
mvn clean install -DskipTests=true
docker build -t user-service-image .

# Build API Gateway
cd ../api-gateway-service
mvn clean install -DskipTests=true
docker build -t gateway-service-image .
```

#### Step 3: Update the `docker-compose.yml`

Ensure your `docker-compose.yml` is set up correctly. Here’s the relevant snippet:

```yaml
version: "3.7"
services:
  eureka-server:
    container_name: eureka-server
    image: eureka-service-image
    hostname: eureka-server
    networks:
      - ms-network
    ports:
      - 8761:8761 
    environment:
      eureka.instance.hostname: eureka-server
      eureka.client.serviceUrl.defaultZone: http://eureka-server:8761/eureka

  order_service:
    container_name: order-service
    image: order-service-image
    build: .
    restart: always
    ports:
      - 8763:8763
    links:
      - eureka-server
    environment:
      spring.datasource.url: jdbc:mysql://mysqldb:3306/order_schema?allowPublicKeyRetrieval=true 
      eureka.client.serviceUrl.defaultZone: http://eureka-server:8761/eureka
    networks:
      - ms-network
    depends_on:
      - mysqldb
      - eureka-server
    volumes:
      - .m2:/root/.m2

  # Other services...

networks:  
  ms-network:  
    name: ms-network  
    driver: bridge
```

#### Step 4: Run Docker Compose

Run the following command to start all services:

```bash
docker-compose up --build
```

#### Step 5: Tag Docker Images

After building your images, tag them for Docker Hub with the `-image` suffix:

```bash
# Tagging Order Service Image
docker tag order-service-image yourusername/order-service-image:latest

# Tagging Product Service Image
docker tag product-service-image yourusername/product-service-image:latest

# Tagging Payment Service Image
docker tag payment-service-image yourusername/payment-service-image:latest

# Tagging User Service Image
docker tag user-service-image yourusername/user-service-image:latest

# Tagging API Gateway Image
docker tag gateway-service-image yourusername/gateway-service-image:latest
```

#### Step 6: Push Images to Docker Hub

Push each tagged image to Docker Hub:

```bash
# Pushing Order Service Image
docker push yourusername/order-service-image:latest

# Pushing Product Service Image
docker push yourusername/product-service-image:latest

# Pushing Payment Service Image
docker push yourusername/payment-service-image:latest

# Pushing User Service Image
docker push yourusername/user-service-image:latest

# Pushing API Gateway Image
docker push yourusername/gateway-service-image:latest
```

#### Step 7: Verify the Push

Check your Docker Hub profile to ensure the images are available:

```
https://hub.docker.com/u/yourusername
```

#### Step 8: Stopping and Cleaning Up

To stop and remove all containers, run:

```bash
docker-compose down
```

### Summary
1. **Create a `Dockerfile`** for each microservice.
2. **Build Docker images** for each microservice.
3. **Run services** with `docker-compose`.
4. **Tag images** for Docker Hub with the `-image` suffix.
5. **Push images** to Docker Hub.
6. **Verify** on Docker Hub.
7. **Clean up** when done.

