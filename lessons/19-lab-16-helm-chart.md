# 19. Lab 16: Helm Chart

Lab นี้นำ manifest หลายไฟล์ของ Kubernetes มาจัดเป็น package ด้วย Helm เพื่อให้ deploy, upgrade และ rollback ได้ง่ายขึ้น เมื่อระบบเริ่มมี Deployment, Service, Ingress, ConfigMap และ Secret หลายไฟล์ การ apply YAML ทีละไฟล์จะเริ่มซ้ำและผิดพลาดง่าย Helm ช่วยแยก template ออกจากค่าที่เปลี่ยนตาม environment

## วัตถุประสงค์

- ลดความซ้ำของ Kubernetes YAML
- แยก values ตาม environment
- upgrade และ rollback ง่ายขึ้น

แนวคิดหลัก:

```text
Chart = package ของ Kubernetes manifests
templates/ = YAML template
values.yaml = ค่า default
release = chart ที่ถูก install เข้า cluster แล้ว
revision = version การ upgrade ของ release
```

Helm ไม่ได้แทน Kubernetes แต่เป็นเครื่องมือสร้างและจัดการ Kubernetes manifest ให้เป็นระบบมากขึ้น

## ติดตั้ง Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

คำสั่งนี้ติดตั้ง Helm CLI บนเครื่องที่ใช้ `kubectl` ได้อยู่แล้ว Helm จะใช้ kubeconfig เดียวกับ `kubectl` เพื่อคุยกับ cluster

ตรวจว่า Helm เห็น cluster:

```bash
kubectl get nodes
helm version
```

ถ้า `kubectl` ยังใช้ไม่ได้ Helm ก็ deploy เข้า cluster ไม่ได้เช่นกัน

## สร้าง Chart

```bash
helm create simple-api-chart
cd simple-api-chart
```

`helm create` จะสร้าง chart template เริ่มต้นให้ ซึ่งมีไฟล์มากกว่าที่ใช้จริงใน lab ได้ หลังสร้าง chart ควรเปิดดูโครงสร้างและลบ/ปรับ template ที่ไม่ต้องใช้เพื่อให้เข้าใจง่าย

โครงสร้าง:

```text
simple-api-chart/
├── Chart.yaml
├── values.yaml
└── templates/
```

ไฟล์สำคัญ:

- `Chart.yaml` metadata ของ chart เช่น name, version, appVersion
- `values.yaml` ค่า default ที่ template จะอ่าน
- `templates/` เก็บ Kubernetes YAML template

ใน template จะมี syntax เช่น:

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

หมายถึงอ่านค่าจาก `values.yaml` มาสร้าง YAML จริงก่อนส่งให้ Kubernetes

ตัวอย่าง `values.yaml`:

```yaml
replicaCount: 2

image:
  repository: devops-control:5000/simple-api
  tag: "1.0.0"

service:
  type: ClusterIP
  port: 80

containerPort: 3000

ingress:
  enabled: true
  className: nginx
  host: simple-api.lab.local
```

เพื่อให้ chart นี้ต่อจาก Lab 14-15 ได้จริง ให้แก้ template หลักให้สร้าง resource ชื่อเดียวกับบทก่อนหน้า ถ้าใช้ไฟล์ที่ `helm create` สร้างมา ให้ลบ template ที่ไม่ใช้ก่อน เช่น `templates/tests/` และปรับ `templates/deployment.yaml`, `templates/service.yaml`, `templates/ingress.yaml` ให้เรียบง่ายตามตัวอย่างด้านล่าง

ตัวอย่าง `templates/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-api
  namespace: {{ .Release.Namespace }}
spec:
  replicas: {{ .Values.replicaCount }}
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
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.containerPort }}
          readinessProbe:
            httpGet:
              path: /health
              port: {{ .Values.containerPort }}
          livenessProbe:
            httpGet:
              path: /health
              port: {{ .Values.containerPort }}
```

ตัวอย่าง `templates/service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: simple-api
  namespace: {{ .Release.Namespace }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: simple-api
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.containerPort }}
```

ตัวอย่าง `templates/ingress.yaml`:

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-api
  namespace: {{ .Release.Namespace }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: simple-api
                port:
                  number: {{ .Values.service.port }}
{{- end }}
```

ตัวอย่างนี้ตั้งชื่อ Deployment, Service และ Ingress เป็น `simple-api` เพื่อให้คำสั่งจากบทก่อน เช่น `kubectl rollout status deployment/simple-api -n devops-lab` ใช้ต่อได้ทันที

ค่าพวกนี้คือสิ่งที่มักเปลี่ยนตาม environment:

- `replicaCount` dev อาจใช้ 1, prod อาจใช้ 3+
- `image.tag` เปลี่ยนตาม release
- `ingress.host` dev/uat/prod ใช้ hostname ต่างกัน
- resource limit, config หรือ secret name อาจต่างกัน

ตัวอย่างที่ผิด:

```text
hardcode image tag, host, replica ใน templates ทุกไฟล์
```

ผลคือแก้หลายจุด เสี่ยงแก้ไม่ครบ และ CI/CD override ค่าได้ยาก

ตัวอย่างที่ถูก:

```text
ใส่ค่าที่เปลี่ยนบ่อยใน values.yaml
ให้ template อ่านจาก .Values
```

## ตรวจ Chart ก่อนติดตั้ง

ก่อน install ควรตรวจ:

```bash
helm lint .
helm template simple-api . -n devops-lab
```

- `helm lint` ตรวจความถูกต้องพื้นฐานของ chart
- `helm template` render template ออกมาเป็น YAML ให้ดู โดยยังไม่ apply เข้า cluster

ถ้า YAML ที่ render ออกมาไม่ถูกต้อง ให้แก้ chart ก่อน install เพราะ error จาก Kubernetes หลัง apply แล้วจะตามแก้ยากกว่า

ตรวจแบบ dry-run:

```bash
helm install simple-api . -n devops-lab --dry-run --debug
```

คำสั่งนี้จำลอง install และแสดง manifest ที่ Helm จะส่งเข้า cluster

## ติดตั้ง / Upgrade / Rollback

```bash
helm install simple-api . -n devops-lab
helm list -n devops-lab
helm upgrade simple-api . -n devops-lab --set image.tag=1.0.1
helm history simple-api -n devops-lab
helm rollback simple-api 1 -n devops-lab
```

ความหมาย:

- `helm install` สร้าง release ใหม่ชื่อ `simple-api`
- `helm list` ดู release ใน namespace
- `helm upgrade` เปลี่ยน release เดิม เช่นเปลี่ยน image tag
- `helm history` ดู revision ของ release
- `helm rollback` ย้อน release ไป revision ที่เลือก

ถ้า namespace ยังไม่มี ให้สร้างก่อน:

```bash
kubectl create namespace devops-lab
```

หรือใช้:

```bash
helm install simple-api . -n devops-lab --create-namespace
```

หลัง upgrade ให้ตรวจ:

```bash
helm status simple-api -n devops-lab
kubectl get all -n devops-lab
kubectl rollout status deployment/simple-api -n devops-lab
```

ข้อควรระวัง: Helm rollback ย้อน manifest/config ของ release ได้ แต่ไม่ได้กู้ข้อมูลใน database หรือ storage ให้ ถ้า release ใหม่ทำให้ข้อมูลเสีย ต้องใช้ backup/restore เพิ่ม

## แยก environment

```text
values-dev.yaml
values-uat.yaml
values-prod.yaml
```

Deploy:

```bash
helm upgrade --install simple-api . -n devops-lab -f values-dev.yaml
```

การแยก values ตาม environment ทำให้ chart เดียวใช้ได้หลายที่:

```text
values-dev.yaml   -> replica น้อย, debug log, dev host
values-uat.yaml   -> ใกล้ prod แต่ใช้ test data
values-prod.yaml  -> replica มาก, resource limit จริง, prod host
```

ตัวอย่าง:

```yaml
# values-dev.yaml
replicaCount: 1
image:
  tag: "dev"
ingress:
  host: simple-api-dev.lab.local
```

```yaml
# values-prod.yaml
replicaCount: 3
image:
  tag: "1.0.0"
ingress:
  host: simple-api.lab.local
```

ใน CI/CD มักใช้คำสั่ง:

```bash
helm upgrade --install simple-api ./helm/simple-api \
  -n devops-lab \
  -f values-dev.yaml \
  --set image.tag=$CI_COMMIT_SHORT_SHA
```

ทำให้ pipeline deploy image tag ที่ build จาก commit นั้นได้

## Debug Helm

คำสั่งที่ควรรู้:

```bash
helm list -n devops-lab
helm status simple-api -n devops-lab
helm history simple-api -n devops-lab
helm get values simple-api -n devops-lab
helm get manifest simple-api -n devops-lab
helm uninstall simple-api -n devops-lab
```

อาการที่พบบ่อย:

```text
template render fail
-> syntax template หรือ values ขาด

install fail เพราะ resource already exists
-> มี resource เดิมที่ไม่ได้ถูก Helm จัดการ

upgrade ผ่านแต่ pod ไม่รัน
-> ตรวจ kubectl describe/logs เหมือนบท Deployment

rollback แล้ว app ยังพัง
-> อาจเป็นข้อมูล/config ภายนอก chart ไม่ได้ rollback
```

หลักคิดคือ Helm จัดการ manifest แต่เมื่อ manifest ถูก apply แล้ว runtime behavior ยังต้อง debug ด้วย `kubectl` อยู่ดี

## ข้อสรุป

Helm ทำให้ deployment บน Kubernetes เป็น package ที่ reusable และเหมาะกับ CI/CD มากกว่า apply YAML ทีละหลายไฟล์

สิ่งที่ควรจำ:

```text
Chart รวม template
values แยกค่าตาม environment
release คือสิ่งที่ install แล้ว
revision ใช้ history/rollback
helm template/lint ช่วยตรวจ ก่อน deploy จริง
```

เมื่อใช้ Helm คล่องแล้ว deployment ใน CI/CD จะสะอาดขึ้น เพราะ pipeline แค่เลือก values และ image tag ที่ต้องการ deploy

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [18. Lab 15: Ingress, ConfigMap, Secret และ Storage](./18-lab-15-ingress-configmap-secret-storage.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [20. Lab 17: DevSecOps ด้วย Trivy](./20-lab-17-devsecops-trivy.md)

<!-- lesson-nav:end -->
