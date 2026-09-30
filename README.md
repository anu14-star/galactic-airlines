# Galactic Airlines Application

A containerized three-tier web application consisting of a frontend served by Nginx, a Flask REST API backend, and a MySQL database.

## 1. Application Architecture

Browser → Nginx Frontend → Flask REST API → MySQL

Application flow:

Browser
↓
Nginx Frontend
↓
/api/*
↓
Flask Backend API
↓
MySQL Database

The browser communicates with the application through `/api/...` endpoints.

Nginx reverse-proxies `/api/...` requests to the internal backend service. The browser does not directly access or resolve the internal Docker/Kubernetes service name.

## 2. Features

* Add flight
* View flights
* Delete flight
* API health check
* REST API backend using Flask
* MySQL database
* Nginx frontend web server
* Nginx reverse proxy for `/api`
* Dockerized frontend, backend and database
* Environment-based database configuration
* Deployable to Kubernetes/Amazon EKS

## 3. Application Components

### Frontend

* HTML
* CSS
* JavaScript
* Nginx
* Serves the application UI
* Proxies `/api/*` requests to the backend

### Backend

* Python
* Flask
* REST API
* Handles flight operations
* Connects to MySQL

### Database

* MySQL
* Stores flight/application data

## 4. Run Locally

Make sure Docker and Docker Compose are installed.

Copy the example environment file:

```bash
cp .env.example .env
```

Update the required database configuration in `.env`.

Build and start the application:

```bash
docker compose up -d --build
```

Check running containers:

```bash
docker compose ps
```

Open the application:

http://localhost:8080

## 5. Verify Application

Check the API health endpoint:

```bash
curl http://localhost:8080/api/health
```

A successful response confirms that the request reached the application through Nginx and the backend API is responding.

Check application logs if required:

```bash
docker compose logs
```

Check individual service logs:

```bash
docker compose logs frontend
docker compose logs backend
docker compose logs database
```

## 6. API Flow

The frontend sends requests using paths such as:

```text
/api/flights
/api/health
```

Nginx receives these requests and forwards them to the internal backend service.

Example:

```text
Browser
   |
   | GET /api/flights
   v
Nginx
   |
   | proxy_pass
   v
Backend / Flask
   |
   v
MySQL
```

The browser never uses an internal service address such as:

```text
http://backend:5000
```

This keeps internal service discovery separate from browser-side application logic.

## 7. Docker Architecture

```text
                 Docker Network
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
   Frontend         Backend        Database
    Nginx           Flask           MySQL
     :80             :5000           :3306
        |
        +---- /api/* ---> Backend
```

Only the frontend is exposed to the user.

The backend and database communicate through the internal Docker network.

## 8. Kubernetes / EKS Deployment

The same application is designed to run on Kubernetes and Amazon EKS.

Kubernetes architecture:

```text
Internet
   |
AWS Load Balancer
   |
Frontend Service
   |
Frontend Pod / Nginx
   |
Backend Service
   |
Backend Pod / Flask
   |
Database Service
   |
MySQL Pod
```

Current Kubernetes service exposure:

* Frontend: `LoadBalancer`
* Backend: `ClusterIP`
* Database: `ClusterIP`

The backend and database are therefore internal services, while the frontend provides the external entry point.

## 9. Configuration

Database configuration is supplied through environment variables rather than being hard-coded into the application.

Example variables:

```text
DB_HOST
DB_PORT
DB_USER
DB_PASSWORD
DB_NAME
```

For Kubernetes, these values can be provided through ConfigMaps and Secrets.

Sensitive credentials should not be committed to GitHub.

## 10. Project Structure

```text
galactic-airlines/
├── frontend/
│   ├── index.html
│   ├── css/
│   ├── js/
│   ├── nginx/
│   └── Dockerfile
│
├── backend/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── database/
│   ├── init.sql
│   └── Dockerfile
│
├── docker-compose.yml
├── .env.example
└── README.md
```

## 11. Deployment Goal

The application can be developed and tested locally using Docker Compose and then deployed as containerized workloads on Kubernetes/Amazon EKS.

The application code remains separate from infrastructure configuration such as:

* Kubernetes manifests
* Jenkins pipeline
* Terraform
* Ansible

This separation allows the same application to be deployed across different environments without duplicating application logic.
