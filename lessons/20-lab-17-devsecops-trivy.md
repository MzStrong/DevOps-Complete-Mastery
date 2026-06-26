# 20. Lab 17: DevSecOps ด้วย Trivy

Lab นี้เพิ่ม security เข้าไปตั้งแต่ช่วง build/test แทนการรอให้ deploy แล้วค่อยตรวจ Trivy ใช้ scan container image, filesystem และ dependency เพื่อหา vulnerability, secret หรือ misconfiguration บางประเภท จุดสำคัญคือใช้ผล scan เป็นข้อมูลตัดสินใจ ไม่ใช่แค่รัน command แล้วจบ

## วัตถุประสงค์

- Scan image vulnerability
- Scan filesystem/repository
- เพิ่ม security gate ใน pipeline

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Vulnerability | ช่องโหว่ |
| CVE | รหัสช่องโหว่สาธารณะ |
| Severity | ระดับความรุนแรง เช่น LOW, HIGH, CRITICAL |
| Least Privilege | ให้สิทธิ์เท่าที่จำเป็น |

DevSecOps ใน lab นี้หมายถึงการเอา security check เข้าไปใน flow เดิม:

```text
code
-> test
-> build image
-> scan source/image
-> push registry
-> deploy
```

ถ้าพบช่องโหว่ระดับที่รับไม่ได้ pipeline ควร fail ก่อน deploy

## ติดตั้ง Trivy

```bash
sudo apt update
sudo apt install -y wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt update
sudo apt install -y trivy
```

คำสั่งนี้เพิ่ม repository ของ Trivy แล้วติดตั้ง CLI บนเครื่องที่ต้องการ scan เช่น `app-server`, `devops-control` หรือ runner host

ตรวจ:

```bash
trivy --version
```

ครั้งแรกที่ scan Trivy จะดาวน์โหลด vulnerability database ถ้า internet/DNS มีปัญหา scan อาจ fail หรือช้ามาก

ถ้าใช้ใน CI ควรระวังเรื่อง cache database เพื่อไม่ให้ pipeline ช้าเกินไป แต่ใน lab เริ่มจากแบบง่ายก่อน

## Scan image

```bash
trivy image simple-api:1.0.0
trivy image --severity HIGH,CRITICAL simple-api:1.0.0
trivy image --exit-code 1 --severity CRITICAL simple-api:1.0.0
```

`trivy image` ใช้ scan image ที่อยู่ local หรืออยู่ใน registry

ความหมาย:

- คำสั่งแรก scan ทุก severity ที่ Trivy ตรวจพบ
- `--severity HIGH,CRITICAL` แสดงเฉพาะระดับสูงและวิกฤต
- `--exit-code 1` ให้ command fail ถ้าพบช่องโหว่ตามเงื่อนไข ใช้ทำ security gate ใน pipeline

ตัวอย่างการ scan image จาก registry:

```bash
trivy image devops-control:5000/simple-api:1.0.0
```

ถ้า image อยู่ใน insecure registry อาจต้องให้ Docker pull image มาก่อน หรือ config registry ให้เครื่อง scan เข้าถึงได้

การอ่านผล:

```text
Library/Package
Vulnerability ID เช่น CVE-xxxx-xxxx
Severity
Installed Version
Fixed Version
Title/Description
```

ถ้ามี `Fixed Version` ให้ update base image หรือ dependency ไป version ที่แก้แล้ว ถ้าไม่มี fixed version ต้องประเมิน risk หรือใช้ mitigation อื่น

ตัวอย่างที่ผิด:

```text
เจอ LOW 100 รายการแล้ว block deploy ทั้งหมดโดยไม่ดู context
```

อาจทำให้ pipeline ใช้งานจริงไม่ได้และทีมเริ่ม ignore scanner

ตัวอย่างที่ถูก:

```text
เริ่ม gate ที่ CRITICAL ก่อน
ดู HIGH เพิ่มเมื่อทีมพร้อม
มี process triage และ update dependency/base image
```

## Scan source code/filesystem

```bash
trivy fs .
```

`trivy fs .` scan filesystem หรือ repository ปัจจุบัน ใช้ตรวจ dependency file, misconfig และ secret บางประเภท

ควรรันจาก root ของ project เช่น directory ที่มี:

```text
Dockerfile
package.json
package-lock.json
k8s/
helm/
```

ตัวอย่าง command เพิ่มเติม:

```bash
trivy fs --scanners vuln,secret,misconfig .
trivy fs --severity HIGH,CRITICAL .
```

ข้อควรระวัง: secret scanner อาจมี false positive หรือ false negative ได้ จึงยังต้องมี discipline เช่นไม่ commit `.env`, token หรือ private key ตั้งแต่แรก

## ตัวอย่าง GitLab CI

```yaml
security_scan:
  stage: test
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy fs --exit-code 1 --severity HIGH,CRITICAL .
```

job นี้ทำให้ pipeline fail ถ้า source/filesystem มี vulnerability ระดับ HIGH หรือ CRITICAL ตามที่ Trivy ตรวจพบ

ถ้าต้องการ scan image หลัง build:

```yaml
image_scan:
  stage: test
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy image --exit-code 1 --severity CRITICAL $IMAGE_NAME:$IMAGE_TAG
```

ใน pipeline จริงต้องแน่ใจว่า image ถูก build และ push/pull ได้ก่อน scan หรือใช้ Trivy scan local image บน runner ที่เข้าถึง Docker daemon ได้

แนวทางตั้ง gate:

```text
เริ่มต้น: fail เฉพาะ CRITICAL
เมื่อทีมเริ่มแก้ช่องโหว่เป็นระบบ: เพิ่ม HIGH
สำหรับ LOW/MEDIUM: รายงานไว้ก่อน แล้ววางแผนแก้ตามรอบ
```

อย่าใช้ security gate ที่เข้มเกินความสามารถของทีมตั้งแต่วันแรก เพราะจะทำให้ทุกคนหาทาง bypass gate แทนที่จะปรับปรุงระบบ

## ลดช่องโหว่ใน image

แนวทางพื้นฐาน:

- ใช้ base image ที่เล็กและดูแลต่อ เช่น alpine หรือ distroless เมื่อเหมาะสม
- update base image เป็นระยะ
- ไม่ติดตั้ง package ที่ไม่จำเป็น
- ไม่ copy secret เข้า image
- ใช้ non-root user ถ้า app ไม่จำเป็นต้องเป็น root
- pin dependency version และ update ตามรอบ

ตัวอย่าง Dockerfile ที่ควรระวัง:

```dockerfile
FROM node:latest
COPY . .
```

ปัญหา:

- `latest` เปลี่ยนได้ตลอด ทำให้ build ไม่ reproducible
- `COPY . .` อาจ copy `.env`, `.git`, ไฟล์ลับ หรือ node_modules เข้า image

แนวทางที่ดีกว่า:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY index.js ./
CMD ["node", "index.js"]
```

และใช้ `.dockerignore` ตัดไฟล์ที่ไม่ควรเข้า image

## Basic Hardening Checklist

- ไม่เก็บ password/token ใน code
- ไม่ commit `.env`
- ไม่ run container เป็น root ถ้าไม่จำเป็น
- ตั้ง resource limit ให้ container
- เปิด firewall เฉพาะ port ที่จำเป็น
- ให้สิทธิ์แบบ least privilege
- scan dependency และ image ก่อน deploy

เพิ่มเติมสำหรับ Kubernetes:

- ตั้ง `resources.requests/limits`
- ไม่ mount hostPath ถ้าไม่จำเป็น
- ใช้ Secret แทนการ hardcode password
- จำกัด RBAC ตามหน้าที่
- ใช้ image tag ชัดเจน ไม่ใช้ `latest` ใน production
- เปิดเฉพาะ Service/Ingress ที่จำเป็น

## ข้อจำกัดของ scanner

Trivy ช่วยหา known vulnerability และ misconfig บางประเภท แต่ไม่ได้รับประกันว่าระบบปลอดภัยทั้งหมด

สิ่งที่ scanner อาจไม่รู้:

- business logic bug
- auth/authorization design ผิด
- secret ที่ถูกส่งผ่านช่องทางอื่น
- runtime behavior ที่ผิด
- network policy หรือ firewall ที่ตั้งผิดนอก repository

ดังนั้น DevSecOps ต้องใช้หลายอย่างร่วมกัน เช่น code review, dependency update, secret management, least privilege, logging/monitoring และ backup

## ข้อสรุป

DevSecOps คือการทำ security ตั้งแต่ต้นทางใน pipeline ไม่ใช่รอให้ขึ้น production แล้วค่อยตรวจ

สิ่งที่ควรจำ:

```text
scan source
scan image
ตั้ง severity gate อย่างมีเหตุผล
อ่าน fixed version
แก้ที่ base image/dependency/config
ไม่ commit secret
ใช้ least privilege
```

บทนี้ทำให้ pipeline มี security gate เบื้องต้น ขั้นถัดไปคือ backup/restore เพื่อให้ระบบรับมือความเสียหายหรือความผิดพลาดได้

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [19. Lab 16: Helm Chart](./19-lab-16-helm-chart.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [21. Lab 18: Backup, Restore และ DR พื้นฐาน](./21-lab-18-backup-restore-dr.md)

<!-- lesson-nav:end -->
