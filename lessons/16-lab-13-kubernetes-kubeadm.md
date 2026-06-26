# 16. Lab 13: Kubernetes Cluster ด้วย kubeadm

Lab นี้สร้าง Kubernetes cluster เองบน VM ด้วย `kubeadm` เพื่อให้เห็นชั้นพื้นฐานของ Kubernetes จริง ๆ ก่อนใช้ managed Kubernetes หรือ tool ที่ automate ให้ทั้งหมด สิ่งที่ต้องเข้าใจคือ Kubernetes ไม่ได้ทำงานลอย ๆ แต่พึ่ง Linux kernel, network, container runtime, kubelet และ CNI ถ้าชั้นใดชั้นหนึ่งผิด node จะ `NotReady` หรือ pod จะรันไม่ได้

## วัตถุประสงค์

- สร้าง Kubernetes Cluster เองบน VMware
- เข้าใจ control plane, worker node, CNI, container runtime

## คำศัพท์

| คำศัพท์ | ความหมาย |
|---|---|
| Control Plane | ส่วนควบคุม cluster |
| Worker Node | เครื่องที่รัน workload |
| Pod | หน่วยเล็กสุดของ Kubernetes |
| kubelet | agent บน node |
| containerd | container runtime |
| CNI | network plugin ของ Kubernetes |

กลุ่ม resource ที่จะเจอบ่อยหลัง cluster พร้อม:

| กลุ่ม | Resource ที่พบบ่อย | หน้าที่ |
|---|---|---|
| Workload | Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob | รัน application หรืองานใน cluster |
| Network | Service, Ingress, NetworkPolicy | ทำให้ traffic ไปถึง workload และควบคุม network access |
| Config | ConfigMap, Secret | ส่งค่า config หรือข้อมูลลับให้ workload |
| Storage | PersistentVolumeClaim | ขอใช้ storage สำหรับข้อมูลที่ต้องอยู่รอด |
| Cluster | Node, Namespace, HPA, ResourceQuota | จัดการ node, ขอบเขต namespace, autoscale และ quota |

บทนี้เน้นสร้าง cluster ให้พร้อมก่อน ส่วน resource เหล่านี้จะเริ่มใช้จริงใน Lab 14 เป็นต้นไป ดูคำอธิบายรวมได้ในบทคำศัพท์ DevOps

ภาพรวม component:

```text
k8s-master-01
-> kube-apiserver, scheduler, controller-manager, etcd

k8s-worker-01/02
-> kubelet, containerd

ทุก node
-> Linux network config, kernel modules, sysctl, CNI
```

control plane เป็นสมองของ cluster ส่วน worker node เป็นเครื่องที่รัน workload จริง `kubelet` บนทุก node คุยกับ API server และสั่ง container runtime ให้รัน pod

## เตรียมทุก node

ปิด swap:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

ต้องทำบนทุก node ทั้ง master และ worker Kubernetes ต้องการให้ scheduler จัดการ resource ตาม memory จริงของ node ถ้าเปิด swap ไว้ kubelet อาจไม่ทำงานหรือ cluster behavior ไม่ตรงกับที่ Kubernetes คาดหวัง

ตรวจว่า swap ปิดแล้ว:

```bash
free -h
swapon --show
```

`swapon --show` ควรไม่แสดงรายการใด ๆ

ถ้าปิดด้วย `swapoff -a` อย่างเดียว หลัง reboot swap อาจกลับมาเปิด จึงต้อง comment บรรทัด swap ใน `/etc/fstab` ด้วย

โหลด module:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

module เหล่านี้ช่วยให้ Linux kernel รองรับ overlay filesystem และการส่ง traffic ผ่าน bridge network ซึ่งจำเป็นกับ container และ Kubernetes networking

ตรวจว่าโหลดแล้ว:

```bash
lsmod | grep overlay
lsmod | grep br_netfilter
```

ตั้ง sysctl:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

sysctl เหล่านี้ทำให้ packet ที่วิ่งผ่าน Linux bridge ถูก iptables เห็น และเปิด IP forwarding ให้ node ส่ง traffic ระหว่าง pod/network ได้

ตรวจค่า:

```bash
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.ipv4.ip_forward
```

ค่าที่ต้องการ:

```text
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

ถ้าค่านี้ผิด pod network หรือ service routing อาจทำงานไม่ถูก แม้ kubeadm init จะผ่าน

ติดตั้ง containerd:

```bash
sudo apt update
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

containerd คือ container runtime ที่ kubelet ใช้สั่งรัน container ใน pod ขั้นตอนนี้ติดตั้ง runtime และตั้ง cgroup driver ให้ใช้ systemd ซึ่งควรตรงกับ kubelet ในระบบสมัยใหม่

ตรวจ containerd:

```bash
sudo systemctl status containerd
sudo grep SystemdCgroup /etc/containerd/config.toml
```

ควรเห็น:

```text
SystemdCgroup = true
```

ถ้า cgroup driver ไม่ตรงกัน อาจเจอปัญหา kubelet start ไม่ขึ้นหรือ node ไม่ Ready

ถ้าจะให้ Kubernetes pull image จาก private registry แบบ HTTP เช่น `devops-control:5000` ต้องตั้งค่าฝั่ง containerd บนทุก node เพิ่มด้วย ตัวอย่าง:

```bash
sudo mkdir -p /etc/containerd/certs.d/devops-control:5000
cat <<EOF | sudo tee /etc/containerd/certs.d/devops-control:5000/hosts.toml
server = "http://devops-control:5000"

[host."http://devops-control:5000"]
  capabilities = ["pull", "resolve", "push"]
EOF
```

ตรวจใน `/etc/containerd/config.toml` ว่า registry config path ชี้ไปที่ `/etc/containerd/certs.d` ถ้ายังไม่ตั้งให้เพิ่มในส่วน registry ของ CRI plugin ตาม version ของ containerd ที่ใช้ แล้ว restart:

```bash
sudo grep -n "config_path" /etc/containerd/config.toml
sudo systemctl restart containerd
```

ค่าที่ต้องเห็นหรือเพิ่มใน `/etc/containerd/config.toml` คือ:

```toml
[plugins."io.containerd.grpc.v1.cri".registry]
  config_path = "/etc/containerd/certs.d"
```

ถ้าไฟล์มี section นี้อยู่แล้ว ให้เพิ่มหรือแก้เฉพาะบรรทัด `config_path` ไม่ต้องสร้าง section ซ้ำ หลัง restart ให้ทดสอบจาก worker node ด้วย:

```bash
sudo crictl pull devops-control:5000/simple-api:1.0.0
```

ถ้า `crictl` ยังไม่มี ให้ติดตั้งภายหลังได้ แต่สำหรับ lab นี้อย่างน้อยต้องดู log ของ containerd/kubelet เมื่อ pull image ไม่ผ่าน

การตั้ง `/etc/docker/daemon.json` ใช้กับ Docker daemon เท่านั้น ไม่ได้ทำให้ kubelet ที่ใช้ containerd pull image จาก HTTP registry ได้

ติดตั้ง kubeadm/kubelet/kubectl:

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

บทบาทของแต่ละตัว:

- `kubeadm` ใช้ bootstrap cluster เช่น init master และ join worker
- `kubelet` เป็น agent ที่รันบนทุก node
- `kubectl` เป็น CLI สำหรับคุยกับ Kubernetes API

`apt-mark hold` ใช้ป้องกัน package upgrade โดยไม่ตั้งใจ เพราะ version Kubernetes ควรควบคุมให้เข้ากันทั้ง cluster

ตรวจ version:

```bash
kubeadm version
kubelet --version
kubectl version --client
```

ถ้า apt install fail ให้ตรวจ DNS, internet และ repository URL ก่อน

## Init บน k8s-master-01

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.56.100 \
  --pod-network-cidr=192.168.0.0/16
```

คำสั่งนี้รันเฉพาะบน `k8s-master-01` เท่านั้น:

- `--apiserver-advertise-address=192.168.56.100` คือ IP ที่ worker node จะใช้ติดต่อ Kubernetes API server ต้องตรงกับ IP ของ master
- `--pod-network-cidr=192.168.0.0/16` คือ CIDR สำหรับ pod network ที่ใช้กับ Calico ในตัวอย่างนี้

ถ้า advertise address ผิด worker จะ join ไม่ได้หรือ join แล้วติดต่อ API server ไม่เสถียร

หลัง init สำเร็จ kubeadm จะแสดง join command ให้ copy เก็บไว้ เช่น:

```text
kubeadm join 192.168.56.100:6443 --token ... --discovery-token-ca-cert-hash sha256:...
```

ถ้า init fail ให้ดู error ตรง preflight เช่น swap เปิด, port ถูกใช้, container runtime ไม่พร้อม หรือ hostname/network ไม่ถูก

ตั้งค่า kubectl:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

ไฟล์ `admin.conf` คือ kubeconfig สำหรับคุยกับ cluster ผ่าน `kubectl` ถ้าไม่ copy มาที่ `$HOME/.kube/config` คุณจะใช้ `kubectl get nodes` ด้วย user ปกติไม่ได้

ตรวจ:

```bash
kubectl cluster-info
kubectl get nodes
```

ช่วงแรก master อาจขึ้น `NotReady` เพราะยังไม่ได้ติดตั้ง CNI

ติดตั้ง Calico:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

CNI ทำให้ pod network ทำงาน ถ้าไม่มี CNI node จะยัง `NotReady` และ pod จำนวนมากจะ stuck เพราะไม่มี network

หลังติดตั้ง Calico ให้รอแล้วตรวจ:

```bash
kubectl get pods -n kube-system
kubectl get nodes -o wide
```

pod ของ Calico ควรเป็น `Running` และ node ควรเปลี่ยนเป็น `Ready`

ถ้า Calico image pull ไม่ได้ ให้ตรวจ internet/DNS หรือ registry access ถ้า Calico pod crash ให้ดู:

```bash
kubectl logs -n kube-system <calico-pod-name>
kubectl describe pod -n kube-system <calico-pod-name>
```

## Join worker

ใช้ command ที่ได้จาก `kubeadm init` ไปรันบน worker เช่น:

```bash
sudo kubeadm join 192.168.56.100:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

รันคำสั่ง join บน worker node เท่านั้น เช่น `k8s-worker-01`, `k8s-worker-02` ก่อน join ต้องเตรียม worker เหมือน master ทุกอย่างในหัวข้อ “เตรียมทุก node”

สิ่งที่ worker ต้องทำได้ก่อน join:

```bash
ping -c 4 k8s-master-01
curl -k https://192.168.56.100:6443
sudo systemctl status containerd
sudo systemctl status kubelet
```

ถ้า worker ติดต่อ `192.168.56.100:6443` ไม่ได้ ให้ตรวจ firewall, route และ IP ของ master ก่อน

ถ้าลืม:

```bash
kubeadm token create --print-join-command
```

คำสั่งนี้รันบน master เพื่อสร้าง join command ใหม่ token มีอายุจำกัด จึงเป็นเรื่องปกติที่ command เดิมจะใช้ไม่ได้หลังเวลาผ่านไป

หลัง worker join แล้ว กลับมาที่ master ตรวจ:

```bash
kubectl get nodes -o wide
```

ควรเห็น worker node เพิ่มเข้ามา ถ้ายัง `NotReady` ให้ดู kubelet และ CNI

## การตรวจสอบ

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

คำสั่งตรวจเพิ่มเติม:

```bash
kubectl describe node k8s-worker-01
kubectl get pods -n kube-system -o wide
sudo systemctl status kubelet
sudo journalctl -u kubelet -n 100
sudo systemctl status containerd
```

การแปลผลเบื้องต้น:

```text
node NotReady
-> ดู kubectl describe node
-> ดู kubelet log
-> ดู CNI pods ใน kube-system

CoreDNS Pending/ContainerCreating
-> CNI ยังไม่พร้อม หรือ node/network มีปัญหา

worker join ไม่ได้
-> token/hash ผิด, API server เข้าไม่ได้, firewall block 6443

pod image pull ไม่ได้
-> DNS/internet/registry มีปัญหา
```

## Debug node NotReady

เริ่มจาก master:

```bash
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl get pods -n kube-system
```

บน node ที่มีปัญหา:

```bash
sudo systemctl status kubelet
sudo journalctl -u kubelet -f
sudo systemctl status containerd
ip a
ip route
```

จุดที่พบบ่อย:

```text
swap ยังเปิดอยู่
containerd ไม่รัน
SystemdCgroup ยังเป็น false
CNI ยังไม่ติดตั้งหรือ pod CNI crash
worker ติดต่อ API server port 6443 ไม่ได้
hostname/IP ซ้ำกัน
```

อย่าแก้ด้วยการ reset cluster ทันที ให้ดู log และเงื่อนไข node ก่อน เพราะปัญหาส่วนใหญ่เกิดจาก layer พื้นฐานที่แก้เฉพาะจุดได้

## ข้อสรุป

Kubernetes ต้องอาศัย Linux, Network, Container Runtime และ CNI ถ้าส่วนใดผิด node หรือ pod จะไม่พร้อมทำงาน

สิ่งที่ควรจำ:

```text
swap off
kernel module/sysctl พร้อม
containerd พร้อม
kubeadm init เฉพาะ master
CNI ต้องติดตั้งหลัง init
worker join ด้วย token
kubectl/describe/log คือเครื่องมือ debug หลัก
```

หลัง cluster Ready แล้ว บทถัดไปจะเริ่ม deploy application เข้า Kubernetes ด้วย Deployment และ Service

---

<!-- lesson-nav:start -->

---

## บทนำทาง

- บทก่อนหน้า: [15. Lab 12: Logging ด้วย Loki และ Promtail](./15-lab-12-loki-promtail-logging.md)
- สารบัญ: [DevOps Lab Lessons](./README.md)
- บทเรียนถัดไป: [17. Lab 14: Deploy App เข้า Kubernetes](./17-lab-14-deploy-app-to-kubernetes.md)

<!-- lesson-nav:end -->
