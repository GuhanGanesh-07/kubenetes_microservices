
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

#### `eureka-service.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eureka-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: eureka-service
  template:
    metadata:
      labels:
        app: eureka-service
    spec:
      containers:
      - name: eureka-service
        image: yourusername/eureka-service-image:latest
        ports:
        - containerPort: 8761
---
apiVersion: v1
kind: Service
metadata:
  name: eureka-service
spec:
  type: NodePort
  ports:
  - port: 8761
    targetPort: 8761
    nodePort: 30001
  selector:
    app: eureka-service
```

#### `order-service.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
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
        image: yourusername/order-service-image:latest
        ports:
        - containerPort: 8763
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  type: NodePort
  ports:
  - port: 8763
    targetPort: 8763
    nodePort: 30002
  selector:
    app: order-service
```

#### `product-service.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
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
        image: yourusername/product-service-image:latest
        ports:
        - containerPort: 8762
---
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  type: NodePort
  ports:
  - port: 8762
    targetPort: 8762
    nodePort: 30003
  selector:
    app: product-service
```

#### `payment-service.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
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
        image: yourusername/payment-service-image:latest
        ports:
        - containerPort: 8764
---
apiVersion: v1
kind: Service
metadata:
  name: payment-service
spec:
  type: NodePort
  ports:
  - port: 8764
    targetPort: 8764
    nodePort: 30004
  selector:
    app: payment-service
```

#### `user-service.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
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
        image: yourusername/user-service-image:latest
        ports:
        - containerPort: 8765
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  type: NodePort
  ports:
  - port: 8765
    targetPort: 8765
    nodePort: 30005
  selector:
    app: user-service
```

#### `api-gateway.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: yourusername/gateway-service-image:latest
        ports:
        - containerPort: 8769
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
spec:
  type: NodePort
  ports:
  - port: 8769
    targetPort: 8769
    nodePort: 30006
  selector:
    app: api-gateway
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
kubectl apply -f eureka-service.yaml
kubectl apply -f order-service.yaml
kubectl apply -f product-service.yaml
kubectl apply -f payment-service.yaml
kubectl apply -f user-service.yaml
kubectl apply -f api-gateway.yaml
```

### Step 8: Verify Deployments

Check the status of your deployments and services:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

### Step 9: Access the Services

Get the Minikube IP:

```bash
minikube ip
```

Access each service using the node ports specified in your YAML files:

- **Eureka Service**: `http://<minikube-ip>:30001`
- **Order Service**: `http://<minikube-ip>:30002`
- **Product Service**: `http://<minikube-ip>:30003`
- **Payment Service**: `http://<minikube-ip>:30004`
- **User Service**: `http://<minikube-ip>:30005`
- **API Gateway**: `http://<minikube-ip>:30006`

### Step 10: Clean Up

To delete all deployed services

 and deployments:

```bash
kubectl delete -f eureka-service.yaml
kubectl delete -f order-service.yaml
kubectl delete -f product-service.yaml
kubectl delete -f payment-service.yaml
kubectl delete -f user-service.yaml
kubectl delete -f api-gateway.yaml
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