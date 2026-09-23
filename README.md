# Node.js-MySQL-Containerized-App
Backend application built with Node.js and MySQL, containerized using Docker and Docker Compose, with Kubernetes/K3s deployment.

## Tech Stack

* **Node.js** — Backend application
* **MySQL** — Relational database
* **Docker** — Containerization
* **Docker Compose** — Multi-container setup
* **K3s** — Container orchestration

## Architecture

The application consists of separate containers for the Node.js application and MySQL database.

```text
                    Docker Network
                 ┌───────────────────┐
                 │                   │
                 │  Node.js App      │
                 │  :3000            │
                 │       │           │
                 │       │ :3306     │
                 │       ▼           │
                 │  MySQL            │
                 │  :3306            │
                 │                   │
                 └───────────────────┘
```

The Node.js application communicates with MySQL through the Docker network using the MySQL service name.

## Docker

Build the Node.js image:

```bash
docker build -t node-mysql-app .
```

Run the Node.js container:

```bash
docker run --name nodeapp --network app_default -p 3000:3000 node-mysql-app
```

## Docker Compose

MySQL is configured using Docker Compose.

The services communicate through the Compose network using service names.

Example MySQL connection from Node.js:

```javascript
const db = mysql.createConnection({
  host: "mysqldb",
  port: 3306,
  user: "admin",
  password: process.env.DB_PASSWORD,
  database: "crudDB",
});
```

Here, `mysqldb` is the MySQL service name defined in the Compose configuration.

## K3s Deployment

The application can also be deployed on a Kubernetes (K3s) cluster instead of Docker Compose, using separate Deployments and Services for the Node.js app and MySQL.

Images are loaded into containerd directly (instead of pulled from a registry), and each Deployment uses `imagePullPolicy: Never` to use the locally imported image.

```bash
sudo k3s ctr images import node-mysql-app.tar
sudo k3s ctr images import mysql.tar
```

Apply the manifests:

```bash
kubectl apply -f mysql-secret.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-service.yaml
kubectl apply -f nodeapp-deployment.yaml
kubectl apply -f nodeapp-service.yaml
```

Inside the cluster, the Node.js app connects to MySQL using the Kubernetes Service name (e.g. `mysqldb`) instead of a Docker Compose network alias — the connection code stays the same, only the underlying network changes.

Check the deployment status:

```bash
kubectl get pods
kubectl get svc
```

## Application Port

The Node.js application runs on:

```text
http://localhost:3000
```

The container port is published using:

```bash
-p 3000:3000
```
