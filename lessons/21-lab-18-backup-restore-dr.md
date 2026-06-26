# 21. Lab 18: Backup, Restore และ DR พื้นฐาน

Lab นี้ฝึกแนวคิดที่มักถูกมองข้ามหลัง deploy สำเร็จ: ระบบต้องกู้คืนได้จริง การ backup ไม่ใช่แค่มีไฟล์สำรอง แต่ต้องรู้ว่า backup นั้นครบไหม เก็บไว้ที่ไหน กู้คืนอย่างไร ใช้เวลานานเท่าไร และยอมเสียข้อมูลย้อนหลังได้แค่ไหน

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

คำถามหลักของบทนี้:

```text
ถ้า database หาย จะกู้กลับมาได้ไหม
ถ้า Kubernetes resource ถูกลบ จะสร้างกลับได้ไหม
ถ้า VM เสีย ต้องใช้ไฟล์อะไรบ้างในการฟื้นระบบ
ใช้เวลานานแค่ไหน
ข้อมูลจะหายย้อนหลังเท่าไร
```

ระบบที่ deploy ได้แต่ restore ไม่ได้ ยังไม่ควรถูกมองว่า production-ready

## Backup PostgreSQL ใน Docker Compose

```bash
cd ~/labs/fullstack-lab
docker compose exec db pg_dump -U appuser appdb > backup-appdb.sql
ls -lh backup-appdb.sql
head backup-appdb.sql
```

คำสั่งนี้ dump database `appdb` จาก container `db` ออกมาเป็นไฟล์ SQL บน host:

- `docker compose exec db` รันคำสั่งใน database container
- `pg_dump -U appuser appdb` export schema และ data ของ database
- `> backup-appdb.sql` redirect output มาเก็บเป็นไฟล์บน host

ตรวจ backup:

```bash
ls -lh backup-appdb.sql
head backup-appdb.sql
grep -i "PostgreSQL database dump" backup-appdb.sql
```

ไฟล์ backup ที่ size เป็น 0 byte หรือเปิดแล้วไม่มีเนื้อหา SQL ถือว่า backup ใช้ไม่ได้

ตั้งชื่อไฟล์ให้มี timestamp จะดีกว่า:

```bash
BACKUP_FILE="backup-appdb-$(date +%Y%m%d-%H%M%S).sql"
docker compose exec db pg_dump -U appuser appdb > "$BACKUP_FILE"
ls -lh "$BACKUP_FILE"
```

เหตุผลคือถ้าใช้ชื่อเดิมทุกครั้งจะเขียนทับ backup เก่าโดยไม่ตั้งใจ

ตัวอย่างที่ผิด:

```bash
docker compose exec db pg_dump -U appuser appdb > backup.sql
```

แล้วไม่ตรวจว่าไฟล์มีข้อมูลจริงหรือไม่

ตัวอย่างที่ถูก:

```bash
docker compose exec db pg_dump -U appuser appdb > backup.sql
ls -lh backup.sql
head backup.sql
```

## Restore

```bash
docker compose exec db psql -U appuser -d appdb -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"
cat backup-appdb.sql | docker compose exec -T db psql -U appuser -d appdb
```

ขั้นตอน restore นี้ล้าง schema `public` แล้วนำ backup กลับเข้า database เดิม

คำเตือน: คำสั่ง `DROP SCHEMA public CASCADE` ลบ object ใน schema จริง ใช้เฉพาะ lab หรือเมื่อมั่นใจว่า restore ถูกต้องแล้ว ในระบบจริงต้องมีแผน, approval และ backup ล่าสุดก่อนทำ

หลัง restore ให้ตรวจข้อมูล:

```bash
docker compose exec db psql -U appuser -d appdb -c "\dt"
docker compose exec db psql -U appuser -d appdb -c "SELECT NOW();"
```

ถ้า application มี endpoint ตรวจ database เช่น `/api/db` ให้ทดสอบด้วย:

```bash
curl http://localhost:8080/api/db
```

สิ่งสำคัญคือ “ต้องซ้อม restore” เพราะ backup ที่ไม่เคย restore อาจมีปัญหา เช่น dump ไม่ครบ, version ไม่เข้ากัน, permission ผิด หรือขั้นตอนกู้คืนใช้เวลานานกว่าที่คิด

## ทดสอบ Restore แบบปลอดภัยกว่า

ถ้าไม่อยากลบ database เดิม ให้สร้าง database ใหม่สำหรับทดสอบ restore:

```bash
docker compose exec db createdb -U appuser appdb_restore_test
cat backup-appdb.sql | docker compose exec -T db psql -U appuser -d appdb_restore_test
docker compose exec db psql -U appuser -d appdb_restore_test -c "\dt"
```

วิธีนี้ช่วยตรวจว่า backup ใช้ restore ได้ โดยไม่ทำลาย database หลักใน lab

## Backup Kubernetes YAML

```bash
kubectl get all -n devops-lab -o yaml > devops-lab-resources.yaml
kubectl get configmap -n devops-lab -o yaml > devops-lab-configmaps.yaml
kubectl get ingress -n devops-lab -o yaml > devops-lab-ingress.yaml
```

คำสั่งนี้ export resource บางส่วนใน namespace `devops-lab` ออกมาเป็น YAML เพื่อใช้อ้างอิงหรือกู้ resource บางอย่าง

ควร backup เพิ่ม:

```bash
kubectl get secret -n devops-lab -o yaml > devops-lab-secrets.yaml
kubectl get pvc -n devops-lab -o yaml > devops-lab-pvc.yaml
kubectl get pv -o yaml > cluster-pv.yaml
```

ข้อควรระวัง:

- Secret YAML มีข้อมูล base64 encoded ที่ถือว่า sensitive ต้องเก็บให้ปลอดภัย
- YAML ที่ export จาก cluster จะมี field runtime เช่น `status`, `resourceVersion`, `uid` ซึ่งไม่ควรใช้เป็น manifest สะอาดสำหรับ GitOps โดยตรง
- Backup resource YAML ไม่เท่ากับ backup ข้อมูลใน volume หรือ database

สำหรับ manifest ที่ควรใช้สร้างระบบซ้ำ ควรเก็บ source manifest/Helm chart ใน Git อยู่แล้ว ส่วน `kubectl get ... -o yaml` เหมาะเป็น snapshot เพื่อดูสถานะหรือกู้ฉุกเฉิน

## RPO และ RTO แบบเข้าใจง่าย

ตัวอย่าง:

```text
Backup database ทุก 24 ชั่วโมง
-> RPO โดยประมาณคือยอมเสียข้อมูลได้สูงสุดเกือบ 24 ชั่วโมง

ขั้นตอน restore ใช้เวลา 2 ชั่วโมง
-> RTO โดยประมาณคือ 2 ชั่วโมง
```

ถ้าธุรกิจยอมเสียข้อมูลได้แค่ 15 นาที แต่ backup วันละครั้ง แปลว่า backup strategy ไม่พอ ต้องเพิ่มความถี่หรือใช้ replication/binlog/WAL backup ตามระบบ database ที่ใช้

ใน lab ให้เริ่มจากการตอบคำถามนี้:

```text
ถ้า database หายตอนนี้ backup ล่าสุดอยู่ที่ไหน
restore ใช้คำสั่งอะไร
ใช้เวลาประมาณกี่นาที
ข้อมูลที่เพิ่มหลัง backup จะหายไหม
```

## เก็บ Backup ไว้ที่ไหน

อย่าเก็บ backup ไว้ที่เดียวกับระบบที่อาจเสียหายทั้งหมด เช่นเก็บ database backup ไว้ใน container เดียวกับ database แล้วลบ container/volume หายพร้อมกัน

แนวทางใน lab:

```text
backup file อยู่บน host
copy ไปอีก VM หรือ external disk
ตั้งชื่อมี timestamp
ทดสอบ restore เป็นระยะ
```

แนวทาง production:

```text
เก็บ backup นอก cluster
เข้ารหัส backup
จำกัดสิทธิ์เข้าถึง
ตั้ง retention policy
monitor job backup
ซ้อม restore
```

## DR Checklist พื้นฐาน

- [ ] รู้ว่าอะไรคือข้อมูลสำคัญ เช่น database, uploaded files, registry data, GitLab data
- [ ] มี backup ของข้อมูลสำคัญ
- [ ] มี manifest/Helm chart/IaC ใน Git
- [ ] รู้วิธี restore database
- [ ] รู้วิธีสร้าง Kubernetes resource กลับมา
- [ ] รู้ว่า backup เก็บที่ไหนและใครเข้าถึงได้
- [ ] เคยทดสอบ restore จริง
- [ ] วัดเวลาที่ใช้ restore ได้

## ข้อสรุป

ระบบที่ deploy ได้แต่ restore ไม่ได้ ยังไม่ถือว่า production-ready ต้องทดสอบ backup/restore จริงเสมอ

สิ่งที่ควรจำ:

```text
Backup ต้องตรวจได้
Restore ต้องซ้อมจริง
RPO = ยอมเสียข้อมูลย้อนหลังได้เท่าไร
RTO = ต้องกู้กลับมาเร็วแค่ไหน
YAML backup ไม่เท่ากับ data backup
Secret backup ต้องเก็บอย่างปลอดภัย
```

บทนี้ปิดวงจรพื้นฐานของ DevOps Lab: deploy ได้, monitor/log ได้, secure ได้ และกู้คืนได้เมื่อเกิดปัญหา

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [20. Lab 17: DevSecOps ด้วย Trivy](./20-lab-17-devsecops-trivy.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [22. Final Project](./22-final-project.md)

<!-- lesson-nav:end -->
