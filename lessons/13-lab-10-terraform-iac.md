# 13. Lab 10: Terraform และ Infrastructure as Code

Lab นี้ใช้ Terraform กับ provider แบบ local เพื่อฝึก workflow ของ Infrastructure as Code โดยยังไม่ต้องใช้ cloud จริง จุดประสงค์คือให้เข้าใจ `init`, `plan`, `apply`, `state` และ `destroy` ก่อน เพราะเมื่อไปใช้กับ cloud คำสั่งเดียวกันนี้จะสร้าง resource จริงที่มีค่าใช้จ่ายและผลกระทบจริง

## วัตถุประสงค์

- เข้าใจ Terraform workflow
- ใช้ provider, resource, state
- เตรียมพื้นฐานไปใช้กับ Cloud จริง

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Provider | plugin ที่ Terraform ใช้คุยกับ platform |
| Resource | สิ่งที่ Terraform สร้างหรือจัดการ |
| State | ไฟล์บันทึกสถานะ infra |
| Plan | preview สิ่งที่จะเปลี่ยน |
| Apply | สั่งสร้าง/แก้ไขจริง |
| Destroy | ลบ resource |

Terraform ทำงานด้วยแนวคิด desired state คือเราเขียน code บอกว่า “อยากให้ infra เป็นแบบไหน” แล้ว Terraform เปรียบเทียบกับ state ปัจจุบันเพื่อสร้างแผนว่าจะต้องเพิ่ม แก้ หรือลบอะไร

flow หลัก:

```text
เขียน .tf
-> terraform init
-> terraform plan
-> review plan
-> terraform apply
-> Terraform update state
```

สิ่งสำคัญคือ `plan` ต้องถูก review ก่อน `apply` เสมอ โดยเฉพาะใน cloud เพราะ plan อาจบอกว่าจะลบ resource สำคัญ

## ติดตั้ง Terraform

```bash
sudo apt update
sudo apt install -y wget unzip
wget https://releases.hashicorp.com/terraform/1.9.8/terraform_1.9.8_linux_amd64.zip
unzip terraform_1.9.8_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform version
```

คำสั่งนี้ติดตั้ง Terraform binary แบบ manual:

- `wget` ดาวน์โหลด zip จาก HashiCorp release
- `unzip` แตกไฟล์
- `mv terraform /usr/local/bin/` ทำให้เรียก `terraform` ได้จาก shell
- `terraform version` ตรวจว่า binary ใช้งานได้

ในเครื่องจริงอาจติดตั้งผ่าน package manager หรือใช้ version manager เช่น `tfenv` เพื่อสลับ version ได้ แต่ใน lab นี้ใช้วิธีตรงไปตรงมาเพื่อให้เห็นว่า Terraform เป็น single binary

ข้อควรระวัง:

```text
Terraform version อาจมีผลกับ provider และ syntax
ทีมเดียวกันควรกำหนด version ที่ใช้ให้ชัดเจน
```

ตรวจ path:

```bash
which terraform
terraform version
```

## Terraform local_file Lab

```bash
mkdir -p ~/labs/terraform-local
cd ~/labs/terraform-local
nano main.tf
```

Lab นี้ใช้ `local_file` resource เพื่อให้ Terraform สร้างไฟล์ในเครื่อง แทนการสร้าง VM หรือ cloud resource จริง เหมาะสำหรับเรียน workflow เพราะเห็นผลได้ง่ายและไม่มีค่าใช้จ่าย

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

provider "local" {}

resource "local_file" "devops_note" {
  filename = "${path.module}/devops-note.txt"
  content  = "Created by Terraform\n"
}
```

อธิบายไฟล์ `main.tf`:

- `terraform.required_providers` ระบุ provider ที่ project ต้องใช้
- `source = "hashicorp/local"` บอกว่าใช้ local provider จาก HashiCorp registry
- `version = "~> 2.5"` จำกัด version provider ให้อยู่ในช่วงที่เข้ากันได้
- `provider "local" {}` configure provider
- `resource "local_file" "devops_note"` ประกาศ resource ชนิด `local_file` ชื่อ logical name ว่า `devops_note`
- `filename` คือ path ของไฟล์ที่จะสร้าง
- `content` คือเนื้อหาในไฟล์

ชื่อ resource แบ่งเป็น:

```text
local_file.devops_note
^ type     ^ local name ใน Terraform
```

ชื่อนี้จะปรากฏใน state และ plan

Run:

```bash
terraform init
terraform plan
terraform apply
cat devops-note.txt
terraform state list
terraform destroy
```

อธิบายแต่ละคำสั่ง:

### terraform init

```bash
terraform init
```

ดาวน์โหลด provider และเตรียม working directory จะสร้าง folder `.terraform/` และ lock file เช่น `.terraform.lock.hcl`

ถ้าไม่รัน `init` ก่อน `plan/apply` Terraform จะยังไม่มี provider plugin และจะทำงานไม่ได้

### terraform plan

```bash
terraform plan
```

แสดง preview ว่า Terraform จะทำอะไร เช่น create `local_file.devops_note` โดยยังไม่เปลี่ยนระบบจริง

ตัวอย่างสิ่งที่ควรดู:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

ถ้า plan บอกว่าจะ destroy resource ที่ไม่คาดคิด ต้องหยุดและตรวจ code/state ก่อน apply

### terraform apply

```bash
terraform apply
```

สั่งให้ Terraform ทำตาม plan จริง หลัง apply จะเกิดไฟล์ `devops-note.txt` และ Terraform จะบันทึกสถานะไว้ใน `terraform.tfstate`

ตรวจผล:

```bash
cat devops-note.txt
terraform state list
```

`terraform state list` ควรเห็น:

```text
local_file.devops_note
```

### terraform destroy

```bash
terraform destroy
```

ลบ resource ที่ Terraform จัดการอยู่ ใน lab นี้คือลบไฟล์ `devops-note.txt` แต่ใน cloud อาจหมายถึงลบ VM, database, network หรือ storage จริง จึงต้องอ่าน plan ก่อนยืนยันเสมอ

## State คือจุดที่ต้องระวัง

ไฟล์ `terraform.tfstate` คือ memory ของ Terraform ว่า resource ไหนถูกสร้างไว้แล้ว ถ้า state หาย Terraform อาจไม่รู้ว่า resource เดิมมีอยู่ และอาจพยายามสร้างใหม่หรือจัดการผิดพลาด

ใน lab local state อยู่ใน directory เดียวกับ code แต่ในทีมจริงควรใช้ remote backend เช่น S3, Azure Storage, Terraform Cloud หรือ backend อื่นที่มี locking เพื่อกันหลายคน apply พร้อมกัน

ตัวอย่างที่ผิด:

```text
ลบ terraform.tfstate เพราะคิดว่าเป็นไฟล์ generated ที่ไม่สำคัญ
```

ผลคือ Terraform สูญเสีย mapping ระหว่าง code กับ resource จริง

ตัวอย่างที่ถูก:

```text
ใช้ remote state สำหรับงานทีม
เปิด state locking
จำกัดสิทธิ์การเข้าถึง state เพราะอาจมีข้อมูล sensitive
```

## ทดลองเปลี่ยนค่า

หลัง apply แล้วลองแก้:

```hcl
content = "Updated by Terraform\n"
```

แล้วรัน:

```bash
terraform plan
terraform apply
cat devops-note.txt
```

คุณจะเห็นว่า Terraform ตรวจพบว่า resource เดิมต้องถูก update ไม่ใช่สร้างใหม่ทั้งหมด นี่คือหัวใจของ IaC: code เปลี่ยน, plan แสดงผลกระทบ, apply ทำให้ระบบตรงกับ code

## การตรวจสอบ

คำสั่งที่ควรใช้หลังแต่ละช่วง:

```bash
terraform fmt
terraform validate
terraform plan
terraform state list
ls -lah
cat devops-note.txt
```

- `terraform fmt` จัด format ไฟล์ `.tf`
- `terraform validate` ตรวจ syntax และ schema เบื้องต้น
- `terraform plan` ดูผลกระทบก่อน apply
- `terraform state list` ดู resource ที่ Terraform จัดการ

ถ้า `terraform init` fail ให้ตรวจ internet/DNS เพราะ Terraform ต้องดาวน์โหลด provider ถ้า `plan` fail ให้ดู syntax หรือ provider config ถ้า `apply` fail ให้ดู permission หรือ path ที่ resource ต้องเขียน

## ข้อสรุป

Terraform ทำให้ infra ถูกจัดการเป็น code, review ได้, version control ได้ และทำซ้ำได้

สิ่งที่ควรจำ:

```text
.tf = desired state
provider = ตัวคุยกับ platform
resource = สิ่งที่ต้องการจัดการ
plan = ดูก่อนทำ
apply = ทำจริง
state = ความจำของ Terraform
destroy = ลบของจริง
```

เมื่อใช้กับ cloud ให้ระวังมากกว่า lab นี้ เพราะ `apply` และ `destroy` อาจสร้างค่าใช้จ่ายหรือลบ resource สำคัญได้เสมอ

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [12. Lab 09: CI/CD ด้วย GitLab CE และ GitLab Runner](./12-lab-09-gitlab-ci-cd.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [14. Lab 11: Monitoring ด้วย Prometheus และ Grafana](./14-lab-11-prometheus-grafana-monitoring.md)

<!-- lesson-nav:end -->
