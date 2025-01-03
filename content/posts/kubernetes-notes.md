---
author: alex
category:
  - uncategorized
date: "2024-05-19T16:07:41+00:00"
draft: "true"
guid: https://netwithalex.blog/?p=712
title: Kubernetes Notes
url: /

---
### Introduction

[_This post is part of DEVNET DevOps (300-910) – Blueprint_](/study-devops-blueprint/)

Kubernetes, often abbreviated as K8s, is an open-source platform designed to automate deploying, scaling, and operating application containers. Below, the most crucial components and concepts..

### Core Concepts

#### Clusters

A cluster is the foundation of Kubernetes. It consists of at least one master node and multiple worker nodes.

- **Master Node**: Manages the cluster, responsible for orchestrating workloads and maintaining the desired state.
- **Worker Nodes**: Execute the tasks assigned by the master. Each node contains the services necessary to run pods.

#### Nodes

A node is a single machine (physical or virtual) within the Kubernetes cluster.

- **Components**:
  - **Kubelet**: An agent that ensures containers are running in a pod.
  - **Kube-proxy**: Manages network rules on the node.
  - **Container Runtime**: Software that runs containers (e.g., Docker).

#### Pods

- **Definition**: The smallest, most basic deployable objects in Kubernetes.
- **Characteristics**:
  - A pod represents a single instance of a running process in your cluster.
  - Can contain one or multiple containers that share the same network namespace.
  - Pods are ephemeral; they are created, destroyed, and re-created based on the desired state specified by the user.

### API Overview

- **Kubernetes API**: The central interface for interacting with the cluster. It allows users to configure and manage Kubernetes resources.
- **API Server**: Acts as the front end for the Kubernetes control plane. It handles RESTful operations and validates and configures the data for the API objects.

### Storage

- **Persistent Volumes (PV)**: Storage resources provisioned by an administrator.
- **Persistent Volume Claims (PVC)**: A user's request for storage, similar to a pod.
- **Storage Classes**: Allow dynamic provision of storage based on demand and policies set by the cluster administrator.

### Networking

- **Cluster Networking**: Ensures communication between various components of the cluster.
- **Service**: An abstraction that defines a logical set of pods and a policy by which to access them.
- **Types of Services**:
  - **ClusterIP**: Exposes the service on a cluster-internal IP.
  - **NodePort**: Exposes the service on each node's IP at a static port.
  - **LoadBalancer**: Exposes the service externally using a cloud provider's load balancer.
- **Ingress**: Manages external access to services, typically HTTP.

### Security

- **Role-Based Access Control (RBAC)**: Controls access to the Kubernetes API.
  - **Roles and ClusterRoles**: Define permissions.
  - **RoleBindings and ClusterRoleBindings**: Attach roles to users or groups.
- **Secrets**: Used to store sensitive information, such as passwords and keys.
- **Network Policies**: Define how pods are allowed to communicate with each other and other network endpoints.

### Summary
