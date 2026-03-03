## Lab #2 – Docker, Flask, & Custom Networking

This repository documents the implementation of Lab #2, focusing on building a custom Python Flask application, managing dependencies within a Dockerfile, and orchestrating containers within a custom-defined Docker network.

### 🛠️ Task 1: Flask Application Development & Dockerization

The first phase involved modifying a Flask application to ensure environment compatibility and deploying it using a custom Docker image.

1. **Dependency Management**  
To resolve build issues, the `requirements.txt` file was updated to use a more stable version of the MarkupSafe library.

   - Update: Changed `MarkupSafe==1.0` to `MarkupSafe==1.1.1`.
     
- This update was necessary to successfully build the Docker image.
  
<img width="500" height="200" alt="editing requirements txt" src="https://github.com/user-attachments/assets/90444e26-8f74-42f1-ba01-5d875767e665" />


2. **Dockerfile Configuration**  
A Dockerfile was created using a  Alpine-based Python image to minimize the image size and ensure a clean environment.

   ```dockerfile
   FROM python:3.6-alpine
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY . .
   EXPOSE 5000
   CMD ["python", "routes.py"]
   ```

3. **Build and Deployment**  
The image was built and then executed with specific resource constraints and port bindings.

   -**Build Image:**:
     ```bash
     docker build -t roumaysaa/iti-flask-lab2 .
     ```

   - **Run Container with Memory Limit & Port Mapping:**

     ```bash
     docker run -d --name pyflask -p 127.0.0.1:80:5000 -m 100m iti-flask-lab2
     ```

   - **Resource Limit**: Restricted the container to 100 MB of RAM.

   - **Verification: Confirmed settings using:**
     ```bash
     docker inspect pyflask | grep "Memory"
     ```
<img width="700" height="100" alt="memory limit" src="https://github.com/user-attachments/assets/193999c1-64bd-45ae-88b1-6fb191a025f0" />

<img width="300" height="150" alt="port binding of pyflask" src="https://github.com/user-attachments/assets/8326223b-8c58-411e-97ea-3ce72231615e" />

    **Verification: http:127.0.0.1 :**
    
<img width="1139" height="977" alt="verify of pyflask" src="https://github.com/user-attachments/assets/1effd6ad-7c73-4a63-8e06-79aa1304ca44" />


4. **Push Image to Docker Hub**  
The final verified image was pushed to a public repository for remote deployment.

   - Repository: `docker.io/roumaysaa/iti-flask-lab2`
   - Command:
     ```bash
     docker push roumaysaa/iti-flask-lab2
     ```
✅ Image successfully published to Docker Hub: `docker.io/roumaysaa/iti-flask-lab2`

<img width="700" height="300" alt="verification of pushing" src="https://github.com/user-attachments/assets/24c01235-df82-4361-999c-c573e7c706ec" />

### 🌐 Task 2: Web Server with Custom Network

This task demonstrated how to isolate containers within a specific subnet and serve custom content via volume mounting.

1. **Custom Network Creation**  
A dedicated Docker network was established to control the IP address space.

   - Subnet: `10.0.0.0/8`
   - Network Name: `iti-network`
   - Command:
     ```bash
     docker network create --subnet 10.0.0.0/8 iti-network
     ```

   - Verification:
     ```bash
     docker network ls
     ```
2. **Custom Index Page** (stored in `index.html`):
   ```html
      <h1>Lab 2 ITI - Roumaysaa</h1>
    ```

3. **Run Nginx Container with Volume Mount:**  
An Nginx container was launched to serve a HTML page by mounting a local file into the container’s web root.

   - Local Path: `/home/roma/Docker labs/basic-flask-app/index.html`
   - Container Path: `/usr/share/nginx/html/index.html`
   - Edit nginx configuration to listen to port 8080 instead of port 80 by creating nginx.conf

   <img width="600" height="200" alt="nginx conf" src="https://github.com/user-attachments/assets/4a6f4a8a-8510-416a-90ca-8ed9f45a68bc" />
   
   - Command:
     ```bash
      docker run -d --name webserver-iti --network iti-network -p 8080:8080 -v "/home/roma/Docker labs/basic-flask-app/index.html":/usr/share/nginx/html/index.html -v "/home/roma/Docker labs/basic-flask-app/nginx.conf":/etc/nginx/nginx.conf  nginx:alpine
     ```
     
3. **Verification**  
Accessing http://localhost:8080 successfully which displays : "Lab 2 ITI - Roumaysaa".

<img width="700" height="156" alt="verify web" src="https://github.com/user-attachments/assets/e1e6a642-f9db-4af9-b087-8e6a29afe89a" />

### Results

- Flask app container built and pushed to Docker Hub.
- Custom Docker network created with subnet `10.0.0.0/8`.
- Nginx container serving a index page via port 8080.

### Notes

- Dependency update: MarkupSafe was changed from `1.0` → `1.1.1` in `requirements.txt` to resolve build issues.
