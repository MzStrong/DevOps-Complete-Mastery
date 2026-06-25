# 11. Lab 08: Git Flow และ Release Version

Lab นี้ฝึกจัดระเบียบ code ก่อนเข้าสู่ CI/CD จริง เป้าหมายคือให้ตอบได้ว่า version ที่ deploy มาจาก commit ไหน ผ่าน branch ไหน และถ้ามีปัญหาจะย้อนกลับไป version ใด การมี Git flow และ tag ที่ชัดเจนช่วยให้ pipeline, Docker image tag และ deployment trace กลับไปยัง source code ได้

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

คำศัพท์เหล่านี้เกี่ยวข้องกับการควบคุมความเสี่ยงของ release:

- `Branch` ใช้แยกงานที่กำลังพัฒนาออกจาก code ที่พร้อม deploy
- `Merge Request` ใช้ review และ approve ก่อนรวม code
- `Tag` ใช้ตรึง commit ให้เป็น version ที่อ้างอิงได้ถาวร
- `Changelog` ใช้บอกว่าผู้ใช้หรือทีม operation ควรรู้อะไรใน release นั้น

## Branch Flow ตัวอย่าง

```text
feature/* -> develop -> uat -> main
```

flow นี้อ่านได้ว่า:

- `feature/*` ใช้พัฒนา feature หรือ bug fix แยกเป็นงาน ๆ
- `develop` รวมงานที่พร้อมทดสอบร่วมกัน
- `uat` ใช้สำหรับ user acceptance test หรือ staging
- `main` เก็บ code ที่พร้อม release หรือ production

ไม่จำเป็นว่าทุกองค์กรต้องใช้ชื่อ branch แบบนี้ แต่ต้องมีหลักการว่า branch ใดปลอดภัยสำหรับ deploy และ branch ใดเป็นงานที่ยังไม่เสร็จ

ตัวอย่าง flow ที่เรียบง่ายสำหรับ lab:

```text
feature/hello
-> merge เข้า develop
-> merge เข้า main
-> tag v1.0.0
-> build image simple-api:1.0.0
```

สิ่งสำคัญคือ tag ของ Git และ tag ของ Docker image ควรสื่อถึง release เดียวกัน เช่น Git tag `v1.0.0` และ image tag `1.0.0`

## Semantic Versioning

Semantic Versioning ใช้รูปแบบ:

```text
MAJOR.MINOR.PATCH
```

ตัวอย่าง:

```text
1.0.0
1.1.0
1.1.1
2.0.0
```

ความหมายทั่วไป:

- `MAJOR` เพิ่มเมื่อมี breaking change
- `MINOR` เพิ่มเมื่อมี feature ใหม่ที่ยัง backward compatible
- `PATCH` เพิ่มเมื่อแก้ bug โดยไม่เปลี่ยน behavior หลัก

ตัวอย่าง:

```text
1.0.0 -> release แรก
1.0.1 -> แก้ bug เล็ก
1.1.0 -> เพิ่ม endpoint ใหม่
2.0.0 -> เปลี่ยน API contract ที่ทำให้ client เดิมใช้ไม่ได้
```

ใน lab นี้ไม่ต้องเข้มงวดระดับ production แต่ควรฝึกตั้ง version ให้มีความหมาย เพราะจะช่วยตอน rollback และตรวจว่าระบบกำลังรัน code ชุดไหน

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

อธิบายลำดับที่เกิดขึ้น:

1. สร้าง repository ใหม่
2. commit แรกเป็นฐานของ project
3. ตั้ง branch หลักเป็น `main`
4. สร้าง `develop` สำหรับรวมงานก่อน release
5. สร้าง `feature/hello` เพื่อจำลองงาน feature
6. merge feature เข้า develop
7. merge develop เข้า main
8. สร้าง tag `v1.0.0` บน commit ที่จะถือเป็น release
9. ดู graph เพื่อเห็น branch และ tag

หลังรันคำสั่งสุดท้าย คุณควรเห็นภาพประมาณนี้:

```text
* commit (HEAD -> main, tag: v1.0.0, develop) add hello feature
* commit initial commit
```

ผลจริงอาจต่างเล็กน้อยตาม Git version และ merge strategy แต่ต้องเห็นว่า `v1.0.0` ชี้ commit ที่ release

ถ้า Git ไม่ยอม commit เพราะยังไม่ได้ตั้ง user:

```bash
git config --global user.name "DevOps Lab"
git config --global user.email "devops@example.local"
```

## ตัวอย่างที่ผิดและถูก

ตัวอย่างที่ผิด:

```text
deploy จาก branch feature โดยตรง
ไม่ tag version
Docker image ใช้ latest อย่างเดียว
เมื่อ production มีปัญหา ไม่รู้ว่า commit ไหนถูก deploy
```

ผลคือ rollback ยากมาก เพราะไม่มีจุดอ้างอิงที่ชัดเจน

ตัวอย่างที่ถูก:

```text
feature ผ่าน review
merge เข้า develop เพื่อทดสอบ
merge เข้า main เมื่อพร้อม release
tag v1.0.0
build image simple-api:1.0.0
deploy image tag นั้น
```

เมื่อมีปัญหา ทีมสามารถตอบได้ว่า deploy version ไหน และย้อนกลับไป tag ก่อนหน้าได้

## เชื่อม Git Tag กับ Docker Image Tag

ใน pipeline จริง มักใช้ Git tag หรือ commit SHA ไปเป็น Docker image tag เช่น:

```text
Git tag:      v1.0.0
Docker image: devops-control:5000/simple-api:1.0.0
```

หรือ:

```text
Git commit:   a1b2c3d
Docker image: devops-control:5000/simple-api:a1b2c3d
```

แบบแรกอ่านง่ายสำหรับ release ส่วนแบบ commit SHA trace ได้แม่นมาก ทั้งสองแบบใช้ร่วมกันได้

สิ่งที่ควรหลีกเลี่ยงคือ deploy ด้วย `latest` โดยไม่มี record เพิ่มเติม เพราะ `latest` เปลี่ยนได้ตลอดและไม่บอกว่าเป็น commit ใด

## Rollback ทำอย่างไรในมุม Git

การ rollback ไม่ได้แปลว่าลบ commit เสมอไป แต่คือการนำ version ก่อนหน้ากลับมา deploy เช่น:

```bash
git checkout v1.0.0
```

คำสั่งนี้ใช้ดู source code ณ tag นั้น แต่ในการ deploy จริงมัก rollback โดยเลือก Docker image tag เก่าหรือใช้ deployment tool เช่น `kubectl rollout undo` หรือ `helm rollback`

Git tag จึงเป็นหลักฐานว่า image หรือ release นั้นควรตรงกับ source code ชุดใด

## การตรวจสอบ

- เห็น branch และ tag
- รู้ว่า release v1.0.0 ชี้ commit ไหน
- rollback โดย checkout tag เก่าได้

คำสั่งตรวจเพิ่มเติม:

```bash
git branch -a
git tag
git show v1.0.0 --stat
git log --oneline --decorate --graph --all
```

คำถามที่ควรตอบได้:

- `main` อยู่ commit ไหน
- `develop` อยู่ commit ไหน
- tag `v1.0.0` ชี้ commit ไหน
- commit ที่ tag มีการเปลี่ยนแปลงอะไร
- ถ้าต้อง build image จาก release นี้ จะใช้ tag อะไร

ถ้าตอบไม่ได้ ให้ดู `git log --decorate --graph --all` จนเห็นความสัมพันธ์ของ branch และ tag ก่อน

## ข้อสรุป

DevOps ต้องตอบได้ว่า deploy commit ไหน, version ไหน, ใคร approve และถ้าพังจะ rollback กลับ version ไหน

Git Flow ไม่ใช่เป้าหมายในตัวเอง แต่เป็นวิธีทำให้การเปลี่ยนแปลงตรวจสอบได้:

```text
code change
-> review
-> merge
-> tag
-> build image
-> deploy
-> rollback ได้ถ้าจำเป็น
```

บทถัดไปเรื่อง GitLab CI/CD จะนำแนวคิดนี้ไปทำ automation เพื่อให้ pipeline build/test/deploy ตาม branch หรือ tag ที่กำหนด

---
