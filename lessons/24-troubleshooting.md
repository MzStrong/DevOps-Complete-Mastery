# 24. Troubleshooting รวม

## SSH เข้าไม่ได้

ตรวจสอบ:

```bash
sudo systemctl status ssh
ip a
sudo ufw status
ping <target-ip>
```

สาเหตุที่พบบ่อย:

- IP ผิด
- SSH service ไม่ทำงาน
- firewall block port 22
- VMware network mode ไม่ตรงกัน

## Docker permission denied

```bash
sudo usermod -aG docker $USER
newgrp docker
```

หรือ logout/login ใหม่

## Container ออกทันที

```bash
docker ps -a
docker logs <container-name>
```

สาเหตุ:

- command ผิด
- app crash
- env variable หาย
- port conflict

## Nginx config error

```bash
sudo nginx -t
sudo tail -f /var/log/nginx/error.log
```

## Kubernetes node NotReady

```bash
kubectl get nodes
kubectl describe node <node-name>
sudo systemctl status kubelet
sudo journalctl -u kubelet -f
```

สาเหตุ:

- CNI ยังไม่พร้อม
- swap ยังเปิดอยู่
- containerd config ผิด
- firewall block port

## Pod CrashLoopBackOff

```bash
kubectl logs -n <namespace> <pod-name>
kubectl describe pod -n <namespace> <pod-name>
```

สาเหตุ:

- app start ไม่สำเร็จ
- config/env ผิด
- secret ไม่มี
- database connect ไม่ได้

## ImagePullBackOff

```bash
kubectl describe pod -n <namespace> <pod-name>
```

สาเหตุ:

- image name/tag ผิด
- registry เข้าไม่ได้
- private registry ไม่มี credential
- insecure registry ยังไม่ได้ config

## Prometheus target Down

```bash
curl http://target-host:9100/metrics
```

สาเหตุ:

- node_exporter ไม่ทำงาน
- firewall block port 9100
- hostname resolve ไม่ได้
- prometheus.yml target ผิด

---
