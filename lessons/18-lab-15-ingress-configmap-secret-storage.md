# 18. Lab 15: Ingress, ConfigMap, Secret และ Storage

## วัตถุประสงค์

- เปิด app จากนอก cluster ด้วย Ingress
- แยก config ด้วย ConfigMap
- เก็บข้อมูลลับด้วย Secret
- ใช้ PersistentVolume/PersistentVolumeClaim พื้นฐาน

## ติดตั้ง Nginx Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/baremetal/deploy.yaml
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

## ConfigMap

```bash
nano simple-api-configmap.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: simple-api-config
  namespace: devops-lab
data:
  APP_ENV: "lab"
  LOG_LEVEL: "debug"
```

```bash
kubectl apply -f simple-api-configmap.yaml
```

## Secret

```bash
kubectl create secret generic simple-api-secret \
  -n devops-lab \
  --from-literal=DB_PASSWORD=apppassword
```

ตรวจสอบ:

```bash
kubectl get secret -n devops-lab
kubectl describe secret simple-api-secret -n devops-lab
```

## Ingress

```bash
nano simple-api-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-api
  namespace: devops-lab
spec:
  ingressClassName: nginx
  rules:
    - host: simple-api.lab.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: simple-api
                port:
                  number: 80
```

```bash
kubectl apply -f simple-api-ingress.yaml
```

เพิ่ม hosts บนเครื่องที่ใช้ทดสอบ:

```text
192.168.56.101 simple-api.lab.local
```

ทดสอบ:

```bash
curl http://simple-api.lab.local
```

## Storage แบบ hostPath สำหรับ Lab

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lab-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/lab-data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lab-pvc
  namespace: devops-lab
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

## ข้อสรุป

ConfigMap ใช้กับค่าที่ไม่ลับ, Secret ใช้กับข้อมูลลับ, Ingress รับ traffic จากภายนอก และ Storage ทำให้ข้อมูลไม่หายเมื่อ pod ถูกลบ

---
