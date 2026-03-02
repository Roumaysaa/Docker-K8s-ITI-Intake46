## Lab 2 – Docker & Flask Web Development

### Overview

This lab demonstrates building and deploying a Python Flask application using Docker, as well as setting up a custom Docker network and running a web server container. The work is divided into two parts:

1. **Part 1:** Build and run a Flask app inside a Docker container, push the image to Docker Hub.
2. **Part 2:** Create a custom Docker network and run an Nginx container serving a custom index page.

### Part 1 – Flask Application in Docker

**Steps :**

- Cloned Repository from `basic-flask-app`.
- **1-Updated Dependencies**: Edited `requirements.txt` to change:
  - `MarkupSafe==1.0`
    + `MarkupSafe==1.1.1`
  - This update was necessary to successfully build the Docker image.

**2-Dockerfile**:
```dockerfile
FROM python:3.6-alpine
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "routes.py"]
```

**3-Build Image:**
```
docker build -t roumaysaa/iti-flask-lab2 .
```

**4-Run Container with Memory Limit & Port Mapping:**
```
docker run -d --name pyflask -p 127.0.0.1:80:5000 -m 100m iti-flask-lab2
```

**5-Verify Memory & Port Binding:**
```
docker inspect pyflask | grep "Memory"
```

**6-Push Image to Docker Hub:**
```
docker push roumaysaa/iti-flask-lab2
```

✅ Image successfully published to Docker Hub: `docker.io/roumaysaa/iti-flask-lab2`

### Part 2 – Web Server with Custom Network

- **1-Create Custom Network:**
```
docker network create --subnet 10.0.0.0/8 iti-network
```
- **2-Verify Network:**
```
docker network ls
```
- **3-Custom Index Page** (stored in `index.html`):
```html
<h1>Lab 2 ITI - Roumaysaa</h1>
```
- **4-Run Nginx Container with Volume Mount:**
```
docker run -d --name webserver-iti --network iti-network -p 8080:80 -v "/home/roma/Docker labs/basic-flask-app/index.html":/usr/share/nginx/html/index.html nginx:alpine
```
- **5-Access Web Page**: Open browser at: `http://localhost:8080` which displays:

```
Lab 2 ITI - Roumaysaa
```

### Results

- Flask app container built and pushed to Docker Hub.
- Custom Docker network created with subnet `10.0.0.0/8`.
- Nginx container serving a personalized index page via port 8080.

### Notes

- Dependency update: MarkupSafe was changed from `1.0` → `1.1.1` in `requirements.txt` to resolve build issues.
