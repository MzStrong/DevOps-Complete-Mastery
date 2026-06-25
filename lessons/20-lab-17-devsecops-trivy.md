# 20. Lab 17: DevSecOps ด้วย Trivy

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

## ติดตั้ง Trivy

```bash
sudo apt update
sudo apt install -y wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt update
sudo apt install -y trivy
```

## Scan image

```bash
trivy image simple-api:1.0.0
trivy image --severity HIGH,CRITICAL simple-api:1.0.0
trivy image --exit-code 1 --severity CRITICAL simple-api:1.0.0
```

## Scan source code/filesystem

```bash
trivy fs .
```

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

## Basic Hardening Checklist

- ไม่เก็บ password/token ใน code
- ไม่ commit `.env`
- ไม่ run container เป็น root ถ้าไม่จำเป็น
- ตั้ง resource limit ให้ container
- เปิด firewall เฉพาะ port ที่จำเป็น
- ให้สิทธิ์แบบ least privilege
- scan dependency และ image ก่อน deploy

## ข้อสรุป

DevSecOps คือการทำ security ตั้งแต่ต้นทางใน pipeline ไม่ใช่รอให้ขึ้น production แล้วค่อยตรวจ

---
