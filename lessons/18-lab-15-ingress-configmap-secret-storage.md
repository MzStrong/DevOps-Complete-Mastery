# 18. Lab 15: Ingress, ConfigMap, Secret และ Storage

Lab นี้เพิ่มส่วนที่ทำให้ workload บน Kubernetes ใช้งานใกล้เคียงระบบจริงขึ้น ได้แก่การเปิด app จากภายนอก cluster, การแยก config ออกจาก image, การจัดการข้อมูลลับ และการเตรียม storage ที่อยู่รอดเมื่อ pod ถูกลบ

## วัตถุประสงค์

- เปิด app จากนอก cluster ด้วย Ingress
- แยก config ด้วย ConfigMap
- เก็บข้อมูลลับด้วย Secret
- ใช้ PersistentVolume/PersistentVolumeClaim พื้นฐาน

ภาพรวม:

```text
client
-> simple-api.lab.local
-> Ingress Controller
-> Ingress rule
-> Service simple-api
-> Pod simple-api

ConfigMap/Secret
-> ส่งค่า config/env เข้า pod

PV/PVC
-> ให้ pod ใช้ storage ถาวร
```

บทก่อนหน้า Service แบบ `ClusterIP` เรียกได้จากใน cluster เป็นหลัก บทนี้เพิ่ม Ingress เพื่อให้เรียกจากเครื่องทดสอบภายนอก cluster ด้วย hostname ได้

## ติดตั้ง Nginx Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/baremetal/deploy.yaml
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

Ingress resource เป็นแค่ rule แต่ต้องมี Ingress Controller คอยอ่าน rule แล้วรับ traffic จริง ในบทนี้ใช้ Nginx Ingress Controller แบบ bare metal

หลังติดตั้งให้ตรวจ:

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
kubectl get ingressclass
```

pod ใน namespace `ingress-nginx` ควรเป็น `Running` ถ้า controller ไม่พร้อม Ingress resource จะถูกสร้างได้ แต่ไม่มีใครรับ traffic ให้

ใน bare metal lab service ของ ingress controller อาจเป็น NodePort ไม่ใช่ LoadBalancer ที่มี external IP อัตโนมัติ ดังนั้นเวลาทดสอบต้องดูว่า traffic ควรเข้าที่ IP/port ใดของ node

ถ้าใช้ NodePort ให้ดู:

```bash
kubectl get svc -n ingress-nginx
```

แล้วตรวจ port ที่ expose เช่น 80 อาจถูก map เป็น node port เช่น 30xxx ขึ้นกับ manifest และ environment

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

ConfigMap ใช้เก็บค่าที่ไม่ลับ เช่น environment name, log level, feature flag หรือ endpoint ที่ไม่ sensitive ข้อดีคือไม่ต้อง build image ใหม่ทุกครั้งเมื่อเปลี่ยน config

```bash
kubectl apply -f simple-api-configmap.yaml
```

ตรวจ:

```bash
kubectl get configmap -n devops-lab
kubectl describe configmap simple-api-config -n devops-lab
```

ตัวอย่างการนำ ConfigMap เข้า Deployment:

```yaml
envFrom:
  - configMapRef:
      name: simple-api-config
```

หรือเลือกเฉพาะ key:

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: simple-api-config
        key: APP_ENV
```

ถ้าแก้ ConfigMap แล้ว pod ไม่เห็นค่าใหม่ทันที อาจต้อง restart/rollout pod ขึ้นกับวิธี mount/env ที่ใช้

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

Secret ใช้กับข้อมูลลับ เช่น password, token, key แต่ต้องเข้าใจว่า Kubernetes Secret ค่า default เป็น base64 encoding ไม่ใช่ encryption เต็มรูปแบบ ถ้าใช้ production ต้องจัดการ RBAC, encryption at rest และอาจใช้เครื่องมืออย่าง External Secrets หรือ Vault

ดูค่าแบบ decode เฉพาะใน lab:

```bash
kubectl get secret simple-api-secret -n devops-lab -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

ตัวอย่างการนำ Secret เข้า Deployment:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: simple-api-secret
        key: DB_PASSWORD
```

ตัวอย่างที่ผิด:

```yaml
env:
  - name: DB_PASSWORD
    value: apppassword
```

เพราะ password ถูกเขียนตรงใน Deployment manifest และมีโอกาสถูก commit ลง Git

ตัวอย่างที่ถูก:

```yaml
valueFrom:
  secretKeyRef:
    name: simple-api-secret
    key: DB_PASSWORD
```

## Ingress

```bash
nano simple-api-ingress.yaml
```

Ingress คือ rule สำหรับรับ HTTP/HTTPS จากภายนอกแล้ว route ไป Service ภายใน cluster โดยอิง host/path

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

อธิบาย:

- `ingressClassName: nginx` บอกให้ Nginx Ingress Controller จัดการ Ingress นี้
- `host: simple-api.lab.local` รับ request ที่ host นี้
- `path: /` รับทุก path ใต้ `/`
- backend ชี้ไป Service `simple-api` port 80

ตรวจ:

```bash
kubectl get ingress -n devops-lab
kubectl describe ingress simple-api -n devops-lab
kubectl get svc,endpoints -n devops-lab
```

ถ้า Ingress มี rule ถูก แต่ Service ไม่มี endpoint traffic จะเข้า controller ได้แต่ไปไม่ถึง pod

เพิ่ม hosts บนเครื่องที่ใช้ทดสอบ:

```text
192.168.56.101 simple-api.lab.local
```

บรรทัดนี้ต้องใส่บนเครื่องที่ใช้เปิด browser หรือรัน curl ไม่ใช่เฉพาะใน Kubernetes node เท่านั้น IP ที่ใช้ต้องเป็น IP ของ node หรือ endpoint ที่ ingress controller รับ traffic ได้จริง

ตัวอย่างที่ผิด:

```text
192.168.56.100 simple-api.lab.local
```

ถ้า ingress controller ไม่ได้ expose อยู่บน master IP นี้ request จะไม่ถึง ingress

ตัวอย่างที่ถูก:

```text
<IP ที่เข้าถึง ingress controller ได้> simple-api.lab.local
```

ใน lab อาจเป็น worker node IP หรือ node ที่ NodePort/Ingress รับ traffic อยู่

ทดสอบ:

```bash
curl http://simple-api.lab.local
```

ถ้าใช้ NodePort และ port ไม่ใช่ 80 อาจต้องระบุ port:

```bash
curl http://simple-api.lab.local:<node-port>
```

debug ingress:

```bash
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <ingress-controller-pod>
kubectl describe ingress simple-api -n devops-lab
curl -H "Host: simple-api.lab.local" http://<node-ip>
```

การใช้ `-H "Host: ..."` ช่วยทดสอบ Ingress rule แม้ DNS/hosts ยังไม่ได้ตั้ง

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

Storage ใน Kubernetes แบ่งเป็นสองฝั่ง:

- `PersistentVolume` คือ storage จริงที่ cluster มีให้
- `PersistentVolumeClaim` คือคำขอใช้ storage จาก namespace/app

`hostPath` ใช้ path บน node จริง เหมาะกับ lab เพื่อเข้าใจ concept แต่ไม่เหมาะกับ production ทั่วไป เพราะ pod จะผูกกับ node ที่มี path นั้น ถ้า pod ย้ายไป node อื่นข้อมูลอาจไม่อยู่

ก่อนใช้ hostPath ให้สร้าง directory บน node ที่จะใช้:

```bash
sudo mkdir -p /mnt/lab-data
sudo chmod 777 /mnt/lab-data
```

สำหรับ lab ใช้ `chmod 777` เพื่อให้ง่าย แต่ production ควรจัด owner/permission ให้เหมาะสมกว่า

Apply:

```bash
nano lab-storage.yaml
kubectl apply -f lab-storage.yaml
```

ตรวจ:

```bash
kubectl get pv
kubectl get pvc -n devops-lab
kubectl describe pvc lab-pvc -n devops-lab
```

PVC ควรเป็น `Bound` ถ้ายัง `Pending` ให้ตรวจว่า PV capacity/accessModes ตรงกับ PVC หรือไม่

ตัวอย่างการ mount PVC เข้า pod:

```yaml
volumes:
  - name: lab-storage
    persistentVolumeClaim:
      claimName: lab-pvc
containers:
  - name: simple-api
    volumeMounts:
      - name: lab-storage
        mountPath: /data
```

## Debug อาการที่พบบ่อย

```text
Ingress เข้าไม่ได้
-> ingress controller running ไหม
-> host ใน /etc/hosts ชี้ถูก IP ไหม
-> Ingress class ถูกไหม
-> Service และ endpoints มีไหม

ConfigMap เปลี่ยนแล้ว app ไม่เห็นค่าใหม่
-> pod อาจต้อง restart
-> Deployment อ้าง ConfigMap ถูกชื่อ/key ไหม

Secret ใช้ไม่ได้
-> secret อยู่ namespace เดียวกับ pod ไหม
-> key ตรงไหม
-> Deployment อ้าง secretKeyRef ถูกไหม

PVC Pending
-> PV capacity/accessModes ไม่ match
-> storageClass หรือ hostPath ไม่พร้อม
```

คำสั่งหลัก:

```bash
kubectl describe ingress -n devops-lab simple-api
kubectl describe pod -n devops-lab <pod-name>
kubectl get configmap,secret,pv,pvc -n devops-lab
kubectl get endpoints -n devops-lab simple-api
```

## ข้อสรุป

ConfigMap ใช้กับค่าที่ไม่ลับ, Secret ใช้กับข้อมูลลับ, Ingress รับ traffic จากภายนอก และ Storage ทำให้ข้อมูลไม่หายเมื่อ pod ถูกลบ

สิ่งที่ควรจำ:

```text
Ingress = HTTP routing จากภายนอกเข้า Service
ConfigMap = config ไม่ลับ
Secret = ข้อมูลลับ
PV = storage จริง
PVC = คำขอใช้ storage
hostPath = ดีสำหรับ lab แต่มีข้อจำกัดเรื่อง node
```

หลังบทนี้ app มีองค์ประกอบพื้นฐานที่ระบบจริงควรมีมากขึ้น ขั้นถัดไปคือใช้ Helm เพื่อลดความซ้ำของ manifest และจัดการ release ได้ง่ายขึ้น

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [17. Lab 14: Deploy App เข้า Kubernetes](./17-lab-14-deploy-app-to-kubernetes.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [19. Lab 16: Helm Chart](./19-lab-16-helm-chart.md)

<!-- lesson-nav:end -->
