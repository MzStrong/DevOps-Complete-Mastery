# 10. Lab 07: Private Container Registry

## วัตถุประสงค์

- สร้าง registry ส่วนตัว
- push/pull image ระหว่าง VM
- เตรียม registry สำหรับ CI/CD และ Kubernetes

## Run Registry บน devops-control

```bash
docker run -d --name registry --restart always -p 5000:5000 -v registry_data:/var/lib/registry registry:2
curl http://localhost:5000/v2/_catalog
```

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

Restart:

```bash
sudo systemctl restart docker
```

## Push image

```bash
docker tag simple-api:1.0.0 devops-control:5000/simple-api:1.0.0
docker push devops-control:5000/simple-api:1.0.0
curl http://devops-control:5000/v2/_catalog
```

## Pull image

```bash
docker pull devops-control:5000/simple-api:1.0.0
```

## ข้อสรุป

Registry เป็นจุดกลางของ workflow: CI/CD build image แล้ว push ไป registry จากนั้น server หรือ Kubernetes pull image ไป run

---
