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
