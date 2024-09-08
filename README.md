# kubernetes
جهت راه اندازی کوبرنتیز به صورت یک مستر و چند ورکر  باید مراحل زیر به ترتیب طی شود لازم به ذکر است تمامی مراحل تا init کردن کلاستر بر روی تمامی ورکر ها و مستر باید انجام شود.
# First step
در اولین قدم  لازم  است تا swap  با دستور زیر غیر فعال گردد و سیستم ریستارت گردد:
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
# Firewall ports needed for communication

The following TCP ports are used by Kubernetes control plane components:
|PORTS|COMPONENTS|
|----|----|
|6443|Kubernetes API server|
|2379-2380|etcd server client API|
|10250|Kubelet API|
|10259|kube-scheduler|
|10257|kube-controller manager|

The following TCP ports are used by Kubernetes nodes:
|PORTS|COMPONENTS|
|----|----|
|10250|Kubelet API|
|30000-32767|NodePort Services|
```
sudo ufw allow 6443/TCP
```
در این مرحله باید حتما این دستورات در فایل ذکر شده ادد شوند:
# Enable and Load Kernel modules
```
cat >> /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```
# Add Kernel settings
اضافه کردن کامند های زیر به کرنل با کامند زیر:
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```
## Install containerd runtime
ما به عنوان اجرا کننده ران تایم ها  از containerd استفاده می کنیم:
```
  apt update
  apt install -y containerd apt-transport-https
  mkdir /etc/containerd
  containerd config default > /etc/containerd/config.toml
  systemctl restart containerd
  systemctl enable containerd
```
# Add apt repo for kubernetes
اضافه کردن ریپو:  
```
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```
# Install Kubernetes components
نصب کامپوننت های کوبرنتیز :
```
  apt update
  apt install -y kubeadm=1.30.0-00 kubelet=1.30.0-00 kubectl=1.30.0-00
```
Initialize Kubernetes Cluster on master1
ایجاد کلاستر کوبرنتیز  از این مرحله به بعد به جز جوین کردن worker ها بر  روی مستر اتفاق می افتد.
لازم به ذکر است  باید از یک vpn  مطمئن استفاده گردد یا یک dns مطمئن:
```
kubeadm init --control-plane-endpoint="192.168.16.1:6443" --upload-certs --apiserver-advertise-address=192.168.16.2 --pod-network-cidr=10.10.0.0/16
```
بعد از ایجاد کلاستر کامند های جوین ورکر ها در خروجی قابل مشاهده است 
### Install Calico network
نصب calico  که  یک  cni است:
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/tigera-operator.yaml
```
### Custom Calico based on your CIDR
```
wget curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/custom-resources.yaml -O
nano custom-resources.yaml
```

```
kind: Installation
apiVersion: operator.tigera.io/v1
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:your-ip-renge-address-for-pods
```
