# 8. Lab 05: Docker พื้นฐาน

## วัตถุประสงค์

- ติดตั้ง Docker
- เข้าใจ image/container/network/volume
- เขียน Dockerfile
- build และ run container ได้

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Dockerfile | recipe สำหรับสร้าง image |
| Image | package ของ app |
| Container | process ที่รันจาก image |
| Volume | storage ที่อยู่รอดแม้ container ถูกลบ |
| Docker Network | network ภายใน Docker |

## ติดตั้ง Docker

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker devops
newgrp docker
```

ตรวจสอบ:

```bash
docker version
docker run hello-world
```

## Dockerfile สำหรับ simple-api

```bash
cd ~/apps/simple-api
nano Dockerfile
```

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY index.js ./
ENV PORT=3000
EXPOSE 3000
CMD ["node", "index.js"]
```

สร้าง `.dockerignore`:

```bash
nano .dockerignore
```

```text
node_modules
npm-debug.log
.git
.env
```

Build:

```bash
docker build -t simple-api:1.0.0 .
```

Run:

```bash
docker run -d --name simple-api -p 3000:3000 simple-api:1.0.0
```

## การทดสอบ

```bash
curl http://localhost:3000
curl http://localhost:3000/health
```

## การตรวจสอบ

```bash
docker ps
docker logs simple-api
docker exec -it simple-api sh
```

ลบ container:

```bash
docker stop simple-api
docker rm simple-api
```

## ข้อสรุป

Docker ช่วยให้ app มี runtime ที่เหมือนกันในทุก environment ลดปัญหา dependency และ config ไม่ตรงกัน

---
