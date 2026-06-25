# DevOps Lab ภาษาไทย สำหรับจำลองบน VMware ที่บ้าน

เอกสารนี้ออกแบบสำหรับคนที่มีพื้นฐาน Fullstack Developer แล้ว ต้องการฝึก DevOps แบบลงมือทำจริงบน PC ที่บ้านด้วย VMware โดยครอบคลุมการติดตั้งเครื่องมือ วิธีการใช้งาน คำศัพท์ การทดสอบ การตรวจสอบ และข้อสรุปของแต่ละหัวข้อ

---

## สารบัญ

1. ภาพรวม Lab
2. สเปกเครื่องและ Network ที่แนะนำ
3. คำศัพท์ DevOps สำคัญ
4. Lab 01: เตรียม VM และ Ubuntu Server
5. Lab 02: Linux, SSH, Service และ Log
6. Lab 03: Network, DNS, Port และ Firewall
7. Lab 04: Nginx Reverse Proxy และ Manual Deploy
8. Lab 05: Docker พื้นฐาน
9. Lab 06: Docker Compose สำหรับ Fullstack App
10. Lab 07: Private Container Registry
11. Lab 08: Git Flow และ Release Version
12. Lab 09: CI/CD ด้วย GitLab CE และ GitLab Runner
13. Lab 10: Terraform และ Infrastructure as Code
14. Lab 11: Monitoring ด้วย Prometheus และ Grafana
15. Lab 12: Logging ด้วย Loki และ Promtail
16. Lab 13: Kubernetes Cluster ด้วย kubeadm
17. Lab 14: Deploy App เข้า Kubernetes
18. Lab 15: Ingress, ConfigMap, Secret และ Storage
19. Lab 16: Helm Chart
20. Lab 17: DevSecOps ด้วย Trivy
21. Lab 18: Backup, Restore และ DR พื้นฐาน
22. Final Project
23. Checklist หลังเรียนจบ
24. Troubleshooting รวม

---

# 1. ภาพรวม Lab

## เป้าหมาย

หลังทำ Lab นี้จบ คุณควรทำสิ่งเหล่านี้ได้:

- ติดตั้งและดูแล Linux Server เบื้องต้น
- ตรวจสอบ SSH, Service, Process, Port, DNS และ Firewall
- Deploy Web/API ด้วย Nginx และ Systemd
- เขียน Dockerfile และ Docker Compose
- สร้าง Private Container Registry
- ทำ Git Flow, Tag และ Release Version
- ทำ CI/CD Pipeline ด้วย GitLab CI
- ใช้ Terraform เพื่อเข้าใจ Infrastructure as Code
- เก็บ Metrics ด้วย Prometheus และแสดงผลด้วย Grafana
- รวม Log ด้วย Loki และ Promtail
- สร้าง Kubernetes Cluster บน VMware
- Deploy App เข้า Kubernetes พร้อม Service, Ingress, ConfigMap, Secret
- ใช้ Helm สำหรับจัดการ Kubernetes Manifest
- Scan Security ด้วย Trivy
- Backup และ Restore ระบบเบื้องต้น

## ภาพรวม Architecture

```text
PC ที่บ้าน
└── VMware Workstation / Player
    ├── devops-control     : GitLab, Registry, เครื่องควบคุม
    ├── app-server         : Deploy app แบบ VM/Docker
    ├── monitor-server     : Prometheus, Grafana, Loki
    ├── k8s-master-01      : Kubernetes Control Plane
    ├── k8s-worker-01      : Kubernetes Worker
    └── k8s-worker-02      : Kubernetes Worker
```

---

# 2. สเปกเครื่องและ Network ที่แนะนำ

## สเปก VM แบบประหยัด

| VM | CPU | RAM | Disk | Role |
|---|---:|---:|---:|---|
| devops-control | 2 vCPU | 4-6 GB | 60 GB | GitLab, Registry |
| app-server | 2 vCPU | 2 GB | 30 GB | Docker/App |
| monitor-server | 2 vCPU | 3-4 GB | 40 GB | Monitoring/Logging |
| k8s-master-01 | 2 vCPU | 3-4 GB | 40 GB | K8s Control Plane |
| k8s-worker-01 | 2 vCPU | 2-3 GB | 40 GB | K8s Worker |
| k8s-worker-02 | 2 vCPU | 2-3 GB | 40 GB | K8s Worker |

ถ้า RAM น้อย ให้เริ่มจาก 3 เครื่องก่อน:

```text
devops-control
app-server
k8s-master-01 + k8s-worker-01 เมื่อถึงบท Kubernetes
```

## Network Plan

แนะนำใช้ VMware NAT หรือ Host-only แล้วกำหนด Static IP

| Hostname | IP ตัวอย่าง | หน้าที่ |
|---|---|---|
| devops-control | 192.168.56.10 | GitLab, Registry |
| app-server | 192.168.56.20 | App, Docker |
| monitor-server | 192.168.56.30 | Prometheus, Grafana, Loki |
| k8s-master-01 | 192.168.56.100 | K8s Master |
| k8s-worker-01 | 192.168.56.101 | K8s Worker |
| k8s-worker-02 | 192.168.56.102 | K8s Worker |

## OS ที่ใช้

```text
Ubuntu Server 22.04 LTS หรือ 24.04 LTS
```

## User ที่ใช้ใน Lab

```text
devops
```

ให้ user นี้มีสิทธิ์ sudo

---

# 3. คำศัพท์ DevOps สำคัญ

| คำศัพท์ | ความหมาย |
|---|---|
| DevOps | แนวทางเชื่อม Development และ Operations เพื่อให้ build/test/deploy/monitor ทำได้เร็วและเสถียร |
| CI | Continuous Integration การ build และ test อัตโนมัติเมื่อมี code ใหม่ |
| CD | Continuous Delivery/Deployment การส่งมอบหรือ deploy อัตโนมัติ |
| Container | environment สำหรับรัน app ที่รวม dependency ไว้ในหน่วยเดียว |
| Image | template สำหรับสร้าง container |
| Registry | ที่เก็บ container image |
| IaC | Infrastructure as Code การจัดการ infra ด้วย code |
| Metrics | ตัวเลขวัดสถานะระบบ เช่น CPU, RAM, Request/sec |
| Log | บันทึกเหตุการณ์ของระบบ |
| Alert | การแจ้งเตือนเมื่อระบบผิดปกติ |
| Kubernetes | ระบบจัดการ container จำนวนมาก |
| Helm | package manager สำหรับ Kubernetes |
| Secret | ข้อมูลลับ เช่น password, token, key |
| Rollback | การย้อน version เมื่อ release มีปัญหา |

---

# 4. Lab 01: เตรียม VM และ Ubuntu Server

## วัตถุประสงค์

- สร้าง VM บน VMware
- ติดตั้ง Ubuntu Server
- ตั้ง hostname และ static IP
- เปิด SSH
- ตั้ง `/etc/hosts` ให้เรียกชื่อเครื่องได้

## ขั้นตอนติดตั้ง VM

1. สร้าง VM ใหม่ใน VMware
2. เลือก ISO Ubuntu Server
3. กำหนด CPU/RAM/Disk ตามตาราง
4. ตั้ง Network เป็น NAT หรือ Host-only
5. ระหว่างติดตั้งให้เลือกติดตั้ง OpenSSH Server
6. สร้าง user ชื่อ `devops`

## ตั้ง hostname

ตัวอย่างบน app-server:

```bash
sudo hostnamectl set-hostname app-server
hostnamectl
```

## ตั้ง Static IP ด้วย Netplan

ตรวจชื่อ interface:

```bash
ip a
```

แก้ไฟล์ netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

ตัวอย่าง:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.56.20/24
      routes:
        - to: default
          via: 192.168.56.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Apply:

```bash
sudo netplan apply
ip a
ip route
```

## ตั้งค่า /etc/hosts ทุกเครื่อง

```bash
sudo nano /etc/hosts
```

เพิ่ม:

```text
192.168.56.10 devops-control
192.168.56.20 app-server
192.168.56.30 monitor-server
192.168.56.100 k8s-master-01
192.168.56.101 k8s-worker-01
192.168.56.102 k8s-worker-02
```

## การทดสอบ

```bash
ping -c 4 app-server
ping -c 4 devops-control
ssh devops@app-server
```

## การตรวจสอบ

- `hostnamectl` แสดงชื่อเครื่องถูกต้อง
- `ip a` แสดง IP ตามที่กำหนด
- ping ข้ามเครื่องได้
- SSH เข้าเครื่องได้

## ข้อสรุป

พื้นฐาน VM, IP, hostname และ SSH ต้องนิ่งก่อนเริ่ม DevOps Lab เพราะทุกเครื่องมือจะพึ่งพา network และ hostname

---

# 5. Lab 02: Linux, SSH, Service และ Log

## วัตถุประสงค์

- ใช้คำสั่ง Linux สำคัญ
- ตรวจ process, disk, memory
- จัดการ service ด้วย systemd
- อ่าน log ด้วย journalctl

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Process | โปรแกรมที่กำลังทำงาน |
| Service | โปรแกรม background ที่ systemd จัดการ |
| Daemon | service ที่รันเบื้องหลัง |
| Permission | สิทธิ์ของไฟล์ เช่น read/write/execute |
| Log | บันทึกเหตุการณ์ของระบบ |

## คำสั่งพื้นฐาน

```bash
pwd
ls -lah
cd /var/log
cat /etc/os-release
whoami
uname -a
```

## ตรวจ resource

```bash
top
free -h
df -h
du -sh /var/log/*
```

## ตรวจ process

```bash
ps aux
ps aux | grep ssh
```

## systemctl

```bash
sudo systemctl status ssh
sudo systemctl restart ssh
sudo systemctl enable ssh
```

## journalctl

```bash
sudo journalctl -xe
sudo journalctl -u ssh
sudo journalctl -u ssh -f
```

## Permission

```bash
touch test.txt
ls -l test.txt
chmod 600 test.txt
ls -l test.txt
```

## การทดสอบ

1. เปิด terminal แรกแล้วดู log SSH

```bash
sudo journalctl -u ssh -f
```

2. เปิดอีก terminal แล้ว SSH เข้าเครื่อง

```bash
ssh devops@app-server
```

## การตรวจสอบ

- เห็น log การ login
- restart service ได้
- ดู disk/memory/process ได้

## ข้อสรุป

DevOps ต้องอ่านอาการระบบจาก Linux ได้ เช่น service ตาย, disk เต็ม, memory เต็ม, permission ผิด หรือ log error

---

# 6. Lab 03: Network, DNS, Port และ Firewall

## วัตถุประสงค์

- ตรวจ IP, route, DNS
- ตรวจ port ที่เปิดอยู่
- ทดสอบ HTTP ด้วย curl
- เปิด/ปิด firewall ด้วย UFW

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| IP Address | หมายเลขเครื่องใน network |
| Port | ช่องทางที่ service เปิดรับ traffic |
| DNS | ระบบแปลงชื่อเป็น IP |
| Firewall | ตัวควบคุม traffic เข้า/ออก |
| TCP | protocol แบบ connection-oriented |
| HTTP/HTTPS | protocol สำหรับ web |

## ตรวจ network

```bash
ip a
ip route
resolvectl status
```

## ตรวจ DNS

```bash
sudo apt update
sudo apt install -y dnsutils
nslookup google.com
dig google.com
```

## ตรวจ port

```bash
ss -tulpen
```

## ติดตั้ง Nginx ทดสอบ port 80

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

ทดสอบ:

```bash
curl http://localhost
curl http://app-server
```

## UFW Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status numbered
```

ทดสอบลบ rule port 80:

```bash
sudo ufw delete allow 80/tcp
curl http://app-server
sudo ufw allow 80/tcp
```

## การตรวจสอบ

- `ss -tulpen` เห็น Nginx listen port 80
- curl เข้าเว็บได้
- ปิด firewall rule แล้วเข้าไม่ได้
- เปิดกลับแล้วเข้าได้

## ข้อสรุป

การแก้ปัญหา production มักเริ่มจาก network: เครื่องถึงไหม, DNS ถูกไหม, port listen ไหม, firewall block ไหม

---

# 7. Lab 04: Nginx Reverse Proxy และ Manual Deploy

## วัตถุประสงค์

- Deploy API แบบ manual
- ใช้ systemd คุม app
- ใช้ Nginx เป็น reverse proxy
- เข้าใจ request flow ก่อนใช้ Docker/Kubernetes

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Reverse Proxy | server ที่รับ request แล้วส่งต่อไป backend |
| Upstream | backend service ปลายทาง |
| Systemd Service | service ที่ Linux จัดการให้ start/restart ได้ |
| Health Check | endpoint ใช้ตรวจว่า app ยังทำงานอยู่ |

## สร้าง Node.js API

```bash
sudo apt update
sudo apt install -y nodejs npm
mkdir -p ~/apps/simple-api
cd ~/apps/simple-api
npm init -y
npm install express
nano index.js
```

ใส่:

```javascript
const express = require('express');
const os = require('os');
const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({ message: 'Hello DevOps Lab', host: os.hostname() });
});

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(port, () => console.log(`API listening on ${port}`));
```

ทดสอบ:

```bash
node index.js
curl http://localhost:3000/health
```

## สร้าง systemd service

```bash
sudo nano /etc/systemd/system/simple-api.service
```

ใส่:

```ini
[Unit]
Description=Simple API Service
After=network.target

[Service]
Type=simple
User=devops
WorkingDirectory=/home/devops/apps/simple-api
Environment=PORT=3000
ExecStart=/usr/bin/node /home/devops/apps/simple-api/index.js
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Start service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now simple-api
sudo systemctl status simple-api
```

## ตั้งค่า Nginx

```bash
sudo nano /etc/nginx/sites-available/simple-api
```

```nginx
server {
    listen 80;
    server_name app-server;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/simple-api /etc/nginx/sites-enabled/simple-api
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

## การทดสอบ

```bash
curl http://app-server
curl http://app-server/health
```

## การตรวจสอบ

```bash
sudo journalctl -u simple-api -f
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

## ข้อสรุป

Manual deploy ช่วยให้เข้าใจว่า app ทำงานบน server อย่างไร ก่อนจะย้ายไป Docker, CI/CD และ Kubernetes

---

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

# 9. Lab 06: Docker Compose สำหรับ Fullstack App

## วัตถุประสงค์

- รันหลาย service พร้อมกัน
- มี frontend, backend, database, nginx
- ใช้ network และ volume

## โครงสร้าง

```text
fullstack-lab/
├── backend/
├── frontend/
├── nginx/
└── docker-compose.yml
```

สร้าง folder:

```bash
mkdir -p ~/labs/fullstack-lab/{backend,frontend,nginx}
cd ~/labs/fullstack-lab
```

## Backend

```bash
cd backend
npm init -y
npm install express pg
nano index.js
```

```javascript
const express = require('express');
const { Pool } = require('pg');
const app = express();
const port = process.env.PORT || 3000;
const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  port: 5432,
});
app.get('/api/health', (req, res) => res.json({ status: 'ok' }));
app.get('/api/db', async (req, res) => {
  const r = await pool.query('SELECT NOW() as now');
  res.json(r.rows[0]);
});
app.listen(port, () => console.log(`Backend running on ${port}`));
```

```bash
nano Dockerfile
```

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY index.js ./
EXPOSE 3000
CMD ["node", "index.js"]
```

## Frontend

```bash
cd ../frontend
nano index.html
```

```html
<!doctype html>
<html>
<body>
<h1>DevOps Fullstack Lab</h1>
<button onclick="checkApi()">Check API</button>
<pre id="result"></pre>
<script>
async function checkApi() {
  const res = await fetch('/api/health');
  const data = await res.json();
  document.getElementById('result').innerText = JSON.stringify(data, null, 2);
}
</script>
</body>
</html>
```

```bash
nano Dockerfile
```

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

## Nginx

```bash
cd ../nginx
nano default.conf
```

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://frontend:80;
    }
    location /api/ {
        proxy_pass http://backend:3000;
    }
}
```

## docker-compose.yml

```bash
cd ..
nano docker-compose.yml
```

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppassword
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - app_net

  backend:
    build: ./backend
    environment:
      DB_HOST: db
      DB_NAME: appdb
      DB_USER: appuser
      DB_PASSWORD: apppassword
    depends_on:
      - db
    networks:
      - app_net

  frontend:
    build: ./frontend
    networks:
      - app_net

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - app_net

volumes:
  db_data:

networks:
  app_net:
```

## Run และทดสอบ

```bash
docker compose up -d --build
docker compose ps
curl http://localhost:8080/api/health
curl http://localhost:8080/api/db
```

เปิด browser:

```text
http://app-server:8080
```

## ตรวจสอบ

```bash
docker compose logs -f backend
docker compose logs -f nginx
```

## ข้อสรุป

Docker Compose เหมาะกับ lab, local development และการ deploy ขนาดเล็ก ช่วยให้เห็น dependency ระหว่าง service ชัดเจน

---

# 10. Lab 07: Private Container Registry

## วัตถุประสงค์

- สร้าง registry ส่วนตัว
- push/pull image ระหว่าง VM
- เตรียม registry สำหรับ CI/CD และ Kubernetes

## Run Registry บน devops-control

```bash
docker run -d --name registry --restart always -p 5000:5000 -v registry_data:/var/lib/registry registry:2
curl http://localhost:5000/v2/_catalog
```

## ตั้ง insecure registry บน client

บน app-server:

```bash
sudo nano /etc/docker/daemon.json
```

```json
{
  "insecure-registries": ["devops-control:5000"]
}
```

Restart:

```bash
sudo systemctl restart docker
```

## Push image

```bash
docker tag simple-api:1.0.0 devops-control:5000/simple-api:1.0.0
docker push devops-control:5000/simple-api:1.0.0
curl http://devops-control:5000/v2/_catalog
```

## Pull image

```bash
docker pull devops-control:5000/simple-api:1.0.0
```

## ข้อสรุป

Registry เป็นจุดกลางของ workflow: CI/CD build image แล้ว push ไป registry จากนั้น server หรือ Kubernetes pull image ไป run

---

# 11. Lab 08: Git Flow และ Release Version

## วัตถุประสงค์

- จัด branch strategy
- ใช้ tag version
- เข้าใจ release และ rollback

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Branch | สายการพัฒนา code |
| Merge Request | คำขอรวม code |
| Tag | ป้าย version ที่ชี้ commit |
| Semantic Versioning | version แบบ MAJOR.MINOR.PATCH เช่น 1.0.0 |
| Changelog | สรุปสิ่งที่เปลี่ยนใน release |

## Branch Flow ตัวอย่าง

```text
feature/* -> develop -> uat -> main
```

## ทดลอง Git Flow

```bash
mkdir ~/labs/git-flow-demo
cd ~/labs/git-flow-demo
git init
echo '# Git Flow Demo' > README.md
git add .
git commit -m 'initial commit'
git branch -M main
git checkout -b develop
git checkout -b feature/hello
echo 'hello feature' >> README.md
git add .
git commit -m 'add hello feature'
git checkout develop
git merge feature/hello
git checkout main
git merge develop
git tag v1.0.0
git log --oneline --decorate --graph --all
```

## การตรวจสอบ

- เห็น branch และ tag
- รู้ว่า release v1.0.0 ชี้ commit ไหน
- rollback โดย checkout tag เก่าได้

## ข้อสรุป

DevOps ต้องตอบได้ว่า deploy commit ไหน, version ไหน, ใคร approve และถ้าพังจะ rollback กลับ version ไหน

---

# 12. Lab 09: CI/CD ด้วย GitLab CE และ GitLab Runner

## วัตถุประสงค์

- ติดตั้ง GitLab CE แบบ container
- ตั้ง GitLab Runner
- เขียน `.gitlab-ci.yml`
- build และ push image เข้า registry

## ติดตั้ง GitLab CE บน devops-control

```bash
mkdir -p ~/gitlab/{config,logs,data}
docker run -d \
  --hostname devops-control \
  --name gitlab \
  --restart always \
  -p 8081:80 \
  -p 2222:22 \
  -v ~/gitlab/config:/etc/gitlab \
  -v ~/gitlab/logs:/var/log/gitlab \
  -v ~/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

ดู password root:

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

เข้าใช้งาน:

```text
http://devops-control:8081
```

## ติดตั้ง GitLab Runner

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

Register runner จาก GitLab UI แล้วเลือก executor เป็น docker

## ตัวอย่าง .gitlab-ci.yml

```yaml
stages:
  - test
  - build
  - deploy

variables:
  IMAGE_NAME: "devops-control:5000/simple-api"
  IMAGE_TAG: "$CI_COMMIT_SHORT_SHA"

test:
  stage: test
  image: node:20-alpine
  script:
    - npm ci
    - node -e "console.log('test passed')"

build:
  stage: build
  image: docker:latest
  script:
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker push $IMAGE_NAME:$IMAGE_TAG

deploy_dev:
  stage: deploy
  script:
    - echo "Deploy to dev server here"
  only:
    - main
```

## การทดสอบ

- Push code เข้า GitLab
- Pipeline ต้องรันครบ stage
- Registry มี image tag ใหม่

## การตรวจสอบ

```bash
curl http://devops-control:5000/v2/_catalog
```

## ข้อสรุป

CI/CD ลดงาน manual และทำให้ทุก release มีประวัติ ตรวจสอบได้ และทำซ้ำได้

---

# 13. Lab 10: Terraform และ Infrastructure as Code

## วัตถุประสงค์

- เข้าใจ Terraform workflow
- ใช้ provider, resource, state
- เตรียมพื้นฐานไปใช้กับ Cloud จริง

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Provider | plugin ที่ Terraform ใช้คุยกับ platform |
| Resource | สิ่งที่ Terraform สร้างหรือจัดการ |
| State | ไฟล์บันทึกสถานะ infra |
| Plan | preview สิ่งที่จะเปลี่ยน |
| Apply | สั่งสร้าง/แก้ไขจริง |
| Destroy | ลบ resource |

## ติดตั้ง Terraform

```bash
sudo apt update
sudo apt install -y wget unzip
wget https://releases.hashicorp.com/terraform/1.9.8/terraform_1.9.8_linux_amd64.zip
unzip terraform_1.9.8_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform version
```

## Terraform local_file Lab

```bash
mkdir -p ~/labs/terraform-local
cd ~/labs/terraform-local
nano main.tf
```

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

provider "local" {}

resource "local_file" "devops_note" {
  filename = "${path.module}/devops-note.txt"
  content  = "Created by Terraform\n"
}
```

Run:

```bash
terraform init
terraform plan
terraform apply
cat devops-note.txt
terraform state list
terraform destroy
```

## ข้อสรุป

Terraform ทำให้ infra ถูกจัดการเป็น code, review ได้, version control ได้ และทำซ้ำได้

---

# 14. Lab 11: Monitoring ด้วย Prometheus และ Grafana

## วัตถุประสงค์

- เก็บ metrics จาก server
- แสดง dashboard
- เข้าใจ scrape target และ exporter

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Metrics | ตัวเลขสถานะระบบ |
| Exporter | agent ที่ expose metrics |
| Scrape | การที่ Prometheus ไปดึง metrics |
| Dashboard | หน้าจอแสดงข้อมูล |

## ติดตั้ง Node Exporter บน app-server

```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/
```

สร้าง service:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
curl http://localhost:9100/metrics | head
```

## Run Prometheus + Grafana บน monitor-server

```bash
mkdir -p ~/monitoring/prometheus
cd ~/monitoring
nano prometheus/prometheus.yml
```

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets:
          - 'app-server:9100'
```

สร้าง compose:

```bash
nano docker-compose.yml
```

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  prometheus_data:
  grafana_data:
```

Run:

```bash
docker compose up -d
```

เข้าใช้งาน:

```text
Prometheus: http://monitor-server:9090
Grafana:    http://monitor-server:3000
Login: admin / admin
```

Prometheus query:

```text
up
node_memory_MemAvailable_bytes
node_cpu_seconds_total
node_filesystem_avail_bytes
```

## ข้อสรุป

Monitoring ทำให้รู้สถานะระบบแบบตัวเลข เช่น CPU, RAM, Disk, Service up/down ก่อนเกิดผลกระทบรุนแรง

---

# 15. Lab 12: Logging ด้วย Loki และ Promtail

## วัตถุประสงค์

- รวม log จาก server เข้าที่เดียว
- Query log ผ่าน Grafana
- เข้าใจ centralized logging

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Log Aggregation | การรวม log หลายเครื่องไว้ที่เดียว |
| Loki | ระบบเก็บ log |
| Promtail | agent ส่ง log ไป Loki |
| Label | metadata สำหรับค้นหา log |

## เพิ่ม Loki ใน docker-compose

บน monitor-server แก้ `docker-compose.yml` เพิ่ม service:

```yaml
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
```

Run:

```bash
docker compose up -d
```

## Run Promtail บน app-server

```bash
mkdir -p ~/promtail
nano ~/promtail/config.yml
```

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://monitor-server:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          host: app-server
          __path__: /var/log/*.log
```

Run:

```bash
docker run -d --name promtail --restart always \
  -v ~/promtail/config.yml:/etc/promtail/config.yml \
  -v /var/log:/var/log:ro \
  grafana/promtail:latest \
  -config.file=/etc/promtail/config.yml
```

## Grafana

เพิ่ม Data Source เป็น Loki:

```text
URL: http://loki:3100
```

Query:

```text
{host="app-server"}
{job="varlogs"}
```

## ข้อสรุป

Centralized logging ทำให้ไม่ต้อง SSH เข้าเครื่องทีละตัว และช่วย debug เหตุการณ์ย้อนหลังได้ง่าย

---

# 16. Lab 13: Kubernetes Cluster ด้วย kubeadm

## วัตถุประสงค์

- สร้าง Kubernetes Cluster เองบน VMware
- เข้าใจ control plane, worker node, CNI, container runtime

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Control Plane | ส่วนควบคุม cluster |
| Worker Node | เครื่องที่รัน workload |
| Pod | หน่วยเล็กสุดของ Kubernetes |
| kubelet | agent บน node |
| containerd | container runtime |
| CNI | network plugin ของ Kubernetes |

## เตรียมทุก node

ปิด swap:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

โหลด module:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

ตั้ง sysctl:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

ติดตั้ง containerd:

```bash
sudo apt update
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

ติดตั้ง kubeadm/kubelet/kubectl:

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Init บน k8s-master-01

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.56.100 \
  --pod-network-cidr=192.168.0.0/16
```

ตั้งค่า kubectl:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

ติดตั้ง Calico:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

## Join worker

ใช้ command ที่ได้จาก `kubeadm init` ไปรันบน worker เช่น:

```bash
sudo kubeadm join 192.168.56.100:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

ถ้าลืม:

```bash
kubeadm token create --print-join-command
```

## การตรวจสอบ

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

## ข้อสรุป

Kubernetes ต้องอาศัย Linux, Network, Container Runtime และ CNI ถ้าส่วนใดผิด node หรือ pod จะไม่พร้อมทำงาน

---

# 17. Lab 14: Deploy App เข้า Kubernetes

## วัตถุประสงค์

- สร้าง Namespace, Deployment, Service
- Scale app
- Rollout และ rollback

## สร้าง Namespace

```bash
kubectl create namespace devops-lab
```

## Deployment

```bash
nano simple-api-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-api
  namespace: devops-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple-api
  template:
    metadata:
      labels:
        app: simple-api
    spec:
      containers:
        - name: simple-api
          image: devops-control:5000/simple-api:1.0.0
          ports:
            - containerPort: 3000
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

Apply:

```bash
kubectl apply -f simple-api-deployment.yaml
```

## Service

```bash
nano simple-api-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: simple-api
  namespace: devops-lab
spec:
  selector:
    app: simple-api
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

Apply:

```bash
kubectl apply -f simple-api-service.yaml
```

## ทดสอบใน cluster

```bash
kubectl run test-curl -n devops-lab --rm -it --image=curlimages/curl -- sh
curl http://simple-api/health
```

## Scale

```bash
kubectl scale deployment simple-api -n devops-lab --replicas=4
kubectl get pods -n devops-lab -o wide
```

## Rollout/Rollback

```bash
kubectl set image deployment/simple-api simple-api=devops-control:5000/simple-api:1.0.1 -n devops-lab
kubectl rollout status deployment/simple-api -n devops-lab
kubectl rollout undo deployment/simple-api -n devops-lab
```

## ตรวจสอบ

```bash
kubectl get all -n devops-lab
kubectl describe pod -n devops-lab <pod-name>
kubectl logs -n devops-lab <pod-name>
```

## ข้อสรุป

Deployment คุมจำนวน pod และ rollout ส่วน Service ทำให้ app ถูกเรียกด้วยชื่อคงที่ แม้ pod จะเปลี่ยน IP

---

# 18. Lab 15: Ingress, ConfigMap, Secret และ Storage

## วัตถุประสงค์

- เปิด app จากนอก cluster ด้วย Ingress
- แยก config ด้วย ConfigMap
- เก็บข้อมูลลับด้วย Secret
- ใช้ PersistentVolume/PersistentVolumeClaim พื้นฐาน

## ติดตั้ง Nginx Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/baremetal/deploy.yaml
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

## ConfigMap

```bash
nano simple-api-configmap.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: simple-api-config
  namespace: devops-lab
data:
  APP_ENV: "lab"
  LOG_LEVEL: "debug"
```

```bash
kubectl apply -f simple-api-configmap.yaml
```

## Secret

```bash
kubectl create secret generic simple-api-secret \
  -n devops-lab \
  --from-literal=DB_PASSWORD=apppassword
```

ตรวจสอบ:

```bash
kubectl get secret -n devops-lab
kubectl describe secret simple-api-secret -n devops-lab
```

## Ingress

```bash
nano simple-api-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-api
  namespace: devops-lab
spec:
  ingressClassName: nginx
  rules:
    - host: simple-api.lab.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: simple-api
                port:
                  number: 80
```

```bash
kubectl apply -f simple-api-ingress.yaml
```

เพิ่ม hosts บนเครื่องที่ใช้ทดสอบ:

```text
192.168.56.101 simple-api.lab.local
```

ทดสอบ:

```bash
curl http://simple-api.lab.local
```

## Storage แบบ hostPath สำหรับ Lab

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lab-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/lab-data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lab-pvc
  namespace: devops-lab
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

## ข้อสรุป

ConfigMap ใช้กับค่าที่ไม่ลับ, Secret ใช้กับข้อมูลลับ, Ingress รับ traffic จากภายนอก และ Storage ทำให้ข้อมูลไม่หายเมื่อ pod ถูกลบ

---

# 19. Lab 16: Helm Chart

## วัตถุประสงค์

- ลดความซ้ำของ Kubernetes YAML
- แยก values ตาม environment
- upgrade และ rollback ง่ายขึ้น

## ติดตั้ง Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

## สร้าง Chart

```bash
helm create simple-api-chart
cd simple-api-chart
```

โครงสร้าง:

```text
simple-api-chart/
├── Chart.yaml
├── values.yaml
└── templates/
```

ตัวอย่าง `values.yaml`:

```yaml
replicaCount: 2

image:
  repository: devops-control:5000/simple-api
  tag: "1.0.0"

service:
  type: ClusterIP
  port: 80

containerPort: 3000

ingress:
  enabled: true
  className: nginx
  host: simple-api.lab.local
```

## ติดตั้ง / Upgrade / Rollback

```bash
helm install simple-api . -n devops-lab
helm list -n devops-lab
helm upgrade simple-api . -n devops-lab --set image.tag=1.0.1
helm history simple-api -n devops-lab
helm rollback simple-api 1 -n devops-lab
```

## แยก environment

```text
values-dev.yaml
values-uat.yaml
values-prod.yaml
```

Deploy:

```bash
helm upgrade --install simple-api . -n devops-lab -f values-dev.yaml
```

## ข้อสรุป

Helm ทำให้ deployment บน Kubernetes เป็น package ที่ reusable และเหมาะกับ CI/CD มากกว่า apply YAML ทีละหลายไฟล์

---

# 20. Lab 17: DevSecOps ด้วย Trivy

## วัตถุประสงค์

- Scan image vulnerability
- Scan filesystem/repository
- เพิ่ม security gate ใน pipeline

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Vulnerability | ช่องโหว่ |
| CVE | รหัสช่องโหว่สาธารณะ |
| Severity | ระดับความรุนแรง เช่น LOW, HIGH, CRITICAL |
| Least Privilege | ให้สิทธิ์เท่าที่จำเป็น |

## ติดตั้ง Trivy

```bash
sudo apt update
sudo apt install -y wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt update
sudo apt install -y trivy
```

## Scan image

```bash
trivy image simple-api:1.0.0
trivy image --severity HIGH,CRITICAL simple-api:1.0.0
trivy image --exit-code 1 --severity CRITICAL simple-api:1.0.0
```

## Scan source code/filesystem

```bash
trivy fs .
```

## ตัวอย่าง GitLab CI

```yaml
security_scan:
  stage: test
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy fs --exit-code 1 --severity HIGH,CRITICAL .
```

## Basic Hardening Checklist

- ไม่เก็บ password/token ใน code
- ไม่ commit `.env`
- ไม่ run container เป็น root ถ้าไม่จำเป็น
- ตั้ง resource limit ให้ container
- เปิด firewall เฉพาะ port ที่จำเป็น
- ให้สิทธิ์แบบ least privilege
- scan dependency และ image ก่อน deploy

## ข้อสรุป

DevSecOps คือการทำ security ตั้งแต่ต้นทางใน pipeline ไม่ใช่รอให้ขึ้น production แล้วค่อยตรวจ

---

# 21. Lab 18: Backup, Restore และ DR พื้นฐาน

## วัตถุประสงค์

- Backup database
- Restore database
- เข้าใจ RPO/RTO

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Backup | สำรองข้อมูล |
| Restore | กู้คืนข้อมูล |
| RPO | ยอมเสียข้อมูลย้อนหลังได้เท่าไร |
| RTO | ต้องกู้ระบบกลับมาในเวลานานเท่าไร |
| DR | Disaster Recovery แผนรับมือเหตุร้ายแรง |

## Backup PostgreSQL ใน Docker Compose

```bash
cd ~/labs/fullstack-lab
docker compose exec db pg_dump -U appuser appdb > backup-appdb.sql
ls -lh backup-appdb.sql
head backup-appdb.sql
```

## Restore

```bash
docker compose exec db psql -U appuser -d appdb -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"
cat backup-appdb.sql | docker compose exec -T db psql -U appuser -d appdb
```

## Backup Kubernetes YAML

```bash
kubectl get all -n devops-lab -o yaml > devops-lab-resources.yaml
kubectl get configmap -n devops-lab -o yaml > devops-lab-configmaps.yaml
kubectl get ingress -n devops-lab -o yaml > devops-lab-ingress.yaml
```

## ข้อสรุป

ระบบที่ deploy ได้แต่ restore ไม่ได้ ยังไม่ถือว่า production-ready ต้องทดสอบ backup/restore จริงเสมอ

---

# 22. Final Project

## เป้าหมาย

เอา Fullstack App ของคุณเองมาทำเป็น DevOps Project แบบครบวงจร

## โครงสร้างที่แนะนำ

```text
my-devops-project/
├── frontend/
├── backend/
├── docker-compose.yml
├── nginx/
├── k8s/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── secret.yaml
├── helm/
├── terraform/
├── monitoring/
├── .gitlab-ci.yml
└── README.md
```

## Pipeline ที่ต้องการ

```text
Developer push code
-> test
-> build Docker image
-> scan image
-> push registry
-> deploy dev
-> manual approve
-> deploy prod/lab
-> monitor
-> rollback ได้
```

## Requirement

- App run ได้บน local
- มี Dockerfile frontend/backend
- มี Docker Compose
- มี Private Registry
- มี GitLab repo
- มี CI/CD pipeline
- มี Trivy scan
- Deploy เข้า Kubernetes ได้
- มี Ingress
- มี ConfigMap/Secret
- มี Helm Chart
- มี Grafana Dashboard
- มี log ใน Loki
- มี backup/restore test
- README อธิบายวิธีติดตั้งและทดสอบครบ

---

# 23. Checklist หลังเรียนจบ

## Linux / Network

- [ ] SSH เข้า server ได้
- [ ] ตั้ง static IP ได้
- [ ] ตรวจ DNS ได้
- [ ] ตรวจ port ด้วย `ss` ได้
- [ ] ใช้ UFW ได้
- [ ] ดู service ด้วย systemctl ได้
- [ ] ดู log ด้วย journalctl ได้

## Docker

- [ ] เขียน Dockerfile ได้
- [ ] build image ได้
- [ ] run container ได้
- [ ] ใช้ volume ได้
- [ ] ใช้ Docker Compose ได้
- [ ] push image เข้า registry ได้

## CI/CD

- [ ] เข้าใจ stage/job
- [ ] run test ใน pipeline ได้
- [ ] build image ได้
- [ ] push registry ได้
- [ ] deploy อัตโนมัติได้
- [ ] rollback ได้

## Kubernetes

- [ ] สร้าง cluster ได้
- [ ] deploy Deployment ได้
- [ ] expose Service ได้
- [ ] ใช้ Ingress ได้
- [ ] ใช้ ConfigMap/Secret ได้
- [ ] scale app ได้
- [ ] rollout/rollback ได้
- [ ] ใช้ Helm ได้

## Monitoring / Logging

- [ ] Prometheus scrape target ได้
- [ ] Grafana dashboard ได้
- [ ] Loki query log ได้
- [ ] เข้าใจ alert concept

## Security

- [ ] scan image ด้วย Trivy ได้
- [ ] ไม่ commit secret
- [ ] ใช้ least privilege
- [ ] ตั้ง resource limit
- [ ] เปิด firewall เฉพาะที่จำเป็น

---

# 24. Troubleshooting รวม

## SSH เข้าไม่ได้

ตรวจสอบ:

```bash
sudo systemctl status ssh
ip a
sudo ufw status
ping <target-ip>
```

สาเหตุที่พบบ่อย:

- IP ผิด
- SSH service ไม่ทำงาน
- firewall block port 22
- VMware network mode ไม่ตรงกัน

## Docker permission denied

```bash
sudo usermod -aG docker $USER
newgrp docker
```

หรือ logout/login ใหม่

## Container ออกทันที

```bash
docker ps -a
docker logs <container-name>
```

สาเหตุ:

- command ผิด
- app crash
- env variable หาย
- port conflict

## Nginx config error

```bash
sudo nginx -t
sudo tail -f /var/log/nginx/error.log
```

## Kubernetes node NotReady

```bash
kubectl get nodes
kubectl describe node <node-name>
sudo systemctl status kubelet
sudo journalctl -u kubelet -f
```

สาเหตุ:

- CNI ยังไม่พร้อม
- swap ยังเปิดอยู่
- containerd config ผิด
- firewall block port

## Pod CrashLoopBackOff

```bash
kubectl logs -n <namespace> <pod-name>
kubectl describe pod -n <namespace> <pod-name>
```

สาเหตุ:

- app start ไม่สำเร็จ
- config/env ผิด
- secret ไม่มี
- database connect ไม่ได้

## ImagePullBackOff

```bash
kubectl describe pod -n <namespace> <pod-name>
```

สาเหตุ:

- image name/tag ผิด
- registry เข้าไม่ได้
- private registry ไม่มี credential
- insecure registry ยังไม่ได้ config

## Prometheus target Down

```bash
curl http://target-host:9100/metrics
```

สาเหตุ:

- node_exporter ไม่ทำงาน
- firewall block port 9100
- hostname resolve ไม่ได้
- prometheus.yml target ผิด

---

# ภาคผนวก: คำสั่งที่ควรจำ

## Linux

```bash
ip a
ip route
ss -tulpen
curl -I http://example.com
systemctl status <service>
journalctl -u <service> -f
df -h
free -h
top
```

## Docker

```bash
docker build -t name:tag .
docker run -d --name app -p 8080:80 name:tag
docker ps
docker logs -f app
docker exec -it app sh
docker compose up -d --build
docker compose logs -f
```

## Kubernetes

```bash
kubectl get nodes
kubectl get pods -A
kubectl get all -n <namespace>
kubectl describe pod <pod>
kubectl logs <pod>
kubectl apply -f file.yaml
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>
```

## Helm

```bash
helm install <release> <chart> -n <namespace>
helm upgrade <release> <chart> -n <namespace>
helm rollback <release> <revision> -n <namespace>
helm history <release> -n <namespace>
```

---

# แนวทางเรียนต่อ

หลังทำ Lab นี้จบ แนะนำเรียนต่อ:

- Ansible สำหรับ configuration management
- Harbor Registry แทน registry:2
- Argo CD สำหรับ GitOps
- cert-manager สำหรับ TLS บน Kubernetes
- External Secrets หรือ Vault สำหรับ secret management
- MetalLB สำหรับ LoadBalancer บน bare metal lab
- Alertmanager สำหรับ alert จริง
- OpenTelemetry สำหรับ tracing
- Velero สำหรับ backup Kubernetes
- Service Mesh เช่น Istio หรือ Linkerd

---

# สรุปสุดท้าย

DevOps คือการเข้าใจ flow ทั้งระบบ ไม่ใช่แค่จำ command หรือเครื่องมือ:

```text
Code
-> Build
-> Test
-> Package as Image
-> Push Registry
-> Deploy
-> Monitor
-> Log
-> Secure
-> Backup
-> Restore
```

ถ้าทำ Lab นี้ครบ คุณจะมีพื้นฐานพร้อมสำหรับสาย DevOps Engineer, Cloud Engineer, Platform Engineer หรือ SRE โดยเฉพาะเมื่อมีพื้นฐาน Fullstack Developer อยู่แล้ว

