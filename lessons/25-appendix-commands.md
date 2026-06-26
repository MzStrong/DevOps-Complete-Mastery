# ภาคผนวก: คำสั่งที่ควรจำ

ภาคผนวกนี้เป็น quick reference สำหรับกลับมาเปิดดูตอนทำ Lab หรือ debug ปัญหา คำสั่งไม่ได้เรียงตามบท แต่เรียงตามประเภทงานที่เจอบ่อย เช่น ตรวจ Linux, Docker, Kubernetes, Helm, Monitoring และ Security

หลักการใช้คำสั่งเหล่านี้คืออย่ารันแบบจำอย่างเดียว ให้ถามตัวเองก่อนว่า “คำสั่งนี้กำลังช่วยตอบคำถามอะไร”

```text
ระบบถึงกันไหม
service รันไหม
port เปิดไหม
log บอกอะไร
resource พอไหม
deployment อยู่ version ไหน
rollback ได้ไหม
```

## Linux / Network

```bash
hostnamectl
whoami
pwd
ls -lah
ip a
ip route
getent hosts <hostname>
ping -c 4 <host-or-ip>
ss -tulpen
curl -I http://example.com
df -h
free -h
top
```

ใช้เมื่อ:

- ตรวจ hostname/user/path ก่อนแก้ config
- ตรวจ IP, route และ hostname resolve
- ตรวจ port ที่ service เปิด
- ตรวจ disk, memory และ process ที่ใช้ resource สูง

คำสั่งที่ใช้บ่อยตอน network มีปัญหา:

```bash
ip a
ip route
getent hosts app-server
ping -c 4 app-server
ss -tulpen | grep ':80'
curl -v http://app-server
```

## Service และ Log

```bash
sudo systemctl status <service>
sudo systemctl restart <service>
sudo systemctl enable <service>
sudo journalctl -u <service> -n 50
sudo journalctl -u <service> -f
```

ใช้เมื่อ:

- service start ไม่ขึ้น
- ต้อง restart service หลังแก้ config
- ต้องดู log ย้อนหลังหรือ follow log แบบ real-time

Pattern ที่ควรจำ:

```bash
sudo systemctl status <service>
sudo journalctl -u <service> -n 50
```

ดู status ก่อน แล้วค่อยอ่าน log เพิ่ม

## Docker

```bash
docker version
docker images
docker build -t name:tag .
docker run -d --name app -p 8080:80 name:tag
docker ps
docker ps -a
docker logs -f app
docker exec -it app sh
docker stop app
docker rm app
```

ใช้เมื่อ:

- build image
- run container
- ตรวจว่า container ยัง running หรือ exited
- ดู log ของ container
- เข้าไปตรวจไฟล์ภายใน container

Pattern debug container:

```bash
docker ps -a
docker logs <container-name>
docker inspect <container-name>
```

ถ้า container ออกทันที ให้ดู `docker logs` ก่อนเสมอ

## Docker Compose

```bash
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose logs -f <service>
docker compose exec <service> sh
docker compose down
docker compose down -v
docker compose config
```

ใช้เมื่อ:

- รันหลาย service เช่น frontend/backend/db/nginx
- ตรวจ log ราย service
- validate compose syntax
- ล้าง container หรือ volume

ข้อควรระวัง:

```bash
docker compose down -v
```

คำสั่งนี้ลบ volume ด้วย ข้อมูล database อาจหาย ใช้เฉพาะเมื่อตั้งใจ reset ข้อมูล

## Registry

```bash
curl http://devops-control:5000/v2/_catalog
curl http://devops-control:5000/v2/simple-api/tags/list
docker tag simple-api:1.0.0 devops-control:5000/simple-api:1.0.0
docker push devops-control:5000/simple-api:1.0.0
docker pull devops-control:5000/simple-api:1.0.0
docker info | grep -A 5 "Insecure Registries"
```

ใช้เมื่อ:

- ตรวจ registry ว่าพร้อมใช้งาน
- push/pull image
- ตรวจ tag ที่อยู่ใน registry
- ตรวจว่า Docker client ยอมใช้ insecure registry แล้วหรือยัง

## Kubernetes

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get all -n <namespace>
kubectl get svc,endpoints -n <namespace>
kubectl describe pod -n <namespace> <pod>
kubectl logs -n <namespace> <pod>
kubectl logs -n <namespace> <pod> --previous
kubectl apply -f file.yaml
kubectl delete -f file.yaml
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout history deployment/<name> -n <namespace>
kubectl rollout undo deployment/<name> -n <namespace>
```

ใช้เมื่อ:

- ตรวจ cluster และ pod
- ตรวจ Service กับ endpoints
- อ่าน Events และ log ของ pod
- apply manifest
- rollout/rollback deployment

Pattern debug pod:

```bash
kubectl get pods -n <namespace> -o wide
kubectl describe pod -n <namespace> <pod>
kubectl logs -n <namespace> <pod>
kubectl logs -n <namespace> <pod> --previous
```

`describe` ใช้ดู Events เช่น image pull fail, probe fail หรือ scheduling fail ส่วน `logs --previous` ใช้ดู log ก่อน container restart

## Helm

```bash
helm lint .
helm template <release> . -n <namespace>
helm install <release> <chart> -n <namespace>
helm upgrade <release> <chart> -n <namespace>
helm upgrade --install <release> <chart> -n <namespace> -f values-dev.yaml
helm rollback <release> <revision> -n <namespace>
helm history <release> -n <namespace>
helm status <release> -n <namespace>
helm get values <release> -n <namespace>
helm get manifest <release> -n <namespace>
```

ใช้เมื่อ:

- ตรวจ chart ก่อน deploy
- render YAML จาก template
- install/upgrade/rollback release
- ดู values และ manifest ที่ release ใช้จริง

## Monitoring

```bash
curl http://app-server:9100/metrics | head
curl http://monitor-server:9090
docker compose logs prometheus
```

Prometheus queries ที่ควรจำ:

```text
up
node_memory_MemAvailable_bytes
node_filesystem_avail_bytes
rate(node_cpu_seconds_total[5m])
```

ใช้เมื่อ:

- ตรวจ exporter เปิด metrics หรือไม่
- ตรวจ target up/down
- เริ่มสร้าง dashboard CPU/RAM/Disk

## Logging

```bash
curl http://monitor-server:3100/ready
docker logs promtail
curl http://localhost:9080/targets
logger "devops lab test log"
```

LogQL ที่ควรจำ:

```text
{host="app-server"}
{job="varlogs"}
{host="app-server"} |= "error"
{host="app-server"} |= "devops lab test"
```

ใช้เมื่อ:

- ตรวจ Loki พร้อมไหม
- ตรวจ Promtail อ่าน log หรือไม่
- query log ผ่าน Grafana

## Terraform

```bash
terraform version
terraform fmt
terraform init
terraform validate
terraform plan
terraform apply
terraform state list
terraform destroy
```

ใช้เมื่อ:

- จัด format และ validate code
- init provider
- preview ก่อนเปลี่ยน infra
- apply หรือ destroy resource
- ดู state

หลักปฏิบัติ:

```text
อ่าน terraform plan ก่อน apply เสมอ
ถ้าเห็น destroy โดยไม่ตั้งใจ ให้หยุดตรวจ code/state ก่อน
```

## Security / Trivy

```bash
trivy --version
trivy image simple-api:1.0.0
trivy image --severity HIGH,CRITICAL simple-api:1.0.0
trivy image --exit-code 1 --severity CRITICAL simple-api:1.0.0
trivy fs .
trivy fs --scanners vuln,secret,misconfig .
```

ใช้เมื่อ:

- scan image ก่อน deploy
- scan source repository
- ตั้ง security gate ใน pipeline

## Backup / Restore

```bash
docker compose exec db pg_dump -U appuser appdb > backup-appdb.sql
ls -lh backup-appdb.sql
head backup-appdb.sql
cat backup-appdb.sql | docker compose exec -T db psql -U appuser -d appdb
kubectl get all -n devops-lab -o yaml > devops-lab-resources.yaml
```

ใช้เมื่อ:

- backup PostgreSQL
- ตรวจไฟล์ backup
- restore database
- export Kubernetes resource YAML

ข้อควรจำ:

```text
backup ที่ไม่เคย restore ยังไม่ถือว่าเชื่อถือได้
```

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [24. Troubleshooting รวม](./24-troubleshooting.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [แนวทางเรียนต่อ](./26-next-steps.md)

<!-- lesson-nav:end -->
