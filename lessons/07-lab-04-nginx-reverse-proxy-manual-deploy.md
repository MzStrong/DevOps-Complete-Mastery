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
