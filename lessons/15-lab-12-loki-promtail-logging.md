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
