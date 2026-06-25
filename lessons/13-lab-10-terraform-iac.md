# 13. Lab 10: Terraform และ Infrastructure as Code

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

## ติดตั้ง Terraform

```bash
sudo apt update
sudo apt install -y wget unzip
wget https://releases.hashicorp.com/terraform/1.9.8/terraform_1.9.8_linux_amd64.zip
unzip terraform_1.9.8_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform version
```

## Terraform local_file Lab

```bash
mkdir -p ~/labs/terraform-local
cd ~/labs/terraform-local
nano main.tf
```

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

Run:

```bash
terraform init
terraform plan
terraform apply
cat devops-note.txt
terraform state list
terraform destroy
```

## ข้อสรุป

Terraform ทำให้ infra ถูกจัดการเป็น code, review ได้, version control ได้ และทำซ้ำได้

---
