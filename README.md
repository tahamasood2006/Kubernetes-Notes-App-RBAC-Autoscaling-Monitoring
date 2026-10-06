# Kubernetes Notes App — RBAC, Autoscaling & Monitoring

A hands-on Kubernetes project demonstrating how to deploy a containerized **Django Notes application** with **Kubernetes RBAC, RoleBindings, Ingress, Horizontal Pod Autoscaling, NGINX, Prometheus, and Grafana**.

The project focuses on practical Kubernetes administration, access control, workload management, and observability.

---

## 🚀 Project Overview

The application consists of a Django backend deployed as a Kubernetes workload, with NGINX used as an additional service behind the Kubernetes Ingress layer.

The project was built to practice several important Kubernetes concepts:

* Kubernetes Deployments
* Services
* Namespaces
* Ingress
* RBAC
* Roles
* RoleBindings
* ServiceAccounts
* Horizontal Pod Autoscaling
* Resource requests and limits
* Prometheus monitoring
* Grafana dashboards
* NGINX
* Docker containerization

---

## 🏗️ Architecture

```text
                         ┌─────────────────────┐
                         │       Client        │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Kubernetes Ingress  │
                         │    NGINX Ingress    │
                         └──────────┬──────────┘
                                    │
                         ┌──────────┴──────────┐
                         │                     │
                         ▼                     ▼
                ┌─────────────────┐   ┌─────────────────┐
                │ Django Service  │   │ NGINX Service   │
                │    Port 8005    │   │    Port 8090    │
                └────────┬────────┘   └────────┬────────┘
                         │                     │
                         ▼                     ▼
                ┌─────────────────┐   ┌─────────────────┐
                │ Django Pods     │   │   NGINX Pods    │
                │   2 replicas    │   │   2 replicas    │
                └────────┬────────┘   └─────────────────┘
                         │
                         │
                  HPA scales pods
                         │
                         ▼
                ┌─────────────────┐
                │  1 → 4 replicas │
                │ based on CPU     │
                └─────────────────┘


       ┌────────────────────────────────────────────┐
       │              Observability                 │
       │                                            │
       │       Prometheus → Metrics → Grafana      │
       └────────────────────────────────────────────┘


       ┌────────────────────────────────────────────┐
       │              Kubernetes RBAC               │
       │                                            │
       │ ServiceAccount / User                      │
       │        ↓                                   │
       │ RoleBinding                                │
       │        ↓                                   │
       │ Role                                       │
       │        ↓                                   │
       │ Allowed Kubernetes Resources & Verbs       │
       └────────────────────────────────────────────┘
```



## 🧰 Tech Stack

| Category         | Technology               |
| ---------------- | ------------------------ |
| Backend          | Django                   |
| Language         | Python 3.9               |
| Containerization | Docker                   |
| Orchestration    | Kubernetes               |
| Reverse Proxy    | NGINX                    |
| Ingress          | NGINX Ingress Controller |
| Access Control   | Kubernetes RBAC          |
| Monitoring       | Prometheus               |
| Visualization    | Grafana                  |
| Autoscaling      | Kubernetes HPA           |
| Frontend         | React                    |


---

## 🐍 Application

The main application is a Django-based Notes application.

The backend exposes the application through a container running:

```text
Python 3.9
Django
Django REST Framework
```

The application is packaged into a Docker image using the project's `Dockerfile`.

The container runs Django on:

```text
0.0.0.0:8000
```

Using Kubernetes then exposing the application internally through a `ClusterIP` service on port `8005`.

---

## ☸️ Kubernetes Architecture

All application resources are organized inside a dedicated namespace:

```text
notes-app-ns
```

This provides isolation from workloads running in other namespaces.

### Django Deployment

The Django application is deployed using a Kubernetes `Deployment`.

Initial configuration:

```text
Replicas: 2

CPU Request:    200m
CPU Limit:      400m

Memory Request: 256Mi
Memory Limit:   512Mi
```

Resource requests and limits provide Kubernetes with information about the resources required by each pod.

---

## 🔐 Kubernetes RBAC

One of the main objectives of this project is practicing **Kubernetes Role-Based Access Control**.

The project defines:

### Role

A namespace-scoped `Role` named:

```text
cluster-manager
```

The role grants permissions over Kubernetes resources including:

* Pods
* Deployments
* Services

Allowed operations include:

```text
get
watch
create
delete
patch
apply
```

This demonstrates the principle of giving a Kubernetes identity only the permissions required within a specific namespace.

---

## 👤 RoleBinding

The `RoleBinding` connects the Kubernetes identity to the role.

```text
User: usera
      │
      ▼
RoleBinding
      │
      ▼
cluster-manager Role
      │
      ▼
Permissions inside notes-app-ns
```


## 🪪 ServiceAccount

The project also defines a Kubernetes `ServiceAccount`:

```text
usera
```

inside:

```text
notes-app-ns
```

This provides an example of Kubernetes identities that can be used by workloads and Kubernetes authentication mechanisms.

---

## 🌐 Kubernetes Ingress

The project uses an NGINX Ingress configuration to route traffic to different Kubernetes services.

### Application route

```text
/
  ↓
django-app-service
  ↓
Django application
```

### NGINX route

```text
/nginx
  ↓
nginx-app-service
  ↓
NGINX pods
```

This demonstrates path-based routing through Kubernetes Ingress.

---

## ⚖️ Horizontal Pod Autoscaling

The Django deployment includes a **Horizontal Pod Autoscaler (HPA)**.

Configuration:

```text
Minimum replicas: 1
Maximum replicas: 4

CPU target utilization: 90%
```

Conceptually:

```text
                 CPU Usage
                    │
                    ▼
              ┌───────────┐
              │    HPA    │
              └─────┬─────┘
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
     Low CPU usage       High CPU usage
          │                   │
          ▼                   ▼
    Fewer replicas       More replicas
```

This allows Kubernetes to automatically adjust the number of Django pods based on CPU utilization.

---

## 📊 Monitoring & Observability

The project integrates **Prometheus and Grafana** for Kubernetes monitoring.

### Prometheus

Prometheus is responsible for collecting and storing metrics from the Kubernetes environment.

The project uses the Prometheus Kubernetes monitoring stack to provide metrics for cluster workloads.

### Grafana

Grafana is used to visualize the collected metrics through dashboards.

This provides visibility into areas such as:

* CPU usage
* Memory usage
* Pod health
* Kubernetes workloads
* Resource utilization
* Cluster performance

The monitoring architecture can be summarized as:

```text
Kubernetes
    │
    │ Metrics
    ▼
Prometheus
    │
    │ Query
    ▼
Grafana
    │
    ▼
Monitoring Dashboards
```

---

## 🐳 Docker

The Django application is containerized using Docker.

The Dockerfile is based on:

```text
python:3.9
```

The application dependencies are installed from:

```text
requirements.txt
```

The container exposes:

```text
8000
```

and runs Django using:

```text
python manage.py runserver 0.0.0.0:8000
```

---

## 📁 Repository Structure

```text
.
├── api/
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   ├── urls.py
│   └── migrations/
│
├── k8s/
│   ├── namespace.yml
│   ├── deployment.yml
│   ├── service.yml
│   ├── ingress.yml
│   ├── rbac.yml
│   ├── role-binding.yml
│   ├── service-account.yml
│   ├── hpa.yml
│   ├── vpa.yml
│   ├── nginx-deployment.yml
│   └── nginx-service.yml
│
├── mynotes/
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── db.json
│
├── Dockerfile
├── docker-compose.yml
├── manage.py
├── requirements.txt
└── README.md
```

---

## 🔑 Kubernetes Concepts Demonstrated

### Namespace

Provides logical isolation for application resources.

```text
notes-app-ns
```

### Deployment

Manages the desired number of Django and NGINX pods.

### Service

Provides stable networking for Kubernetes workloads.

The Django service uses:

```text
ClusterIP
```

### Ingress

Routes external HTTP requests to internal Kubernetes services.

### RBAC

Controls what Kubernetes identities are allowed to do.

### Role

Defines namespace-scoped permissions.

### RoleBinding

Associates a role with a user or identity.

### ServiceAccount

Provides a Kubernetes identity for workloads.

### HPA

Automatically adjusts pod replicas according to CPU utilization.

### Resource Requests & Limits

Controls how much CPU and memory pods request and are allowed to consume.

---

## 🎯 What This Project Demonstrates

This project provides practical experience with:

* Kubernetes Deployments
* Kubernetes Services
* Kubernetes Namespaces
* Kubernetes Ingress
* NGINX
* Kubernetes RBAC
* Roles
* RoleBindings
* ServiceAccounts
* Resource requests and limits
* Horizontal Pod Autoscaling
* Docker
* Django
* Django REST Framework
* Prometheus
* Grafana
* Kubernetes observability

---

## 🔐 Security Considerations

This project demonstrates basic Kubernetes access control, but a production environment would require additional hardening.

Recommended improvements include:

* Follow least-privilege RBAC.
* Use dedicated ServiceAccounts for workloads.
* Use Kubernetes NetworkPolicies.
* Monitor authentication and authorization events.

---

## 📌 Project Status

This is a **DevOps and Kubernetes learning project** focused on understanding how application workloads, access control, networking, autoscaling, and observability work together inside Kubernetes.

The main learning areas are:

```text
Application
     ↓
Docker
     ↓
Kubernetes
     ├── Deployment
     ├── Service
     ├── Ingress
     ├── RBAC
     ├── HPA
     └── Resource Management
            ↓
       Prometheus
            ↓
         Grafana
```

---

## ⭐ Project Goal

The goal of this project was built to gain practical experience with Kubernetes beyond simply deploying containers.

It combines **application deployment, security, networking, autoscaling, and monitoring** into one Kubernetes environment:

```text
Docker
  ↓
Django Application
  ↓
Kubernetes
  ├── RBAC
  ├── Ingress
  ├── Services
  ├── HPA
  └── Resource Management
          ↓
      Prometheus
          ↓
       Grafana
```

This project demonstrates how Kubernetes can be used not only to run applications, but also to control access, manage traffic, automatically scale workloads, and monitor the health of the environment.
