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
| ReplicaSet | resource ที่คุมจำนวน pod ให้ตรงกับจำนวนที่ต้องการ มักถูก Deployment สร้างให้ |
| StatefulSet | resource สำหรับ workload ที่มี identity หรือ storage ต่อ pod เช่น database |
| DaemonSet | resource ที่ทำให้ pod รันบนทุก node หรือ node ที่เลือก เช่น log agent |
| Job | resource สำหรับงานที่รันให้จบ เช่น backup หรือ migration |
| CronJob | resource สำหรับตั้งเวลาสร้าง Job ตาม schedule |
| Service Kubernetes | resource ที่ให้ชื่อและ endpoint คงที่สำหรับเรียก pod |
| Ingress | resource สำหรับรับ traffic จากภายนอกเข้า service ใน cluster |
| NetworkPolicy | resource สำหรับกำหนดว่า pod ใดคุยกับ pod/service ใดได้บ้าง |
| ConfigMap | resource สำหรับเก็บ config ที่ไม่ลับใน Kubernetes |
| Secret Kubernetes | resource สำหรับเก็บข้อมูลลับใน Kubernetes เช่น password หรือ token |
| PersistentVolume | storage จริงที่ Kubernetes ใช้เก็บข้อมูลถาวร |
| PersistentVolumeClaim | คำขอใช้ storage จาก pod หรือ workload |
| Namespace | พื้นที่แยก resource ภายใน Kubernetes cluster |
| Node | เครื่อง worker หรือ control-plane ที่เป็นสมาชิกของ Kubernetes cluster |
| HPA | Horizontal Pod Autoscaler resource สำหรับเพิ่ม/ลดจำนวน replica ตาม metric |
| ResourceQuota | resource สำหรับจำกัดปริมาณ resource ที่ namespace ใช้ได้ |
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

### Kubernetes Resource แยกตามกลุ่ม

เวลาอ่าน manifest หรือใช้ `kubectl get` ให้มอง resource เป็นกลุ่ม จะเข้าใจง่ายกว่าจำชื่อแยกกัน Resource แต่ละกลุ่มตอบคำถามคนละแบบ:

```text
workload -> รันอะไร
network  -> เรียกถึงกันอย่างไร
config   -> ตั้งค่าอะไรให้ app
storage  -> เก็บข้อมูลไว้ที่ไหน
cluster  -> จัดการ cluster/namespace อย่างไร
```

#### Workload

กลุ่มนี้เกี่ยวกับการรันงานหรือ application ใน cluster

| ชื่อย่อ/ชื่อ | ชื่อเต็ม | ใช้ทำอะไร |
|---|---|---|
| pod | Pod | หน่วยเล็กสุดที่รัน container หนึ่งตัวหรือหลายตัว |
| deploy | Deployment | คุม stateless app, จำนวน replica, rollout และ rollback |
| rs | ReplicaSet | คุมจำนวน pod ให้ครบตามที่ต้องการ ปกติ Deployment จะสร้างให้เอง |
| sts | StatefulSet | ใช้กับ app ที่ต้องมีชื่อ pod คงที่หรือ storage ประจำตัว เช่น database |
| ds | DaemonSet | บังคับให้มี pod รันบนทุก node หรือ node ที่เลือก เช่น log collector, node agent |
| job | Job | รันงานครั้งเดียวให้สำเร็จ เช่น database migration, backup |
| cronjob | CronJob | สร้าง Job ตามเวลา เช่น backup ทุกคืน |

ตัวอย่างการดู:

```bash
kubectl get pod -n devops-lab
kubectl get deploy -n devops-lab
kubectl get rs -n devops-lab
kubectl get job,cronjob -n devops-lab
```

ใน Lab นี้จะใช้ `Deployment` เป็นหลัก เพราะ `simple-api` เป็น stateless app ที่ scale และ rollout ได้ง่าย

`pod` หรือ `Pod` คือหน่วยเล็กสุดที่ Kubernetes ใช้รัน workload ข้างใน pod อาจมี container หนึ่งตัวหรือหลายตัวที่แชร์ network namespace เดียวกัน เช่น ใช้ IP เดียวกันและคุยกันผ่าน `localhost` ได้ ปกติเราไม่ค่อยสร้าง pod เดี่ยว ๆ สำหรับ app จริง เพราะถ้า pod หาย Kubernetes จะไม่มี controller คอยสร้างกลับมาให้ ยกเว้นกรณี debug หรือทดสอบสั้น ๆ

ตัวอย่างการดู pod:

```bash
kubectl get pod -n devops-lab
kubectl describe pod <pod-name> -n devops-lab
kubectl logs <pod-name> -n devops-lab
```

เวลามีปัญหาให้ดู pod ก่อนเสมอ เพราะ pod เป็นจุดที่เห็นอาการจริง เช่น `Running`, `Pending`, `CrashLoopBackOff`, `ImagePullBackOff` หรือ probe fail

`deploy` หรือ `Deployment` คือ resource สำหรับรัน stateless application เช่น API, web app หรือ worker ที่แต่ละ replica แทนกันได้ Deployment คุมจำนวน replica, สร้าง ReplicaSet, rollout version ใหม่ และ rollback กลับ version เก่าได้ ถ้าแก้ image tag ใน Deployment Kubernetes จะค่อย ๆ สร้าง pod ชุดใหม่และลด pod ชุดเก่า

ตัวอย่างการดู Deployment:

```bash
kubectl get deploy -n devops-lab
kubectl describe deploy simple-api -n devops-lab
kubectl rollout status deployment/simple-api -n devops-lab
kubectl rollout history deployment/simple-api -n devops-lab
```

ใน Lab นี้ `simple-api` เหมาะกับ Deployment เพราะ app ไม่มี state ในตัว pod ถ้า pod ถูกลบ pod ใหม่สามารถรับงานแทนได้

`rs` หรือ `ReplicaSet` คือ resource ที่คุมให้จำนวน pod ตรงกับจำนวนที่ต้องการ เช่น ต้องมี pod 2 ตัว ถ้าหายไป 1 ตัว ReplicaSet จะสร้างกลับมา โดยทั่วไปเราไม่แก้ ReplicaSet โดยตรง เพราะ Deployment เป็นคนสร้างและจัดการ ReplicaSet ให้ตอน rollout แต่การดู ReplicaSet ช่วยให้เข้าใจว่า Deployment สร้าง pod version ไหนอยู่

ตัวอย่างการดู ReplicaSet:

```bash
kubectl get rs -n devops-lab
kubectl describe rs <rs-name> -n devops-lab
```

ถ้า rollout แล้วมี ReplicaSet เก่ากับใหม่พร้อมกัน เป็นเรื่องปกติ เพราะ Kubernetes เก็บ revision ไว้เพื่อ rollback

`sts` หรือ `StatefulSet` คือ resource สำหรับ workload ที่ต้องมี identity คงที่ เช่น pod ชื่อเดิม ลำดับเดิม และ storage ประจำตัว ตัวอย่างที่มักใช้ StatefulSet คือ database, message queue หรือระบบที่ node แต่ละตัวมีบทบาทเฉพาะ เช่น `postgres-0`, `postgres-1` จุดต่างจาก Deployment คือ pod ใน StatefulSet ไม่ได้ถูกมองว่าแทนกันได้ทั้งหมด

ตัวอย่างการดู StatefulSet:

```bash
kubectl get sts -n devops-lab
kubectl describe sts <sts-name> -n devops-lab
```

ถ้า app ต้องมีข้อมูลถาวรต่อ replica หรือจำเป็นต้องมีชื่อ pod คงที่ ให้เริ่มคิดถึง StatefulSet แต่ถ้าเป็น API stateless ปกติใช้ Deployment ง่ายกว่า

`ds` หรือ `DaemonSet` คือ resource ที่ทำให้ pod รันบนทุก node หรือบน node ที่เลือกตามเงื่อนไข เหมาะกับ agent ที่ต้องอยู่คู่กับ node เช่น log collector, monitoring agent, CNI component หรือ storage agent ตัวอย่างเช่น Promtail, node exporter หรือ network plugin บางตัวมักใช้แนวคิดแบบ DaemonSet

ตัวอย่างการดู DaemonSet:

```bash
kubectl get ds -A
kubectl describe ds <ds-name> -n <namespace>
```

ถ้ามี node 3 เครื่อง DaemonSet อาจสร้าง pod 3 ตัวโดยอัตโนมัติ เครื่องละ 1 ตัว ยกเว้นมี taint, nodeSelector หรือ affinity จำกัดไว้

`job` หรือ `Job` คือ resource สำหรับงานที่ต้องรันให้เสร็จแล้วจบ เช่น migration, import data, backup ครั้งเดียว หรือ batch task เมื่อ command ใน pod จบสำเร็จ Job จะถือว่า complete ไม่ต้องรันค้างเหมือน Deployment

ตัวอย่างการดู Job:

```bash
kubectl get job -n devops-lab
kubectl describe job <job-name> -n devops-lab
kubectl logs job/<job-name> -n devops-lab
```

ถ้างานต้องสำเร็จหนึ่งครั้งแล้วหยุด ใช้ Job ดีกว่าสร้าง Deployment ที่รันค้างโดยไม่จำเป็น

`cronjob` หรือ `CronJob` คือ resource ที่สร้าง Job ตามเวลา เช่น backup ทุกคืน, cleanup ทุกชั่วโมง หรือส่ง report ทุกวัน รูปแบบ schedule คล้าย cron บน Linux เช่น `0 2 * * *` หมายถึงรันทุกวันเวลา 02:00

ตัวอย่างการดู CronJob:

```bash
kubectl get cronjob -n devops-lab
kubectl describe cronjob <cronjob-name> -n devops-lab
kubectl get job -n devops-lab
```

ถ้า CronJob ไม่ทำงาน ให้ดู schedule, timezone ของ cluster, suspend flag และ Job ที่ถูกสร้างขึ้นมาว่าสำเร็จหรือ fail

#### Network

กลุ่มนี้เกี่ยวกับการให้ traffic วิ่งถึง pod อย่างถูกทาง

| ชื่อย่อ/ชื่อ | ชื่อเต็ม | ใช้ทำอะไร |
|---|---|---|
| svc | Service | ให้ endpoint คงที่สำหรับเรียก pod แม้ pod IP จะเปลี่ยน |
| ingress | Ingress | รับ traffic จากภายนอก cluster แล้ว route เข้า Service ตาม host/path |
| netpol | NetworkPolicy | จำกัด traffic เข้า/ออก pod ตาม label, namespace หรือ port |

ตัวอย่างการดู:

```bash
kubectl get svc -n devops-lab
kubectl get ingress -n devops-lab
kubectl get netpol -n devops-lab
```

ถ้า app เข้าไม่ได้ ให้ไล่จาก `Ingress -> Service -> Endpoints -> Pod` อย่าเริ่มแก้ Deployment ทันทีถ้ายังไม่รู้ว่า traffic ไปถึงชั้นไหน

`svc` หรือ `Service` คือ resource ที่ให้ชื่อและ endpoint คงที่สำหรับเรียก pod เพราะ pod IP เปลี่ยนได้ทุกครั้งที่ pod ถูกสร้างใหม่ Service ใช้ selector เลือก pod จาก label แล้วส่ง traffic ไปยัง pod ที่ match กัน ถ้า selector ไม่ตรงกับ label ของ pod Service จะมีอยู่แต่ไม่มี backend

ชนิดของ Service ที่พบบ่อย:

```text
ClusterIP    = เรียกได้ภายใน cluster
NodePort     = เปิด port บน node เพื่อให้ภายนอกเรียกได้
LoadBalancer = ขอ external load balancer จาก cloud provider หรือ MetalLB
```

ตัวอย่างการตรวจ Service:

```bash
kubectl get svc -n devops-lab
kubectl describe svc simple-api -n devops-lab
kubectl get endpoints simple-api -n devops-lab
```

ถ้า `endpoints` ว่าง แปลว่า Service หา pod ไม่เจอ ให้ตรวจ label/selector ก่อนตรวจ network ชั้นอื่น

`ingress` หรือ `Ingress` คือ resource ที่รับ HTTP/HTTPS traffic จากภายนอก cluster แล้ว route เข้า Service ตาม host หรือ path เช่น `simple-api.lab.local` ไปที่ Service `simple-api` การมี Ingress resource อย่างเดียวไม่พอ ต้องมี Ingress Controller เช่น Nginx Ingress Controller คอยอ่าน resource แล้วรับ traffic จริง

ตัวอย่างการตรวจ Ingress:

```bash
kubectl get ingress -n devops-lab
kubectl describe ingress simple-api -n devops-lab
curl -H "Host: simple-api.lab.local" http://<node-ip>
```

ถ้า Ingress เข้าไม่ได้ ให้ตรวจตามลำดับ: DNS/hosts ชี้ถูก IP, Ingress Controller รันอยู่, Ingress rule ถูก, Service มี endpoint และ pod ตอบ health ได้

`netpol` หรือ `NetworkPolicy` คือ resource สำหรับควบคุมว่า pod ใดรับ traffic จากใครได้ หรือออกไปหาใครได้บ้าง โดยอิง label, namespace, IP block และ port ใช้เพื่อลด blast radius เช่น ให้ backend รับ traffic เฉพาะจาก frontend หรือให้ database รับเฉพาะจาก backend

ตัวอย่างการดู NetworkPolicy:

```bash
kubectl get netpol -n devops-lab
kubectl describe netpol <netpol-name> -n devops-lab
```

ข้อควรจำคือ NetworkPolicy จะมีผลก็ต่อเมื่อ CNI ที่ใช้รองรับ policy เช่น Calico ถ้า CNI ไม่ enforce policy การสร้าง NetworkPolicy อาจไม่มีผลจริง

#### Config

กลุ่มนี้เกี่ยวกับค่าที่ส่งให้ application โดยไม่ต้องฝังไว้ใน image

| ชื่อย่อ/ชื่อ | ชื่อเต็ม | ใช้ทำอะไร |
|---|---|---|
| cm | ConfigMap | เก็บ config ที่ไม่ลับ เช่น `APP_ENV`, `LOG_LEVEL`, URL ภายใน |
| secret | Secret | เก็บข้อมูลลับ เช่น password, token, certificate key |

ตัวอย่างการดู:

```bash
kubectl get cm -n devops-lab
kubectl get secret -n devops-lab
kubectl describe cm simple-api-config -n devops-lab
```

ConfigMap และ Secret ช่วยให้ image เดียวกันใช้ได้หลาย environment โดยเปลี่ยนค่า runtime แทนการ build image ใหม่ทุกครั้ง

`cm` หรือ `ConfigMap` คือ resource สำหรับเก็บ config ที่ไม่ลับ เช่น environment name, log level, feature flag, URL ภายใน หรือ config file ที่ mount เข้า pod ได้ ConfigMap ช่วยให้ image เดียวกันใช้ได้หลาย environment โดยเปลี่ยน config ตอน deploy แทนการ build image ใหม่

ตัวอย่างการดู ConfigMap:

```bash
kubectl get cm -n devops-lab
kubectl describe cm simple-api-config -n devops-lab
kubectl get cm simple-api-config -n devops-lab -o yaml
```

ข้อมูลที่ไม่ควรใส่ใน ConfigMap คือ password, token, private key หรือข้อมูลที่ถ้าหลุดแล้วมีผลด้าน security

`secret` หรือ `Secret` คือ resource สำหรับเก็บข้อมูลลับ เช่น password, token, certificate หรือ registry credential ใน Kubernetes ค่าใน Secret มักถูก encode เป็น base64 ซึ่งไม่ใช่ encryption ดังนั้นห้ามเข้าใจผิดว่า Secret ปลอดภัยพอสำหรับทุกกรณี Production จริงควรควบคุม RBAC, encryption at rest และอาจใช้ Vault/External Secrets เพิ่ม

ตัวอย่างการดู Secret:

```bash
kubectl get secret -n devops-lab
kubectl describe secret simple-api-secret -n devops-lab
kubectl get secret simple-api-secret -n devops-lab -o yaml
```

ถ้าต้อง decode ค่าเพื่อ debug ใน lab:

```bash
kubectl get secret simple-api-secret -n devops-lab -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

ให้ใช้คำสั่ง decode เฉพาะตอนจำเป็น และอย่าเอาค่าลับไปใส่ใน README, screenshot หรือ git repository

#### Storage

กลุ่มนี้เกี่ยวกับข้อมูลที่ต้องอยู่รอดหลัง pod ถูกลบหรือสร้างใหม่

| ชื่อย่อ/ชื่อ | ชื่อเต็ม | ใช้ทำอะไร |
|---|---|---|
| pvc | PersistentVolumeClaim | คำขอใช้ storage จาก workload |
| pv | PersistentVolume | storage จริงที่ cluster เตรียมไว้หรือ provision ให้อัตโนมัติ |

ตัวอย่างการดู:

```bash
kubectl get pvc -n devops-lab
kubectl get pv
kubectl describe pvc <pvc-name> -n devops-lab
```

Pod เป็นของชั่วคราว แต่ข้อมูลบางอย่างไม่ควรชั่วคราว เช่น database data หรือ uploaded files จึงต้องใช้ storage ที่ออกแบบไว้ชัดเจน

`pvc` หรือ `PersistentVolumeClaim` คือคำขอใช้ storage จาก workload เช่น ขอ disk ขนาด 5Gi แบบ ReadWriteOnce แล้วนำไป mount เข้า pod PVC เป็นสิ่งที่ application อ้างถึงโดยตรง ส่วน Kubernetes จะจับคู่ PVC กับ PV ที่เหมาะสม หรือให้ storage class provision PV ให้อัตโนมัติ

ตัวอย่างการดู PVC:

```bash
kubectl get pvc -n devops-lab
kubectl describe pvc <pvc-name> -n devops-lab
```

ถ้า PVC อยู่สถานะ `Pending` ให้ตรวจว่ามี PV หรือ StorageClass ที่ตอบคำขอนั้นได้ไหม เช่น access mode, storage size และ storage class name ตรงกันหรือไม่

`pv` หรือ `PersistentVolume` คือ storage จริงที่ cluster ใช้ให้ pod เก็บข้อมูล อาจเป็น local path, NFS, cloud disk หรือ storage backend อื่น ๆ PV มี lifecycle แยกจาก pod ดังนั้น pod ลบแล้วข้อมูลอาจยังอยู่ตาม policy ที่ตั้งไว้ เช่น `Retain` หรือ `Delete`

ตัวอย่างการดู PV:

```bash
kubectl get pv
kubectl describe pv <pv-name>
```

ใน lab อาจใช้ storage แบบง่าย เช่น `hostPath` เพื่อเรียน concept แต่ production ควรใช้ storage backend ที่เหมาะกับการ recover, backup, snapshot และย้าย workload

#### Cluster

กลุ่มนี้เกี่ยวกับการจัดการขอบเขต, node และ resource ของ cluster

| ชื่อย่อ/ชื่อ | ชื่อเต็ม | ใช้ทำอะไร |
|---|---|---|
| node | Node | เครื่องใน cluster ที่รัน workload หรือ control plane |
| ns | Namespace | แยก resource เป็นกลุ่ม เช่น `devops-lab`, `dev`, `prod` |
| hpa | HorizontalPodAutoscaler | เพิ่ม/ลดจำนวน replica อัตโนมัติตาม metric เช่น CPU |
| quota | ResourceQuota | จำกัด resource ที่ namespace ใช้ได้ เช่น CPU, memory, จำนวน pod |

ตัวอย่างการดู:

```bash
kubectl get nodes
kubectl get ns
kubectl get hpa -n devops-lab
kubectl get quota -n devops-lab
```

Resource กลุ่มนี้ช่วยให้ cluster ไม่กลายเป็นพื้นที่รวมทุกอย่างแบบไร้ขอบเขต โดยเฉพาะเมื่อมีหลายทีม หลาย environment หรือ resource จำกัด

`node` หรือ `Node` คือเครื่องที่เป็นสมาชิกของ Kubernetes cluster อาจเป็น control plane หรือ worker node ก็ได้ Node มี kubelet และ container runtime เพื่อรัน pod และรายงานสถานะกลับไปที่ control plane ถ้า node `NotReady` pod บน node นั้นอาจมีปัญหาหรือถูก reschedule ไปที่อื่น

ตัวอย่างการดู Node:

```bash
kubectl get nodes -o wide
kubectl describe node k8s-worker-01
```

เวลา node มีปัญหาให้ตรวจทั้งฝั่ง Kubernetes และ Linux เช่น kubelet, containerd, network, disk และ memory

`ns` หรือ `Namespace` คือพื้นที่แยก resource ภายใน cluster ใช้แบ่ง environment, team หรือ lab เช่น `devops-lab`, `dev`, `prod` ชื่อ resource ส่วนใหญ่ซ้ำกันได้ถ้าอยู่คนละ namespace เช่นมี Service ชื่อ `api` ใน namespace `dev` และ `prod`

ตัวอย่างการดู Namespace:

```bash
kubectl get ns
kubectl get all -n devops-lab
```

Namespace ไม่ใช่ security boundary ที่สมบูรณ์ด้วยตัวเอง ถ้าต้องการแยกสิทธิ์หรือ network จริงต้องใช้ RBAC, NetworkPolicy และ quota ประกอบ

`hpa` หรือ `HorizontalPodAutoscaler` คือ resource สำหรับเพิ่มหรือลดจำนวน replica อัตโนมัติตาม metric เช่น CPU, memory หรือ custom metric HPA มักใช้กับ Deployment เพื่อ scale pod ตามโหลด

ตัวอย่างการดู HPA:

```bash
kubectl get hpa -n devops-lab
kubectl describe hpa <hpa-name> -n devops-lab
```

HPA ต้องมี metrics source เช่น Metrics Server ถ้าไม่มี metric ให้ดึง HPA จะคำนวณจำนวน replica ไม่ได้

`quota` หรือ `ResourceQuota` คือ resource สำหรับจำกัดการใช้ resource ใน namespace เช่น จำกัด CPU, memory, จำนวน pod, จำนวน PVC หรือจำนวน Service ใช้ป้องกันไม่ให้ namespace เดียวใช้ resource ของ cluster เกินควบคุม

ตัวอย่างการดู ResourceQuota:

```bash
kubectl get quota -n devops-lab
kubectl describe quota <quota-name> -n devops-lab
```

ถ้า apply workload แล้วเจอ error ว่าเกิน quota ให้ลด resource request/limit, ลดจำนวน replica หรือปรับ quota ให้เหมาะสม

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

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [2. สเปกเครื่องและ Network ที่แนะนำ](./02-recommended-specs-and-network.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [4. Lab 01: เตรียม VM และ Ubuntu Server](./04-lab-01-vm-and-ubuntu-server.md)

<!-- lesson-nav:end -->
