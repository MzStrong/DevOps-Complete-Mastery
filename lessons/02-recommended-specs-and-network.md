# 2. สเปกเครื่องและ Network ที่แนะนำ

## สเปก VM แบบประหยัด

| VM | CPU | RAM | Disk | Role |
|---|---:|---:|---:|---|
| devops-control | 2 vCPU | 4-6 GB | 60 GB | GitLab, Registry |
| app-server | 2 vCPU | 2 GB | 30 GB | Docker/App |
| monitor-server | 2 vCPU | 3-4 GB | 40 GB | Monitoring/Logging |
| k8s-master-01 | 2 vCPU | 3-4 GB | 40 GB | K8s Control Plane |
| k8s-worker-01 | 2 vCPU | 2-3 GB | 40 GB | K8s Worker |
| k8s-worker-02 | 2 vCPU | 2-3 GB | 40 GB | K8s Worker |

ถ้า RAM น้อย ให้เริ่มจาก 3 เครื่องก่อน:

```text
devops-control
app-server
k8s-master-01 + k8s-worker-01 เมื่อถึงบท Kubernetes
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

## OS ที่ใช้

```text
Ubuntu Server 22.04 LTS หรือ 24.04 LTS
```

## User ที่ใช้ใน Lab

```text
devops
```

ให้ user นี้มีสิทธิ์ sudo

---
