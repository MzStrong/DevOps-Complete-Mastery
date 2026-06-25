# 16. Lab 13: Kubernetes Cluster ด้วย kubeadm

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

## เตรียมทุก node

ปิด swap:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

โหลด module:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
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

## Init บน k8s-master-01

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.56.100 \
  --pod-network-cidr=192.168.0.0/16
```

ตั้งค่า kubectl:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

ติดตั้ง Calico:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

## Join worker

ใช้ command ที่ได้จาก `kubeadm init` ไปรันบน worker เช่น:

```bash
sudo kubeadm join 192.168.56.100:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

ถ้าลืม:

```bash
kubeadm token create --print-join-command
```

## การตรวจสอบ

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

## ข้อสรุป

Kubernetes ต้องอาศัย Linux, Network, Container Runtime และ CNI ถ้าส่วนใดผิด node หรือ pod จะไม่พร้อมทำงาน

---
