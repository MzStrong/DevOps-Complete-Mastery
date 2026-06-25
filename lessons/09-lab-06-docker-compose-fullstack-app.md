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
