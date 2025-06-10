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



