
---

## Complete Guide to Deploy Microservices with Docker and Kubernetes on Minikube

### Step 1: Clone the Repositories

Clone the necessary microservice repositories:

```bash
git clone https://github.com/dineschandgr/product-microservice.git
git clone https://github.com/dineschandgr/payment-microservice.git
git clone https://github.com/dineschandgr/API-Gateway-Service.git
git clone https://github.com/dineschandgr/Eureka_Service_Discovery.git
git clone https://github.com/dineschandgr/user-service.git
git clone https://github.com/dineschandgr/order-microservice.git
```

### Step 2: Create Dockerfiles for All Microservices

Create a `Dockerfile` in the root of each microservice directory with the following content:

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
- ** Eruka service** : `8761`

### Step 3: Build Docker Images

Navigate to each microservice directory and build Docker images:

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
cd ../user-service
mvn clean install -DskipTests=true
docker build -t user-service-image .

# Build API Gateway
cd ../api-gateway-service
mvn clean install -DskipTests=true
docker build -t gateway-service-image .

# Build Eruka server

cd ../Eureka_Service_Discovery
mvn clean install -DskipTests=true
docker build -t eureka-service-image .
```

### Step 4: Tag and Push Docker Images to Docker Hub

Tag and push your Docker images:

```bash
# Tagging and Pushing Order Service Image
docker tag order-service-image yourusername/order-service-image:latest
docker push yourusername/order-service-image:latest

# Tagging and Pushing Product Service Image
docker tag product-service-image yourusername/product-service-image:latest
docker push yourusername/product-service-image:latest

# Tagging and Pushing Payment Service Image
docker tag payment-service-image yourusername/payment-service-image:latest
docker push yourusername/payment-service-image:latest

# Tagging and Pushing User Service Image
docker tag user-service-image yourusername/user-service-image:latest
docker push yourusername/user-service-image:latest

# Tagging and Pushing API Gateway Image
docker tag gateway-service-image yourusername/gateway-service-image:latest
docker push yourusername/gateway-service-image:latest
```

### Step 5: Create Kubernetes Deployment Files

Create a directory (e.g., `k8s`) and create the following YAML files:

#### `microservices-deployment.yml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: microservices

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysqldata
  namespace: microservices
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eureka-server
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: eureka-server
  template:
    metadata:
      labels:
        app: eureka-server
    spec:
      containers:
        - name: eureka-server
          image: vforce1520/eureka-service-image:latest
          ports:
            - containerPort: 8761
          env:
            - name: eureka_instance_hostname
              value: eureka-server
            - name: eureka_client_serviceUrl_defaultZone
              value: http://eureka-server:8761/eureka

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
        - name: order-service
          image: vforce1520/order-service-image:latest
          ports:
            - containerPort: 8763
          env:
            - name: spring_datasource_url
              value: jdbc:mysql://mysqldb:3306/order_schema?allowPublicKeyRetrieval=true
            - name: eureka_client_serviceUrl_defaultZone
              value: http://eureka-server:8761/eureka
          volumeMounts:
            - name: m2-volume
              mountPath: /root/.m2
      volumes:
        - name: m2-volume
          emptyDir: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product-service
  template:
    metadata:
      labels:
        app: product-service
    spec:
      containers:
        - name: product-service
          image: vforce1520/product-service-image:latest
          ports:
            - containerPort: 8762
          env:
            - name: spring_datasource_url
              value: jdbc:mysql://mysqldb:3306/product_schema?allowPublicKeyRetrieval=true
            - name: eureka_client_serviceUrl_defaultZone
              value: http://eureka-server:8761/eureka
          volumeMounts:
            - name: m2-volume
              mountPath: /root/.m2
      volumes:
        - name: m2-volume
          emptyDir: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: payment-service
  template:
    metadata:
      labels:
        app: payment-service
    spec:
      containers:
        - name: payment-service
          image: vforce1520/payment-service-image:latest
          ports:
            - containerPort: 8764
          env:
            - name: spring_datasource_url
              value: jdbc:mysql://mysqldb:3306/payment_schema?allowPublicKeyRetrieval=true
            - name: eureka_client_serviceUrl_defaultZone
              value: http://eureka-server:8761/eureka
          volumeMounts:
            - name: m2-volume
              mountPath: /root/.m2
      volumes:
        - name: m2-volume
          emptyDir: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: vforce1520/user-service-image:latest
          ports:
            - containerPort: 8765
          env:
            - name: spring_datasource_url
              value: jdbc:mysql://mysqldb:3306/user_schema?allowPublicKeyRetrieval=true
            - name: eureka_client_serviceUrl_defaultZone
              value: http://eureka-server:8761/eureka
          volumeMounts:
            - name: m2-volume
              mountPath: /root/.m2
      volumes:
        - name: m2-volume
          emptyDir: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway-service
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gateway-service
  template:
    metadata:
      labels:
        app: gateway-service
    spec:
      containers:
        - name: gateway-service
          image: vforce1520/gateway-service-image:latest
          ports:
            - containerPort: 8769
          env:
            - name: eureka_client_serviceUrl_defaultZone
              value: http://eureka-server:8761/eureka

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysqldb
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysqldb
  template:
    metadata:
      labels:
        app: mysqldb
    spec:
      containers:
        - name: mysqldb
          image: mysql:8.2
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_DATABASE
              value: order_schema
            - name: MYSQL_ROOT_USER
              value: root
            - name: MYSQL_ROOT_PASSWORD
              value: password
          volumeMounts:
            - name: mysqldata
              mountPath: /var/lib/mysql
      volumes:
        - name: mysqldata
          persistentVolumeClaim:
            claimName: mysqldata

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rabbitmq
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rabbitmq
  template:
    metadata:
      labels:
        app: rabbitmq
    spec:
      containers:
        - name: rabbitmq
          image: rabbitmq:3-management-alpine
          ports:
            - containerPort: 5672
            - containerPort: 15672
          volumeMounts:
            - name: rabbitmq-data
              mountPath: /var/lib/rabbitmq
            - name: rabbitmq-log
              mountPath: /var/log/rabbitmq
      volumes:
        - name: rabbitmq-data
          emptyDir: {}
        - name: rabbitmq-log
          emptyDir: {}

---
apiVersion: v1
kind: Service
metadata:
  name: eureka-server
  namespace: microservices
spec:
  type: NodePort
  ports:
    - port: 8761
      targetPort: 8761
  selector:
    app: eureka-server

---
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: microservices
spec:
  type: NodePort
  ports:
    - port: 8763
      targetPort: 8763
  selector:
    app: order-service

---
apiVersion: v1
kind: Service
metadata:
  name: product-service
  namespace: microservices
spec:
  type: NodePort
  ports:
    - port: 8762
      targetPort: 8762
  selector:
    app: product-service

---
apiVersion: v1
kind: Service
metadata:
  name: payment-service
  namespace: microservices
spec:
  type: NodePort
  ports:
    - port: 8764
      targetPort: 8764
  selector:
    app: payment-service

---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: microservices
spec:
  type: NodePort
  ports:
    - port: 8765
      targetPort: 8765
  selector:
    app: user-service

---
apiVersion: v1
kind: Service
metadata:
  name: gateway-service
  namespace: microservices
spec:
  type: NodePort
  ports:
    - port: 8769
      targetPort: 8769
  selector:
    app: gateway-service

---
apiVersion: v1
kind: Service
metadata:
  name: mysqldb
  namespace: microservices
spec:
  type: NodePort
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: mysqldb

---
apiVersion: v1
kind: Service
metadata:
  name: rabbitmq
  namespace: microservices
spec:
  type: NodePort
  ports:
    - name: rabbitmq
      port: 5672
      targetPort: 5672
    - name: rabbitmq-management
      port: 15672
      targetPort: 15672
  selector:
    app: rabbitmq
```

### Step 6: Start Minikube

If you haven't started Minikube yet, do so now:

```bash
minikube start
```

### Step 7: Apply the Kubernetes Configurations

Navigate to the directory where your YAML files are located and run the following commands:

```bash
# Apply all services
kubectl apply -f microservices-deployment.yml

```

### Step 8: Verify Deployments

Check the status of your deployments and services:

```bash
kubectl get deployments -n microservices
kubectl get pods -n microservices
kubectl get services -n microservices
```

### Step 9: Access the Services

Get the Minikube IP:

```bash
minikube ip
```
to access the the services

```bash

minikube service gateway-service --url -n microservices

```
### Step 10: Clean Up

To delete all deployed services

 and deployments:

```bash
kubectl delete -f microservices-deployment.yml

```

### Summary
1. **Clone the repositories**.
2. **Create Dockerfiles** for each microservice.
3. **Build Docker images** and push them to Docker Hub.
4. **Create Kubernetes YAML files** for all services.
5. **Start Minikube** and apply the configurations.
6. **Verify deployments** and access services.
7. **Clean up** when done.

Make sure to replace `yourusername` with your actual Docker Hub username. Enjoy your microservices setup!
