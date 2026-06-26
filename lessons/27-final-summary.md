# สรุปสุดท้าย

DevOps คือการเข้าใจ flow ทั้งระบบ ไม่ใช่แค่จำ command หรือเครื่องมือ:

```text
Code
-> Build
-> Test
-> Package as Image
-> Push Registry
-> Deploy
-> Monitor
-> Log
-> Secure
-> Backup
-> Restore
```

Lab นี้เริ่มจากพื้นฐานที่ดูเรียบง่าย เช่น VM, Linux, SSH, IP, DNS, port และ firewall เพราะสิ่งเหล่านี้คือฐานของทุกเครื่องมือ หลังจากนั้นจึงค่อยต่อยอดไป Docker, Registry, Git Flow, CI/CD, Terraform, Monitoring, Logging, Kubernetes, Helm, Security และ Backup/Restore

## สิ่งที่ควรเข้าใจหลังทำครบ

คุณควรตอบคำถามเหล่านี้ได้โดยไม่ต้องเดา:

- app รันอยู่ที่ไหน
- image ถูก build จาก commit ไหน
- image ถูกเก็บที่ registry ใด
- pipeline มี stage อะไรบ้าง
- deploy ไป environment ไหน
- ถ้า app เข้าไม่ได้จะตรวจ network, service, log และ config อย่างไร
- ถ้า pod pull image ไม่ได้จะตรวจ registry และ node อย่างไร
- ถ้า release พังจะ rollback อย่างไร
- ถ้าข้อมูลหายจะ restore อย่างไร

ถ้าตอบคำถามเหล่านี้ได้ แปลว่าคุณไม่ได้แค่รัน lab ผ่าน แต่เริ่มเห็นระบบเป็น flow ที่ตรวจสอบและกู้คืนได้

## วิธีคิดที่ควรติดตัว

อย่าเริ่มแก้ปัญหาด้วยการสุ่มคำสั่ง ให้ไล่ตรวจเป็นชั้น:

```text
เครื่องถูกไหม
-> network ถึงไหม
-> DNS/hosts ถูกไหม
-> service รันไหม
-> port listen ไหม
-> firewall block ไหม
-> config/env ถูกไหม
-> log บอกอะไร
-> dependency พร้อมไหม
```

แนวคิดนี้ใช้ได้ตั้งแต่ SSH เข้าไม่ได้ ไปจนถึง Kubernetes pod ไม่ขึ้นหรือ pipeline deploy fail

## เครื่องมือไม่ใช่เป้าหมาย

Docker, Kubernetes, Helm, Prometheus, Loki, GitLab CI และ Terraform เป็นเครื่องมือ แต่เป้าหมายจริงคือระบบที่:

- build ได้ซ้ำ
- test ได้ซ้ำ
- deploy ได้ซ้ำ
- monitor ได้
- debug ได้
- rollback ได้
- restore ได้
- ตรวจสอบย้อนหลังได้

ถ้าเครื่องมือเยอะขึ้นแต่ระบบ debug หรือ restore ไม่ได้ ยังไม่ถือว่าดีขึ้นจริง

## เส้นทางต่อไป

หลังจบ Lab นี้ ให้เลือกต่อยอดจากโจทย์จริง:

```text
อยากลด manual work      -> Ansible, CI/CD ขั้นสูง
อยาก deploy เป็นระบบ   -> Helm, Argo CD, GitOps
อยากดูแล Kubernetes    -> cert-manager, MetalLB, External Secrets
อยากทำ observability   -> Alertmanager, OpenTelemetry
อยากทำ DR จริงจัง      -> Velero, backup automation
อยากเพิ่ม security     -> policy, RBAC, secret management, image scanning
```

อย่าเรียนทุกอย่างพร้อมกัน ให้เลือกปัญหาหนึ่งข้อแล้วทำ mini project ให้จบ เช่น “ทำให้ app มี HTTPS”, “ทำให้ rollback ด้วย Helm ได้”, “ทำให้ alert เมื่อ disk ใกล้เต็ม”, หรือ “ทำให้ restore namespace ได้”

## สรุป

ถ้าทำ Lab นี้ครบ คุณจะมีพื้นฐานพร้อมสำหรับสาย DevOps Engineer, Cloud Engineer, Platform Engineer หรือ SRE โดยเฉพาะเมื่อมีพื้นฐาน Fullstack Developer อยู่แล้ว

พื้นฐานสำคัญที่สุดที่ควรเก็บไว้คือการมองระบบแบบ end-to-end:

```text
source code
-> artifact/image
-> environment
-> runtime
-> traffic
-> telemetry
-> security
-> recovery
```

เมื่อเข้าใจภาพนี้ คุณจะเรียนเครื่องมือใหม่ได้เร็วขึ้น เพราะรู้ว่าเครื่องมือนั้นเข้ามาแก้ปัญหาชั้นใดของระบบ

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [แนวทางเรียนต่อ](./26-next-steps.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: ไม่มี

<!-- lesson-nav:end -->
