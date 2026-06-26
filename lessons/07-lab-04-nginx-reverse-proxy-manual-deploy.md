# 7. Lab 04: Nginx Reverse Proxy และ Manual Deploy

Lab นี้เป็นครั้งแรกที่เรานำ application ขึ้นไปรันเป็น service จริงบน server โดยยังไม่ใช้ Docker หรือ Kubernetes จุดประสงค์คือให้เข้าใจพื้นฐานว่า application ต้องมี process, port, service manager, reverse proxy และ log อย่างไร ก่อนที่เครื่องมืออัตโนมัติในบทหลังจะเข้ามาช่วย

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

request flow ของบทนี้คือ:

```text
client
-> http://app-server:80
-> Nginx
-> proxy ไป http://127.0.0.1:3000
-> Node.js simple-api
-> response กลับไป client
```

Nginx รับ traffic จากภายนอกที่ port 80 ส่วน Node.js API รันอยู่ภายในเครื่องที่ port 3000 เท่านั้น การแยกแบบนี้เป็น pattern ที่ใช้บ่อยมาก เพราะ reverse proxy ช่วยจัดการ domain, TLS, header, log, routing และ security policy ได้ดีกว่าให้ app รับ public traffic โดยตรง

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

คำสั่งชุดนี้ติดตั้ง runtime และสร้าง app ง่าย ๆ:

- `nodejs` ใช้รัน JavaScript ฝั่ง server
- `npm` ใช้จัดการ dependency
- `express` เป็น web framework สำหรับสร้าง HTTP API
- directory `~/apps/simple-api` เป็นที่เก็บ app ของ user `devops`

ใน production จริงอาจใช้ Node.js จาก NodeSource หรือ version manager เพื่อควบคุม version ให้ชัดเจนกว่า package จาก apt แต่สำหรับ Lab นี้ใช้ apt เพื่อให้เริ่มง่ายและโฟกัสที่ deployment flow

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

API นี้มี 2 endpoint:

- `/` ส่งข้อความและ hostname กลับมา ใช้ดูว่า request ไปถึง app และ app รันบนเครื่องไหน
- `/health` ส่ง status `ok` ใช้เป็น health check ว่า app ยังตอบสนอง

health check ควรเป็น endpoint ที่เบาและไม่ซับซ้อน ถ้า health check ต้องพึ่ง dependency หนักเกินไป เช่น database หรือ external API ทุกครั้ง อาจทำให้ระบบมองว่า app ไม่พร้อมทั้งที่ตัว process ยังทำงานอยู่ ต้องเลือกตามความหมายที่ต้องการวัด

ทดสอบ:

```bash
node index.js
curl http://localhost:3000/health
```

ตอนรัน `node index.js` process จะผูกอยู่กับ terminal นั้น ถ้าปิด terminal app ก็หยุด นี่คือเหตุผลที่ต้องใช้ systemd ในขั้นถัดไป

ผลที่คาดหวัง:

```json
{"status":"ok"}
```

ถ้า `curl` ไม่ผ่าน ให้ตรวจว่า terminal ที่รัน `node index.js` ยังเปิดอยู่หรือไม่ และมีข้อความ `API listening on 3000` หรือไม่

ตัวอย่างที่ผิด:

```text
รัน node index.js แล้วคิดว่า deploy เสร็จ
ปิด terminal
app หยุดทันที
```

ตัวอย่างที่ถูก:

```text
ทดสอบด้วย node index.js ก่อน
เมื่อ app ทำงานแล้ว จึงย้ายไปให้ systemd เป็นคนคุม process
```

## สร้าง systemd service

```bash
sudo nano /etc/systemd/system/simple-api.service
```

systemd service ทำให้ app กลายเป็น background service ที่ Linux คุมได้ เช่น start, stop, restart, enable หลัง reboot และดู log ผ่าน journalctl ได้ ถ้าไม่ทำขั้นนี้ app จะขึ้นกับ terminal ของ user มากเกินไป

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

ส่วนสำคัญใน service file:

- `User=devops` ให้ service รันด้วย user ปกติ ไม่ใช้ root
- `WorkingDirectory=/home/devops/apps/simple-api` กำหนด directory ที่ app ทำงาน
- `Environment=PORT=3000` กำหนด port ให้ app
- `ExecStart=/usr/bin/node ...` คำสั่งที่ systemd ใช้ start app
- `Restart=always` ให้ systemd start ใหม่ถ้า app crash

จุดที่พลาดบ่อยคือ path ไม่ตรงกับเครื่องจริง ถ้า user ไม่ใช่ `devops` หรือ app อยู่คนละ path service จะ start ไม่ขึ้น

ตรวจ path ก่อน:

```bash
which node
pwd
ls -lah /home/devops/apps/simple-api
```

ถ้า `which node` ไม่ใช่ `/usr/bin/node` ให้แก้ `ExecStart` ให้ตรงกับ path จริง

Start service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now simple-api
sudo systemctl status simple-api
```

`daemon-reload` จำเป็นหลังสร้างหรือแก้ไฟล์ service เพื่อให้ systemd โหลด config ใหม่ ถ้าลืมรัน systemd อาจยังไม่เห็น service หรือยังใช้ config เก่า

หลัง start ให้ตรวจ:

```bash
sudo systemctl status simple-api
sudo journalctl -u simple-api -n 50
curl http://localhost:3000/health
```

ตัวอย่างที่ผิด:

```bash
sudo nano /etc/systemd/system/simple-api.service
sudo systemctl restart simple-api
```

โดยลืม `sudo systemctl daemon-reload` หลังแก้ไฟล์ service

ตัวอย่างที่ถูก:

```bash
sudo systemctl daemon-reload
sudo systemctl restart simple-api
sudo systemctl status simple-api
```

## ตั้งค่า Nginx

```bash
sudo nano /etc/nginx/sites-available/simple-api
```

Nginx จะรับ request ที่ port 80 แล้วส่งต่อไปยัง Node.js ที่ `127.0.0.1:3000` การใช้ `127.0.0.1` หมายความว่า backend รับเฉพาะ traffic ภายในเครื่องเดียวกัน ไม่เปิด port 3000 ให้เครื่องอื่นเรียกโดยตรง

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

อธิบาย config สำคัญ:

- `listen 80` ให้ Nginx รับ HTTP ที่ port 80
- `server_name app-server` ใช้ match hostname ที่ client เรียก
- `proxy_pass http://127.0.0.1:3000` ส่ง request ไป backend
- `proxy_set_header Host $host` ส่ง host เดิมไปให้ backend
- `X-Real-IP` และ `X-Forwarded-For` ช่วยให้ backend รู้ IP ของ client จริง
- `X-Forwarded-Proto` บอก protocol เดิม เช่น http/https

ถ้าไม่ตั้ง header เหล่านี้ app อาจยังทำงานได้ แต่เวลาทำ log, redirect, auth หรือ rate limit ในระบบจริงจะขาดข้อมูล client/protocol ที่ถูกต้อง

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/simple-api /etc/nginx/sites-enabled/simple-api
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

ขั้นตอน enable site ทำงานแบบนี้:

- สร้างไฟล์ config ใน `sites-available`
- symlink ไป `sites-enabled` เพื่อเปิดใช้งาน
- ลบ default site เพื่อไม่ให้ชนกับ config ใหม่
- `nginx -t` ตรวจ syntax ก่อน reload
- reload Nginx โดยไม่ต้อง restart process ทั้งหมด

อย่าข้าม `sudo nginx -t` เพราะถ้า config ผิดแล้ว reload/restart ทันที Nginx อาจไม่รับ config ใหม่หรือ service fail

ตัวอย่างที่ผิด:

```bash
sudo systemctl reload nginx
```

ทันทีหลังแก้ config โดยไม่ตรวจ syntax

ตัวอย่างที่ถูก:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

ถ้า `nginx -t` fail ให้อ่าน error ว่าบรรทัดไหนผิด แล้วแก้ก่อน reload

## การทดสอบ

```bash
curl http://app-server
curl http://app-server/health
```

การทดสอบนี้ตรวจทั้ง flow:

```text
hostname app-server
-> IP ของ app-server
-> port 80 ของ Nginx
-> proxy ไป Node.js port 3000
-> response จาก API
```

ควรทดสอบเพิ่มเป็นลำดับ:

```bash
curl http://localhost:3000/health
curl http://localhost/health
curl http://app-server/health
```

ความหมาย:

- `localhost:3000` ตรวจ Node.js โดยตรง
- `localhost/health` ตรวจ Nginx proxy จากเครื่องเดียวกัน
- `app-server/health` ตรวจจาก hostname/network

ถ้า `localhost:3000` ผ่านแต่ `localhost/health` ไม่ผ่าน ปัญหาอยู่ที่ Nginx หรือ proxy config ถ้า `localhost/health` ผ่านแต่ `app-server/health` ไม่ผ่าน ให้ตรวจ hostname, firewall หรือ network

## การตรวจสอบ

```bash
sudo journalctl -u simple-api -f
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

เปิด log แล้วลอง `curl` อีกครั้งเพื่อดูว่า request เข้าระบบตรงไหน:

- `journalctl -u simple-api` ดู log ของ Node.js service
- `access.log` ดู request ที่ Nginx รับ
- `error.log` ดู error ของ Nginx เช่น proxy backend ไม่ได้

ตัวอย่างอาการและจุดตรวจ:

```text
curl ได้ 502 Bad Gateway
-> Nginx ทำงาน แต่ต่อ backend ไม่ได้
-> ตรวจ simple-api service และ port 3000

curl connection refused ที่ port 80
-> Nginx ไม่ได้ listen port 80 หรือ firewall/block
-> ตรวจ systemctl status nginx และ ss -tulpen

curl app-server ไม่ resolve
-> hostname ผิด
-> ตรวจ /etc/hosts หรือ DNS
```

คำสั่ง debug ที่ใช้บ่อย:

```bash
sudo systemctl status simple-api
sudo systemctl status nginx
ss -tulpen | grep -E ':80|:3000'
sudo nginx -t
curl -v http://app-server/health
```

## ข้อสรุป

Manual deploy ช่วยให้เข้าใจว่า app ทำงานบน server อย่างไร ก่อนจะย้ายไป Docker, CI/CD และ Kubernetes

เมื่อเข้าใจบทนี้ บท Docker และ Kubernetes จะง่ายขึ้น เพราะ container และ pod ก็ยังมีแนวคิดเดิมอยู่ข้างใน:

```text
process ต้องรัน
port ต้องเปิด
config ต้องถูก
log ต้องอ่านได้
proxy/service ต้องส่ง traffic ถูกที่
health check ต้องบอกสถานะได้
```

เครื่องมือหลังจากนี้จะช่วย automate ขั้นตอน แต่ไม่ได้ลบพื้นฐานเหล่านี้ออกไป

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [6. Lab 03: Network, DNS, Port และ Firewall](./06-lab-03-network-dns-port-firewall.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [8. Lab 05: Docker พื้นฐาน](./08-lab-05-docker-basics.md)

<!-- lesson-nav:end -->
