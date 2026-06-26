# DevOps Lab ภาษาไทย สำหรับจำลองบน VMware ที่บ้าน

เอกสารนี้ออกแบบสำหรับคนที่มีพื้นฐาน Fullstack Developer แล้ว ต้องการฝึก DevOps แบบลงมือทำจริงบน PC ที่บ้านด้วย VMware โดยครอบคลุมการติดตั้งเครื่องมือ วิธีการใช้งาน คำศัพท์ การทดสอบ การตรวจสอบ และข้อสรุปของแต่ละหัวข้อ

## ภาพรวมของเอกสาร

Lab ชุดนี้ไม่ได้เป็นแค่รายการคำสั่งสำหรับติดตั้งเครื่องมือ แต่ตั้งใจให้เห็นภาพการทำงาน DevOps ทั้งเส้นทาง ตั้งแต่เตรียม Linux server, ตั้ง network, deploy application, package เป็น container, build pipeline, deploy เข้า Kubernetes, เฝ้าดูระบบด้วย monitoring/logging, ตรวจ security และเตรียม backup/restore

แนวคิดหลักคือทำให้เข้าใจว่า code หนึ่งชุดเดินทางจากเครื่อง developer ไปเป็นระบบที่รันอยู่บน server ได้อย่างไร และเมื่อระบบมีปัญหาควรตรวจจากจุดไหนก่อน ไม่ใช่จำคำสั่งแยกเป็นท่อน ๆ

## เหมาะกับใคร

- Fullstack Developer ที่อยากเข้าใจงานหลังบ้านของการ deploy และ operate ระบบ
- คนที่เคยใช้ Docker หรือ Linux มาบ้าง แต่อยากเรียงความรู้ให้เป็นระบบ
- คนที่อยากลอง GitLab CI/CD, Monitoring, Logging หรือ Kubernetes ในเครื่องตัวเอง
- คนที่ต้องการ lab ส่วนตัวสำหรับฝึก troubleshooting โดยไม่ต้องใช้ cloud ตั้งแต่แรก

ถ้ายังไม่ชำนาญ Linux มาก สามารถทำตามได้ แต่ควรช้าลงในบทแรก ๆ เพราะพื้นฐานเรื่อง SSH, service, log, port และ firewall จะถูกใช้ซ้ำตลอดทั้งเอกสาร

## วิธีเรียนที่แนะนำ

ให้ทำตามลำดับบท เพราะแต่ละช่วงจะวางพื้นฐานให้ช่วงถัดไป เช่น ก่อนเข้า Kubernetes ควรเข้าใจ Linux network, Docker image, private registry และการตรวจ log มาก่อน ไม่อย่างนั้นเวลาพบ error อย่าง `ImagePullBackOff`, `CrashLoopBackOff` หรือ node `NotReady` จะไม่รู้ว่าปัญหาอยู่ที่ application, network, registry หรือ cluster

ลำดับการเรียนโดยภาพรวมคือ:

```text
เตรียม VM และ Linux
-> ตรวจ network และ service
-> deploy app แบบ manual
-> ย้าย app เข้า Docker
-> ใช้ Registry และ GitLab CI/CD
-> เพิ่ม Monitoring และ Logging
-> สร้าง Kubernetes cluster
-> deploy ด้วย Manifest และ Helm
-> เพิ่ม Security scan และ Backup/Restore
```

เวลาทำแต่ละบท อย่าดูแค่คำสั่งว่ารันผ่านหรือไม่ผ่าน ให้ดูด้วยว่าคำสั่งนั้นเปลี่ยนอะไรในระบบ เช่น เปิด port อะไร, สร้าง service ชื่ออะไร, image ถูก tag ยังไง, config ถูก mount เข้า container หรือ pod ตรงไหน และ log ไปอยู่ที่ใด

## ข้อผิดพลาดที่พบบ่อยตอนเรียน

ข้อผิดพลาดแรกคือข้ามบทพื้นฐานแล้วเริ่มที่ Kubernetes ทันที วิธีนี้ทำให้ lab ดูเหมือนเดินเร็ว แต่จะติดตอน debug เพราะ Kubernetes ซ่อนรายละเอียดหลายชั้นไว้ข้างหลัง เช่น container runtime, DNS, service discovery, image registry และ network plugin

อีกข้อผิดพลาดคือ copy คำสั่งโดยไม่ปรับค่าตามเครื่องตัวเอง เช่น hostname, interface name, IP address หรือ path ของ user ถ้าคำสั่งตัวอย่างใช้ `ens33` แต่ VM ของคุณใช้ `ens160` แล้วนำไปใช้ตรง ๆ network จะไม่ขึ้น หรือถ้า IP ซ้ำกับเครื่องอื่นจะทำให้ SSH และ service เรียกผิดเครื่อง

ตัวอย่างที่ควรระวัง:

```text
ผิด:  ใช้ IP จากตัวอย่างทุกเครื่องโดยไม่ดู network จริงของ VMware
ถูก:  ตรวจ subnet ของ VMware ก่อน แล้วกำหนด IP ที่ไม่ชนกับเครื่องอื่น

ผิด:  รันคำสั่งติดตั้งแล้วข้ามส่วนตรวจสอบ
ถูก:  รันคำสั่งตรวจ เช่น systemctl status, curl, docker ps, kubectl get pods ทุกครั้ง

ผิด:  แก้ปัญหาด้วยการลงใหม่ทันทีเมื่อเจอ error
ถูก:  อ่าน log และตรวจทีละชั้นก่อน เช่น service, port, firewall, DNS, config
```

## สิ่งที่ควรจดระหว่างทำ Lab

ให้ทำไฟล์ note ส่วนตัวไว้คู่กับ lab เพราะค่าบางอย่างจะถูกใช้ซ้ำหลายบท:

- IP และ hostname ของ VM ทุกเครื่อง
- username ที่ใช้ SSH และ path home directory
- network mode ของ VMware ที่เลือกใช้
- port สำคัญ เช่น 22, 80, 443, 3000, 5000, 9090, 3000 ของ Grafana, 6443 ของ Kubernetes API
- image name และ tag ที่ build
- URL ของ GitLab, Registry, Grafana และ application
- token, password หรือ secret ที่สร้างไว้ โดยเก็บนอก git repository

การจดค่าเหล่านี้ช่วยลดปัญหา “จำไม่ได้ว่าเคยตั้งอะไรไว้” ซึ่งเป็นสาเหตุที่ทำให้ lab สับสนมากกว่าตัวเครื่องมือเอง

## วิธีตรวจว่าทำถูกทาง

หลังจบแต่ละบท ควรตอบคำถามเหล่านี้ได้:

- บทนี้เพิ่ม component อะไรเข้าไปในระบบ
- component นั้นรันอยู่ที่เครื่องไหน
- เปิด port อะไร และใครเป็นคนเรียกใช้งาน
- ถ้า component นี้หยุดทำงาน จะตรวจ log จากที่ไหน
- ถ้าต้องลบแล้วสร้างใหม่ ต้องใช้คำสั่งหรือไฟล์ config ใด

ถ้าตอบไม่ได้ ให้ย้อนกลับไปอ่านส่วนตรวจสอบของบทนั้นก่อนเริ่มบทถัดไป เพราะบทหลัง ๆ จะใช้สิ่งที่ทำไว้ก่อนหน้าเป็นฐาน

## Checklist ก่อนเริ่ม Lab

- [ ] มีเครื่องที่สามารถรัน VMware Workstation หรือ VMware Player ได้
- [ ] มี ISO ของ Ubuntu Server 22.04 LTS หรือ 24.04 LTS
- [ ] เข้าใจว่าจะใช้ network แบบ NAT หรือ Host-only
- [ ] เตรียมแผน IP และ hostname สำหรับ VM แต่ละเครื่อง
- [ ] มีพื้นฐาน command line และ Git เบื้องต้น หรือพร้อมเปิดเอกสารอ้างอิงระหว่างทำ
- [ ] พร้อมจดค่า config ที่ใช้จริงระหว่างทำ Lab
- [ ] พร้อมตรวจ output และ log ทุกครั้ง ไม่ใช่ดูแค่ว่าคำสั่งรันจบ

---

## สารบัญ

1. ภาพรวม Lab
2. สเปกเครื่องและ Network ที่แนะนำ
3. คำศัพท์ DevOps สำคัญ
4. Lab 01: เตรียม VM และ Ubuntu Server
5. Lab 02: Linux, SSH, Service และ Log
6. Lab 03: Network, DNS, Port และ Firewall
7. Lab 04: Nginx Reverse Proxy และ Manual Deploy
8. Lab 05: Docker พื้นฐาน
9. Lab 06: Docker Compose สำหรับ Fullstack App
10. Lab 07: Private Container Registry
11. Lab 08: Git Flow และ Release Version
12. Lab 09: CI/CD ด้วย GitLab CE และ GitLab Runner
13. Lab 10: Terraform และ Infrastructure as Code
14. Lab 11: Monitoring ด้วย Prometheus และ Grafana
15. Lab 12: Logging ด้วย Loki และ Promtail
16. Lab 13: Kubernetes Cluster ด้วย kubeadm
17. Lab 14: Deploy App เข้า Kubernetes
18. Lab 15: Ingress, ConfigMap, Secret และ Storage
19. Lab 16: Helm Chart
20. Lab 17: DevSecOps ด้วย Trivy
21. Lab 18: Backup, Restore และ DR พื้นฐาน
22. Final Project
23. Checklist หลังเรียนจบ
24. Troubleshooting รวม

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: ไม่มี
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [1. ภาพรวม Lab](./01-overview.md)

<!-- lesson-nav:end -->
