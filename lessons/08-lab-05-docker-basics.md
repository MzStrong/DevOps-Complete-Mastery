# 8. Lab 05: Docker พื้นฐาน

Lab นี้นำ `simple-api` จากบท manual deploy มาทำเป็น Docker image เพื่อให้ app พร้อม runtime และ dependency ถูก package ไว้ด้วยกัน จุดสำคัญคือแยกให้ชัดว่า `image` คือสิ่งที่ build ได้ ส่วน `container` คือสิ่งที่รันจาก image

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

ภาพรวม flow ของบทนี้:

```text
source code + Dockerfile
-> docker build
-> image: simple-api:1.0.0
-> docker run
-> container: simple-api
-> curl เข้า app ที่ port 3000
```

ถ้าแก้ source code แต่ไม่ build image ใหม่ container จะยังใช้ code เก่า ถ้า build image ใหม่แต่ไม่ recreate container container เดิมก็ยังรัน image เดิมอยู่เช่นกัน

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

คำสั่งติดตั้งชุดนี้ใช้ repository ทางการของ Docker แทน package docker ที่อาจมากับ Ubuntu repository เพราะมักได้ version และ plugin ที่ครบกว่า เช่น Buildx และ Docker Compose plugin

อธิบายส่วนสำคัญ:

- `ca-certificates`, `curl`, `gnupg` ใช้สำหรับเพิ่ม repository ที่ signed ด้วย GPG key
- `/etc/apt/keyrings/docker.gpg` เก็บ key ของ Docker repository
- `/etc/apt/sources.list.d/docker.list` เพิ่มแหล่ง package ของ Docker
- `docker-ce`, `docker-ce-cli`, `containerd.io` เป็น component หลัก
- `docker-compose-plugin` ทำให้ใช้คำสั่ง `docker compose`
- `usermod -aG docker devops` ให้ user `devops` ใช้ docker ได้โดยไม่ต้อง sudo

หลังเพิ่ม user เข้า group docker ต้อง logout/login ใหม่ หรือใช้ `newgrp docker` เพื่อให้ shell ปัจจุบันรับ group ใหม่ ถ้าไม่ทำจะเจอ permission denied ตอนรัน `docker ps`

ตัวอย่างที่ผิด:

```bash
sudo usermod -aG docker devops
docker ps
```

แล้วเจอ permission denied เพราะ session ปัจจุบันยังไม่รับ group ใหม่

ตัวอย่างที่ถูก:

```bash
sudo usermod -aG docker devops
newgrp docker
docker ps
```

ข้อควรระวัง: user ที่อยู่ใน group `docker` มีสิทธิ์สูงมากใกล้เคียง root ในเครื่องนั้น จึงควรให้เฉพาะ user ที่จำเป็น ใน lab นี้ใช้ `devops` เพื่อความสะดวก

ตรวจสอบ:

```bash
docker version
docker run hello-world
```

`docker version` ตรวจว่า client และ server ติดต่อกันได้ ถ้าเห็นเฉพาะ client แต่ server error แปลว่า Docker daemon อาจยังไม่รัน

ตรวจ daemon:

```bash
sudo systemctl status docker
```

`docker run hello-world` เป็น smoke test ว่า Docker สามารถ pull image และรัน container ได้ ถ้า pull ไม่ได้ให้ตรวจ internet/DNS ถ้ารันไม่ได้ให้ดู Docker daemon และ permission

## Dockerfile สำหรับ simple-api

```bash
cd ~/apps/simple-api
nano Dockerfile
```

Dockerfile คือสูตรสำหรับ build image ให้ app เดิมจากบทก่อนรันใน container ได้ โดยไม่ต้องติดตั้ง Node.js และ dependency ลงบน host ทุกครั้ง

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

อธิบายทีละบรรทัด:

- `FROM node:20-alpine` ใช้ base image ที่มี Node.js 20 บน Alpine Linux ซึ่งเล็กกว่า image แบบ full OS
- `WORKDIR /app` ตั้ง directory ทำงานภายใน image
- `COPY package*.json ./` copy package metadata ก่อน เพื่อให้ Docker cache step install dependency ได้ดี
- `RUN npm ci --only=production` ติดตั้ง dependency ตาม lock file สำหรับ production
- `COPY index.js ./` copy source code เข้า image
- `ENV PORT=3000` ตั้งค่า default environment variable
- `EXPOSE 3000` บอกว่า container ตั้งใจเปิด port 3000
- `CMD ["node", "index.js"]` คำสั่ง default เมื่อ container start

`EXPOSE` ไม่ได้เปิด port ออกมาที่ host เอง ต้องใช้ `-p` ตอน `docker run` เพื่อ map port

ถ้า project ยังไม่มี `package-lock.json` คำสั่ง `npm ci` อาจ fail ให้รัน:

```bash
npm install
```

เพื่อสร้าง `package-lock.json` ก่อน แล้วค่อย build ใหม่

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

`.dockerignore` ช่วยตัดไฟล์ที่ไม่ควรถูกส่งเข้า build context เช่น `node_modules`, `.git` และ `.env`

เหตุผล:

- `node_modules` จาก host อาจไม่ตรงกับ OS ใน container
- `.git` ทำให้ build context ใหญ่และไม่จำเป็น
- `.env` อาจมี secret ไม่ควรถูกฝังเข้า image

ตัวอย่างที่ผิด:

```text
ไม่ใช้ .dockerignore แล้ว COPY ทั้ง project รวม .env เข้า image
```

ผลคือ image อาจมี secret ติดไปด้วย และ build ช้าจากไฟล์ที่ไม่จำเป็น

Build:

```bash
docker build -t simple-api:1.0.0 .
```

คำสั่งนี้ build image จาก Dockerfile ใน directory ปัจจุบัน:

- `-t simple-api:1.0.0` ตั้งชื่อ image และ tag
- `.` คือ build context หรือ directory ที่ส่งให้ Docker build

การตั้ง tag ชัดเจนสำคัญมาก เพราะช่วยรู้ว่า container รัน version ใด อย่าใช้ `latest` อย่างเดียวใน lab ที่ต้องฝึก rollback หรือ deploy หลาย environment

ตรวจ image:

```bash
docker images
```

Run:

```bash
docker run -d --name simple-api -p 3000:3000 simple-api:1.0.0
```

อธิบาย option:

- `-d` รันแบบ detached กลับมาที่ shell ทันที
- `--name simple-api` ตั้งชื่อ container
- `-p 3000:3000` map port host:container
- `simple-api:1.0.0` image ที่ใช้รัน

port mapping ต้องอ่านจากซ้ายไปขวา:

```text
-p 3000:3000
   ^    ^
   |    container port
   host port
```

ถ้า host port 3000 ถูกใช้อยู่แล้ว จะ run ไม่ได้ ให้ตรวจด้วย:

```bash
ss -tulpen | grep ':3000'
docker ps
```

ตัวอย่างที่ผิด:

```bash
docker run -d --name simple-api simple-api:1.0.0
curl http://localhost:3000
```

ลืม `-p` ทำให้ host เข้า container port 3000 ไม่ได้

ตัวอย่างที่ถูก:

```bash
docker run -d --name simple-api -p 3000:3000 simple-api:1.0.0
```

## การทดสอบ

```bash
curl http://localhost:3000
curl http://localhost:3000/health
```

ถ้า curl ผ่าน แปลว่า:

```text
container running
-> app process ทำงาน
-> port 3000 ใน container เปิด
-> port 3000 ถูก map ออก host
```

ถ้าไม่ผ่าน ให้ไล่ตรวจ:

```bash
docker ps
docker ps -a
docker logs simple-api
ss -tulpen | grep ':3000'
```

ความหมาย:

- `docker ps` เห็น container ที่กำลังรัน
- `docker ps -a` เห็น container ที่หยุดไปแล้ว
- `docker logs` บอกว่า app crash หรือ start ได้
- `ss` บอกว่า host เปิด port 3000 หรือไม่

## การตรวจสอบ

```bash
docker ps
docker logs simple-api
docker exec -it simple-api sh
```

คำสั่งตรวจสอบ:

- `docker ps` ดู container ที่รันอยู่, port mapping และชื่อ container
- `docker logs simple-api` ดู stdout/stderr ของ app
- `docker exec -it simple-api sh` เข้า shell ภายใน container เพื่อดูไฟล์หรือทดสอบคำสั่ง

อย่าแก้ไฟล์ใน container แล้วคิดว่าเป็นการแก้ถาวร เพราะ container ถูกลบแล้วสร้างใหม่ไฟล์ที่แก้ด้วยมือจะหาย การแก้ถาวรต้องแก้ source code หรือ Dockerfile แล้ว build image ใหม่

ตัวอย่าง workflow ที่ถูกเมื่อแก้ code:

```bash
docker stop simple-api
docker rm simple-api
docker build -t simple-api:1.0.1 .
docker run -d --name simple-api -p 3000:3000 simple-api:1.0.1
```

ลบ container:

```bash
docker stop simple-api
docker rm simple-api
```

`docker stop` หยุด container แบบ graceful ส่วน `docker rm` ลบ container ที่หยุดแล้ว ถ้าไม่ลบ container เดิม จะใช้ชื่อ `simple-api` ซ้ำไม่ได้

ถ้าต้องลบ image:

```bash
docker rmi simple-api:1.0.0
```

แต่ให้ทำเฉพาะเมื่อไม่มี container ใช้ image นั้นอยู่ หรือไม่ต้องการเก็บ version นั้นแล้ว

## ข้อสรุป

Docker ช่วยให้ app มี runtime ที่เหมือนกันในทุก environment ลดปัญหา dependency และ config ไม่ตรงกัน

สิ่งที่ควรจำจากบทนี้:

```text
Dockerfile -> build image
image -> run container
container -> process ที่มี filesystem/network แยก
-p host:container -> เปิดทางให้ host เรียก app
logs -> จุดเริ่มต้นของการ debug container
```

เมื่อเข้าใจ Docker พื้นฐานแล้ว บทถัดไปจะใช้ Docker Compose เพื่อรันหลาย service พร้อมกัน เช่น frontend, backend, database และ Nginx

---
