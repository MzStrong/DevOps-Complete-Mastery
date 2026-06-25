# 2. สเปกเครื่องและ Network ที่แนะนำ

บทนี้เป็นส่วนวางแผนก่อนสร้าง VM จริง เป้าหมายคือให้กำหนดขนาดเครื่อง, ชื่อเครื่อง, IP และ user ให้เป็นระบบตั้งแต่ต้น เพราะค่าเหล่านี้จะถูกใช้ซ้ำในทุกบท ถ้าวางผิดตั้งแต่ช่วงนี้ บทหลังจะติดปัญหาแบบตามหายาก เช่น SSH ผิดเครื่อง, registry เข้าไม่ได้, Kubernetes node join ไม่ได้ หรือ monitoring scrape target ไม่เจอ

## สเปก VM แบบประหยัด

| VM | CPU | RAM | Disk | Role |
|---|---:|---:|---:|---|
| devops-control | 2 vCPU | 4-6 GB | 60 GB | GitLab, Registry |
| app-server | 2 vCPU | 2 GB | 30 GB | Docker/App |
| monitor-server | 2 vCPU | 3-4 GB | 40 GB | Monitoring/Logging |
| k8s-master-01 | 2 vCPU | 3-4 GB | 40 GB | K8s Control Plane |
| k8s-worker-01 | 2 vCPU | 2-3 GB | 40 GB | K8s Worker |
| k8s-worker-02 | 2 vCPU | 2-3 GB | 40 GB | K8s Worker |

สเปกนี้เป็นจุดเริ่มต้นแบบประหยัด ไม่ใช่ production sizing จริง จุดประสงค์คือให้เครื่องพอรัน lab ได้โดยไม่กิน resource ของ PC มากเกินไป บาง service เช่น GitLab, Grafana, Loki และ Kubernetes ใช้ RAM ค่อนข้างมาก ถ้าให้ RAM น้อยเกินไปอาจติดตั้งผ่านแต่ใช้งานแล้วช้า, service restart เอง หรือ pipeline timeout

แนวทางเลือกสเปก:

- `devops-control` ต้องใช้ RAM มากกว่า `app-server` เพราะ GitLab หนักกว่า web app ทั่วไป
- `monitor-server` ควรมี disk พอสมควร เพราะ metrics และ log โตขึ้นเรื่อย ๆ
- `k8s-master-01` ไม่ควรเล็กเกินไป เพราะ control plane ต้องรันหลาย component เช่น API server, scheduler และ controller
- worker node ควรมี RAM พอสำหรับ pod ที่จะ deploy ในบท Kubernetes

ถ้า RAM น้อย ให้เริ่มจาก 3 เครื่องก่อน:

```text
devops-control
app-server
k8s-master-01 + k8s-worker-01 เมื่อถึงบท Kubernetes
```

ถ้าเครื่องหลักมี RAM จำกัด ไม่จำเป็นต้องเปิด VM ทุกเครื่องพร้อมกันตั้งแต่ต้น ให้เปิดเฉพาะเครื่องที่ใช้ในบทนั้น เช่น ช่วง Lab 01-08 อาจใช้ `devops-control` และ `app-server` ก่อน แล้วค่อยเพิ่ม `monitor-server` และ Kubernetes node ในบทที่เกี่ยวข้อง

ตัวอย่างที่ควรหลีกเลี่ยง:

```text
ผิด:  เปิด VM ทั้งหมดพร้อมกันบนเครื่อง RAM 16 GB แล้วทุกเครื่องได้ RAM ต่ำมาก
ผล:   GitLab ช้า, Kubernetes ไม่เสถียร, disk swap หนัก

ถูก:  เปิดเฉพาะ VM ที่ใช้ในบทปัจจุบัน และ snapshot ก่อนเริ่มบทใหญ่
ผล:   debug ง่ายกว่าและเครื่องหลักไม่ทำงานหนักเกินไป
```

## Network Plan

แนะนำใช้ VMware NAT หรือ Host-only แล้วกำหนด Static IP

| Hostname | IP ตัวอย่าง | หน้าที่ |
|---|---|---|
| devops-control | 192.168.56.10 | GitLab, Registry |
| app-server | 192.168.56.20 | App, Docker |
| monitor-server | 192.168.56.30 | Prometheus, Grafana, Loki |
| k8s-master-01 | 192.168.56.100 | K8s Master |
| k8s-worker-01 | 192.168.56.101 | K8s Worker |
| k8s-worker-02 | 192.168.56.102 | K8s Worker |

สิ่งสำคัญคือทุก VM ต้องมองเห็นกันด้วย IP และ hostname ที่คงที่ ถ้าใช้ DHCP อย่างเดียว IP อาจเปลี่ยนหลัง reboot ทำให้ GitLab Runner, registry, Kubernetes join command หรือ Prometheus target ชี้ผิดเครื่อง

### NAT หรือ Host-only เลือกแบบไหนดี

`NAT` เหมาะถ้าต้องการให้ VM ออก internet ได้ง่าย เช่น apt install package, download Docker image หรือดึง manifest จาก internet แต่เครื่องอื่นใน network บ้านอาจเข้าถึง VM ได้ยากกว่า ขึ้นกับ VMware NAT configuration

`Host-only` เหมาะสำหรับ lab ที่ต้องการ network แยกและนิ่งระหว่าง host กับ VM โดยไม่รบกวน network บ้าน แต่ถ้าต้องออก internet อาจต้องเพิ่ม adapter อีกตัวหรือ config routing เพิ่ม

สำหรับ lab นี้ วิธีที่ง่ายคือใช้ NAT หรือ Host-only ให้ทุก VM อยู่ subnet เดียวกัน และกำหนด static IP เองให้ชัดเจน ถ้าต้องใช้ internet บ่อย ให้ตรวจตั้งแต่ต้นว่า VM ทุกเครื่อง `apt update` ได้

ตัวอย่างการวาง IP ที่ผิด:

```text
devops-control  192.168.56.10
app-server      192.168.1.20
monitor-server  10.0.0.30
```

สามเครื่องนี้อยู่คนละ subnet กัน ถ้าไม่มี routing เพิ่มจะติดต่อกันไม่ได้ ทำให้ SSH, registry, Prometheus หรือ Kubernetes มีปัญหา

ตัวอย่างการวาง IP ที่ถูก:

```text
devops-control  192.168.56.10
app-server      192.168.56.20
monitor-server  192.168.56.30
k8s-master-01   192.168.56.100
k8s-worker-01   192.168.56.101
k8s-worker-02   192.168.56.102
```

ทุกเครื่องอยู่ subnet เดียวกัน อ่านง่าย และแบ่งช่วง IP ตามหน้าที่ เช่น server ทั่วไปอยู่ช่วง `.10-.30` ส่วน Kubernetes อยู่ช่วง `.100+`

### สิ่งที่ต้องตรวจหลังตั้ง network

หลังตั้ง IP แล้วควรตรวจอย่างน้อย:

```bash
ip a
ip route
ping -c 4 devops-control
ping -c 4 app-server
ssh devops@app-server
```

ถ้า ping ด้วย IP ได้แต่ ping ด้วย hostname ไม่ได้ แปลว่ายังมีปัญหาที่ DNS หรือ `/etc/hosts` ถ้า ping ไม่ได้เลยให้ตรวจ IP, subnet, gateway, VMware network mode และ firewall ก่อน

## OS ที่ใช้

```text
Ubuntu Server 22.04 LTS หรือ 24.04 LTS
```

ควรใช้ Ubuntu Server รุ่น LTS เพราะ package, document และตัวอย่างคำสั่งของเครื่องมือ DevOps ส่วนใหญ่รองรับดี รุ่น desktop ใช้ได้แต่ไม่จำเป็น และกิน resource มากกว่าโดยไม่ได้ช่วยให้เข้าใจ server operation เพิ่มขึ้น

ข้อควรระวังคือบางคำสั่งติดตั้ง Kubernetes หรือ Docker อาจต่างกันเล็กน้อยตาม version ของ Ubuntu และ repository ที่ใช้ ถ้าเลือก 24.04 LTS แล้วเจอ package บางตัวไม่ตรงกับตัวอย่าง ให้ตรวจเอกสารของเครื่องมือนั้นประกอบ แต่สำหรับแนวคิดของ Lab ยังเหมือนเดิม

## User ที่ใช้ใน Lab

```text
devops
```

ให้ user นี้มีสิทธิ์ sudo

การใช้ user ชื่อเดียวกันทุกเครื่องช่วยลดความสับสนเวลารันคำสั่ง SSH, copy file หรือสร้าง service ที่อ้าง path เช่น `/home/devops/...` ถ้าแต่ละเครื่องใช้ user คนละชื่อ คำสั่งตัวอย่างจำนวนมากจะต้องแก้ path ตามเครื่องนั้น ๆ

ตัวอย่างที่ผิด:

```text
devops-control ใช้ user admin
app-server ใช้ user devops
k8s-master-01 ใช้ user ubuntu
```

ผลคือคำสั่งที่อ้าง `/home/devops` จะใช้ไม่ได้ทุกเครื่อง และเวลาเขียน systemd service หรือ pipeline deploy จะสับสนง่าย

ตัวอย่างที่ถูก:

```text
ทุก VM ใช้ user: devops
ทุก VM ให้ user devops ใช้ sudo ได้
```

ตรวจสอบว่า user มี sudo:

```bash
whoami
sudo -v
sudo hostnamectl
```

ถ้า `sudo -v` ไม่ผ่าน ให้แก้สิทธิ์ก่อนเริ่ม lab ถัดไป เพราะเกือบทุกบทต้องติดตั้ง package, แก้ service, เปิด firewall หรือแก้ config ระบบ

---
