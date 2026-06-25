# 22. Final Project

## เป้าหมาย

เอา Fullstack App ของคุณเองมาทำเป็น DevOps Project แบบครบวงจร

## โครงสร้างที่แนะนำ

```text
my-devops-project/
├── frontend/
├── backend/
├── docker-compose.yml
├── nginx/
├── k8s/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── secret.yaml
├── helm/
├── terraform/
├── monitoring/
├── .gitlab-ci.yml
└── README.md
```

## Pipeline ที่ต้องการ

```text
Developer push code
-> test
-> build Docker image
-> scan image
-> push registry
-> deploy dev
-> manual approve
-> deploy prod/lab
-> monitor
-> rollback ได้
```

## Requirement

- App run ได้บน local
- มี Dockerfile frontend/backend
- มี Docker Compose
- มี Private Registry
- มี GitLab repo
- มี CI/CD pipeline
- มี Trivy scan
- Deploy เข้า Kubernetes ได้
- มี Ingress
- มี ConfigMap/Secret
- มี Helm Chart
- มี Grafana Dashboard
- มี log ใน Loki
- มี backup/restore test
- README อธิบายวิธีติดตั้งและทดสอบครบ

---
