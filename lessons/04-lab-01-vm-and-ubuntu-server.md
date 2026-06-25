# 4. Lab 01: เตรียม VM และ Ubuntu Server

Lab นี้เป็นการสร้างฐานของ environment ทั้งหมด สิ่งที่ต้องได้จากบทนี้คือ VM ที่เข้าใช้งานผ่าน SSH ได้ มี hostname ชัดเจน มี IP คงที่ และทุกเครื่องเรียกชื่อกันได้ ถ้าส่วนนี้ไม่นิ่ง บทถัดไปจะเริ่มมีปัญหาต่อเนื่อง เช่น GitLab Runner หา server ไม่เจอ, registry push/pull ไม่ได้, Prometheus scrape target ไม่เจอ หรือ Kubernetes worker join cluster ไม่สำเร็จ

## วัตถุประสงค์

- สร้าง VM บน VMware
- ดาวน์โหลด Ubuntu Server ISO
- ติดตั้ง Ubuntu Server
- ตั้ง hostname และ static IP
- เปิด SSH
- ตั้ง `/etc/hosts` ให้เรียกชื่อเครื่องได้

## เตรียม VMware Workstation

ก่อนสร้าง VM ต้องมี hypervisor สำหรับจำลองเครื่อง ใน Lab นี้ใช้ VMware Workstation Pro หรือ VMware Workstation Player บน Windows/Linux เครื่องหลัก ถ้าใช้ VMware เวอร์ชันใหม่ ช่องทางดาวน์โหลดอาจพาไปที่ VMware/Broadcom portal ให้ใช้แหล่งทางการเป็นหลัก:

```text
VMware Workstation:
https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion
```

แนวทางติดตั้งบน Windows โดยทั่วไป:

1. ดาวน์โหลด installer จากเว็บทางการ
2. เปิดไฟล์ installer ด้วยสิทธิ์ Administrator
3. กด Next ตาม wizard
4. เปิดตัวเลือก network adapter และ enhanced keyboard driver ตามค่า default ได้
5. ติดตั้งเสร็จแล้ว restart เครื่องถ้าระบบร้องขอ
6. เปิด VMware Workstation แล้วตรวจว่าเมนูสร้าง VM ใช้งานได้

ถ้าเปิด VM แล้วขึ้น error เกี่ยวกับ virtualization ให้ตรวจ BIOS/UEFI ว่าเปิด Intel VT-x หรือ AMD-V แล้วหรือยัง บางเครื่องจะใช้ชื่อเมนูเช่น `Intel Virtualization Technology`, `SVM Mode` หรือ `AMD-V`

ตัวอย่างที่ผิด:

```text
ติดตั้ง VMware แล้วสร้าง VM ทันที แต่ BIOS ยังปิด virtualization
```

ผลคือ VM อาจเปิดไม่ได้ หรือรันช้ามาก เพราะ CPU virtualization ไม่พร้อม

ตัวอย่างที่ถูก:

```text
เปิด virtualization ใน BIOS/UEFI
ติดตั้ง VMware
เปิด VMware แล้วลองสร้าง VM เปล่าได้
ค่อยเริ่มสร้าง Ubuntu Server VM
```

ถ้าเครื่องเปิด Hyper-V, WSL2 หรือ Windows virtualization feature อยู่ VMware รุ่นใหม่มักทำงานร่วมกันได้ดีขึ้น แต่ถ้าเจอปัญหา performance หรือเปิด VM ไม่ได้ ให้ตรวจว่า VMware แจ้ง error อะไร อย่าปิด feature แบบสุ่มโดยไม่อ่าน error เพราะอาจกระทบงานอื่นในเครื่อง

## ดาวน์โหลด Ubuntu Server ISO

ให้ดาวน์โหลด ISO จากเว็บ Ubuntu โดยตรง:

```text
Ubuntu Server:
https://ubuntu.com/download/server
```

เลือก Ubuntu Server รุ่น LTS เช่น 22.04 LTS หรือ 24.04 LTS สำหรับ Lab นี้ แนะนำไฟล์แบบ `live-server-amd64.iso` เพราะเหมาะกับเครื่อง x86_64/AMD64 ทั่วไป

หลังดาวน์โหลดควรเก็บ ISO ไว้ในโฟลเดอร์ที่หาเจอง่าย เช่น:

```text
C:\ISO\ubuntu-24.04-live-server-amd64.iso
```

เหตุผลที่ควรใช้ Server ISO:

- ไม่มี desktop GUI ทำให้กิน RAM และ disk น้อยกว่า
- ใกล้เคียง server จริงที่ใช้ในงาน DevOps
- บังคับให้คุ้นกับ command line, SSH และ service management

ตัวอย่างที่ผิด:

```text
ดาวน์โหลด Ubuntu Desktop ISO เพราะหน้าตาดูใช้ง่ายกว่า
```

ใช้ได้ในทางเทคนิค แต่ไม่เหมาะกับ Lab นี้ เพราะกิน resource มากกว่าและทำให้ไม่ได้ฝึก server workflow เท่าที่ควร

ตัวอย่างที่ถูก:

```text
ดาวน์โหลด Ubuntu Server LTS ISO
ใช้ ISO เดียวกันสร้าง VM ทุกเครื่อง
ตั้ง hostname/IP ต่างกันตาม role ของแต่ละ VM
```

## ขั้นตอนติดตั้ง VM

1. สร้าง VM ใหม่ใน VMware
2. เลือก ISO Ubuntu Server
3. กำหนด CPU/RAM/Disk ตามตาราง
4. ตั้ง Network เป็น NAT หรือ Host-only
5. ระหว่างติดตั้งให้เลือกติดตั้ง OpenSSH Server
6. สร้าง user ชื่อ `devops`

ขั้นตอนสร้าง VM ใน VMware โดยละเอียด:

1. เปิด VMware Workstation
2. เลือก `Create a New Virtual Machine`
3. เลือก `Typical` สำหรับ lab ทั่วไป
4. เลือก `Installer disc image file (iso)` แล้ว browse ไปที่ Ubuntu Server ISO
5. ตั้งชื่อ VM ตาม role เช่น `app-server`
6. เลือกตำแหน่งเก็บ VM ที่มี disk เหลือพอ
7. กำหนด disk size ตามตาราง เช่น 30 GB สำหรับ `app-server`
8. เลือก `Store virtual disk as a single file` หรือ `Split into multiple files` ก็ได้ สำหรับ lab ใช้ได้ทั้งคู่
9. กด `Customize Hardware`
10. ตั้ง CPU/RAM ตามบทสเปก
11. ตั้ง Network Adapter เป็น NAT หรือ Host-only ตามแผน network
12. กด Finish แล้วเปิด VM

คำแนะนำเรื่อง disk:

- `Single file` มัก performance ดีกว่าเล็กน้อยและจัดการง่ายถ้าอยู่บน disk เดียว
- `Split into multiple files` ย้ายข้าม filesystem หรือ external drive ง่ายกว่า
- อย่าตั้ง disk เล็กเกินไป เพราะ GitLab, Docker image, log และ Kubernetes image ใช้พื้นที่มากขึ้นเรื่อย ๆ

ในขั้นตอนนี้ให้สร้าง VM ตาม role ที่กำหนดไว้ในบทสเปก แต่ไม่จำเป็นต้องสร้างทุกเครื่องพร้อมกันถ้าเครื่องหลักมี RAM จำกัด อย่างน้อยในช่วงเริ่มต้นควรมี `devops-control` และ `app-server` ก่อน ส่วน `monitor-server` และกลุ่ม Kubernetes ค่อยสร้างเมื่อถึงบทที่ต้องใช้

จุดที่ควรระวังตอนติดตั้ง:

- เลือก Ubuntu Server ไม่ใช่ Desktop เพื่อลด resource และให้ใกล้เคียง server จริง
- เลือกติดตั้ง OpenSSH Server ตั้งแต่ตอน install เพื่อให้ remote เข้าเครื่องได้ทันที
- ตั้ง username เป็น `devops` ให้เหมือนกันทุกเครื่อง ลดปัญหา path และ permission ในบทหลัง
- ตั้ง disk ให้พอ โดยเฉพาะ `devops-control` เพราะ GitLab และ registry ใช้พื้นที่มากกว่า app server

ตัวอย่างที่ผิด:

```text
สร้าง VM ชื่อ Ubuntu-1, Ubuntu-2 แล้วค่อยจำเองว่าเครื่องไหนทำอะไร
```

ปัญหาคือเมื่อมีหลายเครื่อง จะสับสนว่าเครื่องไหนเป็น GitLab, เครื่องไหนเป็น app, เครื่องไหนเป็น Kubernetes node

ตัวอย่างที่ถูก:

```text
ตั้งชื่อ VM ตาม role:
devops-control
app-server
monitor-server
k8s-master-01
k8s-worker-01
k8s-worker-02
```

ชื่อ VM ใน VMware ไม่จำเป็นต้องตรงกับ hostname ใน OS เสมอไป แต่ควรตั้งให้ตรงกันเพื่อกันสับสน

## ติดตั้ง Ubuntu Server ใน VM

เมื่อเปิด VM แล้ว installer ของ Ubuntu Server จะเริ่มทำงาน ขั้นตอนใน installer อาจต่างกันเล็กน้อยตามเวอร์ชัน แต่แนวทางหลักเหมือนกัน:

1. เลือก `Try or Install Ubuntu Server`
2. เลือกภาษาและ keyboard layout
3. เลือก installation type แบบ default ได้
4. ตั้ง network ชั่วคราวด้วย DHCP ก่อนได้ เดี๋ยวค่อยเปลี่ยนเป็น static IP หลังติดตั้ง
5. ถ้ามี proxy ให้ตั้งค่า ถ้าไม่มีให้ข้าม
6. ใช้ mirror default ของ Ubuntu ได้
7. เลือก storage แบบใช้ disk ทั้งลูกของ VM
8. ตรวจ summary ก่อน confirm เพราะขั้นตอนนี้จะเขียน partition ลง virtual disk
9. ตั้ง profile:
   - Your name: `DevOps`
   - Server name: เช่น `app-server`
   - Username: `devops`
   - Password: ตั้งรหัสผ่านที่จำได้สำหรับ lab
10. เลือกติดตั้ง OpenSSH Server
11. ยังไม่ต้องเลือก snap package เพิ่ม ถ้าไม่จำเป็น
12. รอ install จบ แล้ว reboot
13. ถ้า VMware ยัง mount ISO อยู่ ให้ disconnect ISO หรือปล่อยให้ boot จาก disk

ในหน้า storage ให้แน่ใจว่ากำลังติดตั้งลง virtual disk ของ VM ไม่ใช่ disk จริงของเครื่อง host ปกติ VMware จะแสดงเฉพาะ virtual disk ภายใน VM แต่ควรอ่าน summary ก่อน confirm ทุกครั้ง

เหตุผลที่ติดตั้ง OpenSSH Server ตั้งแต่แรกคือหลัง OS พร้อมแล้วเราจะ remote เข้าเครื่องจาก terminal ได้ ถ้าลืมเลือก OpenSSH Server ยังติดตั้งภายหลังได้ แต่จะเสียเวลาเข้า console เพื่อรัน:

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

หลัง reboot ให้ login ด้วย user `devops` แล้วตรวจเบื้องต้น:

```bash
whoami
hostname
ip a
sudo systemctl status ssh
```

ผลที่คาดหวังคือ login ได้, hostname ตรงกับ role, มี IP จาก DHCP ชั่วคราว และ SSH service เป็น active/running จากนั้นค่อยไปตั้ง hostname และ static IP ให้ตรงกับแผนถาวร

ตัวอย่างที่ผิด:

```text
ตั้ง username เป็นชื่อคนจริงบนเครื่องหนึ่ง
ตั้งอีกเครื่องเป็น ubuntu
อีกเครื่องเป็น admin
```

ผลคือคำสั่งใน Lab ที่อ้าง user `devops` และ path `/home/devops` จะใช้ไม่ได้สม่ำเสมอ

ตัวอย่างที่ถูก:

```text
ทุก VM ใช้ username: devops
hostname เปลี่ยนตาม role ของ VM
password จดไว้ในที่ปลอดภัยนอก repository
```

## ตั้ง hostname

ตัวอย่างบน app-server:

```bash
sudo hostnamectl set-hostname app-server
hostnamectl
```

hostname คือชื่อเครื่องในระดับ OS ใช้ช่วยระบุว่าเรากำลังอยู่บนเครื่องไหน และใช้ใน `/etc/hosts`, monitoring target, Kubernetes node name หรือคำสั่ง SSH ได้ ถ้า hostname ซ้ำกันหรือไม่ตรงกับแผน จะทำให้ debug ยาก โดยเฉพาะเมื่อเปิด terminal หลายเครื่องพร้อมกัน

ตั้ง hostname ให้ตรงกับ role ของเครื่อง เช่น:

```bash
sudo hostnamectl set-hostname devops-control
sudo hostnamectl set-hostname app-server
sudo hostnamectl set-hostname monitor-server
```

ให้รันเฉพาะชื่อที่ตรงกับเครื่องนั้น ไม่ใช่ copy คำสั่งเดียวกันไปทุกเครื่อง

ตรวจสอบ:

```bash
hostname
hostnamectl
```

ตัวอย่างที่ผิด:

```text
ทุก VM มี hostname เป็น ubuntu
```

ผลคือเวลา SSH เข้าไปหลายเครื่องหรือดู log จาก monitoring จะไม่รู้ว่า event มาจากเครื่องไหน

## ตั้ง Static IP ด้วย Netplan

ตรวจชื่อ interface:

```bash
ip a
```

คำสั่ง `ip a` ใช้ดูชื่อ network interface และ IP ปัจจุบัน ชื่อ interface อาจไม่ใช่ `ens33` เสมอไป บางเครื่องอาจเป็น `ens160`, `enp0s3` หรือชื่ออื่น ต้องใช้ชื่อที่เห็นจริงในเครื่องนั้น

แก้ไฟล์ netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

ไฟล์ netplan เป็น config network ของ Ubuntu Server ใช้กำหนดว่าจะรับ IP จาก DHCP หรือใช้ static IP เอง ใน Lab นี้แนะนำ static IP เพราะ service หลายตัวต้องอ้าง IP/hostname คงที่

ตัวอย่าง:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.56.20/24
      routes:
        - to: default
          via: 192.168.56.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

ให้แก้ค่าตามเครื่องจริง:

- `ens33` ต้องตรงกับ interface จาก `ip a`
- `192.168.56.20/24` ต้องเป็น IP ของเครื่องนั้น เช่น `app-server`
- `192.168.56.1` ต้องเป็น gateway ของ network ที่ VMware ใช้
- nameserver ใช้ DNS ที่ออก internet ได้ เช่น `8.8.8.8` หรือ router/DNS ใน lab

ตัวอย่างที่ผิด:

```yaml
ethernets:
  ens33:
    dhcp4: no
    addresses:
      - 192.168.1.20/24
```

ถ้า VMware network ของคุณอยู่ subnet `192.168.56.0/24` แต่ตั้ง IP เป็น `192.168.1.20/24` เครื่องจะอยู่ผิด network และติดต่อเครื่องอื่นใน lab ไม่ได้

ตัวอย่างที่ถูก:

```yaml
ethernets:
  ens33:
    dhcp4: no
    addresses:
      - 192.168.56.20/24
```

โดยต้องแน่ใจว่า subnet นี้ตรงกับ VMware network จริง

Apply:

```bash
sudo netplan apply
ip a
ip route
```

`sudo netplan apply` จะนำ config ใหม่ไปใช้ทันที ถ้าแก้ผ่าน SSH แล้ว config ผิด อาจทำให้ connection หลุด ดังนั้นในช่วงแรกควรทำผ่าน console ของ VMware ก่อนจนมั่นใจว่า network ถูกต้อง

หลัง apply ให้ตรวจสองอย่าง:

- `ip a` เห็น IP ที่ตั้งไว้บน interface ที่ถูกต้อง
- `ip route` มี default route ผ่าน gateway ที่ถูกต้อง

ถ้าไม่มี default route เครื่องอาจ ping เครื่องใน subnet เดียวกันได้ แต่ออก internet ไม่ได้ ทำให้ `apt update` หรือ download package ล้มเหลว

## ตั้งค่า /etc/hosts ทุกเครื่อง

```bash
sudo nano /etc/hosts
```

เพิ่ม:

```text
192.168.56.10 devops-control
192.168.56.20 app-server
192.168.56.30 monitor-server
192.168.56.100 k8s-master-01
192.168.56.101 k8s-worker-01
192.168.56.102 k8s-worker-02
```

`/etc/hosts` ใช้ map hostname เป็น IP แบบ local ในแต่ละเครื่อง เหมาะกับ lab เพราะยังไม่ต้องตั้ง DNS server เอง ทุกเครื่องควรมีรายการเดียวกัน เพื่อให้เรียกชื่อกันได้สม่ำเสมอ

ถ้าไม่ตั้ง `/etc/hosts` คุณยังอาจ ping ด้วย IP ได้ แต่คำสั่งที่ใช้ hostname เช่น `ssh devops@app-server`, Prometheus target หรือ Kubernetes config บางส่วนจะ resolve ชื่อไม่ได้

ตรวจสอบหลังแก้:

```bash
getent hosts app-server
getent hosts devops-control
ping -c 4 app-server
```

ตัวอย่างที่ผิด:

```text
192.168.56.20 devops-control
192.168.56.10 app-server
```

IP สลับกันแบบนี้อันตรายกว่าไม่ตั้ง เพราะ command จะไปผิดเครื่องโดยที่ชื่อดูเหมือนถูก

## การทดสอบ

```bash
ping -c 4 app-server
ping -c 4 devops-control
ssh devops@app-server
```

ให้ทดสอบทั้งจากเครื่อง host และระหว่าง VM ถ้าเป็นไปได้:

- จาก PC host ping/SSH เข้า VM
- จาก `devops-control` ping/SSH ไป `app-server`
- จาก `app-server` ping กลับไป `devops-control`

ถ้า ping IP ได้แต่ SSH ไม่ได้ ให้ตรวจว่า SSH service เปิดหรือ firewall block port 22 หรือไม่ ถ้า SSH ได้ด้วย IP แต่ไม่ได้ด้วย hostname ให้ตรวจ `/etc/hosts`

## การตรวจสอบ

- `hostnamectl` แสดงชื่อเครื่องถูกต้อง
- `ip a` แสดง IP ตามที่กำหนด
- ping ข้ามเครื่องได้
- SSH เข้าเครื่องได้

คำสั่งตรวจเพิ่มเติมที่มีประโยชน์:

```bash
systemctl status ssh
ss -tulpen | grep ':22'
getent hosts devops-control
getent hosts app-server
```

ผลที่คาดหวัง:

- `systemctl status ssh` แสดง service เป็น active/running
- `ss` เห็น ssh listen ที่ port 22
- `getent hosts` แสดง IP ตรงกับ network plan

ถ้าผลไม่ตรง อย่าเพิ่งไปบทถัดไป เพราะบทต่อ ๆ ไปจะพึ่ง SSH, hostname และ IP เหล่านี้ทั้งหมด

## ข้อสรุป

พื้นฐาน VM, IP, hostname และ SSH ต้องนิ่งก่อนเริ่ม DevOps Lab เพราะทุกเครื่องมือจะพึ่งพา network และ hostname

เมื่อจบบทนี้ คุณควรมีอย่างน้อยหนึ่งหรือสอง VM ที่พร้อมใช้งานจริง เข้า SSH ได้ และเรียกชื่อกันได้ด้วย hostname ขั้นนี้ดูเหมือนพื้นฐาน แต่เป็นจุดที่ทำให้ Lab ทั้งชุดเสถียร ถ้าต้องเสียเวลาตรงนี้ถือว่าคุ้ม เพราะจะลดปัญหาในบท Docker, GitLab, Monitoring และ Kubernetes อย่างมาก

---
