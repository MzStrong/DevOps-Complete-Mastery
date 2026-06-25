# 10. Lab 07: Private Container Registry

Lab นี้สร้างที่เก็บ Docker image ส่วนตัวใน lab ของเราเอง Registry จะเป็นจุดกลางระหว่างเครื่องที่ build image กับเครื่องที่นำ image ไปรัน เช่น GitLab Runner build image แล้ว push เข้า registry จากนั้น `app-server` หรือ Kubernetes worker pull image เดียวกันไปรัน

## วัตถุประสงค์

- สร้าง registry ส่วนตัว
- push/pull image ระหว่าง VM
- เตรียม registry สำหรับ CI/CD และ Kubernetes

ภาพรวม flow:

```text
app-server หรือ GitLab Runner
-> docker build simple-api:1.0.0
-> docker tag devops-control:5000/simple-api:1.0.0
-> docker push devops-control:5000/simple-api:1.0.0
-> server/Kubernetes pull image จาก devops-control:5000
```

ถ้าไม่มี registry กลาง คุณต้อง copy image ไปแต่ละเครื่องเอง ซึ่งไม่เหมาะกับ CI/CD และ Kubernetes เพราะ cluster ต้องมีที่ให้ node ดึง image ได้เสมอ

## Run Registry บน devops-control

```bash
docker run -d --name registry --restart always -p 5000:5000 -v registry_data:/var/lib/registry registry:2
curl http://localhost:5000/v2/_catalog
```

คำสั่งนี้รัน Docker Registry official image:

- `--name registry` ตั้งชื่อ container
- `--restart always` ให้ registry start ใหม่หลัง reboot หรือ container crash
- `-p 5000:5000` เปิด registry ที่ port 5000 ของ `devops-control`
- `-v registry_data:/var/lib/registry` เก็บ image data ใน named volume
- `registry:2` คือ image ของ Docker Registry v2

endpoint `/v2/_catalog` ใช้ตรวจว่า registry ตอบสนอง ถ้า registry ยังว่าง จะเห็น repository list ว่าง เช่น:

```json
{"repositories":[]}
```

ตรวจเพิ่มเติม:

```bash
docker ps
docker logs registry
ss -tulpen | grep ':5000'
curl http://devops-control:5000/v2/_catalog
```

ถ้า `localhost:5000` ใช้ได้บน `devops-control` แต่เครื่องอื่นเข้า `devops-control:5000` ไม่ได้ ให้ตรวจ `/etc/hosts`, firewall และ network ก่อน

ข้อควรระวัง: registry ใน Lab นี้ใช้ HTTP ไม่ใช่ HTTPS เพื่อให้ setup ง่าย ใน production ควรใช้ HTTPS, authentication และ access control

## ตั้ง insecure registry บน client

บน app-server:

```bash
sudo nano /etc/docker/daemon.json
```

```json
{
  "insecure-registries": ["devops-control:5000"]
}
```

Docker client ปกติคาดหวังว่า registry ภายนอกจะใช้ HTTPS แต่ lab นี้ใช้ HTTP จึงต้องบอก Docker ว่า `devops-control:5000` เป็น insecure registry ที่อนุญาตให้ใช้ได้

ต้องตั้งค่านี้บนทุกเครื่องที่จะ pull/push image จาก registry เช่น:

- `app-server`
- `devops-control` ถ้าจะ push จากเครื่องเดียวกันด้วย hostname
- GitLab Runner host
- Kubernetes node ทุกเครื่องในบท Kubernetes

Restart:

```bash
sudo systemctl restart docker
```

หลังแก้ `/etc/docker/daemon.json` ต้อง restart Docker daemon เพื่อให้ config มีผล

ตรวจว่า Docker อ่าน config แล้ว:

```bash
docker info | grep -A 5 "Insecure Registries"
```

ควรเห็น `devops-control:5000` อยู่ในรายการ

ตัวอย่างที่ผิด:

```json
{
  "insecure-registries": ["http://devops-control:5000"]
}
```

Docker ต้องการค่า host:port ไม่ใช่ URL ที่มี `http://`

ตัวอย่างที่ถูก:

```json
{
  "insecure-registries": ["devops-control:5000"]
}
```

ถ้าไฟล์ `daemon.json` มี JSON ผิดรูป Docker อาจ start ไม่ขึ้น ให้ตรวจด้วย:

```bash
sudo systemctl status docker
sudo journalctl -u docker -n 50
```

## Push image

```bash
docker tag simple-api:1.0.0 devops-control:5000/simple-api:1.0.0
docker push devops-control:5000/simple-api:1.0.0
curl http://devops-control:5000/v2/_catalog
```

Docker image ที่จะ push เข้า registry ต้องมีชื่อเต็มตามรูปแบบ:

```text
registry-host:port/repository:tag
```

ในตัวอย่าง:

```text
devops-control:5000/simple-api:1.0.0
^ registry         ^ repo      ^ tag
```

`docker tag` ไม่ได้ copy image ใหม่ทั้งก้อน แต่เพิ่มชื่อ/tag อีกชื่อให้ image เดิม จากนั้น `docker push` จึงส่ง image ไป registry

ตรวจ image ก่อน push:

```bash
docker images | grep simple-api
```

หลัง push แล้ว `_catalog` จะเห็น repository:

```json
{"repositories":["simple-api"]}
```

ถ้าต้องดู tag ของ repository:

```bash
curl http://devops-control:5000/v2/simple-api/tags/list
```

ตัวอย่าง error ที่พบบ่อย:

```text
http: server gave HTTP response to HTTPS client
```

แปลว่า Docker client ยังไม่ได้ตั้ง insecure registry หรือ restart Docker หลังตั้งค่า

```text
lookup devops-control: no such host
```

แปลว่า hostname resolve ไม่ได้ ให้ตรวจ `/etc/hosts`

```text
connection refused
```

แปลว่าไปถึง host แล้วแต่ registry ไม่ได้ listen port 5000 หรือ container registry ไม่ทำงาน

## Pull image

```bash
docker pull devops-control:5000/simple-api:1.0.0
```

การ pull ทดสอบว่า client เครื่องอื่นดึง image จาก registry ได้จริง ก่อนนำไปใช้ใน CI/CD หรือ Kubernetes

เพื่อทดสอบให้ชัด สามารถลบ image local ก่อนแล้ว pull ใหม่:

```bash
docker rmi devops-control:5000/simple-api:1.0.0
docker pull devops-control:5000/simple-api:1.0.0
```

แล้วลอง run:

```bash
docker run --rm -p 3000:3000 devops-control:5000/simple-api:1.0.0
```

ถ้า pull ได้แต่ run ไม่ได้ ปัญหาไม่ได้อยู่ที่ registry แล้ว ให้กลับไปตรวจ Dockerfile/app log

## การตรวจสอบ

ตรวจจาก `devops-control`:

```bash
docker ps
docker logs registry
curl http://localhost:5000/v2/_catalog
curl http://devops-control:5000/v2/_catalog
```

ตรวจจาก client เช่น `app-server`:

```bash
getent hosts devops-control
curl http://devops-control:5000/v2/_catalog
docker info | grep -A 5 "Insecure Registries"
docker pull devops-control:5000/simple-api:1.0.0
```

ถ้า `curl` ผ่านแต่ `docker pull` ไม่ผ่าน มักเกี่ยวกับ Docker insecure registry config ถ้า `curl` ไม่ผ่าน ให้กลับไปตรวจ network, DNS/hosts, firewall หรือ registry container

## ข้อสรุป

Registry เป็นจุดกลางของ workflow: CI/CD build image แล้ว push ไป registry จากนั้น server หรือ Kubernetes pull image ไป run

สิ่งที่ต้องจำ:

```text
build image ได้บนเครื่องเดียว ไม่พอสำหรับระบบหลายเครื่อง
ต้อง push image ไป registry กลาง
ทุกเครื่องที่ต้องใช้ image ต้อง pull จาก registry ได้
ถ้า registry ใช้ HTTP ต้องตั้ง insecure registry บน client ทุกเครื่อง
```

บทนี้จะถูกใช้ต่อใน GitLab CI/CD และ Kubernetes เพราะทั้ง pipeline และ cluster ต้องอ้าง image ด้วยชื่อแบบ `devops-control:5000/simple-api:<tag>`

---
