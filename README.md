# Rabbitmq
RabbitMQ in HA Mode using Kubernetes Operator

# What is RabbitMQ ?

RabbitMQ is an open-source message broker software that facilitates communication between different services or applications by handling the sending and receiving of messages. It is based on the Advanced Message Queuing Protocol (AMQP), which is a widely used protocol for handling messaging in a distributed system

# Features of RabbitMQ

1. Clustering: Group nodes together for scalability and fault tolerance.

2. High Availability: Mirror queues across nodes for redundancy.

3. Client Libraries: Supports multiple languages for easy integration.

4. Web-UI & HTTP API: Simplify management, monitoring, and automation.

5. Multiple Protocols: Flexible messaging with AMQP, MQTT, STOMP, and more.


# How RabbitMQ Works:

1. Producer sends a message to the Exchange.

2. The Exchange decides how to route the message (based on type and routing rules) and places it in a Queue.

3. Consumer retrieves the message from the Queue for processing.


# Setting up RabbitMQ Cluster in Kubernetes

This guide will walk you through the steps to deploy a RabbitMQ cluster on Kubernetes using the Bitnami RabbitMQ Cluster Operator. We will also set up persistent storage using a custom `StorageClass` and `PersistentVolume`.

## Prerequisites

- A Kubernetes cluster (either local or cloud-based)
- kubectl CLI installed and configured to interact with your Kubernetes cluster
- `helm` CLI installed and configured to manage Kubernetes charts


## Steps to Deploy RabbitMQ Cluster

### 1. Add Bitnami Helm Repository

First, add the Bitnami repository to Helm to install the RabbitMQ Cluster Operator.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### 2. Install RabbitMQ Cluster Operator

Install the RabbitMQ Cluster Operator in a specific namespace (rabbitmq-system) using Helm:

```bash
helm install rabbitmq-cluster-operator bitnami/rabbitmq-cluster-operator -n rabbitmq-system --create-namespace
```

### 3. Check Kubernetes Resources

Verify that all the resources are created by the operator.
```bash

kubectl get all -n rabbitmq-system
```
### Kubernetes Resources in the `rabbitmq-system` Namespace



| **Resource Type** | **Name**                                                              | **Ready** | **Status** | **Restarts** | **Age**  | **ClusterIP** | **External-IP** | **Ports** |
|-------------------|----------------------------------------------------------------------|-----------|------------|--------------|----------|---------------|-----------------|-----------|
| **Pod**           | rabbitmq-cluster-operator-55c77f57cd-gh8fm                           | 1/1       | Running    | 0            | 5d22h    |               |                 |           |
| **Pod**           | rabbitmq-cluster-operator-rabbitmq-messaging-topology-operq44tk      | 1/1       | Running    | 0            | 5d22h    |               |                 |           |
| **Service**       | rabbitmq-cluster-operator-rabbitmq-messaging-topology-operator       |           |            |              | 5d22h    | 10.108.69.11  | `<none>`        | 443/TCP   |
| **Deployment**    | rabbitmq-cluster-operator                                            | 1/1       | Running    | 0            | 5d22h    |               |                 |           |
| **Deployment**    | rabbitmq-cluster-operator-rabbitmq-messaging-topology-operator       | 1/1       | Running    | 0            | 5d22h    |               |                 |           |
| **ReplicaSet**    | rabbitmq-cluster-operator-55c77f57cd                                 | 1         | 1          | 1           | 5d22h    |               |                 |           |
| **ReplicaSet**    | rabbitmq-cluster-operator-rabbitmq-messaging-topology-operator-584f96cbf4 | 1       | 1          | 1         | 5d22h    |               |                 |           |


### 4. Create Custom StorageClass
To use local persistent storage for RabbitMQ, create a custom StorageClass. This example uses the Immediate volume binding mode

```bash
kubectl apply -f storageclass.yaml
```

### 5. Create RabbitMQ Cluster Resource
Now, you can create a RabbitmqCluster custom resource to define your RabbitMQ cluster. Customize the configuration as needed, including resources, replicas, and RabbitMQ-specific settings.

Create a file called rabbitmq-cluster.yaml:
```bash
kubectl apply -f rabbitmq-cluster.yaml
```

### 6. Verify RabbitMQ Cluster Deployment

Check if the RabbitMQ cluster is running successfully:

### Kubernetes Resources Overview
```bash
kubectl get all -n rabbitmq-system

| **NAME**                                               | **READY** | **STATUS** | **RESTARTS** | **AGE**     |
|--------------------------------------------------------|-----------|------------|--------------|-------------|      |
| pod/rabbitmqcluster-prod-server-0                      | 1/1       | Running    | 0            | 5d22h       |
| pod/rabbitmqcluster-prod-server-1                      | 1/1       | Running    | 0            | 5d22h       |
| pod/rabbitmqcluster-prod-server-2                      | 1/1       | Running    | 0            | 5d22h       |

| **NAME**                                               | **TYPE**     | **CLUSTER-IP**  | **EXTERNAL-IP**  | **PORT(S)**                            | **AGE**     |
|--------------------------------------------------------|-------------|-----------------|------------------|----------------------------------------|-------------|

| service/rabbitmqcluster-prod                           | NodePort    | 10.104.167.172  | <none>           | 5672:30730/TCP, 15672:32326/TCP, 15692:31775/TCP | 5d22h       |
| service/rabbitmqcluster-prod-nodes                     | ClusterIP   | None            | <none>           | 4369/TCP, 25672/TCP                    | 5d22h       |

| **NAME**                                               | **DESIRED** | **CURRENT** | **READY** | **AGE**     |
|--------------------------------------------------------|-------------|-------------|-----------|-------------|
| deployment.apps/receive-service-rabbitmq-helm          | 1           | 1           | 1         | 5d20h       |


| **NAME**                                               | **ALLREPLICASREADY** | **RECONCILESUCCESS** | **AGE**   |
|--------------------------------------------------------|---------------------|---------------------|-------------|
| statefulset.apps/rabbitmqcluster-prod-server           | 3/3                 | True                | 5d22h       |

```

### 7. Access the RabbitMQ Management Console
To access the RabbitMQ management console externally (using NodePort), get the service details:

Find the NodePort assigned to the RabbitMQ service and access it through 

http://<node-ip>:<node-port>

![image](https://github.com/user-attachments/assets/2d6f2b4b-23ff-4c7f-881e-84d0e57efcf6)

![image](https://github.com/user-attachments/assets/292bbfaa-5f95-47e8-901d-a41e86d2a509)



####  The default credentials for RabbitMQ are:

FOR RabbitMQ login: get username and password details from below secrets


kubectl get secret -n rabbitmq-system rabbitmqcluster-prod-default-user -o jsonpath='{.data.username}' | base64 --decode


kubectl get secret -n rabbitmq-system rabbitmqcluster-prod-default-user -o jsonpath='{.data.password}' | base64 --decode

#### TO ADD NEW  USER AND PASSWORD IN RABBITMQ THROUGH CTL :

 kubectl exec -it rabbitmqcluster-prod-server-0 -- rabbitmqctl list_users   --- FOR LISTING THE USERS
 
 kubectl exec -it rabbitmqcluster-prod-server-0 -- rabbitmqctl add_user admin mypass   ---- USERNAME AND PASSWORD 
 
 kubectl exec -it rabbitmqcluster-prod-server-0 -- rabbitmqctl set_user_tags admin administrator  --- PROVIDE ADMIN PRIVILEGES
 
 kubectl exec -it rabbitmqcluster-prod-server-0 -- rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"



 


### RabbitMQ Application Testing

This project provides a minimal example of how to test a RabbitMQ instance deployed on a Kubernetes cluster using a simple Python-based producer and consumer.

Prerequisite:

1.RabbitMQ is deployed and accessible in your Kubernetes cluster.

2. You have the RabbitMQ service name (mycase: rabbitmqcluster-prod.default.svc.cluster.local).

3. Python 3 image or container is available (you can use python:3.9).

4. pika installed inside the application container.
   

producer.py: Sends a simple message to a RabbitMQ queue.

consumer.py: Listens to the queue and prints received messages.

Kubernetes manifests to run the producer and consumer as pods.

ConfigMap used to inject Python scripts into the pods.


1. Create a configmap 
   
   kubectl apply -f  rabbitmq-test-configmap.yaml

   
   
3. craete a pod

   kubectl apply -f rabbitmq-test-pod.yaml


   | **Pod Name**  | **Ready** | **Status** | **Restarts** | **Age** |
| ------------- | --------- | ---------- | ------------ | ------- |
| rabbitmq-test | 1/1       | Running    | 0            | 48m     |


Run producer and consumer scripts inside the pods

producer.py:

Sends a message "Hello this is Raghav, how are you" to the hello queue.

![image](https://github.com/user-attachments/assets/e83ef9ef-192e-42b1-911e-d9f63336082c)

Now lets check it in rabbitmq queue with message


![image](https://github.com/user-attachments/assets/030a0e46-0374-4f54-8d2a-e485040b28d9)

lets run consumer script inside the pod

consumer.py

Listens on the hello queue and prints any received messages.

![image](https://github.com/user-attachments/assets/5a0b59d2-9786-4199-9f84-513b570829c4)

Check in the rabbitmq GUI whether the queue is consumed or not

![image](https://github.com/user-attachments/assets/159d66cf-6ee7-4318-95c5-d629b510df00)

we can see queue is empty now

## Successfully we have Tested our Appplication in RabbitMQ !!!!!!!!





