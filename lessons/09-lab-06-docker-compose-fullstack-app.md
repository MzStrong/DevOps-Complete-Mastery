# 9. Lab 06: Docker Compose สำหรับ Fullstack App

Lab นี้ขยับจากการรัน container เดียวไปเป็นระบบที่มีหลาย service ทำงานร่วมกัน ได้แก่ frontend, backend, database และ Nginx reverse proxy Docker Compose ช่วยให้กำหนด service, network, volume และ environment ไว้ในไฟล์เดียว ทำให้เริ่ม/หยุดทั้งระบบได้ด้วยคำสั่งเดียว

## วัตถุประสงค์

- รันหลาย service พร้อมกัน
- มี frontend, backend, database, nginx
- ใช้ network และ volume

ภาพรวม request flow:

```text
browser/curl
-> app-server:8080
-> nginx container
-> /      proxy ไป frontend:80
-> /api/  proxy ไป backend:3000
-> backend ต่อ db:5432
-> postgres container
```

ชื่อ `frontend`, `backend`, `db` ไม่ใช่ DNS จริงภายนอก แต่เป็น service name ใน Docker Compose network ที่ container ใช้เรียกกันได้

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

แยก folder ตามหน้าที่เพื่อให้เห็นชัดว่าแต่ละ service มี source/config ของตัวเอง:

- `backend/` เก็บ Node.js API และ Dockerfile ของ backend
- `frontend/` เก็บ static HTML และ Dockerfile ของ frontend
- `nginx/` เก็บ reverse proxy config
- `docker-compose.yml` เป็นไฟล์ประกอบทุก service เข้าด้วยกัน

ถ้าโครงสร้างไม่ชัด ตอน debug จะสับสนว่า config ไหนเป็นของ container ไหน โดยเฉพาะเมื่อมี Nginx ทั้ง frontend image และ reverse proxy image

## Backend

```bash
cd backend
npm init -y
npm install express pg
nano index.js
```

backend ในบทนี้มีหน้าที่รับ request `/api/*` และเชื่อมต่อ PostgreSQL ผ่าน environment variable แทนการ hardcode ค่าใน source code วิธีนี้ทำให้ image เดียวกันใช้กับ environment ต่างกันได้ เพียงเปลี่ยน env ตอน run

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

ค่าที่ backend ใช้ต่อ database:

```text
DB_HOST=db
DB_NAME=appdb
DB_USER=appuser
DB_PASSWORD=apppassword
```

`DB_HOST=db` สำคัญมาก เพราะใน Docker Compose container จะ resolve service name `db` เป็น IP ของ database container ได้เอง ห้ามใช้ `localhost` เพื่อเชื่อม database จาก backend container เพราะ `localhost` ใน container หมายถึงตัว backend container เอง ไม่ใช่ db container

ตัวอย่างที่ผิด:

```javascript
host: 'localhost'
```

ผลคือ backend จะพยายามต่อ PostgreSQL ใน container ตัวเอง แล้ว connection fail

ตัวอย่างที่ถูก:

```javascript
host: process.env.DB_HOST
```

แล้วกำหนด `DB_HOST: db` ใน Compose

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

Dockerfile ของ backend เหมือนแนวคิดในบท Docker พื้นฐาน คือ build image ที่มี runtime และ dependency ของ API พร้อมรัน แต่ตอนนี้ backend ไม่ได้เปิด port ออก host โดยตรง เพราะ Nginx จะเป็นทางเข้าหลักของระบบ

## Frontend

```bash
cd ../frontend
nano index.html
```

frontend ในบทนี้เป็น static HTML ที่เรียก API ผ่าน path `/api/health` จุดสำคัญคือ frontend ไม่ต้องรู้ว่า backend container อยู่ที่ IP ไหน เพราะ browser เรียกผ่าน Nginx reverse proxy ที่ port 8080 แล้ว Nginx เป็นคน route `/api/` ไป backend

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

ข้อดีของการเรียก API แบบ relative path:

```javascript
fetch('/api/health')
```

คือ browser จะเรียก API ผ่าน host เดียวกับหน้าเว็บ เช่น `http://app-server:8080/api/health` ลดปัญหา CORS ใน lab นี้

ถ้าเขียนเป็น `http://backend:3000/api/health` ใน frontend จะผิด เพราะ browser ของผู้ใช้ไม่ได้อยู่ใน Docker network และไม่รู้จัก hostname `backend`

```bash
nano Dockerfile
```

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

frontend ใช้ `nginx:alpine` เพื่อ serve static file เท่านั้น อย่าสับสนกับ Nginx reverse proxy อีกตัวใน service `nginx` ที่อยู่ด้านหน้า ทั้งสองตัวใช้ image Nginx ได้เหมือนกันแต่ทำหน้าที่ต่างกัน

## Nginx

```bash
cd ../nginx
nano default.conf
```

Nginx ตัวนี้เป็น reverse proxy หลักของระบบ รับ traffic จาก host ที่ port 8080 แล้ว route ไป service ภายใน Docker network

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

อธิบาย routing:

- request `/` ไป `frontend:80`
- request `/api/` ไป `backend:3000`

ชื่อ `frontend` และ `backend` มาจากชื่อ service ใน `docker-compose.yml` ถ้าชื่อ service เปลี่ยน ต้องแก้ proxy_pass ให้ตรงกัน

ตัวอย่างที่ผิด:

```nginx
proxy_pass http://localhost:3000;
```

ใน container Nginx, `localhost` คือ container Nginx เอง ไม่ใช่ backend container

ตัวอย่างที่ถูก:

```nginx
proxy_pass http://backend:3000;
```

เพราะ Docker Compose network resolve ชื่อ `backend` ได้

## docker-compose.yml

```bash
cd ..
nano docker-compose.yml
```

ไฟล์ Compose นี้คือแผนประกอบระบบทั้งหมด แต่ละ service จะถูกสร้างใน network เดียวกันชื่อ `app_net` และ database ใช้ volume `db_data` เพื่อเก็บข้อมูลถาวร

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

อธิบายส่วนสำคัญ:

- `db.image` ใช้ PostgreSQL image สำเร็จรูป
- `db.environment` กำหนด database, user และ password ตอน init
- `db.volumes` เก็บ data ไว้ใน named volume ไม่หายเมื่อ container ถูกลบ
- `backend.build` build image จาก `./backend`
- `backend.environment` ส่งค่า config ให้ app
- `depends_on` กำหนดลำดับ start container แต่ไม่ได้แปลว่า service พร้อมใช้งาน 100%
- `nginx.ports` map host port `8080` ไป container port `80`
- `nginx.volumes` mount config จาก host เข้า container แบบ read-only
- `networks` ทำให้ service คุยกันด้วยชื่อ service ได้

ข้อควรระวังเรื่อง `depends_on`:

```text
depends_on บอกว่าให้ start db ก่อน backend
แต่ไม่ได้รอจน PostgreSQL พร้อมรับ connection เสมอไป
```

ในระบบจริงควรมี retry logic ใน app หรือ healthcheck ใน Compose/Kubernetes แต่ใน lab นี้ backend query จะใช้ได้หลัง db พร้อม ถ้าช่วงแรก `/api/db` fail ให้ดู log และลองอีกครั้งหลัง db init เสร็จ

เรื่อง volume:

```text
db_data:/var/lib/postgresql/data
```

ทำให้ข้อมูล PostgreSQL อยู่ใน Docker volume ถ้า `docker compose down` เฉย ๆ volume ยังอยู่ แต่ถ้าใช้ `docker compose down -v` volume จะถูกลบและข้อมูลหาย

## Run และทดสอบ

```bash
docker compose up -d --build
docker compose ps
curl http://localhost:8080/api/health
curl http://localhost:8080/api/db
```

คำสั่ง `docker compose up -d --build` ทำหลายอย่างพร้อมกัน:

- build image ของ service ที่มี `build`
- create network และ volume ถ้ายังไม่มี
- create/start container ทุก service
- run แบบ detached ด้วย `-d`

ตรวจสถานะ:

```bash
docker compose ps
```

ผลที่คาดหวังคือทุก service อยู่สถานะ running ถ้า service ใด exited ให้ดู log ของ service นั้นทันที

การทดสอบ API:

- `/api/health` ตรวจว่า backend ตอบได้ผ่าน Nginx
- `/api/db` ตรวจว่า backend ต่อ PostgreSQL ได้

ถ้า `/api/health` ผ่านแต่ `/api/db` fail ปัญหาน่าจะอยู่ที่ database connection ไม่ใช่ Nginx หรือ frontend

เปิด browser:

```text
http://app-server:8080
```

ถ้าเปิดจากเครื่อง host ให้แน่ใจว่า:

- `app-server` resolve เป็น IP ของ VM ได้
- UFW อนุญาต port 8080 หรือปิด firewall สำหรับ lab
- Compose service `nginx` map port `8080:80` อยู่

ถ้า browser เข้าไม่ได้แต่ `curl http://localhost:8080` บน VM ผ่าน แปลว่าปัญหาอยู่ที่ network/firewall/hostname จาก host ไป VM

## ตรวจสอบ

```bash
docker compose logs -f backend
docker compose logs -f nginx
```

คำสั่ง log ราย service ช่วยแยกปัญหา:

- `backend` ใช้ดูว่า API start หรือ database connection error
- `nginx` ใช้ดู request และ proxy error
- `db` ใช้ดู PostgreSQL init และ readiness

คำสั่งที่ควรใช้เพิ่ม:

```bash
docker compose logs -f db
docker compose exec backend sh
docker compose exec db psql -U appuser -d appdb
docker network ls
docker volume ls
```

ตัวอย่างอาการและจุดตรวจ:

```text
502 Bad Gateway
-> nginx ติดต่อ backend ไม่ได้
-> ตรวจ docker compose ps และ logs backend

/api/db error
-> backend รัน แต่ต่อ database ไม่ได้
-> ตรวจ env DB_HOST/DB_USER/DB_PASSWORD และ logs db

หน้าเว็บเปิดได้ แต่ปุ่ม Check API ไม่ขึ้นผล
-> เปิด browser devtools หรือ curl /api/health
-> ตรวจว่า Nginx route /api/ ไป backend ถูกไหม
```

ถ้าต้องล้าง container แต่เก็บ database:

```bash
docker compose down
docker compose up -d --build
```

ถ้าต้องล้าง database ด้วย:

```bash
docker compose down -v
```

ใช้ `-v` อย่างระวัง เพราะจะลบ volume และข้อมูลใน PostgreSQL

## ข้อสรุป

Docker Compose เหมาะกับ lab, local development และการ deploy ขนาดเล็ก ช่วยให้เห็น dependency ระหว่าง service ชัดเจน

สิ่งที่ควรเข้าใจจากบทนี้:

```text
service name = DNS ภายใน Compose network
environment = config ที่ส่งเข้า container
volume = data ที่ต้องอยู่รอด
ports = เปิด service จาก container ออก host
logs ราย service = จุดเริ่มต้นของการ debug
```

Compose เป็นสะพานสำคัญก่อนเข้า Kubernetes เพราะแนวคิดหลายอย่างคล้ายกัน เช่น service, environment, volume, network และ health ของ workload เพียงแต่ Kubernetes จะจัดการในระดับ cluster

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [8. Lab 05: Docker พื้นฐาน](./08-lab-05-docker-basics.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [10. Lab 07: Private Container Registry](./10-lab-07-private-container-registry.md)

<!-- lesson-nav:end -->
