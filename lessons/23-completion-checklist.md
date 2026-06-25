# 23. Checklist หลังเรียนจบ

Checklist นี้ใช้ตรวจว่าคุณไม่ได้แค่รันคำสั่งตาม Lab ผ่าน แต่เข้าใจและสามารถพิสูจน์ผลลัพธ์ได้จริง วิธีใช้คือทำเครื่องหมายเมื่อคุณสามารถแสดง command output, screenshot, dashboard, pipeline log หรือไฟล์ config ที่เกี่ยวข้องได้

หลักฐานที่ดีควรตอบได้ว่า:

```text
ทำอะไร
รันที่เครื่องไหน
ตรวจด้วยคำสั่งอะไร
ผลลัพธ์ที่ถูกต้องหน้าตาเป็นอย่างไร
ถ้าพังจะเริ่ม debug จากจุดไหน
```

## Linux / Network

- [ ] SSH เข้า server ได้
- [ ] ตั้ง static IP ได้
- [ ] ตรวจ DNS ได้
- [ ] ตรวจ port ด้วย `ss` ได้
- [ ] ใช้ UFW ได้
- [ ] ดู service ด้วย systemctl ได้
- [ ] ดู log ด้วย journalctl ได้

หลักฐานที่ควรมี:

```bash
ssh devops@app-server
hostnamectl
ip a
ip route
getent hosts app-server
ss -tulpen
sudo ufw status numbered
sudo systemctl status ssh
sudo journalctl -u ssh -n 20
```

ควรอธิบายได้ว่า IP ของแต่ละ VM คืออะไร, hostname resolve อย่างไร, service ใด listen port ใด และ firewall เปิดอะไรไว้บ้าง

## Docker

- [ ] เขียน Dockerfile ได้
- [ ] build image ได้
- [ ] run container ได้
- [ ] ใช้ volume ได้
- [ ] ใช้ Docker Compose ได้
- [ ] push image เข้า registry ได้

หลักฐานที่ควรมี:

```bash
docker version
docker images
docker ps
docker logs <container-name>
docker compose ps
curl http://localhost:3000/health
curl http://devops-control:5000/v2/_catalog
```

ควรอธิบายได้ว่า Dockerfile แต่ละบรรทัดทำอะไร, image ต่างจาก container อย่างไร, `-p host:container` อ่านอย่างไร และ Compose service เรียกกันด้วยชื่อ service ได้เพราะอะไร

## CI/CD

- [ ] เข้าใจ stage/job
- [ ] run test ใน pipeline ได้
- [ ] build image ได้
- [ ] push registry ได้
- [ ] deploy อัตโนมัติได้
- [ ] rollback ได้

หลักฐานที่ควรมี:

```text
GitLab pipeline ล่าสุดผ่าน
job log ของ test/build/scan/deploy
image tag ใหม่ใน registry
.gitlab-ci.yml อยู่ใน repository
rollback strategy ระบุไว้ใน README หรือ runbook
```

ควรอธิบายได้ว่า Runner อยู่ที่ไหน, job รันใน image อะไร, image tag มาจาก commit/tag ไหน และถ้า pipeline fail จะเปิด log ที่ไหน

## Kubernetes

- [ ] สร้าง cluster ได้
- [ ] deploy Deployment ได้
- [ ] expose Service ได้
- [ ] ใช้ Ingress ได้
- [ ] ใช้ ConfigMap/Secret ได้
- [ ] scale app ได้
- [ ] rollout/rollback ได้
- [ ] ใช้ Helm ได้

หลักฐานที่ควรมี:

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get all -n devops-lab
kubectl get ingress -n devops-lab
kubectl get configmap,secret -n devops-lab
kubectl rollout history deployment/simple-api -n devops-lab
helm list -n devops-lab
helm history simple-api -n devops-lab
```

ควรอธิบายได้ว่า Deployment, Pod, Service, Ingress, ConfigMap, Secret และ Helm release เชื่อมกันอย่างไร ถ้า pod เป็น `ImagePullBackOff` หรือ `CrashLoopBackOff` จะตรวจตรงไหนก่อน

## Monitoring / Logging

- [ ] Prometheus scrape target ได้
- [ ] Grafana dashboard ได้
- [ ] Loki query log ได้
- [ ] เข้าใจ alert concept

หลักฐานที่ควรมี:

```text
Prometheus Targets แสดง app-server/node-exporter เป็น UP
Grafana dashboard มี CPU/RAM/Disk จริง
Grafana Loki query เช่น {host="app-server"} เจอ log
มีคำอธิบายว่า metric ไหนควร alert
```

คำสั่งตรวจ:

```bash
curl http://app-server:9100/metrics | head
curl http://monitor-server:3100/ready
docker compose ps
```

ควรอธิบายได้ว่า metrics กับ logs ต่างกันอย่างไร และเมื่อ Grafana ไม่มีข้อมูลควรตรวจ datasource, target, exporter หรือ promtail ตรงไหน

## Security

- [ ] scan image ด้วย Trivy ได้
- [ ] ไม่ commit secret
- [ ] ใช้ least privilege
- [ ] ตั้ง resource limit
- [ ] เปิด firewall เฉพาะที่จำเป็น

หลักฐานที่ควรมี:

```bash
trivy image devops-control:5000/simple-api:1.0.0
trivy fs .
git status --ignored
sudo ufw status numbered
kubectl describe deployment simple-api -n devops-lab
```

ควรแสดงได้ว่า `.env` ไม่ถูก commit, secret ถูกจัดการผ่าน Secret/CI variables, container มี resource request/limit และ pipeline มี security scan อย่างน้อยหนึ่งจุด

## Backup / Restore

- [ ] backup database ได้
- [ ] restore database ได้
- [ ] backup Kubernetes YAML หรือ Helm values ได้
- [ ] เข้าใจ RPO/RTO ของ lab
- [ ] เก็บ backup ไว้นอก container/volume ที่อาจหายพร้อมกัน

หลักฐานที่ควรมี:

```bash
ls -lh backup-appdb-*.sql
head backup-appdb-*.sql
docker compose exec db psql -U appuser -d appdb -c "\dt"
kubectl get all -n devops-lab -o yaml > devops-lab-resources.yaml
```

ควรมีบันทึกว่าเคย restore test แล้วใช้เวลาประมาณเท่าไร และข้อมูลที่ backup ครอบคลุมอะไรบ้าง

## เกณฑ์ผ่านแบบ Practical

ถือว่าพร้อมต่อยอดได้เมื่อคุณทำ demo ได้ครบตามนี้:

1. SSH เข้า VM และแสดง network plan ได้
2. เปิด app ผ่าน Docker Compose ได้
3. push code แล้ว GitLab pipeline build/scan/push image ได้
4. deploy app เข้า Kubernetes ได้
5. เปิด app ผ่าน Ingress ได้
6. ดู metrics ใน Grafana ได้
7. query log ใน Loki ได้
8. scan image/source ด้วย Trivy ได้
9. rollback release หรืออธิบาย rollback path ได้
10. restore backup อย่างน้อยหนึ่งครั้งได้

ถ้าข้อใดทำไม่ได้ ให้ย้อนกลับไปบทที่เกี่ยวข้องและแก้ให้มีหลักฐานก่อนเดินหน้าต่อ

---
