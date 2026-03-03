# Lab #3 – Docker Private Registry & Multi-Service Orchestration

This repository documents the implementation of Lab #3, which focuses on deploying a private Docker registry, orchestrating multi-container applications using Docker Compose (WordPress & MySQL), and setting up Nginx as a reverse proxy for a Flask application.

## 🛠️ Task 1: Private Docker Registry & Custom Image

The first phase involved setting up a local, insecure registry to host private images and building a custom Nginx image based on Alpine Linux.

### 1. Registry Deployment

A local Docker registry was deployed to manage internal images.

**Command:**
```bash
docker run -d --name my-registry -p 5000:5000 --restart=always registry:2
```

**Configuration:** 
To allow communication with the local registry, the Docker daemon was updated to include `localhost:5000` as an insecure registry in `/etc/docker/daemon.json`.

-File: /etc/docker/daemon.json 

-Content: ```bash
{ "insecure-registries": ["localhost:5000"] }
     ```

### 2. Custom Nginx Image

Custom Nginx image was built from scratch using the Alpine Linux base.

**Dockerfile:**
```dockerfile
FROM alpine:latest
RUN apk update && apk add nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
<img width="400" height="150" alt="Dockerfile of nginx-alpine" src="https://github.com/user-attachments/assets/89baeb9e-a24c-4f00-8eec-0f47dc681d41" />

**Build & Tag:** The image was built and tagged specifically for the local registry.

```bash
docker build -t localhost:5000/nginx-alpine:v1 .
```
<img width="700" height="150" alt="build nginx-alpine" src="https://github.com/user-attachments/assets/e057d75e-4d08-4435-ade7-46745ee74338" />

---

## 🌐 Task 2: WordPress & MySQL Orchestration

The second task utilized Docker Compose to deploy a stateful WordPress site with a persistent MySQL database.

### 1. Environment Composition

The `docker-compose.yml` was configured to manage two services: `wordpress-app` and `wordpress-db`.

- **Database (MySQL 5.7):** Configured with root credentials and a persistent volume mount to `./mysql-data` on the host.
- **Application (WordPress):** Linked to the database service and exposed on host port 8080.
  
<img width="700" height="400" alt="docker-compose of wordpress   mysql" src="https://github.com/user-attachments/assets/33bdbcf1-4603-4fc4-be79-ceb7a72f7130" />

### 2. Deployment

**Command:**
```bash
docker compose up -d
```
<img width="700" height="400" alt="compose up of wp" src="https://github.com/user-attachments/assets/1925129f-d652-4667-8189-68447d0bf07b" />

**Verification:** The deployment was verified by accessing the WordPress installation dashboard at `http://localhost:8080`.

<img width="700" height="400" alt="wordpress app" src="https://github.com/user-attachments/assets/ea846283-5e31-4324-8fad-6b4da2f46934" />

---

## 🚀 Task 3: Reverse Proxy for Flask App

In this section, a Flask application image was pushed to the private registry and served through an Nginx reverse proxy.

### 1. Registry Integration

The Flask image from Lab 2 (iti-flask-lab2:latest) was tagged and pushed to the local registry.

**Push Command:**
```bash
docker push localhost:5000/iti-flask-lab2:latest
```
<img width="700" height="150" alt="push flask image to private register" src="https://github.com/user-attachments/assets/ee50ce96-4c85-4065-b1df-14a7a8e868f2" />

### 2. Nginx Reverse Proxy Configuration

A custom `nginx.conf` was created to route traffic from Nginx (port 80) to the Flask application.

**Config Fragment:**
```nginx
server {
    listen 80;
    location / {
        proxy_pass http://flask-app:5000;
    }
}
```

### 3. Multi-Service Stack

A Docker Compose file orchestrated the Flask app and the Nginx proxy as a single stack.

- **Flask Service:** Pulls the image from `localhost:5000/iti-flask-lab2`.
- **Nginx Service:** Mounts the custom configuration and depends on the Flask app.

<img width="685" height="400" alt="Docker-compose of flask   nginx" src="https://github.com/user-attachments/assets/e2018aac-e958-4239-b48e-4b2d51572d41" />

### 4. Deployment

**Command:**
```bash
docker compose up 
```
<img width="700" height="400" alt="compose up of flask" src="https://github.com/user-attachments/assets/7c00fa50-e53e-4702-97bd-b0f22a038239" />

### 5.Result: 

The application, featuring the "Tiger Home Page," was successfully served via the proxy.

<img width="500" height="300" alt="flask app" src="https://github.com/user-attachments/assets/072a44ce-82d0-48ed-a1f5-d62c2ec0c150" />

---
