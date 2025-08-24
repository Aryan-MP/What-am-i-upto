# 🚀 DevOps Notes (Linux, Networking, Jenkins, Docker)

---

## 🔹 Linux Systemd Services

### How to start your application like a system service (`systemctl`)

1. Create a service file in `/etc/systemd/system/` (e.g., `myapp.service`).

   ```ini
   [Unit]
   Description=My Custom Application Service

   [Service]
   ExecStart=/usr/bin/python3 /path/to/app.py
   Restart=always   # (optional: restart on crash)

   [Install]
   WantedBy=multi-user.target
   ```

2. Reload systemd after saving:

   ```bash
   sudo systemctl daemon-reload
   ```

3. Start and enable:

   ```bash
   sudo systemctl start myapp
   sudo systemctl enable myapp   # starts at boot
   ```

👉 Useful commands:

```bash
systemctl status myapp
systemctl stop myapp
journalctl -u myapp   # logs
```

---

## 🌐 Networking Basics

### Common `ip` commands

- `ip link` → List interfaces  
- `ip addr` → Show IP addresses  
- `ip addr add <IP>/<CIDR> dev eth0` → Add IP to interface  
- `ip route` → Show routing table  
- `ip route add <targetIP> via <routerIP>` → Add static route  

### Packet Forwarding

```bash
cat /proc/sys/net/ipv4/ip_forward
```

- `1` = enabled (forwarding allowed)  
- `0` = disabled  

### Hostname Resolution

- Local mapping → `/etc/hosts`  
  ```
  192.168.1.10   myserver
  ```

- For dynamic or large-scale resolution → **DNS**.  
  - Config file: `/etc/resolv.conf`  
  - Example:
    ```
    nameserver 8.8.8.8
    nameserver 1.1.1.1
    ```

---

## ⚙️ Jenkins Pipelines

### Minimal Example

```groovy
pipeline {
    agent any
    stages {
        stage("Build Docker Image") {
            steps {
                script {
                    docker.build("myorg/myapp")
                }
            }
        }
    }
}
```

---

### Realistic Example (Go App + Docker Push)

```groovy
pipeline {
    agent any

    environment {
        APP = 'go-webapp-sample'
        TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage("Checkout") {
            steps {
                git 'https://github.com/kodekloudhub/go-webapp-sample.git'
            }
        }

        stage("Build") {
            steps {
                sh 'go build -o app .'
            }
        }

        stage("Test") {
            steps {
                sh 'go test ./...'
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    def image = docker.build("adminturneddevops/${APP}:${TAG}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials-id') {
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build & Push completed"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
```

---

### Jenkinsfile Template

```groovy
pipeline {
    agent any
    environment {
        KEY = "value"   // global variables
    }

    options {
        timeout(time: 30, unit: 'MINUTES') // pipeline-wide settings
    }

    stages {
        stage("Stage 1") {
            steps {
                echo "Hello World"
            }
        }
    }

    post {
        always {
            echo "Cleanup actions"
        }
        success {
            echo "Pipeline succeeded"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
```

---

## 🐳 Docker Basics

### Common Commands

- `docker images` → List images  
- `docker pull nginx` → Pull an image  
- `docker ps` → Running containers  
- `docker ps -a` → All containers (including stopped)  
- `docker stop <id|name>` → Stop container  
- `docker rm <id|name>` → Remove container  
- `docker rmi <image>` → Remove image  
- `docker run -it ubuntu bash` → Run interactive container  
- `docker run -d nginx` → Run in background (detached)  

### Container Lifecycle

- Containers are **ephemeral** → once process ends, container stops.  

### Attach/Detach

- **Foreground**:
  ```bash
  docker run nginx
  ```
- **Background**:
  ```bash
  docker run -d nginx
  ```
- **Reattach**:
  ```bash
  docker attach <container_id>
  ```

⚡ One-liner for **complete cleanup**:
```bash
docker stop $(docker ps -q) && docker rm $(docker ps -aq) && docker rmi $(docker images -q)
```

### `docker run` Modes

- `docker run` → run the app in foreground  
- `docker run -d` → run in **detached mode**  
- `docker run -i` → run in **interactive mode** (input allowed)  
- `docker run -it` → run in **interactive + pseudo-terminal mode**  

### Port Mapping in Docker

- Map container’s port to host:
  ```bash
  docker run -p 80:5000 image
  ```

### Volume Mapping

- To persist data across container restarts:  
  ```bash
  docker run -v /host/dir:/container/dir image
  ```

### Logs & Inspect

- `docker logs <container>` → view logs  
- `docker inspect <container>` → detailed info  

---

## 🏗️ Creating Your Own Docker Image

### Example Dockerfile

```dockerfile
FROM ubuntu

RUN apt-get update
RUN apt-get install -y python3 python3-pip

RUN pip install flask flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run --host=0.0.0.0
```

### Basic Template

```dockerfile
FROM <base-image>
LABEL key=value
ENV KEY=value
WORKDIR /app
COPY . .
RUN <commands>
EXPOSE <port>
CMD ["executable","param1","param2"]
```

### Go Web App (Multi-stage Build)

```dockerfile
# Build stage
FROM golang:1.20-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o app .

# Final stage
FROM alpine:3.18
WORKDIR /app
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]
```

### Build & Run Commands

```bash
docker build -t my-go-webapp:latest .
docker run -p 8080:8080 my-go-webapp:latest
docker images
docker ps
```

### ENTRYPOINT vs CMD

- **ENTRYPOINT** → fixed executable, runtime args are appended  
- **CMD** → default command, overridden if args passed  

---

## 🌐 Docker Networking

### Custom Network Example

```bash
docker network create   --subnet=182.18.0.0/24   --gateway=182.18.0.1   wp-mysql-network
```

### Docker Registry

- Private registry can be set up using:
  ```bash
  docker run -d -p 5000:5000 --name registry registry:2
  ```
- Push images to your private registry:
  ```bash
  docker tag myapp localhost:5000/myapp
  docker push localhost:5000/myapp
  ```

---
✅ End of Notes  
