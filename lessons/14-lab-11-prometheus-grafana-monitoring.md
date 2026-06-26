# 14. Lab 11: Monitoring ด้วย Prometheus และ Grafana

Lab นี้เพิ่มการมองเห็นสถานะระบบด้วย metrics ก่อนหน้านี้เรา debug ด้วยคำสั่งบนเครื่องโดยตรง เช่น `top`, `free`, `df`, `systemctl` แต่ monitoring ทำให้เราเก็บข้อมูลต่อเนื่องและดูย้อนหลังได้ เช่น CPU สูงตอนไหน, disk เหลือน้อยลงเร็วแค่ไหน หรือ server ยังตอบ metrics อยู่หรือไม่

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

ภาพรวมการทำงาน:

```text
app-server
-> node_exporter เปิด metrics ที่ :9100
-> Prometheus บน monitor-server scrape app-server:9100
-> Grafana อ่านข้อมูลจาก Prometheus
-> Dashboard แสดง CPU/RAM/Disk/Network
```

Prometheus ไม่ได้รอให้ agent push ข้อมูลมา แต่จะไปดึงข้อมูลจาก target ตามรอบที่กำหนด เรียกว่า scrape ถ้า network, DNS หรือ firewall มีปัญหา target จะขึ้น down

## ติดตั้ง Node Exporter บน app-server

```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/
```

Node Exporter เป็น exporter สำหรับ metrics ของ Linux server เช่น CPU, memory, disk, filesystem และ network

เหตุผลที่สร้าง user `node_exporter` แยก:

- ไม่ต้องรัน exporter ด้วย root ถ้าไม่จำเป็น
- จำกัดสิทธิ์ของ process ให้แคบลง
- เป็นแนวทางที่ใช้กับ service daemon ทั่วไป

หลัง copy binary แล้วตรวจได้:

```bash
ls -lah /usr/local/bin/node_exporter
/usr/local/bin/node_exporter --version
```

ถ้า binary execute ไม่ได้ ให้ตรวจ permission:

```bash
chmod +x /usr/local/bin/node_exporter
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

service นี้ทำให้ Node Exporter start อัตโนมัติและถูก systemd ดูแลเหมือน service อื่น:

- `User=node_exporter` รันด้วย user เฉพาะ
- `ExecStart=/usr/local/bin/node_exporter` ระบุ binary ที่จะรัน
- ค่า default ของ Node Exporter จะ listen ที่ port `9100`

Start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
curl http://localhost:9100/metrics | head
```

หลัง start ให้ตรวจ 3 อย่าง:

```bash
sudo systemctl status node_exporter
ss -tulpen | grep ':9100'
curl http://localhost:9100/metrics | head
```

ผลของ `/metrics` จะเป็น text format ที่ Prometheus อ่านได้ เช่น:

```text
node_cpu_seconds_total{cpu="0",mode="idle"} ...
node_memory_MemAvailable_bytes ...
```

ถ้า `localhost:9100/metrics` ไม่ได้ ให้แก้ที่ app-server ก่อน อย่าเพิ่งไปแก้ Prometheus เพราะ target ยังไม่พร้อมให้ scrape

ถ้าจะให้ monitor-server เข้าถึง ต้องตรวจจาก monitor-server ด้วย:

```bash
curl http://app-server:9100/metrics | head
```

ถ้าจาก app-server ผ่าน แต่จาก monitor-server ไม่ผ่าน ให้ตรวจ `/etc/hosts`, route หรือ firewall port 9100

## Run Prometheus + Grafana บน monitor-server

```bash
mkdir -p ~/monitoring/prometheus
cd ~/monitoring
nano prometheus/prometheus.yml
```

ไฟล์ `prometheus.yml` คือ config หลักของ Prometheus ใช้กำหนดว่าจะ scrape target อะไรบ้างและ scrape ถี่แค่ไหน

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets:
          - 'app-server:9100'
```

อธิบาย config:

- `scrape_interval: 15s` ให้ Prometheus ดึง metrics ทุก 15 วินาที
- `job_name: 'node-exporter'` ชื่อกลุ่ม target
- `targets: ['app-server:9100']` endpoint ที่ Prometheus จะ scrape

ชื่อ `app-server` ต้อง resolve ได้จาก `monitor-server` เพราะ Prometheus container จะรันบน `monitor-server` ถ้า container resolve ชื่อนี้ไม่ได้ target จะ down

ตัวอย่างที่ผิด:

```yaml
targets:
  - 'localhost:9100'
```

ใน Prometheus container, `localhost` หมายถึง container Prometheus เอง ไม่ใช่ app-server

ตัวอย่างที่ถูก:

```yaml
targets:
  - 'app-server:9100'
```

หรือใช้ IP ตรง:

```yaml
targets:
  - '192.168.56.20:9100'
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

Compose นี้รันสอง service:

- `prometheus` เก็บ metrics และเปิด UI/API ที่ port 9090
- `grafana` ใช้สร้าง dashboard และเปิด UI ที่ port 3000

volume:

- `prometheus_data` เก็บ time series data
- `grafana_data` เก็บ dashboard, datasource และ config ของ Grafana

ถ้าใช้ `docker compose down -v` ข้อมูล metrics และ dashboard จะหาย เพราะ volume ถูกลบ

Run:

```bash
docker compose up -d
```

ตรวจ container:

```bash
docker compose ps
docker compose logs -f prometheus
docker compose logs -f grafana
```

เข้าใช้งาน:

```text
Prometheus: http://monitor-server:9090
Grafana:    http://monitor-server:3000
Login: admin / admin
```

หลัง login Grafana ครั้งแรก ระบบอาจให้เปลี่ยน password ให้จดไว้ใน note ส่วนตัวและอย่า commit ลง repository

## ตั้งค่า Grafana Datasource

ใน Grafana ให้เพิ่ม datasource:

```text
Type: Prometheus
URL:  http://prometheus:9090
```

เหตุผลที่ใช้ `http://prometheus:9090` ไม่ใช่ `http://monitor-server:9090` เพราะ Grafana container กับ Prometheus container อยู่ใน Docker Compose network เดียวกัน จึงเรียกกันด้วย service name ได้

ถ้า Grafana test datasource ไม่ผ่าน ให้ตรวจ:

```bash
docker compose ps
docker compose logs prometheus
docker compose logs grafana
```

และลอง exec เข้า Grafana container:

```bash
docker compose exec grafana sh
```

แล้วทดสอบ network ถ้ามีเครื่องมือที่ image รองรับ

Prometheus query:

```text
up
node_memory_MemAvailable_bytes
node_cpu_seconds_total
node_filesystem_avail_bytes
```

ความหมายของ query:

- `up` บอก target scrape สำเร็จหรือไม่ `1` คือ up, `0` คือ down
- `node_memory_MemAvailable_bytes` คือ memory ที่เหลือใช้งานได้
- `node_cpu_seconds_total` คือ counter เวลา CPU แยกตาม mode
- `node_filesystem_avail_bytes` คือพื้นที่ disk ที่เหลือ

ตัวอย่าง query ที่อ่านง่ายขึ้น:

```text
node_memory_MemAvailable_bytes / 1024 / 1024 / 1024
```

แปลง memory available เป็น GiB โดยประมาณ

สำหรับ CPU จริง ๆ ต้องใช้ rate เพราะ `node_cpu_seconds_total` เป็น counter:

```text
rate(node_cpu_seconds_total[5m])
```

## Debug target down

ถ้า Prometheus หน้า Targets แสดง `app-server:9100` เป็น down ให้ไล่ตรวจ:

จาก `monitor-server`:

```bash
getent hosts app-server
curl http://app-server:9100/metrics | head
```

จาก Prometheus container:

```bash
docker compose exec prometheus sh
```

แล้วลองตรวจ network เท่าที่ image มีเครื่องมือให้ ถ้าไม่มี `curl` ให้ตรวจจาก host ก่อน

จาก `app-server`:

```bash
sudo systemctl status node_exporter
ss -tulpen | grep ':9100'
sudo ufw status numbered
```

อาการที่พบบ่อย:

```text
curl localhost:9100 บน app-server ผ่าน แต่ monitor-server curl ไม่ผ่าน
-> firewall หรือ network ระหว่างเครื่อง

monitor-server curl app-server:9100 ไม่ได้เพราะ no such host
-> /etc/hosts หรือ DNS ผิด

Prometheus config ใช้ localhost:9100
-> Prometheus scrape ตัวเอง ไม่ใช่ app-server
```

## Dashboard ควรดูอะไร

Dashboard เบื้องต้นควรมี:

- CPU usage
- Memory available/used
- Disk available/used
- Network receive/transmit
- Target up/down

อย่าเริ่มจาก dashboard ที่สวยอย่างเดียว ให้เริ่มจาก panel ที่ตอบคำถาม operation ได้จริง เช่น “disk จะเต็มไหม”, “server ยังส่ง metrics อยู่ไหม”, “memory ลดลงเรื่อย ๆ หรือเปล่า”

## ข้อสรุป

Monitoring ทำให้รู้สถานะระบบแบบตัวเลข เช่น CPU, RAM, Disk, Service up/down ก่อนเกิดผลกระทบรุนแรง

สิ่งที่ควรจำ:

```text
Exporter expose metrics
Prometheus scrape metrics
Grafana visualize metrics
up == 1 แปลว่า scrape สำเร็จ
target down ต้องไล่จาก service, port, network, config
```

Monitoring ไม่ได้แทน logging แต่ช่วยบอกว่า “ระบบเริ่มผิดปกติตอนไหนและผิดปกติแค่ไหน” ส่วน log จะช่วยอธิบายรายละเอียดของเหตุการณ์ ซึ่งจะต่อในบทถัดไป

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [13. Lab 10: Terraform และ Infrastructure as Code](./13-lab-10-terraform-iac.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [15. Lab 12: Logging ด้วย Loki และ Promtail](./15-lab-12-loki-promtail-logging.md)

<!-- lesson-nav:end -->
