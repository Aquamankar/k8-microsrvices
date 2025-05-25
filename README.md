# Spring boot and Microservice with Kubernates 
Spring demo with Microservice and Kubernetes


## Running kubernates dashboard
- kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml

- kubectl proxy

- Get token : kubectl -n kubernetes-dashboard create token admin-user

## Understanding building individual Microservice

Each microservice contains pom.xml. Before building make sure the docker hub credentials are setting settings.xml file.

To run only build : mvn clean install

To build and push to docker : mvn clean install jlib:build

## Applying kubernates configuration to make pod running.

Go to k8s directory which contains 
- Apply config maps first : kubectl apply -f generic-config-map.yaml
- Setting up mysql pod : kubectl apply -f mysql.yaml
This yaml contains mysql password as well , you can use secrets/config for passwords too.
- Set up zipkin : kubectl apply -f zipkin.yaml
Access zipkin dashboard with http://localhost:9411 which was exposed using loadbalancer
- Set up service registry pod : kubectl apply -f service-registry-deployment.yaml
- Set up config server pod : kubectl apply -f config-service-deployment.yaml
- Set up cloud gateway pod : kubectl apply -f cloud-gateway-deployment.yaml
- Set up other microservices pod : kubectl apply -f payment-service-deployment.yaml/order-service-deployment.yaml/product-service-deployment.yaml

## Related to OKTA Configuration
First create new application in okta . Application type should be open id application.
Authroization server has configuration related to access token and refresh token validity. 

Go to security -> API -> default audience to modify access policies and timing.

To get refresh token , we need to ensure we have grant type refresh token for application and also scope is offline_access provided when retriving tokens. Refer application yaml of cloud gateway.



# Kubernetes Microservices Architecture Explanation

This README explains the Kubernetes YAML configuration for a microservices-based architecture consisting of the following components:

## Components Overview

### 1. **MySQL Database**

* **StatefulSet**: Runs a MySQL pod with persistent storage.
* **PersistentVolume (PV)** and **PersistentVolumeClaim (PVC)**: Provide 1Gi of storage.
* **ConfigMap (mysql-initdb-cm)**: Initializes databases (`orderdb`, `paymentdb`, `productdb`).
* **Internal Service (`mysql`)**: Used for internal cluster communication with the MySQL database.

### 2. **Eureka Service Registry**

* **StatefulSet**: Hosts Eureka service.
* **Headless Service (`eureka`)**: Allows direct pod communication.
* **NodePort Service (`eureka-lb`)**: Exposes Eureka on a node port for external access.
* **ConfigMap (`eureka-cm`)**: Contains the Eureka URL used by other services.

### 3. **Config Server**

* **Deployment (`config-server-app`)**: Hosts Spring Cloud Config Server.
* **Service (`config-server-svc`)**: Exposes config server on port 80 (maps to 9296).
* **Environment Variables**:

  * `EUREKA_SERVER_ADDRESS` from `eureka-cm`.

### 4. **Cloud Gateway**

* **Deployment (`cloud-gateway-app`)**: Hosts API Gateway.
* **Service (`cloud-gateway-svc`)**: LoadBalancer service exposing gateway on port 80 (maps to 9090).
* **Environment Variables**:

  * `CONFIG_SERVER_URL` from `config-cm`.
  * `EUREKA_SERVER_ADDRESS` from `eureka-cm`.

### 5. **Order Service**

* **Deployment (`order-service-app`)**: Hosts order microservice.
* **Service (`order-service-svc`)**: Exposes service on port 80 (maps to 8082).
* **Environment Variables**:

  * `CONFIG_SERVER_URL` from `config-cm`.
  * `DB_HOST` from `mysql-cm`.
  * `EUREKA_SERVER_ADDRESS` from `eureka-cm`.

### 6. **Product Service**

* **Deployment (`product-service-app`)**: Hosts product microservice.
* **Service (`product-service-svc`)**: Exposes service on port 80 (maps to 8082).
* **Environment Variables**:

  * `CONFIG_SERVER_URL` from `config-cm`.
  * `DB_HOST` from `mysql-cm`.
  * `EUREKA_SERVER_ADDRESS` from `eureka-cm`.

### 7. **ConfigMaps**

* `config-cm`: Holds the Config Server URL.
* `eureka-cm`: Holds the Eureka Server address.
* `mysql-cm`: Holds the hostname of the MySQL pod.

---

## Summary of Communication Flow

* **Cloud Gateway** → `config-server` & `eureka`
* **Order/Product Services** → `mysql`, `config-server`, `eureka`
* **Eureka** → Registered by all services
* **MySQL** ← Initialized by `mysql-initdb-cm` and used by order/product

---

## Access Points

* **Cloud Gateway**: Public entry via LoadBalancer.
* **Eureka Dashboard**: Exposed via NodePort.
* **Config Server**: Internal access via `config-server-svc`.

Use these service names within the cluster for service discovery and communication.

