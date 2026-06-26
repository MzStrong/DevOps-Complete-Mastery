# 5. Lab 02: Linux, SSH, Service และ Log

Lab นี้ฝึกอ่านอาการของ server จาก command line ซึ่งเป็นทักษะพื้นฐานของ DevOps ทุกเรื่องหลังจากนี้ ไม่ว่าจะเป็น Docker, GitLab, Monitoring หรือ Kubernetes สุดท้ายเมื่อระบบมีปัญหา คุณจะต้องกลับมาดู process, service, port, disk, memory และ log บน Linux เสมอ

## วัตถุประสงค์

- ใช้คำสั่ง Linux สำคัญ
- ตรวจ process, disk, memory
- จัดการ service ด้วย systemd
- อ่าน log ด้วย journalctl

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Process | โปรแกรมที่กำลังทำงาน |
| Service | โปรแกรม background ที่ systemd จัดการ |
| Daemon | service ที่รันเบื้องหลัง |
| Permission | สิทธิ์ของไฟล์ เช่น read/write/execute |
| Log | บันทึกเหตุการณ์ของระบบ |

คำศัพท์ในบทนี้จะถูกใช้ซ้ำหลายครั้ง เช่น เวลา Nginx ไม่ทำงานจะตรวจ `service`, เวลา app crash จะดู `process` และ `log`, เวลา Docker หรือ database เขียนไฟล์ไม่ได้มักเกี่ยวกับ `permission`

## คำสั่งพื้นฐาน

```bash
pwd
ls -lah
cd /var/log
cat /etc/os-release
whoami
uname -a
```

คำสั่งกลุ่มนี้ใช้ตอบคำถามพื้นฐานว่า “ตอนนี้เราอยู่ที่ไหน เป็น user อะไร และเครื่องนี้เป็น OS/Kernel อะไร”

- `pwd` ดู path ปัจจุบัน ป้องกันการแก้ไฟล์ผิด directory
- `ls -lah` ดูไฟล์พร้อม permission, owner, size และ hidden files
- `cd /var/log` ไปยัง directory ที่มักเก็บ log ของระบบ
- `cat /etc/os-release` ตรวจ version ของ Ubuntu
- `whoami` ตรวจ user ปัจจุบัน
- `uname -a` ดู kernel และ architecture

ตัวอย่างที่ผิด:

```bash
sudo rm app.log
```

ถ้าไม่ตรวจ `pwd` ก่อน คุณอาจลบไฟล์ผิด directory ได้ แม้คำสั่งนี้จะไม่อยู่ใน Lab แต่หลักการคือ command line ไม่มีปุ่ม undo แบบ GUI

ตัวอย่างที่ถูก:

```bash
pwd
ls -lah
```

ตรวจ location และไฟล์ก่อนรันคำสั่งที่เปลี่ยนระบบ

## ตรวจ resource

```bash
top
free -h
df -h
du -sh /var/log/*
```

คำสั่งกลุ่มนี้ใช้ดูว่าเครื่องยังมี resource พอหรือไม่:

- `top` ดู process ที่ใช้ CPU/RAM สูง
- `free -h` ดู memory และ swap
- `df -h` ดู disk ของ filesystem ว่าเต็มหรือใกล้เต็มไหม
- `du -sh /var/log/*` ดูว่า log ตัวไหนกินพื้นที่มาก

อาการที่มักเจอ:

```text
service start ไม่ขึ้น -> disk เต็ม
GitLab ช้ามาก -> RAM ไม่พอและ swap หนัก
Docker build fail -> disk เต็ม
Loki/Grafana มีปัญหา -> log หรือ data directory โตเกิน
```

ตัวอย่างการอ่านผล:

```text
df -h เห็น / ใช้ไป 100%
แปลว่า root filesystem เต็ม
ผลที่ตามมา: package install ไม่ได้, service เขียน log ไม่ได้, database อาจ start ไม่ขึ้น
```

เมื่อ disk เต็ม อย่าเพิ่งลบไฟล์แบบสุ่ม ให้หาก่อนว่า directory ไหนใหญ่ เช่น `/var/log`, Docker image/cache หรือ application data

## ตรวจ process

```bash
ps aux
ps aux | grep ssh
```

`ps aux` แสดง process ทั้งระบบ ใช้ดูว่าโปรแกรมที่คาดว่าจะรันอยู่จริงไหม ส่วน `grep ssh` ใช้กรองเฉพาะ process ที่เกี่ยวกับ SSH

ข้อควรระวังคือ `grep ssh` อาจแสดง process ของคำสั่ง grep เองด้วย เช่น:

```text
devops  1234  ... sshd: devops@pts/0
devops  5678  ... grep ssh
```

บรรทัด `grep ssh` ไม่ใช่ SSH service จริง เป็นแค่คำสั่งค้นหาที่เรารันอยู่

ในงานจริง ถ้าต้องตรวจว่า port เปิดอยู่หรือไม่ ใช้ `ss` จะชัดกว่า แต่ `ps` ยังมีประโยชน์สำหรับดูว่า process มีอยู่และใช้ resource เท่าไร

## systemctl

```bash
sudo systemctl status ssh
sudo systemctl restart ssh
sudo systemctl enable ssh
```

`systemctl` ใช้จัดการ service ที่ systemd ดูแล:

- `status` ดูสถานะปัจจุบันและ log ล่าสุดบางส่วน
- `restart` หยุดแล้วเริ่ม service ใหม่
- `enable` ตั้งให้ service start อัตโนมัติหลัง reboot

ผลที่ต้องดูใน `status`:

```text
Active: active (running)
```

ถ้าเห็น `failed`, `inactive` หรือมี error สีแดง ให้ดูบรรทัด log ใต้ status ต่อทันที

ตัวอย่างที่ผิด:

```bash
sudo systemctl restart ssh
```

แล้วไม่ตรวจต่อว่า service กลับมารันจริงไหม

ตัวอย่างที่ถูก:

```bash
sudo systemctl restart ssh
sudo systemctl status ssh
```

restart แล้วต้องตรวจผลเสมอ โดยเฉพาะ service สำคัญอย่าง SSH เพราะถ้า config ผิดและคุณกำลัง remote อยู่ อาจ reconnect ไม่ได้

## journalctl

```bash
sudo journalctl -xe
sudo journalctl -u ssh
sudo journalctl -u ssh -f
```

`journalctl` ใช้อ่าน log จาก systemd journal:

- `sudo journalctl -xe` ดู error ล่าสุดของระบบแบบรวม
- `sudo journalctl -u ssh` ดู log เฉพาะ SSH service
- `sudo journalctl -u ssh -f` follow log แบบ real-time คล้าย `tail -f`

เวลามีปัญหา service start ไม่ขึ้น ให้ใช้ pattern นี้:

```bash
sudo systemctl status <service>
sudo journalctl -u <service> -n 50
```

`status` บอกภาพรวม ส่วน `journalctl` ให้รายละเอียด log มากกว่า

ตัวอย่างการใช้จริง:

```text
SSH เข้าไม่ได้
-> เปิด console VMware
-> sudo systemctl status ssh
-> sudo journalctl -u ssh -n 50
-> ตรวจว่า service down, config error หรือ authentication fail
```

## Permission

```bash
touch test.txt
ls -l test.txt
chmod 600 test.txt
ls -l test.txt
```

permission เป็นสาเหตุปัญหาที่เจอบ่อยมาก เช่น app อ่าน config ไม่ได้, service เขียน log ไม่ได้ หรือ script execute ไม่ได้

หลัง `touch test.txt` แล้ว `ls -l test.txt` จะเห็น permission เช่น:

```text
-rw-r--r-- 1 devops devops 0 Jun 25 10:00 test.txt
```

ความหมายคร่าว ๆ:

```text
owner  group  others
rw-    r--    r--
```

หลัง `chmod 600 test.txt` จะเหลือเฉพาะ owner ที่อ่าน/เขียนได้:

```text
-rw------- 1 devops devops 0 Jun 25 10:00 test.txt
```

ตัวอย่างที่ผิด:

```bash
chmod 777 config.env
```

ทำให้ทุก user อ่านและแก้ไฟล์ได้ ไม่เหมาะกับไฟล์ config หรือ secret

ตัวอย่างที่ถูก:

```bash
chmod 600 config.env
```

ให้เฉพาะ owner อ่าน/เขียนไฟล์ได้ เหมาะกับไฟล์ที่มีข้อมูลสำคัญใน lab

## การทดสอบ

1. เปิด terminal แรกแล้วดู log SSH

```bash
sudo journalctl -u ssh -f
```

2. เปิดอีก terminal แล้ว SSH เข้าเครื่อง

```bash
ssh devops@app-server
```

สิ่งที่กำลังทดสอบคือความสัมพันธ์ระหว่าง action กับ log เมื่อมีการ SSH เข้าเครื่อง คุณควรเห็น log ใหม่ใน terminal แรก วิธีนี้ช่วยฝึกดูเหตุการณ์แบบ real-time ซึ่งจะใช้ต่อกับ Nginx, Docker, GitLab Runner และ Kubernetes ได้

ถ้า SSH แล้วไม่เห็น log ให้ตรวจว่า:

- terminal แรกดู log ของ service ถูกตัวหรือไม่
- SSH เข้าเครื่องเดียวกับที่กำลังดู log หรือเปล่า
- service ชื่อ `ssh` หรือ `sshd` ขึ้นกับ distro/config บางกรณี

## การตรวจสอบ

- เห็น log การ login
- restart service ได้
- ดู disk/memory/process ได้

เพิ่มคำถามตรวจความเข้าใจ:

- ถ้า disk เต็ม จะใช้คำสั่งใดเริ่มตรวจ
- ถ้า SSH เข้าไม่ได้ จะดู service และ log อย่างไร
- ถ้า service failed จะเริ่มจาก `systemctl` หรือ `journalctl` ตรงไหน
- ถ้าไฟล์อ่านไม่ได้ จะดู permission ด้วยคำสั่งใด

ถ้าตอบคำถามเหล่านี้ได้ แปลว่าพร้อมไป Lab network และ firewall ต่อ

## ข้อสรุป

DevOps ต้องอ่านอาการระบบจาก Linux ได้ เช่น service ตาย, disk เต็ม, memory เต็ม, permission ผิด หรือ log error

บทนี้ไม่ได้ต้องการให้จำ option ของทุกคำสั่ง แต่ต้องการให้รู้ workflow การตรวจระบบ:

```text
อาการผิดปกติ
-> ตรวจ service/process
-> ตรวจ resource
-> ตรวจ permission ถ้าเกี่ยวกับไฟล์
-> อ่าน log
-> แก้ไข
-> ตรวจซ้ำ
```

ถ้าฝึก pattern นี้จนคุ้น บทต่อ ๆ ไปจะ debug ง่ายขึ้นมาก

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [4. Lab 01: เตรียม VM และ Ubuntu Server](./04-lab-01-vm-and-ubuntu-server.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [6. Lab 03: Network, DNS, Port และ Firewall](./06-lab-03-network-dns-port-firewall.md)

<!-- lesson-nav:end -->
