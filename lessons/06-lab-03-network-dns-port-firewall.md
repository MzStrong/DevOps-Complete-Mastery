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
