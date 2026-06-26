# 15. Lab 12: Logging ด้วย Loki และ Promtail

Lab นี้เพิ่ม centralized logging เพื่อเก็บ log จาก server ไว้ที่เดียว บท monitoring ก่อนหน้าช่วยบอกว่า “ระบบผิดปกติเมื่อไรและมากแค่ไหน” ส่วน logging ช่วยตอบว่า “เกิดเหตุการณ์อะไรขึ้น” เช่น service start fail, database connect ไม่ได้ หรือ request error

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

ภาพรวมการทำงาน:

```text
app-server
-> Promtail อ่าน /var/log/*.log
-> ส่ง log ไป Loki ที่ monitor-server:3100
-> Grafana query Loki ด้วย LogQL
-> ผู้ใช้ค้น log จาก browser
```

Loki ไม่เหมือน Elasticsearch ตรงที่ Loki เน้น index label มากกว่า index full text ทั้งหมด ดังนั้นการเลือก label ให้ดี เช่น `host`, `job`, `service` จะช่วยให้ query ง่ายและเร็วขึ้น

## เพิ่ม Loki ใน docker-compose

บน monitor-server แก้ `docker-compose.yml` เพิ่ม service:

```yaml
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
```

Loki เป็น service สำหรับรับและเก็บ log:

- port `3100` เป็น HTTP API ของ Loki
- `local-config.yaml` เป็น config default ที่มากับ image เหมาะกับ lab

ถ้าใช้ Compose จากบท Monitoring อยู่แล้ว ให้เพิ่ม service `loki` ในไฟล์เดียวกัน เพื่อให้ Grafana เรียก Loki ผ่าน service name `loki` ได้ใน Docker network เดียวกัน

ข้อควรระวัง: config นี้เหมาะกับ lab ไม่ใช่ production storage ระยะยาว ถ้าใช้จริงต้องออกแบบ retention, storage backend, auth และ resource ให้เหมาะสม

Run:

```bash
docker compose up -d
```

ตรวจ Loki:

```bash
docker compose ps
docker compose logs -f loki
curl http://localhost:3100/ready
```

ถ้า `/ready` ตอบ `ready` แปลว่า Loki พร้อมรับ log

จาก app-server ควรตรวจว่าเข้าถึง Loki ได้:

```bash
curl http://monitor-server:3100/ready
```

ถ้า monitor-server เรียกได้แต่ app-server เรียกไม่ได้ ให้ตรวจ `/etc/hosts`, firewall หรือ network ระหว่างเครื่อง

## Run Promtail บน app-server

```bash
mkdir -p ~/promtail
nano ~/promtail/config.yml
```

Promtail เป็น agent ที่อ่านไฟล์ log แล้วส่งไป Loki ในบทนี้รัน Promtail เป็น container บน `app-server` และ mount `/var/log` แบบ read-only เพื่ออ่าน log ของ host

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

อธิบาย config:

- `server.http_listen_port: 9080` เปิด endpoint ของ Promtail เอง สำหรับดูสถานะ/debug
- `positions.filename` ใช้จำว่าอ่าน log ไปถึงบรรทัดไหนแล้ว ป้องกันการส่งซ้ำมากเกินไปหลัง restart
- `clients.url` คือ Loki endpoint ที่จะส่ง log ไป
- `scrape_configs` กำหนดว่าจะอ่าน log จากที่ไหน
- `labels` คือ metadata ที่ติดไปกับ log
- `__path__: /var/log/*.log` บอก path log ที่ Promtail จะอ่าน

label ที่ดีควรช่วยตอบคำถามตอน query เช่น log นี้มาจาก host ไหน หรือ job อะไร

ตัวอย่างที่ผิด:

```yaml
clients:
  - url: http://localhost:3100/loki/api/v1/push
```

ถ้า Promtail รันบน `app-server`, `localhost` หมายถึง container/host ฝั่ง app-server ไม่ใช่ Loki บน monitor-server

ตัวอย่างที่ถูก:

```yaml
clients:
  - url: http://monitor-server:3100/loki/api/v1/push
```

ข้อควรระวัง: Promtail รันเป็น container จึงอาจไม่เห็น `/etc/hosts` ของ host โดยตรง ถ้า container resolve `monitor-server` ไม่ได้ ให้ใช้ IP ของ monitor-server ใน `clients.url` หรือเพิ่ม `--add-host monitor-server:192.168.56.30` ตอน `docker run`

เพราะต้องส่ง log ไปยัง Loki ที่ monitor-server

Run:

```bash
docker run -d --name promtail --restart always \
  --add-host monitor-server:192.168.56.30 \
  -v ~/promtail/config.yml:/etc/promtail/config.yml \
  -v /var/log:/var/log:ro \
  grafana/promtail:latest \
  -config.file=/etc/promtail/config.yml
```

คำสั่งนี้:

- mount config เข้า container
- mount `/var/log` ของ host เข้า container แบบ read-only
- `--add-host monitor-server:192.168.56.30` ทำให้ container resolve ชื่อ `monitor-server` ได้ใน lab ที่ไม่มี DNS จริง ให้เปลี่ยน IP ตาม network plan ของคุณ
- รัน Promtail ด้วย config ที่กำหนด
- restart อัตโนมัติเมื่อ reboot หรือ container crash

ตรวจ Promtail:

```bash
docker ps | grep promtail
docker logs promtail
curl http://localhost:9080/targets
```

ถ้า `docker logs promtail` มี error เรื่อง permission หรือ path ไม่พบ ให้ตรวจว่า mount `/var/log:/var/log:ro` ถูกต้องหรือไม่ และไฟล์ log ที่ต้องการอ่านมีอยู่จริงไหม

ถ้า error ส่ง log ไป Loki ไม่ได้ ให้ตรวจ:

```bash
getent hosts monitor-server
curl http://monitor-server:3100/ready
```

## Grafana

เพิ่ม Data Source เป็น Loki:

```text
URL: http://loki:3100
```

ตั้ง URL เป็น `http://loki:3100` เพราะ Grafana กับ Loki อยู่ใน Docker Compose network เดียวกันบน monitor-server ถ้าใช้ `http://monitor-server:3100` ก็อาจได้ในบางกรณี แต่ service name ตรงกว่าใน Compose

Query:

```text
{host="app-server"}
{job="varlogs"}
```

Query เหล่านี้ใช้ LogQL:

- `{host="app-server"}` แสดง log ที่มี label host เป็น app-server
- `{job="varlogs"}` แสดง log ใน job varlogs

ตัวอย่าง query เพิ่มเติม:

```text
{host="app-server"} |= "error"
{job="varlogs"} |= "ssh"
{host="app-server"} != "debug"
```

ความหมาย:

- `|= "error"` กรอง log line ที่มีคำว่า error
- `|= "ssh"` หา log ที่เกี่ยวกับ ssh
- `!= "debug"` ตัด log line ที่มีคำว่า debug

อย่าใส่ label เยอะเกินจำเป็นหรือใส่ค่าที่เปลี่ยนตลอดเวลา เช่น request id เป็น label เพราะจะทำให้ cardinality สูงและระบบหนักขึ้น ใน lab นี้ใช้ `host` และ `job` เพียงพอ

## ทดสอบสร้าง log

บน app-server ลองสร้าง log ง่าย ๆ:

```bash
logger "devops lab test log from app-server"
```

แล้ว query ใน Grafana:

```text
{host="app-server"} |= "devops lab test"
```

ถ้าไม่เจอทันที ให้รอสักครู่แล้ว refresh เพราะ Promtail ต้องอ่านไฟล์และ push ไป Loki ก่อน

## Debug log ไม่เข้า Loki

ไล่ตรวจตามลำดับ:

บน monitor-server:

```bash
docker compose ps
docker compose logs loki
curl http://localhost:3100/ready
```

บน app-server:

```bash
docker ps | grep promtail
docker logs promtail
curl http://monitor-server:3100/ready
ls -lah /var/log/*.log
curl http://localhost:9080/targets
```

ใน Grafana:

```text
ตรวจ datasource Loki ว่า Save & test ผ่าน
ลอง query {job="varlogs"}
ลองขยาย time range เช่น Last 1 hour
```

อาการที่พบบ่อย:

```text
Grafana datasource fail
-> Grafana ติดต่อ Loki ไม่ได้ หรือ URL ผิด

Promtail error connect Loki
-> monitor-server resolve ไม่ได้, port 3100 block, Loki ไม่พร้อม

Promtail container resolve monitor-server ไม่ได้
-> ใช้ IP ตรงใน clients.url หรือเพิ่ม --add-host ตอน docker run

Query แล้วว่าง
-> label ผิด, time range แคบเกิน, __path__ ไม่ match ไฟล์ log

เห็น log เก่าแต่ไม่เห็น log ใหม่
-> positions file หรือ Promtail target มีปัญหา ตรวจ /targets และ docker logs
```

## ข้อสรุป

Centralized logging ทำให้ไม่ต้อง SSH เข้าเครื่องทีละตัว และช่วย debug เหตุการณ์ย้อนหลังได้ง่าย

สิ่งที่ควรจำ:

```text
Promtail อ่าน log
Loki เก็บ log
Grafana query log
label ใช้เลือกชุด log
LogQL ใช้ค้นข้อความใน log
```

Monitoring และ Logging ควรใช้คู่กัน: metrics บอกว่าปัญหาเกิดเมื่อไรและรุนแรงแค่ไหน ส่วน log ช่วยบอกว่าระบบทำอะไรในช่วงเวลานั้น

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [14. Lab 11: Monitoring ด้วย Prometheus และ Grafana](./14-lab-11-prometheus-grafana-monitoring.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [16. Lab 13: Kubernetes Cluster ด้วย kubeadm](./16-lab-13-kubernetes-kubeadm.md)

<!-- lesson-nav:end -->
