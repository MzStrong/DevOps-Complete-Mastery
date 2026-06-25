# 17. Lab 14: Deploy App เข้า Kubernetes

Lab นี้นำ image ที่เคย build และ push เข้า private registry ไป deploy บน Kubernetes สิ่งที่ต้องเข้าใจคือ Kubernetes ไม่ได้รัน container เดี่ยว ๆ แบบ `docker run` แต่ใช้ resource เช่น Deployment และ Service เพื่อกำหนด desired state, จำนวน replica, health check, rollout และ endpoint ที่เรียกได้คงที่

## วัตถุประสงค์

- สร้าง Namespace, Deployment, Service
- Scale app
- Rollout และ rollback

ภาพรวม flow:

```text
Deployment
-> สร้าง ReplicaSet
-> สร้าง Pod
-> Pod pull image จาก devops-control:5000
-> container รัน simple-api ที่ port 3000
-> Service ชื่อ simple-api ส่ง traffic ไป pod
```

ถ้า image pull ไม่ได้ pod จะไม่รัน ถ้า app start แล้ว crash pod จะเข้า CrashLoopBackOff ถ้า Service selector ไม่ตรงกับ label ของ pod จะเรียก service ไม่เจอ backend

## สร้าง Namespace

```bash
kubectl create namespace devops-lab
```

Namespace ใช้แยก resource ใน cluster ให้เป็นกลุ่ม เช่น lab, dev, uat หรือ prod ในบทนี้ใช้ `devops-lab` เพื่อไม่ปนกับ resource ระบบใน namespace อื่น

ตรวจ:

```bash
kubectl get namespace
```

ถ้าสร้างซ้ำแล้ว error ว่ามีอยู่แล้ว ไม่ใช่ปัญหา สามารถใช้ namespace เดิมต่อได้

## Deployment

```bash
nano simple-api-deployment.yaml
```

Deployment ใช้กำหนดว่า app ควรรันกี่ replica, ใช้ image อะไร, เปิด port อะไร, ตรวจ health อย่างไร และจำกัด resource เท่าไร

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-api
  namespace: devops-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple-api
  template:
    metadata:
      labels:
        app: simple-api
    spec:
      containers:
        - name: simple-api
          image: devops-control:5000/simple-api:1.0.0
          ports:
            - containerPort: 3000
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

อธิบายส่วนสำคัญ:

- `metadata.name` คือชื่อ Deployment
- `metadata.namespace` ระบุว่าจะสร้างใน namespace ใด
- `replicas: 2` ต้องการ pod 2 ตัว
- `selector.matchLabels` ใช้เลือก pod ที่ Deployment คุม
- `template.metadata.labels` label ที่ติดกับ pod ต้องตรงกับ selector
- `image` คือ image จาก private registry
- `containerPort: 3000` port ที่ app listen ใน container
- `readinessProbe` ใช้บอกว่า pod พร้อมรับ traffic หรือยัง
- `livenessProbe` ใช้บอกว่า container ยังมีชีวิตดีหรือควร restart
- `resources.requests` คือ resource ขั้นต่ำที่ scheduler ใช้วาง pod
- `resources.limits` คือเพดานสูงสุดที่ container ใช้ได้

ข้อสำคัญเรื่อง label:

```yaml
selector:
  matchLabels:
    app: simple-api
template:
  metadata:
    labels:
      app: simple-api
```

สองส่วนนี้ต้องตรงกัน ถ้าไม่ตรง Deployment จะคุม pod ไม่ถูก และ Service อาจหา pod ไม่เจอ

เรื่อง probe:

```text
readinessProbe fail -> pod ยังไม่รับ traffic จาก Service
livenessProbe fail  -> kubelet restart container
```

ถ้า path `/health` ไม่มีอยู่จริงหรือ app listen port ผิด pod อาจไม่ ready หรือ restart วน

เรื่อง image:

```text
devops-control:5000/simple-api:1.0.0
```

ทุก Kubernetes node ต้อง resolve `devops-control` ได้ และ container runtime ต้อง pull จาก registry นี้ได้ ถ้ายังไม่ได้ config insecure registry บน worker node จะเจอ ImagePullBackOff

Apply:

```bash
kubectl apply -f simple-api-deployment.yaml
```

หลัง apply ให้ตรวจทันที:

```bash
kubectl get deployment -n devops-lab
kubectl get pods -n devops-lab -o wide
kubectl describe deployment simple-api -n devops-lab
```

ถ้า pod ไม่ขึ้น Running ให้ดู:

```bash
kubectl describe pod -n devops-lab <pod-name>
kubectl logs -n devops-lab <pod-name>
```

## Service

```bash
nano simple-api-service.yaml
```

Pod มี IP ที่เปลี่ยนได้เมื่อถูกสร้างใหม่ Service จึงทำหน้าที่ให้ชื่อและ virtual IP คงที่เพื่อเรียก app โดยไม่ต้องรู้ IP ของ pod

```yaml
apiVersion: v1
kind: Service
metadata:
  name: simple-api
  namespace: devops-lab
spec:
  selector:
    app: simple-api
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

อธิบาย:

- `selector.app: simple-api` เลือก pod ที่มี label `app=simple-api`
- `port: 80` port ของ Service
- `targetPort: 3000` port ของ container ที่ Service ส่ง traffic ไป
- `type: ClusterIP` เปิดให้เรียกได้ภายใน cluster

ถ้า selector ไม่ตรงกับ label ของ pod Service จะถูกสร้างได้ แต่ไม่มี endpoint

ตรวจ endpoint:

```bash
kubectl get endpoints simple-api -n devops-lab
```

ควรเห็น IP ของ pod ถ้า endpoints ว่าง ให้ตรวจ label:

```bash
kubectl get pods -n devops-lab --show-labels
```

Apply:

```bash
kubectl apply -f simple-api-service.yaml
```

## ทดสอบใน cluster

```bash
kubectl run test-curl -n devops-lab --rm -it --image=curlimages/curl -- sh
curl http://simple-api/health
```

คำสั่งนี้สร้าง temporary pod จาก image `curlimages/curl` ใน namespace `devops-lab` เพื่อทดสอบเรียก Service จากภายใน cluster

ใน pod test นี้ DNS ของ Kubernetes จะ resolve `simple-api` เป็น Service ใน namespace เดียวกัน ดังนั้นไม่ต้องรู้ ClusterIP

ถ้า curl ไม่ผ่าน ให้ไล่ตรวจ:

```bash
kubectl get svc -n devops-lab
kubectl get endpoints simple-api -n devops-lab
kubectl get pods -n devops-lab -o wide
kubectl logs -n devops-lab <simple-api-pod>
```

ถ้า endpoints มี pod แต่ curl fail อาจเป็น app port/path ผิด ถ้า endpoints ว่าง selector/label มักผิด

## Scale

```bash
kubectl scale deployment simple-api -n devops-lab --replicas=4
kubectl get pods -n devops-lab -o wide
```

Scale คือการเปลี่ยนจำนวน desired replicas จาก 2 เป็น 4 Kubernetes จะสร้าง pod เพิ่มให้เอง ถ้ามี worker node หลายตัว scheduler อาจกระจาย pod ไปคนละ node ตาม resource ที่เหลือ

ตรวจ:

```bash
kubectl get deployment simple-api -n devops-lab
kubectl get pods -n devops-lab -o wide
```

ถ้า pod ใหม่ Pending ให้ดู:

```bash
kubectl describe pod -n devops-lab <pod-name>
```

สาเหตุอาจเป็น resource ไม่พอ, node NotReady หรือ image pull ไม่ได้

## Rollout/Rollback

```bash
kubectl set image deployment/simple-api simple-api=devops-control:5000/simple-api:1.0.1 -n devops-lab
kubectl rollout status deployment/simple-api -n devops-lab
kubectl rollout undo deployment/simple-api -n devops-lab
```

Rollout คือการเปลี่ยน version ของ workload เช่นเปลี่ยน image tag จาก `1.0.0` เป็น `1.0.1` Kubernetes จะค่อย ๆ สร้าง pod version ใหม่และลด pod version เก่า

ตรวจ history:

```bash
kubectl rollout history deployment/simple-api -n devops-lab
```

ถ้า rollout ใหม่มีปัญหา เช่น image pull ไม่ได้หรือ app crash:

```bash
kubectl rollout status deployment/simple-api -n devops-lab
kubectl rollout undo deployment/simple-api -n devops-lab
```

ข้อควรระวัง: ถ้าใช้ image tag เดิมซ้ำ เช่น `latest` Kubernetes และ node cache อาจทำให้ไม่รู้ว่ารัน version ไหน ควรใช้ tag ที่เปลี่ยนตาม release หรือ commit

## ตรวจสอบ

```bash
kubectl get all -n devops-lab
kubectl describe pod -n devops-lab <pod-name>
kubectl logs -n devops-lab <pod-name>
```

คำสั่งที่ควรใช้บ่อย:

```bash
kubectl get deployment -n devops-lab
kubectl get rs -n devops-lab
kubectl get pods -n devops-lab -o wide
kubectl get svc,endpoints -n devops-lab
kubectl describe pod -n devops-lab <pod-name>
kubectl logs -n devops-lab <pod-name>
kubectl logs -n devops-lab <pod-name> --previous
```

`--previous` มีประโยชน์เมื่อ container crash แล้ว restart เพราะใช้ดู log ของ container รอบก่อนหน้า

## Debug อาการที่พบบ่อย

```text
ImagePullBackOff
-> image name/tag ผิด
-> worker node pull registry ไม่ได้
-> insecure registry ยังไม่ได้ config
-> devops-control resolve ไม่ได้

CrashLoopBackOff
-> app start แล้ว crash
-> config/env ผิด
-> port หรือ command ผิด
-> ดู kubectl logs และ logs --previous

Pod Pending
-> resource ไม่พอ
-> node NotReady
-> scheduler วาง pod ไม่ได้

Service เรียกไม่ได้
-> selector ไม่ตรง label pod
-> endpoints ว่าง
-> targetPort ผิด
```

เริ่ม debug ด้วย `kubectl describe pod` เพราะ Events ด้านล่างมักบอกสาเหตุชัดเจน เช่น pull image fail, probe fail หรือ scheduling fail

## ข้อสรุป

Deployment คุมจำนวน pod และ rollout ส่วน Service ทำให้ app ถูกเรียกด้วยชื่อคงที่ แม้ pod จะเปลี่ยน IP

สิ่งที่ควรจำ:

```text
Deployment = desired state ของ pod
Pod = instance ที่รัน container
Service = endpoint คงที่สำหรับเรียก pod
label/selector = ตัวเชื่อม Service กับ Pod
probe = ตรวจ health
requests/limits = กำหนด resource
rollout/rollback = คุม version
```

หลังบทนี้ app รันใน Kubernetes ได้แล้ว แต่ยังเข้าจากภายนอก cluster ได้ไม่สะดวก บทถัดไปจะเพิ่ม Ingress, ConfigMap, Secret และ Storage

---
