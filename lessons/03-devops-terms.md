# 3. คำศัพท์ DevOps สำคัญ

บทนี้รวมคำศัพท์ที่เจอบ่อยใน Lab ทั้งชุด จุดประสงค์ไม่ใช่ให้จำคำจำกัดความแบบท่องสอบ แต่ให้เข้าใจว่าคำเหล่านี้สัมพันธ์กับงานจริงตรงไหน เวลาอ่าน log, pipeline หรือ Kubernetes manifest จะได้รู้ว่ากำลังพูดถึงส่วนใดของระบบ

| คำศัพท์ | ความหมาย |
|---|---|
| DevOps | แนวทางเชื่อม Development และ Operations เพื่อให้ build/test/deploy/monitor ทำได้เร็วและเสถียร |
| CI | Continuous Integration การ build และ test อัตโนมัติเมื่อมี code ใหม่ |
| CD | Continuous Delivery/Deployment การส่งมอบหรือ deploy อัตโนมัติ |
| Container | environment สำหรับรัน app ที่รวม dependency ไว้ในหน่วยเดียว |
| Image | template สำหรับสร้าง container |
| Registry | ที่เก็บ container image |
| IaC | Infrastructure as Code การจัดการ infra ด้วย code |
| Metrics | ตัวเลขวัดสถานะระบบ เช่น CPU, RAM, Request/sec |
| Log | บันทึกเหตุการณ์ของระบบ |
| Alert | การแจ้งเตือนเมื่อระบบผิดปกติ |
| Kubernetes | ระบบจัดการ container จำนวนมาก |
| Helm | package manager สำหรับ Kubernetes |
| Secret | ข้อมูลลับ เช่น password, token, key |
| Rollback | การย้อน version เมื่อ release มีปัญหา |
| VM | Virtual Machine เครื่องจำลองที่รัน OS แยกจากเครื่องหลัก |
| Hostname | ชื่อเครื่อง ใช้เรียก server แทนการจำ IP |
| Static IP | IP ที่กำหนดคงที่ ไม่เปลี่ยนหลัง reboot |
| SSH | วิธี remote เข้า server ผ่าน command line อย่างปลอดภัย |
| Service | โปรแกรมที่รันเป็น background และมักถูก systemd จัดการ |
| Process | โปรแกรมหรือคำสั่งที่กำลังทำงานอยู่ในระบบ |
| Port | หมายเลขช่องทางที่ service เปิดรับ connection |
| Firewall | ระบบควบคุม traffic เข้าออกเครื่องหรือ network |
| Reverse Proxy | server ที่รับ request จากภายนอกแล้วส่งต่อไป backend |
| Upstream | backend ปลายทางที่ reverse proxy ส่ง request ไปหา |
| Health Check | endpoint หรือคำสั่งที่ใช้ตรวจว่า service ยังทำงานปกติ |
| Volume | พื้นที่เก็บข้อมูลที่อยู่รอดแม้ container ถูกลบ |
| Docker Network | network ที่ Docker ใช้เชื่อม container เข้าด้วยกัน |
| Tag | ชื่อ version ของ image หรือ Git commit เช่น `1.0.0` |
| Runner | เครื่องหรือ process ที่รับงานจาก CI/CD pipeline ไปรัน |
| Pipeline | ชุดขั้นตอนอัตโนมัติ เช่น test, build, scan, deploy |
| Stage | กลุ่มงานใน pipeline เช่น test, build, deploy |
| Job | งานย่อยใน pipeline ที่รันคำสั่งจริง |
| Artifact | ไฟล์ผลลัพธ์จาก pipeline เช่น build output หรือ test report |
| State | ข้อมูลสถานะที่เครื่องมือ IaC ใช้จำว่า infra ปัจจุบันเป็นอย่างไร |
| Scrape | การที่ Prometheus ไปดึง metrics จาก target |
| Dashboard | หน้ารวมกราฟและตัวเลขเพื่อดูสถานะระบบ |
| Query | คำสั่งค้นหาข้อมูล เช่น query log หรือ metrics |
| Control Plane | ส่วนควบคุม Kubernetes cluster |
| Worker Node | เครื่องใน Kubernetes ที่ใช้รัน workload |
| Pod | หน่วยเล็กสุดที่ Kubernetes ใช้รัน container |
| Deployment | resource ที่คุมจำนวน pod และ rollout version |
| Service Kubernetes | resource ที่ให้ชื่อและ endpoint คงที่สำหรับเรียก pod |
| Ingress | resource สำหรับรับ traffic จากภายนอกเข้า service ใน cluster |
| ConfigMap | resource สำหรับเก็บ config ที่ไม่ลับใน Kubernetes |
| PersistentVolume | storage จริงที่ Kubernetes ใช้เก็บข้อมูลถาวร |
| PersistentVolumeClaim | คำขอใช้ storage จาก pod หรือ workload |
| Namespace | พื้นที่แยก resource ภายใน Kubernetes cluster |
| Manifest | ไฟล์ YAML ที่อธิบาย resource ที่ต้องการสร้างใน Kubernetes |
| Chart | package ของ Helm ที่รวม template และ values สำหรับ deploy |
| Vulnerability | ช่องโหว่ด้าน security |
| CVE | รหัสมาตรฐานสำหรับอ้างอิงช่องโหว่สาธารณะ |
| Severity | ระดับความรุนแรงของปัญหา เช่น LOW, HIGH, CRITICAL |
| Backup | การสำรองข้อมูลหรือ config เพื่อใช้กู้คืน |
| Restore | การนำข้อมูลหรือ config จาก backup กลับมาใช้งาน |
| RPO | ระยะข้อมูลย้อนหลังที่ยอมเสียได้เมื่อเกิดเหตุ |
| RTO | เวลาสูงสุดที่ยอมให้ระบบล่มก่อนกู้กลับมา |
| DR | Disaster Recovery แผนกู้ระบบเมื่อเกิดเหตุร้ายแรง |

## วิธีมองคำศัพท์เหล่านี้ใน Lab

คำศัพท์ DevOps จะเข้าใจง่ายขึ้นถ้ามองเป็นเส้นทางของ application:

```text
Developer เขียน code
-> CI ตรวจและ build
-> Image ถูกสร้างจาก Dockerfile
-> Image ถูก push เข้า Registry
-> CD deploy image ไปยัง server หรือ Kubernetes
-> Metrics และ Log ใช้ตรวจสถานะ
-> Alert แจ้งเมื่อมีปัญหา
-> Rollback ใช้ย้อนกลับเมื่อ release เสีย
```

ถ้าเข้าใจ flow นี้ คำศัพท์จะไม่ลอย เช่น `Registry` ไม่ใช่แค่ที่เก็บ image แต่เป็นจุดกลางที่ pipeline และ Kubernetes ต้องเข้าถึงได้ ถ้า registry ล่มหรือ network ไปไม่ถึง deployment ใหม่จะดึง image ไม่ได้

## คำที่มักสับสน

### VM, Host และ Container

`VM` คือเครื่องจำลองที่มี OS ของตัวเอง เช่น Ubuntu Server บน VMware ส่วน `Host` ในบริบท lab นี้มักหมายถึงเครื่อง PC จริงที่รัน VMware หรือชื่อเครื่องใน network แล้วแต่ประโยค ส่วน `Container` ไม่ใช่ VM เต็มเครื่อง แต่เป็น process ที่ถูกแยก environment สำหรับรัน application

จำง่าย ๆ:

```text
PC จริง       = เครื่อง host ที่เปิด VMware
VM            = server จำลอง เช่น app-server
Container     = process ของ app ที่รันใน VM
```

ถ้าสับสนสามคำนี้ เวลา debug จะหลงชั้น เช่น app ใน container เข้า database ไม่ได้ แต่ไปแก้ network adapter ของ VMware ทั้งที่ปัญหาอาจอยู่ที่ Docker network หรือ env variable

### Service, Process และ Port

`Process` คือสิ่งที่กำลังรันอยู่ ส่วน `Service` คือ process ที่ถูกจัดการแบบ background เช่น start/stop/restart ด้วย systemd และ `Port` คือช่องทางที่ service เปิดให้เครื่องอื่นเชื่อมต่อ

ตัวอย่าง:

```text
Nginx เป็น service
nginx worker เป็น process
port 80 คือช่องทางที่ Nginx รับ HTTP
```

เวลา service เข้าไม่ได้ อย่าดูแค่ว่า process มีหรือไม่มี ให้ตรวจด้วยว่า service listen port ถูกต้องไหม และ firewall อนุญาต port นั้นหรือเปล่า

### CI กับ CD

`CI` เน้นการรวม code แล้วตรวจว่า code ยัง build/test ผ่านหรือไม่ ตัวอย่างใน Lab คือ GitLab Runner รัน test หรือ build Docker image เมื่อ push code

`CD` เน้นการนำผลลัพธ์ที่ผ่านแล้วไป deploy ต่อ เช่น deploy ไป `app-server` หรือ Kubernetes namespace

ตัวอย่างที่ผิด:

```text
คิดว่า pipeline ที่ build image อย่างเดียวคือ deploy แล้ว
```

ตัวอย่างที่ถูก:

```text
CI: test + build image
CD: push image + deploy ไป environment เป้าหมาย
```

บาง pipeline อาจรวม CI/CD ในไฟล์เดียวกัน เช่น `.gitlab-ci.yml` แต่ stage แต่ละช่วงทำหน้าที่ต่างกัน

### Image กับ Container

`Image` คือ template หรือ package ที่ยังไม่ได้รัน ส่วน `Container` คือ process ที่รันจาก image แล้ว

จำง่าย ๆ:

```text
Image     = แบบพิมพ์/แพ็กเกจ
Container = สิ่งที่กำลังรันจาก image
```

ถ้าแก้ code แล้ว build image ใหม่ แต่ container เดิมยังรัน image เก่าอยู่ app จะไม่เปลี่ยน ต้อง recreate container หรือ rollout deployment ใหม่

### Tag, Version และ Release

`Tag` เป็นชื่อกำกับ version เช่น image tag `simple-api:1.0.0` หรือ Git tag `v1.0.0` ส่วน `Release` คือ version ที่ถูกเตรียมส่งมอบหรือ deploy แล้ว

ตัวอย่างที่ผิด:

```text
ใช้ latest ทุก environment แล้วไม่รู้ว่า production รัน code version ไหน
```

ตัวอย่างที่ถูก:

```text
build image เป็น simple-api:1.0.0
deploy dev ด้วย 1.0.0
ถ้าผ่านจึง promote tag เดิมไป environment ถัดไป
```

การใช้ tag ชัดเจนทำให้ rollback และ troubleshooting ง่ายขึ้น เพราะรู้ว่าแต่ละ environment ใช้ version ใด

### Metrics กับ Log

`Metrics` คือค่าตัวเลขตามเวลา เช่น CPU, memory, request count, error rate เหมาะกับดูแนวโน้มและทำ alert

`Log` คือข้อความเหตุการณ์ เช่น app start, database connect fail, request error เหมาะกับดูรายละเอียดว่าปัญหาเกิดจากอะไร

ตัวอย่าง:

```text
Metrics บอกว่า error rate สูงขึ้น
Log บอกว่า error มาจาก database password ผิด
```

สองอย่างนี้ต้องใช้คู่กัน Monitoring ช่วยบอกว่า “มีปัญหา” ส่วน Logging ช่วยตอบว่า “เกิดอะไรขึ้น”

### Dashboard, Alert และ Query

`Dashboard` ใช้ดูภาพรวม เช่น CPU, memory, request rate หรือ error rate ส่วน `Alert` ใช้แจ้งเตือนเมื่อค่าบางอย่างเกินเงื่อนไข และ `Query` คือภาษาหรือคำสั่งที่ใช้ดึงข้อมูล เช่น PromQL ใน Prometheus หรือ LogQL ใน Loki

ตัวอย่างการใช้งานร่วมกัน:

```text
Query ดึง error rate
Dashboard แสดงกราฟ error rate
Alert แจ้งเมื่อ error rate สูงเกิน threshold
```

ถ้ามี dashboard แต่ไม่มี alert คุณต้องเปิดดูเองตลอด ถ้ามี alert แต่ query ผิด ก็อาจแจ้งเตือนผิดหรือไม่เตือนเมื่อมีปัญหาจริง

### Secret กับ Config

`Secret` ใช้กับข้อมูลลับ เช่น password, token, private key ส่วน config ทั่วไปเช่น `APP_ENV=dev` หรือ `LOG_LEVEL=debug` ไม่ควรเก็บเป็น Secret ถ้าไม่ได้ลับ

ตัวอย่างที่ผิด:

```text
commit DB_PASSWORD ลงใน repository
ใส่ token ลงใน Dockerfile
เขียน password ไว้ใน Kubernetes Deployment ตรง ๆ
```

ตัวอย่างที่ถูก:

```text
ใช้ .env เฉพาะในเครื่องและไม่ commit
ใช้ GitLab CI/CD variables สำหรับ pipeline
ใช้ Kubernetes Secret สำหรับค่า sensitive ใน cluster
```

### Rollback กับ Restore

`Rollback` คือย้อน version ของ application หรือ deployment เช่น deploy image `1.0.2` แล้วมีปัญหา จึงย้อนกลับไป `1.0.1`

`Restore` คือกู้คืนข้อมูลหรือ resource จาก backup เช่น database หายแล้วนำ backup กลับมา

สองคำนี้แก้ปัญหาคนละแบบ ถ้า release ใหม่มี bug ใช้ rollback แต่ถ้าข้อมูลเสียหรือถูกลบ ต้องใช้ restore

### Kubernetes Service กับ Linux Service

สองคำนี้ชื่อเหมือนกันแต่คนละเรื่อง:

```text
Linux Service      = background service บน server เช่น nginx หรือ ssh
Kubernetes Service = resource ที่ให้ endpoint คงที่สำหรับเรียก pod
```

ถ้าบอกว่า “service ไม่ทำงาน” ต้องดูบริบทว่าเป็น systemd service บน Linux หรือ Kubernetes Service ใน cluster วิธีตรวจจะต่างกัน:

```bash
sudo systemctl status nginx
kubectl get svc -n devops-lab
```

### Deployment, Pod และ Container

ใน Kubernetes `Deployment` เป็นตัวคุม desired state เช่นต้องการ pod 2 ตัว ส่วน `Pod` เป็นหน่วยรัน workload และข้างใน pod มี container หนึ่งตัวหรือมากกว่า

```text
Deployment -> สร้าง/คุม ReplicaSet -> สร้าง Pod -> รัน Container
```

ถ้าแก้ image ใน Deployment แล้ว Kubernetes จะ rollout pod ใหม่ให้ ถ้าเข้าไปแก้ container ด้วยมือโดยตรง การแก้จะหายเมื่อ pod ถูกสร้างใหม่

### ConfigMap, Secret และ Volume

`ConfigMap` ใช้เก็บค่าที่ไม่ลับ, `Secret` ใช้เก็บค่าลับ และ `Volume` ใช้ mount file หรือ storage เข้า container/pod

ตัวอย่าง:

```text
APP_ENV=lab           -> ConfigMap
DB_PASSWORD=secret    -> Secret
/var/lib/postgresql   -> Volume/PersistentVolume
```

การแยกให้ถูกช่วยให้ config เปลี่ยนง่ายขึ้น และลดโอกาสเผลอเก็บ secret ลง repository

### Backup, Restore, RPO และ RTO

`Backup` คือการสำรองข้อมูล ส่วน `Restore` คือการกู้กลับมา `RPO` ตอบคำถามว่า “ยอมเสียข้อมูลย้อนหลังได้เท่าไร” และ `RTO` ตอบว่า “ต้องกู้ระบบกลับมาในกี่นาทีหรือกี่ชั่วโมง”

ตัวอย่าง:

```text
RPO 1 ชั่วโมง = ยอมเสียข้อมูลล่าสุดได้ไม่เกิน 1 ชั่วโมง
RTO 30 นาที   = ต้องกู้ระบบให้กลับมาใช้ได้ภายใน 30 นาที
```

ถ้าไม่มี RPO/RTO จะออกแบบ backup ไม่ได้ชัดเจน เพราะไม่รู้ว่าต้อง backup ถี่แค่ไหนและต้อง restore เร็วระดับใด

## คำศัพท์กับเครื่องมือใน Lab

| คำศัพท์ | เจอในบทไหน | เครื่องมือ/ไฟล์ที่เกี่ยวข้อง |
|---|---|---|
| VM/Hostname/Static IP | Lab 01-03 | VMware, Netplan, `/etc/hosts` |
| SSH/Service/Process | Lab 01-04 | ssh, systemd, journalctl |
| Port/Firewall | Lab 03-04 | ss, UFW, curl |
| Reverse Proxy/Health Check | Lab 04 | Nginx, `/health` endpoint |
| CI/CD | Lab 09 | GitLab CE, GitLab Runner, `.gitlab-ci.yml` |
| Container/Image | Lab 05-06 | Dockerfile, Docker Compose |
| Volume/Docker Network | Lab 05-06 | Docker volume, Docker Compose network |
| Registry/Tag | Lab 07-09 | Docker Registry, image tag |
| Pipeline/Stage/Job/Artifact | Lab 09 | GitLab CI/CD |
| IaC/State | Lab 10 | Terraform, state file |
| Metrics/Scrape/Dashboard | Lab 11 | Prometheus, Grafana |
| Log/Query | Lab 12 | Loki, Promtail, journalctl |
| Kubernetes/Pod/Deployment/Service | Lab 13-15 | kubeadm, kubectl, YAML manifest |
| Ingress/ConfigMap/Secret/Storage | Lab 15 | Ingress, ConfigMap, Secret, PV/PVC |
| Helm | Lab 16 | Chart.yaml, values.yaml, templates |
| Rollback | Lab 14, Lab 16 | kubectl rollout undo, helm rollback |
| Vulnerability/CVE/Severity | Lab 17 | Trivy |
| Backup/Restore/RPO/RTO/DR | Lab 18 | pg_dump, psql, Kubernetes YAML backup |

## วิธีตรวจว่าเข้าใจคำศัพท์แล้ว

ลองอธิบายประโยคนี้ด้วยคำของตัวเอง:

```text
เมื่อ developer push code, CI จะ test และ build image จากนั้น push ไป registry แล้ว CD deploy image นั้นเข้า Kubernetes ถ้ามีปัญหาให้ดู metrics/log และ rollback ได้
```

ถ้าอธิบายได้ว่าแต่ละคำเกิดที่ขั้นตอนไหน ใช้เครื่องมืออะไร และตรวจสอบอย่างไร แปลว่าพร้อมไปต่อบทถัดไป

---
