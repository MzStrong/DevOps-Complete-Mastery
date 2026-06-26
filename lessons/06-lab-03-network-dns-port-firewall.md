# 6. Lab 03: Network, DNS, Port และ Firewall

Lab นี้ฝึกไล่ตรวจปัญหา network แบบเป็นลำดับ เพราะงาน DevOps จำนวนมากเริ่มจากคำถามง่าย ๆ ว่า “เครื่องถึงกันไหม”, “ชื่อ resolve ได้ไหม”, “service เปิด port อยู่ไหม” และ “firewall อนุญาตหรือเปล่า” ถ้าตอบคำถามเหล่านี้ไม่ได้ จะ debug Docker, GitLab, Monitoring หรือ Kubernetes ได้ยากมาก

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

คำศัพท์ในบทนี้เป็นพื้นฐานของทุกระบบที่มีการคุยกันผ่าน network เช่น Nginx รับ HTTP ที่ port 80, GitLab ใช้ HTTP/HTTPS และ SSH, Kubernetes API ใช้ port 6443, Prometheus scrape target ผ่าน HTTP endpoint

## ตรวจ network

```bash
ip a
ip route
resolvectl status
```

คำสั่งกลุ่มนี้ใช้ดู network ชั้นพื้นฐาน:

- `ip a` ดูว่าเครื่องมี IP อะไร และอยู่ interface ไหน
- `ip route` ดูว่า traffic ออกไปทาง gateway ไหน
- `resolvectl status` ดู DNS server ที่ระบบใช้ resolve ชื่อ

สิ่งที่ควรตรวจจาก `ip a`:

```text
interface ที่ใช้ต้องมี IP ตรงกับ network plan
เช่น app-server ควรเห็น 192.168.56.20/24
```

ถ้า interface ไม่มี IP หรือ IP ไม่ตรงกับแผน ให้ย้อนกลับไปตรวจ Netplan ใน Lab 01

สิ่งที่ควรตรวจจาก `ip route`:

```text
default via 192.168.56.1 dev ens33
```

ถ้าไม่มี default route เครื่องอาจคุยกับ VM ใน subnet เดียวกันได้ แต่ download package จาก internet ไม่ได้

ตัวอย่างอาการ:

```text
ping app-server ได้
apt update ไม่ได้
```

มักแปลว่า local network ใช้ได้ แต่ default gateway หรือ DNS มีปัญหา

## ตรวจ DNS

```bash
sudo apt update
sudo apt install -y dnsutils
nslookup google.com
dig google.com
```

`dnsutils` เพิ่มเครื่องมืออย่าง `nslookup` และ `dig` สำหรับตรวจ DNS โดยตรง ถ้า DNS เสีย คุณอาจ ping IP ได้แต่ ping hostname ไม่ได้ หรือ `apt update` fail เพราะ resolve ชื่อ repository ไม่ได้

การแปลผลแบบง่าย:

```bash
nslookup google.com
```

ถ้าได้ IP กลับมา แปลว่า DNS ใช้งานได้ในระดับหนึ่ง

```bash
dig google.com
```

ดูส่วน `ANSWER SECTION` ถ้ามี record กลับมาแปลว่า resolve สำเร็จ

ตัวอย่างที่ผิด:

```text
curl http://app-server ไม่ได้
สรุปทันทีว่า Nginx พัง
```

ตัวอย่างที่ถูก:

```bash
getent hosts app-server
ping -c 4 app-server
curl http://app-server
```

ตรวจชื่อก่อนว่า resolve ไป IP ถูกไหม แล้วค่อยตรวจ service

ใน Lab ที่ใช้ `/etc/hosts` ชื่อภายในอย่าง `app-server` อาจไม่ได้ผ่าน DNS จริง แต่คำสั่ง `getent hosts app-server` จะช่วยตรวจว่าระบบ resolve ชื่อจาก hosts หรือ DNS ได้หรือไม่

## ตรวจ port

```bash
ss -tulpen
```

`ss -tulpen` ใช้ดู port ที่ service เปิดฟังอยู่:

- `t` แสดง TCP
- `u` แสดง UDP
- `l` แสดงเฉพาะ listening socket
- `p` แสดง process ที่เปิด port
- `e/n` แสดงรายละเอียดและไม่ resolve ชื่อให้ช้า

ตัวอย่างผลที่อยากเห็นหลังติดตั้ง Nginx:

```text
LISTEN 0 511 0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=...,fd=...))
```

ถ้าไม่มี port 80 ใน `ss` แปลว่า Nginx ยังไม่ได้ listen ต่อให้ firewall เปิดก็เข้าเว็บไม่ได้

ตัวอย่างการไล่ตรวจ:

```text
curl เข้าไม่ได้
-> ss มี port 80 ไหม
-> ถ้าไม่มี ตรวจ systemctl status nginx
-> ถ้ามี ตรวจ firewall และ route ต่อ
```

## ติดตั้ง Nginx ทดสอบ port 80

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

คำสั่งนี้ติดตั้ง Nginx และสั่งให้ service start ทันทีพร้อมตั้งให้ start หลัง reboot:

- `apt install nginx` ติดตั้ง package
- `systemctl enable --now nginx` เปิดใช้งาน service เดี๋ยวนั้นและ enable ตอน boot

ตรวจหลังติดตั้ง:

```bash
sudo systemctl status nginx
ss -tulpen | grep ':80'
```

ถ้า `apt update` fail ให้ย้อนตรวจ DNS/gateway ก่อน ถ้า `systemctl status nginx` fail ให้ดู log ด้วย:

```bash
sudo journalctl -u nginx -n 50
```

ทดสอบ:

```bash
curl http://localhost
curl http://app-server
```

`curl http://localhost` ทดสอบจากเครื่องเดียวกัน ถ้าผ่านแปลว่า Nginx รันและรับ request ในเครื่องได้

`curl http://app-server` ทดสอบผ่าน hostname ถ้าตัวนี้ fail แต่ `localhost` ผ่าน ให้สงสัย DNS/hosts, firewall หรือ IP binding

ตัวอย่างการแปลผล:

```text
localhost ผ่าน, app-server ไม่ผ่าน
-> service รันแล้ว
-> ปัญหาอาจอยู่ที่ hostname, firewall หรือ network ระหว่างเครื่อง

localhost ไม่ผ่าน
-> ตรวจ Nginx service และ port ก่อน
```

## UFW Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status numbered
```

UFW เป็น firewall frontend ที่ใช้งานง่ายบน Ubuntu ใน Lab นี้ใช้เพื่อเข้าใจว่า service ที่ listen port แล้ว ยังอาจถูก firewall block ได้

ให้ allow OpenSSH ก่อน `ufw enable` เสมอ โดยเฉพาะถ้าทำผ่าน SSH:

```bash
sudo ufw allow OpenSSH
sudo ufw enable
```

ถ้าเปิด UFW โดยยังไม่ allow SSH อาจทำให้ session หลุดและ remote กลับเข้าเครื่องไม่ได้ ต้องเข้า VMware console เพื่อแก้

ตรวจ rule:

```bash
sudo ufw status numbered
```

ผลที่ควรเห็นอย่างน้อย:

```text
22/tcp หรือ OpenSSH ALLOW
80/tcp ALLOW
```

ทดสอบลบ rule port 80:

```bash
sudo ufw delete allow 80/tcp
curl http://app-server
sudo ufw allow 80/tcp
```

การลบ rule port 80 เป็นการทดลองให้เห็นว่า firewall มีผลจริง เมื่อ rule ถูกลบ Nginx อาจยังรันและ `ss` ยังเห็น port 80 แต่เครื่องอื่นจะเข้าไม่ได้

ลำดับการเข้าใจที่ถูก:

```text
Nginx running != เข้าเว็บจากข้างนอกได้เสมอ
ต้องมี:
1. service listen port 80
2. network route ถึงเครื่อง
3. firewall allow port 80
4. hostname/IP ชี้ถูกเครื่อง
```

หลังทดสอบอย่าลืมเปิด rule กลับ ไม่อย่างนั้นบท Nginx reverse proxy ถัดไปจะติดปัญหา HTTP เข้าไม่ได้

## การตรวจสอบ

- `ss -tulpen` เห็น Nginx listen port 80
- curl เข้าเว็บได้
- ปิด firewall rule แล้วเข้าไม่ได้
- เปิดกลับแล้วเข้าได้

เพิ่ม checklist ตรวจแบบไล่ชั้น:

```bash
ip a
ip route
getent hosts app-server
ping -c 4 app-server
sudo systemctl status nginx
ss -tulpen | grep ':80'
sudo ufw status numbered
curl http://app-server
```

ถ้า command ใด fail ให้แก้ที่ชั้นนั้นก่อน อย่าข้ามไปแก้ application ถ้า network พื้นฐานยังไม่ผ่าน

ตัวอย่าง root cause ที่พบบ่อย:

```text
getent hosts ผิด IP        -> /etc/hosts ผิด
ping ไม่ได้                -> IP/subnet/gateway/firewall ผิด
ss ไม่เห็น port 80         -> Nginx ไม่ได้รันหรือ config ผิด
ufw block 80/tcp           -> firewall ยังไม่ allow
curl localhost ผ่านแต่ชื่อไม่ผ่าน -> hostname หรือ firewall/network มีปัญหา
```

## ข้อสรุป

การแก้ปัญหา production มักเริ่มจาก network: เครื่องถึงไหม, DNS ถูกไหม, port listen ไหม, firewall block ไหม

จำ flow ตรวจ network พื้นฐานนี้ไว้:

```text
IP ถูกไหม
-> route ถูกไหม
-> ชื่อ resolve ถูกไหม
-> service listen port ไหม
-> firewall allow ไหม
-> client เรียกถูก URL ไหม
```

ถ้าฝึกไล่แบบนี้จนคล่อง จะช่วยลดเวลาการแก้ปัญหาในบท Docker, GitLab, Monitoring และ Kubernetes ได้มาก

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [5. Lab 02: Linux, SSH, Service และ Log](./05-lab-02-linux-ssh-service-log.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [7. Lab 04: Nginx Reverse Proxy และ Manual Deploy](./07-lab-04-nginx-reverse-proxy-manual-deploy.md)

<!-- lesson-nav:end -->
