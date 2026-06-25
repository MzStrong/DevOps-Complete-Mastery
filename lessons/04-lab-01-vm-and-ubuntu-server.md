# 4. Lab 01: เตรียม VM และ Ubuntu Server

## วัตถุประสงค์

- สร้าง VM บน VMware
- ติดตั้ง Ubuntu Server
- ตั้ง hostname และ static IP
- เปิด SSH
- ตั้ง `/etc/hosts` ให้เรียกชื่อเครื่องได้

## ขั้นตอนติดตั้ง VM

1. สร้าง VM ใหม่ใน VMware
2. เลือก ISO Ubuntu Server
3. กำหนด CPU/RAM/Disk ตามตาราง
4. ตั้ง Network เป็น NAT หรือ Host-only
5. ระหว่างติดตั้งให้เลือกติดตั้ง OpenSSH Server
6. สร้าง user ชื่อ `devops`

## ตั้ง hostname

ตัวอย่างบน app-server:

```bash
sudo hostnamectl set-hostname app-server
hostnamectl
```

## ตั้ง Static IP ด้วย Netplan

ตรวจชื่อ interface:

```bash
ip a
```

แก้ไฟล์ netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

ตัวอย่าง:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.56.20/24
      routes:
        - to: default
          via: 192.168.56.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Apply:

```bash
sudo netplan apply
ip a
ip route
```

## ตั้งค่า /etc/hosts ทุกเครื่อง

```bash
sudo nano /etc/hosts
```

เพิ่ม:

```text
192.168.56.10 devops-control
192.168.56.20 app-server
192.168.56.30 monitor-server
192.168.56.100 k8s-master-01
192.168.56.101 k8s-worker-01
192.168.56.102 k8s-worker-02
```

## การทดสอบ

```bash
ping -c 4 app-server
ping -c 4 devops-control
ssh devops@app-server
```

## การตรวจสอบ

- `hostnamectl` แสดงชื่อเครื่องถูกต้อง
- `ip a` แสดง IP ตามที่กำหนด
- ping ข้ามเครื่องได้
- SSH เข้าเครื่องได้

## ข้อสรุป

พื้นฐาน VM, IP, hostname และ SSH ต้องนิ่งก่อนเริ่ม DevOps Lab เพราะทุกเครื่องมือจะพึ่งพา network และ hostname

---
