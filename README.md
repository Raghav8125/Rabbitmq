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

Create a file called sc.yaml:

