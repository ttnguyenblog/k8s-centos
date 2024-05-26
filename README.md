# Intruction Install K8s on Centos 7

## Steps

* Install container runtime
* Install kubeadm, kubelet, kubectl
* Configuration Cgroup drive
* Configuration load balance
* Init Kubernetes Cluster
* Install CNI
* Add master node, worker node

## Setup

* VM Load Balancer: 10.0.2.4
* VM Master Node: 10.0.1.4
* VM Worker Node: 10.0.0.4

## Configuration

### 1. Environment

- Execute on all nodes
- Turn off SELinux, Firewall

`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld`

- Install Docker

`curl -fsSL https://get.docker.com/ | sh
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl enable docker`

- Setup containerd

`cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
yum install yum-utils -y
yum install containerd.io -y

mkdir /etc/containerd
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
systemctl restart containerd
`
### 2. Init K8S Cluster

- Setup k8s repo:

`cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF`

- Install kubelet kubeadm kubectl

Lưu ý:

* docker: để làm môi trường chạy các container.
* kubeadm: Công cụ khởi tạo Cluster k8s.
* kubelet: Thành phần chạy trên các host, có nhiệm vụ kích hoạt các pod và container trong cụm Cluser của K8S.
* kubectl: Công cụ CLI (Giao diện dòng lệnh) để tương tác với K8s.

`sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet`

- Init Cluster

`sudo kubeadm init --apiserver-advertise-address 10.0.1.4 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU`

Lưu ý:

--apiserver-advertise-address: Địa chỉ của node Master
--pod-network-cidr: Dải địa chỉ sử dụng, phụ thuộc vào công nghệ sử dụng, trong bài sử dụng công nghệ network flannel nên giá trị bằng 10.244.0.0/16

- Khởi tạo ENV để sử dụng câu lệnh kubectl và Pod Network

`mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`
### 3. Join worker node

`kubeadm join 10.0.1.4:6443 --token ukhlng.8500qnhpgclimkn5 \
        --discovery-token-ca-cert-hash sha256:9cb0e67e851c4c3d0f8492684d388ab2a71ad47bc358895e38901c240f569cf4`


1. Kiểm tra và Xóa Deployment hiện có

`kubectl get deployments
kubectl delete deployment nginx-deployment`

2. Kiểm tra và Xóa Service hiện có

`kubectl get services
kubectl delete service nginx-deployment

kubectl get pods --all-namespaces
kubectl get nodes
`












