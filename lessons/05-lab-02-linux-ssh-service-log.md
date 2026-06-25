# 5. Lab 02: Linux, SSH, Service และ Log

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

## คำสั่งพื้นฐาน

```bash
pwd
ls -lah
cd /var/log
cat /etc/os-release
whoami
uname -a
```

## ตรวจ resource

```bash
top
free -h
df -h
du -sh /var/log/*
```

## ตรวจ process

```bash
ps aux
ps aux | grep ssh
```

## systemctl

```bash
sudo systemctl status ssh
sudo systemctl restart ssh
sudo systemctl enable ssh
```

## journalctl

```bash
sudo journalctl -xe
sudo journalctl -u ssh
sudo journalctl -u ssh -f
```

## Permission

```bash
touch test.txt
ls -l test.txt
chmod 600 test.txt
ls -l test.txt
```

## การทดสอบ

1. เปิด terminal แรกแล้วดู log SSH

```bash
sudo journalctl -u ssh -f
```

2. เปิดอีก terminal แล้ว SSH เข้าเครื่อง

```bash
ssh devops@app-server
```

## การตรวจสอบ

- เห็น log การ login
- restart service ได้
- ดู disk/memory/process ได้

## ข้อสรุป

DevOps ต้องอ่านอาการระบบจาก Linux ได้ เช่น service ตาย, disk เต็ม, memory เต็ม, permission ผิด หรือ log error

---
