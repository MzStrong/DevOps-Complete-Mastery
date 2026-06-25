# 11. Lab 08: Git Flow และ Release Version

## วัตถุประสงค์

- จัด branch strategy
- ใช้ tag version
- เข้าใจ release และ rollback

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Branch | สายการพัฒนา code |
| Merge Request | คำขอรวม code |
| Tag | ป้าย version ที่ชี้ commit |
| Semantic Versioning | version แบบ MAJOR.MINOR.PATCH เช่น 1.0.0 |
| Changelog | สรุปสิ่งที่เปลี่ยนใน release |

## Branch Flow ตัวอย่าง

```text
feature/* -> develop -> uat -> main
```

## ทดลอง Git Flow

```bash
mkdir ~/labs/git-flow-demo
cd ~/labs/git-flow-demo
git init
echo '# Git Flow Demo' > README.md
git add .
git commit -m 'initial commit'
git branch -M main
git checkout -b develop
git checkout -b feature/hello
echo 'hello feature' >> README.md
git add .
git commit -m 'add hello feature'
git checkout develop
git merge feature/hello
git checkout main
git merge develop
git tag v1.0.0
git log --oneline --decorate --graph --all
```

## การตรวจสอบ

- เห็น branch และ tag
- รู้ว่า release v1.0.0 ชี้ commit ไหน
- rollback โดย checkout tag เก่าได้

## ข้อสรุป

DevOps ต้องตอบได้ว่า deploy commit ไหน, version ไหน, ใคร approve และถ้าพังจะ rollback กลับ version ไหน

---
