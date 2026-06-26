# 24. Troubleshooting รวม

ไฟล์นี้เป็น runbook สำหรับไล่ปัญหาที่พบบ่อยใน Lab ทั้งชุด หลักคิดคืออย่าเริ่มด้วยการลบหรือติดตั้งใหม่ทันที ให้ตรวจจากชั้นพื้นฐานขึ้นไปก่อน

ลำดับตรวจทั่วไป:

```text
1. เครื่องถูกไหม
2. IP/hostname ถูกไหม
3. service รันไหม
4. port listen ไหม
5. firewall เปิดไหม
6. config/env ถูกไหม
7. log บอก error อะไร
8. dependency ภายนอก เช่น registry/database พร้อมไหม
```

## SSH เข้าไม่ได้

ตรวจสอบบนเครื่องปลายทางผ่าน VMware console:

```bash
sudo systemctl status ssh
ip a
ip route
sudo ufw status numbered
ss -tulpen | grep ':22'
```

ตรวจจากเครื่องต้นทาง:

```bash
ping <target-ip>
ssh -v devops@<target-ip>
getent hosts app-server
```

สาเหตุที่พบบ่อย:

- IP ผิดหรืออยู่คนละ subnet
- SSH service ไม่ทำงาน
- firewall block port 22
- VMware network mode ไม่ตรงกัน
- hostname resolve ไปผิด IP
- user/password ผิด

แยกปัญหา:

```text
ping IP ไม่ได้
-> ตรวจ IP, subnet, VMware network, firewall

ping IP ได้ แต่ SSH ไม่ได้
-> ตรวจ ssh service, port 22, UFW

SSH ด้วย IP ได้ แต่ hostname ไม่ได้
-> ตรวจ /etc/hosts หรือ DNS
```

## Docker permission denied

อาการ:

```text
permission denied while trying to connect to the Docker daemon socket
```

แก้ไข:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

หรือ logout/login ใหม่

ตรวจ:

```bash
groups
docker ps
sudo systemctl status docker
```

ถ้ายังไม่ได้ ให้ตรวจว่า Docker daemon รันอยู่หรือไม่ ไม่ใช่แค่ permission:

```bash
sudo journalctl -u docker -n 50
```

## Container ออกทันที

ตรวจ:

```bash
docker ps -a
docker logs <container-name>
docker inspect <container-name>
```

สาเหตุ:

- command ผิด
- app crash
- env variable หาย
- port conflict
- dependency เช่น database ไม่พร้อม

แยกปัญหา:

```text
container Exited ทันที
-> docker logs ก่อน

logs บอก module/package missing
-> Dockerfile หรือ dependency install ผิด

logs บอก env missing
-> docker run/compose ไม่ได้ส่ง environment

container Running แต่เข้าไม่ได้
-> ตรวจ port mapping ด้วย docker ps และ ss
```

## Docker Compose service ไม่ขึ้น

ตรวจ:

```bash
docker compose ps
docker compose logs -f
docker compose logs -f <service-name>
docker compose config
```

สาเหตุ:

- YAML indentation ผิด
- service build ไม่ผ่าน
- environment variable ไม่ครบ
- database ยังไม่พร้อม
- port บน host ถูกใช้ไปแล้ว
- volume permission ผิด

แยกปัญหา:

```text
docker compose config fail
-> YAML หรือ Compose syntax ผิด

service Exited
-> docker compose logs <service-name>

backend ต่อ db ไม่ได้
-> ตรวจ DB_HOST ต้องเป็นชื่อ service เช่น db ไม่ใช่ localhost

เข้า app จาก host ไม่ได้
-> ตรวจ ports เช่น "8080:80" และ firewall
```

## Nginx config error

ตรวจ syntax:

```bash
sudo nginx -t
```

ดู log:

```bash
sudo tail -f /var/log/nginx/error.log
sudo journalctl -u nginx -n 50
```

ตรวจ port:

```bash
sudo systemctl status nginx
ss -tulpen | grep ':80'
```

สาเหตุที่พบบ่อย:

- config syntax ผิด
- proxy_pass ชี้ backend ผิด
- backend ไม่รัน
- port 80 ถูก process อื่นใช้
- firewall block port 80

ถ้าเจอ `502 Bad Gateway`:

```text
Nginx รับ request ได้
แต่ส่งต่อไป backend ไม่ได้
-> ตรวจ backend service, port, proxy_pass
```

## GitLab pipeline pending

ตรวจ:

```bash
docker ps | grep gitlab-runner
docker logs gitlab-runner
docker exec -it gitlab-runner gitlab-runner list
```

ใน GitLab UI ตรวจ:

```text
Project > Settings > CI/CD > Runners
```

สาเหตุ:

- runner ยังไม่ registered
- runner offline
- job ต้องการ tag แต่ runner ไม่มี tag นั้น
- project ไม่อนุญาตให้ใช้ runner
- GitLab หรือ runner network ไม่ถึงกัน

ถ้า runner รับงานแล้วแต่ job fail ให้เปิด job log ดู command ที่ fail แทนการเดาจากสถานะ pipeline

## Docker push registry ไม่ได้

ตรวจ registry:

```bash
curl http://devops-control:5000/v2/_catalog
docker ps | grep registry
docker logs registry
```

ตรวจ client:

```bash
getent hosts devops-control
docker info | grep -A 5 "Insecure Registries"
docker tag simple-api:1.0.0 devops-control:5000/simple-api:1.0.0
docker push devops-control:5000/simple-api:1.0.0
```

อาการที่พบบ่อย:

```text
server gave HTTP response to HTTPS client
-> ยังไม่ได้ตั้ง insecure registry หรือยังไม่ได้ restart Docker

no such host
-> /etc/hosts หรือ DNS ผิด

connection refused
-> registry container ไม่รันหรือ port 5000 ไม่เปิด
```

## Kubernetes node NotReady

ตรวจจาก master:

```bash
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl get pods -n kube-system
```

ตรวจบน node ที่มีปัญหา:

```bash
sudo systemctl status kubelet
sudo journalctl -u kubelet -f
sudo systemctl status containerd
swapon --show
sysctl net.ipv4.ip_forward
```

สาเหตุ:

- CNI ยังไม่พร้อม
- swap ยังเปิดอยู่
- containerd config ผิด
- firewall block port
- kubelet ติดต่อ API server ไม่ได้
- cgroup driver ไม่ตรง

อย่า reset cluster ก่อนอ่าน `kubectl describe node` และ `journalctl -u kubelet` เพราะ Events และ log มักบอกสาเหตุชัดเจน

## Pod CrashLoopBackOff

ตรวจ:

```bash
kubectl logs -n <namespace> <pod-name>
kubectl logs -n <namespace> <pod-name> --previous
kubectl describe pod -n <namespace> <pod-name>
```

สาเหตุ:

- app start ไม่สำเร็จ
- config/env ผิด
- secret ไม่มี
- database connect ไม่ได้
- command หรือ entrypoint ผิด
- livenessProbe ทำให้ container restart ซ้ำ

แยกปัญหา:

```text
logs มี stack trace
-> แก้ที่ app/config

describe บอก secret/configmap not found
-> สร้าง resource หรือแก้ชื่ออ้างอิง

probe failed
-> ตรวจ path/port ของ readiness/livenessProbe
```

## ImagePullBackOff

ตรวจ:

```bash
kubectl describe pod -n <namespace> <pod-name>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

บน worker node:

```bash
getent hosts devops-control
curl http://devops-control:5000/v2/_catalog
sudo crictl images
sudo journalctl -u containerd -n 50
```

สาเหตุ:

- image name/tag ผิด
- registry เข้าไม่ได้
- private registry ไม่มี credential
- insecure registry ยังไม่ได้ config
- worker resolve hostname ไม่ได้

ถ้าใช้ private registry แบบ HTTP ต้อง config container runtime บน worker node ทุกเครื่อง ไม่ใช่แค่เครื่องที่ build image

## Service เรียกไม่ได้ใน Kubernetes

ตรวจ:

```bash
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace> <service-name>
kubectl get pods -n <namespace> --show-labels
kubectl describe svc -n <namespace> <service-name>
```

สาเหตุ:

- selector ของ Service ไม่ตรงกับ label ของ pod
- targetPort ผิด
- pod ยังไม่ Ready
- app ไม่ listen port ที่ระบุ

ถ้า endpoint ว่าง ให้แก้ label/selector ก่อน ถ้า endpoint มีแต่ curl ไม่ได้ ให้ตรวจ targetPort และ app log

## Ingress เข้าไม่ได้

ตรวจ:

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
kubectl get ingress -n devops-lab
kubectl describe ingress -n devops-lab simple-api
kubectl get endpoints -n devops-lab simple-api
```

ทดสอบแบบระบุ Host header:

```bash
curl -H "Host: simple-api.lab.local" http://<node-ip>
```

สาเหตุ:

- ingress controller ไม่รัน
- ingressClassName ไม่ตรง
- host ใน `/etc/hosts` ชี้ผิด IP
- Service ไม่มี endpoint
- NodePort/port ที่ใช้ทดสอบผิด

## Helm install หรือ upgrade fail

ตรวจ:

```bash
helm lint .
helm template simple-api . -n devops-lab
helm install simple-api . -n devops-lab --dry-run --debug
helm list -n devops-lab
helm status simple-api -n devops-lab
```

สาเหตุ:

- template syntax ผิด
- values key ไม่มีหรือชื่อผิด
- namespace ยังไม่มี
- resource เดิมมีอยู่แล้วแต่ไม่ได้ถูก Helm จัดการ
- Kubernetes manifest ที่ render ออกมาไม่ valid

แยกปัญหา:

```text
helm lint fail
-> แก้ chart structure หรือ template syntax

helm template ออกมา YAML ผิด
-> แก้ values/templates ก่อน install

install ผ่านแต่ pod ไม่ขึ้น
-> ไป debug ด้วย kubectl describe/logs

rollback แล้ว app ยังพัง
-> Helm rollback manifest ได้ แต่ไม่ได้ rollback database/data
```

ดู manifest ที่ release ใช้อยู่:

```bash
helm get values simple-api -n devops-lab
helm get manifest simple-api -n devops-lab
helm history simple-api -n devops-lab
```

## Prometheus target Down

ตรวจจาก monitor-server:

```bash
curl http://target-host:9100/metrics
getent hosts target-host
```

ตรวจบน target:

```bash
sudo systemctl status node_exporter
ss -tulpen | grep ':9100'
sudo ufw status numbered
```

สาเหตุ:

- node_exporter ไม่ทำงาน
- firewall block port 9100
- hostname resolve ไม่ได้
- prometheus.yml target ผิด
- Prometheus container resolve hostname ไม่ได้

ถ้า curl จาก host ได้ แต่ Prometheus ยัง down ให้ตรวจ config และ logs:

```bash
docker compose logs prometheus
```

## Loki ไม่มี log

ตรวจ Loki:

```bash
curl http://monitor-server:3100/ready
docker compose logs loki
```

ตรวจ Promtail:

```bash
docker ps | grep promtail
docker logs promtail
curl http://localhost:9080/targets
```

สาเหตุ:

- Promtail ส่งไป URL ผิด
- monitor-server resolve ไม่ได้
- path ใน `__path__` ไม่ match ไฟล์ log
- label ที่ query ไม่ตรง
- Grafana time range แคบเกินไป

ทดสอบสร้าง log:

```bash
logger "devops lab test log"
```

แล้ว query:

```text
{host="app-server"} |= "devops lab test"
```

## Terraform apply ผิดพลาด

ตรวจ:

```bash
terraform fmt
terraform validate
terraform plan
terraform state list
```

สาเหตุ:

- syntax HCL ผิด
- provider ยังไม่ได้ init
- state ไม่ตรงกับ resource จริง
- permission ไม่พอ
- plan กำลังจะลบ resource ที่ไม่คาดคิด

หลักปฏิบัติ: อ่าน plan ก่อน apply เสมอ โดยเฉพาะถ้าเห็น `destroy`

## Trivy scan fail หรือ pipeline fail เพราะ security scan

ตรวจบนเครื่องที่รัน scan:

```bash
trivy --version
trivy image simple-api:1.0.0
trivy fs .
```

สาเหตุ:

- Trivy download vulnerability database ไม่ได้
- image tag ไม่มีใน local/registry
- registry เข้าถึงไม่ได้
- pipeline ตั้ง `--exit-code 1` แล้วเจอ vulnerability ตาม threshold
- ไม่มี permission อ่านไฟล์บาง path

แยกปัญหา:

```text
scan fail เพราะ DB download ไม่ได้
-> ตรวจ internet/DNS/proxy

scan image ไม่เจอ
-> docker images หรือ docker pull image ก่อน

pipeline fail เพราะ CRITICAL
-> อ่าน CVE, fixed version และ update base image/dependency

เจอ LOW/MEDIUM จำนวนมาก
-> อย่าเพิ่ง block ทั้งหมด ให้ตั้ง severity gate ตาม risk ที่ทีมรับได้
```

ถ้าใช้ใน GitLab CI ให้เปิด job log และดูว่า command ไหน exit code ไม่เป็น 0

## Backup หรือ Restore ใช้ไม่ได้

ตรวจ backup file:

```bash
ls -lh backup-appdb.sql
head backup-appdb.sql
grep -i "PostgreSQL database dump" backup-appdb.sql
```

ทดสอบ restore:

```bash
docker compose exec db createdb -U appuser appdb_restore_test
cat backup-appdb.sql | docker compose exec -T db psql -U appuser -d appdb_restore_test
docker compose exec db psql -U appuser -d appdb_restore_test -c "\dt"
```

สาเหตุ:

- backup file ว่างหรือ dump ไม่ครบ
- backup มาจาก database คนละ version แล้ว restore มี warning/error
- restore เข้า database ผิดตัว
- permission อ่านไฟล์ backup ไม่ได้
- backup Kubernetes YAML มี field runtime เช่น `resourceVersion`, `uid`, `status`
- backup resource YAML แต่ไม่ได้ backup data ใน volume/database

หลักปฏิบัติ:

```text
backup ที่ไม่เคย restore = ยังไม่ถือว่าเชื่อถือได้
restore test ควรทำใน database แยกก่อน
เก็บ backup นอก container/volume ที่อาจหายพร้อมกัน
Secret backup ต้องเก็บอย่างปลอดภัย
```

---
