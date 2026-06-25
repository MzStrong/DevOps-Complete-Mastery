# 1. ภาพรวม Lab

## เป้าหมาย

หลังทำ Lab นี้จบ คุณควรทำสิ่งเหล่านี้ได้:

- ติดตั้งและดูแล Linux Server เบื้องต้น
- ตรวจสอบ SSH, Service, Process, Port, DNS และ Firewall
- Deploy Web/API ด้วย Nginx และ Systemd
- เขียน Dockerfile และ Docker Compose
- สร้าง Private Container Registry
- ทำ Git Flow, Tag และ Release Version
- ทำ CI/CD Pipeline ด้วย GitLab CI
- ใช้ Terraform เพื่อเข้าใจ Infrastructure as Code
- เก็บ Metrics ด้วย Prometheus และแสดงผลด้วย Grafana
- รวม Log ด้วย Loki และ Promtail
- สร้าง Kubernetes Cluster บน VMware
- Deploy App เข้า Kubernetes พร้อม Service, Ingress, ConfigMap, Secret
- ใช้ Helm สำหรับจัดการ Kubernetes Manifest
- Scan Security ด้วย Trivy
- Backup และ Restore ระบบเบื้องต้น

## ภาพรวม Architecture

```text
PC ที่บ้าน
└── VMware Workstation / Player
    ├── devops-control     : GitLab, Registry, เครื่องควบคุม
    ├── app-server         : Deploy app แบบ VM/Docker
    ├── monitor-server     : Prometheus, Grafana, Loki
    ├── k8s-master-01      : Kubernetes Control Plane
    ├── k8s-worker-01      : Kubernetes Worker
    └── k8s-worker-02      : Kubernetes Worker
```

---
