# 21. Lab 18: Backup, Restore และ DR พื้นฐาน

## วัตถุประสงค์

- Backup database
- Restore database
- เข้าใจ RPO/RTO

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Backup | สำรองข้อมูล |
| Restore | กู้คืนข้อมูล |
| RPO | ยอมเสียข้อมูลย้อนหลังได้เท่าไร |
| RTO | ต้องกู้ระบบกลับมาในเวลานานเท่าไร |
| DR | Disaster Recovery แผนรับมือเหตุร้ายแรง |

## Backup PostgreSQL ใน Docker Compose

```bash
cd ~/labs/fullstack-lab
docker compose exec db pg_dump -U appuser appdb > backup-appdb.sql
ls -lh backup-appdb.sql
head backup-appdb.sql
```

## Restore

```bash
docker compose exec db psql -U appuser -d appdb -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"
cat backup-appdb.sql | docker compose exec -T db psql -U appuser -d appdb
```

## Backup Kubernetes YAML

```bash
kubectl get all -n devops-lab -o yaml > devops-lab-resources.yaml
kubectl get configmap -n devops-lab -o yaml > devops-lab-configmaps.yaml
kubectl get ingress -n devops-lab -o yaml > devops-lab-ingress.yaml
```

## ข้อสรุป

ระบบที่ deploy ได้แต่ restore ไม่ได้ ยังไม่ถือว่า production-ready ต้องทดสอบ backup/restore จริงเสมอ

---
