# 19. Lab 16: Helm Chart

## วัตถุประสงค์

- ลดความซ้ำของ Kubernetes YAML
- แยก values ตาม environment
- upgrade และ rollback ง่ายขึ้น

## ติดตั้ง Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

## สร้าง Chart

```bash
helm create simple-api-chart
cd simple-api-chart
```

โครงสร้าง:

```text
simple-api-chart/
├── Chart.yaml
├── values.yaml
└── templates/
```

ตัวอย่าง `values.yaml`:

```yaml
replicaCount: 2

image:
  repository: devops-control:5000/simple-api
  tag: "1.0.0"

service:
  type: ClusterIP
  port: 80

containerPort: 3000

ingress:
  enabled: true
  className: nginx
  host: simple-api.lab.local
```

## ติดตั้ง / Upgrade / Rollback

```bash
helm install simple-api . -n devops-lab
helm list -n devops-lab
helm upgrade simple-api . -n devops-lab --set image.tag=1.0.1
helm history simple-api -n devops-lab
helm rollback simple-api 1 -n devops-lab
```

## แยก environment

```text
values-dev.yaml
values-uat.yaml
values-prod.yaml
```

Deploy:

```bash
helm upgrade --install simple-api . -n devops-lab -f values-dev.yaml
```

## ข้อสรุป

Helm ทำให้ deployment บน Kubernetes เป็น package ที่ reusable และเหมาะกับ CI/CD มากกว่า apply YAML ทีละหลายไฟล์

---
