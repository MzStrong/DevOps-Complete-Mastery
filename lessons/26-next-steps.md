# แนวทางเรียนต่อ

หลังทำ Lab นี้จบ คุณจะมีพื้นฐานตั้งแต่ Linux, Docker, CI/CD, Monitoring, Logging, Kubernetes, Security และ Backup แล้ว ขั้นต่อไปควรเลือกเรียนตามเป้าหมายงาน ไม่จำเป็นต้องเรียนทุกเครื่องมือพร้อมกัน ให้เลือกเส้นทางที่ช่วยแก้ปัญหาจริงในระบบของคุณก่อน

## ระดับถัดไปควรเรียนอะไร

ถ้าต้องการยืนให้มั่นในระดับ Junior DevOps:

```text
Linux troubleshooting -> Docker/Compose -> GitLab CI/CD -> Registry -> Kubernetes basics -> Monitoring/Logging basics
```

เป้าหมายคือทำงานประจำวันได้ เช่น อ่าน pipeline log, แก้ service ไม่ขึ้น, ตรวจ image pull fail, ดู pod crash, เปิด dashboard และอธิบาย rollback path ได้

ถ้าต้องการขยับไป Mid-level DevOps:

```text
Terraform cloud resource -> GitOps -> Helm/Kustomize จริงจัง -> Secret management -> Alerting -> Backup automation
```

เป้าหมายคือออกแบบ flow deploy ที่ทำซ้ำได้, ลด manual step, แยก environment, จัดการ secret อย่างปลอดภัย และทำให้ระบบมี alert/restore ที่พิสูจน์ได้

ถ้าต้องการไปทาง Platform Engineer:

```text
Kubernetes operations -> RBAC -> NetworkPolicy -> Ingress/TLS -> cert-manager -> External Secrets -> Cluster upgrade -> Multi-environment platform
```

เป้าหมายคือดูแล platform ให้ทีมอื่นใช้ได้ ไม่ใช่แค่ deploy app ของตัวเอง ต้องคิดเรื่อง standard, permission, isolation, reliability และ developer experience

ถ้าต้องการไปทาง SRE:

```text
Prometheus -> Alertmanager -> Loki -> OpenTelemetry -> SLO/SLI -> Incident response -> Capacity planning
```

เป้าหมายคือวัดความน่าเชื่อถือของระบบได้จริง รู้ว่าเมื่อไรควร alert, ลด noise, วิเคราะห์ incident และปรับระบบจากข้อมูล ไม่ใช่จากความรู้สึก

ถ้าต้องการไปทาง DevSecOps:

```text
Trivy -> Secret scanning -> Dependency policy -> Image signing -> Admission controller -> RBAC audit -> Compliance evidence
```

เป้าหมายคือเอา security เข้าไปใน flow การ build/deploy โดยไม่ทำให้ทีม deploy ช้าจนใช้งานไม่ได้

## ถ้าอยากเก่ง Automation และ Server Management

### Ansible สำหรับ configuration management

Ansible ใช้จัดการ config บน server หลายเครื่อง เช่น ติดตั้ง package, สร้าง user, ตั้งค่า firewall, deploy config file หรือ restart service

เหมาะต่อยอดจาก:

```text
Lab 01-04: Linux, SSH, Service, Nginx
```

Mini project ที่ควรลอง:

```text
เขียน Ansible playbook เพื่อติดตั้ง Docker และตั้งค่า /etc/hosts ให้ทุก VM
```

สิ่งที่ควรเข้าใจ:

- inventory
- playbook
- task
- idempotency
- variables
- handlers

## ถ้าอยากทำ Registry ให้ใกล้ production

### Harbor Registry แทน registry:2

Harbor เป็น container registry ที่มี UI, authentication, project, vulnerability scanning และ policy ดีกว่า `registry:2` ที่ใช้ใน lab

เหมาะต่อยอดจาก:

```text
Lab 07: Private Container Registry
Lab 17: DevSecOps ด้วย Trivy
```

Mini project ที่ควรลอง:

```text
ติดตั้ง Harbor แล้วให้ GitLab CI push image เข้า Harbor แทน devops-control:5000
```

สิ่งที่ควรเข้าใจ:

- project/repository
- robot account
- image retention
- vulnerability scan
- registry authentication

## ถ้าอยากทำ GitOps

### Argo CD สำหรับ GitOps

Argo CD ทำให้ Kubernetes sync สถานะจาก Git repository แทนการ deploy ด้วยคำสั่ง manual หรือ pipeline ที่ ssh/kubectl เข้า cluster โดยตรง

เหมาะต่อยอดจาก:

```text
Lab 09: CI/CD
Lab 14-16: Kubernetes และ Helm
```

Mini project ที่ควรลอง:

```text
เก็บ Helm values ใน Git แล้วให้ Argo CD sync app เข้า namespace devops-lab
```

สิ่งที่ควรเข้าใจ:

- desired state
- sync
- drift detection
- app of apps
- Helm/Kustomize integration

## ถ้าอยากทำ HTTPS/TLS บน Kubernetes

### cert-manager สำหรับ TLS บน Kubernetes

cert-manager ช่วยออกและต่ออายุ certificate อัตโนมัติให้ Ingress หรือ service ใน Kubernetes

เหมาะต่อยอดจาก:

```text
Lab 15: Ingress
```

Mini project ที่ควรลอง:

```text
ตั้ง cert-manager แล้วทำ HTTPS ให้ simple-api.lab.local ด้วย self-signed issuer หรือ local CA
```

สิ่งที่ควรเข้าใจ:

- Certificate
- Issuer / ClusterIssuer
- TLS secret
- Ingress annotation
- renewal

## ถ้าอยากจัดการ Secret ให้จริงจัง

### External Secrets หรือ Vault สำหรับ secret management

Kubernetes Secret แบบพื้นฐานยังต้องระวังเรื่องการเก็บและการเข้าถึง External Secrets หรือ Vault ช่วยดึง secret จากระบบกลางเข้ามาใช้ใน cluster

เหมาะต่อยอดจาก:

```text
Lab 15: Secret
Lab 17: DevSecOps
```

Mini project ที่ควรลอง:

```text
เก็บ DB_PASSWORD ไว้ใน Vault หรือ secret backend แล้ว sync เข้า Kubernetes Secret
```

สิ่งที่ควรเข้าใจ:

- secret backend
- access policy
- service account
- secret rotation
- audit log

## ถ้าอยากทำ LoadBalancer ใน lab/bare metal

### MetalLB สำหรับ LoadBalancer บน bare metal lab

ใน cloud, Service type LoadBalancer จะได้ external IP จาก cloud provider แต่ใน lab บน VMware ไม่มี cloud load balancer MetalLB ช่วยแจก IP ให้ service type LoadBalancer ใน bare metal/lab environment

เหมาะต่อยอดจาก:

```text
Lab 13-15: Kubernetes, Service, Ingress
```

Mini project ที่ควรลอง:

```text
ติดตั้ง MetalLB แล้ว expose Nginx Ingress Controller ด้วย Service type LoadBalancer
```

สิ่งที่ควรเข้าใจ:

- IP address pool
- L2 mode
- Service type LoadBalancer
- ARP
- network subnet ของ lab

## ถ้าอยากทำ Alert จริง

### Alertmanager สำหรับ alert จริง

Prometheus เก็บ metrics และประเมิน rule ได้ ส่วน Alertmanager จัดการ routing, grouping, silence และส่งแจ้งเตือนไปช่องทางต่าง ๆ

เหมาะต่อยอดจาก:

```text
Lab 11: Monitoring ด้วย Prometheus และ Grafana
```

Mini project ที่ควรลอง:

```text
สร้าง alert เมื่อ node_exporter down หรือ disk เหลือน้อยกว่า threshold
```

สิ่งที่ควรเข้าใจ:

- alert rule
- threshold
- severity
- silence
- notification route

## ถ้าอยากทำ Observability ให้ครบขึ้น

### OpenTelemetry สำหรับ tracing

Metrics บอกตัวเลข, logs บอกเหตุการณ์, tracing บอก request เดินทางผ่าน service ไหนและใช้เวลาที่จุดใด

เหมาะต่อยอดจาก:

```text
Lab 11-12: Monitoring และ Logging
```

Mini project ที่ควรลอง:

```text
instrument backend API ด้วย OpenTelemetry แล้วส่ง trace ไป collector
```

สิ่งที่ควรเข้าใจ:

- trace
- span
- context propagation
- collector
- exporter

## ถ้าอยาก backup Kubernetes จริงจัง

### Velero สำหรับ backup Kubernetes

Velero ใช้ backup/restore Kubernetes resource และ persistent volume snapshot ตาม backend ที่รองรับ

เหมาะต่อยอดจาก:

```text
Lab 18: Backup, Restore และ DR
```

Mini project ที่ควรลอง:

```text
ติดตั้ง Velero แล้ว backup namespace devops-lab จากนั้นลบ namespace และ restore กลับมา
```

สิ่งที่ควรเข้าใจ:

- backup schedule
- restore
- namespace backup
- volume snapshot
- object storage

## ถ้าอยากเรียน Service Mesh

### Service Mesh เช่น Istio หรือ Linkerd

Service Mesh ช่วยเรื่อง traffic management, mTLS, observability และ policy ระหว่าง service โดยใช้ sidecar หรือ dataplane

เหมาะต่อยอดจาก:

```text
Lab 14-16: Kubernetes deployment และ Helm
```

Mini project ที่ควรลอง:

```text
ติดตั้ง Linkerd หรือ Istio แล้วดู mTLS/traffic metrics ระหว่าง frontend และ backend
```

สิ่งที่ควรเข้าใจ:

- sidecar proxy
- mTLS
- traffic split
- retry/timeout
- service-to-service telemetry

## ลำดับแนะนำตามสายงาน

ถ้าอยากไปทาง DevOps Engineer:

```text
Ansible -> GitLab CI/CD ขั้นสูง -> Harbor -> Argo CD -> Alertmanager
```

ถ้าอยากไปทาง Platform Engineer:

```text
Kubernetes ขั้นสูง -> Helm -> Argo CD -> External Secrets/Vault -> MetalLB -> cert-manager
```

ถ้าอยากไปทาง SRE:

```text
Prometheus -> Alertmanager -> Loki -> OpenTelemetry -> incident response -> SLO/SLI
```

ถ้าอยากไปทาง Security/DevSecOps:

```text
Trivy -> Secret management -> image policy -> RBAC -> admission controller -> audit
```

## หลักคิดในการเรียนต่อ

อย่าเรียนเครื่องมือโดยไม่มีโจทย์ ให้ตั้งโจทย์เล็ก ๆ เช่น:

```text
ทำให้ deploy ไม่ต้อง SSH
ทำให้ secret ไม่อยู่ใน Git
ทำให้ app มี HTTPS
ทำให้รู้ก่อน disk เต็ม
ทำให้ restore namespace กลับมาได้
```

เมื่อมีโจทย์ เครื่องมือจะมีความหมายและจำได้ดีกว่าการอ่าน documentation แบบไม่มีบริบท

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [ภาคผนวก: คำสั่งที่ควรจำ](./25-appendix-commands.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [สรุปสุดท้าย](./27-final-summary.md)

<!-- lesson-nav:end -->
